# BalanceFood — Optimiza tu Saldo JUNAEB

Plataforma web para que estudiantes con Beca de Alimentación JUNAEB comparen precios de platos, registren sus gastos y sepan qué pueden costear cada día para que el saldo alcance hasta fin de mes.

Proyecto de la asignatura **Computación Web y Móvil** (UTEM) — Tarea 1: Aplicación Web.
Equipo: Ian Nicolás Tseng Pereira.

Este paquete corresponde a los aportes de **Ian Tseng**: logout con revocación de token (con test), motor recomendador (con test), CRUD de locales/platos/gastos, y Swagger de la API. Está pensado para subirse al repo del equipo como PRs separados — ver sección "Cómo subir esto al repo del equipo".

## URLs (completar antes de entregar)
- Frontend: `https://...`
- API: `https://.../api/v1`
- Swagger: `https://.../api-docs`
- Tablero Jira: `https://...`
- Credenciales de prueba: `test@balancefood.cl` / `Password123`

## Stack
Ruby 3.3 · Rails 7.1 (modo API) · API REST + JSON · PostgreSQL · React 18 (Vite) · Git/GitHub · Jira.

## Arquitectura
```
React (SPA, Vite)  --HTTP/JSON + Bearer JWT-->  Rails API (/api/v1)  -->  PostgreSQL
```
El frontend guarda el JWT en `localStorage` y lo envía en `Authorization: Bearer <token>`.

### Autenticación
1. `POST /auth/signup` o `/auth/login` valida credenciales (`bcrypt` vía `has_secure_password`) y devuelve un JWT HS256 con `user_id`, `jti` y `exp` (24 h).
2. Cada request protegida pasa por `ApplicationController#authenticate_request!`, que verifica firma, expiración y que el `jti` no esté en `revoked_tokens`.
3. `DELETE /auth/logout` guarda el `jti` en `revoked_tokens`. Ese token deja de ser válido aunque no haya expirado — probado en `test/controllers/api/v1/auth_logout_test.rb`.

## Modelo de datos
| Tabla | Campos principales | Relaciones |
|---|---|---|
| users | name, email (único), password_digest, current_balance (≥ 0) | tiene muchos expenses |
| restaurants | name, category, address, description | tiene muchos menu_items |
| menu_items | restaurant_id (FK), name, price (> 0), category, available | pertenece a restaurant |
| expenses | user_id (FK), menu_item_id (FK opcional), description, amount (> 0), spent_on | pertenece a user |
| revoked_tokens | jti (único), expires_at | — |

Las restricciones (`price > 0`, `amount > 0`, `current_balance >= 0`, FKs, email único) están en la base de datos (check constraints) y en los modelos. **Se probaron con SQL real contra PostgreSQL**, no solo en el papel.

## API REST (`/api/v1`)
| Recurso | Endpoints |
|---|---|
| Auth | `POST /auth/signup`, `POST /auth/login`, `DELETE /auth/logout`, `GET /auth/me` |
| **CRUD 1** Locales | `GET/POST /restaurants`, `GET/PATCH/DELETE /restaurants/:id` (filtros `category`, `q`) |
| **CRUD 2** Platos | `GET/POST /menu_items`, `GET/PATCH/DELETE /menu_items/:id` (filtros `restaurant_id`, `category`, `min_price`, `max_price`) |
| **CRUD 3** Gastos | `GET/POST /expenses`, `GET/PATCH/DELETE /expenses/:id` (solo del usuario autenticado) |
| Recomendador | `GET /recommendations?budget=NNNN` |
| Swagger | `GET /api-docs` |

Códigos HTTP: 200, 201, 204, 400, 401, 404, 422. Errores con formato `{ "error": "...", "details": {...} }`. Roles y permisos no forman parte de esta entrega (Tarea 2).

## Instalación y ejecución local
Requisitos: Ruby 3.3, Rails 7.1, PostgreSQL 14+, Node 20+.

```bash
# Backend
cd backend
bundle install
bin/rails db:create db:migrate db:seed
bin/rails s            # http://localhost:3000
bin/rails test         # corre los tests, incluido el de logout y el del recomendador

# Documentación Swagger (opcional, requiere el servidor corriendo)
RAILS_ENV=test bin/rails rswag:specs:swaggerize
# luego visita http://localhost:3000/api-docs

# Frontend
cd frontend
npm install
npm run dev            # http://localhost:5173
```

## Variables de entorno
| Variable | Dónde | Descripción |
|---|---|---|
| `DATABASE_URL` | backend (producción) | Conexión a PostgreSQL |
| `SECRET_KEY_BASE` | backend (producción) | Firma de JWT (`bin/rails secret`) |
| `FRONTEND_ORIGIN` | backend | Orígenes permitidos por CORS, separados por coma |
| `RAILS_ENV` | backend | `production` en el deploy |
| `VITE_API_URL` | frontend (build) | URL de la API, ej. `https://api.../api/v1` |

## Tests incluidos
- `test/controllers/api/v1/auth_logout_test.rb` — confirma que **el mismo token recibe 401 después del logout** (evidencia de la Etapa 2 de la hoja de ruta).
- `test/services/budget_recommendation_service_test.rb` — presupuesto diario en escenarios crítico, medio, holgado y en el último día hábil del mes (Etapa 3).
- `test/controllers/api/v1/restaurants_test.rb` — CRUD completo + 401/404/422.
- `spec/requests/api/v1/*_spec.rb` — specs de rswag que generan `swagger/v1/swagger.yaml`.

## Nota importante sobre esta entrega
Este proyecto se armó y se revisó en un entorno sin acceso a internet, así que **no pude ejecutar `bundle install`, `npm install` ni los tests** para confirmarlos de punta a punta. Sí hice lo que pude verificar sin red:
- Los 41 archivos `.rb` pasan `ruby -c` (sintaxis válida).
- Las migraciones se probaron como SQL real contra PostgreSQL: los 3 check constraints y la FK fallan exactamente como deben.
- Los 11 archivos `.jsx`/`.js` del frontend transpilan sin errores con esbuild, y cada import fue cruzado a mano contra el nombre real del archivo (así evitamos el error de `Registeer.jsx`).
- Los números del test del recomendador (días hábiles, presupuesto diario, estado) se recalcularon aparte en Ruby puro y coinciden.

Lo único que falta es que tú corras `bundle install` y `bin/rails test` una vez, con internet, para confirmar que todo pasa en tu máquina. Si algo falla, pégame el error exacto y lo corrijo.
