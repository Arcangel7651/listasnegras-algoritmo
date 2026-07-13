# Buscador PLD-FT — Coincidencias contra listas negras (CNBV)

Aplicación web en Django que compara nombres de clientes contra una lista de personas
señaladas en oficios de la CNBV, usando coincidencia difusa (fuzzy matching) para detectar
alertas aun cuando los nombres tengan errores de captura, abreviaciones o distinto orden.

## Sobre el proyecto

Proyecto académico desarrollado en equipo de 6 personas. Como parte del equipo, colaboré en la
elaboración del Documento de Diseño de Software (SDD) y, sobre todo, en el apartado visual de la
aplicación (interfaz de usuario). La aplicación permite comparar nombres de clientes contra una
lista de personas señaladas en oficios de la CNBV, usando coincidencia difusa para detectar
alertas aun con errores de captura, abreviaciones o distinto orden en los nombres.

## Stack / Tecnologías

- **Lenguaje:** Python
- **Framework web:** Django 5.2.5
- **Base de datos:** SQLite (configuración por defecto del proyecto)
- **Coincidencia de nombres:** rapidfuzz 3.14.5 (ratios y distancia de Levenshtein),
  jellyfish 1.2.1 (Jaro-Winkler y metaphone fonético), Unidecode 1.4.0 (normalización de acentos)
- **Frontend:** plantillas de Django (HTML) con CSS propio; sin framework de JavaScript

## Características principales

- **Búsqueda por texto libre** contra la lista negra, con umbral de coincidencia mínima y
  límite de resultados configurables.
- **Motor de coincidencia difusa combinado** que pondera varias métricas: ratio de similitud,
  token sort/set, distancia de Levenshtein, Jaro-Winkler, comparación fonética (metaphone) y
  esqueleto consonántico. Cada resultado incluye una explicación legible del porqué de la coincidencia.
- **Expansión de abreviaciones de apellidos** (por ejemplo, `HDEZ` → `hernandez`) mediante un
  diccionario almacenado en base de datos con respaldo interno y caché.
- **Revisión por lotes de clientes** contra la lista negra en un rango de fechas (por defecto,
  los últimos 7 días), con conteo de clientes alertados.
- **Comparación de un oficio específico**: contrasta las personas de un oficio de la CNBV contra
  los clientes registrados.
- **Importación de datos** vía comandos de gestión: desde archivos XML de la CNBV
  (`import_cnbv`), desde una base MySQL (`import_mysql`) y carga de datos de ejemplo (`seed_names`).

## Cómo ejecutarlo

Requisitos: Python (compatible con Django 5.2). El código de la aplicación está dentro de la
carpeta `search_name/`.

```bash
# 1. Clonar el repositorio y entrar a la carpeta de la aplicación
cd search_name

# 2. Crear y activar un entorno virtual
python -m venv .venv
# Windows (PowerShell):
.venv\Scripts\Activate.ps1
# Linux / macOS:
source .venv/bin/activate

# 3. Instalar dependencias
pip install -r requirements.txt

# 4. Aplicar migraciones
python manage.py migrate

# 5. (Opcional) Cargar datos de ejemplo para probar el buscador
python manage.py seed_names

# 6. Levantar el servidor de desarrollo
python manage.py runserver
```

La aplicación queda disponible en `http://127.0.0.1:8000/`.

Notas:
- La configuración incluida es de desarrollo (`DEBUG = True` y una `SECRET_KEY` de ejemplo en
  `name_search/settings.py`). Para un despliegue real deben ajustarse estos valores. [POR CONFIRMAR]
- El comando `import_mysql` requiere el paquete `pymysql`, que no está listado en
  `requirements.txt`; instalarlo aparte si se va a usar esa importación (`pip install pymysql`).
- Para importar oficios reales desde XML: `python manage.py import_cnbv <carpeta_con_xml>`.

## Estructura del proyecto

```
search_name/
├── manage.py
├── requirements.txt
├── name_search/            # Configuración del proyecto Django (settings, urls, wsgi/asgi)
└── coincidencias/          # App principal
    ├── matching.py         # Motor de coincidencia difusa y normalización de nombres
    ├── models.py           # Modelos: Oficio, PersonaCNBV, NameRecord, Abreviacion
    ├── views.py            # Vistas: búsqueda, revisión de clientes, comparación de oficios
    ├── forms.py
    ├── templates/          # Plantillas HTML
    ├── management/commands # import_cnbv, import_mysql, seed_names
    └── tests.py            # Pruebas del motor de coincidencia y de las vistas
docs/                       # Documentación (SRS, SDD)
```

Las pruebas se ejecutan con `python manage.py test` desde la carpeta `search_name/`.

## Documentación

La carpeta [docs/](docs/) contiene la documentación técnica del proyecto elaborada por el equipo:

- **Especificación de Requisitos de Software (SRS):** [Software Requirements Specification (SRS).docx](docs/Software%20Requirements%20Specification%20(SRS).docx)
- **Documento de Diseño de Software (SDD):** [Software Design Document (SDD).docx](docs/Software%20Design%20Document%20(SDD).docx)
