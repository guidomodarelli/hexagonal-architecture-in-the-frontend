# 🧩 Active Record vs DataMapper

---

## ⚙️ Active Record

El enfoque **Active Record** se basa en que las propias entidades del dominio contengan toda la infraestructura necesaria para ser **persistidas, actualizadas y recuperadas** desde la base de datos.

Este patrón está **estrechamente ligado a la arquitectura en capas (Layered Architecture)**, donde la **capa de dominio** se comunica directamente con la base de datos.

### 📘 Características principales

* Las entidades **heredan de una clase base** que gestiona la persistencia.
* Los métodos de interacción con la base de datos **residen dentro de la propia entidad**.

```ts
// Crear una instancia de user y persistirla en la base de datos.
const user = User.create({ name: "David", occupation: "Code Artist" })

// Guardar una instancia creada a través del constructor `new`.
user.save()

// Buscar un usuario en la base de datos según un criterio.
const david = User.findByName("David")

// Actualizar una instancia existente en la base de datos.
david.update({ name: "Pepe" })
```

---

## 🧠 DataMapper

El patrón **DataMapper** propone que las entidades del dominio **no conozcan absolutamente nada** sobre cómo se almacenan o recuperan de la base de datos.

### 🧩 Estructura básica

1️⃣ **Entidad del dominio (ejemplo: `Video`)**

```ts
class Video {
  id: VideoId;
  title: VideoTitle;
}
```

2️⃣ **Archivo de mapeo (YAML o XML)**
Define la correspondencia entre las entidades y sus atributos con las **tablas y columnas** en la base de datos.

3️⃣ **Implementación de persistencia**
Se requiere una capa adicional para definir cómo se realizan las operaciones de lectura y escritura en la base de datos.

---

### 🏗️ Arquitectura

El enfoque **DataMapper** exige más trabajo de configuración e implementación, pero se apoya en los principios de la **arquitectura hexagonal**, donde la infraestructura debe ser **altamente tolerante al cambio**.

👉 El dominio **no debe depender** ni acoplarse a los detalles de persistencia.

---

## 🧱 Patrón Repository

Para lograr el **desacoplamiento** entre dominio e infraestructura, se define una **interfaz de repositorio** en la capa de dominio.
Esto representa un **contrato o puerto**, siguiendo los principios de **Ports and Adapters**.

```ts
interface VideoRepository {
  save(video: Video): void;
  search(id: VideoId): Video | null;
}
```

Luego, se implementa esa interfaz en la capa de **infraestructura**, creando un **adaptador**, por ejemplo:

```ts
class VideoRepositoryMySQL implements VideoRepository {
  save(video: Video): void {
    // Lógica para guardar en MySQL
  }

  search(id: VideoId): Video | null {
    // Lógica para buscar en MySQL
  }
}
```

### 💡 Aplicación del Principio de Inversión de Dependencias (DIP)

Gracias a este enfoque, los casos de uso pueden depender **de abstracciones** y no de implementaciones concretas:

```ts
class VideoFinder {
  constructor(private videoRepository: VideoRepository) {}

  execute(id: VideoId): Video | null {
    return this.videoRepository.search(id);
  }
}
```

➡️ De esta manera, el **caso de uso `VideoFinder`** no depende de los detalles de la infraestructura.
Podemos cambiar la implementación de `VideoRepository` (por ejemplo, de MySQL a MongoDB) **sin modificar la capa de aplicación ni la de dominio**.

---

## 🔍 Repasemos

**¿Por qué ubicar cada componente, puerto y adaptador en capas específicas?**

Porque, gracias a la **regla de dependencia**, garantizamos la **tolerancia al cambio** al aislar los conceptos externos y mantener un dominio puro y estable ante variaciones tecnológicas.
