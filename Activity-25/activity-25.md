## Actividad 25

2. Empaquetado y publicación con Docker
 2.1 Estructura multistage:

- Etapa builder: En esta estapa se instalan todas las dependencias necesarias para ejecutar la aplicación con el flag de --user para indicar de que se las colocas en root/local
- Etapa de producción: Se copia lo mismo del builder pero solo lo necesario haciendo de que la imagen sea más ligera
 2.2 Variables de entorno
- PYTHONDONTWRITEBYTECODE=1: Evita la generación de archivos .pyc
- PYTHONUNBUFFERED=1: permite que los logs se muestren en la consola
 Entrypoint: es importante porque define la ejecución principal del contenedor

2.2 Comandos de construcción

```bash
docker build -t ejemplo-microservice:0.1.0 .
```

- Construye la imagen llamada "ejemplo-microservicio" con la versión 0.1.0 del Dockerfile ubicado en la ruta actual.
Cuando se le agrega el flag `--cache` es para indicar de que se vuelva a construir sin importar los datos anteriores(cache) para mitigar los errores de construcciones pasadas
