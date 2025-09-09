# Mejora los tests con Object Mother Pattern

El patrón **Object Mother** facilita y agiliza la creación de datos falsos en nuestras pruebas. En lugar de instanciar manualmente cada objeto o definir valores fijos dentro de los tests, utilizamos funciones que generan datos dinámicos. Para ello podemos apoyarnos en librerías como **Faker**, que se encarga de producir valores aleatorios, o **Fishery**, que nos permite construir funciones de tipo *Factory*.

Además de aportar rapidez en la preparación de los datos, la aleatoriedad en su generación añade **robustez y fiabilidad**, evitando que siempre probemos con los mismos valores. Otro beneficio importante es la **legibilidad**: al trabajar con objetos que encapsulan conjuntos de datos, el código de los tests resulta más claro, conciso y fácil de mantener.

```typescript
import { Course } from '../domain/Course';
import { CreateCourseForm } from './CreateCourseForm';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
// https://www.npmjs.com/package/@faker-js/faker
import { faker } from '@faker-js/faker';
// https://www.npmjs.com/package/fishery
import { Factory } from 'fishery';

// Definimos una fábrica para crear objetos Course
const courseFactory = Factory.define<Course>(() => ({
  title: faker.lorem.words(3), // Título aleatorio
  duration: faker.datatype.number({ min: 30, max: 180 }), // Duración aleatoria entre 30 y 180
}));

const CourseMother = {
  createCourse: (overrides?: Partial<Course>) => courseFactory.build(overrides), // Crea un curso con datos aleatorios
  createCourses: (count: number, overrides?: Partial<Course>) => courseFactory.buildList(count, overrides), // Crea una lista de cursos
};

// Ejemplo de uso en un test
test('envía datos válidos', async () => {
  const user = userEvent.setup();
  const onCreate = jest.fn().mockResolvedValue(undefined);
  const course = CourseMother.createCourse(); // Creamos un curso con datos aleatorios
  render(<CreateCourseForm onCreate={onCreate} />);
  await user.type(screen.getByLabelText(/title/i), course.title);
  await user.type(screen.getByLabelText(/duration/i), course.duration.toString());
  await user.click(screen.getByRole('button', { name: /create/i }));
  expect(onCreate).toHaveBeenCalledWith({ title: course.title, duration: course.duration });
});
```
