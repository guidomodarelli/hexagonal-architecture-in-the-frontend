# DTOs en Aplicación vs Infraestructura: cuándo, desde dónde y por qué

Esta guía complementa `DTOs-Puertos-Adaptadores.md` diferenciando explícitamente los DTOs de **aplicación** (inputs/outputs internos de casos de uso) y los DTOs de **infraestructura** (contratos con el exterior).

> Terminología: usamos “DTO de aplicación” para inputs/outputs de casos de uso; y “DTO de infraestructura” para requests/responses de HTTP/SDK/storage.

---

## DTOs de Aplicación (Inputs/Outputs de Casos de Uso)

- Cuándo se usa
  - Como “comando” o “consulta” de un caso de uso: parámetros de entrada; opcionalmente su resultado.
  - En el límite Presentación ↔ Aplicación (no en el límite con el mundo externo).
- Dónde se define
  - `modulos/<x>/aplicacion/comandos` (inputs) y, si lo necesitás, `modulos/<x>/aplicacion/resultados` (outputs).
  - También podés usar `aplicacion/consultas` para inputs de casos de uso de lectura.
- Desde dónde se importa
  - Presentación (Views/Pages/UI) para invocar casos de uso.
  - Tests de aplicación.
  - Adaptadores “entrantes” de infraestructura que llamen casos de uso (router, efectos) — permitido porque infra → app está permitido.
  - Nunca desde dominio. Nunca desde adaptadores “salientes” (repos) para no acoplar el puerto a la forma externa.
- Para qué se usa
  - Expresar el contrato interno del caso de uso de forma estable y orientada al negocio/UX.
  - Validar y modelar requisitos de negocio al nivel de aplicación (p. ej., campos obligatorios mínimamente consistentes antes de llegar a dominio).
- Por qué se usa
  - Evita que los formatos externos contaminen la API del caso de uso.
  - Facilita testabilidad y evolución de la UI sin romper casos de uso.

Ejemplo mínimo (entrada de comando):

```ts
// modulos/usuarios/aplicacion/comandos/CrearUsuarioInput.ts
export interface CrearUsuarioInput {
  nombre: string;
  email: string;
}
```

---

## DTOs de Infraestructura (Contratos Externos)

- Cuándo se usa
  - En el límite Infraestructura ↔ Mundo externo: HTTP/GraphQL/SDK/Storage/IndexedDB.
  - Requests y responses “crudos” y sus mappers hacia/desde el dominio.
- Dónde se define
  - `modulos/<x>/infraestructura/api/dto` (o `.../gateway/dto`), más `.../dto/mapper.ts`.
- Desde dónde se importa
  - Solo desde infraestructura (api/gateways/adapters/repositorios).
  - Nunca desde aplicación, dominio o UI.
- Para qué se usa
  - Tipar el contrato de red/SDK y centralizar parseo/mapeo y saneamiento.
- Por qué se usa
  - Aísla formatos externos cambiantes de tu modelo interno (dominio/aplicación).
  - Permite mapear inconsistencias (snake_case, opcionales) a entidades/VO estables.

Ejemplo mínimo (response externo):

```ts
// modulos/usuarios/infraestructura/api/dto/UsuarioDto.ts
export interface UsuarioDto {
  id: string;
  nombre: string;
  email: string;
}
```

---

## Reglas prácticas (cheat‑sheet)

- Presentación importa casos de uso y sus inputs/outputs de aplicación. No importa DTOs de infraestructura.
- Casos de uso dependen de puertos (interfaces en `aplicacion/puertos`) y del dominio. No conocen DTOs externos.
- Adaptadores/Repos (infraestructura) implementan puertos y convierten DTOs externos ↔ dominio antes de cruzar el puerto.
- Dominio no importa nada fuera del dominio.

Si dudás:

- “¿Esto modela el contrato con la UI/caso de uso?” → Aplicación (`comandos`/`consultas`/`resultados`).
- “¿Esto modela el contrato con API/SDK/Storage?” → Infraestructura (`api/dto` + `mapper`).

### Nota sobre Read Models (aplicación)

Para consultas (lectura) podés definir modelos de lectura optimizados para la UI (p. ej., `UsuarioListItem`) en `aplicacion/resultados` y hacer que el puerto de consulta los devuelva. No son DTOs de infraestructura: son contratos internos de aplicación. Ver `Repositorios-Contratos-y-CQRS.md`.
