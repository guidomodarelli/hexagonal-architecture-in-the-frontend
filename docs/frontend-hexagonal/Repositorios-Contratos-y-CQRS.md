# Repositorios: contratos, retornos y CQRS (Frontend Hexagonal)

Objetivo: decidir qué retorna un repositorio, cuándo usar entidades de dominio y cuándo usar “read models” de aplicación, sin introducir DTOs de infraestructura fuera de su capa.

## Principios

- El puerto de repositorio (interface) modela necesidades del caso de uso, no detalles de red/DB.
- La implementación concreta (adaptador) mapea DTO externo ↔ modelo interno antes de cruzar el puerto.
- Para comandos (escritura) priorizar entidades/IDs del dominio; para consultas (lectura) considerar read models de aplicación (CQRS) cuando tenga sentido.

```
Infra (DTO externo) → [mapper] → Dominio (Entidad) o App (ReadModel)
```

## Contratos recomendados

### Comandos (escritura)

- `create`: `Promise<Usuario>` o `Promise<UsuarioId>`
- `update`: `Promise<Usuario>` o `Promise<void>`
- `delete`: `Promise<void>`

Razón: los comandos afectan invariantes del dominio; devolver la entidad/ID asegura consistencia y claridad del flujo. Nunca devuelvas un DTO externo.

### Consultas (lectura)

Opciones válidas:
- `findById`: `Promise<Usuario | null>` (entidad de dominio)
- `search/getAll`: `Promise<Usuario[]>` (dominio) o `Promise<UsuarioListItem[]>` (read model de aplicación)

Usá un “read model” de aplicación cuando:
- La lectura no necesita reglas/invariantes del dominio.
- Querés optimizar forma/cantidad de datos para UI (paginado, proyecciones).
- Practicás CQRS separando explícitamente “write models” (dominio) y “read models” (aplicación).

Dónde definirlos: `modulos/<x>/aplicacion/resultados` o `.../consultas/models`.

## Dónde NO usar DTOs

- Los DTOs de infraestructura (HTTP/SDK/storage) se definen y usan solo en `infraestructura/api/dto` y vecinos; no aparecen en `aplicacion` ni `dominio`.

## Ejemplos de puertos

### Solo dominio (simple)

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

### CQRS para lectura (read model de aplicación)

```ts
// aplicacion/resultados/UsuarioListItem.ts
export interface UsuarioListItem {
  id: string;
  nombre: string;
}

// aplicacion/puertos/UsuarioQuery.ts
import { UsuarioListItem } from '../resultados/UsuarioListItem';

export interface UsuarioQuery {
  search(params: { q?: string; page?: number; limit?: number }): Promise<{ items: UsuarioListItem[]; total: number }>;
}
```

## Adaptadores (infraestructura)

El adaptador importa dominio y/o modelos de aplicación, nunca al revés. Mapea DTO externo → entidad o → read model.

```ts
// infraestructura/repositorios/UsuarioQueryFetch.ts
import { UsuarioQuery } from '../../aplicacion/puertos/UsuarioQuery';
import { UsuarioListItem } from '../../aplicacion/resultados/UsuarioListItem';
import { getUsuarios } from '../api/getUsuarios';

export class UsuarioQueryFetch implements UsuarioQuery {
  async search(params: { q?: string; page?: number; limit?: number }) {
    const { items, total } = await getUsuarios({ query: params.q, page: params.page, limit: params.limit });
    const vm: UsuarioListItem[] = items.map(i => ({ id: i.id, nombre: i.nombre }));
    return { items: vm, total };
  }
}
```

## Decisión rápida

- ¿Necesitás invariantes/lógica de dominio en la respuesta? → retorna entidades de dominio.
- ¿Solo necesitás proyecciones para UI? → retorna read models de aplicación.
- En ambos casos: jamás retornes DTOs de infraestructura desde el puerto.

## Relación con los docs existentes

- DTOs de aplicación vs infraestructura: `DTOs-Aplicacion-vs-Infraestructura.md`
- Guía general de DTOs/puertos/adaptadores: `DTOs-Puertos-Adaptadores.md`
- Ejemplos: `Ejemplo-CreateUser.md` (comandos) y `Ejemplo-GetUsers.md` (lectura).

