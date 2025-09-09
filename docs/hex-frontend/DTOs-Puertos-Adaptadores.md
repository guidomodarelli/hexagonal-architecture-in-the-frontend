# DTOs, Puertos y Adaptadores (Frontend Hexagonal)

Guía práctica para ubicar DTOs, definir puertos y escribir adaptadores en un frontend con arquitectura hexagonal y Screaming Architecture.

## Regla de oro

- Los DTOs representan contratos externos (HTTP/SDK/storage) y viven en `infraestructura`.
- La aplicación define sus propios contratos internos (inputs/comandos) y NO depende de `infraestructura`.
- Los puertos (interfaces) viven en `aplicacion`; las implementaciones concretas (adaptadores) viven en `infraestructura`.
- El dominio es puro (entidades/VO/reglas) y no conoce ni aplicación ni infraestructura.

Dirección de dependencias permitida:

```
infraestructura  →  aplicacion  →  dominio
```

Nunca: `aplicacion → infraestructura` ni `dominio → (aplicacion|infraestructura)`.

Más detalle en: `docs/hex-frontend/Reglas-de-Dependencias.md`.

---

## Ubicación por capas (sugerida)

```
/modulos/<nombre>
├── dominio/
│   ├── entidades/
│   └── value-objects/
├── aplicacion/
│   ├── casos-uso/
│   ├── comandos/              # inputs internos de casos de uso
│   └── puertos/               # interfaces (Repository, Services)
└── infraestructura/
    ├── api/                   # funciones HTTP/SDK
    │   ├── dto/               # DTOs externos
    │   └── ...
    ├── adapters/ o repositorios/
    │   └── ...                # implementaciones de puertos
    └── gateways/ (opcional)   # cliente de red/SDK, bajo nivel
```

### Responsabilidades

- `aplicacion/puertos`: interfaces que expresan lo que la app necesita. Ej.: `RepositorioDeUsuarios`.
- `infraestructura/repositorios`: adaptadores que implementan puertos. Ej.: `RepositorioDeUsuariosFetch`.
- `infraestructura/api`: funciones que hablan con el exterior (HTTP/SDK). Ej.: `crearUsuario()`.
- `infraestructura/api/dto`: formas de datos externas. Ej.: `UsuarioDto`, `CrearUsuarioDto`.
- `infraestructura/api/dto/mapper.ts`: transforma DTO ↔ entidades del dominio.
- `aplicacion/comandos`: inputs internos para casos de uso. Ej.: `CrearUsuarioInput`.
- `dominio`: entidades/VO/reglas puras. Ej.: `Usuario`, `Email`.

---

## ¿Por qué los DTOs van en infraestructura?

Porque describen contratos que no controlás (API, SDK, storage). Su forma puede cambiar y no debe “contaminar” tu lenguaje de dominio ni la API interna de tus casos de uso. La infraestructura los convierte a modelos del dominio (o a inputs internos) mediante mappers.

Ejemplo: backend devuelve snake_case, campos extra u opcionales. El adaptador mapea eso al modelo estable del dominio.

---

## ¿Puertos en dominio o aplicación?

En aplicación. Los puertos (interfaces) son el contrato que los casos de uso dependen para cumplir su trabajo. La implementación concreta se resuelve fuera (infraestructura/ composición).

```
// aplicacion/puertos/RepositorioDeUsuarios.ts
export interface RepositorioDeUsuarios {
  crear(nombre: string, email: string): Promise<Usuario>;
}
```

---

## ¿El adaptador puede importar el dominio?

Sí. Las capas externas pueden depender de las internas. Es correcto que un repositorio en `infraestructura` construya `Usuario` a partir de un `UsuarioDto`. Lo prohibido es lo inverso: dominio y aplicación no pueden importar `infraestructura`.

---

## Separar `api/crearUsuario.ts` del repositorio: ¿cuándo?

- Separar (recomendado) cuando querés:
  - Testear la llamada HTTP de forma aislada.
  - Reutilizar la función desde otros adaptadores (SSR, jobs, etc.).
  - Cambiar cliente (fetch/axios/SDK) sin tocar el repositorio.
- Integrar en el repo (válido) si buscás máxima simplicidad y no hay reutilización.

Patrón recomendado (separado):

```
infraestructura/api/crearUsuario.ts   # hace la request
infraestructura/api/dto/mapper.ts     # mapea DTO → dominio
infraestructura/repositorios/...      # implementa el puerto usando lo anterior
```

---

## Convenciones de nombres

- Puerto: `RepositorioDeUsuarios` (aplicacion/puertos)
- Adaptador: `RepositorioDeUsuariosFetch` (infraestructura/repositorios)
- DTOs: `UsuarioDto`, `CrearUsuarioDto` (infraestructura/api/dto)
- Mapper: `dtoToUsuario`, `usuarioToDto` (infraestructura/api/dto/mapper.ts)
- Comando de caso de uso: `CrearUsuarioInput` (aplicacion/comandos)

---

## Testing

- Dominio: unit tests puros de entidades/VO.
- Aplicación: unit tests de casos de uso con dobles de puertos (mocks/stubs).
- Infraestructura: tests de integración de adaptadores (mapeos y llamadas reales o simuladas a `fetch`).

---

## Antipatrones a evitar

- Importar DTOs de `infraestructura` desde `aplicacion` o `dominio`.
- Reutilizar un `User` genérico en `shared` para todos los contextos (termina con muchos opcionales y invariantes débiles).
- Hacer que el dominio conozca `fetch`, `axios`, `localStorage` o el formato JSON externo.

---

## Recursos relacionados

- Reglas de importación y ejemplo ESLint: `docs/hex-frontend/Reglas-de-Dependencias.md`
- Ejemplo completo CreateUser: `docs/hex-frontend/Ejemplo-CreateUser.md`
 - DTOs de aplicación vs infraestructura (cuándo/desde dónde/por qué): `docs/hex-frontend/DTOs-Aplicacion-vs-Infraestructura.md`
 - Ejemplo de lectura (GetUsers): `docs/hex-frontend/Ejemplo-GetUsers.md`
