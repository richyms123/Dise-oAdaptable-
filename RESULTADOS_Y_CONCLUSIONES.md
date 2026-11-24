# Práctica: Compila una App con un Diseño Adaptable (Reply App)

**Estudiante:** Gustavo Iván Vega  
**Fecha:** 24 de noviembre de 2026  
**Materia:** Desarrollo de Aplicaciones Móviles / Android con Jetpack Compose  

---

## 1. Introducción

En el desarrollo de aplicaciones modernas para Android, es fundamental garantizar que la interfaz de usuario se adapte sin problemas a una amplia variedad de tamaños de pantalla y factores de forma, incluidos teléfonos inteligentes en modo retrato y paisaje, tabletas, dispositivos plegables y pantallas de escritorio.

El objetivo de esta práctica es implementar un diseño adaptable (Adaptive Layout) en la aplicación **Reply** utilizando **Jetpack Compose** y la API `WindowSizeClass` de Material 3. La aplicación adapta dinámicamente tanto la estructura de navegación como el diseño del contenido en función del tamaño de ventana disponible.

---

## 2. Arquitectura del Diseño Adaptable

La adaptabilidad de la aplicación se gestiona evaluando la clase de tamaño de ancho (`WindowWidthSizeClass`) que proporciona la biblioteca Material 3 Window Size Class. Se definieron tres configuraciones principales:

### 2.1 Clasificación de Tamaños de Ventana (`WindowWidthSizeClass`)

1. **Compact (Ancho < 600dp):**
   - **Navegación:** `ReplyNavigationType.BOTTOM_NAVIGATION` (Barra de navegación inferior).
   - **Contenido:** `ReplyContentType.LIST_ONLY` (Muestra únicamente la lista de correos; al hacer clic en un correo, se navega a una pantalla de detalles independiente en pantalla completa).

2. **Medium (600dp ≤ Ancho < 840dp):**
   - **Navegación:** `ReplyNavigationType.NAVIGATION_RAIL` (Riel de navegación lateral).
   - **Contenido:** `ReplyContentType.LIST_ONLY` (Muestra la lista de correos; al seleccionar un correo, se abre la vista de detalles a pantalla completa).

3. **Expanded (Ancho ≥ 840dp):**
   - **Navegación:** `ReplyNavigationType.PERMANENT_NAVIGATION_DRAWER` (Cajón de navegación lateral permanente).
   - **Contenido:** `ReplyContentType.LIST_AND_DETAIL` (Diseño de dos paneles: la lista de correos se muestra a la izquierda y el detalle del correo seleccionado a la derecha de forma simultánea).

---

## 3. Implementación del Código Fuente

### 3.1 Captura del Tamaño de Ventana en `MainActivity`
```kotlin
val windowSize = calculateWindowSizeClass(this)
ReplyApp(
    windowSize = windowSize.widthSizeClass
)
```

### 3.2 Lógica de Selección en `ReplyApp`
```kotlin
when (windowSize) {
    WindowWidthSizeClass.Compact -> {
        navigationType = ReplyNavigationType.BOTTOM_NAVIGATION
        contentType = ReplyContentType.LIST_ONLY
    }
    WindowWidthSizeClass.Medium -> {
        navigationType = ReplyNavigationType.NAVIGATION_RAIL
        contentType = ReplyContentType.LIST_ONLY
    }
    WindowWidthSizeClass.Expanded -> {
        navigationType = ReplyNavigationType.PERMANENT_NAVIGATION_DRAWER
        contentType = ReplyContentType.LIST_AND_DETAIL
    }
    else -> {
        navigationType = ReplyNavigationType.BOTTOM_NAVIGATION
        contentType = ReplyContentType.LIST_ONLY
    }
}
```

### 3.3 Preservación del Estado en Cambios de Configuración
Se implementó la gestión de estado mediante `ReplyViewModel` y `ReplyUiState` usando `StateFlow`, asegurando que al rotar la pantalla o cambiar el tamaño de ventana entre `Compact`, `Medium` y `Expanded`:
- El correo seleccionado actualmente se preserve sin perder contexto.
- La navegación retroceda adecuadamente sin fallas.

---

## 4. Pruebas y Resultados

### 4.1 Pruebas Unitarias e UI Automatizadas
Se ejecutaron las pruebas automatizadas en `app/src/androidTest/java/com/example/reply/test/`:
- **`ReplyAppTest`**: Verificación de componentes de navegación correctos según la clase de tamaño (`navigation_bottom` para Compact, `navigation_rail` para Medium, `navigation_drawer` para Expanded).
- **`ReplyAppStateRestorationTest`**: Verificación de la restauración del estado (correo seleccionado retenido tras rotación/cambios de configuración) utilizando `StateRestorationTester`.

### 4.2 Resultados Obtenidos
- **Compilaciones sin errores:** El proyecto compila y construye correctamente (`assembleDebug` finalizado exitosamente).
- **Fluidez visual:** La transición entre la vista de un solo panel (Compact/Medium) y la vista de dos paneles (Expanded) funciona de forma fluida y responsiva.
- **Experiencia de usuario optimizada:** Maximiza el aprovechamiento del espacio en pantallas grandes (tabletas/escritorio) mientras mantiene una experiencia cómoda en teléfonos.

---

## 5. Conclusiones

1. **Ventajas de `WindowSizeClass`:** A diferencia de las orientaciones fijas (Portrait/Landscape), el uso de `WindowWidthSizeClass` permite abstraer las decisiones de diseño según los puntos de interrupción (`breakpoints`) recomendados por Material Design 3, ofreciendo compatibilidad nativa con pantallas divididas, ventanas flotantes y dispositivos plegables.
2. **Modularidad en Jetpack Compose:** La separación entre los enum de tipo de navegación (`ReplyNavigationType`) y tipo de contenido (`ReplyContentType`) permite que la UI sea altamentemodular, limpia y fácil de mantener o extender en el futuro.
3. **Manejo Efectivo del Estado:** Garantizar la persistencia del estado (`ReplyUiState`) entre cambios de tamaño de ventana es esencial para una experiencia de usuario continua y sin interrupciones.

---
