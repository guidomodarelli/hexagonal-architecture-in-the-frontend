# Repositorios: contratos y retornos

**Objetivo**: decidir qué retorna un repositorio y cuándo usar entidades de dominio, sin introducir DTOs de infraestructura fuera de su capa.

## Principios

- El puerto de repositorio (interface) modela necesidades del caso de uso, no detalles de red/DB.
- La implementación concreta (adaptador) mapea DTO externo ↔ modelo interno antes de cruzar el puerto.
- Priorizar siempre entidades de dominio como retorno.

```
Infra (DTO externo) → [mapper] → Dominio (Entidad)
```

## Contratos recomendados

### Escritura

- `create`: `Promise<Usuario>` o `Promise<UsuarioId>`
- `update`: `Promise<Usuario>` o `Promise<void>`
- `delete`: `Promise<void>`

Razón: afectan invariantes del dominio; devolver la entidad/ID asegura consistencia y claridad del flujo. Nunca devuelvas un DTO externo.

### Lectura

- `findById`: `Promise<Usuario | null>` (entidad de dominio)
- `search/getAll`: `Promise<Usuario[]>` (entidad de dominio)

Razón: mantener consistencia usando siempre entidades de dominio simplifica el modelo mental y evita duplicación de estructuras.

## Dónde NO usar DTOs

- Los DTOs de infraestructura (HTTP/SDK/storage) se definen y usan solo en `infraestructura/api/dto` y vecinos; no aparecen en `aplicacion` ni `dominio`.

## Ejemplo de puerto

```ts
// aplicacion/puertos/RepositorioDeUsuarios.ts
import { Usuario } from '../../dominio/Usuario';

export interface RepositorioDeUsuarios {
  create(nombre: string, email: string): Promise<Usuario>;      // o Promise<UsuarioId>
  findById(id: string): Promise<Usuario | null>;
  getAll(): Promise<Usuario[]>;
  update(user: Usuario): Promise<Usuario>;                      // o Promise<void>
}
```

## Adaptadores (infraestructura)

El adaptador importa entidades de dominio, nunca al revés. Mapea DTO externo → entidad.

```ts
// infraestructura/repositorios/RepositorioDeUsuariosFetch.ts
import { RepositorioDeUsuarios } from '../../aplicacion/puertos/RepositorioDeUsuarios';
import { Usuario } from '../../dominio/Usuario';
import { getUsuarios } from '../api/getUsuarios';

export class RepositorioDeUsuariosFetch implements RepositorioDeUsuarios {
  async getAll(): Promise<Usuario[]> {
    const dtos = await getUsuarios();
    return dtos.map(dto => new Usuario(dto.id, dto.nombre, dto.email));
  }

  // ... resto de métodos
}
```

## Decisión rápida

- Retorna siempre entidades de dominio desde los puertos.
- Jamás retornes DTOs de infraestructura desde el puerto.

## Relación con los docs existentes

- DTOs de aplicación vs infraestructura: `docs/frontend-hexagonal/DTOs-Aplicacion-vs-Infraestructura.md`
- Guía general de DTOs/puertos/adaptadores: `docs/frontend-hexagonal/DTOs-Puertos-Adaptadores.md`
- Ejemplos: `docs/frontend-hexagonal/Ejemplo-CreateUser.md` y `docs/frontend-hexagonal/Ejemplo-GetUsers.md`.
