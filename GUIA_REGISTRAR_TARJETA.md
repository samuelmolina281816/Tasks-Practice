
---

# Guía para entender `registrarTarjeta`

## 1. Qué hace el proyecto

`tarjetas-api` es una API REST hecha con Java y Spring Boot. Un cliente envía una petición HTTP y la aplicación responde, normalmente con datos JSON y un código de estado HTTP.

Para registrar una tarjeta, el flujo es:

```text
Cliente
  │ POST /api/v1/tarjetas con JSON
  ▼
TarjetaController.registrarTarjeta(...)
  │ valida los datos
  ├── inválidos ───────────────► HTTP 400
  ├── número ya registrado ───► HTTP 409
  └── válidos
        │ asigna estado ACTIVA
        ▼
     TarjetaRepository.guardar(...)
        │ agrega a la ArrayList
        ▼
     HTTP 201 con la tarjeta creada
```

En esta versión, las tarjetas se guardan en memoria. No se utiliza una base de datos, por lo que los registros agregados se pierden al reiniciar la aplicación.

## 2. Conceptos básicos de Java y Spring usados por el método

- **Método:** bloque de código que realiza una tarea. En este caso, `registrarTarjeta` recibe, valida y registra una tarjeta.
- **Parámetro:** dato que recibe el método. `Tarjeta tarjeta` es el objeto construido a partir del JSON enviado por el cliente.
- **`if`:** permite tomar decisiones según una condición.
- **`return`:** termina el método y entrega su respuesta.
- **`null`:** indica que no hay un objeto o valor disponible.
- **`ResponseEntity`:** permite definir explícitamente el código HTTP y el cuerpo de la respuesta.
- **`@RequestBody`:** indica a Spring que convierta el cuerpo JSON de la petición a un objeto Java `Tarjeta`.
- **`@PostMapping`:** asocia el método con las peticiones HTTP `POST` a `/api/v1/tarjetas`.

La clase `Tarjeta` define los datos del objeto: número, titular, tipo, límite y estado. El repositorio mantiene una `ArrayList` y proporciona las operaciones de búsqueda y guardado.

## 3. El método paso a paso

La firma del endpoint es:

```java
@PostMapping
public ResponseEntity<Tarjeta> registrarTarjeta(@RequestBody Tarjeta tarjeta) {
```

`@PostMapping` registra un recurso nuevo. La ruta completa se forma combinando esta anotación con `@RequestMapping("/api/v1/tarjetas")` de la clase, así que el endpoint es:

```text
POST /api/v1/tarjetas
```

`@RequestBody` permite que Spring convierta un JSON como este en un objeto `Tarjeta`:

```json
{
  "numero": "4111000000000006",
  "titular": "Pedro Salas",
  "tipo": "CREDITO",
  "limite": 2500.00
}
```

El estado no necesita venir en la petición: el servidor lo fija al registrar.

### Validación R1: número y titular obligatorios

```java
if (tarjeta == null || tarjeta.getNumero() == null || tarjeta.getNumero().length() != 16
        || tarjeta.getTitular() == null || tarjeta.getTitular().trim().isEmpty()) {
    return ResponseEntity.badRequest().build();
}
```

Se rechaza la petición si el objeto o el número son nulos, si el número no tiene 16 caracteres o si el titular es nulo o está vacío (incluidos los espacios en blanco).

Si ocurre cualquiera de esas situaciones, `badRequest()` responde con **400 Bad Request** y `build()` genera una respuesta sin cuerpo.

### Validación R2: tipo permitido

```java
if (!"CREDITO".equals(tarjeta.getTipo()) && !"DEBITO".equals(tarjeta.getTipo())) {
    return ResponseEntity.badRequest().build();
}
```

Solo se aceptan exactamente `CREDITO` y `DEBITO`. El uso de `"CREDITO".equals(...)` evita un error si el tipo recibido es `null`. La comparación distingue mayúsculas y minúsculas; por ejemplo, `credito` se rechaza.

### Validación R3: límite para crédito

```java
if ("CREDITO".equals(tarjeta.getTipo()) && tarjeta.getLimite() <= 0) {
    return ResponseEntity.badRequest().build();
}
```

Una tarjeta de crédito debe tener un límite estrictamente mayor que cero. Esta regla no exige límite positivo para una tarjeta de débito.

Si falla R1, R2 o R3, el método termina en ese momento y devuelve **400**: no continúa con la búsqueda de duplicados ni guarda la tarjeta.

### Validación R4: número no duplicado

```java
if (tarjetaRepository.buscarPorNumero(tarjeta.getNumero()) != null) {
    return ResponseEntity.status(HttpStatus.CONFLICT).build();
}
```

El controlador delega la búsqueda al repositorio. `buscarPorNumero` recorre la lista y devuelve la tarjeta encontrada, o `null` si no existe.

Si devuelve una tarjeta, el número ya está registrado y se responde con **409 Conflict**. Así se evita agregar el mismo número otra vez.

### Regla R5: guardar y responder

```java
tarjeta.setEstado("ACTIVA");
tarjetaRepository.guardar(tarjeta);
return ResponseEntity.status(HttpStatus.CREATED).body(tarjeta);
```

El servidor establece el estado `ACTIVA`, aunque el cliente haya enviado otro estado. Después, el repositorio agrega el objeto a la lista en memoria. Finalmente, `201 Created` indica que se creó el recurso, y `body(tarjeta)` incluye la tarjeta en la respuesta JSON.

## 4. Criterios de aceptación

| Regla | Entrada o condición | Resultado esperado |
|---|---|---|
| R1 | Número nulo o con longitud distinta de 16 | **400 Bad Request** |
| R1 | Titular nulo, vacío o compuesto solo por espacios | **400 Bad Request** |
| R2 | Tipo distinto de `CREDITO` o `DEBITO` | **400 Bad Request** |
| R3 | Tipo `CREDITO` con límite menor o igual a cero | **400 Bad Request** |
| R4 | Número igual al de una tarjeta existente | **409 Conflict** |
| R5 | Tarjeta válida con número nuevo | Se guarda con estado `ACTIVA` y se responde **201 Created** |

Las reglas se evalúan en orden. Si una tarjeta incumple varias, se responde por la primera regla que falla. Por ejemplo, si su tipo es inválido y el número también está repetido, R2 ocurre antes de R4 y el resultado es **400**, no **409**.

## 5. Pruebas manuales

Puedes iniciar la aplicación y ejecutar solicitudes desde `requests.http` o Swagger UI.

**Alta válida:** espera **201** y verifica que la respuesta incluya el estado `ACTIVA`.

```http
POST http://localhost:8080/api/v1/tarjetas
Content-Type: application/json

{ "numero": "4111000000000006", "titular": "Pedro Salas", "tipo": "CREDITO", "limite": 2500.00 }
```

**Crédito sin límite:** espera **400**.

```json
{ "numero": "4111000000000007", "titular": "Lucía Paz", "tipo": "CREDITO", "limite": 0 }
```

**Tipo no permitido:** espera **400**.

```json
{ "numero": "4111000000000008", "titular": "Lucía Paz", "tipo": "PREPAGO", "limite": 100 }
```

**Número existente:** envía un número que ya esté en la lista inicial, por ejemplo `4111000000000001`. Espera **409** si el resto de los datos cumple las validaciones previas.

Tras registrar una tarjeta válida, consulta su número con `GET /api/v1/tarjetas/{numero}`. Debe aparecer en la lista mientras la aplicación siga ejecutándose. Al reiniciarla, la lista vuelve a los datos iniciales porque el repositorio no utiliza almacenamiento permanente.

## 6. Observaciones para continuar el proyecto

- R1 comprueba la **longitud** del número, no que sus 16 caracteres sean dígitos. Si el criterio exige únicamente dígitos, esa validación tendría que agregarse.
- El tipo debe coincidir exactamente en mayúsculas: `CREDITO` o `DEBITO`.
- Las respuestas 400 y 409 no incluyen un mensaje explicativo; incluir un cuerpo con detalles podría facilitar el uso de la API.
- La comprobación de duplicados y el guardado son dos operaciones separadas. En un sistema con varias peticiones simultáneas, una base de datos con una restricción de unicidad sería más segura.
- Los datos monetarios usan `double`; para sistemas financieros normalmente se prefiere `BigDecimal`, que evita ciertos problemas de precisión decimal.
- La lista del repositorio es temporal: sirve para aprender y hacer pruebas, pero no reemplaza una base de datos.

**Idea clave:** el controlador recibe la petición y aplica las reglas; el repositorio busca y guarda; `ResponseEntity` comunica al cliente si la operación fue exitosa o por qué categoría de error no se completó.