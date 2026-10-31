# BalanceFood

Plataforma para que estudiantes con la Beca de Alimentación JUNAEB comparen precios de platos, registren sus gastos y sepan cuánto pueden gastar cada día para que el saldo les alcance hasta fin de mes.

Proyecto de Computación Web y Móvil (UTEM) — Trabajo 1.

Este repositorio es el **índice principal**. El código vive en dos repositorios independientes:

| Repositorio | Contenido | Stack |
|---|---|---|
| [balancefood-backend](https://github.com/tsengian36-rgb/balancefood-backend) | API REST, autenticación JWT, 3 CRUD y recomendador | Ruby on Rails 7.1 (API) · PostgreSQL |
| [balancefood-frontend](https://github.com/tsengian36-rgb/balancefood-frontend) | Aplicación web | React 18 · Vite |

## Arquitectura

```
React (balancefood-frontend)  --HTTP/JSON + JWT-->  Rails API (balancefood-backend)  -->  PostgreSQL
```

## Funcionalidades

- Registro, inicio y cierre de sesión (JWT con revocación).
- CRUD de locales, platos y gastos.
- Recomendador de presupuesto diario: saldo ÷ días hábiles restantes (crítico < $3.000, medio < $5.000, holgado desde $5.000).

## Gestión del proyecto

Tareas gestionadas en Jira. Cada repositorio trabaja con commits y PR atómicos hacia la rama `release`, que se integra a `main` mediante un PR final.

## Autor

Ian Tseng
