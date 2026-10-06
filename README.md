# Automatización Trello → n8n → GitHub

Automatización desarrollada con **n8n** para integrar la gestión de tareas de **Trello** con un repositorio de **GitHub**.

El objetivo es utilizar las tarjetas de Trello como punto de entrada para automatizar acciones sobre el repositorio, principalmente la creación de ramas asociadas a las tareas de desarrollo.

## Flujo

```text
Trello
   │
   │ Evento de tarjeta
   ▼
n8n
   │
   ├── Obtiene información de la tarjeta
   │
   ├── Formatea y valida los datos
   │
   ├── Detecta etiquetas
   │
   └── Determina el tipo de rama
          │
          ▼
       GitHub
```

## Reglas actuales

El tablero utilizado para la demostración es:

```text
n8n_DAS
```

Las tarjetas que contienen la etiqueta:

```text
dev
```

se procesan como tareas de desarrollo y generan una rama con el formato:

```text
feature/<nombre-de-la-tarea>
```

Ejemplo:

```text
Trello:
interfaz de login5

Git:
feature/interfaz-de-login5
```

Las reglas para otros tipos de ramas pueden agregarse posteriormente.

Ejemplo previsto:

```text
dev      → feature/
hotfix   → hotfix/
```

## Datos procesados

n8n normaliza la información recibida desde Trello antes de enviarla a las operaciones de GitHub.

La estructura principal generada es:

```json
{
  "event": {
    "type": "updateCard",
    "actionId": "..."
  },
  "board": {
    "id": "...",
    "name": "...",
    "url": "..."
  },
  "card": {
    "id": "...",
    "shortId": 38,
    "name": "...",
    "description": "...",
    "shortLink": "...",
    "url": "..."
  },
  "list": {
    "id": "...",
    "name": "..."
  },
  "label": {
    "name": "dev",
    "color": "green"
  },
  "git": {
    "type": "feature",
    "branchName": "feature/...",
    "branchSlug": "..."
  },
  "member": {
    "id": "...",
    "username": "...",
    "fullName": "..."
  }
}
```

## Tecnologías

* [n8n](https://n8n.io/)
* Trello API / Webhooks
* GitHub API
* Docker
* PostgreSQL

## Arquitectura

n8n funciona como capa de automatización entre Trello y GitHub.

```text
┌──────────────┐
│    Trello    │
│              │
│   Tarjetas   │
│   Etiquetas  │
└──────┬───────┘
       │
       │ Webhook
       ▼
┌──────────────┐
│     n8n      │
│              │
│  Trigger     │
│      ↓       │
│  Formateo    │
│      ↓       │
│   Reglas     │
│      ↓       │
│  GitHub API  │
└──────┬───────┘
       │
       │ API
       ▼
┌──────────────┐
│    GitHub    │
│              │
│  Repository  │
│   Branches   │
└──────────────┘
```

## Ejecución

El proyecto utiliza Docker Compose para ejecutar n8n y PostgreSQL.

Copiar el archivo de variables de entorno:

```bash
cp .env.example .env
```

Configurar las variables necesarias y levantar los servicios:

```bash
docker compose up -d
```

Comprobar el estado:

```bash
docker compose ps
```

Ver los logs de n8n:

```bash
docker logs -f n8n
```

## Credenciales

Las credenciales de Trello y GitHub deben configurarse directamente dentro de n8n.

**No almacenar tokens, API keys, secretos OAuth ni contraseñas dentro del repositorio.**

El archivo `.env` tampoco debe subirse a GitHub.

## Estado del proyecto

### Implementado

* [x] Recepción de eventos desde Trello
* [x] Identificación del tablero
* [x] Identificación de la tarjeta
* [x] Obtención de descripción y nombre
* [x] Identificación de lista
* [x] Identificación de etiqueta `dev`
* [x] Generación del nombre de rama
* [x] Normalización de datos para GitHub
* [x] Conexión con GitHub

### Pendiente

* [ ] Crear automáticamente la rama en GitHub
* [ ] Evitar duplicación de ramas
* [ ] Actualizar información cuando cambia una tarjeta
* [ ] Procesar etiqueta `hotfix`
* [ ] Crear commits automáticamente
* [ ] Asociar cambios de GitHub con tarjetas de Trello
* [ ] Manejar errores y reintentos
* [ ] Registrar ejecuciones de automatización

## Ejemplo

Una tarjeta:

```text
Nombre:
Implementar autenticación

Descripción:
Agregar login mediante JWT
```

con la etiqueta:

```text
dev
```

produce:

```text
feature/implementar-autenticacion
```

Esta rama será utilizada posteriormente por la automatización de GitHub.

## Seguridad

Nunca subir al repositorio:

```text
.env
*.pem
*.key
credentials.json
tokens
API keys
OAuth secrets
```

Utilizar `.gitignore` para excluir información sensible.
