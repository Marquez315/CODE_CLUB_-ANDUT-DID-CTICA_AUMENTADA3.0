# Terminal Blog — Code Club Ñandutí / Didáctica Aumentada 3.0

Landing page estilo blog con estética retro arcade (pantalla CRT, scanlines,
tipografía pixel) que carga las entradas en vivo desde un Google Doc, sin
necesidad de base de datos ni backend propio.

- **Demo/archivo principal:** `index.html` (un solo archivo, HTML+CSS+JS)
- **Créditos:** [@codeclubnanduti](https://www.instagram.com/codeclubnanduti/#) ·
  [@didactica_aumentada3.0](https://www.instagram.com/didactica_aumentada3.0/#)

---

## 1. Publicar el sitio en GitHub Pages

1. Creá un repositorio nuevo en GitHub (público, para usar Pages gratis).
2. Subí el archivo `index.html` a la raíz del repositorio.
3. Andá a **Settings → Pages**.
4. En "Source" elegí la rama `main` y la carpeta `/ (root)` → **Save**.
5. Esperá uno o dos minutos. Tu sitio queda publicado en:
   `https://tu-usuario.github.io/tu-repositorio/`

GitHub Pages sirve todo por HTTPS automáticamente, así que la conexión al
Google Apps Script (paso siguiente) funciona sin configuración extra.

---

## 2. Conectar el blog a un Google Doc

El Doc funciona como "panel de administración": cada entrada nueva que
escribís ahí aparece sola en el sitio.

### 2.1 Escribir las entradas en el Doc

- Cada **título de entrada** va con el estilo de párrafo **Título 1**
  (menú Formato → Estilos de párrafo → Título 1).
- Opcional: una línea justo debajo con estilo **Título 2** para la fecha o
  bajada (ej. "Marzo 2026").
- El resto del texto, en estilo **Normal**, es el cuerpo de la entrada.
- Repetís esta estructura para cada entrada nueva (la más reciente arriba
  del documento).

### 2.2 Crear el Apps Script que expone el Doc como JSON

1. Con el Doc abierto: **Extensiones → Apps Script**.
2. Borrá el contenido de `Code.gs` y pegá esto:

```javascript
function doGet(e) {
  var doc = DocumentApp.getActiveDocument();
  var body = doc.getBody();
  var n = body.getNumChildren();
  var posts = [];
  var current = null;

  for (var i = 0; i < n; i++) {
    var el = body.getChild(i);
    if (el.getType() !== DocumentApp.ElementType.PARAGRAPH) continue;
    var para = el.asParagraph();
    var heading = para.getHeading();
    var text = para.getText();

    if (heading === DocumentApp.ParagraphHeading.HEADING1) {
      if (current) posts.push(current);
      current = { title: text, date: "", body: "" };
    } else if (current && heading === DocumentApp.ParagraphHeading.HEADING2 && !current.date) {
      current.date = text;
    } else if (current && text.trim() !== "") {
      current.body += text + "\n\n";
    }
  }
  if (current) posts.push(current);

  var out = ContentService.createTextOutput(JSON.stringify({ posts: posts }));
  out.setMimeType(ContentService.MimeType.JSON);
  return out;
}
```

3. Guardá el proyecto (ícono de disquete).
4. **Implementar → Nueva implementación → tipo: Aplicación web**.
5. En "Quién tiene acceso" elegí **Cualquier usuario** → **Implementar**.
6. Autorizá los permisos que pida Google (es tu propio script leyendo tu
   propio Doc).
7. Copiá la URL que termina en `/exec`.

### 2.3 Conectar la URL al sitio

**Opción rápida (para probar):** pegá la URL en el campo de texto del panel
"▸ Configurar fuente de datos" de la página y tocá "CARGAR ENTRADAS".

**Opción definitiva (recomendada):** abrí `index.html`, buscá cerca del
final del `<script>` la línea:

```javascript
const DEFAULT_API_URL = "";
```

y pegá tu URL adentro de las comillas:

```javascript
const DEFAULT_API_URL = "https://script.google.com/macros/s/XXXXXXXX/exec";
```

Subí el archivo modificado al repositorio — así el blog carga las entradas
solo, sin que cada visitante tenga que configurar nada.

---

## 3. Actualizar entradas

Simplemente editá el Google Doc: agregá una entrada nueva arriba (Título 1
+ cuerpo) y guardá. No hace falta volver a implementar el Apps Script ni
tocar el repositorio — el sitio la va a mostrar la próxima vez que alguien
lo cargue.

---

## 4. Estructura del archivo

Todo vive en `index.html`:

- **CSS** — estética de computadora retro (case, pantalla CRT, scanlines,
  tipografía pixel `Press Start 2P` + `VT323`).
- **HTML** — hero con efecto de tipeo, panel de configuración colapsable,
  grilla de entradas tipo listado de archivos, modal de lectura tipo
  ventana de terminal.
- **JS** — carga y parseo del JSON del Apps Script, renderizado de las
  tarjetas, apertura del modal, botón de datos de ejemplo.

No requiere build, ni `npm install`, ni backend propio: es un archivo
estático que podés hostear en GitHub Pages, Netlify, o cualquier hosting.
