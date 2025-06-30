## Actividad : Orquestador local de entornos de desarrollo simulados con Terraform

**1.Ejercicio de evolvabilidad y resolución de problemas:**

Primero agregamos el servicio de database_connector con su version,puerto y enlace de conexión

```
locals {
  common_app_config = {
    app1 = { version = "1.0.2", port = 8081 }
    app2 = { version = "0.5.0", port = 8082 }
    database_connector = {
      version              = "1.0.0",
      port                 = 5432,
      connection_string_tpl = "postgres://user:pass@localhost:5432/mydb"
    }
    # Se pueden añadir más para superar las 700 líneas fácilmente
    # app3 = { version = "2.1.0", port = 8083 }
    # app4 = { version = "1.0.0", port = 8084 }
  }
}
```

En el `main.tf de aplicaction_service` agregué una variable más que será usada en el template de la configuracion json de los servicios

```
data "template_file" "app_config" {
  template = file("${path.module}/templates/config.json.tpl")
  vars = {
    app_name_tpl    = var.app_name
    app_version_tpl = var.app_version
    port_tpl        = var.app_port
    deployed_at_tpl = timestamp()
    message_tpl     = var.global_message_from_root
    connection_string_tpl = var.connection_string_tpl
  }
}
```

Pero para que pueda ser reconocida esta variable se tuvo que declarar en `variables.tf`

```
variable "connection_string_tpl" {
  type        = string
  description = "Cadena de conexión para la bd"
}

```

El valor de dicha variable fue definida en el `main.tf global`. Donde a cada servicio se le agregó esta variable con un valor por defecto de "" para los servicios que no usan esta variable.

```
module "simulated_apps" {
  for_each = local.common_app_config

  source                   = "./modules/application_service"
  app_name                 = each.key
  app_version              = each.value.version
  app_port                 = each.value.port
  base_install_path        = "${path.cwd}/generated_environment/services"
  global_message_from_root = var.mensaje_global # Pasar la variable sensible
  python_exe               = var.python_executable

  #modificacion
  connection_string_tpl = try(each.value.connection_string_tpl,"") 
}
```

Así en el template de la configuracion se especifica que en caso de que el valor sea "" no será considerado como una propiedad más

```
 %{ if connection_string_tpl != "" },
        "connection_string" : "${connection_string_tpl}"
    %{endif}
```

Para verificar que la sintaxis y el formato sea correcto se modificó el archivo `valid_conf.py`. De manera que si se está ejecutando el archivo para el servicio de `database_conector` se analiza de que la propiedad `connection_string` sea un string y comienze con "postgres://" que fue el servicio de base de datos con el que se está trabajando

```python
    if "database_connector" in file_path: #verificar si estamos en la direccion del servicio database_connector
        conn_str = config_data.get("connection_string") #recupera el valor de connection_string
        
        if not isinstance(conn_str, str) or not conn_str.startswith("postgres://"): #si es que no es un string o no cumple con el formato entonces agrega un error
            errors.append(f"[{file_path}] 'connection_string' debe ser una cadena válida y debe comenzar con postgres://")
```

Como adicional se crea un archivo `.db_lock` para el servicio de `database_connector`, para eso verifica que se trate del servicio, define el nombre del archivo, lo crea y por ultimo agrega esta acción al archivo log del servicio

```sh
#start_simulated_service.sh
if [ "$1" == "database_connector" ]; then
  LOCK_FILE="$INSTALL_PATH/.db_lock"
  touch "$LOCK_FILE"
  echo "LOCK FILE creado" >> "$LOG_FILE"
fi
```
