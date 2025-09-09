Tests unitarios en frontend: mantenemos la unidad de negocio (caso de uso) pequeña y testeable, pero para cubrir el flujo real del usuario solemos empezar por el componente. Así validamos la ruta UI → caso de uso con bajo acoplamiento.

View → Component → Use Case → Repository ← Implementación

Qué cubren los tests de un componente:

- Render básico y estados de UI.
- Validaciones de UI (no reglas de dominio).
- Interacción: que invoca el caso de uso con los datos correctos.
- Estados de envío/errores propios de la presentación.

Qué testear (resumen)

- Component: que habilita/deshabilita el botón, muestra errores de UI y llama `onCreate` con `{ title, duration }` válidos.
- View: opcionalmente, que cablea bien dependencias (doble del repo y esperar a `save`).

Test básico del componente (Jest + Testing Library)

```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { CreateCourseForm } from './CreateCourseForm';

test('envía datos válidos', async () => {
  const user = userEvent.setup();
  const onCreate = jest.fn().mockResolvedValue(undefined);

  render(<CreateCourseForm onCreate={onCreate} />);

  await user.type(screen.getByLabelText(/title/i), 'React 101');
  await user.type(screen.getByLabelText(/duration/i), '60');
  await user.click(screen.getByRole('button', { name: /create/i }));

  expect(onCreate).toHaveBeenCalledWith({ title: 'React 101', duration: 60 });
});
```

Notas

- Mantener validaciones de dominio en Domain/Application; aquí se muestran validaciones de UI para simplificar el ejemplo.
- El caso de uso real debería recibir interfaces (repositorios) por parámetro y no conocer implementaciones concretas.
- Si prefieres, puedes envolver el caso de uso en un hook (`useCreateCourse`) y el componente lo consumiría, pero pasar `onCreate` por props suele ser más explícito y fácil de testear.
