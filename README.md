# Blog Hugo - Azure Static Web App

Este es un blog generado con [Hugo](https://gohugo.io/) y desplegado automáticamente en [Azure Static Web Apps](https://learn.microsoft.com/azure/static-web-apps/overview) mediante GitHub Actions.

## 🚀 Tecnologías usadas

- **Hugo v0.145.0 extended**
- **GitHub Actions**
- **Azure Static Web Apps (plan Free)**

## ⚙️ Flujo de despliegue automático

Cada vez que se hace un `push` a la rama `main`:

1. GitHub Actions instala Hugo.
2. Se compila el sitio en la carpeta `public/`.
3. El contenido se despliega automáticamente en Azure.

## 📁 Estructura del proyecto

├── archetypes/
├── content/
├── layouts/
├── static/
├── themes/
├── public/ # Aquí se generan los archivos HTML finales
├── config.toml # Configuración principal del blog
└── .github/workflows/
└── azure-static-web-apps-*.yml

## 🌐 Sitio en producción

El blog está accesible públicamente en:  
**[[https://blogeq05.azurestaticapps.net]](https://orange-sky-0bdee7910.6.azurestaticapps.net)**


---

## 📝 Créditos

Desarrollado por [Diego Castañeda, Abdelaziz Errahbi, Antonio Cano]  
Implementado con ❤️ en Azure y GitHub.
