# Instrucciones para GitHub Copilot

Proyecto **tarjetas-api**: una API REST sencilla con Java 17 y Spring Boot, usada en un ejercicio para desarrolladores junior.

## Reglas para el código
- Mantén el código simple y fácil de leer para alguien que está aprendiendo.
- Usa solo dos clases para la lógica: `TarjetaController` (endpoints) y `TarjetaRepository` (datos).
- Los datos se guardan en una `ArrayList` en memoria. No agregues base de datos.
- Usa ciclos `for` e `if`. **No uses streams, lambdas, Optional ni programación reactiva.**
- No agregues dependencias nuevas al `pom.xml`.
- No generes pruebas unitarias.
- Agrega comentarios cortos en español que expliquen cada paso.

## Respuestas HTTP
- 200 OK para consultas y actualizaciones correctas.
- 201 Created al registrar una tarjeta.
- 400 Bad Request cuando los datos no son válidos.
- 404 Not Found cuando la tarjeta no existe.
- 409 Conflict cuando el número de tarjeta ya existe.

Responde siempre en español y explica los cambios en pocas líneas.
