# VisualPbAutobuild225 ⚙️

![PowerBuilder](https://img.shields.io/badge/PowerBuilder-2025-orange?style=flat-square&logo=appveyor&logoColor=white)
![Tipo](https://img.shields.io/badge/herramienta-build%20automation-blue?style=flat-square)
![Backend](https://img.shields.io/badge/PbAutobuild225-orquestaci%C3%B3n-9cf?style=flat-square)
![Blog](https://img.shields.io/badge/blog-rsrsystem-FF5722?style=flat-square&logo=blogger&logoColor=white)

> Una capa visual sobre **`PbAutobuild225.exe`** (el compilador por línea de comandos que viene con PowerBuilder 2025) para automatizar y orquestar el *build* de tus proyectos sin pelearte con parámetros a mano.

---

## 📋 ¿Qué es esto?

`PbAutobuild225.exe` es una herramienta estupenda para compilar proyectos PowerBuilder **sin abrir el IDE**, ideal para integraciones continuas y *builds* automatizados. ¿El "pero"? Que se maneja con un archivo **JSON** y un buen puñado de parámetros, y montar todo eso a mano cansa.

**VisualPbAutobuild225** es un programa que pone una **interfaz cómoda por encima**: cargas el JSON de tu proyecto y dejas que la herramienta orqueste la compilación. Soporta los tres sabores de proyecto:

- **Cliente/Servidor (Nativo)**
- **PowerClient**
- **PowerServer**

Y de paso se encarga de cosas tediosas: rellenar plantillas de conexión, gestionar credenciales por proyecto, controlar versiones contra GitHub y descargar los `.ini` parametrizados.

## ✨ Cómo funciona

La lógica vive en estas piezas:

- **`w_main`** → ventana principal: carga el JSON, lee la configuración de `setup.ini` y lanza el *build*.
- **`w_setup`** → editor visual del archivo `setup.ini` (para no editarlo a pelo).
- **`dw_ini` / `dw_iniparams`** → manejan el `.ini` como un DataWindow, con DropDownDataWindow de ayuda para elegir parámetros.
- **`n_cst_functions`** → utilidades: lectura de parámetros (`of_profilestring`, con *fallback* al apartado `[Setup]`), descarga de ficheros desde GitHub (`of_download_file`)…
- **`n_cst_security`** / **`u_json`** → credenciales y parseo del JSON del proyecto.
- **`topwiz.pbl`** (`n_osversion`, `n_runandwait`) → control de versión de la app y ejecución de procesos externos esperando su fin.

El flujo: lee el JSON → resuelve parámetros desde `setup.ini` (apartado del proyecto, y si falta, hereda del `[Setup]` común) → prepara plantillas e `.ini` → invoca `PbAutobuild225.exe` → limpia al terminar.

## 🛠️ Requisitos

- **PowerBuilder 2025** con **`PbAutobuild225.exe`** instalado (indica su ruta en `PbAutobuildPath`).
- **Windows 10/11**.
- Para control de versiones / credenciales remotas: una cuenta de **GitHub** y su *token* personal.

## ▶️ Cómo probarlo

1. Clona el repositorio (viene **en modo solución**).
2. Abre `vpbautobuild.pbsln` desde el IDE de PowerBuilder (o lanza `vpbautobuild.exe`).
3. Configura el `setup.ini` (a mano o con `w_setup`) — ver referencia abajo.
4. Carga el JSON de tu proyecto y lanza la compilación.

🎥 **Vídeo demostrativo:** <https://youtu.be/pruqqhBwN2Q>

## 🧩 Referencia de configuración (`setup.ini`)

> 💡 Truco: lo que sea común a todos los proyectos ponlo en `[Setup]`. El programa primero busca en el apartado del proyecto y, si no lo encuentra, recurre a `[Setup]`.

### Apartado `[Setup]`

| Parámetro | Para qué |
|-----------|----------|
| `json` | Recordar el último JSON abierto. |
| `PbAutobuildPath` | Ruta de instalación de `PbAutobuild225.exe`. |
| `PBNativePath` | Ruta del programa compilado en proyectos Nativos. |

### Apartado por proyecto `[projectName.json]`

**Cadena de conexión a BD (Nativos / PowerClient):**
`DBMS`, `LogPass`, `ServerName`, `LogId`, `AutoCommit`, `DBParm`

**Archivo `.ini` de cada proyecto:**

| Parámetro | Para qué |
|-----------|----------|
| `IniFile` | Nombre del `.ini` donde guardar los parámetros de conexión (TODOS los proyectos). |
| `IniConnectionKey` | Sección donde guardar la cadena de conexión (Nativos / PowerClient). |
| `IniUsersKey` | Sección donde guardar las credenciales (PowerServer). |
| `IniTokenKey` | Sección donde guardar la URL del Token (PowerServer). |
| `UserName` / `UserPass` | Usuario y contraseña en la plantilla JWT (PowerServer). |

**Cuenta de Git** (si `version_control='S'` o el proyecto usa `IniFile` para credenciales):
`version_control`, `GitHubProfileName`, `ProfileVisibility` (`Private`/`Public`), `PersonalToken`, `GitHubRepository`, `GitBranch` (`main`/`master`), `Pbl`, `filename` (nombre del `*.srj`).

**Exclusivos de PowerServer:**
`PowerServerPath` (ruta física del sitio web de PowerServer en el servidor).

**Reclamos adicionales de la plantilla JWT (PowerServer):**
`Scope`, `Name`, `GivenName`, `FamilyName`, `WebSite`, `Email`, `EmailVerified`

## 🗒️ Historial de versiones

**Release 3 — 13-02-2023**
- `w_setup` para configurar cómodamente el `setup.ini`.
- `dw_ini` para manejar el `.ini` como DataWindow y `dw_iniparams` como DropDownDataWindow de ayuda.
- Objeto `n_osversion` (Topwiz) para controlar la versión de la aplicación.
- `of_profilestring` en `n_cst_functions`: si un parámetro no está en su apartado, lo busca en `[Setup]`.
- Variables de instancia compartidas con `w_setup`; nuevas `is_JsonFile`/`is_JsonPath`.
- Contempla que no haya `.ini` configurado para credenciales (PowerServer) o cadenas de conexión (resto).

**Release 2 — 07-02-2023**
- Archivos `.ini` parametrizados por proyecto (sin suponer `CloudSetting.ini`/`Setting.ini`).
- Descarga del `.ini` parametrizado desde GitHub y relleno dinámico desde `setup.ini`.
- Refactor de descarga (`wf_download_version_control`, `of_download_file`, `wf_download_inifile`).
- `w_build` ahora elimina los `.ini` descargados al terminar.

**Release 1 — 27-01-2023**
- Primera versión: carga un JSON de proyecto Cliente/Servidor, PowerClient o PowerServer y facilita el uso de `PbAutobuild225.exe`.

## 🔗 Repo PowerBuilder

Tenéis el ejemplo publicado en modo solución aquí:
👉 <https://github.com/rasanfe/vpbautobuild>

---

> ¡Nos vemos en el próximo artículo! Y recuerda: en PowerBuilder, los límites solo están en nuestra imaginación. 🚀

📨 **Blog:** <https://rsrsystem.blogspot.com/>
