# Informe de pruebas — API Académica

| Dato | Valor |
| --- | --- |
| Estudiante | Manuel Pianeta |
| Fecha | 12 de octubre de 2026 |
| Versión probada | Rama `main` con los retos 1, 2 y 3 implementados |
| Comando de inicio | `python -m uvicorn main:app --reload` |
| URL base | `http://127.0.0.1:8000` |
| Herramientas | Swagger UI (`/docs`) para las capturas y un cliente HTTP (httpx) para ejecutar la batería completa contra el mismo servidor |
| Estado inicial de la base | Vacía (se eliminó `database.db` antes de iniciar) |
| Datos utilizados | Ficticios |

## Resumen

Se ejecutaron **62 casos** y **62 resultaron aprobados**. Los códigos `404` y `422` esperados ante recursos inexistentes o datos inválidos cuentan como pruebas aprobadas.

Después de reiniciar Uvicorn, `database.db` contiene las tablas: `course`, `student`. Ambos recursos comparten la misma base de datos; no se creó `students.db`.

La captura [P00-swagger-rutas.png](evidencias/P00-swagger-rutas.png) muestra en Swagger las rutas de Courses y Students.

## P01 — POST /courses

| Caso | Método y URL | Datos enviados | Resultado esperado | Resultado obtenido | Código HTTP | Estado | Evidencia |
| --- | --- | --- | --- | --- | --- | --- | --- |
| P01a | `POST /courses` | `{"name": "Desarrollo Web II", "credits": 3, "active": true}` | 201 y curso con ID generado | `{"id": 1, "name": "Desarrollo Web II", "credits": 3, "active": true}` | 201 | Aprobado | [P01a-crear-curso.png](evidencias/P01a-crear-curso.png) |
| P01b | `POST /courses` | `{"name": "Redes", "credits": 3}` | 201 y `active: true` por defecto | `{"id": 2, "name": "Redes", "credits": 3, "active": true}` | 201 | Aprobado | [P01b-crear-curso-active-omitido.png](evidencias/P01b-crear-curso-active-omitido.png) |
| P01c | `POST /courses` | `{"name": "Ética", "credits": 2, "active": false}` | 201 y `active: false` | `{"id": 3, "name": "Ética", "credits": 2, "active": false}` | 201 | Aprobado | [P01c-crear-curso-active-false.png](evidencias/P01c-crear-curso-active-false.png) |
| P01d | `POST /courses` | `{"name": "Bdd", "credits": 1, "active": true}` | 201 (nombre de 3 caracteres y créditos 1 aceptados) | `{"id": 4, "name": "Bdd", "credits": 1, "active": true}` | 201 | Aprobado | Registro de ejecución (sin captura propia) |
| P01e | `POST /courses` | `{"name": "A×80", "credits": 6, "active": true}` | 201 (nombre de 80 caracteres y créditos 6 aceptados) | `{"id": 5, "name": "A×80", "credits": 6, "active": true}` | 201 | Aprobado | Registro de ejecución (sin captura propia) |
| P01f | `POST /courses` | `{"name": "Re", "credits": 3, "active": true}` | 422 (nombre de 2 caracteres) | Error de validación en `name`: String should have at least 3 characters | 422 | Aprobado | [P01f-crear-curso-nombre-corto.png](evidencias/P01f-crear-curso-nombre-corto.png) |
| P01g | `POST /courses` | `{"name": "A×81", "credits": 3, "active": true}` | 422 (nombre de 81 caracteres) | Error de validación en `name`: String should have at most 80 characters | 422 | Aprobado | Registro de ejecución (sin captura propia) |
| P01h | `POST /courses` | `{"name": "Redes", "credits": 0, "active": true}` | 422 (créditos 0) | Error de validación en `credits`: Input should be greater than or equal to 1 | 422 | Aprobado | [P01h-crear-curso-creditos-0.png](evidencias/P01h-crear-curso-creditos-0.png) |
| P01i | `POST /courses` | `{"name": "Redes", "credits": 7, "active": true}` | 422 (créditos 7) | Error de validación en `credits`: Input should be less than or equal to 6 | 422 | Aprobado | Registro de ejecución (sin captura propia) |
| P01j | `POST /courses` | `{"credits": 3}` | 422 (falta name) | Error de validación en `name`: Field required | 422 | Aprobado | Registro de ejecución (sin captura propia) |
| P01k | `POST /courses` | `{"name": "Redes", "credits": "abc"}` | 422 (credits no entero) | Error de validación en `credits`: Input should be a valid integer, unable to parse string as an integer | 422 | Aprobado | Registro de ejecución (sin captura propia) |
| P01l | `POST /courses` | `{"name": "Redes", "credits": 3, "active": "invalido"}` | 422 (active no booleano) | Error de validación en `active`: Input should be a valid boolean, unable to interpret input | 422 | Aprobado | Registro de ejecución (sin captura propia) |

## P02 — GET /courses

| Caso | Método y URL | Datos enviados | Resultado esperado | Resultado obtenido | Código HTTP | Estado | Evidencia |
| --- | --- | --- | --- | --- | --- | --- | --- |
| P02a | `GET /courses` | — | 200 y lista vacía `[]` | `[]` | 200 | Aprobado | [P02a-listar-cursos-vacio.png](evidencias/P02a-listar-cursos-vacio.png) |
| P02b | `GET /courses` | — | 200 y 5 cursos (activos e inactivos); los rechazados no se crearon | Lista con 5 registro(s) | 200 | Aprobado | [P02b-listar-cursos.png](evidencias/P02b-listar-cursos.png) |

## P03 — GET /courses/{course_id}

| Caso | Método y URL | Datos enviados | Resultado esperado | Resultado obtenido | Código HTTP | Estado | Evidencia |
| --- | --- | --- | --- | --- | --- | --- | --- |
| P03a | `GET /courses/1` | — | 200 y datos del curso | `{"id": 1, "name": "Desarrollo Web II", "credits": 3, "active": true}` | 200 | Aprobado | [P03a-consultar-curso.png](evidencias/P03a-consultar-curso.png) |
| P03b | `GET /courses/999` | — | 404 `Course not found` | `{"detail": "Course not found"}` | 404 | Aprobado | [P03b-consultar-curso-inexistente.png](evidencias/P03b-consultar-curso-inexistente.png) |
| P03c | `GET /courses/abc` | — | 422 (ID no entero) | Error de validación en `course_id`: Input should be a valid integer, unable to parse string as an integer | 422 | Aprobado | Registro de ejecución (sin captura propia) |

## P04 — PUT /courses/{course_id}

| Caso | Método y URL | Datos enviados | Resultado esperado | Resultado obtenido | Código HTTP | Estado | Evidencia |
| --- | --- | --- | --- | --- | --- | --- | --- |
| P04a | `PUT /courses/2` | `{"name": "Programación II", "credits": 4, "active": false}` | 200, mismo ID y nuevos valores (`active` pasa a false) | `{"id": 2, "name": "Programación II", "credits": 4, "active": false}` | 200 | Aprobado | [P04a-actualizar-curso.png](evidencias/P04a-actualizar-curso.png) |
| P04b | `GET /courses/2` | — | 200 y cambios guardados con el mismo ID | `{"id": 2, "name": "Programación II", "credits": 4, "active": false}` | 200 | Aprobado | [P04b-consulta-despues-update.png](evidencias/P04b-consulta-despues-update.png) |
| P04c | `GET /courses` | — | 200 y siguen siendo 5 cursos (el PUT no crea registros) | Lista con 5 registro(s) | 200 | Aprobado | Registro de ejecución (sin captura propia) |
| P04d | `PUT /courses/2` | `{"name": "Pro", "credits": 1, "active": true}` | 200 (límites válidos: nombre 3, créditos 1) | `{"id": 2, "name": "Pro", "credits": 1, "active": true}` | 200 | Aprobado | Registro de ejecución (sin captura propia) |
| P04e | `PUT /courses/2` | `{"name": "Programación II", "credits": 4, "active": false}` | 200 (se restauran los valores) | `{"id": 2, "name": "Programación II", "credits": 4, "active": false}` | 200 | Aprobado | Registro de ejecución (sin captura propia) |
| P04f | `PUT /courses/2` | `{"name": "Programación II", "credits": 0, "active": false}` | 422 (créditos 0) | Error de validación en `credits`: Input should be greater than or equal to 1 | 422 | Aprobado | Registro de ejecución (sin captura propia) |
| P04g | `PUT /courses/2` | `{"name": "Programación II", "credits": 7, "active": false}` | 422 (créditos 7) | Error de validación en `credits`: Input should be less than or equal to 6 | 422 | Aprobado | [P04g-actualizar-creditos-7.png](evidencias/P04g-actualizar-creditos-7.png) |
| P04h | `PUT /courses/2` | `{"name": "Pr", "credits": 4, "active": false}` | 422 (nombre de 2 caracteres) | Error de validación en `name`: String should have at least 3 characters | 422 | Aprobado | Registro de ejecución (sin captura propia) |
| P04i | `PUT /courses/2` | `{"name": "Programación II", "credits": 4}` | 422 (falta active) | Error de validación en `active`: Field required | 422 | Aprobado | [P04i-actualizar-sin-active.png](evidencias/P04i-actualizar-sin-active.png) |
| P04j | `PUT /courses/2` | `{"credits": 4, "active": false}` | 422 (falta name) | Error de validación en `name`: Field required | 422 | Aprobado | Registro de ejecución (sin captura propia) |
| P04k | `PUT /courses/2` | `{"name": "Programación II", "active": false}` | 422 (falta credits) | Error de validación en `credits`: Field required | 422 | Aprobado | Registro de ejecución (sin captura propia) |
| P04l | `GET /courses/2` | — | 200 y el curso conserva sus valores tras los PUT rechazados | `{"id": 2, "name": "Programación II", "credits": 4, "active": false}` | 200 | Aprobado | Registro de ejecución (sin captura propia) |
| P04m | `PUT /courses/999` | `{"name": "Programación II", "credits": 4, "active": false}` | 404 `Course not found` | `{"detail": "Course not found"}` | 404 | Aprobado | [P04m-actualizar-inexistente.png](evidencias/P04m-actualizar-inexistente.png) |
| P04n | `PUT /courses/abc` | `{"name": "Programación II", "credits": 4, "active": false}` | 422 (ID no entero) | Error de validación en `course_id`: Input should be a valid integer, unable to parse string as an integer | 422 | Aprobado | Registro de ejecución (sin captura propia) |

## P05 — DELETE /courses/{course_id}

| Caso | Método y URL | Datos enviados | Resultado esperado | Resultado obtenido | Código HTTP | Estado | Evidencia |
| --- | --- | --- | --- | --- | --- | --- | --- |
| P05a | `DELETE /courses/3` | — | 200 y `{"message": "Course deleted"}` | `{"message": "Course deleted"}` | 200 | Aprobado | [P05a-eliminar-curso.png](evidencias/P05a-eliminar-curso.png) |
| P05b | `GET /courses/3` | — | 404 (el curso ya no existe) | `{"detail": "Course not found"}` | 404 | Aprobado | [P05b-consulta-despues-delete.png](evidencias/P05b-consulta-despues-delete.png) |
| P05c | `GET /courses` | — | 200, 4 cursos, sin el eliminado y con los demás intactos | Lista con 4 registro(s) | 200 | Aprobado | [P05c-listar-despues-delete.png](evidencias/P05c-listar-despues-delete.png) |
| P05d | `DELETE /courses/3` | — | 404 al repetir el DELETE | `{"detail": "Course not found"}` | 404 | Aprobado | [P05d-eliminar-de-nuevo.png](evidencias/P05d-eliminar-de-nuevo.png) |
| P05e | `DELETE /courses/999` | — | 404 (ID inexistente) | `{"detail": "Course not found"}` | 404 | Aprobado | Registro de ejecución (sin captura propia) |
| P05f | `DELETE /courses/abc` | — | 422 (ID no entero) | Error de validación en `course_id`: Input should be a valid integer, unable to parse string as an integer | 422 | Aprobado | Registro de ejecución (sin captura propia) |

## P06 — POST /students

| Caso | Método y URL | Datos enviados | Resultado esperado | Resultado obtenido | Código HTTP | Estado | Evidencia |
| --- | --- | --- | --- | --- | --- | --- | --- |
| P06a | `POST /students` | `{"name": "Ana Pérez", "email": "ana@universidad.edu", "semester": 4}` | 201, ID generado y `active: true` (omitido) | `{"id": 1, "name": "Ana Pérez", "email": "ana@universidad.edu", "semester": 4, "active": true}` | 201 | Aprobado | [P06a-crear-estudiante.png](evidencias/P06a-crear-estudiante.png) |
| P06b | `POST /students` | `{"name": "Luis Gómez", "email": "luis@universidad.edu", "semester": 2, "active": false}` | 201 y `active: false` | `{"id": 2, "name": "Luis Gómez", "email": "luis@universidad.edu", "semester": 2, "active": false}` | 201 | Aprobado | [P06b-crear-estudiante-inactivo.png](evidencias/P06b-crear-estudiante-inactivo.png) |
| P06c | `POST /students` | `{"name": "Eva", "email": "eva@universidad.edu", "semester": 1}` | 201 (nombre de 3 caracteres y semestre 1) | `{"id": 3, "name": "Eva", "email": "eva@universidad.edu", "semester": 1, "active": true}` | 201 | Aprobado | Registro de ejecución (sin captura propia) |
| P06d | `POST /students` | `{"name": "B×80", "email": "b@universidad.edu", "semester": 10}` | 201 (nombre de 80 caracteres y semestre 10) | `{"id": 4, "name": "B×80", "email": "b@universidad.edu", "semester": 10, "active": true}` | 201 | Aprobado | Registro de ejecución (sin captura propia) |
| P06e | `POST /students` | `{"name": "Ana Pérez", "email": "ana@universidad.edu", "semester": 0}` | 422 (semestre 0) | Error de validación en `semester`: Input should be greater than or equal to 1 | 422 | Aprobado | [P06e-semestre-0.png](evidencias/P06e-semestre-0.png) |
| P06f | `POST /students` | `{"name": "Ana Pérez", "email": "ana@universidad.edu", "semester": 11}` | 422 (semestre 11) | Error de validación en `semester`: Input should be less than or equal to 10 | 422 | Aprobado | [P06f-semestre-11.png](evidencias/P06f-semestre-11.png) |
| P06g | `POST /students` | `{"name": "An", "email": "ana@universidad.edu", "semester": 4}` | 422 (nombre de 2 caracteres) | Error de validación en `name`: String should have at least 3 characters | 422 | Aprobado | [P06g-nombre-corto.png](evidencias/P06g-nombre-corto.png) |
| P06h | `POST /students` | `{"name": "C×81", "email": "c@universidad.edu", "semester": 4}` | 422 (nombre de 81 caracteres) | Error de validación en `name`: String should have at most 80 characters | 422 | Aprobado | Registro de ejecución (sin captura propia) |
| P06i | `POST /students` | `{"name": "Ana Pérez", "email": "", "semester": 4}` | 422 (email vacío) | Error de validación en `email`: String should have at least 1 character | 422 | Aprobado | [P06i-email-vacio.png](evidencias/P06i-email-vacio.png) |
| P06j | `POST /students` | `{"name": "Ana Pérez", "semester": 4}` | 422 (falta email) | Error de validación en `email`: Field required | 422 | Aprobado | Registro de ejecución (sin captura propia) |
| P06k | `POST /students` | `{"name": "Ana Pérez", "email": "ana@universidad.edu"}` | 422 (falta semester) | Error de validación en `semester`: Field required | 422 | Aprobado | Registro de ejecución (sin captura propia) |
| P06l | `POST /students` | `{"name": "Ana Pérez", "email": "ana@universidad.edu", "semester": "abc"}` | 422 (semester no entero) | Error de validación en `semester`: Input should be a valid integer, unable to parse string as an integer | 422 | Aprobado | Registro de ejecución (sin captura propia) |
| P06m | `POST /students` | `{"name": "Ana Pérez", "email": "ana@universidad.edu", "semester": 4, "active": "invalido"}` | 422 (active no booleano) | Error de validación en `active`: Input should be a valid boolean, unable to interpret input | 422 | Aprobado | Registro de ejecución (sin captura propia) |

## P07 — GET /students

| Caso | Método y URL | Datos enviados | Resultado esperado | Resultado obtenido | Código HTTP | Estado | Evidencia |
| --- | --- | --- | --- | --- | --- | --- | --- |
| P07a | `GET /students` | — | 200 y lista vacía `[]` | `[]` | 200 | Aprobado | Registro de ejecución (sin captura propia) |
| P07b | `GET /students` | — | 200 y 4 estudiantes (activos e inactivos); los rechazados no se crearon | Lista con 4 registro(s) | 200 | Aprobado | [P07b-listar-estudiantes.png](evidencias/P07b-listar-estudiantes.png) |

## P08 — GET /students/{student_id}

| Caso | Método y URL | Datos enviados | Resultado esperado | Resultado obtenido | Código HTTP | Estado | Evidencia |
| --- | --- | --- | --- | --- | --- | --- | --- |
| P08a | `GET /students/1` | — | 200 y datos de Ana Pérez | `{"id": 1, "name": "Ana Pérez", "email": "ana@universidad.edu", "semester": 4, "active": true}` | 200 | Aprobado | [P08a-consultar-estudiante.png](evidencias/P08a-consultar-estudiante.png) |
| P08b | `GET /students/999` | — | 404 `Student not found` | `{"detail": "Student not found"}` | 404 | Aprobado | [P08b-estudiante-inexistente.png](evidencias/P08b-estudiante-inexistente.png) |
| P08c | `GET /students/abc` | — | 422 (ID no entero) | Error de validación en `student_id`: Input should be a valid integer, unable to parse string as an integer | 422 | Aprobado | Registro de ejecución (sin captura propia) |

## P09 — Persistencia e integración (tras reiniciar Uvicorn)

| Caso | Método y URL | Datos enviados | Resultado esperado | Resultado obtenido | Código HTTP | Estado | Evidencia |
| --- | --- | --- | --- | --- | --- | --- | --- |
| P09a | `GET /courses` | — | Tras reiniciar Uvicorn: 200 y los mismos 4 cursos | Lista con 4 registro(s) | 200 | Aprobado | [P09a-cursos-tras-reinicio.png](evidencias/P09a-cursos-tras-reinicio.png) |
| P09b | `GET /courses/2` | — | Tras reiniciar: el cambio del PUT permanece | `{"id": 2, "name": "Programación II", "credits": 4, "active": false}` | 200 | Aprobado | Registro de ejecución (sin captura propia) |
| P09c | `GET /courses/3` | — | Tras reiniciar: el curso eliminado sigue sin existir (404) | `{"detail": "Course not found"}` | 404 | Aprobado | [P09c-eliminado-tras-reinicio.png](evidencias/P09c-eliminado-tras-reinicio.png) |
| P09d | `GET /students` | — | Tras reiniciar: 200 y los mismos 4 estudiantes | Lista con 4 registro(s) | 200 | Aprobado | [P09d-estudiantes-tras-reinicio.png](evidencias/P09d-estudiantes-tras-reinicio.png) |
| P09e | `GET /students/2` | — | Tras reiniciar: Luis Gómez conserva `active: false` | `{"id": 2, "name": "Luis Gómez", "email": "luis@universidad.edu", "semester": 2, "active": false}` | 200 | Aprobado | Registro de ejecución (sin captura propia) |
| P09f | `DELETE /courses/4` | — | 200; eliminar un curso no afecta a estudiantes | `{"message": "Course deleted"}` | 200 | Aprobado | Registro de ejecución (sin captura propia) |
| P09g | `GET /students` | — | 200 y siguen 4 estudiantes | Lista con 4 registro(s) | 200 | Aprobado | Registro de ejecución (sin captura propia) |

## Observaciones

- Las peticiones rechazadas con `422` no crearon registros ni modificaron datos: la cantidad de cursos y estudiantes se verificó después de cada grupo (P02b, P04c, P04l, P07b).
- El `PUT` conserva el ID y no crea registros nuevos (P04a–P04c).
- Tras detener y volver a iniciar Uvicorn, los cursos, estudiantes y cambios del `PUT` permanecen, y el curso eliminado sigue sin existir (P09a–P09e).
- Las operaciones sobre cursos no afectan a los estudiantes (P09f–P09g).
- Una misma captura puede respaldar varios casos del mismo grupo; los casos sin captura propia se verificaron con la respuesta registrada en la tabla.
