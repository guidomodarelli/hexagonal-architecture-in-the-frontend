# 🧱 Arquitectura Hexagonal

La **Arquitectura Hexagonal** (también conocida como *Ports and Adapters*) busca desacoplar la lógica de negocio central del resto del sistema, permitiendo una mayor flexibilidad, mantenibilidad y capacidad de prueba.

---

## 🧠 Metáfora útil

El Dominio dice:
“Para hacer mi trabajo, necesito que alguien me dé X. Acá está la interfaz de cómo deben dármelo.”

La Aplicación dice:
“Ok, voy a coordinar esta operación usando esas interfaces.”

La Infraestructura dice:
“Yo implemento esa interfaz usando una BD, un API, un archivo o lo que sea.”

---

## ✔ Flujo completo (para que lo tengas visual)

```
Infraestructura (controller/http/json)
          ↓ (transforma)
Aplicación (DTO input)
          ↓ (transforma)
Dominio (Entidad/ValueObject)
```

---

## 🧩 Capas Principales

### **1️⃣ Dominio (Domain)**

Es el núcleo de la aplicación y contiene **la lógica de negocio central**.
Define las **reglas de negocio** y no depende de ninguna otra capa.

Elementos típicos del dominio:

* **Entidades**
* **Value Objects**
* **Servicios de dominio**
* **Tipos e interfaces**
* **Funciones de validación**

📌 *Ejemplo:* `Auth`, `Product`, `Course`, `AuthRepository`, `ProductRepository`, `CourseRepository`

---

### **2️⃣ Aplicación (Application)**

Actúa como **puente entre el dominio y el mundo exterior**.
Se encarga de los **casos de uso** y del **flujo transaccional** de la aplicación.
Aquí es donde se orquesta la comunicación entre las diferentes capas.

📌 *Ejemplo:* `AuthCommand`, `AuthCommandHandler`, `Authenticator`, `CourseCreator`, `CourseRenamer`

---

### **3️⃣ Infraestructura (Infrastructure)**

Contiene las **implementaciones concretas** de los detalles técnicos:

* Llamadas a **APIs**
* Acceso a **bases de datos**
* **Ficheros** e **I/O**
* Código **acoplado a librerías o vendors externos**

Aquí se implementan las **interfaces definidas en el dominio**, traduciéndolas a código funcional según la tecnología utilizada.

📌 *Ejemplo:* `MySQLCourseRepository`, `RedisAuthRepository`

---

## 🧭 Dependencias entre Capas

Las dependencias **siempre deben apuntar hacia el interior**:

> Las capas externas dependen de las internas, **nunca al revés**.

```
❌ Estructura Incorrecta:
⊢- application
  ⊢- AuthCommand
  ⊢- AuthCommandHandler
  ⊢- Authenticator
  ⊢- CourseCreator
  ⊢- CourseRenamer
⊢- domain
  ⊢- Auth
  ⊢- Product
  ⊢- Course
  ⊢- AuthRepository
  ⊢- ProductRepository
  ⊢- CourseRepository
⊢- infrastructure
  ⊢- MySQLCourseRepository
  ⊢- RedisAuthRepository
```

---

## 🏗️ Arquitectura Hexagonal + Vertical Slicing

El concepto de **Vertical Slicing** propone dividir el sistema en **funcionalidades verticales completas**, donde cada *slice* incluye todas las capas necesarias (dominio, aplicación e infraestructura) para entregar un valor funcional al usuario.

Cada módulo es **independiente**, lo que favorece la modularidad, la escalabilidad y el trabajo en paralelo entre equipos.

```
✅ Estructura Recomendada:
⊢- auth
  ⊢- application
    ...
  ⊢- domain
    ...
  ⊢- infrastructure
    ...
⊢- courses
  ⊢- application
    ...
  ⊢- domain
    ...
  ⊢- infrastructure
    ...
...
```

---

## ⚙️ Regla de Dependencia

La **regla de dependencia** establece que **cada capa solo debe conocer las clases de la capa inmediatamente inferior**.

**Orden jerárquico (de exterior a interior):**

> Infraestructura → Aplicación → Dominio

🔒 Este principio permite modificar las capas externas sin afectar las internas.
Por ello, los elementos **más variables o dependientes de terceros** se ubican en la **capa de Infraestructura**.

---

## 🔌 Puertos y Adaptadores

* **Puertos (Ports):**
  Son las **interfaces** definidas en la capa de dominio para desacoplar la lógica de negocio de la infraestructura.
  📍 *Ejemplo:* `UserRepository`

* **Adaptadores (Adapters):**
  Son las **implementaciones concretas** de los puertos, las cuales traducen los contratos definidos en el dominio hacia la lógica específica de un proveedor o tecnología.
  📍 *Ejemplo:* `MySQLUserRepository`

---

🧠 En resumen, la arquitectura hexagonal junto al enfoque de vertical slicing permite desarrollar sistemas **modulares, escalables y fácilmente mantenibles**, donde la lógica de negocio permanece protegida de los detalles técnicos externos.

-----

## 🏛️ Arquitectura Hexagonal en el Frontend: Una Guía Detallada

Esta sección del documento explora la implementación de la **Arquitectura Hexagonal** (también conocida como Arquitectura de Puertos y Adaptadores) en aplicaciones frontend, destacando su rol en la separación de responsabilidades, la mejora de la mantenibilidad y la escalabilidad del código.

### 🧠 Lógica de Negocio en el Frontend

Sí, las aplicaciones frontend **contienen lógica de negocio significativa**. Esta lógica procesa datos, valida entradas del usuario, aplica reglas específicas del dominio y coordina la interacción entre componentes de la interfaz.

**El objetivo clave es mantener esta lógica organizada, testeable y separada de los detalles de presentación (framework UI) e infraestructura (APIs, almacenamiento)**, lo cual se logra mediante la Arquitectura Hexagonal.

  * **Propósito:** Definir y controlar las reglas y procesos de la aplicación, independientemente del *framework*. Incluye validaciones, cálculos y la orquestación de llamadas a servicios externos.
  * **Ejemplo de Validación:** Al crear un nuevo *issue* en GitHub, el frontend valida que no se pueda crear sin título (aunque esta validación también ocurra en el backend).
  * **Estructura de Datos:** La lógica de negocio también guía la estructura de datos específica para cada contexto.
      * **Anti-patrón:** Usar una única interfaz genérica con campos opcionales (`?`), lo que añade condicionales innecesarios y complica el mantenimiento.

        ```typescript
        interface User {
          id: string;
          username: string;
          avatarUrl: string;
          name?: string; // Opcional
          status?: 'active' | 'inactive'; // Opcional
        }
        ```

      * **Recomendación:** Crear interfaces específicas para cada caso de uso. Por ejemplo, la interfaz `User` puede ser distinta a la interfaz `Assignee`, aunque compartan campos base, ya que `Assignee` requiere campos específicos para su contexto (e.g., el desplegable de asignación).

        ```typescript
        interface User {
          id: string;
          username: string;
          avatarUrl: string;
        }

        interface Assignee {
          id: string;
          username: string;
          avatarUrl: string;
          name: string; // Campo requerido para el desplegable
          status: 'active' | 'inactive'; // Campo requerido para el desplegable
        }
        ```

Estas decisiones de diseño confirman la existencia de lógica de negocio en el frontend y subrayan la importancia de mantenerla bien organizada y explícita.

Ambas interfaces comparten campos base (`id`, `username`, `avatarUrl`), pero `Assignee` declara como **obligatorios** los campos `name` y `status`, que son necesarios para su contexto específico (por ejemplo, mostrar información completa en un desplegable de asignación). Esta separación tiene múltiples beneficios:

* **Claridad semántica:** cada interfaz expresa exactamente qué campos son necesarios en su contexto, eliminando ambigüedades y condicionales innecesarios (`if (user.name)`).
* **Seguridad de tipos:** el compilador de TypeScript valida que los campos obligatorios estén presentes, previniendo errores en tiempo de ejecución.
* **Mantenibilidad:** al cambiar los requisitos de un contexto (e.g., agregar un campo a `Assignee`), solo afectamos el código que realmente lo utiliza, sin propagar cambios a todos los usos de `User`.
* **Documentación implícita:** la estructura de datos sirve como documentación viva de las reglas del dominio para ese caso de uso.

En resumen, preferir interfaces específicas por contexto sobre interfaces genéricas con campos opcionales mejora la expresividad del código, reduce errores y facilita la evolución del sistema.

-----

### 🛠️ *Frameworks* y Arquitectura Hexagonal

La Arquitectura Hexagonal es **agnóstica** al *framework*. Puede implementarse con React, Vue, Angular o cualquier otro. La clave está en seguir los principios de separación de responsabilidades y aislar la lógica de negocio de la presentación y la infraestructura.

#### 🗺️ Capas y Responsabilidades

El patrón divide la aplicación en las siguientes capas, con una dirección de dependencia definida ($\rightarrow$):

| Capa                           | Responsabilidad Principal                                                                                     |
| :----------------------------- | :------------------------------------------------------------------------------------------------------------ |
| **View (Page) + Components**   | Orquestación de la UI, navegación y lógica de presentación (React, Vue, etc.).                                |
| **Application (Casos de Uso)** | Casos de uso y lógica de negocio pura.                                                                        |
| **Domain**                     | Entidades, reglas de negocio, validaciones y contratos (*interfaces* de repositorio).                         |
| **Infrastructure**             | Adaptadores para APIs, almacenamiento (REST, GraphQL, *localStorage*) e implementaciones de los repositorios. |

> La dirección de dependencias es: **Presentation** $\rightarrow$ **Application** $\rightarrow$ **Domain**. La capa **Infrastructure** implementa adaptadores que dependen del Dominio (no al revés).

```
+---------------------+
|  View + Components  |
|  (React, Vue, etc.) |
+----------+----------+
           |
           v
+----------+----------+
|    Application      |
| (Use Cases, Logic)  |
+----------+----------+
           |
           v
+----------+----------+
|      Domain         |
| (Entities, Rules)   |
+----------+----------+
           |
           v
+----------+----------+
|   Infrastructure    |
| (APIs, Storage, etc.)|
+---------------------+
```

#### 📁 Estructura de Carpetas Sugerida

Un ejemplo de estructura de módulos (e.g., `courses`):

```text
src/
  App.tsx ó main.ts (Punto de entrada, composición de dependencias)
  modules/
    courses/
      application/
        use-cases/
          CreateCourse.ts
          DeleteCourse.ts
          GetCourses.ts
          UpdateCourse.ts
      domain/
        entities/
          Course.ts (Modelos inmutables, estructuras centrales)
        repositories/
          CourseRepository.ts (Contratos/interfaces de acceso a datos)
        value-objects/
          CourseId.ts, CourseTitle.ts, CourseDuration.ts (Validaciones y reglas encapsuladas)
      infrastructure/
        rest/
          api/
            CourseApi.ts
          repositories/
            CoursePostgreSQLRepository.ts (Implementaciones concretas)
        graphql/
          api/
            CourseGraphQLApi.ts
          repositories/
            CourseGraphQLRepository.ts
      presentation/
        components/
          CourseList.tsx, CourseForm.tsx
        pages/
          CoursesPage.tsx (Páginas / Views)

#### Infraestructura con múltiples implementaciones

Cada módulo puede requerir varias implementaciones concretas para un mismo puerto (HTTP, persistence, message brokers, email, etc.). Organiza esas implementaciones dentro de `infrastructure/<tipo>` creando una carpeta por tecnología (`Axios/`, `Fetch/`, `PostgreSQL/`, `Kafka/`, …). Cada carpeta encapsula configuraciones, clientes e implementaciones de repositorios, mientras que los DTOs externos y su `mapper` permanecen en `infrastructure/api/dto`. Consulta `docs/frontend-hexagonal/Organizacion-Infraestructura-Implementaciones.md` para ver estructuras sugeridas y ejemplos de Axios vs Fetch, PostgreSQL vs MySQL, RabbitMQ vs Kafka, y adaptadores de email.
```

### 📂 Resumen rápido (qué hace cada carpeta)

| Carpeta / Archivo               | Responsabilidad                                                                                                                                                                |
| :------------------------------ | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **App.tsx ó main.ts**           | Punto de entrada de la aplicación; composición de dependencias (bootstrapping).                                                                                                |
| **application/use-cases/**      | Casos de uso puros; funciones que orquestan la lógica de negocio usando interfaces (repositorios).                                                                             |
| **domain/entities/**            | Modelos inmutables y estructuras centrales del dominio (sin dependencias de infraestructura).                                                                                  |
| **domain/repositories/**        | Contratos (interfaces) que definen cómo acceder a datos; puertos de la arquitectura.                                                                                           |
| **domain/value-objects/**       | Validaciones y reglas encapsuladas en tipos semánticos (ej: `CourseId`, `CourseTitle`, `CourseDuration`).                                                                      |
| **infrastructure/**             | Implementaciones concretas de repositorios y adaptadores de I/O (REST, GraphQL, localStorage, etc.); adaptadores de la arquitectura.                                           |
| **presentation/pages/ (Views)** | Punto de entrada a nivel de ruta/pantalla, orquesta la UI, compone componentes, maneja navegación y conecta casos de uso, sin lógica de dominio.                               |
| **presentation/components/**    | Piezas de UI reutilizables con estado/efectos de presentación y validaciones de UI. Pueden invocar casos de uso a través de *props* o *hooks*; no contienen lógica de dominio. |

> **Regla clave:** La capa de presentación (pages/components) **usa** casos de uso; la aplicación **depende** de interfaces del dominio; la infraestructura **implementa** esas interfaces. El dominio nunca importa de infraestructura o presentación.

### View vs Component (en React)

```text
View (Page) --> Component --> Use Case --> Repository <--- Impl Repository
```

> La separación entre **View (Page)** y **Component** mejora la reusabilidad, testeabilidad y claridad de responsabilidades:

#### ✨ Buenas Prácticas Clave

* **Inyección de dependencias:** Pasar repositorios (interfaces) a los casos de uso para evitar el acoplamiento a implementaciones concretas. Esto facilita el testing y el cambio de adaptadores.
* **Validaciones en Domain:** Mantener las validaciones en la capa de **Domain** (e.g., *value-objects*), permitiendo su reutilización tanto en casos de uso como en la UI para feedback inmediato.
* **Traducción en Infrastructure:** Los adaptadores de **Infrastructure** deben traducir DTOs externos a entidades de dominio y viceversa, aislando el dominio de cambios en APIs externas.
* **Presentation como orquestadora:** La capa de **Presentation** solo orquesta la interacción del usuario y muestra errores/validaciones provistas por Domain/Application, sin contener lógica de negocio.
* **Casos de uso para operaciones complejas:** Evitar lógica de negocio compleja en componentes de UI; delegar operaciones complejas a casos de uso bien definidos.
* **Hooks personalizados:** Usar hooks personalizados (en React) para encapsular lógica de presentación reutilizable (estado de formularios, manejo de errores de UI, efectos visuales), manteniendo los componentes limpios.
* **Dirección de dependencias:** `presentation → application → domain`; `infrastructure` implementa adaptadores que dependen del dominio (no al revés). El dominio nunca importa de capas superiores.
* **Testing por capas:** Escribir tests unitarios para casos de uso y lógica de dominio (usando mocks de repositorios), tests de integración para adaptadores (verificando traducción de DTOs) y tests E2E para la interacción UI completa.

##### 🎯 Sobre la Capa de Presentación

La arquitectura hexagonal busca separar la **lógica de negocio** de la **lógica de presentación** (mostrar/ocultar elementos, manejo de inputs, animaciones, routing).

Si decidimos cambiar de framework en el futuro, la lógica de negocio permanecerá intacta y solo será necesario reescribir la capa de presentación.

**¿Dónde ubicar la lógica del framework?**

Podríamos considerarla infraestructura, ya que el framework es una dependencia externa. Sin embargo, esta capa:

* Sirve como **punto de entrada** en aplicaciones frontend (tradicionalmente asociado con la capa de aplicación).
* Tiene **particularidades** que limitan la estructura (e.g., `main.ts` en `src/`, convenciones de routing).
* Orquesta la **experiencia del usuario**, conectando casos de uso con la interfaz visible.

Por ello, tratamos **Presentation** como una capa independiente con responsabilidades claras:

* **Pages (Views):** Punto de entrada de rutas, orquesta componentes, maneja navegación y conecta casos de uso. Sin lógica de dominio.
* **Components:** Piezas de UI reutilizables con estado/efectos de presentación. Invocan casos de uso a través de *props* o *hooks*; no contienen lógica de dominio.
* **Hooks personalizados:** Encapsulan lógica de presentación reutilizable (gestión de formularios, estados de carga, efectos visuales).

Esta separación garantiza que cambiar de React a Vue, Svelte o cualquier otro framework solo afecte la capa de presentación, preservando intacta toda la lógica de negocio en `application` y `domain`.

-----

### 📝 Casos de Uso y Patrón Repositorio

Vamos a crear un caso de uso desde cero: la creación de un curso. Contamos con un componente React encargado de renderizar el formulario y manejar los errores de validación. Lo que nos interesa ahora es definir cómo gestionaremos la lógica de negocio y cómo guardaremos los datos del curso.

#### 🏗️ Creación de un Caso de Uso (Ejemplo: `CreateCourse`)

1.  **Definir la Entidad del Dominio** (`Course.ts` dentro de `src/modules/courses/domain/entities`):

    ```typescript
    export interface Course {
      id: string;
      title: string;
      description: string;
      duration: number; // duración en segundos
    }
    ```

2.  **Definir la Request del Caso de Uso** (`CreateCourse.ts` dentro de `src/modules/courses/application/use-cases`):

    ```typescript
    import { Course } from '../../domain/entities/Course';

    export interface CreateCourseRequest {
      title: string;
      description: string;
      duration: number; // duración en segundos
    }

    export type CreateCourseResponse = void;

    export function CreateCourse(request: CreateCourseRequest): Promise<CreateCourseResponse> {
      // Implementación del caso de uso
    }
    ```

    * Se usa un `CreateCourseRequest` separado de la entidad `Course` para asegurar que el cliente solo proporcione los campos necesarios para la creación (e.g., excluyendo el `id`, que es generado por el sistema).
    * `CreateCourseResponse` es `void` en este ejemplo, pero podría devolver el `id` o el objeto `Course` si fuera necesario.

3.  **Inyectar Dependencias** (enfoque funcional): Los casos de uso son funciones puras que reciben sus dependencias (e.g., el `CourseRepository`) como parámetros.

**¿Cómo guardamos el curso, si desde la capa de aplicación no sabemos nada de infraestructura?**

#### 🛡️ Value Objects y Validaciones

Las validaciones deben residir en la capa de **Domain** (preferiblemente en *value-objects* o funciones de validación).

  * **Value Objects Funcionales:** En el frontend, se pueden implementar como **funciones puras** y **tipos alias** en archivos separados (*ValueFile*), sin necesidad de clases, encapsulando el tipo, las reglas de validación y las funciones auxiliares.
      * Esto permite reutilizar las funciones de validación en la lógica de UI para proporcionar *feedback* inmediato al usuario.
  * **Función de Validación Central:** Se puede definir una función como `ensureCourseIsValid(course)` que agrupa las validaciones individuales del dominio y lanza errores si no se cumplen.

#### 🔄 Patrón Repositorio (Puerto)

Para resolver este problema, utilizamos el patrón repositorio.

El Patrón Repositorio define una interfaz (`CourseRepository`) en la capa de **Domain** (puerto) para acceder a los datos, sin exponer los detalles de su implementación.

```typescript
// src/modules/courses/domain/repositories/CourseRepository.ts
import { Course } from '../entities/Course';

export interface CourseRepository {
  save(course: Course): Promise<void>;
  findById(id: string): Promise<Course | null>;
  findAll(): Promise<Course[]>;
  delete(id: string): Promise<void>;
}
```

#### 🧩 Mapeo en Infraestructura (Adaptador)

La capa de **Infrastructure** (el adaptador) es responsable de la **traducción** (mapeo) entre el modelo de persistencia (DTOs externos, filas de DB) y las entidades de **Domain**.

  * **Principio:** El dominio no debe ser condicionado por la estructura de los datos externos (APIs, JSON, etc.).

##### Ejemplo de Mapeo

**Contexto del ejemplo**

Imaginemos una aplicación que muestra una lista de localizaciones en un mapa. La lógica de renderizado (pintar puntos, zoom, popups, interacción con Google Maps) reside en la **capa de presentación** (componentes React). Por su parte, la **infraestructura** implementa un repositorio que obtiene las localizaciones desde una API externa.

**El problema**

La API externa devuelve los datos con una estructura diferente a la que necesitamos en nuestro dominio. Aquí es donde el repositorio debe actuar como **traductor**.

El repositorio debe mapear una `ApiLocation` (con `coords: { lat, lng }`) a la entidad de dominio `Location` (con `latitude`, `longitude`).

**Estructura de los datos recibidos (vendor externa)**

El repositorio recibe objetos con esta forma:

```ts
export interface ApiLocation {
  coords: {
    lat: number;
    lng: number;
  };
  name: string;
}
```

**Estructura de nuestro dominio**

En cambio, dentro de nuestro **dominio** definimos la entidad de la forma que nosotros queremos trabajar:

```ts
export interface Location {
  latitude: number;
  longitude: number;
  name: string;
}
```

**¿Por qué no adaptamos el dominio al vendor externo?**

No deberíamos condicionar nuestro dominio a cómo nos llegan los datos externos.

* **No tenemos control** sobre los servicios externos, y su contrato podría cambiar en cualquier momento.
* Preferimos definir **nuestro propio lenguaje** y nomenclatura en el dominio, de manera consistente con las reglas del negocio y el equipo de desarrollo.

Por eso, el repositorio en infraestructura se encarga de hacer la **traducción** entre el `ApiLocation` y nuestro `Location`. De esta forma aislamos la aplicación de los cambios en la fuente de datos, y mantenemos un dominio limpio, estable y expresivo.

-----

### Guías ampliadas (DTOs, puertos y adaptadores)

Para profundizar en dudas comunes al aplicar esta arquitectura en frontend, hay guías dedicadas:

- Dónde van los DTOs, puertos y adaptadores, con convenciones y anti‑patrones: `docs/frontend-hexagonal/DTOs-Ports-Adapters.md`
- Reglas de dependencias e importaciones permitidas + ejemplo ESLint: `docs/frontend-hexagonal/Reglas-de-Dependencias.md`
- Ejemplo completo CreateUser (árbol de carpetas y código): `docs/frontend-hexagonal/examples/CreateUser.md`
 - DTOs de aplicación vs infraestructura (cuándo/desde dónde/por qué): `docs/frontend-hexagonal/DTOs-Aplicacion-vs-Infraestructura.md`
 - Ejemplo de lectura (GetUsers) con filtros/paginado: `docs/frontend-hexagonal/examples/GetUsers.md`
 - Repositorios: contratos de retorno (entidades vs read models) y CQRS: `docs/frontend-hexagonal/Repositorios-Contratos-y-CQRS.md`
 - Manejo de errores (dominio vs infraestructura) + ejemplo OpenSearch: `docs/frontend-hexagonal/Errores-y-Excepciones.md`

Resumen de decisiones clave:
- DTOs externos viven en `infraestructura/api/dto`; la aplicación define sus propios inputs (comandos) y no importa DTOs de infra.
- Puertos (interfaces) en `domain/repositories`; adaptadores (repositorios concretos) en `infraestructura/`.
- Infraestructura puede importar dominio y aplicación; dominio y aplicación no importan infraestructura.
