# InnovaTech — Backend Despachos (Spring Boot)

**Descripción**  
API REST de despachos (Spring Boot 3, Java 21). Misma línea DevOps que ventas: **ECR**, **EC2 backend**, MySQL compartido `mysql-innovatech` (base `despachos_db`, puerto **3306**). Despliegue por push a `deploy`.

---

## 🧭 Estructura del proyecto

```
DevopsEV2-backend-despachos/
├── README.md
├── .github/workflows/deploy.yml
└── Springboot-API-REST-DESPACHO/
    ├── src/main/java/
    ├── Dockerfile
    ├── docker-compose.yml
    ├── entrypoint.sh
    └── pom.xml
```

---

## 🚀 Requisitos

- Docker y Docker Compose v2
- Java 21 y Maven 3.9+
- AWS Academy Learner Lab
- Secrets en GitHub (ver `infra/README.md`)

---

## ⚙️ Flujo de uso

### Local

```bash
cd Springboot-API-REST-DESPACHO
docker compose up -d --build
```

API: **http://localhost:8082** (mapeo según compose del repo)  
Swagger: **/swagger-ui.html**

### Despliegue AWS

1. Desplegar **ventas** primero (crea MySQL y BD `ventas_db`).
2. Push a **`deploy`** en este repo → contenedor en puerto **8082**.
3. El workflow crea `despachos_db` si no existe.

---

## 📦 ¿Qué despliega este proyecto?

| Entorno | Contenedor | Puerto |
|---------|------------|--------|
| Local | `backend-despachos` | 8082 |
| Local | `db-despachos` (MySQL) | 3306 |
| AWS | `innovatech-backend-despachos` | 8082 |
| AWS | MySQL (host compartido) | `DB_HOST` = IP privada backend, `DB_PORT` = 3306 |

---

## 🧭 Diagrama de arquitectura

```
Frontend Nginx  /api/v1/despachos
        ↓
EC2 Backend :8082  →  mysql-innovatech:3306 / despachos_db
```

---

## 📌 Mejores prácticas incluidas

**Multi-stage + JRE 21 + usuario `spring` (no-root).**

**Volúmenes**

| Tipo | Dónde | Motivo |
|------|--------|--------|
| **Named volume** | `docker-compose.yml` → `despachos-data-local` | Persistencia local de MySQL al reiniciar el contenedor de BD. |
| **Named vs bind** | No usamos bind mount para datos de BD | Evita rutas frágiles en el host; los named volumes son portables entre máquinas de desarrollo. |

En producción, MySQL no usa named volume en el workflow actual; el contenedor `mysql-innovatech` se reutiliza entre deploys.

**Orden de deploy:** ventas → despachos → frontend (evita OOM y asegura MySQL arriba).

---

## 🔧 Cómo extender este proyecto

- Cola de mensajes entre ventas y despachos (SQS).
- Migraciones con Flyway/Liquibase en el pipeline.
- Volume nombrado en EC2 para `/var/lib/mysql`.
- Pruebas de integración contra MySQL en CI.
