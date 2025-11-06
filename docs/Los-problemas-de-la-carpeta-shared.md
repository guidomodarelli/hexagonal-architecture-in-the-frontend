# 🛑 Problemas de la Carpeta `shared` y Tipado Genérico

Una entidad genérica (`User`) centralizada en una carpeta `shared` con muchos campos opcionales (`?` o `null`) puede generar problemas, llevando a un acoplamiento excesivo y dificultades de mantenimiento, escalabilidad y testabilidad.

```typescript
// ❌ Mal: Interfaz genérica con campos opcionales
export interface User {
  id: string;
  username: string;
  avatarUrl: string;
  name: string | null; // Opcional
  status?: 'active' | 'inactive' | null; // Opcional
}

// ✅ Bien: Interfaces específicas por contexto
export interface User {
  id: string;
  username: string;
  avatarUrl: string;
}

export interface Assignee {
  id: string;
  username: string;
  avatarUrl: string;
  name: string; // Campo requerido
  status: 'active' | 'inactive'; // Campo requerido
}
```

## ¿Y si tuviéramos `User` en `shared` y extendiéramos otras interfaces desde ahí?

Tener una única interfaz `User` en `shared` con muchos campos opcionales puede parecer conveniente, pero genera problemas significativos:

- **Campos nullable que rompen invariantes**: No se pueden garantizar restricciones del dominio
- **Validaciones dispersas**: La lógica de validación se esparce por toda la aplicación
- **Tablas con muchos NULL**: Dificulta aplicar restricciones y garantías a nivel de base de datos

### Ejemplo problemático: "Todo en una tabla"

```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  username VARCHAR(50) NOT NULL,
  avatar_url VARCHAR(255) NOT NULL,
  name VARCHAR(100),       -- ⚠️ Nullable
  status VARCHAR(10)       -- ⚠️ Nullable
);
```

**Desventajas**: No puedes garantizar que `name` o `status` existan cuando el dominio los requiere. La semántica se pierde y las restricciones se vuelven débiles.

## Alternativas recomendadas

### 1. Tipos compartidos mínimos + tipos de dominio específicos

Mantener en `shared` solo lo estrictamente común y definir interfaces explícitas en cada módulo/contexto (`Assignee`) que representen las necesidades reales, extendiendo solo una `BaseUser` mínima si es necesario:

```typescript
// shared/domain/BaseUser.ts
export interface BaseUser {
  id: string;
  username: string;
  avatarUrl: string;
}

// assignee/domain/Assignee.ts
export interface Assignee extends BaseUser {
  name: string;
  status: 'active' | 'inactive';
}
```

### 2. Modelado relacional normalizado (recomendado)

Separar la información obligatoria de la contextual en tablas distintas para reforzar invariantes (restricciones) y evitar `NULLs` innecesarios:

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

**Beneficios**: Restricciones claras, validaciones a nivel de BD, sin NULLs innecesarios.
**Coste**: Joins para leer datos compuestos (generalmente aceptable).

### 3. Capas de mapeo en infrastructure

La capa de infraestructura debe mapear rows/DTOs a las entidades de dominio que usa la aplicación:

```typescript
// infrastructure/AssigneeMapper.ts
export function mapUserRowToAssignee(row: UserRow): Assignee {
  return {
    id: String(row.user_id),
    username: row.username,
    avatarUrl: row.avatar_url,
    name: row.name,
    status: row.status as 'active' | 'inactive'
  };
}
```

## Reglas prácticas

1. **Mantén `shared` mínimo**: Solo lo realmente compartido entre múltiples contextos
2. **Tipos explícitos por contexto**: Evita condicionales y errores en tiempo de ejecución
3. **Modela para invariantes**: Usa tablas separadas o constraints, no tipos genéricos con NULLs
4. **Usa adaptadores/mappers**: Traduce entre el modelo de persistencia y las entidades de dominio; la aplicación debe trabajar con tipos claros y validados.

---

## ⚠️ Principales Problemas de usar mal `shared`

### 1. Mantenibilidad

Al centralizar todo en `shared`, se pierde la cohesión de los módulos específicos. Con el tiempo, `shared` crece descontroladamente:

- Dificulta la navegación del código
- Mezcla responsabilidades de diferentes contextos
- Requiere reorganizaciones costosas para recuperar la cohesión

### 2. Escalabilidad

El uso excesivo de entidades genéricas genera **acoplamiento entre contextos**:

**Problema en base de datos:**
- Tablas con múltiples campos opcionales/nulos
- Alteraciones costosas en tablas con millones de registros
- Cambios en un contexto bloquean o afectan a otros

**Ejemplo**: Modificar `User` para `Admin` impacta en `Auth`, `Assignee`, etc.

**Problema en APIs:**
- ¿Un endpoint `/user` o múltiples como `/assignee`, `/auth-user`?
- Endpoint único requiere lógica adicional para discriminar tipos
- Controladores más complejos y difíciles de mantener
- Dependencias innecesarias incluso con controladores separados

### 3. Testabilidad

Tipos genéricos llevan a **duplicación de pruebas**:

- Pruebas separadas para `User`, `Assignee`, `AuthUser`, etc.
- Riesgo de Object Mothers gigantes con métodos para cada variación
- Dificultad para mantener consistencia en los tests

**Solución**: Factories/Builders específicos por tipo:

```typescript
// test/factories/UserFactory.ts
export const createUser = (overrides: Partial<User> = {}): User => ({
  id: 'default-id',
  username: 'default-username',
  avatarUrl: 'default-avatar-url',
  ...overrides,
});

// test/factories/AssigneeFactory.ts
export const createAssignee = (overrides: Partial<Assignee> = {}): Assignee => ({
  id: 'default-id',
  username: 'default-username',
  avatarUrl: 'default-avatar-url',
  name: 'default-name',
  status: 'active',
  ...overrides,
});
```

**Ventajas**: Tests enfocados (en los casos específicos de cada tipo de usuario), sin duplicación, código más limpio.

## Cómo usar `shared` correctamente

A pesar de los problemas, contar con un módulo `shared` es útil para evitar duplicación y simplificar ciertas partes del sistema. Para mantenerlo bajo control y evitar que se convierta en un repositorio desorganizado, seguiremos estas reglas:

### Reglas de oro

1. **Capas compartidas limitadas**
   - Limitar capas compartidas solo a **Domain** e **Infrastructure**, excluyendo la de **Application** (casos de uso).
   - Esto asegura que la lógica específica de cada contexto permanezca encapsulada y no se propague innecesariamente

2. **Regla de tres**
   - Solo mover código a `shared` cuando se haya duplicado al menos dos veces (evaluar al identificar una tercera duplicación).
   - Este enfoque evita la sobreingeniería prematura y asegura que solo se comparte lo que realmente lo amerita

Siguiendo estas reglas, `shared` se mantiene limpio, enfocado y útil, sin convertirse en un punto de acoplamiento excesivo o en una fuente de complejidad innecesaria.

