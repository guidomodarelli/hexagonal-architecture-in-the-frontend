# Servicios de Aplicación

- Son los puntos de entrada de nuestra aplicación. Como se muestra en el gráfico, los controladores de línea de comandos o de nuestra API HTTP invocan a los servicios de aplicación.
- Representan de forma atómica un caso de uso del sistema. En caso de que se produzcan modificaciones en el estado de la aplicación:
  - Actúan como barrera transaccional con el sistema de persistencia.
  - Publican los eventos de dominio correspondientes.
- Coordinan las llamadas a los diferentes componentes del sistema para ejecutar un caso de uso específico.
- Los denominaremos indistintamente servicios de aplicación o casos de uso.

# Servicios de Dominio

Los servicios de dominio agrupan la lógica de negocio que puede reutilizarse desde múltiples servicios de aplicación.

Por ejemplo, supongamos que tenemos dos casos de uso en nuestra aplicación. En ambos necesitamos la lógica de negocio que accede al repositorio de videos para buscar un video específico según su identificador, lanzar una excepción de dominio del tipo **VideoNotFound** si el video no se encuentra y devolverlo en caso de hallarlo. Es importante destacar que quien lanza esta excepción no es la implementación del repositorio.

Para evitar duplicar esta lógica de negocio en ambos servicios de aplicación, lo habitual es extraerla a un servicio de dominio que pueda ser invocado desde los dos casos de uso. Cabe resaltar que los servicios de dominio nunca deben publicar eventos de dominio ni gestionar transacciones; esa responsabilidad recae en el **Application Service** que los invoca, ya que es quien garantiza la atomicidad del caso de uso.

# Diferencias entre usar Servicios de Dominio y de Infraestructura desde Aplicación

## Servicio de Aplicación --> Servicio de Infraestructura

Cuando interactuamos con un servicio de infraestructura desde uno de aplicación, aplicamos el principio de inversión de dependencias. Es decir, el servicio de aplicación recibe por constructor el colaborador de infraestructura, pero lo hace a través de una interfaz definida en el dominio para evitar acoplamientos.

De esta forma, nos aislamos de posibles cambios en las implementaciones de terceros y podemos simular estos colaboradores al ejecutar nuestras pruebas. En los tests, en lugar de inyectar un repositorio en MySQL, inyectamos uno en memoria para que la ejecución sea más rápida.

## Servicio de Aplicación --> Servicio de Dominio

Al invocar un servicio de dominio desde uno de aplicación, nos encontramos con una situación diferente, por lo que debemos tratar este caso de manera distinta. Los servicios de dominio, por definición, contienen únicamente lógica de negocio, por lo que no es necesario desacoplarnos de ellos como sí lo hacemos con los servicios de infraestructura.

Además, al no involucrar operaciones de entrada y salida, no tiene sentido inyectar una implementación diferente del servicio de dominio durante la ejecución de las pruebas. De hecho, nos interesa que los tests pasen por el servicio de dominio al probar un caso de uso, para así cubrirlo de forma indirecta.

Por esta razón, los servicios de dominio no necesitan una interfaz, ya que sería totalmente innecesaria y solo añadiría complejidad mediante un nivel adicional de indirección en el sistema.

Un punto debatible es si la lógica de instanciación del servicio de dominio debe pertenecer al servicio de aplicación o si, a pesar de no tener una interfaz, conviene inyectarlo ya instanciado. Esta decisión se deja al criterio del desarrollador, pues ambas alternativas tienen sus ventajas y desventajas.

## Repasemos

1. Si tenemos un caso de uso llamado **AddProductToCart**, que utiliza un servicio de dominio **ProductToCartAdder**, ¿quién publicará el evento de dominio **ProductAddedToCart**?
   ✅ **AddProductToCart**, es decir, el servicio de aplicación.

2. ¿Es necesario definir una interfaz para nuestro servicio de dominio?
   ✅ **No.** Los servicios de dominio solo deben modificarse cuando cambia la lógica de negocio, y deben probarse siempre de forma indirecta a través de los casos de uso.

3. ¿Encapsularemos siempre nuestra lógica de negocio en servicios de dominio?
   ✅ **No.** Solo lo haremos cuando sea necesario reutilizarla en múltiples casos de uso o cuando queramos encapsularla para simplificar y hacer más legible el servicio de aplicación en casos complejos.
