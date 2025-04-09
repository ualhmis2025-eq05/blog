+++
date = '2025-04-02T12:03:38+02:00'
draft = true
title = 'Vaadin'
+++

---

# 🧩 Tutorial Básico de Vaadin

Vaadin es un **framework Java** para construir **interfaces de usuario web modernas** y reactivas sin necesidad de escribir JavaScript o HTML directamente. Te permite usar **Java puro** para diseñar la interfaz y gestionar la lógica del lado del servidor.

---

## 🔧 Requisitos Previos

- Java JDK 17 o superior
- Maven o Gradle
- IDE (IntelliJ, Eclipse o VS Code)
- Navegador web moderno

---

## 🚀 Crear un Proyecto Vaadin con Maven

### 1. Generar el proyecto

Puedes usar el [starter oficial de Vaadin](https://start.vaadin.com):

O ejecutar en terminal:

```bash
mvn -npr archetype:generate \
  -DarchetypeGroupId=com.vaadin \
  -DarchetypeArtifactId=vaadin-archetype-application \
  -DarchetypeVersion=24.2.0 \
  -DgroupId=com.ejemplo \
  -DartifactId=miapp-vaadin \
  -Dversion=1.0-SNAPSHOT \
  -Dpackaging=jar \
  -DinteractiveMode=false
```

> Cambia `groupId` y `artifactId` según tu proyecto.

---

### 2. Ejecutar la aplicación

```bash
cd miapp-vaadin
mvn clean install
mvn spring-boot:run
```

Accede en tu navegador a:  
📍 `http://localhost:8080`

---

## 🖥️ Crear una Vista Básica

Vaadin usa clases Java para definir las vistas. Por ejemplo:

```java
@Route("")
public class MainView extends VerticalLayout {

    public MainView() {
        add(new H1("Hola desde Vaadin"));
        Button boton = new Button("Haz clic");
        boton.addClickListener(e -> boton.setText("¡Hecho!"));
        add(boton);
    }
}
```

- `@Route("")` → Define la ruta raíz de la app.
- `VerticalLayout` → Componente de diseño vertical.
- `H1`, `Button` → Componentes de UI listos para usar.

---

## 🧱 Componentes Comunes

| Componente      | Descripción                       |
|-----------------|-----------------------------------|
| `TextField`     | Campo de texto                    |
| `Button`        | Botón                             |
| `Grid`          | Tabla de datos                    |
| `Dialog`        | Ventana emergente (modal)         |
| `ComboBox`      | Selector desplegable              |
| `Binder`        | Validación y binding de formularios |

Ejemplo de formulario con campo de texto:

```java
TextField nombre = new TextField("Nombre");
Button enviar = new Button("Enviar");

enviar.addClickListener(e -> {
    Notification.show("Hola " + nombre.getValue());
});

add(nombre, enviar);
```

---

## 🗂️ Navegación entre Vistas

Puedes crear varias vistas con diferentes rutas:

```java
@Route("otra")
public class OtraVista extends VerticalLayout {
    public OtraVista() {
        add(new H2("Otra página"));
    }
}
```

Y navegar desde otra vista:

```java
Button ir = new Button("Ir a otra vista", 
    e -> getUI().ifPresent(ui -> ui.navigate("otra")));
add(ir);
```

---

## 🎨 Estilos y Temas

Vaadin permite usar CSS personalizados o temas Lumo/Dark:

```java
Button btn = new Button("Claro");
btn.addThemeVariants(ButtonVariant.LUMO_PRIMARY);
```

También puedes añadir tu propio CSS en `frontend/styles/styles.css` y usar `@CssImport`.

---

## 🗄️ Integración con Back-end

Vaadin se integra fácilmente con bases de datos, Spring Boot, REST APIs, etc. Por ejemplo, puedes usar un `@Service` para acceder a datos y mostrarlos en un `Grid`.

```java
@Autowired
private PersonaService personaService;

Grid<Persona> grid = new Grid<>(Persona.class);
grid.setItems(personaService.listarPersonas());
add(grid);
```

---

## 📦 Desplegar

Al ser una app Java estándar, puedes empaquetar con:

```bash
mvn clean package
```

Y desplegar el `.jar` en cualquier servidor compatible con Java.

---

## 📚 Recursos Recomendados

- 🌐 [Vaadin Starter](https://start.vaadin.com)
- 📘 [Documentación oficial](https://vaadin.com/docs)
- 💡 [Componentes Vaadin](https://vaadin.com/components)
- 🧪 [Vaadin Labs](https://vaadin.com/labs)

---

