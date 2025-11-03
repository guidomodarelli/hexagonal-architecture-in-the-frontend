# 🏗️ Servicios de Infraestructura

## 🔍 Aspectos clave a tener en cuenta

* Las particularidades de cada **adaptador** o **implementación** de nuestras interfaces se definen mediante **inyección a través del constructor**.
  **Ejemplos:**

  * Conexión con la base de datos en los repositorios.
  * *Sender* y credenciales SMTP en el servicio de notificación por correo electrónico.
  * Canal y API Key en el servicio de notificación vía Slack.
  * Otros servicios análogos.

---

### 🚫 Evitar el acoplamiento estructural

Debemos **evitar el acoplamiento estructural** en nuestras interfaces, garantizando que los contratos, la lógica y el flujo de llamadas **no dependan** de una implementación específica.

* Esto implica que las interfaces deben diseñarse de forma **independiente**, sin verse condicionadas por cómo se implementarán.
* **Beneficios principales:**

  * ✅ **Flexibilidad:** Permite intercambiar implementaciones sin afectar otras partes del sistema.
  * ✅ **Pruebas:** Facilita el uso de *mocks* o *stubs* en tests unitarios.
  * ✅ **Mantenimiento:** Mejora la extensibilidad y la capacidad de mantener el código, reduciendo el impacto de los cambios.

---

### 🧪 Pruebas

Para ejecutar los *tests*, se utilizarán **implementaciones falsas (*fakes*)** de los servicios del sistema, como por ejemplo el servicio de envío de correos electrónicos.

---

## 🗂️ Estructura de carpetas

Dentro de cada módulo de la aplicación —*usuarios* y *videos*— existen tres carpetas principales, una por cada capa de la arquitectura:

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

---

## 🧩 Módulos o subdominios

Los **módulos** agrupan el código en función de los conceptos centrales de la aplicación.
En el ejemplo anterior, los módulos *videos* y *usuarios* contienen sus casos de uso, sus entidades de dominio y la infraestructura correspondiente.

Este enfoque **invierte la jerarquía tradicional de carpetas**, priorizando el dominio sobre la arquitectura.

### 📁 Ejemplo de estructura modular

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

---

### 💡 Ventajas del enfoque modular

* ✅ **Cohesión y localización eficiente**

  * La aplicación resalta los conceptos de dominio por encima de la arquitectura.
  * Los elementos relacionados entre sí están próximos, facilitando su mantenimiento y comprensión.

* ✅ **Escalabilidad y mantenibilidad**

  * Dividir la aplicación en módulos o subdominios mejora la mantenibilidad.
  * Cada módulo (junto con *shared*) contiene todos los elementos necesarios para su funcionamiento, favoreciendo el aislamiento.

---

### 🔄 Correspondencia entre elementos conceptuales y concretos

| Concepto                      | Implementación         |
| ----------------------------- | ---------------------- |
| **Controller**                | `VideoGetController`   |
| **ApplicationService**        | `VideoSearcher`        |
| **Model**                     | `Video`                |
| **Repository Contract**       | `VideoRepository`      |
| **Repository Implementation** | `MySQLVideoRepository` |

---

## ⚙️ Infraestructura compartida

Los elementos de infraestructura comunes a varios módulos (como la configuración y conexión a la base de datos) se ubican dentro del módulo **shared**.

```plain
module/shared/infrastructure
--> config
  --> DBConfig
--> DependencyInjection
  --> SharedModuleDependencyContainer
--> Persistence
  --> ...
```

Todos los módulos acceden a este espacio compartido.
Si alguno se traslada a un servicio independiente, deberá llevar consigo la parte correspondiente de *shared*.

---

## 🧱 Dominio compartido

Determinados *Value Objects* —como los identificadores de entidades— deben compartirse entre módulos.
Por ejemplo, un *video* puede contener el identificador del *usuario* que lo publicó.

Como las entidades de distintos módulos se relacionan mediante **identificadores** y no mediante asociaciones directas, el uso compartido de objetos como `UserId` **evita el acoplamiento** entre módulos.

---

## 🧭 Repaso general

1️⃣ **¿Cómo especificar el canal de Slack para notificaciones de nuevos videos?**
✅ Mediante **inyección de parámetros en el constructor**, ya que no forma parte de la interfaz.

2️⃣ **¿Cómo evitar el envío real de correos electrónicos al ejecutar tests?**
✅ Utilizando un **mock** del componente de infraestructura o **inyectando una implementación *fake***.

3️⃣ **¿Cuándo ocurre el acoplamiento estructural?**
✅ Cuando, pese a usar una **interfaz**, esta refleja la **semántica de una implementación concreta**, obliga a seguir un orden de llamadas o está influida por una implementación específica.
