# Servicios de Infraestructura

Aspectos a tener en cuenta:

* Las particularidades de cada adaptador o implementación de nuestras interfaces se especifican mediante **inyección a través del constructor**. Ejemplos:

  * Conexión con la base de datos en los repositorios.
  * *Sender* y credenciales SMTP en el servicio de notificación por correo electrónico.
  * Canal y API Key en el servicio de notificación vía Slack.
  * ...

* Debemos **evitar el acoplamiento estructural** en nuestras interfaces, asegurándonos de no vincular los contratos, la lógica o el flujo de llamadas a conceptos relacionados con una implementación específica.

  * Esto significa que, al diseñar nuestras interfaces, no debemos limitarlas ni condicionarlas a una implementación concreta. En su lugar, deben ser **independientes** de cómo se realice la implementación real.
  * **Beneficios:**

    * ✅ **Flexibilidad:** Permite cambiar o reemplazar implementaciones sin afectar el resto del sistema.
    * ✅ **Pruebas:** Facilita el uso de *mocks* o *stubs* en las pruebas unitarias, ya que las dependencias no están acopladas a una implementación específica.
    * ✅ **Mantenimiento:** El código resulta más fácil de mantener y extender, dado que los cambios en una implementación no impactan otras partes del sistema.

* Para ejecutar los *tests*, utilizaremos **implementaciones falsas (fakes)** de los servicios del sistema, como el servicio de envío de correos electrónicos.

---

## Estructura de carpetas

En esta jerarquía podemos observar que, dentro de cada módulo de nuestra aplicación —*usuarios* y *videos*—, existen tres carpetas principales, una para cada capa de la arquitectura:

```plain
--> entry-point
  --> controller
    --> status
    --> user
    --> video
--> module
  --> shared
    --> infrastructure
  --> user
    --> application
    --> domain
    --> infrastructure
  --> video
    --> application
    --> domain
    --> infrastructure
```

## Módulo o subdominios

Los módulos son agrupaciones de código basadas en los conceptos principales de nuestra aplicación.
En el ejemplo anterior se mostraban los módulos de *videos* y *usuarios*. En cada uno de ellos se agrupan los casos de uso correspondientes, los conceptos de dominio y la infraestructura relacionada.

Este enfoque es importante porque invierte la jerarquía tradicional del directorio. Al agrupar el código por conceptos, la estructura resultante sería similar a la siguiente:

```plain
module/video
--> application
  --> create
    --> VideoCreator
  --> search
    --> VideoSearch
--> domain
  --> Video
  --> VideoId
  --> VideoTitle
  --> VideoRepository
--> infrastructure
  --> repository
    --> MySQLVideoRepository
```

Las razones para situar los conceptos del sistema en el primer nivel de la jerarquía son las siguientes:

* ✅ **Cohesión y facilidad para encontrar lo que se busca:**

  * La aplicación prioriza los conceptos del dominio por encima de los relacionados con la arquitectura del software. Esto hace que el dominio sea más visible y simplifica la localización de los elementos necesarios.
  * Los conceptos pertenecientes a un mismo módulo se encuentran próximos entre sí, lo que facilita moverse entre los distintos componentes que deben modificarse.

* ✅ **Escalabilidad y mantenibilidad del código:**

  * Dividir la aplicación en módulos o subdominios favorece su mantenibilidad a lo largo del tiempo.
  * Este enfoque promueve un cierto grado de aislamiento entre los módulos, ya que cada carpeta de módulo, junto con la carpeta *shared*, contiene todo lo necesario para su funcionamiento, lo que facilita dicho aislamiento.

Podemos observar cómo los elementos se traducen del plano conceptual a términos concretos:

* **Controller:** VideoGetController
* **ApplicationService:** VideoSearcher
* **Model:** Video
* **Repository Contract:** VideoRepository
* **Repository Implementation:** MySQLVideoRepository

## Infraestructura compartida

¿Qué hacemos con los aspectos de infraestructura compartidos entre los distintos módulos?

Nos referimos, por ejemplo, a la configuración de la base de datos, la conexión con esta y otros elementos similares. En estos casos, los ubicamos dentro de un módulo denominado *shared*.

```plain
module/shared/infrastructure
--> config
  --> DBConfig
--> DependencyInjection
  --> SharedModuleDependencyContainer
--> Persistence
  --> ...
```

Asumimos que todos los módulos de la aplicación tendrán acceso a este módulo compartido. Por tanto, si en algún momento trasladamos alguno de ellos a un servicio externo, será necesario llevar también la parte correspondiente del módulo *shared*.

## Dominio compartido

Otro de los conceptos que compartiremos entre módulos serán los pequeños *Value Objects*, que modelan, por ejemplo, los identificadores de nuestras entidades.

Esto se debe a que, por ejemplo, un video podría contener el identificador del usuario que lo ha publicado. Dado que la relación entre una entidad de un módulo y otra entidad de un módulo distinto se establece a través de este identificador —y no mediante una asociación directa—, con el fin de evitar el acoplamiento, resulta necesario compartir el *UserId* entre ambos módulos.
