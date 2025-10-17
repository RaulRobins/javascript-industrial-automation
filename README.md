# Guía Visual Profesional – JavaScript para Automatización Industrial con Siemens y Docker (12 Semanas)

**Autor:** Raúl Robins  
**Formato:** Cuaderno técnico (horizontal-friendly)  
**Duración:** 12 semanas · 10 h/semana

> Enfoque: Integración con **PLC Siemens S7-1500 / PLCSIM Advanced**, **OPC UA**, **Node.js**, **PostgreSQL**, **React**, **Node-RED**, todo en **Docker**.

---

## 1) Arquitectura General

**Flujo:** PLC/PLCSIM ⇄ OPC UA ⇄ Node-RED (adquisición) ⇄ Node.js API ⇄ PostgreSQL ⇄ React Dashboard

- **Adquisición:** Node-RED con `node-red-contrib-opcua` para leer variables (tags) del PLC.
- **Backend:** Node.js + Express: endpoints `/data`, `/status`, `/export.csv`.
- **DB:** PostgreSQL: tabla `parts` y `events` (ejemplo), índices por tiempo.
- **UI:** React (Vite) para dashboard (tablas, filtros, gráficas).
- **Infra:** Docker Compose para servicios: `postgres`, `pgadmin`, `nodered`, `react`, y **opcua-sim** (opcional).

---

## 2) Guía de Instalación Paso a Paso

### 2.1 Requisitos

- Windows 11/10 con **WSL2** habilitado (Ubuntu recomendado).
- **Docker Desktop** con WSL2 backend.
- **Node.js LTS** (20.x o superior) y **npm**/**pnpm**.
- **Git**.
- **PLCSIM Advanced** (para entorno Siemens) y **UaExpert** para pruebas OPC UA.

### 2.2 Node.js + pnpm

```bash
# Windows Terminal (WSL Ubuntu)
sudo apt update && sudo apt install -y curl git
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
npm i -g pnpm
node -v && pnpm -v
```

### 2.3 Docker + Compose

- Instala **Docker Desktop** y activa integración con WSL2.
- Verifica:

```bash
docker version
docker compose version
```

### 2.4 Clonar proyecto base (opcional)

```bash
mkdir -p ~/projects/siemens-js && cd ~/projects/siemens-js
# Copia aquí los archivos del anexo que vienen en esta guía
```

### 2.5 PostgreSQL + pgAdmin (vía Docker)

- Incluidos en `docker-compose.yml` del anexo.
- Acceso pgAdmin: `http://localhost:5050` (usuario/clave definidos en variables).

### 2.6 Node-RED

- Disponible en `http://localhost:1880` una vez levantado con Compose.
- Instala `node-red-dashboard` y `node-red-contrib-opcua` desde el Palette Manager.

### 2.7 React (Vite)

```bash
pnpm create vite@latest dashboard --template react
cd dashboard
pnpm install
pnpm run dev
```

### 2.8 PLCSIM Advanced + OPC UA

- Habilita el **OPC UA Server** en PLCSIM Advanced o en el S7-1500 real.
- Verifica con **UaExpert** que puedes leer tags.
- Para pruebas rápidas sin PLC, usa el servicio **opcua-sim** (incluido en el anexo).

---

## 3) Roadmap de 12 Semanas (10 h/semana)

### Semanas 1–2 — Fundamentos JS modernos

**Objetivo:** Sintaxis moderna y manipulación de datos industriales.

- ES6+: let/const, arrow functions, destructuring, spread/rest.
- Arrays/Objects (map, filter, reduce) aplicados a datos de sensores/PLC.
- DOM básico y eventos (para UI simple).
  **Práctica:** script que normalice lecturas (JSON) y exporte CSV.  
  **Checklist:** ✅ variables, funciones, arrays, objetos, módulos.  
  **Bibliografía:** _Eloquent JavaScript_ (cap. 1–6), MDN JS, YDKJSY: _Scope & Closures_.

### Semanas 3–4 — Asincronía, modularidad y patrones

**Objetivo:** Adquirir datos en tiempo real y tratarlos.

- Promesas, `async/await`, `fetch`, manejo de errores.
- Patrones (Observer, Factory, Singleton) para flujos industriales.
- Estructura de proyecto y módulos ES.
  **Práctica:** lector de API + pipeline con reintentos y logs.  
  **Checklist:** ✅ async/await, errores, módulos, patrones.  
  **Bibliografía:** _Effective JavaScript_, YDKJSY: _Async & Performance_.

### Semanas 5–6 — Backend Industrial (Node.js + Express + DB + OPC UA)

**Objetivo:** API de datos industriales.

- Express: rutas `/data`, `/status`, `/export.csv`.
- PostgreSQL: conexión, migraciones, índices por tiempo.
- Introducción a OPC UA (node-red-contrib-opcua o node-opcua).
  **Práctica:** endpoint que ingesta lecturas y las guarda.  
  **Checklist:** ✅ Express, SQL básico, pooling, seguridad mínima.  
  **Bibliografía:** _Express in Action_, _Node.js Design Patterns_.

### Semanas 7–8 — Dashboards (Node-RED y React)

**Objetivo:** Visualización y control.

- Node-RED Dashboard: tablas, indicadores, botones de exportación.
- React: componentes, estado, fetch, DataGrid/Chart.js.
  **Práctica:** tabla con filtros por rango de fecha y export CSV.  
  **Checklist:** ✅ dashboard funcional, filtros, exportación.  
  **Bibliografía:** _Learning React_, React Docs.

### Semanas 9–10 — Integración IoT + Dockerización

**Objetivo:** Empaquetar y desplegar.

- MQTT/HTTP/OPC-UA, logs y backups.
- Dockerizar Node, React, Node-RED; volúmenes para persistencia.
  **Práctica:** `docker compose up -d` con toda la pila.  
  **Checklist:** ✅ imágenes, redes, volúmenes, env vars.  
  **Bibliografía:** _Docker para Developers_ (referencia), MQTT Essentials (HiveMQ).

### Semanas 11–12 — Proyecto Final Siemens

**Objetivo:** Trazabilidad y monitoreo en línea con exportación.

- PLCSIM/OPC UA → Node-RED → Node.js → PostgreSQL → React.
- CSV export, filtro de fechas, búsqueda por etiqueta final.
- Backups automáticos y script de mantenimiento.
  **Checklist:** ✅ sistema corriendo en Docker, endpoints y UI ok.  
  **Entregable:** demo funcional + README con pasos.

---

## 4) Tablas de Referencia Rápida

### 4.1 Estructura de base de datos (ejemplo)

```sql
CREATE TABLE parts (
  id              bigserial PRIMARY KEY,
  final_assembly  text NOT NULL,
  backframe_40_lh text,
  backframe_40_rh text,
  backframe_20    text,
  cushion_100     text,
  torque          real[10],
  angle           numeric(10,3)[10],
  created_at      timestamptz DEFAULT now()
);
CREATE INDEX parts_final_assembly_idx ON parts (final_assembly);
CREATE INDEX parts_created_at_idx     ON parts (created_at);
```

### 4.2 Endpoints básicos (sugeridos)

- `GET /status` → healthcheck.
- `GET /data?from=...&to=...&final_assembly=...` → lecturas filtradas.
- `GET /export.csv?...` → exportación CSV con filtros.
- `POST /ingest` → ingesta desde Node-RED.

---

## 5) Bibliografía y Recursos

- **Eloquent JavaScript (3rd)** — Marijn Haverbeke (gratuito)
- **You Don’t Know JS Yet** — Kyle Simpson
- **Effective JavaScript** — David Herman
- **Node.js Design Patterns** — Mario Casciaro
- **Learning React** — Alex Banks & Eve Porcello
- **Clean Code in JavaScript** — James Padolsey
- **Node-RED Cookbook** — Nick O’Leary
- **MQTT Essentials** — HiveMQ Blog
- **React Docs** — https://react.dev
- **MDN Web Docs** — https://developer.mozilla.org

---

## 6) Anexo: Docker Compose base

Consulta el archivo `Anexo_Tecnico_Docker_Base.yml` incluido junto a esta guía.
Incluye servicios: `postgres`, `pgadmin`, `nodered`, `react`, y **opcua-sim** (opcional).

> Recomendación: crea una red interna `siemens-net` y volúmenes para persistencia. Ajusta puertos si tienes colisiones.

---

## 7) Notas finales

- Esta guía está optimizada para **WSL2 + Docker Desktop** en Windows.
- Si usas Linux nativo, elimina lo específico de WSL2 y conserva Docker + Compose.
- Para producción, considera **backups**, **monitoring** (Prometheus/Grafana) y **users/roles** en PostgreSQL.
