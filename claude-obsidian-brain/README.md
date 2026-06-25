# 🧠 Claude → Obsidian Brain

Convertí tus conversaciones con Claude Desktop en un "segundo cerebro" en Obsidian: notas en Markdown organizadas por tema, con frontmatter y `[[wikilinks]]` automáticos entre conversaciones relacionadas.

No requiere código propio — usa el protocolo **MCP (Model Context Protocol)** de Anthropic junto con el plugin **"Local REST API with MCP"** de Obsidian (por Adam Coddington), que ya trae un servidor MCP integrado, para que Claude pueda leer y escribir directamente en tu vault.

## ¿Qué resuelve esto?

Por defecto, las conversaciones de Claude quedan sueltas en el historial del chat, sin conexión entre ellas y sin integrarse a tu sistema de notas. Este repo te da:

- Una **estructura de carpetas** para tu vault pensada para conversaciones de IA
- Un **comando** para pegar al final de cualquier chat suelto y que se guarde automáticamente
- Una **plantilla de instrucciones de Proyecto** para que el guardado funcione sin copiar/pegar nada dentro de Projects de Claude
- Una convención de **tags y notas "hub"** para que el grafo de Obsidian se mantenga conectado en vez de ser cientos de notas aisladas

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

Abrí el archivo de configuración:
- **Mac**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

Y agregá (o fusioná si ya tenés otros `mcpServers`):

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

### Paso 3 — Reiniciar
Cerrá completamente Claude Desktop y volvé a abrirlo. En un chat nuevo deberías ver `obsidian` disponible entre las herramientas/MCP conectadas.

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

## Notas y limitaciones

- El plugin usa un **certificado autofirmado**. Si tu cliente MCP se queja del certificado HTTPS, activá "Enable HTTP server" en la configuración del plugin y usá `http://127.0.0.1:27123/mcp/` en vez de la URL HTTPS.
- Esto **no es automático en tiempo real** — necesitás pedirle explícitamente a Claude que guarde (o tener el comando en Project Instructions y escribir `/guardar`). Claude Desktop no tiene un modo "guardar todo sin que yo intervenga".
- Las **Project Instructions solo aplican dentro de ese Project**. Para chats sueltos hay que usar el comando manual.
- Si renombrás o movés archivos del vault directamente desde el sistema de archivos (no desde Obsidian), los `[[wikilinks]]` pueden romperse, porque Obsidian solo actualiza links automáticamente cuando el renombrado pasa por la propia app.
- Esto guarda **resúmenes**, no transcripciones completas, para mantener las notas livianas. Si preferís transcripción completa, editá la plantilla.

## Licencia

MIT — usalo, modificalo y compartilo como quieras.
