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
