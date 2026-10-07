# tarjetas-api — PROYECTO BASE DEL EJERCICIO

Completa esta API REST de tarjetas con ayuda de **GitHub Copilot** siguiendo el enunciado.

- Java 17 · Spring Boot 3.5 · Maven · puerto 8080
- Datos en memoria: no hay base de datos
- Swagger UI: http://localhost:8080/swagger-ui.html

## Qué ya está hecho
- `Tarjeta`: la clase con los datos de una tarjeta
- `TarjetaRepository`: la lista con 5 tarjetas y el método `listarTodas()`
- `TarjetaController`: el endpoint de ejemplo `GET /api/v1/tarjetas`

## Qué debes completar (busca los comentarios `TODO PARTE n`)
| Parte | Tarea |
|---|---|
| 2 | Consultar una tarjeta por número |
| 3 | Registrar una tarjeta nueva con validaciones |
| 4 | Bloquear una tarjeta |

Ejecuta `TarjetasApiApplication` con el botón ▶ y prueba con `requests.http` o Swagger.
