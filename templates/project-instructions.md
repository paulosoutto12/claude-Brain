# Project Instructions — pegar en "Custom Instructions" del Project

Al final de cada conversación en este proyecto, si el usuario escribe "/guardar"
o pide explícitamente guardar la sesión, usá las herramientas MCP de Obsidian para:

1. Resumir la conversación en 3-5 líneas + lista de puntos clave (NO transcripción completa)
2. Detectar 2-4 tags temáticos relevantes
3. Crear una nota nueva en la carpeta Claude/AAAA-MM/ del vault, con nombre
   "AAAA-MM-DD-titulo-breve.md", usando este frontmatter:
   ---
   tags: [tag1, tag2]
   fecha: AAAA-MM-DD
   proyecto: [nombre de este proyecto]
   ---
4. Buscar en la carpeta Temas/ si existe ya una nota hub para alguno de los tags
   detectados (ej: Temas/Finanzas.md).
   - Si existe, usar la herramienta de anexar contenido (vault_append o vault_patch)
     para agregarle un link [[nombre-de-la-nota-nueva]] bajo un encabezado
     "## Conversaciones relacionadas"
   - Si no existe, crear el hub con ese mismo link
5. Confirmarle al usuario qué archivo creó y qué hubs actualizó

## Personalización

- Cambiá "/guardar" por cualquier otra palabra clave que prefieras
- Si querés transcripción completa en vez de resumen, reemplazá el punto 1
- Si usás una estructura de carpetas distinta a Claude/ y Temas/, ajustá las rutas

## Migrar el historial de este Project (conversaciones anteriores a este setup)

Si este Project ya tenía conversaciones antes de configurar la integración con
Obsidian, podés migrarlas todas de una vez — o incluso migrar TODO tu historial
de Claude.ai sin filtrar, no solo este Project. Ver la sección "Migrar
proyectos anteriores a la integración" en el README del repo para las tres
opciones disponibles (volcado completo sin filtro, filtrado por proyecto, o
búsqueda directa sin exportar datos). No hace falta repetirlo acá manualmente.
