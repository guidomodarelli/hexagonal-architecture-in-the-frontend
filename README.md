# Arquitectura Hexagonal en el Frontend

Este proyecto demuestra cómo implementar la arquitectura hexagonal (también conocida como arquitectura de puertos y adaptadores) en aplicaciones frontend. Esta arquitectura promueve la separación de responsabilidades y mejora la mantenibilidad y escalabilidad del código.

## ¿Existe lógica de negocio en el frontend?

Efectivamente, las aplicaciones frontend contienen lógica de negocio responsable de procesar datos, validar entradas y coordinar la interacción entre componentes de la interfaz. El objetivo es organizar esta lógica manteniéndola separada de la presentación e infraestructura, lo cual se logra mediante la arquitectura hexagonal.

La lógica de negocio define y controla las reglas y procesos que rigen la operación de una aplicación, independientemente del framework utilizado. Incluye validaciones, cálculos y orquestación de llamadas a servicios externos, y debe ser fácilmente testeable y mantenible.

Por ejemplo, al crear una nueva issue en GitHub, el frontend valida que no se pueda crear sin título, aunque esta validación también ocurra en el backend. Esta lógica reside en el frontend.

También consideramos la estructura de datos específica para cada contexto. Los datos de usuario necesarios en la página principal difieren de los requeridos en el desplegable de asignación de issues, requiriendo modelos distintos aunque compartan algunos campos.

Podríamos crear una interfaz TypeScript con campos opcionales:

```typescript
interface User {
  id: string;
  username: string;
  avatarUrl: string;
  name?: string; // Opcional
  status?: 'active' | 'inactive'; // Opcional
}
```

Sin embargo, esto añadiría condicionales innecesarios y complicaría el mantenimiento. Preferimos interfaces específicas para cada caso de uso:

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

Estas decisiones de diseño confirman la existencia de lógica de negocio en el frontend y la importancia de mantenerla bien organizada.

Ambas interfaces comparten campos base, pero `Assignee` incluye campos adicionales necesarios para el contexto específico del desplegable de asignación. Esta separación permite que cada parte de la aplicación utilice la estructura de datos más apropiada, mejorando la claridad y mantenibilidad del código.

## Frameworks + Arquitectura Hexagonal

La arquitectura hexagonal es agnóstica al framework utilizado. Puede implementarse con React, Vue, Angular o cualquier otro framework frontend. La clave está en seguir los principios de separación de responsabilidades y mantener la lógica de negocio aislada de la infraestructura y presentación.

```
--> View (Page): orquestación y navegación de la UI
--> Component: piezas de UI reutilizables y lógica de presentación
--> Application: Casos de uso y lógica de negocio
--> Domain: Entidades y reglas de negocio
--> Infrastructure: Adaptadores para APIs, almacenamiento, etc.
```

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
          Course.ts
        repositories/
          CourseRepository.ts
        value-objects/
          CourseId.ts
          CourseTitle.ts
          CourseDuration.ts
      infrastructure/
        rest/
          api/
            CourseApi.ts
          repositories/
            CourseRepositoryImpl.ts
        graphql/
          api/
            CourseGraphQLApi.ts
          repositories/
            CourseGraphQLRepositoryImpl.ts
      presentation/
        components/
          CourseList.tsx
          CourseForm.tsx
        pages/
          CoursesPage.tsx
```

### Resumen rápido (qué hace cada carpeta)
- App.tsx: punto de entrada de la aplicación / composición de dependencias.
- application/use-cases: casos de uso puros; funciones que orquestan la lógica usando interfaces (repositorios).
- domain/entities: modelos inmutables y estructuras centrales (sin dependencias de infra).
- domain/repositories: contratos (interfaces) que definen cómo acceder a datos.
- domain/value-objects: validaciones y reglas encapsuladas (por ejemplo, CourseId, CourseTitle, CourseDuration).
- infrastructure/*: implementaciones concretas de los repositorios y adaptadores de I/O (REST, GraphQL, etc.).
- presentation/*: componentes y páginas del UI; deben usar casos de uso o servicios, no lógica de dominio.
- presentation/*: páginas (Views) y componentes (Components) del UI; deben usar casos de uso o servicios, no lógica de dominio.

### Buenas prácticas sugeridas
- Inyectar repositorios en los casos de uso (evitar acoplar a implementaciones).
- Mantener validaciones en value-objects / domain.
- Los adaptadores (infrastructure) traducen DTOs ↔ domain entities.
- La capa presentation solo orquesta interacción y muestra errores/validaciones ya provistas por domain/application.
- Evitar lógica de negocio en componentes UI; usar casos de uso para operaciones complejas.
- Usar hooks personalizados para encapsular lógica de presentación reutilizable.
- Dirección de dependencias: presentation → application → domain; infrastructure implementa adaptadores que dependen del dominio (no al revés).
- Escribir tests unitarios para casos de uso y lógica de dominio, y tests de integración para adaptadores e interacción UI.

La arquitectura hexagonal busca separar la lógica de negocio de la lógica de interfaz de usuario (mostrar/ocultar elementos, manejo de inputs, etc.).

Así, si decidimos cambiar de framework en el futuro, la lógica de negocio permanecerá intacta y solo será necesario reescribir la capa de presentación.

Al organizar la aplicación en capas, surge la pregunta sobre dónde ubicar la lógica de interfaz de usuario que maneja el framework: ¿es dominio, aplicación o infraestructura?

Podríamos considerarla infraestructura, ya que el framework es una dependencia externa. Sin embargo, esta capa a menudo sirve como punto de entrada en los tests unitarios, tradicionalmente asociada con la capa de aplicación.

Además, las particularidades de los frameworks frecuentemente limitan la estructura de la aplicación, por ejemplo, requiriendo un archivo `main.ts` dentro de la carpeta `src`. Esto sugiere que la capa de presentación trasciende la simple infraestructura.

### View vs Component (en React)

```text
View (Page) --> Component --> Use Case --> Repository <--- Impl Repository
```

- View (Page): entrada a nivel de ruta/pantalla; orquesta la UI, compone componentes, maneja navegación y conecta casos de uso. Debe evitar lógica de dominio.
- Component: pieza de UI reutilizable con estado/efectos de presentación y validaciones de UI; no contiene reglas de negocio, pero puede invocar casos de uso a través de props o hooks.
- Beneficio: separar View y Component mejora reusabilidad, testeo y claridad de responsabilidades dentro de la capa de presentación.

## Tu primer caso de uso utilizando arquitectura hexagonal: Aplicación, dominio e infraestructura

Vamos a crear un caso de uso desde cero: la creación de un curso. Tenemos un componente React que se encargará de pintar el formulario, manejar errores de validación, etc. Lo que nos interesa ahora es cómo gestionar la lógica de negocio y cómo guardamos los datos del curso.

### Definiendo la entidad del dominio

Dentro de `src/modules/courses/domain/entities` creamos el archivo `Course.ts`:

```typescript
export interface Course {
  id: string;
  title: string;
  description: string;
  duration: number; // duración en segundos
}
```

Dentro de `src/modules/courses/application/use-cases` creamos el archivo `CreateCourse.ts`:

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

¿Cómo guardamos el curso, si desde la capa de aplicación no sabemos nada de infraestructura?

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

### Patrón Repositorio

Para resolver este problema, utilizamos el patrón repositorio. Este patrón define una interfaz para acceder a los datos sin exponer los detalles de la implementación.

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

## Añade Arquitectura Hexagonal a un proyecto existente

### Plan de refactor: de main.js a arquitectura hexagonal

Objetivo: extraer la lógica, introducir TypeScript y aplicar separación de capas manteniendo la aplicación funcional mediante tests de aceptación.

1. Añadir tests de aceptación (E2E)
  - Implementar pruebas que cubran los flujos críticos (crear curso, listar, borrar) con Playwright o Cypress.
  - Asegurar un entorno determinista (seed de localStorage, mocks de red).
  - Usar estos tests como guardia durante el refactor: la app debe seguir pasando las E2E en cada cambio.

2. Identificar seams y responsabilidades en main.js
  - Mapear responsabilidades: UI, orquestación, casos de uso, validaciones, persistencia.
  - Definir puntos de extracción (funciones/objetos que pueden recibir dependencias).

3. Migración incremental a TypeScript
  - Activar allowJs en tsconfig y renombrar gradualmente archivos a .ts/.tsx.
  - Migrar primero tipos esenciales (Course, DTOs, interfaces de repositorio) para ganar seguridad de tipo sin bloquear el desarrollo.

4. Definir interfaz de repositorio (infraestructura)
  - Extraer las interacciones con localStorage a una interfaz en domain/application:
    ```typescript
    export interface CourseRepository {
     save(course: Course): Promise<void>;
     findAll(): Promise<Course[]>;
     findById(id: string): Promise<Course | null>;
     delete(id: string): Promise<void>;
    }
    ```
  - Proveer una implementación concreta (LocalStorageCourseRepository) en infrastructure/.

5. Implementar casos de uso en la capa de aplicación
  - Crear funciones puras que reciban la interfaz del repositorio (inyección de dependencias):
    ```typescript
    export function CreateCourse(repo: CourseRepository) {
     return async (req: CreateCourseRequest): Promise<void> => { /* ... */ };
    }
    ```
  - Mantener la orquestación y la lógica de negocio en use-cases; UI sólo invoca estos use-cases.

6. Mover reglas de negocio al dominio
  - Crear value-objects y validadores (CourseTitle, CourseDuration, CourseId, ensureCourseIsValid).
  - Lanzar errores de dominio desde el dominio; los use-cases los capturan y transforman si es necesario.

7. Modularizar main.ts (composición y bootstrap)
  - main.ts debe encargarse solo de composición: crear repositorios, construir use-cases y conectar handlers de UI.
  - Evitar lógica de negocio y acceso directo a localStorage en main.ts.

8. Escribir tests unitarios una vez aislada la lógica
  - Tests unitarios para domain y application (use-cases) usando repositorios mock.
  - Tests de integración para adaptadores (LocalStorageCourseRepository).

Checklist rápido
- [ ] E2E cubriendo flujos críticos en el código legacy.
- [ ] Interfaz CourseRepository y adaptación a localStorage.
- [ ] Use-cases puros con inyección de dependencias.
- [ ] Domain: value-objects y validadores.
- [ ] main.ts reducido a composición/entorno.
- [ ] Migración completa a TypeScript y suite de tests unitarios funcionando.

Resultado esperado: código desacoplado, testable y preparado para futuros cambios de infraestructura o framework sin tocar la lógica de negocio.

## No siempre es necesario usar arquitectura hexagonal

Existen múltiples enfoques válidos para conseguir aplicaciones mantenibles y testeables; la elección debe ser pragmática. Lo fundamental es el desacoplamiento de dependencias externas y la claridad en las responsabilidades, no el cumplimiento estricto de un patrón. Consideraciones prácticas:

- ¿Cuándo considerar hexagonal?
  - Dominio con reglas de negocio relevantes o complejas.
  - Necesidad de múltiples adaptadores (REST, GraphQL, CLI, pruebas, migración de infra).
  - Equipos grandes o proyectos a largo plazo donde imponer límites claros mejora la colaboración.
  - Requisito de pruebas aisladas (unitarias sobre use-cases/domain y tests de integración sobre adaptadores).

- ¿Cuándo no conviene?
  - Prototipos, MVPs o proyectos muy pequeños y de corta vida.
  - Aplicaciones extremadamente simples (CRUD sin lógica de negocio).
  - Cuando la sobrecarga de capas y archivos complica más que ayuda al equipo.

- Alternativas y estrategias intermedias
  - Empezar con una separación mínima: domain (modelos/validaciones) + servicios de acceso a datos.
  - Aplicar principios SOLID, composición y tests antes de introducir más capas.
  - Evolucionar incrementalmente: identificar seams (puntos donde cambiar dependencias) y extraer repositorios/use-cases cuando sea necesario.

- Reglas prácticas para decidir
  - Priorizar claridad y coste de mantenimiento sobre la pureza del patrón.
  - Introducir hexagonal de forma incremental: primero interfaces y tests, luego implementaciones concretas.
  - Mantener un “composition root” (main/composición) separado de la lógica de negocio.

Resumen: la arquitectura hexagonal es una herramienta poderosa, pero no un requisito universal. Evaluar costo/beneficio según el contexto y preferir una adopción progresiva cuando aporte valor real al mantenimiento, pruebas y evolución del sistema.

## Clean Architecture: los problemas de la carpeta `shared`

```typescript

// Mal

export interface User {
  id: string;
  username: string;
  avatarUrl: string;
  name: string | null; // Opcional
  status?: 'active' | 'inactive' | null; // Opcional
}

// Bien

export interface User {
  id: string;
  username: string;
  avatarUrl: string;
}

export interface Assignee {
  id: string;
  username: string;
  avatarUrl: string;
  name: string; // Campo requerido para el desplegable
  status: 'active' | 'inactive'; // Campo requerido para el desplegable
}
```

### ¿Y si tuviéramos `User` en `shared` y extendiéramos otras interfaces desde ahí?

Tener una única interfaz `User` en `shared` con muchos campos opcionales puede parecer cómodo, pero tiende a enmascarar responsabilidades y a provocar problemas en la base de datos y en la validación. Los principales problemas son: campos nullable que rompen invariantes, validaciones dispersas y tablas con muchos NULL que dificultan aplicar restricciones.

Problema con la aproximación "todo en una tabla" (ejemplo problemático):

```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  username VARCHAR(50) NOT NULL,
  avatar_url VARCHAR(255) NOT NULL,
  name VARCHAR(100),       -- Nullable
  status VARCHAR(10)       -- Nullable
);
```

Desventajas: no puedes garantizar que `name` o `status` existan cuando el dominio los requiere; la semántica se pierde y las restricciones se vuelven débiles.

Alternativas recomendadas

1) Tipos compartidos mínimos + tipos de dominio específicos
- Mantener en `shared` sólo lo estrictamente común (id, username, avatarUrl).
- Definir en cada bounded context/paquete interfaces explícitas que representen las necesidades reales.

```typescript
// shared
export interface BaseUser {
  id: string;
  username: string;
  avatarUrl: string;
}

// domain (assignee)
export interface Assignee extends BaseUser {
  name: string;
  status: 'active' | 'inactive';
}
```

2) Modelado relacional normalizado (recomendado para invariantes fuertes)
- Separar la información obligatoria de la información contextual en tablas distintas.

```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  username VARCHAR(50) NOT NULL,
  avatar_url VARCHAR(255) NOT NULL
);

CREATE TABLE assignees (
  user_id INT PRIMARY KEY REFERENCES users(id),
  name VARCHAR(100) NOT NULL,
  status VARCHAR(10) NOT NULL
);
```

Beneficios: restricciones claras, validaciones a nivel de BD y datos sin NULLs innecesarios. Coste: joins para leer datos compuestos.

3) Capas de mapeo en infrastructure
- La capa de infraestructura debe mapear rows/DTOs a las entidades de dominio que usa la aplicación.

```typescript
// infrastructure/mapper.ts
function mapUserRowToAssignee(row: any): Assignee {
  return {
    id: String(row.user_id),
    username: row.username,
    avatarUrl: row.avatar_url,
    name: row.name,
    status: row.status as 'active' | 'inactive'
  };
}
```

Reglas prácticas
- Mantén `shared` lo más pequeño posible: sólo lo realmente compartido.
- Crea tipos explícitos por contexto en domain/presentation para evitar condicionales y errores en tiempo de ejecución.
- Modela la base de datos para reforzar invariantes (usar tablas separadas o constraints), no para acomodar tipos genéricos con NULLs.
- Usa adaptadores/mappers para traducir entre el modelo de persistencia y las entidades de dominio; la aplicación debe trabajar con tipos claros y validados.

De ese modo evitas la complejidad de un “discrimination mapping” con muchos NULLs y mantienes invariantes claras y verificables tanto en código como en la BD.

---

Todo esto hace que si usamos mal la carpeta `shared`, tengamos 3 grandes problemas:

1. **Mantenibilidad**: Al centralizar todo en `shared`, incluyendo entidades y tipos, se pierde la cohesión que se logra al mantenerlos dentro de módulos específicos. Con el tiempo, `shared` puede crecer descontroladamente, dificultando la navegación y el mantenimiento del código. Esto puede llevar a un punto en el que sea necesario reorganizar el contenido para recuperar la cohesión perdida.

2. **Escalabilidad**: El uso excesivo de una entidad genérica como `User` puede llevar a un acoplamiento excesivo entre diferentes contextos. Por ejemplo, si `User` también representa a un `Assignee`, un `AuthUser` o cualquier otro concepto, la tabla correspondiente en la base de datos puede terminar con múltiples campos opcionales o nulos. Esto no solo complica el diseño, sino que también introduce problemas de rendimiento y mantenimiento.

Cuando la tabla crece con campos adicionales para soportar nuevos conceptos, realizar alteraciones en una base de datos con millones de registros puede ser costoso y arriesgado. Además, los cambios en un contexto pueden bloquear o afectar a otros, como cuando se necesita modificar `User` para `Admin` y esto impacta en `Auth`. Este diseño incrementa la complejidad y dificulta la evolución del sistema.

Por ejemplo, ¿deberíamos tener un único endpoint `/user` o múltiples endpoints específicos como `/assignee`, `/auth-user`, etc.? Optar por endpoints separados facilita la gestión y el mantenimiento, ya que cada uno puede manejar un contexto específico. Sin embargo, al centralizar todo en un único endpoint, es común que se introduzca lógica adicional para identificar el tipo de usuario mediante parámetros, lo que puede derivar en controladores más complejos y difíciles de mantener.

Incluso si los controladores están separados, compartir una base de datos con una estructura genérica y mezclada puede generar dependencias innecesarias y aumentar la complejidad. Esto dificulta la implementación de cambios y la optimización de las consultas, afectando la escalabilidad y el rendimiento del sistema.

3. **Testabilidad**: Si tenemos `User` y `Assignee`, es probable que terminemos duplicando pruebas para cubrir ambos casos, ya que cada uno puede tener sus propias opciones y validaciones. Esto puede llevar a redundancia y a un aumento innecesario en la cantidad de pruebas.

Para abordar este problema, podríamos considerar el uso del patrón Object Mother. Sin embargo, este enfoque puede complicarse rápidamente. Por ejemplo, si necesitamos crear un `User` con o sin `name`, tendríamos que añadir métodos específicos para cada caso. Si además tenemos 10 tipos diferentes de usuarios, podríamos terminar con 10 Object Mothers separados o con un único Object Mother gigante lleno de métodos para manejar todas las combinaciones posibles.

Una solución más escalable sería utilizar Factories o Builders específicos para cada tipo de usuario. Esto permite encapsular la lógica de creación de objetos de manera más modular y evita la proliferación de métodos en un único Object Mother. Además, facilita la personalización de los objetos creados para cada contexto, mejorando la claridad y la mantenibilidad de las pruebas.

Por ejemplo:

```typescript
// Factory para User
export const createUser = (overrides: Partial<User> = {}): User => ({
  id: 'default-id',
  username: 'default-username',
  avatarUrl: 'default-avatar-url',
  ...overrides,
});

// Factory para Assignee
export const createAssignee = (overrides: Partial<Assignee> = {}): Assignee => ({
  id: 'default-id',
  username: 'default-username',
  avatarUrl: 'default-avatar-url',
  name: 'default-name',
  status: 'active',
  ...overrides,
});
```

Con este enfoque, las pruebas pueden centrarse en los casos específicos de cada tipo de usuario, evitando duplicación y manteniendo el código de prueba más limpio y enfocado.

Sin embargo, contar con un módulo `shared` puede ser muy útil para evitar la duplicación de código y simplificar ciertas partes del sistema. Para mantenerlo bajo control y evitar que se convierta en un repositorio desorganizado, seguiremos estas reglas:

1. **Capas compartidas limitadas**: Solo compartiremos las capas de **dominio** e **infraestructura**, excluyendo la capa de **aplicación** (casos de uso). Esto asegura que la lógica específica de cada contexto permanezca encapsulada y no se propague innecesariamente.

2. **Regla de tres**: Solo moveremos código a `shared` cuando se haya duplicado al menos dos veces. Es decir, al identificar una tercera duplicación, evaluaremos si es apropiado centralizar ese código en `shared`. Este enfoque evita la sobreingeniería prematura y asegura que solo se compartan elementos que realmente lo ameriten.

Siguiendo estas reglas, el módulo `shared` se mantendrá limpio, enfocado y útil, sin convertirse en un punto de acoplamiento excesivo o en una fuente de complejidad innecesaria.

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
