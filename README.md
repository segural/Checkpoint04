# Checkpoint 4 — Orquestador de Soporte Técnico IT (Versión Corregida)

## 1. Cambios aplicados según correcciones
1. **Resolución interna del canal de equipo (sin sub-workflow externo huérfano):** Se reemplazó el nodo `Delegar_a_Worker_2_Slack` (que apuntaba a un sub-workflow externo no incluido en el JSON) por el nodo nativo **`Escalamiento_Slack_Canal`** (`n8n-nodes-base.slack`) directamente dentro del workflow, recibiendo el payload ya sanitizado por **`Sanitizar_W2`**.
2. **Trigger de correo entrante real:** Se reemplazó `Form_Trigger_Soporte` por **`Gmail_Trigger_Soporte`** (`n8n-nodes-base.gmailTrigger`) para capturar correos entrantes y evaluar en **`IF_Filtrar_Mensajes_Automaticos`** el remitente real (`email` / `no-reply@`) y el asunto/cuerpo (`asunto_correo` / `Auto-reply`, `Out of office`, `Undeliverable`).
3. **Barrera Human-in-the-loop y CRM:** Se conservan la búsqueda previa en HubSpot (`CRM_Buscar_Contacto` $\rightarrow$ `IF_Existe_En_CRM` $\rightarrow$ `CRM_Crear_Contacto`) para evitar duplicados / Error 409 y la creación de borrador en Gmail (`Crear_Borrador_Usuario` en modo `draft`).

## 2. Captura de Credenciales en n8n (3 conexiones activas en verde)
<img width="1175" height="348" alt="image" src="https://github.com/user-attachments/assets/145d5d57-dad8-4546-87f4-96e1acdde45e" />
<img width="1167" height="373" alt="image" src="https://github.com/user-attachments/assets/1c5368c2-df49-4504-9f0f-97c53ab9f97b" />
<img width="1180" height="311" alt="image" src="https://github.com/user-attachments/assets/9c854852-dfe8-4620-aede-1889c8a0083a" />

## 3. Los 4 nodos clave de la rúbrica
- **`IF_Filtrar_Mensajes_Automaticos`**: Filtra mensajes automáticos (`Auto-reply`, `Out of office`, `Undeliverable`) y casillas `no-reply@` sobre remitente y asunto reales del correo entrante.
- **`CRM_Buscar_Contacto` / `IF_Existe_En_CRM` / `CRM_Crear_Contacto`**: Realiza *Look up* por correo en el CRM antes de crear un contacto nuevo para prevenir duplicados y conflictos 409.
- **`Sanitizar_W2` + `Escalamiento_Slack_Canal`**: Recorta y limpia los campos enviados al canal del equipo (`sessionId`, `usuario`, `email`, `chatInput`, `motivo_escalamiento`) sin sub-workflows externos faltantes.
- **`Crear_Borrador_Usuario`**: Genera un borrador (`draft`) en Gmail para revisión humana (*Human-in-the-loop*) antes de cualquier envío externo.
