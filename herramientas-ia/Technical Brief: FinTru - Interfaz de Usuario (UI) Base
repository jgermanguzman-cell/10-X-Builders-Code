# Technical Brief: FinTru - Interfaz de Usuario (UI) Base

## 1. Título de la tarea

Desarrollo de Interfaz de Usuario (UI) Base y Sistema de Diseño con Tailwind CSS.

---

## 2. Contexto

Este es el primer bloque iterativo del desarrollo de **FinTru**.

El objetivo es construir exclusivamente la capa visual (el "cascarón") de la aplicación, asegurando que:

- Se vea alineada con el sistema de diseño (Figma Tokens).
- Cumpla con los colores y espacios definidos en el sistema de diseño.
- Funcione perfectamente en teléfonos móviles (mobile first).

En esta etapa no conectaremos bases de datos reales ni lógicas financieras complejas. Usaremos datos simulados (*mock data*) para ver cómo luce la interfaz.

---

## 3. Requerimientos técnicos

### 3.1 Lenguaje y stack tecnológico

- **Frontend:** React (para crear los componentes visuales) y HTML5.
- **Estilos:** Tailwind CSS.
  - Traduciremos las variables del archivo JSON (FinTru DS) a la configuración de Tailwind (colores `Orange 500`, neutros, tipografía Inter, espacios, etc.).
- **Iconos:** `lucide-react`.
- **Sin backend por ahora:** no usaremos Supabase ni Node.js en esta fase.

### 3.2 Arquitectura

Basada en componentes (UI Components):

- `Boton`
- `TarjetaDeGasto`
- `BarraDeProgreso503020`
- `ModalDeMovimiento`

Manejo de estado local (lógica básica para interacción de pantalla):

- Clic en "+" abre un modal.
- Clic en una pestaña cambia la vista.

### 3.3 Modelo de datos (mock data)

Usaremos un archivo local con información estática para simular que la app tiene datos.

**Input — datos falsos a inyectar:**

- `mockProfile`: nombre del usuario, GSD simulado (50,000 COP), MDL simulado (5.2 meses).
- `mockAccounts`:
  - Una cuenta **"Cash"** con 100,000 COP.
  - Una cuenta bancaria con 500,000 COP.
- `mockTransactions`: lista de 3–4 gastos de ejemplo para llenar el historial visual.

**Output — visual:**

- Interfaz renderizada usando estos datos falsos.

### 3.4 Comportamiento esperado

**Layout principal:**

- En móvil: navegación en la parte inferior (estilo app móvil).
- En escritorio/web: navegación lateral.

**Dashboard visual:**

- Tarjeta principal del **GSD (Gasto Seguro Diario)**.
- Barra de colores del **50/30/20**.
- Lista inferior con atajos (*shortcuts*) visuales.

**Interacciones de UI:**

- Al hacer clic en el botón flotante principal (FAB "+"), debe aparecer el **Modal Universal de Movimientos** con sus campos de formulario.
- Al dar "Guardar", por ahora solo ejecuta un `console.log` sin guardar en base de datos.

---

## 4. Constraints (Restricciones)

| # | Restricción | Detalle |
|---|-------------|---------|
| 1 | **Mobile first estricto** | Diseñar primero para 390 px de ancho. Botones fáciles de tocar según los tokens de spacing. |
| 2 | **Fidelidad al sistema de diseño** | Usar estrictamente los colores del JSON (`Color/Orange/500`, `Color/Neutral/Bg Base`, etc.). |
| 3 | **Cero lógica de negocio** | No calcular el GSD matemáticamente; mostrar solo el número estático del mock data. |
| 4 | **Descartado temporalmente** | Sin registro por correo, Gemini API, notificaciones push ni base de datos. Solo pantallas estáticas interactivas. |

---

## 5. Definition of Done (DoD)

- [ ] La configuración de Tailwind incluye los colores, tipografías (Inter) y radios del archivo JSON proporcionado.
- [ ] El dashboard principal se renderiza correctamente con datos falsos y es 100 % responsivo.
- [ ] El **Modal de Movimientos** se abre, muestra un formulario básico (Tipo, Monto, Descripción) y se cierra al interactuar.
- [ ] La aplicación puede navegarse visualmente sin arrojar errores en consola.
