+++
date = '2025-04-02T12:03:35+02:00'
draft = true
title = 'Angular'
+++


# Introducción a Angular para Desarrolladores

## ¿Qué es Angular?
Angular es un framework de desarrollo web basado en TypeScript, creado y mantenido por Google. Se utiliza para construir aplicaciones web dinámicas y de una sola página (SPA).

## Instalación de Angular
Para instalar Angular, necesitas tener Node.js instalado en tu sistema. Puedes descargarlo desde [nodejs.org](https://nodejs.org/).

Una vez instalado Node.js, instala Angular CLI ejecutando el siguiente comando:

```sh
npm install -g @angular/cli
```

Para verificar que la instalación fue exitosa:

```sh
ng version
```

## Crear una aplicación Angular
Para iniciar un nuevo proyecto Angular, usa el siguiente comando:

```sh
ng new mi-aplicacion-angular
```

Navega al directorio del proyecto:

```sh
cd mi-aplicacion-angular
```

Para ejecutar la aplicación en un servidor de desarrollo:

```sh
ng serve
```

Luego, abre tu navegador y accede a `http://localhost:4200/` para ver la aplicación en funcionamiento.

## Estructura de un Proyecto Angular
Un proyecto Angular tiene la siguiente estructura básica:

```
mi-aplicacion-angular/
│-- src/
│   │-- app/       # Componentes y módulos de la aplicación
│   │-- assets/    # Recursos estáticos como imágenes
│   │-- environments/ # Configuración de entornos
│   │-- main.ts    # Punto de entrada de la aplicación
│   │-- styles.css # Estilos globales
│-- angular.json  # Configuración de Angular
│-- package.json  # Dependencias del proyecto
```

## Crear un Componente en Angular
Para crear un nuevo componente, usa el siguiente comando:

```sh
ng generate component mi-componente
```

Esto generará los siguientes archivos en `src/app/mi-componente/`:

- `mi-componente.component.ts` (Lógica del componente)
- `mi-componente.component.html` (Plantilla del componente)
- `mi-componente.component.css` (Estilos del componente)
- `mi-componente.component.spec.ts` (Pruebas unitarias)

Puedes usar el componente en `app.component.html` de la siguiente manera:

```html
<app-mi-componente></app-mi-componente>
```

## Enrutamiento en Angular
Angular permite la navegación entre páginas con el módulo `RouterModule`. Para agregar rutas, edita `app-routing.module.ts`:

```typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { MiComponenteComponent } from './mi-componente/mi-componente.component';

const routes: Routes = [
  { path: 'mi-componente', component: MiComponenteComponent }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule {}
```

Ahora puedes navegar a `http://localhost:4200/mi-componente` para ver el componente en acción.

## Conclusión
Angular es una poderosa herramienta para desarrollar aplicaciones web modernas. En este tutorial, aprendiste a instalar Angular, crear un proyecto, generar componentes y configurar rutas básicas.

Para más información, visita la documentación oficial en [angular.io](https://angular.io/).
