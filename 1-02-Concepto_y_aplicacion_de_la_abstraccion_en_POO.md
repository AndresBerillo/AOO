# Abstracción en Programación Orientada a Objetos con Python

## 1. Conceptos Fundamentales

La abstracción es un pilar fundamental de la POO que nos permite:
- Simplificar sistemas complejos
- Reutilizar código eficientemente
- Crear diseños flexibles y escalables
- Mejorar la seguridad del código

## 2. Implementación en Python

### 2.1 Clases Abstractas

```python
from abc import ABC, abstractmethod

class Animal(ABC):
    def __init__(self, nombre: str):
        self.nombre = nombre
    
    @abstractmethod
    def hacer_sonido(self) -> str:
        """Método que debe ser implementado por todas las subclases."""
        pass
    
    @abstractmethod
    def moverse(self) -> str:
        """Define cómo se mueve el animal."""
        pass

class Perro(Animal):
    def hacer_sonido(self) -> str:
        return "¡Guau!"
    
    def moverse(self) -> str:
        return "Caminando en cuatro patas"

class Pajaro(Animal):
    def hacer_sonido(self) -> str:
        return "¡Pío!"
    
    def moverse(self) -> str:
        return "Volando"
```

#### Diagrama de Clases

```mermaid
classDiagram
    class Animal {
        <<abstract>>
        +nombre: str
        +hacer_sonido(): str
        +moverse(): str
    }
    class Perro {
        +hacer_sonido(): str
        +moverse(): str
    }
    class Pajaro {
        +hacer_sonido(): str
        +moverse(): str
    }
    Animal <|-- Perro
    Animal <|-- Pajaro
```

### 2.2 Ejemplo Práctico: Sistema de Pagos

```python
from abc import ABC, abstractmethod
from datetime import datetime
from typing import List, Dict

class MetodoPago(ABC):
    @abstractmethod
    def procesar_pago(self, monto: float) -> bool:
        pass
    
    @abstractmethod
    def verificar_fondos(self, monto: float) -> bool:
        pass

class TarjetaCredito(MetodoPago):
    def __init__(self, numero: str, fecha_vencimiento: str):
        self.numero = numero
        self.fecha_vencimiento = fecha_vencimiento
    
    def procesar_pago(self, monto: float) -> bool:
        if self.verificar_fondos(monto):
            print(f"Procesando pago de ${monto} con tarjeta {self.numero}")
            return True
        return False
    
    def verificar_fondos(self, monto: float) -> bool:
        # Simulación de verificación de fondos
        return True

class PayPal(MetodoPago):
    def __init__(self, email: str):
        self.email = email
    
    def procesar_pago(self, monto: float) -> bool:
        if self.verificar_fondos(monto):
            print(f"Procesando pago de ${monto} con PayPal ({self.email})")
            return True
        return False
    
    def verificar_fondos(self, monto: float) -> bool:
        # Simulación de verificación de fondos
        return True

class Transaccion:
    def __init__(self, metodo_pago: MetodoPago):
        self.metodo_pago = metodo_pago
        self.fecha = datetime.now()
    
    def ejecutar_pago(self, monto: float) -> bool:
        return self.metodo_pago.procesar_pago(monto)
```

### Diagrama de clases

```mermaid
classDiagram
    class MetodoPago {
        <<abstract>>
        +procesar_pago(monto: float) bool
        +verificar_fondos(monto: float) bool
    }

    class TarjetaCredito {
        -numero: str
        -fecha_exp: str
        +procesar_pago(monto: float) bool
        +verificar_fondos(monto: float) bool
    }

    class PayPal {
        -email: str
        +procesar_pago(monto: float) bool
        +verificar_fondos(monto: float) bool
    }

    class Transaccion {
        -metodo_pago: MetodoPago
        -fecha: datetime
        +ejecutar_pago(monto: float) bool
    }

    MetodoPago <|-- TarjetaCredito
    MetodoPago <|-- PayPal
    Transaccion --> MetodoPago
```

#### Flujo de datos

```mermaid
flowchart TD
    %% Definición de entidades y procesos
    usuario[Usuario] -->|Inicia transacción| procesoTransaccion[Ejecutar Transacción]
    procesoTransaccion -->|Selecciona método de pago| metodoPago[MetodoPago]
    tarjetaCredito[TarjetaCredito] -->|Verifica fondos| verificarFondos[Verificar Fondos]
    paypal[PayPal] -->|Verifica fondos| verificarFondos[Verificar Fondos]
    verificarFondos -->|Fondos disponibles?| resultadoFondos{Fondos disponibles?}

    %% Procesos con condiciones
    resultadoFondos -- Sí --> procesoPago[Procesar Pago]
    resultadoFondos -- No --> mensajeError[Mostrar error: Fondos insuficientes]

    %% Resultado final
    procesoPago -->|Transacción exitosa| mensajeExito[Mostrar mensaje de éxito]
    mensajeError --> usuario
    mensajeExito --> usuario

    %% Métodos de pago específicos
    metodoPago --> tarjetaCredito[TarjetaCredito]
    metodoPago --> paypal[PayPal]
```

#### Diagrama de secuencia

```mermaid
sequenceDiagram
    participant Usuario
    participant Transaccion
    participant MetodoPago
    participant TarjetaCredito
    participant PayPal

    Usuario ->> Transaccion: ejecutar_pago(monto)
    Transaccion ->> MetodoPago: procesar_pago(monto)

    alt Metodo de Pago: Tarjeta de Crédito
        MetodoPago ->> TarjetaCredito: verificar_fondos(monto)
        TarjetaCredito -->> MetodoPago: fondos disponibles (True/False)
        MetodoPago ->> TarjetaCredito: procesar_pago(monto)
        TarjetaCredito -->> MetodoPago: resultado del pago (True/False)
        MetodoPago -->> Transaccion: resultado del pago
    else Metodo de Pago: PayPal
        MetodoPago ->> PayPal: verificar_fondos(monto)
        PayPal -->> MetodoPago: fondos disponibles (True/False)
        MetodoPago ->> PayPal: procesar_pago(monto)
        PayPal -->> MetodoPago: resultado del pago (True/False)
        MetodoPago -->> Transaccion: resultado del pago
    end

    Transaccion -->> Usuario: resultado de la transacción (True/False)
```

### 2.3 Pruebas Unitarias

```python
import unittest

class TestMetodosPago(unittest.TestCase):
    def setUp(self):
        self.tarjeta = TarjetaCredito("1234-5678-9012-3456", "12/25")
        self.paypal = PayPal("usuario@email.com")
    
    def test_pago_tarjeta(self):
        transaccion = Transaccion(self.tarjeta)
        self.assertTrue(transaccion.ejecutar_pago(100.0))
    
    def test_pago_paypal(self):
        transaccion = Transaccion(self.paypal)
        self.assertTrue(transaccion.ejecutar_pago(50.0))

if __name__ == '__main__':
    unittest.main()
```

## 3. Ejercicio Práctico: Sistema de Biblioteca

```python
from abc import ABC, abstractmethod
from typing import List, Optional
from datetime import datetime, timedelta

class MaterialBiblioteca(ABC):
    def __init__(self, codigo: str, titulo: str):
        self.codigo = codigo
        self.titulo = titulo
        self.prestado = False
        self.fecha_devolucion: Optional[datetime] = None
    
    @abstractmethod
    def calcular_fecha_devolucion(self) -> datetime:
        pass
    
    def prestar(self) -> bool:
        if not self.prestado:
            self.prestado = True
            self.fecha_devolucion = self.calcular_fecha_devolucion()
            return True
        return False

class Libro(MaterialBiblioteca):
    def calcular_fecha_devolucion(self) -> datetime:
        return datetime.now() + timedelta(days=14)

class Revista(MaterialBiblioteca):
    def calcular_fecha_devolucion(self) -> datetime:
        return datetime.now() + timedelta(days=7)

class DVD(MaterialBiblioteca):
    def calcular_fecha_devolucion(self) -> datetime:
        return datetime.now() + timedelta(days=3)
```

## 4. Mejores Prácticas

1. **Diseño**:
   - Usar clases abstractas para definir interfaces comunes
   - Mantener la abstracción en el nivel adecuado
   - Separar responsabilidades claramente

2. **Implementación**:
   - Utilizar type hints para mejor documentación
   - Implementar métodos abstractos en todas las subclases
   - Mantener la cohesión alta y el acoplamiento bajo

3. **Testing**:
   - Probar cada implementación concreta
   - Verificar el comportamiento polimórfico
   - Cubrir casos especiales y errores

## 5. Ejercicios Propuestos

1. Implementar un sistema de notificaciones con diferentes medios (email, SMS, push)
2. Crear un simulador de vehículos con distintos tipos de transporte
3. Diseñar un sistema de procesamiento de archivos para diferentes formatos

## Conclusión

La abstracción en Python permite crear sistemas flexibles y mantenibles. La clave está en identificar las características comunes y modelarlas de manera que permitan extensibilidad y reutilización del código.
