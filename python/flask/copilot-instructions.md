# Copilot Instructions

> **Importante:** Las instrucciones de este archivo deben ser seguidas por GitHub Copilot al generar codigo o sugerencias para este repositorio. Estas instrucciones proporcionan contexto sobre el proyecto, las tecnologias utilizadas y las mejores practicas a seguir. Ademas estas deben respetersa sin importar el agente o el entorno en el que se utilice Copilot.

## <Nombre del Proyecto> - Project Overview
<Descripcion del Proyecto>

## Stack Tecnologico
- Lenguaje de Programacion: Python
- Framework Web: Flask
- Base de Datos: MariaDB
- ORM: SQLAlchemy
- Migraciones de Base de Datos: Flask-Migrate
- Frontend: HTML, CSS, JavaScript, Bootstrap, Jinja3
- Control de Versiones: Git

### Root Folders
- `/app`: Contiene el codigo fuente principal de la aplicacion Flask, incluyendo rutas, modelos y vistas.
- `/app/controllers`: Controladores que manejan la logica de las rutas y las solicitudes HTTP, ya que se usara Flask usalmente estos seran los blueprints.
- `/ap/services`: Logica de negocio y servicios de aplicacion que interactuan y son llamados con los modelos y controladores.
- `/app/models`: Modelos de datos que representan las tablas de la base de datos y las relaciones entre ellas usando SQLAlchemy.
- `/app/templates`: Plantillas HTML que definen la estructura de las paginas web renderizadas por Flask.
- `/app/static`: Archivos estaticos en la cuales estaran divididos por carpetas de CSS, JavaScript (js), img, fonts, etc.
- `/app/helpers`: Funciones y utilidades comunes que pueden ser usadas en diferentes partes del proyecto.
- `/migrations`: Scripts de migracion de base de datos gestionados por Flask-Migrate para versionar y aplicar cambios en el esquema de la base de datos.
- `/docs`: Documentacion del proyecto, incluyendo especificaciones, guias y cualquier otro recurso relevante.
- `/logs`: Archivos de registro generados por la aplicacion para monitorear errores y eventos importantes.
- `/tests`: Pruebas unitarias e integracion para asegurar la calidad y funcionalidad del codigo.

## 1. Arquitectura y responsabilidades
- **Controladores (`/app/controllers`)**: *Routing y orquestación ligera*. Reciben/validan entrada, llaman a **servicios de aplicación** y devuelven respuesta. **No** acceden directamente a la DB.
- **Modelos (`/app/models`)**: *Mapeo ORM (SQLAlchemy)* y **reglas de dominio simples** (invariantes, validaciones básicas). **No** incluyen lógica procedural extensa ni llamadas externas. Se debe procurar tener modelos bases y usar herencia para modelos especializados.
- **Servicios / Aplicacion (`/app/services`)**: Contienen la lógica de negocio y servicios de aplicación que interactúan y son llamados por los modelos y controladores. Aquí es donde se implementan las reglas de negocio complejas, orquestación de múltiples modelos y cualquier lógica que no encaje en controladores o modelos. Pueden interactuar con APIs externas, manejar transacciones, etc.
- **Templates (`/app/templates`)**: Plantillas HTML que definen la estructura de las páginas web renderizadas por Flask. Usar Jinja3 para lógica de presentación mínima (bucles, condicionales simples).
- **Static (`/app/static`)**: Archivos estáticos organizados en subcarpetas por tipo (CSS, JS, imágenes, fuentes, etc.) para una gestión clara y eficiente. 
<br><br>

> **Ejemplo de flujo de una solicitud:**
> 1. El usuario hace una solicitud HTTP a una ruta gestionada por un `controlador`.
> 2. El **controlador** valida la entrada y llama a un `Servicio / Aplicacion` para procesar la lógica de negocio.
> 3. El `Servicio / Aplicacion` interactúa con uno o más `modelos` para recuperar o modificar datos en la base de datos.
> 4. Los `modelos` aplican reglas de dominio simples y devuelven datos al `Servicio / Aplicacion`.
> 5. El `Servicio / Aplicacion` devuelve los resultados al `controlador`.
> 6. El `controlador` renderiza una plantilla HTML usando `templates` y devuelve la respuesta al usuario.

## 2. Convenciones de estructura de código
- Los nombres de archivos deben ser descriptivos y usar snake_case para Python (ej. user_controller.py, data_service.py).
- Los nombres de clases deben usar PascalCase (ej. UserController, DataService).
-Los nombres de los modelos deben ser singulares y en PascalCase (ej. User, Product), asi como los nombres de las tablas en plural y en snake_case (ej. users, products).
- Las funciones y métodos deben usar verbos descriptivos en snake_case (ej. get_user_by_id, calculate_total).
- Se trabajara con archivos bases para modelos que seran heredados por modelos mas especializados, asi como archivos bases para otras funciones comunes que se repitan en el codigo como por ejemplo css, vistas, js, funciones, modulos enteros. Permitiendo que en muchas vistas tengamos el mismo template pero reciba distinta info como una API.
- El como se dividira el template de cada vista del proyecto sera por medio del archivo base.html el cual tendra al principio el header.html en medio el template para llenar con las distintas vistas del proyecto y al final el footer.html. Esto con el fin de tener una estructura mas limpia y facil de manejar.
- El proyecto tienen un archivo .env el cual contendra las variables de entorno necesarias para el correcto funcionamiento del proyecto, como las credenciales de la base de datos, claves secretas, configuraciones de debug, entre otras. Este archivo no debe ser subido al control de versiones por razones de seguridad.
- Cada Modulo debe ser registrado como un blueprint en Flask para mantener una estructura modular y facilitar el mantenimiento del codigo.

## 3. Coding guidelines
- **Identacion:** usa tabs o 2 espacios de indentacion, pero se consistente en todo el proyecto.
- **Imports:** organiza los imports en tres secciones: librerias estandar, librerias de terceros y modulos locales. Usa lineas en blanco para separar estas secciones. Estos deben ir al inicio y deben no repetirse a lo largo del codigo
- **Comentarios:** Los comentarios en cada funcion, clase o metodo deben seguir el formato de docstring de Python (triple comillas) y describir claramente el propósito, los parámetros y el valor de retorno, por ejemplo.
```python
def get_user_by_id(user_id: int) -> User:
    """
    Recupera un usuario por su ID.
    :param user_id: ID del usuario a recuperar.
    :return: Instancia del modelo User.
    """
    pass
```
- **Funciones y Métodos:** deben ser cortos y enfocados en una sola tarea. Si una función/método excede las 80-100 líneas, considera dividirla en funciones/métodos más pequeños. **Si una funcion se puede usar en diferentes partes del codigo esta debe ser un helper en `app/helpers/` y asi ser llamada desde cualquier parte del proyecto.


## 4. Migraciones y base de datos

*Evita drift entre modelos y esquema real.*

- **Herramienta:** Flask‑Migrate (Alembic).
- **Reglas:**
  - Cada cambio en modelos ➜ **nueva migración** con mensaje claro.
  - **No** alteres la DB manualmente en producción.
  - Pruebas deben correr con migraciones aplicadas (DB de test).
  - Para modelos que ocupen herencia por lo general para los campos id, created_at, updated_at, deleted_at, estos deben ordenarse en la migracion de la siguiente forma:
    - Primero el campo id
    - Luego campos del modelo
    - Finalmente los campos created_at, updated_at, deleted_at
- **Comandos tipo:**
  - `flask db migrate -m "añade tabla scripts"`
  - `flask db upgrade`
  - `flask db downgrade` (solo en dev)

  ## 5. Testing y calidad
- **Tipos de tests:** unitarios (modelos, servicios), integracion (rutas, controladores).
- **Framework:** `pytest`.
- **Estructura:** tests/conftest.py # fixtures comunes (app, client, db), test_usuarios_service.py, test_scripts_repo.py, test_routes_usuarios.py
- **Cobertura mínima:** 70% en `services`.
- **Fixtures:** Cliente Flask de test, DB en memoria / contenedor MariaDB de test.
- **Nomenclatura:** `/tests/<modulo>/<comportamiento>.py`.
- **Reglas:** No ejecutar tests si falla lint/typing (CI corta antes).

## 6. Logging y Observabilidad

*Logs claros para debugging, monitoreo y trazas reproducibles.*

- **Librería:** módulo `logging` de Python.
- **Logger base:** configuración en `app/__init__.py` para toda la app.
- **Niveles:** DEBUG (desarrollo), INFO (eventos normales), WARNING (eventos inusuales), ERROR (errores manejados), CRITICAL (errores graves).
- **Formato:** incluir timestamp, nivel, módulo, mensaje.
- **Manejo de errores:** loggear excepciones con stack trace usando `logger.exception()`.
- **Archivos de log:** rotación diaria o por tamaño (ej. `logging.handlers.RotatingFileHandler`).
- **Estructura de logs:** JSON para facilitar análisis y parsing por herramientas externas.
- **Buenas prácticas:** no loggear datos sensibles (contraseñas, tokens), usar mensajes claros y consistentes, incluir IDs de correlación para trazabilidad.

## 7. Restricciones explícitas (qué NO debe hacer Copilot)

*Limita comportamientos que generan deuda técnica o riesgos.*

- **NO** acceder a la DB desde controllers; usar **services**.
- **NO** escribir SQL crudo salvo necesidad y **justificación**. Preferir SQLAlchemy.
- **NO** poner lógica de negocio en templates.
- **NO** hardcodear rutas, credenciales ni constantes de entorno.
- **NO** duplicar funciones/helpers si ya existen en `app/base` o `app/services`.
- **NO** mezclar responsabilidades: cada módulo con un propósito claro.

## 8. Documentacion y Readme
- Mantener actualizado el README.md con instrucciones de instalación, configuración, ejecución y testing.
- Dentro de `docs/`, debes tener un unico archivo que sea el manual tecnico del proyecto este se debe ir actualizando conforme se vayan agregando nuevas funcionalidades o cambios importantes en la arquitectura del proyecto.
