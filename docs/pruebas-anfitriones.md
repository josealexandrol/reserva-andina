# Pruebas del microservicio Anfitriones

**Responsable:** JOSE ALEXANDRO LOPEZ MAMANI
**Puerto:** 3001 · **Comando:** `docker compose exec anfitriones npm test`

| Microservicio | Prueba | Qué verifica | Resultado esperado |
|---|---|---|---|
| Anfitriones | GET /health responde que el servicio está activo | El servicio responde | 200 y `servicio: anfitriones` |
| Anfitriones | GET /anfitriones devuelve la lista | Consulta de todos los anfitriones | 200 y una lista con datos |
| Anfitriones | GET /anfitriones/1 devuelve un anfitrión existente | Consulta por id | 200 y `id: 1` |
| Anfitriones | GET /anfitriones/9999 responde 404 | Manejo de un anfitrión inexistente | 404 "Anfitrión no encontrado" |
| Anfitriones | POST /anfitriones rechaza un registro sin teléfono | Validación de campos obligatorios | 400 |
| Anfitriones | POST /anfitriones registra un anfitrión válido | Registro correcto | 201 y el anfitrión creado |

## Ejecución de las pruebas

<img width="886" height="413" alt="image" src="https://github.com/user-attachments/assets/c4920ec9-b339-48c0-ad79-1fc041f67dfc" />


## Prueba manual de la API

<img width="886" height="832" alt="image" src="https://github.com/user-attachments/assets/3f8f2d51-e53e-4507-bd09-693ce3c8d731" />
