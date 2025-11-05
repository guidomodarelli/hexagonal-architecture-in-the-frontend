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
```

### 📂 Resumen rápido (qué hace cada carpeta)

| Carpeta / Archivo | Responsabilidad |
|:------------------|:----------------|
| **App.tsx ó main.ts** | Punto de entrada de la aplicación; composición de dependencias (bootstrapping). |
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

#### FAQs

**¿Para qué sirven `CreateCourseRequest` y `CreateCourseResponse`?**
`CreateCourseRequest` define la estructura de los datos necesarios para crear un curso, mientras que `CreateCourseResponse` especifica el tipo de respuesta que el caso de uso devolverá. En este caso, `CreateCourseResponse` es `void`, indicando que no se espera ningún valor de retorno al completar la operación.

**¿Por qué no simplemente usar `Course` como request?**
Usar `Course` directamente como request podría parecer una solución sencilla, pero no es ideal porque `Course` representa una entidad completa que incluye un `id`, el cual generalmente es generado por el sistema (por ejemplo, una base de datos) al momento de crear el curso. Al definir `CreateCourseRequest`, podemos especificar solo los campos necesarios para la creación del curso, evitando confusión y asegurando que el `id` no sea proporcionado por el cliente.

**¿Y qué pasa con `CreateCourseResponse`?**
En este caso, `CreateCourseResponse` es `void` porque no necesitamos devolver ningún dato específico tras la creación del curso. Sin embargo, en otros casos de uso, podríamos querer devolver información relevante, como el `id` del curso recién creado o un objeto que represente el curso completo. Definir un tipo de respuesta explícito nos permite mantener la flexibilidad para futuros cambios sin afectar la interfaz del caso de uso.

**¿Por qué no usar `Course` como response?**
Usar `Course` como response podría ser útil si quisiéramos devolver el curso completo tras su creación. Sin embargo, en este caso específico, decidimos que no es necesario devolver el curso completo, ya que la operación de creación no requiere que el cliente reciba esa información. Al definir `CreateCourseResponse` como `void`, dejamos claro que no se espera ningún valor de retorno, lo que simplifica la interfaz del caso de uso. Esto también nos permite cambiar la implementación en el futuro sin afectar a los consumidores del caso de uso.

**¿Por qué CreateCourse es una función y no una clase?**
Optamos por una función para mantener la simplicidad y claridad del caso de uso. Las funciones son fáciles de entender y utilizar, especialmente para operaciones que no requieren mantener estado interno. Sin embargo, si el caso de uso necesitara gestionar estado o dependencias complejas, podrías considerar usar una función constructora, una clase, o incluso un patrón de inyección de dependencias para manejar esas necesidades.

**¿Y si necesito dependencias en CreateCourse?**
Si `CreateCourse` requiere dependencias, como un repositorio para guardar el curso, podemos inyectarlas como parámetros de la función. Por ejemplo:

```typescript
export function CreateCourse(
  courseRepository: CourseRepository
): (request: CreateCourseRequest) => Promise<CreateCourseResponse> {
  return async (request: CreateCourseRequest): Promise<CreateCourseResponse> => {
    // Implementación del caso de uso utilizando courseRepository
  };
}
```

**¿Por qué se usan casos de uso en lugar de servicios?**
Los casos de uso representan acciones específicas que un usuario puede realizar en el sistema, encapsulando la lógica de negocio asociada a esas acciones. Esto proporciona una estructura clara y enfocada para la lógica de negocio, facilitando su comprensión y mantenimiento. Los servicios, por otro lado, pueden volverse genéricos y abarcar múltiples responsabilidades, lo que puede complicar la gestión del código. Al utilizar casos de uso, promovemos una arquitectura más modular y orientada a las acciones del usuario. Mientras que los servicios pueden ser útiles para agrupar funcionalidades relacionadas, los casos de uso ofrecen una manera más directa de representar las operaciones del sistema desde la perspectiva del usuario.

#### 🔄 Patrón Repositorio (Puerto)

Para resolver este problema, utilizamos el patrón repositorio.

El Patrón Repositorio define una interfaz (`CourseRepository`) en la capa de **Domain** (puerto) para acceder a los datos, sin exponer los detalles de su implementación.

Dentro de `src/modules/courses/domain/repositories` creamos el archivo `CourseRepository.ts`:

```typescript
import { Course } from '../entities/Course';

export interface CourseRepository {
  save(course: Course): Promise<void>;
  findById(id: string): Promise<Course | null>;
  findAll(): Promise<Course[]>;
  delete(id: string): Promise<void>;
}
```

#### Añade validaciones a la creación del curso

Tenemos nuestro caso de uso para crear cursos, pero ahora queremos añadir validación para evitar guardar cursos con un formato incorrecto.

Por ejemplo, queremos validar que el título del curso tenga un mínimo y un máximo de caracteres. Para ello creamos la validación en la carpeta de dominio.

Dentro de `src/modules/courses/domain/value-objects` creamos el archivo `CourseTitle.ts`:

```typescript
export class CourseTitleNotValidError extends Error {
  constructor(title: string) {
    super(`Course title not valid: ${title}`);
    this.name = 'CourseTitleNotValidError';
  }
}

export const MIN_COURSE_TITLE_LENGTH = 5;
export const MAX_COURSE_TITLE_LENGTH = 100;

export const isCourseTitleValid = (title: string): boolean => {
  return title.length >= MIN_COURSE_TITLE_LENGTH && title.length <= MAX_COURSE_TITLE_LENGTH;
};
```

También podemos crear una función adicional en la capa de dominio para validar el curso completo, lanzando errores si no cumple con los requisitos.

Dentro de `src/modules/courses/domain/entities` creamos el archivo `CourseValidator.ts`:

```typescript
import { Course } from './Course';
import { isCourseIdValid, CourseIdNotValidError } from '../value-objects/CourseId';
import { isCourseTitleValid, CourseTitleNotValidError } from '../value-objects/CourseTitle';
import { isCourseDurationValid, CourseDurationNotValidError } from '../value-objects/CourseDuration';

export const ensureCourseIsValid = (course: Course): void => {
  if (!isCourseIdValid(course.id)) {
    throw new CourseIdNotValidError(course.id);
  }
  if (!isCourseTitleValid(course.title)) {
    throw new CourseTitleNotValidError(course.title);
  }
  if (!isCourseDurationValid(course.duration)) {
    throw new CourseDurationNotValidError(course.duration);
  }
  // Aquí podríamos añadir más validaciones para otros campos del curso
};
```

Llamaríamos a esta función justo antes de guardar el curso dentro de la función `CreateCourse`.

Sin esperar al envío del formulario, queremos proporcionar feedback inmediato mientras el usuario completa los campos; de este modo podemos invocar las funciones de validación individuales y mostrar errores en tiempo real.

```typescript
  import { useEffect } from 'react';
  import { isCourseTitleValid, MIN_COURSE_TITLE_LENGTH, MAX_COURSE_TITLE_LENGTH } from '../../domain/value-objects/CourseTitle';
  import { isCourseDurationValid } from '../../domain/value-objects/CourseDuration';

  useEffect(() => {
    const isTitleValid = isCourseTitleValid(formData.title);
    const isDurationValid = isCourseDurationValid(formData.duration);

    setErrors({
      title: isTitleValid ? null : `El título debe tener entre ${MIN_COURSE_TITLE_LENGTH} y ${MAX_COURSE_TITLE_LENGTH} caracteres.`,
      duration: isDurationValid ? null : `La duración debe ser un número positivo.`,
    });
  }, [formData]);
```

### Programación funcional en arquitectura hexagonal

En el contexto de **JavaScript en frontend**, el uso de clases no es tan común como en otros lenguajes orientados a objetos. Por eso, en este enfoque hemos preferido evitarlas y adoptar un estilo más funcional.

Algunos puntos interesantes de este planteo:

1. **Sin instanciación de objetos**
   No existe un `new Course`. El objeto que recibimos como parámetro ya cumple con la interfaz de curso, así que la responsabilidad se limita a **validar sus propiedades**.

2. **Validaciones como funciones puras**
   Al no haber constructores, no podemos ejecutar las validaciones en la creación de la instancia. En su lugar, definimos funciones independientes —por ejemplo `courseID.ts`, `courseTitle.ts`, etc.— que validan cada aspecto del curso y lanzan una excepción si algo es incorrecto.
   De esta forma, podemos tener una función central `ensureCourseIsValid(course)` que agrupe estas validaciones.
   Además, estas funciones son **reutilizables en la lógica de UI**, manteniendo consistencia entre capas.

3. **Repositorios como funciones sueltas**
   En vez de definir una interfaz de repositorio y luego instanciarla con algo como `createLocalStorageRepository()`, el tipado y la definición del repositorio pueden hacerse directamente como funciones exportadas.
   Así, un caso de uso recibe directamente la función que necesita (`saveCourse`, `findCourseById`, etc.), sin necesidad de encapsularlas en un objeto.

---

👉 En resumen, este enfoque aprovecha la naturaleza funcional de JavaScript para simplificar la arquitectura:

* **Validaciones** como funciones puras, reutilizables y testeables.
* **Repositorios** como funciones tipadas, inyectables sin boilerplate.
* **Casos de uso** que orquestan funciones en lugar de depender de objetos instanciados.

### ¿Tienen sentido los ValueObjects en el frontend?

En programación orientada a objetos, un **ValueObject** encapsula un valor y concentra la lógica asociada a él. Así evitamos que esa lógica termine dispersa en la entidad principal. Por ejemplo, en lugar de que la clase `Course` tenga propiedades primitivas como `string` o `number`, cada propiedad se representa mediante su propio ValueObject: `CourseTitle`, `ImageUrl`, `CourseId`, etc.

La ventaja es que, si necesitamos agregar validaciones (ejemplo: longitud mínima o máxima del título), no lo haríamos en la clase `Course`, sino en el ValueObject correspondiente.

---

### ¿Y en el frontend?

En el frontend podemos aplicar el mismo patrón, pero de forma más ligera y funcional. En lugar de definir clases, podemos encapsular cada valor en un archivo independiente —lo que podríamos llamar un **ValueFile**— que exporta:

1. **El tipo semántico** (alias sobre un primitivo).
2. **Las reglas de validación**.
3. **Las funciones auxiliares** (errores, normalizaciones, etc.).

Ejemplo en **TypeScript** (`CourseTitle.ts`):

```ts
// Tipo semántico
export type CourseTitle = string;

// Constantes de validación
export const COURSE_TITLE_MIN_LENGTH = 5;
export const COURSE_TITLE_MAX_LENGTH = 100;

// Validaciones
export function isCourseTitleValid(title: string): boolean {
  return (
    title.length >= COURSE_TITLE_MIN_LENGTH &&
    title.length <= COURSE_TITLE_MAX_LENGTH
  );
}

// Error asociado
export function CourseTitleNotValidError(title: string): Error {
  return new Error(`Title "${title}" is not valid.`);
}
```

De esta forma, en la interfaz `Course` ya no trabajamos con `string`, sino con `CourseTitle`:

```ts
export interface Course {
  id: CourseId;
  title: CourseTitle;
  imageUrl: ImageUrl;
}
```

---

### Beneficios de este enfoque

* **Semántica fuerte:** el código expresa mejor el dominio (`CourseTitle` vs `string`).
* **Consistencia:** las reglas viven junto al valor que afectan.
* **Evolutivo:** si al principio un valor no tiene lógica extra, basta con un alias de tipo. Si más adelante necesita validaciones, lo ampliamos en el mismo archivo, sin ensuciar la entidad principal.
* **Funcional:** no dependemos de clases ni instancias, pero seguimos respetando la filosofía de los ValueObjects.

---

👉 En resumen: **sí tiene sentido aplicar ValueObjects en el frontend**, pero con un enfoque práctico: tipos alias + funciones puras en archivos separados. Es más liviano que en backend, pero mantiene la semántica y disciplina del dominio.

### Ejemplo real con arquitectura hexagonal

Imaginemos una aplicación en la que debemos mostrar una lista de localizaciones sobre un mapa. Toda la lógica relacionada con pintar los puntos y manejar la interacción con Google Maps (plugins, zoom, popups, etc.) vive en la **UI**, dentro de nuestros componentes de React.

Por otro lado, en la **infraestructura** tenemos un repositorio encargado de obtener esa lista de localizaciones desde una fuente externa. En este ejemplo, la fuente es un **JSON**. Lo importante es que no podemos modificar ni la estructura ni los nombres de los campos que vienen en ese JSON (como ocurriría si la información viniera de un servicio HTTP externo).

#### Estructura de los datos recibidos (API)

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

#### Estructura de nuestro dominio

En cambio, dentro de nuestro **dominio** definimos la entidad de la forma que nosotros queremos trabajar:

```ts
export interface Location {
  latitude: number;
  longitude: number;
  name: string;
}
```

#### Por qué no adaptamos el dominio al API

No deberíamos condicionar nuestro dominio a cómo nos llegan los datos externos.

* **No tenemos control** sobre los servicios externos, y su contrato podría cambiar en cualquier momento.
* Preferimos definir **nuestro propio lenguaje** y nomenclatura en el dominio, de manera consistente con las reglas del negocio y el equipo de desarrollo.

Por eso, el repositorio en infraestructura se encarga de hacer la **traducción** entre el `ApiLocation` y nuestro `Location`. De esta forma aislamos la aplicación de los cambios en la fuente de datos, y mantenemos un dominio limpio, estable y expresivo.

---

## Guías ampliadas (DTOs, puertos y adaptadores)

Para profundizar en dudas comunes al aplicar esta arquitectura en frontend, hay guías dedicadas:

- Dónde van los DTOs, puertos y adaptadores, con convenciones y anti‑patrones: `docs/hex-frontend/DTOs-Puertos-Adaptadores.md`
- Reglas de dependencias e importaciones permitidas + ejemplo ESLint: `docs/hex-frontend/Reglas-de-Dependencias.md`
- Ejemplo completo CreateUser (árbol de carpetas y código): `docs/hex-frontend/Ejemplo-CreateUser.md`
- ADR sobre la decisión de ubicar DTOs en infraestructura: `docs/adr/0001-dtos-en-infraestructura.md`
 - DTOs de aplicación vs infraestructura (cuándo/desde dónde/por qué): `docs/hex-frontend/DTOs-Aplicacion-vs-Infraestructura.md`
 - Ejemplo de lectura (GetUsers) con filtros/paginado: `docs/hex-frontend/Ejemplo-GetUsers.md`
 - Repositorios: contratos de retorno (entidades vs read models) y CQRS: `docs/hex-frontend/Repositorios-Contratos-y-CQRS.md`
 - Manejo de errores (dominio vs infraestructura) + ejemplo OpenSearch: `docs/hex-frontend/Errores-y-Excepciones.md`

Resumen de decisiones clave:
- DTOs externos viven en `infraestructura/api/dto`; la aplicación define sus propios inputs (comandos) y no importa DTOs de infra.
- Puertos (interfaces) en `aplicacion/puertos`; adaptadores (repositorios concretos) en `infraestructura/`.
- Infraestructura puede importar dominio y aplicación; dominio y aplicación no importan infraestructura.
- Separar funciones `api/*` del repositorio mejora testabilidad/modularidad; integrarlas en el repositorio es válido si se prefiere simplicidad.
