# Arquitectura Hexagonal

**Infraestructura**
Esta capa contiene los detalles de implementación concretos, como las llamadas a API, DBs, ficheros, I/O, o código que este acoplado a clases de un vendor externo.
Están las implementaciones de las interfaces que definiremos a nivel de dominio.

**Application**
Esta capa se encarga de la comunicación entre la capa de dominio y el mundo exterior, lo que denominamos "casos de uso". Actúa como una barrera transaccional.

**Domain**
Esta capa contiene la lógica de negocio central de la aplicación y define las reglas de negocio. No depende de ninguna otra capa y puede contener cosas como tipos, interfaces, funciones de validación, Value Objects, Entidades, Servicios de dominio.

Las dependencias deben apuntar hacia el interior de las capas, es decir, las capas de nivel inferior deben definir interfaces que las capas de nivel superior pueden utilizar para interactuar con ellas.

```
❌
⊢- application
  ⊢- AuthCommand
  ⊢- AuthCommandHandler
  ⊢- Authenticator
  ⊢- CourseCreator
  ⊢- CourseRenamer
⊢- domain
  ⊢- Auth
  ⊢- Product
  ⊢- Course
  ⊢- AuthRepository
  ⊢- ProductRepository
  ⊢- CourseRepository
⊢- infrastructure
  ⊢- MySQLCourseRepository
  ⊢- RedisAuthRepository
```

## Arquitectura Hexagonal + Vertical Slicing

**Vertical Slicing** consiste en dividir el sistema en funcionalidades verticales completas, que atraviesan todas las capas de la arquitectura hexagonal.
Cada vertical slice es una conjunto de características que proporcionan un valor tangible al usuario y que se implementa de forma independiente.

```
✅
⊢- auth
  ⊢- application
    ...
  ⊢- domain
    ...
  ⊢- infrastructure
    ...
⊢- courses
  ⊢- application
    ...
  ⊢- domain
    ...
  ⊢- infrastructure
    ...
...
```

### Regla de dependencia

Esta regla establece que el código de cada capa solo debe conocer las clases ubicadas en la capa inmediatamente inferior. El orden de las capas se entiende desde el exterior hacia el interior del círculo:

Infraestructura --> Aplicación --> Dominio

Esta norma nos permite modificar los elementos de las capas más externas sin afectar las internas. Por ello, tiene más sentido que los aspectos con mayor variabilidad —aquellos que no dependen directamente de nosotros— se ubiquen en la capa más externa, es decir, en Infraestructura.

## Puertos y Adaptadores

- Los puertos son las interfaces definidas en la capa de dominio para desacoplarnos de nuestra infraestructura. Por ejemplo, `UserRepository`.
- Los adaptadores son las implementaciones posibles de esos puertos. Estas implementaciones traducirán esos contratos definidos en la interfaz a la lógica necesaria de ejecutar en base a un determinado proveedor. Por ejemplo, `MySQLUserRepository`.
