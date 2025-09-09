# Ejemplo de consulta: GetUsers (lectura)

Caso de uso de lectura con filtro en aplicación y DTOs externos en infraestructura.

## Árbol de carpetas (módulo `usuarios`)

```
/modulos/usuarios/
├── dominio/
│   ├── Usuario.ts
│   └── Email.ts
├── aplicacion/
│   ├── casos-uso/
│   │   └── ObtenerUsuarios.ts
│   ├── consultas/
│   │   └── FiltroUsuariosInput.ts
│   └── puertos/
│       └── RepositorioDeUsuarios.ts
└── infraestructura/
    ├── api/
    │   ├── getUsuarios.ts
    │   └── dto/
    │       ├── UsuarioDto.ts
    │       ├── GetUsersResponseDto.ts
    │       └── mapper.ts
    └── repositorios/
        └── RepositorioDeUsuariosFetch.ts
```

## Código

### dominio/Usuario.ts (reutilizable con CreateUser)

```ts
import { Email } from './Email';

export class Usuario {
  constructor(
    public readonly id: string,
    public readonly nombre: string,
    public readonly email: Email,
  ) {}
}
```

### aplicacion/puertos/RepositorioDeUsuarios.ts (Puerto extendido)

```ts
import { Usuario } from '../../dominio/Usuario';

export interface RepositorioDeUsuarios {
  // ya existente en CreateUser:
  crear(nombre: string, email: string): Promise<Usuario>;

  // búsqueda/paginación simple:
  buscar(params: { query?: string; page?: number; limit?: number }): Promise<{ items: Usuario[]; total: number }>; 
}
```

### aplicacion/consultas/FiltroUsuariosInput.ts

```ts
export interface FiltroUsuariosInput {
  query?: string;
  page?: number;  // 1-based
  limit?: number; // cantidad por página
}
```

### aplicacion/casos-uso/ObtenerUsuarios.ts

```ts
import { RepositorioDeUsuarios } from '../puertos/RepositorioDeUsuarios';
import { FiltroUsuariosInput } from '../consultas/FiltroUsuariosInput';

export class ObtenerUsuarios {
  constructor(private readonly repo: RepositorioDeUsuarios) {}

  async ejecutar(input: FiltroUsuariosInput): Promise<{ items: any[]; total: number }> {
    const page = input.page ?? 1;
    const limit = input.limit ?? 20;
    const query = input.query?.trim() || undefined;
    return this.repo.buscar({ query, page, limit });
  }
}
```

### infraestructura/api/dto/UsuarioDto.ts

```ts
export interface UsuarioDto {
  id: string;
  nombre: string;
  email: string;
}
```

### infraestructura/api/dto/GetUsersResponseDto.ts

```ts
import { UsuarioDto } from './UsuarioDto';

export interface GetUsersResponseDto {
  items: UsuarioDto[];
  total: number;
}
```

### infraestructura/api/dto/mapper.ts

```ts
import { Usuario } from '../../../dominio/Usuario';
import { Email } from '../../../dominio/Email';
import { UsuarioDto } from './UsuarioDto';

export function dtoToUsuario(dto: UsuarioDto): Usuario {
  return new Usuario(dto.id, dto.nombre, new Email(dto.email));
}
```

### infraestructura/api/getUsuarios.ts

```ts
import { GetUsersResponseDto } from './dto/GetUsersResponseDto';

export async function getUsuarios(params: { query?: string; page?: number; limit?: number }): Promise<GetUsersResponseDto> {
  const url = new URL('/api/usuarios', window.location.origin);
  if (params.query) url.searchParams.set('q', params.query);
  if (params.page) url.searchParams.set('page', String(params.page));
  if (params.limit) url.searchParams.set('limit', String(params.limit));

  const res = await fetch(url.toString(), { method: 'GET' });
  if (!res.ok) throw new Error('No se pudieron obtener los usuarios');
  return res.json();
}
```

### infraestructura/repositorios/RepositorioDeUsuariosFetch.ts (Adaptador)

```ts
import { RepositorioDeUsuarios } from '../../aplicacion/puertos/RepositorioDeUsuarios';
import { dtoToUsuario } from '../api/dto/mapper';
import { getUsuarios } from '../api/getUsuarios';

export class RepositorioDeUsuariosFetch implements RepositorioDeUsuarios {
  async crear(nombre: string, email: string) {
    throw new Error('No implementado aquí (ver ejemplo CreateUser)');
  }

  async buscar(params: { query?: string; page?: number; limit?: number }) {
    const resp = await getUsuarios(params);
    return { items: resp.items.map(dtoToUsuario), total: resp.total };
  }
}
```

### Uso desde la UI

```ts
import { ObtenerUsuarios } from '@/modulos/usuarios/aplicacion/casos-uso/ObtenerUsuarios';
import { RepositorioDeUsuariosFetch } from '@/modulos/usuarios/infraestructura/repositorios/RepositorioDeUsuariosFetch';

export async function buscarUsuariosDesdeUI(query: string) {
  const repo = new RepositorioDeUsuariosFetch();
  const caso = new ObtenerUsuarios(repo);
  return caso.ejecutar({ query, page: 1, limit: 20 });
}
```

## Puntos clave

- El input `FiltroUsuariosInput` es un DTO de aplicación: contrato interno Presentación ↔ Aplicación.
- `GetUsersResponseDto` y `UsuarioDto` son DTOs de infraestructura: contrato externo HTTP.
- El repositorio (adaptador) traduce DTO externo → dominio antes de devolver el resultado al caso de uso.

