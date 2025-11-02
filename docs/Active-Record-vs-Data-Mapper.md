# Active Record vs DataMapper

## Active Record

Este enfoque se basa en que las propias entidades de nuestro dominio contengan la infraestructura necesaria a nivel de persistencia para poder ser persistidas, actualizadas y recuperadas.

Es, por tanto, un patrón ligado a la layered architecture, es decir, la capa de dominio es la que se comunica con la base de datos.

Esto se refleja en el código:
- Las entidades se extienden de una clase base que permite la gestión de persistencia.
- Los métodos de interacción con la base de datos están en la propia entidad.
  ```ts
  // Crear una instancia de user y a la vez la persistir en base de datos.
  const user = User.create({ name: "David", occupation: "Code Artist" })
  // Guardar una instancia ya creada a través del constructor `new`
  user.save
  // Busco un usuario en base a datos, en base al criterio de selección.
  const david = User.findByName("David")
  // Actualizar el estado de una instancia en base de datos.
  david.update({ name: "Pepe" })
  ```

## DataMapper

El patrón DataMapper lo que permite es justamente que nuestras entidades no conozcan nada relativo a cómo son persistidas en la base de datos.

- Por un lado tendremos nuestra entidad video
  ```ts
  class Video {
    id: VideoId;
    title: VideoTitle;
  }
  ```
- Por otro lado tendremos el fichero de definición YAML o XML que establece el mapeo entre cada una de las entidades y atributos de nuestro sistema, y las tablas y columnas en las que son persistidas.
- Algo necesario será definir la implementación necesaria para interacciones con la base de datos.

---

El trabajo que requiere a nivel de configuración e implementación en el enfoque Data Mapper es mayor. Data Mapper se basa en el pilar de la arquitectura hexagonal. Es decir, partimos de la premisa de que los detalles de infraestructura deben ser altamente tolerantes al cambio, con lo cual nuestro dominio no debe estar acoplado a esta infraestructura.

## Patrón Repository

Para lograr ese desacoplamiento, establecemos la interfaz `VideoRepository` a nivel de contrato de dominio, es decir, lo que denominaríamos como puerto en términos de Ports and Adapters.

```ts
interface VideoRepository {
  save(video: Video): void;
  search(id: VideoId): Video | null;
}
```

Y declaramos la implementación `VideoRepositoryMySQL` en la capa de infraestructura, es decir, lo que denominaríamos como adaptador en términos de Ports and Adapters.

Esto permite que a la hora de necesitar recuperar videos o persistirlos, podamos aplicar el Principio de Inversión de Dependencias, o DIP por sus siglas en inglés.

```ts
class VideoFinder {
  constructor(private videoRepository: VideoRepository) {}

  execute(id: VideoId): Video | null {
    return this.videoRepository.search(id);
  }
}
```

Como se puede ver en el caso de uso de `VideoFinder`, la ganancia de optar por un patrón DataMapper que separe la lógica de gestión de persistencia de nuestras entidades es que en ningún caso nos estaremos acoplando a detalles de infraestructura, pudiendo cambiar la implementación de la interfaz `VideoRepository` que recibamos desde el exterior sin que tengamos que modificar ni la capa de aplicación ni la de dominio.

---

## Repasemos

**¿Por qué ubicar cada componente, puertos y adaptadores en dichas capas?**

Porque gracias a la regla de dependencia garantizaremos la tolerancia al cambio aislando conceptos externos.