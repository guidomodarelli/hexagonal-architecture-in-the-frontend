# To-Do

- [ ] También me gustaría definir cómo organizar la parte de infraestructura para los distintos tipos de implementaciones concretas. Por ejemplo, en el módulo HTTP podés usar Fetch, Axios u otra librería. Para mí, lo ideal sería crear una carpeta dentro de infraestructura y, a primer nivel, tener carpetas como “Axios” o “Fetch”. De este modo, la carpeta “Axios” contendría toda la implementación concreta relacionada con Axios, y la carpeta “Fetch” tendría la implementación específica de Fetch.
  - Los DTOs (Data Transfer Objects) no deberían estar dentro de estas carpetas particulares, sino ubicarse en **/api/dto**, dentro de la carpeta API.
- [ ] 
