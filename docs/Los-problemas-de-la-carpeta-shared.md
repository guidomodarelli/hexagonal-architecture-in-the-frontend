# Los problemas de la carpeta `shared`

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

## ¿Y si tuviéramos `User` en `shared` y extendiéramos otras interfaces desde ahí?

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
