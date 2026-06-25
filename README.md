# 🧠 Claude → Obsidian Brain

Convertí tus conversaciones con Claude Desktop en un "segundo cerebro" en Obsidian: notas en Markdown organizadas por tema, con frontmatter y `[[wikilinks]]` automáticos entre conversaciones relacionadas.

No requiere código propio — usa el protocolo **MCP (Model Context Protocol)** de Anthropic junto con el plugin **"Local REST API with MCP"** de Obsidian (por Adam Coddington), que ya trae un servidor MCP integrado, para que Claude pueda leer y escribir directamente en tu vault.

## ¿Qué resuelve esto?

Por defecto, las conversaciones de Claude quedan sueltas en el historial del chat, sin conexión entre ellas y sin integrarse a tu sistema de notas. Este repo te da:

- Una **estructura de carpetas** para tu vault pensada para conversaciones de IA
- Un **comando** para pegar al final de cualquier chat suelto y que se guarde automáticamente
- Una **plantilla de instrucciones de Proyecto** para que el guardado funcione sin copiar/pegar nada dentro de Projects de Claude
- Una convención de **tags y notas "hub"** para que el grafo de Obsidian se mantenga conectado en vez de ser cientos de notas aisladas

📋 **Índice:** [Cómo funciona](#cómo-funciona-arquitectura) · [Instalación](#instalación) · [Uso](#uso) · [Troubleshooting](#troubleshooting) · [Limitaciones](#notas-y-limitaciones)

## Cómo funciona (arquitectura)

```
Claude Desktop ──MCP (Local REST API)──> Tu vault de Obsidian (.md files)
```

Claude no "sabe" automáticamente que existís en Obsidian — necesita el servidor MCP del plugin configurado una sola vez (vía `mcp-remote` como puente, porque Claude Desktop no soporta MCP remoto nativamente). Después de eso, cuando le pedís que guarde una conversación, Claude:

1. Resume el chat (resumen + puntos clave, no la transcripción completa)
2. Detecta 2-4 tags temáticos
3. Crea una nota nueva en `Claude/AAAA-MM/`
4. Busca si existe una nota "hub" para esos temas en `Temas/`
5. Si existe, le agrega un link a la nota nueva. Si no existe, la crea.

## Instalación

### Requisitos
- [Obsidian](https://obsidian.md) instalado, con un vault creado
- [Claude Desktop](https://claude.ai/download) instalado
- [Node.js](https://nodejs.org) instalado (npx viene incluido)

### Paso 1 — Plugin de Obsidian
1. En Obsidian: **Settings → Community plugins → Browse**
2. Buscar `Local REST API`, instalar **"Local REST API with MCP" (por Adam Coddington)** y activarlo
3. Ir a **Settings → Local REST API** y copiar el **API Key** que aparece ahí
4. El puerto por defecto es `27124` (HTTPS). Si preferís evitar temas de certificados, podés activar **"Enable HTTP server"** en esa misma pantalla, que corre en HTTP plano por el puerto `27123` (razonable para uso solo local en tu propia máquina)

### Paso 2 — Configurar Claude Desktop
Claude Desktop no soporta MCP remoto vía HTTP de forma nativa, así que se conecta a través de un puente llamado `mcp-remote` (se descarga solo vía `npx`, no hay que instalar nada por separado).

> ⚠️ **Windows: confirmá qué archivo de config usa tu instalación.** Si instalaste Claude Desktop desde la Microsoft Store (o tu sistema usa rutas `AppData\Local\Packages\...`), el archivo que edites a mano en `%APPDATA%\Claude\` puede no ser el que la app realmente lee. Ver la sección **[Troubleshooting](#troubleshooting)** antes de seguir si no estás seguro.

Abrí el archivo de configuración:
- **Mac**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

Y agregá (o fusioná si ya tenés otros `mcpServers`):

**Mac/Linux:**
```json
{
  "mcpServers": {
    "obsidian": {
      "command": "npx",
      "args": [
        "mcp-remote@latest",
        "https://127.0.0.1:27124/mcp/",
        "--header",
        "Authorization: Bearer <tu_api_key_del_paso_1>"
      ]
    }
  }
}
```

**Windows (recomendado, evita varios bugs conocidos — ver troubleshooting):**
```json
{
  "mcpServers": {
    "obsidian": {
      "command": "cmd",
      "args": [
        "/c",
        "npx",
        "mcp-remote@latest",
        "http://127.0.0.1:27123/mcp/",
        "--allow-http",
        "--transport",
        "http-only",
        "--header",
        "Authorization: Bearer <tu_api_key_del_paso_1>"
      ]
    }
  }
}
```

> En Windows usamos `http://` puerto `27123` (servidor no cifrado, activable en **Settings → Local REST API with MCP → "Enable non-encrypted (HTTP) server"**) en vez de `https://` puerto `27124`, para evitar que `mcp-remote` tenga que validar el certificado autofirmado del plugin. Ver por qué en la sección de troubleshooting.

### Paso 3 — Reiniciar
Cerrá completamente Claude Desktop y volvé a abrirlo. En un chat nuevo deberías ver `obsidian` disponible entre las herramientas/MCP conectadas.

> 📄 Plantillas listas para copiar: [`claude_desktop_config.example.json`](./claude_desktop_config.example.json) (Windows) y [`claude_desktop_config.mac-linux.example.json`](./claude_desktop_config.mac-linux.example.json) (Mac/Linux).

### Paso 4 — Estructura del vault
Creá estas carpetas en tu vault (o mirá el ejemplo en [`vault-structure-example/`](./vault-structure-example) de este repo):

```
TuVault/
├── Claude/
│   └── AAAA-MM/          ← notas de conversaciones, una carpeta por mes
├── Temas/                ← notas "hub" por tema (ej: Finanzas.md)
└── Proyectos/            ← opcional, un sub-hub por proyecto de Claude
```

## Uso

### Chats sueltos (fuera de un Project)
Copiá y pegá el contenido de [`templates/comando-chat-suelto.md`](./templates/comando-chat-suelto.md) al final de la conversación.

### Dentro de un Project de Claude
Pegá el contenido de [`templates/project-instructions.md`](./templates/project-instructions.md) en **Custom Instructions** del Project. A partir de ahí, basta con escribir `/guardar` al final de cualquier chat de ese Project.

## Troubleshooting

Esta sección documenta los problemas reales encontrados al armar este setup en Windows, en el orden en que conviene descartarlos.

### "obsidian" no aparece en Settings → Desarrollador → Servidores MCP locales

**Causa más común: estás editando el archivo de config equivocado.**

Algunas instalaciones de Claude Desktop en Windows (especialmente las de Microsoft Store, identificables porque la app vive bajo `Program Files\WindowsApps\Claude_...`) **no leen** `%APPDATA%\Claude\claude_desktop_config.json`. En su lugar usan:

```
%LOCALAPPDATA%\Packages\Claude_<id-aleatorio>\LocalCache\Roaming\Claude\claude_desktop_config.json
```

**Cómo confirmarlo sin adivinar:** en Claude Desktop, abrí **Settings → Desarrollador → Servidores MCP locales** y hacé clic en **"Editar configuración"**. Eso abre, garantizado, el archivo real que la app está usando — sea cual sea la ruta. Editá ese archivo, no el de `%APPDATA%` a ciegas.

Este archivo suele contener ya configuración interna de la app (preferencias, IDs de cuenta, etc.) — **no lo reemplaces entero sin revisar antes su contenido**. Agregá la clave `"mcpServers": { ... }` como hermana de las claves existentes, al mismo nivel, con una coma después de cerrar el bloque.

### Error: `"C:\Program" no se reconoce como un comando interno o externo`

Causa: en Windows, cuando `command` es directamente `npx` y el ejecutable real está en una ruta con espacios (`C:\Program Files\nodejs\npx.cmd`), el proceso que lanza Claude Desktop no encierra esa ruta en comillas, y Windows la corta en el primer espacio.

**Solución:** no uses `"command": "npx"` directo. Usá `"command": "cmd"` con `"args"` empezando en `"/c", "npx", ...`:

```json
"command": "cmd",
"args": ["/c", "npx", "mcp-remote@latest", "..."]
```

### Error: `Server disconnected` / `write EPIPE` inmediatamente después de conectar

Si ves `"Server started and connected successfully"` seguido casi al instante por `write EPIPE` y `Server transport closed unexpectedly`, normalmente es consecuencia del problema anterior (el comando falló silenciosamente puertas adentro). Solucionar el bug de `cmd /c` arriba resuelve esto también.

### Error: `HTTP 401: Invalid OAuth error response` / `Authorization required`

Este es el más confuso porque **tu API key puede estar perfecta** y aun así ver este error. Hay dos causas posibles, en este orden:

**1. `mcp-remote` intenta negociar OAuth antes de usar tu header.** El log va a mostrar `Discovering OAuth server configuration...` y `registerClient` antes del 401 — el plugin de Obsidian no implementa ese flujo OAuth (solo bearer token simple), así que falla en ese paso intermedio antes de llegar a usar tu key. Mitigalo agregando estos flags:

```json
"args": [
  "/c", "npx", "mcp-remote@latest",
  "http://127.0.0.1:27123/mcp/",
  "--allow-http",
  "--transport", "http-only",
  "--header", "Authorization: Bearer <key>"
]
```

Esto no siempre elimina el intento de discovery (es un comportamiento del propio `mcp-remote`), pero es la configuración más robusta disponible.

**2. La API key está incompleta o mal copiada — la causa real más probable.** Al copiar la key manualmente desde la pantalla de Obsidian (sin botón de "copiar"), es muy fácil perder caracteres del medio sin notarlo — visualmente una key de 64 caracteres hex es casi imposible de verificar a ojo. **Esta terminó siendo la causa real en nuestra prueba**, no el comportamiento de OAuth.

**Cómo diagnosticarlo con certeza, sin depender de Claude Desktop ni de los logs:**

```cmd
curl -v http://127.0.0.1:27123/vault/ -H "Authorization: Bearer <tu_key>"
```

- Si responde `200 OK` con la lista de archivos del vault → tu key y el servidor están perfectos, el problema es de `mcp-remote`/Claude Desktop.
- Si responde `401` con `"authenticated": false` → la key que estás usando no es válida. Volvé a Obsidian, mirá la key completa carácter por carácter, y volvé a copiarla — preferentemente seleccionando todo el campo de texto con el mouse en vez de tipear o recortar visualmente.

Repetir esta prueba con `curl` antes de tocar la config de Claude Desktop ahorra muchísimos ciclos de "editar → reiniciar → revisar log".

### Cómo ver los logs en general
**Settings → Desarrollador → Servidores MCP locales → (nombre del servidor) → "Ver registros"**. Las líneas más recientes están al final; buscá el bloque que empieza con `Initializing server...` con la marca de tiempo más nueva.

## Notas y limitaciones

- El plugin usa un **certificado autofirmado**. Si tu cliente MCP se queja del certificado HTTPS, activá "Enable HTTP server" en la configuración del plugin y usá `http://127.0.0.1:27123/mcp/` en vez de la URL HTTPS.
- Esto **no es automático en tiempo real** — necesitás pedirle explícitamente a Claude que guarde (o tener el comando en Project Instructions y escribir `/guardar`). Claude Desktop no tiene un modo "guardar todo sin que yo intervenga".
- Las **Project Instructions solo aplican dentro de ese Project**. Para chats sueltos hay que usar el comando manual.
- Si renombrás o movés archivos del vault directamente desde el sistema de archivos (no desde Obsidian), los `[[wikilinks]]` pueden romperse, porque Obsidian solo actualiza links automáticamente cuando el renombrado pasa por la propia app.
- Esto guarda **resúmenes**, no transcripciones completas, para mantener las notas livianas. Si preferís transcripción completa, editá la plantilla.

## Licencia

MIT — usalo, modificalo y compartilo como quieras.
