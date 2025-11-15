# DTOs en Aplicación vs Infraestructura: cuándo, desde dónde y por qué

Esta guía complementa `DTOs-Puertos-Adaptadores.md` diferenciando explícitamente los DTOs de **aplicación** (inputs/outputs internos de casos de uso) y los DTOs de **infraestructura** (contratos con el exterior).

> Terminología: usamos "DTO de aplicación" para inputs/outputs de casos de uso; y "DTO de infraestructura" para requests/responses de HTTP/SDK/storage.

---

## DTOs de Aplicación (Inputs/Outputs de Casos de Uso)

### Cuándo se usa
- Como "comando" o "consulta" de un caso de uso: parámetros de entrada y, opcionalmente, su resultado.
- En el límite **Presentación ↔ Aplicación** (no en el límite con el mundo externo).

### Dónde se define
- `modules/<x>/application/commands` — **inputs** para casos de uso de escritura (CreateUserInput, UpdateUserInput, DeleteUserInput)
- `modules/<x>/application/queries` — **inputs** para casos de uso de lectura (GetUserByIdQuery, ListUsersQuery)
- `modules/<x>/application/results` — **outputs** opcionales cuando la respuesta del caso de uso necesita un formato específico para la UI (UserDetailResult, UserListResult) (**Importante:** Estos NO son DTOs de infraestructura — son contratos internos de aplicación que expresan las necesidades de lectura de la UI sin acoplarse a formatos externos.)

> **Nota sobre combinación:** Un caso de uso puede usar **command/query como input** y **result como output** simultáneamente. Por ejemplo: `createUser(input: CreateUserInput): Promise<UserCreatedResult>` o `listUsers(query: ListUsersQuery): Promise<UserListResult[]>`. No es obligatorio tener las tres carpetas; usá solo lo que tu caso de uso requiera.

### Desde dónde se importa ✅
- **Presentación** (Views/Pages/UI) para invocar casos de uso
- **Tests de aplicación**
- **Adaptadores entrantes** de infraestructura (router, efectos) — permitido porque `infra → app` está permitido

### Desde dónde NO se importa ❌
- **Nunca desde dominio** (el dominio no conoce la aplicación)
- **Nunca desde adaptadores salientes** (repos) para no acoplar el puerto a formatos externos

### Para qué se usa
- Expresar el contrato interno del **caso de uso** de forma estable y orientada al negocio/UX
- Validar y modelar requisitos de negocio al **nivel de aplicación** (campos obligatorios, consistencia básica antes de llegar a dominio)

### Por qué se usa
- Evita que los formatos externos contaminen la API del **caso de uso**
- Facilita testabilidad y evolución de la **UI** sin romper casos de uso
- Proporciona un punto de validación independiente del **dominio**

**Ejemplo mínimo:**

```ts
// modules/users/application/commands/CreateUserInput.ts
export interface CreateUserInput {
  nombre: string;
  email: string;
}
```

---

## DTOs de Infraestructura (Contratos Externos)

### Cuándo se usa
- En el límite **Infraestructura ↔ Mundo externo**: `HTTP`/`GraphQL`/`SDK`/`Storage`/`IndexedDB`
- Para requests y responses "crudos" que requieren mapeo hacia/desde el dominio

### Dónde se define
- `modules/<x>/infrastructure/api/dto`
- `modules/<x>/infrastructure/api/dto/mapper.ts` (lógica de conversión)

### Desde dónde se importa ✅
- **Solo desde infraestructura**: api/gateways/adapters/repositorios

### Desde dónde NO se importa ❌
- **Nunca desde aplicación**
- **Nunca desde dominio**
- **Nunca desde UI/Presentación**

### Para qué se usa
- Tipar el contrato de red/SDK y centralizar parseo/mapeo
- Saneamiento y validación de datos externos antes de cruzar hacia el núcleo

### Por qué se usa
- Aísla formatos externos cambiantes de tu modelo interno (**dominio**/**aplicación**)
- Permite mapear inconsistencias (snake_case, opcionales, nullables) a entidades/ValueObjects estables
- Protege el núcleo de cambios en APIs externas

**Ejemplo mínimo:**

```ts
// modules/users/infrastructure/api/dto/UserDto.ts
export interface UserDto {
  id: string;
  nombre: string;
  email: string;
}

// modules/users/infrastructure/api/dto/mapper.ts
export const toDomain = (dto: UserDto): User => {
  return new User(dto.id, dto.nombre, dto.email);
};
```

---

## Reglas Prácticas (Cheat Sheet)

| Capa                                 | Puede Importar                   | No Puede Importar                |
| ------------------------------------ | -------------------------------- | -------------------------------- |
| **Presentación**                     | Casos de uso, DTOs de aplicación | DTOs de infraestructura          |
| **Aplicación (Casos de Uso)**        | Puertos del dominio (interfaces), Entidades/VO del dominio | DTOs de infraestructura          |
| **Infraestructura (Adapters/Repos)** | Puertos, Dominio, DTOs de infra  | —                                |
| **Dominio**                          | Solo otros módulos de dominio    | Cualquier cosa fuera del dominio |

### Flujo de conversión

```
Mundo Externo → DTO Infra → Mapper → Entidad/ValueObject Dominio → Caso de Uso → DTO Aplicación → UI
```

### Pregunta de decisión rápida

- **"¿Esto modela el contrato con la UI/caso de uso?"**
  → Aplicación (`commands`/`queries`/`results`)

- **"¿Esto modela el contrato con API/SDK/Storage?"**
  → Infraestructura (`api/dto` + `mapper`)
