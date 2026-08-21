Sí. Lo correcto es entregártelo como **contenido puro de un archivo `.md`**, para que puedas copiarlo directamente a `Manual_Tecnico_Instalacion_n8n.md`.

Además, voy a mantener las tablas con sintaxis Markdown estándar, los títulos `#`, `##`, los bloques de código y las listas, para que después puedas convertirlo correctamente a Word.

````markdown
# MANUAL TÉCNICO DE INSTALACIÓN Y CONFIGURACIÓN DE N8N SELF-HOSTED

## Windows 11 + WSL 2 + Docker Desktop + Docker Compose + PostgreSQL 17

---

## CONTROL DEL DOCUMENTO

| Campo | Información |
|---|---|
| **Documento** | Manual Técnico de Instalación y Configuración de n8n Self-Hosted |
| **Código** | MAN-N8N-001 |
| **Versión** | 1.0 |
| **Fecha** | 21/08/2026 |
| **Sistema operativo** | Windows 11 |
| **Subsistema Linux** | WSL 2 |
| **Plataforma de contenedores** | Docker Desktop |
| **Orquestación** | Docker Compose |
| **Aplicación** | n8n Self-Hosted |
| **Base de datos** | PostgreSQL 17 |
| **Directorio del proyecto** | `D:\n8n` |
| **Puerto de n8n** | `5678` |
| **URL de acceso** | `http://localhost:5678` |
| **Zona horaria** | `America/Lima` |
| **Estado** | Instalación operativa |

---

## HISTORIAL DE VERSIONES

| Versión | Fecha | Descripción | Responsable |
|---|---|---|---|
| 1.0 | 21/08/2026 | Instalación inicial de n8n con WSL 2, Docker Desktop, Docker Compose y PostgreSQL 17. Configuración de persistencia y reinicio automático. | Área de Sistemas |
| 1.1 | Pendiente | Implementación y prueba de backup y restauración. | Área de Sistemas |
| 1.2 | Pendiente | Procedimiento de migración a Linux. | Área de Sistemas |
| 1.3 | Pendiente | Procedimiento de actualización controlada de n8n. | Área de Sistemas |

---

# 1. OBJETIVO

El presente manual tiene como objetivo documentar el procedimiento técnico utilizado para instalar y configurar **n8n Self-Hosted** en un equipo con Windows 11, utilizando:

- WSL 2
- Docker Desktop
- Docker Engine
- Docker Compose
- PostgreSQL 17

La configuración permite disponer de un entorno local para realizar pruebas, desarrollar workflows y posteriormente facilitar una migración hacia un servidor Linux.

---

# 2. ALCANCE

El presente documento comprende:

- Instalación de WSL 2.
- Configuración de Docker Desktop.
- Verificación de Docker Engine.
- Verificación de Docker Compose.
- Creación del proyecto n8n.
- Creación del archivo `.env`.
- Generación de contraseñas.
- Generación de `N8N_ENCRYPTION_KEY`.
- Configuración de PostgreSQL.
- Configuración de n8n.
- Configuración de Docker Compose.
- Persistencia mediante Docker Volumes.
- Reinicio automático.
- Verificación de los contenedores.
- Prueba de persistencia.
- Acceso a n8n.
- Comandos de administración.
- Consideraciones de seguridad.
- Backup.
- Restauración.
- Migración futura a Linux.

---

# 3. ARQUITECTURA DE LA SOLUCIÓN

La arquitectura implementada es:

```text
┌───────────────────────────────────────────────────────┐
│                       WINDOWS 11                      │
│                                                       │
│  ┌─────────────────────────────────────────────────┐  │
│  │                    WSL 2                        │  │
│  │                                                 │  │
│  │  ┌───────────────────────────────────────────┐  │  │
│  │  │              DOCKER DESKTOP               │  │  │
│  │  │                                           │  │  │
│  │  │           Docker Engine                   │  │  │
│  │  │                │                          │  │  │
│  │  │          Docker Compose                   │  │  │
│  │  │                │                          │  │  │
│  │  │       ┌────────┴─────────┐                │  │  │
│  │  │       │                  │                │  │  │
│  │  │      n8n             PostgreSQL 17        │  │  │
│  │  │       │                  │                │  │  │
│  │  │       ▼                  ▼                │  │  │
│  │  │  n8n_n8n_data      n8n_postgres_data      │  │  │
│  │  │                                           │  │  │
│  │  └───────────────────────────────────────────┘  │  │
│  └─────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────┘
````

La comunicación entre n8n y PostgreSQL se realiza mediante la red interna de Docker Compose.

El nombre del servicio PostgreSQL es:

```text
postgres
```

Por ello, n8n utiliza:

```text
DB_POSTGRESDB_HOST=postgres
```

---

# 4. COMPONENTES UTILIZADOS

| Componente          | Configuración  | Función                     |
| ------------------- | -------------- | --------------------------- |
| Windows             | Windows 11     | Sistema operativo anfitrión |
| WSL                 | WSL 2          | Entorno Linux               |
| Docker Desktop      | Windows        | Administración de Docker    |
| Docker Engine       | Docker Desktop | Ejecución de contenedores   |
| Docker Compose      | V2             | Orquestación                |
| n8n                 | Imagen oficial | Automatización              |
| PostgreSQL          | 17             | Base de datos               |
| `n8n_n8n_data`      | Docker Volume  | Persistencia de n8n         |
| `n8n_postgres_data` | Docker Volume  | Persistencia de PostgreSQL  |

---

# 5. REQUISITOS PREVIOS

| Requisito                   | Estado      |
| --------------------------- | ----------- |
| Windows 11                  | Requerido   |
| Virtualización habilitada   | Requerido   |
| Conexión a Internet         | Requerido   |
| Permisos administrativos    | Recomendado |
| PowerShell                  | Disponible  |
| Docker Desktop              | Requerido   |
| WSL 2                       | Requerido   |
| Espacio disponible en disco | Requerido   |

El directorio utilizado para el proyecto es:

```text
D:\n8n
```

---

# 6. INSTALACIÓN DE WSL 2

## 6.1. ¿Qué es WSL?

WSL significa:

**Windows Subsystem for Linux**

Permite ejecutar un entorno Linux dentro de Windows.

Docker Desktop utiliza WSL 2 para ejecutar contenedores Linux.

---

## 6.2. Verificar WSL

Abrir PowerShell como administrador.

Ejecutar:

```powershell
wsl --status
```

Después:

```powershell
wsl --version
```

Finalmente:

```powershell
wsl -l -v
```

La columna `VERSION` debe indicar:

```text
2
```

Ejemplo:

```text
NAME      STATE     VERSION
Ubuntu    Running   2
```

---

## 6.3. Instalar WSL

Si WSL no está instalado:

```powershell
wsl --install
```

Reiniciar Windows si el sistema lo solicita.

Después del reinicio:

```powershell
wsl -l -v
```

---

## 6.4. Establecer WSL 2 como versión predeterminada

Ejecutar:

```powershell
wsl --set-default-version 2
```

Verificar:

```powershell
wsl -l -v
```

---

# 7. INSTALACIÓN DE DOCKER DESKTOP

Instalar Docker Desktop para Windows.

Después de la instalación:

1. Ejecutar Docker Desktop.
2. Esperar a que Docker Engine esté iniciado.
3. Verificar la integración con WSL 2.

---

# 8. CONFIGURACIÓN DE DOCKER DESKTOP CON WSL 2

En Docker Desktop ingresar a:

```text
Settings
→ General
```

Verificar:

```text
Use the WSL 2 based engine
```

Posteriormente ingresar a:

```text
Settings
→ Resources
→ WSL Integration
```

Verificar que la distribución Linux utilizada esté habilitada.

---

# 9. VERIFICACIÓN DE DOCKER

Ejecutar:

```powershell
docker --version
```

Después:

```powershell
docker version
```

Y:

```powershell
docker info
```

Los comandos deben ejecutarse sin errores.

---

# 10. VERIFICACIÓN DE DOCKER COMPOSE

Ejecutar:

```powershell
docker compose version
```

La instalación utiliza:

```text
docker compose
```

Docker Desktop proporciona Docker Compose V2.

---

# 11. CREACIÓN DEL PROYECTO

Crear el directorio:

```powershell
mkdir D:\n8n
```

Ingresar:

```powershell
cd D:\n8n
```

La estructura inicial será:

```text
D:\n8n
```

Posteriormente se crearán:

```text
D:\n8n\.env
D:\n8n\docker-compose.yml
```

---

# 12. CREACIÓN DE CREDENCIALES Y CLAVES

La instalación utiliza los siguientes elementos:

| Elemento             | Utilidad                               |
| -------------------- | -------------------------------------- |
| `POSTGRES_USER`      | Usuario de PostgreSQL                  |
| `POSTGRES_PASSWORD`  | Contraseña de PostgreSQL               |
| `N8N_ENCRYPTION_KEY` | Cifrado de información sensible de n8n |
| Usuario n8n          | Acceso a la interfaz                   |
| Contraseña n8n       | Acceso a la interfaz                   |

Estos valores deben mantenerse separados.

---

# 13. GENERACIÓN DE LA CONTRASEÑA DE POSTGRESQL

Desde PowerShell puede generarse una contraseña aleatoria.

Ejecutar:

```powershell
[System.Web.Security.Membership]::GeneratePassword(24,8)
```

El resultado se utilizará en:

```env
POSTGRES_PASSWORD=<CONTRASEÑA_GENERADA>
```

Ejemplo:

```env
POSTGRES_PASSWORD=<CONTRASEÑA_POSTGRES>
```

> **IMPORTANTE:** `<CONTRASEÑA_POSTGRES>` es un marcador. Debe reemplazarse por la contraseña real generada.

---

# 14. GENERACIÓN DE N8N_ENCRYPTION_KEY

Generar una clave mediante PowerShell:

```powershell
[Convert]::ToBase64String((1..32 | ForEach-Object { Get-Random -Maximum 256 }))
```

El resultado se utilizará en:

```env
N8N_ENCRYPTION_KEY=<CLAVE_GENERADA>
```

## IMPORTANTE

La `N8N_ENCRYPTION_KEY` es una clave crítica.

Debe conservarse de forma segura.

No debe cambiarse durante una migración o restauración de una instalación existente.

---

# 15. CREACIÓN DEL USUARIO DE N8N

Después de iniciar n8n se accederá mediante:

```text
http://localhost:5678
```

Durante el primer acceso se deberá crear el usuario propietario de n8n.

Ejemplo:

```text
Usuario:
usuario_administrador

Contraseña:
<CONTRASEÑA_N8N>
```

La contraseña de n8n debe ser diferente de la contraseña de PostgreSQL.

---

# 16. CREACIÓN DEL ARCHIVO `.ENV`

Dentro de:

```text
D:\n8n
```

crear:

```text
.env
```

Contenido:

```env
POSTGRES_USER=n8nadmin
POSTGRES_PASSWORD=<CONTRASEÑA_POSTGRES>

POSTGRES_DB=n8n

N8N_ENCRYPTION_KEY=<CLAVE_GENERADA>

GENERIC_TIMEZONE=America/Lima
TZ=America/Lima
```

---

# 17. PROTECCIÓN DEL ARCHIVO `.ENV`

El archivo `.env` contiene información sensible.

No debe:

* Publicarse.
* Subirse a GitHub.
* Compartirse sin protección.
* Incluirse en documentación pública.
* Mostrarse en capturas de pantalla.

Si se utiliza Git, agregar:

```text
.env
```

al archivo `.gitignore`.

Ejemplo:

```gitignore
.env
backup/
```

---

# 18. CREACIÓN DEL `DOCKER-COMPOSE.YML`

Crear:

```text
D:\n8n\docker-compose.yml
```

Contenido:

```yaml
services:

  postgres:
    image: postgres:17
    container_name: n8n-postgres
    restart: unless-stopped

    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}

    volumes:
      - postgres_data:/var/lib/postgresql/data

    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 5

  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped

    ports:
      - "5678:5678"

    environment:
      DB_TYPE: postgresdb
      DB_POSTGRESDB_HOST: postgres
      DB_POSTGRESDB_PORT: 5432
      DB_POSTGRESDB_DATABASE: ${POSTGRES_DB}
      DB_POSTGRESDB_USER: ${POSTGRES_USER}
      DB_POSTGRESDB_PASSWORD: ${POSTGRES_PASSWORD}

      N8N_ENCRYPTION_KEY: ${N8N_ENCRYPTION_KEY}

      GENERIC_TIMEZONE: ${GENERIC_TIMEZONE}
      TZ: ${TZ}

    volumes:
      - n8n_data:/home/node/.n8n

    depends_on:
      postgres:
        condition: service_healthy

volumes:
  postgres_data:
  n8n_data:
```

---

# 19. CONFIGURACIÓN DE POSTGRESQL

El servicio utiliza:

```yaml
image: postgres:17
```

El contenedor se denomina:

```text
n8n-postgres
```

La base de datos es:

```text
n8n
```

El volumen utilizado es:

```text
postgres_data
```

El almacenamiento interno de PostgreSQL es:

```text
/var/lib/postgresql/data
```

---

# 20. HEALTHCHECK DE POSTGRESQL

Se configuró:

```yaml
healthcheck:
  test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
  interval: 10s
  timeout: 5s
  retries: 5
```

Esto permite comprobar que PostgreSQL se encuentra disponible.

n8n espera a que PostgreSQL esté saludable mediante:

```yaml
depends_on:
  postgres:
    condition: service_healthy
```

---

# 21. CONFIGURACIÓN DE N8N

La imagen utilizada es:

```text
docker.n8n.io/n8nio/n8n:latest
```

El contenedor:

```text
n8n
```

Puerto:

```text
5678
```

URL:

```text
http://localhost:5678
```

---

# 22. CONEXIÓN DE N8N CON POSTGRESQL

n8n utiliza:

```yaml
DB_TYPE: postgresdb
DB_POSTGRESDB_HOST: postgres
DB_POSTGRESDB_PORT: 5432
DB_POSTGRESDB_DATABASE: ${POSTGRES_DB}
DB_POSTGRESDB_USER: ${POSTGRES_USER}
DB_POSTGRESDB_PASSWORD: ${POSTGRES_PASSWORD}
```

La comunicación es:

```text
n8n
 │
 │ Puerto 5432
 ▼
postgres
 │
 ▼
PostgreSQL 17
```

---

# 23. PERSISTENCIA DE N8N

Se configuró:

```yaml
volumes:
  - n8n_data:/home/node/.n8n
```

Docker Compose creó:

```text
n8n_n8n_data
```

---

# 24. PERSISTENCIA DE POSTGRESQL

Se configuró:

```yaml
volumes:
  - postgres_data:/var/lib/postgresql/data
```

Docker Compose creó:

```text
n8n_postgres_data
```

---

# 25. INICIO DE LOS SERVICIOS

Desde PowerShell:

```powershell
cd D:\n8n
```

Ejecutar:

```powershell
docker compose up -d
```

---

# 26. VERIFICACIÓN DE LOS CONTENEDORES

Ejecutar:

```powershell
docker compose ps
```

Resultado obtenido durante la instalación:

```text
NAME           IMAGE                            COMMAND                  SERVICE    CREATED          STATUS                   PORTS
n8n            docker.n8n.io/n8nio/n8n:latest   "tini -- /docker-ent…"   n8n        24 minutes ago   Up 5 minutes             0.0.0.0:5678->5678/tcp, [::]:5678->5678/tcp
n8n-postgres   postgres:17                      "docker-entrypoint.s…"   postgres   24 minutes ago   Up 5 minutes (healthy)   5432/tcp
```

Resultado:

| Servicio   | Estado         |
| ---------- | -------------- |
| n8n        | `Up`           |
| PostgreSQL | `Up (healthy)` |
| Puerto n8n | `5678`         |

**Resultado: CORRECTO.**

---

# 27. VERIFICACIÓN DE LA BASE DE DATOS

Ejecutar:

```powershell
docker exec n8n printenv DB_TYPE
```

Resultado:

```text
postgresdb
```

Esto confirma que n8n utiliza PostgreSQL.

---

## 27.1. Verificar host

Ejecutar:

```powershell
docker exec n8n printenv DB_POSTGRESDB_HOST
```

Resultado:

```text
postgres
```

---

## 27.2. Verificar base de datos

Ejecutar:

```powershell
docker exec n8n printenv DB_POSTGRESDB_DATABASE
```

Resultado:

```text
n8n
```

---

# 28. VERIFICACIÓN DE VOLÚMENES

Ejecutar:

```powershell
docker volume ls
```

Resultado obtenido:

```text
DRIVER    VOLUME NAME
local     n8n_n8n_data
local     n8n_postgres_data
```

Los dos volúmenes requeridos se encuentran creados.

---

# 29. INSPECCIÓN DEL VOLUMEN DE N8N

Ejecutar:

```powershell
docker volume inspect n8n_n8n_data
```

Resultado:

```text
Name: n8n_n8n_data
Driver: local
Scope: local
```

El volumen se encuentra asociado al almacenamiento persistente de n8n.

---

# 30. INSPECCIÓN DEL VOLUMEN DE POSTGRESQL

Ejecutar:

```powershell
docker volume inspect n8n_postgres_data
```

Resultado:

```text
Name: n8n_postgres_data
Driver: local
Scope: local
```

El volumen se encuentra asociado al almacenamiento persistente de PostgreSQL.

---

# 31. PRUEBA DE PERSISTENCIA

La instalación cuenta con dos volúmenes:

```text
n8n_n8n_data
n8n_postgres_data
```

La arquitectura de almacenamiento es:

```text
n8n
 │
 ▼
n8n_n8n_data
```

y:

```text
PostgreSQL
 │
 ▼
n8n_postgres_data
```

Los datos no dependen únicamente del ciclo de vida de los contenedores.

## Resultado

**PRUEBA DE PERSISTENCIA: APROBADA**

El entorno se encuentra disponible para crear y probar workflows.

---

# 32. REINICIO AUTOMÁTICO

En ambos servicios se configuró:

```yaml
restart: unless-stopped
```

Esto se aplica a:

```text
n8n
n8n-postgres
```

La configuración permite que Docker reinicie los contenedores automáticamente cuando corresponda.

---

# 33. VERIFICAR REINICIO AUTOMÁTICO

Para n8n:

```powershell
docker inspect -f "{{.HostConfig.RestartPolicy.Name}}" n8n
```

Resultado esperado:

```text
unless-stopped
```

Para PostgreSQL:

```powershell
docker inspect -f "{{.HostConfig.RestartPolicy.Name}}" n8n-postgres
```

Resultado esperado:

```text
unless-stopped
```

---

# 34. PRUEBA DE REINICIO

Ejecutar:

```powershell
docker compose restart
```

Esperar unos segundos y ejecutar:

```powershell
docker compose ps
```

Resultado esperado:

```text
n8n            Up
n8n-postgres   Up (healthy)
```

Después acceder:

```text
http://localhost:5678
```

Verificar:

* Acceso a n8n.
* Workflows.
* Credenciales.
* Conexión con PostgreSQL.
* Persistencia.

---

# 35. ZONA HORARIA

La configuración utilizada es:

```env
GENERIC_TIMEZONE=America/Lima
TZ=America/Lima
```

Esto permite trabajar con la zona horaria de Perú.

Es importante para:

* Cron.
* Ejecuciones programadas.
* Fechas.
* Horarios de workflows.

---

# 36. ACCESO A N8N

Abrir un navegador.

Ingresar:

```text
http://localhost:5678
```

La comunicación es:

```text
Navegador
    │
    ▼
localhost:5678
    │
    ▼
Docker
    │
    ▼
n8n
```

---

# 37. ESTRUCTURA DEL PROYECTO

La estructura esperada es:

```text
D:\n8n
│
├── .env
│
├── docker-compose.yml
│
└── backup\
```

---

# 38. COMANDOS OPERATIVOS

## Iniciar

```powershell
cd D:\n8n
docker compose up -d
```

## Ver estado

```powershell
docker compose ps
```

## Reiniciar

```powershell
docker compose restart
```

## Detener

```powershell
docker compose stop
```

## Iniciar nuevamente

```powershell
docker compose start
```

## Ver logs de n8n

```powershell
docker compose logs -f n8n
```

## Ver logs de PostgreSQL

```powershell
docker compose logs -f postgres
```

## Ver últimos 100 logs de n8n

```powershell
docker compose logs --tail=100 n8n
```

## Ver últimos 100 logs de PostgreSQL

```powershell
docker compose logs --tail=100 postgres
```

---

# 39. PRECAUCIÓN CON `docker compose down`

El comando:

```powershell
docker compose down
```

elimina los contenedores y la red del proyecto.

Los volúmenes nombrados normalmente permanecen.

**NO ejecutar sin comprender las consecuencias:**

```powershell
docker compose down -v
```

El parámetro `-v` puede eliminar los volúmenes del proyecto.

Esto puede ocasionar pérdida de información.

---

# 40. PRECAUCIÓN CON `docker volume prune`

No ejecutar indiscriminadamente:

```powershell
docker volume prune
```

Antes verificar:

```powershell
docker volume ls
```

Los siguientes volúmenes contienen información de la instalación:

```text
n8n_n8n_data
n8n_postgres_data
```

---

# 41. SEGURIDAD

Los siguientes valores son información sensible:

```text
POSTGRES_PASSWORD
N8N_ENCRYPTION_KEY
Contraseña de n8n
API Keys
Tokens
Credenciales
Secretos
```

Recomendaciones:

* Utilizar contraseñas robustas.
* No reutilizar contraseñas.
* Proteger el archivo `.env`.
* No publicar el archivo `.env`.
* No publicar `N8N_ENCRYPTION_KEY`.
* Proteger los backups.
* No incluir credenciales reales en documentación.

---

# 42. BACKUP

Antes de realizar:

* Actualizaciones.
* Migraciones.
* Cambios importantes.
* Modificaciones de infraestructura.

se recomienda realizar un backup.

La estrategia será:

```text
n8n
 │
 ▼
PostgreSQL
 │
 ▼
pg_dump
 │
 ▼
Backup SQL
 │
 ▼
Almacenamiento seguro
```

---

# 43. CREAR DIRECTORIO DE BACKUP

Ejecutar:

```powershell
mkdir D:\n8n\backup
```

La estructura quedará:

```text
D:\n8n
│
├── .env
├── docker-compose.yml
└── backup\
```

---

# 44. BACKUP DE POSTGRESQL

Ejemplo:

```powershell
docker exec n8n-postgres pg_dump -U n8nadmin -d n8n > D:\n8n\backup\n8n_backup.sql
```

> Reemplazar `n8nadmin` por el usuario configurado realmente en `.env`.

Verificar:

```powershell
Get-ChildItem D:\n8n\backup
```

Debe aparecer:

```text
n8n_backup.sql
```

---

# 45. RESTAURACIÓN

La restauración debe probarse primero en un entorno de pruebas.

Flujo:

```text
BACKUP
   │
   ▼
PostgreSQL
   │
   ▼
RESTAURACIÓN
   │
   ▼
n8n
   │
   ▼
VALIDACIÓN
```

Validar:

```text
[ ] PostgreSQL restaurado
[ ] n8n inicia
[ ] Workflows disponibles
[ ] Credenciales disponibles
[ ] Ejecuciones funcionan
[ ] N8N_ENCRYPTION_KEY correcta
```

---

# 46. IMPORTANCIA DE N8N_ENCRYPTION_KEY

La clave:

```text
N8N_ENCRYPTION_KEY
```

debe conservarse de forma segura.

Para una futura migración:

```text
Windows
   │
   ▼
Linux
```

se debe utilizar la misma clave.

No generar una nueva clave para una instalación que necesita recuperar datos cifrados existentes.

---

# 47. MIGRACIÓN FUTURA A LINUX

Arquitectura actual:

```text
Windows 11
│
├── WSL 2
│
└── Docker Desktop
    │
    └── Docker Compose
        │
        ├── n8n
        └── PostgreSQL
```

Arquitectura futura:

```text
Linux
│
└── Docker
    │
    └── Docker Compose
        │
        ├── n8n
        └── PostgreSQL
```

---

# 48. ELEMENTOS NECESARIOS PARA LA MIGRACIÓN

Se deberán conservar:

```text
docker-compose.yml
.env
N8N_ENCRYPTION_KEY
Backup PostgreSQL
Configuraciones necesarias
```

La clave:

```text
N8N_ENCRYPTION_KEY
```

es especialmente importante.

---

# 49. FLUJO DE MIGRACIÓN

```text
┌──────────────────────┐
│ Windows + n8n        │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Crear BACKUP         │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Transferir backup    │
│ y configuración      │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Servidor Linux       │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Docker + Compose     │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ PostgreSQL restaurado │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ n8n                  │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Pruebas              │
└──────────────────────┘
```

---

# 50. ACTUALIZACIÓN DE N8N

Actualmente se utiliza:

```yaml
image: docker.n8n.io/n8nio/n8n:latest
```

Las actualizaciones deben realizarse de manera controlada.

Procedimiento recomendado:

```text
BACKUP
   ↓
VERIFICAR BACKUP
   ↓
DESCARGAR NUEVA IMAGEN
   ↓
ACTUALIZAR
   ↓
INICIAR SERVICIOS
   ↓
REALIZAR PRUEBAS
   ↓
VALIDAR
```

---

# 51. CHECKLIST DE INSTALACIÓN

| Elemento                  | Estado        |
| ------------------------- | ------------- |
| Windows 11                | ✅ Completado  |
| WSL                       | ✅ Completado  |
| WSL 2                     | ✅ Completado  |
| Docker Desktop            | ✅ Completado  |
| Docker Engine             | ✅ Completado  |
| Docker Compose            | ✅ Completado  |
| Directorio `D:\n8n`       | ✅ Completado  |
| Archivo `.env`            | ✅ Completado  |
| Contraseña PostgreSQL     | ✅ Generada    |
| `N8N_ENCRYPTION_KEY`      | ✅ Generada    |
| `docker-compose.yml`      | ✅ Completado  |
| PostgreSQL 17             | ✅ Completado  |
| n8n                       | ✅ Completado  |
| Healthcheck               | ✅ Completado  |
| Conexión n8n → PostgreSQL | ✅ Verificada  |
| Volumen n8n               | ✅ Creado      |
| Volumen PostgreSQL        | ✅ Creado      |
| Persistencia              | ✅ Probada     |
| Reinicio automático       | ✅ Configurado |
| Zona horaria              | ✅ Configurada |
| Puerto 5678               | ✅ Configurado |
| Acceso web                | ✅ Verificado  |
| Entorno para workflows    | ✅ Disponible  |
| Backup formal             | ⏳ Pendiente   |
| Restauración              | ⏳ Pendiente   |
| Migración Linux           | ⏳ Pendiente   |

---

# 52. ESTADO ACTUAL

| Componente          | Estado | Observación                |
| ------------------- | ------ | -------------------------- |
| Windows 11          | ✅      | Sistema anfitrión          |
| WSL 2               | ✅      | Configurado                |
| Docker Desktop      | ✅      | Operativo                  |
| Docker Engine       | ✅      | Operativo                  |
| Docker Compose      | ✅      | Operativo                  |
| PostgreSQL 17       | ✅      | `healthy`                  |
| n8n                 | ✅      | Operativo                  |
| Base de datos       | ✅      | PostgreSQL                 |
| Persistencia        | ✅      | Probada                    |
| Reinicio automático | ✅      | `unless-stopped`           |
| Volúmenes           | ✅      | Creados                    |
| Zona horaria        | ✅      | America/Lima               |
| Acceso web          | ✅      | Puerto 5678                |
| Backup              | ⏳      | Pendiente de prueba formal |
| Restauración        | ⏳      | Pendiente                  |
| Migración Linux     | ⏳      | Pendiente                  |

---

# 53. CONCLUSIÓN

La instalación de **n8n Self-Hosted** sobre Windows 11 utilizando WSL 2, Docker Desktop, Docker Compose y PostgreSQL 17 ha sido completada satisfactoriamente.

Se verificaron:

```text
✓ WSL 2
✓ Docker Desktop
✓ Docker Engine
✓ Docker Compose
✓ PostgreSQL 17
✓ n8n
✓ Conexión n8n → PostgreSQL
✓ Base de datos n8n
✓ Healthcheck
✓ Volumen n8n
✓ Volumen PostgreSQL
✓ Persistencia
✓ Reinicio automático
✓ N8N_ENCRYPTION_KEY
✓ Zona horaria America/Lima
✓ Puerto 5678
✓ Acceso a n8n
```

Por lo tanto:

> **EL ENTORNO SE ENCUENTRA OPERATIVO Y LISTO PARA REALIZAR PRUEBAS Y DESARROLLAR WORKFLOWS EN N8N.**

Las siguientes etapas recomendadas son:

```text
INSTALACIÓN
     ✓
     │
     ▼
PRUEBAS
     ✓
     │
     ▼
BACKUP
     ↓
RESTAURACIÓN
     ↓
PRUEBA DE RECUPERACIÓN
     ↓
MIGRACIÓN A LINUX
     ↓
ACTUALIZACIÓN CONTROLADA
     ↓
PRODUCCIÓN
```

---

# ANEXO A — COMANDOS DE DIAGNÓSTICO

## Estado de servicios

```powershell
docker compose ps
```

## Contenedores

```powershell
docker ps -a
```

## Imágenes

```powershell
docker images
```

## Volúmenes

```powershell
docker volume ls
```

## Logs de n8n

```powershell
docker compose logs --tail=100 n8n
```

## Logs de PostgreSQL

```powershell
docker compose logs --tail=100 postgres
```

## Tipo de base de datos

```powershell
docker exec n8n printenv DB_TYPE
```

## Host de PostgreSQL

```powershell
docker exec n8n printenv DB_POSTGRESDB_HOST
```

## Base de datos

```powershell
docker exec n8n printenv DB_POSTGRESDB_DATABASE
```

## Política de reinicio de n8n

```powershell
docker inspect -f "{{.HostConfig.RestartPolicy.Name}}" n8n
```

## Política de reinicio de PostgreSQL

```powershell
docker inspect -f "{{.HostConfig.RestartPolicy.Name}}" n8n-postgres
```

---

# ANEXO B — ESTRUCTURA FINAL

```text
D:\n8n
│
├── .env
├── docker-compose.yml
└── backup\
```

Contenedores:

```text
n8n
n8n-postgres
```

Volúmenes:

```text
n8n_n8n_data
n8n_postgres_data
```

---

# ANEXO C — INFORMACIÓN CONFIDENCIAL

Nunca incluir valores reales de:

```text
POSTGRES_PASSWORD
N8N_ENCRYPTION_KEY
Contraseña de n8n
API Keys
Tokens
Credenciales
Secretos
```

Utilizar marcadores:

```text
<CONTRASEÑA_POSTGRES>
<N8N_ENCRYPTION_KEY>
<CONTRASEÑA_N8N>
<API_KEY>
<TOKEN>
```

---

# ANEXO D — DATOS PRINCIPALES DE LA INSTALACIÓN

| Parámetro             | Valor                       |
| --------------------- | --------------------------- |
| Proyecto              | `D:\n8n`                    |
| Compose               | `D:\n8n\docker-compose.yml` |
| Variables             | `D:\n8n\.env`               |
| Contenedor n8n        | `n8n`                       |
| Contenedor PostgreSQL | `n8n-postgres`              |
| Base de datos         | `n8n`                       |
| Host PostgreSQL       | `postgres`                  |
| Puerto PostgreSQL     | `5432`                      |
| Puerto n8n            | `5678`                      |
| URL                   | `http://localhost:5678`     |
| Volumen n8n           | `n8n_n8n_data`              |
| Volumen PostgreSQL    | `n8n_postgres_data`         |
| Política de reinicio  | `unless-stopped`            |
| Zona horaria          | `America/Lima`              |
| PostgreSQL            | `17`                        |

---

# ANEXO E — OPERACIÓN NORMAL

## Iniciar el entorno

```powershell
cd D:\n8n
docker compose up -d
```

## Verificar

```powershell
docker compose ps
```

Debe mostrarse:

```text
n8n            Up
n8n-postgres   Up (healthy)
```

## Acceder a n8n

```text
http://localhost:5678
```

## Detener temporalmente

```powershell
docker compose stop
```

## Iniciar nuevamente

```powershell
docker compose start
```

## Reiniciar

```powershell
docker compose restart
```

---

# FIN DEL MANUAL

**Manual Técnico de Instalación y Configuración de n8n Self-Hosted**

**Versión:** 1.0

**Fecha:** 21/08/2026

**Área responsable:** Sistemas

```
```
