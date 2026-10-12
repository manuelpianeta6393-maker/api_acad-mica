# API Académica

API educativa para gestionar cursos (CRUD completo) y estudiantes (creación y consulta) con **FastAPI, SQLModel y SQLite**. El código incluye comentarios en español para estudiar cómo se conectan las rutas HTTP, la validación de datos y la persistencia.

La explicación detallada del proyecto, la práctica con Swagger y los ejercicios están en la [guía para estudiantes](docs.md).

## Funcionalidades

- **Cursos:** crear, listar, consultar por ID, actualizar (`PUT`, actualización completa) y eliminar.
- **Estudiantes:** crear, listar y consultar por ID.
- Validar los datos de entrada y responder con errores HTTP (`404` para recursos inexistentes, `422` para datos o IDs inválidos).
- Conservar los registros en una única base de datos SQLite local (`database.db`) con las tablas `course` y `student`.

Quedan como posibles ampliaciones: actualización y eliminación de estudiantes, matrícula (relación estudiante–curso), autenticación y paginación.

## Tecnologías

| Herramienta | Uso |
| --- | --- |
| FastAPI | Aplicación web, rutas y documentación interactiva. |
| Uvicorn | Servidor que ejecuta la API. |
| SQLModel | Modelos de tablas y operaciones de base de datos. |
| Pydantic | Esquemas y validación de datos. |
| SQLite | Almacenamiento en el archivo `database.db`. |

Las versiones utilizadas están en [requirements.txt](requirements.txt). El proyecto se ha ejecutado con **Python 3.12**.

## Inicio rápido

Abre una terminal en la carpeta del proyecto, donde está `main.py`.

### 1. Crear y activar el entorno virtual

En Linux o macOS:

```bash
python3 -m venv venv
source venv/bin/activate
```

En Windows PowerShell:

```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
```

Si el entorno `venv` ya existe, basta con activarlo.

### 2. Instalar las dependencias

```bash
python -m pip install -r requirements.txt
```

### 3. Ejecutar la API

```bash
python -m uvicorn main:app --reload --host 127.0.0.1 --port 8000
```

`main:app` identifica el objeto `app` del archivo `main.py`. La opción `--reload` recarga el servidor cuando editas el código durante el desarrollo. Para detenerlo, presiona `Ctrl+C`.

Abre la [documentación interactiva Swagger](http://127.0.0.1:8000/docs) para probar los endpoints con **Try it out → Execute**.

La base de datos y sus tablas se crean al iniciar la aplicación. Ejecuta el comando desde la raíz del proyecto para usar siempre el mismo archivo `database.db`.

## Endpoints

| Método | Ruta | Operación | Estado de éxito |
| --- | --- | --- | --- |
| `POST` | `/courses` | Crear un curso. | `201` |
| `GET` | `/courses` | Listar todos los cursos. | `200` |
| `GET` | `/courses/{course_id}` | Consultar un curso por ID. | `200` |
| `PUT` | `/courses/{course_id}` | Actualizar todos los campos de un curso. | `200` |
| `DELETE` | `/courses/{course_id}` | Eliminar un curso; responde `{"message": "Course deleted"}`. | `200` |
| `POST` | `/students` | Crear un estudiante. | `201` |
| `GET` | `/students` | Listar todos los estudiantes. | `200` |
| `GET` | `/students/{student_id}` | Consultar un estudiante por ID. | `200` |

Un curso o estudiante inexistente devuelve `404` (`Course not found` / `Student not found`). Los datos que incumplen el esquema devuelven `422`. La ruta `/` no está definida; utiliza `/docs` o `/courses`.

## Ejemplo de uso

Para crear un curso, envía este cuerpo JSON a `POST /courses` desde Swagger:

```json
{
  "name": "Programación I",
  "credits": 3,
  "active": true
}
```

También puedes hacerlo desde otra terminal con `curl` en Bash:

```bash
curl -i -X POST http://127.0.0.1:8000/courses \
  -H 'Content-Type: application/json' \
  -d '{"name": "Programación I", "credits": 3, "active": true}'
```

La API responde con `201 Created` y el curso guardado:

```json
{
  "id": 1,
  "name": "Programación I",
  "credits": 3,
  "active": true
}
```

El ID es ilustrativo: SQLite lo asigna automáticamente. Usa el valor recibido para consultar el curso en `/courses/{course_id}`. Cada creación válida guarda un nuevo registro.

### Reglas de entrada

**Cursos — `POST /courses` (`CourseCreate`) y `PUT /courses/{course_id}` (`CourseUpdate`)**

| Campo | Regla |
| --- | --- |
| `name` | Obligatorio; entre 3 y 80 caracteres. |
| `credits` | Obligatorio; entero entre 1 y 6. |
| `active` | Booleano. Opcional al crear (predeterminado `true`); obligatorio al actualizar. |

**Estudiantes — `POST /students` (`StudentCreate`)**

| Campo | Regla |
| --- | --- |
| `name` | Obligatorio; entre 3 y 80 caracteres. |
| `email` | Obligatorio; texto no vacío. |
| `semester` | Obligatorio; entero entre 1 y 10. |
| `active` | Opcional; booleano con valor predeterminado `true`. |

Ejemplo para crear un estudiante:

```json
{
  "name": "Ana Pérez",
  "email": "ana@universidad.edu",
  "semester": 4
}
```

## Estructura del proyecto

```text
api_acad-mica/
├── main.py                       # Aplicación, ciclo de vida y registro de routers
├── database/database.py          # Motor SQLite y sesiones (compartidos)
├── models/
│   ├── course.py                 # Tabla course
│   └── student.py                # Tabla student
├── schemas/
│   ├── course.py                 # CourseCreate, CourseUpdate, CourseResponse, MessageResponse
│   └── student.py                # StudentCreate, StudentResponse
├── services/
│   ├── course_service.py         # Crear, listar, consultar, actualizar y eliminar cursos
│   └── student_service.py        # Crear, listar y consultar estudiantes
├── routers/
│   ├── courses.py                # Endpoints /courses
│   └── students.py               # Endpoints /students
├── entrega-pruebas/
│   ├── informe-pruebas.md        # Informe de pruebas
│   └── evidencias/               # Capturas de Swagger
├── requirements.txt              # Dependencias
├── docs.md                       # Guía didáctica detallada
├── ejercicio.md                  # Enunciado de los retos
└── README.md                     # Presentación e inicio rápido
```

El flujo principal es: **petición HTTP → router y validación → servicio → base de datos → respuesta JSON**.

Para seguir el código paso a paso, comprender `Depends`, `Session`, `commit()` y `refresh()`, o resolver problemas de ejecución, consulta [docs.md](docs.md).
