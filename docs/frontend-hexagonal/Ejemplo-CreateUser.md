# Ejemplo completo: CreateUser

Ejemplo realista con módulos, puertos, adaptador (repositorio), DTOs y mappers.

## Árbol de carpetas (módulo `usuarios`)

```
/modulos/usuarios/
├── dominio/
│   ├── Usuario.ts
│   ├── Email.ts
│   └── repositorios/
│       └── RepositorioDeUsuarios.ts
├── aplicacion/
│   ├── casos-uso/
│   │   └── CrearUsuario.ts
│   ├── comandos/
│   │   └── CrearUsuarioInput.ts
└── infraestructura/
    ├── api/
    │   ├── crearUsuario.ts
    │   └── dto/
    │       ├── CrearUsuarioDto.ts
    │       ├── UsuarioDto.ts
    │       └── mapper.ts
    └── repositorios/
        └── RepositorioDeUsuariosFetch.ts
```

## Código

### dominio/Usuario.ts

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

### dominio/Email.ts

```ts
export class Email {
  constructor(private readonly valor: string) {
    if (!valor.includes('@')) throw new Error('Email inválido');
  }
  get value(): string {
    return this.valor;
  }
}
```

### dominio/repositorios/RepositorioDeUsuarios.ts (Puerto)

```ts
import { Usuario } from '../Usuario';

export interface RepositorioDeUsuarios {
  crear(nombre: string, email: string): Promise<Usuario>;
}
```

### aplicacion/comandos/CrearUsuarioInput.ts

```ts
export interface CrearUsuarioInput {
  nombre: string;
  email: string;
}
```

### aplicacion/casos-uso/CrearUsuario.ts

```ts
import { RepositorioDeUsuarios } from '../../dominio/repositorios/RepositorioDeUsuarios';
import { Usuario } from '../../dominio/Usuario';
import { Email } from '../../dominio/Email';
import { CrearUsuarioInput } from '../comandos/CrearUsuarioInput';

export class CrearUsuario {
  constructor(private readonly repo: RepositorioDeUsuarios) {}

  async ejecutar(input: CrearUsuarioInput): Promise<Usuario> {
    const emailVO = new Email(input.email);
    return this.repo.crear(input.nombre, emailVO.value);
  }
}
```

### infraestructura/api/dto/CrearUsuarioDto.ts

```ts
export interface CrearUsuarioDto {
  nombre: string;
  email: string;
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

### infraestructura/api/dto/mapper.ts

```ts
import { UsuarioDto } from './UsuarioDto';
import { Usuario } from '../../../dominio/Usuario';
import { Email } from '../../../dominio/Email';

export function dtoToUsuario(dto: UsuarioDto): Usuario {
  return new Usuario(dto.id, dto.nombre, new Email(dto.email));
}
```

### infraestructura/api/crearUsuario.ts

```ts
import { CrearUsuarioDto } from './dto/CrearUsuarioDto';
import { UsuarioDto } from './dto/UsuarioDto';

export async function crearUsuario(dto: CrearUsuarioDto): Promise<UsuarioDto> {
  const res = await fetch('/api/usuarios', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(dto),
  });
  if (!res.ok) throw new Error('No se pudo crear el usuario');
  return res.json();
}
```

### infraestructura/repositorios/RepositorioDeUsuariosFetch.ts (Adaptador)

```ts
import { RepositorioDeUsuarios } from '../../dominio/repositorios/RepositorioDeUsuarios';
import { Usuario } from '../../dominio/Usuario';
import { crearUsuario } from '../api/crearUsuario';
import { dtoToUsuario } from '../api/dto/mapper';

export class RepositorioDeUsuariosFetch implements RepositorioDeUsuarios {
  async crear(nombre: string, email: string): Promise<Usuario> {
    const usuarioDto = await crearUsuario({ nombre, email });
    return dtoToUsuario(usuarioDto);
  }
}
```

### Uso desde la UI (ejemplo)

```ts
import { CrearUsuario } from '@/modulos/usuarios/aplicacion/casos-uso/CrearUsuario';
import { RepositorioDeUsuariosFetch } from '@/modulos/usuarios/infraestructura/repositorios/RepositorioDeUsuariosFetch';

export async function crearUsuarioDesdeUI(nombre: string, email: string) {
  const repo = new RepositorioDeUsuariosFetch();
  const caso = new CrearUsuario(repo);
  const usuario = await caso.ejecutar({ nombre, email });
  return usuario;
}
```

## Variante minimalista (sin `api/` separado)

Mover la llamada `fetch` directamente al repo es válido si no necesitás reutilizarla ni testearla aparte.

```ts
export class RepositorioDeUsuariosFetch implements RepositorioDeUsuarios {
  async crear(nombre: string, email: string): Promise<Usuario> {
    const res = await fetch('/api/usuarios', { method: 'POST', body: JSON.stringify({ nombre, email }) });
    if (!res.ok) throw new Error('No se pudo crear el usuario');
    const dto = await res.json();
    return dtoToUsuario(dto);
  }
}
```

## Puntos clave

- DTOs (externos) en `infraestructura/api/dto`. Inputs internos de casos de uso en `aplicacion/comandos`.
- Puerto en `dominio/repositorios`. Adaptador en `infraestructura/repositorios`.
- Infraestructura puede importar dominio y aplicación. Aplicación y dominio no importan infraestructura.
