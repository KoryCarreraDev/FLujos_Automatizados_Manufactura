# Flujos Automatizados — n8n Workflows

Coleccion de flujos de automatizacion construidos en [n8n](https://n8n.io/) para la gestion operativa de una empresa de manufactura. Los flujos integran **Google Sheets**, **Google Drive**, **PostgreSQL**, **Gmail**, **Telegram** y modelos de IA (Google Gemini / Ollama) para automatizar reportes, alertas, sincronizacion de datos y operaciones administrativas.

> **Nota:** Todos los IDs de documentos, credenciales y constantes sensibles han sido reemplazados por placeholders (p.ej. `YOUR_GOOGLE_SHEET_ID_INVENTARIO`). Debes configurar tus propios valores antes de importar los flujos.

---

## Tabla de contenido

- [Estructura del proyecto](#estructura-del-proyecto)
- [Categorias de flujos](#categorias-de-flujos)
  - [Administrativo](#administrativo)
  - [Produccion](#produccion)
  - [Reportes Con IA](#reportes-con-ia)
  - [Reportes Sin IA](#reportes-sin-ia)
  - [Sincronizacion Bilateral](#sincronizacion-bilateral)
- [Tecnologias utilizadas](#tecnologias-utilizadas)
- [Infraestructura Docker](#infraestructura-docker)
  - [Servicios](#servicios)
  - [Variables de entorno (.env)](#variables-de-entorno-env)
  - [Levantar el entorno](#levantar-el-entorno)
- [Importar flujos en n8n](#importar-flujos-en-n8n)
- [Placeholders en los flujos](#placeholders-en-los-flujos)

---

## Estructura del proyecto

```
Flujos_Automatizados_Beiplas/
+-- Administrativo/
|   +-- Alertas de Stock Minimo Cintas.json
|   +-- Calculador de Precios Santiago Alzate.json
|   +-- Respuesta movimiento inventario.json
|   +-- Solicitud de movimientos.json
+-- Produccion/
|   +-- Alertas y gestion de novedades.json
|   +-- Consolidado Mantenimiento.json
|   +-- Historial de observaciones.json
|   +-- Insertar y Calcular Simulador.json
|   +-- Tareas Preventivas Mantenimiento.json
+-- Reportes Con IA/
|   +-- Alertas Novedades.json
|   +-- Bot de Mantenimiento Beiplas.json
|   +-- Informe de Simulador.json
|   +-- Informe diario de entrega de turnos.json
+-- Reportes Sin IA/
|   +-- Alertas Lider Comercial.json
|   +-- Limpiar e Insertar Informe Simulador.json
+-- Sincronizacion Bilateral/
|   +-- Sincronizar Inventarios.json
|   +-- Sincronizar paros.json
|   +-- Sincronizar rendimiento x hora.json
|   +-- Sincronizar tiempos de maquina (kilos reales).json
|   +-- Sincronizar Ordenes de Trabajo (OTs).json
|   +-- sincronizar clientes.json
+-- docker-compose.example.yml   <- Infraestructura Docker de referencia
+-- .env.example                 <- Variables de entorno (copia como .env)
+-- volumenes/                   <- Datos persistentes (gitignored)
    +-- postgres/
    +-- n8n/
    +-- ollama/
    +-- node_app/
```

---

## Categorias de flujos

### Administrativo

Flujos orientados a la gestion de inventarios, precios y movimientos internos de almacen.

| Flujo | Trigger | Descripcion |
|---|---|---|
| **Alertas de Stock Minimo Cintas** | Programado | Consulta el inventario de cintas en Google Sheets y envia alertas por Gmail cuando el stock cae por debajo del minimo. |
| **Calculador de Precios** | Programado | Recalcula precios automaticamente segun condiciones definidas en Google Sheets y actualiza los valores. |
| **Solicitud de movimientos** | Programado | Revisa solicitudes de movimiento de inventario pendientes y notifica a los responsables por Gmail. |
| **Respuesta movimiento inventario** | Webhook | Recibe confirmaciones de movimientos via webhook, actualiza Google Sheets y responde al solicitante por Gmail. |

---

### Produccion

Flujos para el seguimiento de novedades, mantenimiento de maquinaria y simulacion de produccion.

| Flujo | Trigger | Descripcion |
|---|---|---|
| **Alertas y gestion de novedades** | Programado | Monitorea novedades del piso de produccion en Google Sheets. Envia alertas por Gmail y Telegram segun el tipo de evento. |
| **Consolidado Mantenimiento** | Programado | Consolida registros de mantenimiento desde Google Sheets, agrupa por lotes y genera un resumen actualizado. |
| **Historial de observaciones** | Sub-workflow | Registra el historial de observaciones de operarios y maquinas en Google Sheets. Se invoca desde otros flujos. |
| **Insertar y Calcular Simulador** | Programado | Lee datos de Google Sheets y PostgreSQL, aplica calculos del simulador de produccion y escribe los resultados de vuelta en ambas fuentes. |
| **Tareas Preventivas Mantenimiento** | Programado | Revisa las tareas de mantenimiento preventivo vencidas o proximas en Google Sheets y notifica a los responsables. |

---

### Reportes Con IA

Flujos que utilizan modelos de lenguaje (Google Gemini y Ollama) para generar analisis e informes inteligentes.

| Flujo | Trigger | IA | Descripcion |
|---|---|---|---|
| **Alertas Novedades** | Programado | Google Gemini | Lee novedades de Google Sheets, las analiza con un agente IA y envia un resumen por Gmail y Telegram. |
| **Bot de Mantenimiento** | Programado | Ollama (local) | Consulta el estado de mantenimiento en Google Sheets, lo procesa con un modelo local y envia el reporte por Telegram. |
| **Informe de Simulador** | Programado | Google Gemini | Exporta datos del simulador desde Google Drive/Sheets, los analiza con IA y distribuye el informe por Gmail. |
| **Informe diario de entrega de turnos** | Programado | Google Gemini | Recopila el estado del turno desde Google Sheets/Drive, genera un informe analitico con Gemini y lo envia por Gmail. |

---

### Reportes Sin IA

Flujos de reporte automatizados sin LLMs, basados en logica condicional y transformacion de datos.

| Flujo | Trigger | Descripcion |
|---|---|---|
| **Alertas Lider Comercial** | Programado | Consulta metricas comerciales en Google Sheets y envia alertas por Gmail al lider comercial segun condiciones predefinidas. |
| **Limpiar e Insertar Informe Simulador** | Sub-workflow | Limpia la hoja del simulador en Google Sheets y vuelve a insertar los datos calculados. |

---

### Sincronizacion Bilateral

Flujos que mantienen sincronizados Google Sheets y PostgreSQL de forma bidireccional.

| Flujo | Trigger | Descripcion |
|---|---|---|
| **Sincronizar Inventarios** | Programado | Compara y sincroniza el inventario entre Google Sheets y la base de datos PostgreSQL. |
| **Sincronizar paros** | Sub-workflow | Sincroniza los registros de paros de maquina entre Sheets y PostgreSQL. |
| **Sincronizar rendimiento x hora** | Programado | Escribe los datos de rendimiento horario desde Sheets a PostgreSQL. |
| **Sincronizar tiempos de maquina** | Sub-workflow | Sincroniza los tiempos reales de maquina y kilogramos producidos entre Sheets y PostgreSQL. |
| **Sincronizar Ordenes de Trabajo (OTs)** | Sub-workflow | Sincroniza las ordenes de trabajo activas desde Google Sheets hacia PostgreSQL. |
| **Sincronizar clientes** | Sub-workflow | Replica el maestro de clientes desde Google Sheets a PostgreSQL. |

---

## Tecnologias utilizadas

| Tecnologia | Uso |
|---|---|
| **n8n** | Motor de automatizacion (self-hosted) |
| **Google Sheets** | Fuente principal de datos operativos |
| **Google Drive** | Almacenamiento de archivos e informes |
| **Gmail** | Canal de notificaciones por correo |
| **Telegram** | Canal de alertas en tiempo real |
| **PostgreSQL** | Base de datos relacional de produccion |
| **Google Gemini (Langchain)** | Analisis e informes generados por IA |
| **Ollama (Langchain)** | Modelo de IA local para el bot de mantenimiento |

---

## Infraestructura Docker

El archivo [`docker-compose.example.yml`](./docker-compose.example.yml) define el entorno completo self-hosted para correr los flujos. Renombralo a `docker-compose.yml` antes de usarlo.

### Servicios

| Servicio | Imagen | Puerto | Descripcion |
|---|---|---|---|
| **postgres** | `postgres:16-alpine` | `5432` | Base de datos principal de n8n y datos de produccion. Incluye healthcheck. |
| **n8n** | `docker.n8n.io/n8nio/n8n:latest` | `${N8N_PORT}` | Motor de automatizacion. Usa PostgreSQL como backend y se conecta a Ollama. |
| **node_app** | `node:20-alpine` | `${NODE_PORT}` | Script runner Node.js auxiliar con acceso a PostgreSQL y Ollama. |
| **ollama** | `ollama/ollama:latest` | `11434` | Servidor de modelos de IA local. Los modelos persisten en `./volumenes/ollama`. |

Todos los servicios comparten la red interna `automation_network` (bridge). Los volumenes se montan en `./volumenes/` para persistencia de datos.

### Variables de entorno (.env)

Copia `.env.example` como `.env` y rellena los valores:

```bash
cp .env.example .env
```

| Variable | Descripcion | Ejemplo |
|---|---|---|
| `POSTGRES_USER` | Usuario de PostgreSQL | `n8n_user` |
| `POSTGRES_PASSWORD` | Contrasena de PostgreSQL | `contrasena_segura` |
| `POSTGRES_DB` | Nombre de la base de datos | `n8n_db` |
| `N8N_PORT` | Puerto del host para acceder a n8n | `5678` |
| `N8N_ENCRYPTION_KEY` | Clave para cifrar credenciales en n8n (32 bytes hex) | `openssl rand -hex 32` |
| `N8N_WEBHOOK_URL` | URL publica de n8n para recibir webhooks | `https://tu-dominio.com/` |
| `NODE_PORT` | Puerto del host para el script runner Node.js | `3000` |
| `NODE_ENV` | Entorno de Node.js | `production` |

> **Nunca subas el archivo `.env` real al repositorio.** Ya esta incluido en `.gitignore`.

### Levantar el entorno

```bash
# 1. Clonar el repo y preparar variables
cp .env.example .env
# Editar .env con tus valores reales

# 2. Levantar todos los servicios
docker compose -f docker-compose.example.yml up -d

# 3. Verificar que todos los contenedores esten healthy
docker compose -f docker-compose.example.yml ps

# 4. (Opcional) Descargar un modelo de Ollama
docker exec -it ollama_automation ollama pull llama3

# 5. Acceder a n8n
# Abre http://localhost:${N8N_PORT} en el navegador
```

Para detener el entorno:

```bash
docker compose -f docker-compose.example.yml down
```

---

## Importar flujos en n8n

1. Levanta el entorno Docker (ver seccion anterior).
2. Abre n8n en `http://localhost:${N8N_PORT}`.
3. Ve a **Workflows → Import from file**.
4. Selecciona el archivo `.json` del flujo que deseas importar.
5. En cada nodo con credenciales, asigna las tuyas (Google, Telegram, PostgreSQL, etc.).
6. Reemplaza los placeholders del flujo (ver seccion siguiente).
7. Activa el flujo con el toggle.

---

## Placeholders en los flujos

Los flujos JSON tienen placeholders que debes reemplazar con tus valores reales tras importar.
El ID de Google Sheet lo encuentras en la URL: `docs.google.com/spreadsheets/d/**ID_AQUI**/edit`.

| Placeholder | Flujo(s) afectado(s) | Descripcion |
|---|---|---|
| `YOUR_GOOGLE_SHEET_ID_INVENTARIO_CINTAS` | Alertas Stock Cintas, Sincronizar Inventarios | Hoja de inventario de cintas |
| `YOUR_GOOGLE_SHEET_ID_CALCULADOR_PRECIOS` | Calculador de Precios | Hoja del calculador de precios |
| `YOUR_GOOGLE_SHEET_ID_INVENTARIO` | Solicitud/Respuesta movimientos | Hoja general de inventario |
| `YOUR_GOOGLE_SHEET_ID_NOVEDADES` | Alertas novedades, Bot Mantenimiento | Hoja de novedades de produccion |
| `YOUR_GOOGLE_SHEET_ID_SIMULADOR` | Insertar Simulador, Informe Simulador | Hoja del simulador de produccion |
| `YOUR_GOOGLE_SHEET_ID_TIEMPOS` | Sincronizar tiempos, paros, OTs | Hoja de tiempos de maquina |
| `YOUR_GOOGLE_SHEET_ID_TURNOS` | Informe diario turnos | Hoja de entrega de turnos |
| `YOUR_GOOGLE_SHEET_ID_CLIENTES` | Sincronizar clientes | Hoja del maestro de clientes |
| `YOUR_GOOGLE_DRIVE_FILE_ID_SIMULADOR` | Informe Simulador | Archivo de Drive del simulador |
| `YOUR_GOOGLE_DRIVE_FOLDER_ID` | Calculador de Precios | Carpeta de Drive destino |
| `YOUR_TELEGRAM_CHAT_ID` | Alertas novedades, Bot Mantenimiento | ID del grupo/canal de Telegram |
| `YOUR_WEBHOOK_URL` | Solicitud de movimientos | URL publica de n8n para webhooks |
| `YOUR_N8N_INSTANCE_ID` | Todos | Se genera automaticamente al importar el flujo |

---

## Licencia

Ver [LICENSE](./LICENSE).
