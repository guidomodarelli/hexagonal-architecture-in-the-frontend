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
  id: UUID;
  username: string;
  avatarUrl: string;
  name?: string; // Opcional
  status?: 'active' | 'inactive'; // Opcional
}
```

Sin embargo, esto añadiría condicionales innecesarios y complicaría el mantenimiento. Preferimos interfaces específicas para cada caso de uso:

```typescript
interface User {
  id: UUID;
  username: string;
  avatarUrl: string;
}

interface Assignee {
  id: UUID;
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
--> View: Componentes de UI (React, Vue, etc.)
--> Application: Casos de uso y lógica de negocio
--> Domain: Entidades y reglas de negocio
--> Infrastructure: Adaptadores para APIs, almacenamiento, etc.
```

```
+---------------------+
|      View           |
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

```
*
```text
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
        CourseName.ts
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

Resumen rápido (qué hace cada carpeta)
- App.tsx: punto de entrada de la aplicación / composición de dependencias.
- application/use-cases: casos de uso puros; funciones que orquestan la lógica usando interfaces (repositorios).
- domain/entities: modelos inmutables y estructuras centrales (sin dependencias de infra).
- domain/repositories: contratos (interfaces) que definen cómo acceder a datos.
- domain/value-objects: validaciones y reglas encapsuladas (por ejemplo, CourseId, CourseName).
- infrastructure/*: implementaciones concretas de los repositorios y adaptadores de I/O (REST, GraphQL, etc.).
- presentation/*: componentes y páginas del UI; deben usar casos de uso o servicios, no lógica de dominio.

Buenas prácticas sugeridas
- Inyectar repositorios en los casos de uso (evitar acoplar a implementaciones).
- Mantener validaciones en value-objects / domain.
- Los adaptadores (infrastructure) traducen DTOs ↔ domain entities.
- La capa presentation solo orquesta interacción y muestra errores/validaciones ya provistas por domain/application.
- Evitar lógica de negocio en componentes UI; usar casos de uso para operaciones complejas.
- Usar hooks personalizados para encapsular lógica de presentación reutilizable.
- Mantener las dependencias unidireccionales: presentation → application → domain → infrastructure.
- Escribir tests unitarios para casos de uso y lógica de dominio, y tests de integración para adaptadores e interacción UI.
```

La arquitectura hexagonal busca separar la lógica de negocio de la lógica de interfaz de usuario (mostrar/ocultar elementos, manejo de inputs, etc.).

Así, si decidimos cambiar de framework en el futuro, la lógica de negocio permanecerá intacta y solo será necesario reescribir la capa de presentación.

Al organizar la aplicación en capas, surge la pregunta sobre dónde ubicar la lógica de interfaz de usuario que maneja el framework: ¿es dominio, aplicación o infraestructura?

Podríamos considerarla infraestructura, ya que el framework es una dependencia externa. Sin embargo, esta capa a menudo sirve como punto de entrada en los tests unitarios, tradicionalmente asociado con la capa de aplicación.

Además, las particularidades de los frameworks frecuentemente limitan la estructura de la aplicación, por ejemplo, requiriendo un archivo `main.ts` dentro de la carpeta `src`. Esto sugiere que la capa de presentación trasciende la simple infraestructura.

## Tú primer caso de uso utilizando arquitectura hexagonal: Aplicación, dominio e infraestructura

Vamos a crear un caso de uso desde cero: la creación de un curso. Tenemos un componente React que se encargará de pintar el formulario, manejar errores de validación, etc. Lo que nos interesa ahora es cómo gestionar la lógica de negocio y cómo guardamos los datos del curso.

### Definiendo la entidad del dominio

Dentro `src/modules/courses/domain/entities` creamos el archivo `Course.ts`:

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

**¿Y para que sirve CreateCourseRequest y CreateCourseResponse?**
`CreateCourseRequest` define la estructura de los datos necesarios para crear un curso, mientras que `CreateCourseResponse` especifica el tipo de respuesta que el caso de uso devolverá. En este caso, `CreateCourseResponse` es `void`, indicando que no se espera ningún valor de retorno al completar la operación.

**¿Por qué no simplemente usar `Course` como request?**
Usar `Course` directamente como request podría parecer una solución sencilla, pero no es ideal porque `Course` representa una entidad completa que incluye un `id`, el cual generalmente es generado por el sistema (por ejemplo, una base de datos) al momento de crear el curso. Al definir `CreateCourseRequest`, podemos especificar solo los campos necesarios para la creación del curso, evitando confusión y asegurando que el `id` no sea proporcionado por el cliente.

**¿Y qué pasa con CreateCourseResponse?**
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

Esta función nos llamaríamos justo antes de guardar el curso dentro de la función `CreateCourse`.

Sin esperar al envío del formulario, queremos proporcionar feedback inmediato mientras el usuario completa los campos; de este modo podemos invocar las funciones de validación individuales y mostrar errores en tiempo real.

```typescript
  import { useEffect, useState } from 'react';
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

