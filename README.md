# ClassroomHub

ClassroomHub es un proyecto de aprendizaje con **Django** que presenta un catálogo de recursos de estudio. En el Laboratorio 06 los recursos estaban escritos dentro de `pages/views.py`; en el **Laboratorio 07** pasan a almacenarse en **MySQL/MariaDB** mediante modelos, migraciones y el ORM de Django, y se administran desde **Django Admin**.

## Funcionalidades

- **Inicio** (`/`), **Recursos** (`/resources/`) y **About** (`/about/`), con la plantilla compartida `pages/base.html`.
- `/resources/` consulta la base de datos con el ORM y muestra título, descripción, categoría y disponibilidad (*Available* / *Coming soon*). Si no hay registros, muestra *No resources are available.*
- Modelos `Category` y `Resource` (relación uno a muchos con `on_delete=PROTECT` y `related_name="resources"`). `Resource` incluye el campo `resource_type`, agregado mediante una migración posterior.
- Administración de categorías y recursos en `/admin/` (columnas, filtros y búsqueda).
- 10 pruebas automatizadas en `pages/tests.py`, que usan una base de pruebas independiente (`test_classroom_hub`).

## Requisitos

- Python 3.12
- **MySQL 8.0 o superior** o **MariaDB 10.4 o superior** (basta con uno de los dos)
- Django 4.2.11, `mysqlclient` y `python-dotenv` (se instalan con `requirements.txt`)

## 1. Entorno virtual (Windows)

Desde la carpeta `classroom-hub/` (donde está `manage.py`):

```bash
python -m venv venv
venv\Scripts\activate
```

En Linux/macOS: `python3 -m venv venv` y `source venv/bin/activate`.

## 2. Instalar dependencias

```bash
python -m pip install -r requirements.txt
```

## 3. Preparar la base de datos

Conéctate al servidor con una cuenta administrativa **solo para este paso** (`mysql -u root -p`) y ejecuta:

```sql
CREATE DATABASE classroom_hub CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

CREATE USER 'classroomhub_user'@'localhost' IDENTIFIED BY 'cambia_esta_contrasena';

GRANT ALL PRIVILEGES ON classroom_hub.* TO 'classroomhub_user'@'localhost';
GRANT ALL PRIVILEGES ON test_classroom_hub.* TO 'classroomhub_user'@'localhost';
FLUSH PRIVILEGES;
```

La aplicación usa **solo** la cuenta `classroomhub_user` (sin privilegios administrativos globales). El segundo `GRANT` permite que Django cree y elimine la base de pruebas `test_classroom_hub` al ejecutar los tests.

## 4. Variables de entorno

Copia el archivo de ejemplo y completa tus valores reales:

```bash
copy .env.example .env
```

(Linux/macOS: `cp .env.example .env`)

```
DB_NAME=classroom_hub
DB_USER=classroomhub_user
DB_PASSWORD=replace_with_a_private_password
DB_HOST=127.0.0.1
DB_PORT=3306
```

## 5. Migraciones

```bash
python manage.py check
python manage.py migrate
python manage.py showmigrations pages
```

Migraciones de la app `pages`: `0001_initial` (crea `Category` y `Resource`) y las posteriores que agregan `resource_type` a `Resource`.

Para inspeccionar el SQL de una migración: `python manage.py sqlmigrate pages 0001`.

## 6. Crear un superusuario e iniciar el servidor

```bash
python manage.py createsuperuser
python manage.py runserver
```

- Sitio: <http://127.0.0.1:8000/>
- Catálogo: <http://127.0.0.1:8000/resources/>
- Administración: <http://127.0.0.1:8000/admin/> (crea categorías y recursos; aparecen de inmediato en `/resources/`)

Los datos se conservan al detener y reiniciar el servidor porque están almacenados en MySQL/MariaDB.

## 7. Pruebas

```bash
python manage.py test -v 2
```

Resultado esperado: `Ran 10 tests ... OK`. Django crea la base `test_classroom_hub`, ejecuta las pruebas y la elimina; **la base `classroom_hub` no se modifica**.

## Estructura

```
classroom-hub/
├── manage.py
├── requirements.txt
├── .env.example
├── config/            # settings.py (DATABASES con django.db.backends.mysql), urls.py
└── pages/
    ├── models.py      # Category y Resource
    ├── admin.py       # CategoryAdmin y ResourceAdmin
    ├── views.py       # home, resources (ORM con select_related), about
    ├── migrations/
    ├── tests.py       # 10 pruebas
    └── templates/pages/
```