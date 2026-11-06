ruebas del microservicio Notificaciones

**Responsable:** OSCAR AILTHON ESPINOZA NIETO
**Puerto:** 3005 · **Comando:** `docker compose exec notificaciones npm test`
| Microservicio | Prueba | Qué verifica | Resultado esperado |
|---|---|---|---|
| Notificaciones | GET /health responde que el servicio está activo | El servicio responde | 200 y `servicio: notificaciones` |
| Notificaciones | GET /notificaciones devuelve el historial | Consulta de avisos | 200 y una lista con datos |
| Notificaciones | POST /notificaciones registra un aviso válido | Registro por WhatsApp | 201 y `canal: whatsapp` |
| Notificaciones | POST /notificaciones usa SMS por defecto | Canal por defecto | 201 y `canal: sms` |
| Notificaciones | POST /notificaciones rechaza un aviso sin destinatario | Validación de campos obligatorios | 400 |
| Notificaciones | POST /notificaciones rechaza un canal no permitido | Validación del canal | 400 |

## Ejecución de las pruebas
<img width="987" height="523" alt="image" src="https://github.com/user-attachments/assets/feaceb06-18fb-4b46-bca1-3f74e464a96a" />

## Prueba manual de la API
<img width="1655" height="463" alt="image" src="https://github.com/user-attachments/assets/f39ef134-acd8-4089-a820-b896b3eb68b5" />

