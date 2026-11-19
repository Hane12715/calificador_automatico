# Calificador de Labs

Script en Python que ayuda a revisar laboratorios entregados en **Canvas** con un enlace a **GitHub**. Descarga los repositorios, muestra el historial de commits y, opcionalmente, usa una IA local (**Ollama**) para comparar el código con las instrucciones del lab.

> La IA solo genera una revisión de apoyo. La nota final siempre la decide el instructor.

## ¿Qué hace?

1. Obtiene las entregas de una tarea en Canvas.
2. Busca el enlace de GitHub de cada estudiante (en la URL, el texto o los comentarios de la entrega).
3. Detecta repositorios **públicos**, entregas hechas **por comentario** y repos **sin acceso**.
4. Lista los **commits** de cada estudiante con fecha, hora y mensaje, y marca los que se hicieron después de la fecha límite.
5. Clona los repositorios (o los actualiza si ya existen).
6. *(Opcional)* Lee el PDF del lab y genera una revisión por estudiante: requisitos cumplidos, errores y nota sugerida.
7. Genera un reporte con todo.

Todo el código se procesa en tu computadora. La IA corre localmente, así que el código de los estudiantes no se envía a servicios externos.

## Requisitos

- [Python 3](https://www.python.org/downloads/)
- [Git](https://git-scm.com/)
- [Ollama](https://ollama.com/) *(solo si vas a usar la revisión con IA)*
- Rol de instructor o asistente en el curso de Canvas
- Acceso como colaborador a los repositorios de los estudiantes

## Instalación

```bash
git clone https://github.com/Hane12715/calificador_automatico.git
cd calificador
pip install requests pypdf
```

Si vas a usar la IA, descargá el modelo:

```bash
ollama pull qwen2.5-coder:7b
```

## Configuración

Abrí `revisar_tarea_ollama.py` y completá la sección `CONFIG`:

```python
CANVAS_URL = "https://tu-institucion.instructure.com"
CANVAS_TOKEN = "tu_token_de_canvas"
GITHUB_TOKEN = "tu_token_de_github"
COURSE_ID = "12345"
ASSIGNMENT_ID = "67890"

REVISAR_CON_IA = True   # False = solo commits, clonado y reporte
```

### ¿De dónde saco cada dato?

| Dato | Dónde encontrarlo |
|------|-------------------|
| `CANVAS_URL` | La dirección base de tu Canvas, sin `/` al final. |
| `COURSE_ID` y `ASSIGNMENT_ID` | En la URL de la tarea: `.../courses/12345/assignments/67890` |
| `CANVAS_TOKEN` | Canvas → Cuenta → Configuración → **+ Nuevo token de acceso** |
| `GITHUB_TOKEN` | GitHub → Settings → Developer settings → **Tokens (classic)**, con el permiso `repo` |

El token de GitHub debe ser **classic**. Los tokens *fine-grained* no pueden acceder a repositorios de otras personas aunque seas colaborador.

## Uso

1. Poné el PDF con las instrucciones del lab en la carpeta del proyecto, con el nombre `lab.pdf`.
2. Si usás IA, asegurate de que Ollama esté corriendo (abrí `http://localhost:11434` en el navegador y debería decir *Ollama is running*).
3. Ejecutá:

```bash
python revisar_tarea_ollama.py
```

Recomendación: la primera vez corrélo con `REVISAR_CON_IA = False` para verificar que Canvas y GitHub funcionan.

## Resultados

Se crea una carpeta `tarea_<ASSIGNMENT_ID>/` con:

```
tarea_67890/
├── Juan_Perez/               ← repositorio de cada estudiante
├── Maria_Lopez/
├── revisiones/
│   ├── Juan_Perez.md         ← revisión de la IA
│   └── Maria_Lopez.md
└── Log_Tarea_67890.txt       ← reporte general
```

El reporte incluye:

- Repositorios públicos
- Entregas hechas por comentario
- Repositorios sin acceso
- Estudiantes que entregaron sin enlace de GitHub
- Detalle de commits por estudiante (⏰ = después de la fecha límite)

Si lo volvés a correr, los repositorios se actualizan con `git pull` y las revisiones y el reporte se sobrescriben.

## Ajustes útiles

| Variable | Para qué sirve |
|----------|----------------|
| `MODELO` | Modelo de Ollama a usar. Con 8 GB de RAM probá `qwen2.5-coder:3b`. |
| `NUM_CTX` | Cuánto texto puede leer el modelo. Bajalo a `16384` si la computadora se traba. |
| `MAX_CHARS` | Máximo de código por estudiante que se envía a la IA. |

## Limitaciones

- La IA **lee** el código pero no lo ejecuta. Lo que dependa de correr el programa lo marca con ❔.
- Un modelo local pequeño puede equivocarse. Usá la revisión como guía, no como nota final.
- El PDF debe tener texto seleccionable; los PDF escaneados no se pueden leer.
- Si un estudiante cambia su enlace a otro repositorio, borrá su carpeta y volvé a correr el script.

## Seguridad

- **Nunca subas el script con tus tokens puestos.** Dejá los campos vacíos antes de hacer commit.
- No subas las carpetas `tarea_*`: contienen código y nombres de estudiantes.
- Mantené Ollama actualizado y no lo expongas fuera de tu computadora (no cambies `OLLAMA_HOST` a `0.0.0.0`).
- Si alguna vez subiste un token por error, revocalo y generá uno nuevo: queda en el historial de Git aunque lo borres después.
