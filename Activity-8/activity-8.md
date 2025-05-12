## Actividad: El patrón Arrange-Act-Assert

En este actividad se muestra el uso del patrón AAA en las pruebas unitarias

**Método para vaciar el carrito**

El método vaciar_carrito borra todos los productos del carrito, reseteando su lista de items a una lista vacía.

```python
#src/carrito.py
 def vaciar_carrito(self):
        self.items = []

```

Test

```python
#tests/test_carrito.py
def test_vaciar_carrito(carrito):

    # Arrange
    producto = ProductoFactory(nombre="Tablet", precio=500.00)
    producto2 = ProductoFactory(nombre="smartphone", precio=200.00)
    producto3 = ProductoFactory(nombre="smartTv", precio=1200.00)
    carrito.agregar_producto(producto)
    carrito.agregar_producto(producto2)
    carrito.agregar_producto(producto3)

    # Act
    carrito.vaciar_carrito()

    # Assert
    lista = carrito.obtener_items()

    assert lista == []
    assert len(lista) == 0


```

**Descuento por compra mínima**

Este método aplica un descuento condicional solo si el total de la compra supera un valor mínimo.

```python

    def aplicar_descuento_condicional(self, porcentaje, minimo):
        if self.calcular_total() >= minimo:
            return self.aplicar_descuento(porcentaje)
        else:
            return self.calcular_total()

```

Test

```python

def test_descuento_condicional_exitoso(carrito):

    # Arrange
    producto = ProductoFactory(nombre="Television", precio=1500.00)
    producto2 = ProductoFactory(nombre="Licuadora", precio=500.00)
    producto3 = ProductoFactory(nombre="Tostadora", precio=250.00)
    carrito.agregar_producto(producto)
    carrito.agregar_producto(producto2)
    carrito.agregar_producto(producto3)

    # Act
    total_a_pagar = carrito.aplicar_descuento_condicional(
        porcentaje=15, minimo=500)

    # Assert

    assert total_a_pagar == 1912.50


def test_descuento_condicional_fallido(carrito):

    # Arrange
    producto = ProductoFactory(nombre="Television", precio=250.00)
    producto2 = ProductoFactory(nombre="Licuadora", precio=100.00)
    producto3 = ProductoFactory(nombre="Tostadora", precio=50.00)
    carrito.agregar_producto(producto)
    carrito.agregar_producto(producto2)
    carrito.agregar_producto(producto3)

    # Act
    total_a_pagar = carrito.aplicar_descuento_condicional(
        porcentaje=15, minimo=500)

    # Assert

    assert total_a_pagar != 340


```

**Manejo de stock en producto**

Este método agrega productos al carrito y asegura que no se agregue más cantidad de la que haya disponible en el stock.

```python
def agregar_producto(self, producto, cantidad=1):
        """
        Agrega un producto al carrito. Si el producto ya existe, incrementa la cantidad.
        """

        if cantidad > producto.stock:
            raise ValueError("No hay suficiente stock")
        for item in self.items:
            if item.producto.nombre == producto.nombre:
                if cantidad + item.cantidad <= producto.stock:
                    item.cantidad += cantidad
                    return
                else:
                    raise ValueError("No hay suficiente stock")
        self.items.append(ItemCarrito(producto, cantidad))
```

Test

```python

def test_agregar_producto_en_stock(carrito):

    # Arrange
    producto = ProductoFactory(nombre="Television", precio=250.00, stock=5)

    # Act
    carrito.agregar_producto(producto, 4)

    # Assert
    item = carrito.obtener_items().pop()
    assert item.cantidad == 4


def test_agregar_producto_no_en_stock(carrito):
    # Arrange
    producto = ProductoFactory(nombre="Television", precio=250.00, stock=5)

    # Act
    with pytest.raises(ValueError) as exc_info:
        carrito.agregar_producto(producto, 6)

    # Assert
    assert str(exc_info.value) == "No hay suficiente stock"
```

**Ordenar items del carrito**

Este método ordena los productos del carrito en base a un criterio definido (precio o nombre).

```python
    def obtener_items_ordenados(self, criterio: str):
        if criterio not in ['precio', 'nombre']:
            raise ValueError("Criterio no definido")

        if criterio == 'precio':
            return sorted(self.items, key=lambda item: item.producto.precio)

        if criterio == 'nombre':
            return sorted(self.items, key=lambda item: item.producto.nombre)

```

Test

```python

def test_ordenar_lista_de_productos_criterio_definido(carrito):
    # Arrange

    producto = ProductoFactory(nombre="Television", precio=250.00)
    producto2 = ProductoFactory(nombre="Licuadora", precio=100.00)
    producto3 = ProductoFactory(nombre="Tostadora", precio=50.00)
    carrito.agregar_producto(producto)
    carrito.agregar_producto(producto2)
    carrito.agregar_producto(producto3)

    # Act

    list_by_Price = carrito.obtener_items_ordenados('precio')
    list_by_Name = carrito.obtener_items_ordenados('nombre')

    # Assert
    assert [item.producto.precio for item in list_by_Price] == [
        50.00, 100.00, 250.00]
    assert [item.producto.nombre for item in list_by_Name] == [
        'Licuadora', 'Television', 'Tostadora']

```

**Uso de Pytest Fixtures**

Las fixtures se usan para crear configuraciones que se ejecutan automáticamente antes de cada prueba. En este caso, crea un carrito y un producto genérico que se usa en varias pruebas.

```python
   
@pytest.fixture
def carrito():
    return Carrito()

@pytest.fixture
def producto_generico():
    return ProductoFactory(nombre="Television", precio=250.00, stock=5)

def test_agregar_producto_no_en_stock(carrito,producto_generico):
    # Arrange
    producto = producto_generico

    # Act
    with pytest.raises(ValueError) as exc_info:
        carrito.agregar_producto(producto, 6)

    # Assert
    assert str(exc_info.value) == "No hay suficiente stock"



```

**Pruebas parametrizadas**

Las pruebas parametrizadas permiten ejecutar el mismo test con diferentes entradas. En este caso, se prueba la aplicación de descuentos con diferentes porcentajes.

```python

@pytest.mark.parametrize("descuento,pagar", [(60, 400), (20, 800), (30, 700), (50, 500)])
def test_probar_descuentos_parametrize(descuento, pagar, carrito: Carrito):
    # Arrange

    producto = ProductoFactory(nombre="Television", precio=200.00, stock=5)
    producto2 = ProductoFactory(nombre="Licuadora", precio=100.00, stock=5)
    producto3 = ProductoFactory(nombre="Tostadora", precio=50.00, stock=5)
    carrito.agregar_producto(producto, 3)
    carrito.agregar_producto(producto2, 3)
    carrito.agregar_producto(producto3, 2)

    # Act

    esperado = carrito.aplicar_descuento(descuento)

    # Assert
    assert esperado == pagar
```

**Calcular impuestos en el carrito**

Este método calcula el monto de los impuestos basado en un porcentaje proporcionado.

```python
def calcular_impuestos(self, porcentaje):
        """
        Calcula el valor de los impuestos basados en el porcentaje indicado.

        Args:
            porcentaje (float): Porcentaje de impuesto a aplicar (entre 0 y 100).

        Returns:
            float: Monto del impuesto.

        Raises:
            ValueError: Si el porcentaje no está entre 0 y 100.
        """
        if porcentaje < 0 or porcentaje > 100:
            raise ValueError("El porcentaje debe estar entre 0 y 100")
        total = self.calcular_total()
        return total * (porcentaje / 100)

```

Test

```python
import pytest
from src.carrito import Carrito
from src.factories import ProductoFactory

# Ejercicio 7


@pytest.mark.parametrize("taza,impuesto", [(3.0, 30), (5.0, 50), (10.0, 100)])
def test_calcular_impuestos(taza, impuesto):
    # Arrange
    carrito = Carrito()
    producto = ProductoFactory(nombre="Television", precio=200.00, stock=5)
    producto2 = ProductoFactory(nombre="Licuadora", precio=100.00, stock=5)
    producto3 = ProductoFactory(nombre="Tostadora", precio=50.00, stock=5)
    carrito.agregar_producto(producto, 3)
    carrito.agregar_producto(producto2, 3)
    carrito.agregar_producto(producto3, 2)

    # Act

    impuesto_calculado = carrito.calcular_impuestos(taza)

    # Assert
    assert impuesto == impuesto_calculado
```

**Aplicar cupón de descuento con límite máximo**

Este método aplica un descuento basado en un porcentaje, asegurándose de que el descuento no exceda un valor máximo.

```python
def aplicar_cupon(self, descuento_porcentaje, descuento_maximo):
        """
        Aplica un cupón de descuento al total del carrito, asegurando que el descuento no exceda el máximo permitido.

        Args:
            descuento_porcentaje (float): Porcentaje de descuento a aplicar.
            descuento_maximo (float): Valor máximo de descuento permitido.

        Returns:
            float: Total del carrito después de aplicar el cupón.

        Raises:
            ValueError: Si alguno de los valores es negativo.
        """
        if descuento_porcentaje < 0 or descuento_maximo < 0:
            raise ValueError("Los valores de descuento deben ser positivos")

        total = self.calcular_total()
        descuento_calculado = total * (descuento_porcentaje / 100)
        descuento_final = min(descuento_calculado, descuento_maximo)
        return total - descuento_final
```

Test

```python
import pytest
from src.carrito import Carrito
from src.factories import ProductoFactory

# Ejercicio 8


def test_aplicar_cupon_con_limite():
    """
    Red: Se espera que al aplicar un cupón, el descuento no supere el límite máximo.
    """
    # Arrange
    carrito = Carrito()
    producto = ProductoFactory(nombre="Producto", precio=200.00)
    carrito.agregar_producto(producto, cantidad=2)  # Total = 400

    # Act
    total_con_cupon = carrito.aplicar_cupon(
        20, 50)  # 20% de 400 = 80, pero límite es 50

    # Assert
    assert total_con_cupon == 350.00

```
