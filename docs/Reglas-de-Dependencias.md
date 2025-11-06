# Reglas de Dependencias

Estas reglas concretan la dirección de dependencias y qué importaciones están permitidas entre capas.

## Dirección de dependencias

```
infraestructura  →  aplicacion  →  dominio
```

- Dominio no importa nada de `aplicacion` ni `infraestructura`.
- Aplicación importa `dominio`, pero no importa `infraestructura`.
- Infraestructura puede importar `aplicacion` (para implementar puertos) y `dominio` (para construir entidades/Value Objects).

## Matriz de importaciones

- Permitido:
  - `aplicacion/*` → `dominio/*`
  - `infraestructura/*` → `aplicacion/*`
  - `infraestructura/*` → `dominio/*`
- Prohibido:
  - `dominio/*` → `aplicacion/*` o `infraestructura/*`
  - `aplicacion/*` → `infraestructura/*`

## Estructura sugerida por módulo

```
/modulos/<nombre>
├── dominio/
├── aplicacion/
└── infraestructura/
```

## Enforzar con ESLint (opcional)

Ejemplo con `eslint-plugin-import` (`import/no-restricted-paths`) y alias TS `@modules/*`:

```js
// .eslintrc.cjs (fragmento ilustrativo)
module.exports = {
  rules: {
    'import/no-restricted-paths': [
      'error',
      {
        zones: [
          // dominio no puede importar app/infra
          { target: './src/modules/**/domain', from: './src/modules/**/application' },
          { target: './src/modules/**/domain', from: './src/modules/**/infrastructure' },

          // aplicacion no puede importar infra
          { target: './src/modules/**/application', from: './src/modules/**/infrastructure' },
        ],
      },
    ],
  },
};
```

Alternativa con `eslint-plugin-boundaries` (zonas por carpeta):

```js
// .eslintrc.cjs (fragmento ilustrativo)
module.exports = {
  plugins: ['boundaries'],
  settings: {
    'boundaries/elements': [
      { type: 'domain', pattern: 'src/modules/*/domain/**' },
      { type: 'application', pattern: 'src/modules/*/application/**' },
      { type: 'infrastructure', pattern: 'src/modules/*/infrastructure/**' },
    ],
  },
  rules: {
    'boundaries/element-types': [
      'error',
      {
        default: 'allow',
        rules: [
          { from: 'domain', disallow: ['application', 'infrastructure'] },
          { from: 'application', disallow: ['infrastructure'] },
        ],
      },
    ],
  },
};
```

Nota: Los fragmentos son ejemplos; ajustá paths y alias según tu proyecto.
