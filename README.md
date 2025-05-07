
# Blog-eq05 - Despliegue directo de carpeta `public/`

Este repositorio contiene un blog estático generado manualmente. En lugar de usar Hugo para generar los archivos,
hemos dejado directamente los archivos finales en la carpeta `public/`, que es la que se desplegará a Azure Static Web Apps.

## Estructura del proyecto

```
📁 public/
   ├── index.html
   ├── css/
   ├── contacto/
   ├── pages/
   ├── tutos/
   └── ...otros archivos y carpetas estáticas

📁 .github/
└── workflows/
    └── azure-static-web-apps.yml
```

## ¿Qué se ha hecho?
- Se ha editado manualmente el `index.html` y otras páginas dentro de `public/`.
- No se usa ningún generador como Hugo, ni ningún tema como Hyde.
- El flujo de despliegue automático sube los contenidos de `public/` directamente.

## Despliegue automático

Se usa Azure Static Web Apps con GitHub Actions.

### Archivo `.github/workflows/azure-static-web-apps.yml`:

Este archivo ya está configurado para subir únicamente la carpeta `public/`.

---
