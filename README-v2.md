# 🏛️ Arquitectura Hexagonal en el Frontend: Una Guía Detallada

Este documento explora la implementación de la **Arquitectura Hexagonal** (también conocida como Arquitectura de Puertos y Adaptadores) en aplicaciones frontend, destacando su rol en la separación de responsabilidades, la mejora de la mantenibilidad y la escalabilidad del código.

-----

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

| Capa | Responsabilidad Principal |
| :--- | :--- |
| **View (Page) + Components** | Orquestación de la UI, navegación y lógica de presentación (React, Vue, etc.). |
| **Application (Casos de Uso)** | Casos de uso y lógica de negocio pura. |
| **Domain** | Entidades, reglas de negocio, validaciones y contratos (*interfaces* de repositorio). |
| **Infrastructure** | Adaptadores para APIs, almacenamiento (REST, GraphQL, *localStorage*) e implementaciones de los repositorios. |

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
  App.tsx
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
```

### 📂 Resumen rápido (qué hace cada carpeta)

| Carpeta / Archivo | Responsabilidad |
|:------------------|:----------------|
| **App.tsx** | Punto de entrada de la aplicación; composición de dependencias (bootstrapping). |
| **application/use-cases/** | Casos de uso puros; funciones que orquestan la lógica de negocio usando interfaces (repositorios). |
| **domain/entities/** | Modelos inmutables y estructuras centrales del dominio (sin dependencias de infraestructura). |
| **domain/repositories/** | Contratos (interfaces) que definen cómo acceder a datos; puertos de la arquitectura. |
| **domain/value-objects/** | Validaciones y reglas encapsuladas en tipos semánticos (ej: `CourseId`, `CourseTitle`, `CourseDuration`). |
| **infrastructure/** | Implementaciones concretas de repositorios y adaptadores de I/O (REST, GraphQL, localStorage, etc.); adaptadores de la arquitectura. |
| **presentation/pages/ (Views)** | Punto de entrada a nivel de ruta/pantalla, orquesta la UI, compone componentes, maneja navegación y conecta casos de uso, sin lógica de dominio. |
| **presentation/components/** | Piezas de UI reutilizables con estado/efectos de presentación y validaciones de UI. Pueden invocar casos de uso a través de *props* o *hooks*; no contienen lógica de dominio. |

> **Regla clave:** La capa de presentación (pages/components) **usa** casos de uso; la aplicación **depende** de interfaces del dominio; la infraestructura **implementa** esas interfaces. El dominio nunca importa de infraestructura o presentación.

### View vs Component (en React)

```text
View (Page) --> Component --> Use Case --> Repository <--- Impl Repository
```

> La separación entre **View (Page)** y **Component** mejora la reusabilidad, testeabilidad y claridad de responsabilidades:

#### ✨ Buenas Prácticas Clave

  * Inyectar repositorios (interfaces) en los casos de uso para evitar el acoplamiento a implementaciones.
  * Mantener las validaciones en la capa de **Domain** (e.g., *value-objects*).
  * Los adaptadores de **Infrastructure** deben traducir DTOs a entidades de dominio y viceversa.
  * La capa de **Presentation** solo orquesta la interacción y muestra los errores/validaciones provistas por el Dominio/Aplicación.
  * Evitar lógica de negocio compleja en los componentes de UI; usar casos de uso para operaciones complejas.

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
export interface CourseRepository {
  save(course: Course): Promise<void>;
  findById(id: string): Promise<Course | null>;
  // ...
}
```

#### 🧩 Mapeo en Infraestructura (Adaptador)

La capa de **Infrastructure** (el adaptador) es responsable de la **traducción** (mapeo) entre el modelo de persistencia (DTOs externos, filas de DB) y las entidades de **Domain**.

  * **Principio:** El dominio no debe ser condicionado por la estructura de los datos externos (APIs, JSON, etc.).
  * **Ejemplo:** El repositorio debe mapear una `ApiLocation` (con `coords: { lat, lng }`) a la entidad de dominio `Location` (con `latitude`, `longitude`).

-----

### 📝 Estrategia de Refactor (*Legacy* a Hexagonal)

El refactor debe ser incremental y con garantías mediante *tests* de aceptación.

1.  **1️⃣ Añadir *Tests* de Aceptación (E2E):** Cubrir flujos críticos (crear, listar, borrar) y asegurar un entorno determinista (mocks, *seed*).
2.  **2️⃣ Identificar Responsabilidades (*Seams*):** Mapear UI, orquestación, casos de uso, validaciones y persistencia en el código existente.
3.  **3️⃣ Migración Gradual a TypeScript:** Migrar primero tipos esenciales (Entidades, DTOs, *interfaces* de repositorio) para ganar seguridad.
4.  **4️⃣ Definir Interfaz de Repositorio:** Extraer interacciones con la fuente de datos (*localStorage*, API) a una *interface* en *domain* y proveer la implementación concreta en *infrastructure*.
5.  **5️⃣ Implementar Casos de Uso:** Crear funciones puras en *application* que reciban la interfaz del repositorio (inyección de dependencias).
6.  **6️⃣ Mover Reglas de Negocio al Dominio:** Crear *value-objects* y validadores. El dominio lanza errores; los casos de uso los capturan.
7.  **7️⃣ Modularizar `main.ts`:** Reducir a solo la **composición de dependencias** (*Composition Root*): crear repositorios, construir casos de uso y conectar *handlers* de UI.
8.  **8️⃣ Escribir *Tests* Unitarios:** Una vez aislada la lógica, escribir *tests* unitarios para *domain* y *application* (usando repositorios *mock*) y *tests* de integración para los adaptadores (*infrastructure*).

-----

### 🛑 Problemas de la Carpeta `shared` y Tipado Genérico

Una entidad genérica (`User`) centralizada en una carpeta `shared` con muchos campos opcionales (`?` o `null`) puede generar problemas, llevando a un acoplamiento excesivo y dificultades de mantenimiento, escalabilidad y testabilidad.

#### ⚠️ Principales Problemas

  * **Mantenibilidad:** `shared` crece descontroladamente, perdiendo cohesión y dificultando la navegación.
  * **Escalabilidad:** Se fuerza a tener una única tabla/estructura en la DB con campos *nullable* innecesarios, lo que debilita las restricciones, complica las alteraciones en bases de datos grandes y genera dependencias innecesarias entre contextos (e.g., `Admin` impacta en `Auth`).
  * **Testabilidad:** Se puede recurrir a duplicar pruebas o a un `Object Mother` gigante, lo que se soluciona mejor con *Factories* o *Builders* específicos por contexto.

#### ✅ Soluciones Recomendadas

1.  **Tipos Específicos por Contexto:** Definir interfaces explícitas en cada módulo/contexto (`Assignee`) que representen las necesidades reales, extendiendo solo una `BaseUser` mínima si es necesario.
2.  **Modelado Normalizado en BD:** Separar la información obligatoria de la contextual en tablas distintas para reforzar invariantes (restricciones) y evitar `NULLs` innecesarios.
3.  **Reglas para `shared`:**
      * Limitar capas compartidas solo a **Domain** e **Infrastructure**, excluyendo la de **Application** (casos de uso).
      * Aplicar la **Regla de Tres:** Solo mover código a `shared` cuando se haya duplicado al menos dos veces (evaluar al identificar una tercera duplicación).

-----

### 🤔 ¿Cuándo Usar y Cuándo Evitar Hexagonal?

La elección debe ser pragmática; lo fundamental es el desacoplamiento, no el cumplimiento estricto del patrón.

#### 🟢 ¿Cuándo Considerarla?

  * Dominio con **reglas de negocio relevantes o complejas**.
  * Necesidad de **múltiples adaptadores** (REST, GraphQL, CLI, migración de infraestructura).
  * Proyectos a **largo plazo** o con **equipos grandes** donde la claridad de límites es crucial para la colaboración.
  * Requisito de **pruebas unitarias** aisladas (sobre casos de uso/dominio) y *tests* de integración sobre adaptadores.

#### 🔴 ¿Cuándo No Conviene?

  * **Prototipos**, MVPs o proyectos **muy pequeños** y de corta vida.
  * Aplicaciones **extremadamente simples** (CRUD sin lógica de negocio).
  * Cuando la sobrecarga de capas y archivos **complica más que ayuda** al equipo.

#### 💡 Estrategias Intermedias

  * Comenzar con una separación mínima: *domain* (modelos/validaciones) + servicios de acceso a datos.
  * Priorizar principios SOLID, composición y *tests* antes de introducir más capas.
  * Evolucionar incrementalmente, extrayendo repositorios/casos de uso cuando se identifiquen puntos clave de cambio (*seams*).
  * Mantener el *Composition Root* (donde se inyectan dependencias) separado de la lógica de negocio.