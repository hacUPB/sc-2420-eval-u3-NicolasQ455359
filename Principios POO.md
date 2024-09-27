# 1. Abstracción
La abstracción es el principio que permite ocultar los detalles complejos de una implementación y exponer solo lo necesario. 

La estructura Personaje actúa como una clase base que define los atributos y métodos comunes (como nombre, vida, nivel y mostrar_estado). Esto permite que las subclases (Guerrero y Mago) utilicen estos atributos y métodos sin necesidad de implementar sus detalles.

# 2. Herencia
La herencia permite crear nuevas clases basadas en clases existentes, heredando sus atributos y métodos. 

La clase Guerrero y Mago heredan de la clase Personaje. Esto significa que Guerrero y Mago tienen acceso a todos los atributos y métodos de Personaje, como nombre, vida, nivel y mostrar_estado.
La herencia también permite que estas subclases añadan sus propios atributos y métodos. Por ejemplo, Guerrero tiene un atributo adicional fuerza, mientras que Mago tiene mana.

# 3. Encapsulamiento
El encapsulamiento es el principio que restringe el acceso a algunos de los componentes de un objeto, lo que ayuda a proteger el estado interno del objeto. 

# 4. Polimorfismo
El polimorfismo permite que una sola función se comporte de diferentes maneras según el objeto que la invoque. 


