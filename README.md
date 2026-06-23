# InnovaTech — Backend Despachos (Spring Boot)

**Descripción**  
API REST de despachos (Spring Boot 3, Java 21). Misma línea DevOps que ventas: **ECR**, despliegue en **EKS** vía pipeline central de `DevopsEV2-infra`, MySQL compartido en el clúster (`mysql:3306`, base `despachos_db`).

---

## 🧭 Estructura del proyecto

```
DevopsEV2-backend-despachos/
├── README.md
└── Springboot-API-REST-DESPACHO/
    ├── src/main/java/
    ├── src/test/java/             # context tests
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
- Infra y secrets en `DevopsEV2-infra/README.md`

---

## ⚙️ Flujo de uso

### Local

```bash
cd Springboot-API-REST-DESPACHO
docker compose up -d --build
```

API: **http://localhost:8082** (mapeo según compose del repo)  
Swagger: **/swagger-ui.html**

### Despliegue AWS (EV3 — EKS)

1. Aplicar Terraform en `DevopsEV2-infra` (`etapa_1` + `etapa_3`).
2. Configurar secrets AWS en **DevopsEV2-infra**.
3. Push a **`deploy`** en **DevopsEV2-infra** → el workflow despliega ventas, despachos y frontend en un solo pipeline.

Verificar:

```bash
kubectl get pods -l app=backend-despachos
kubectl get hpa backend-despachos-hpa
```

> El despliegue AWS se dispara únicamente desde **DevopsEV2-infra** (rama `deploy`).

---

## 📦 ¿Qué despliega este proyecto?

| Entorno | Contenedor / Pod | Puerto |
|---------|------------------|--------|
| Local | `backend-despachos` | 8082 |
| Local | `db-despachos` (MySQL) | 3306 |
| AWS (EKS) | `backend-despachos` (3 réplicas, HPA 1–6) | Service ClusterIP **8081** |
| AWS (EKS) | MySQL (pod compartido) | `mysql:3306`, BD `despachos_db` |

Variables en K8s: `DB_HOST=mysql`, `DB_NAME=despachos_db`, credenciales desde Secret `db-credentials`.

**ECR:** `innovatech-backend-despachos`

---

## 🧭 Diagrama de arquitectura

```
Pod frontend (Nginx)  /api/v1/despachos
        ↓
Pod backend-despachos :8081  ← HPA (CPU 50%)
        ↓
Pod MySQL :3306 / despachos_db
```

```
DevopsEV2-infra (cd.yml)
        ├── build + push → ECR
        └── kubectl set image deployment/backend-despachos
```

---

## 📌 Mejores prácticas incluidas

**Multi-stage + JRE 21 + usuario `spring` (no-root).**

**Volúmenes**

| Tipo | Dónde | Motivo |
|------|--------|--------|
| **Named volume** | `docker-compose.yml` → `despachos-data-local` | Persistencia local de MySQL al reiniciar el contenedor de BD. |
| **Named vs bind** | No usamos bind mount para datos de BD | Evita rutas frágiles en el host; los named volumes son portables entre máquinas de desarrollo. |

En EKS, MySQL corre en un pod compartido; `despachos_db` se crea vía ConfigMap `mysql-init` en `DevopsEV2-infra/k8s/mysql.yml`.

**CI/CD (EV3):** el pipeline central construye y despliega ventas, despachos y frontend en secuencia dentro del mismo workflow.

**HPA:** `backend-despachos-hpa` escala entre 1 y 6 réplicas según CPU.

---

## 🔧 Cómo extender este proyecto

- Cola de mensajes entre ventas y despachos (SQS).
- Migraciones con Flyway/Liquibase en el pipeline.
- PersistentVolumeClaim para datos de MySQL en EKS.
- Pruebas de integración contra MySQL en CI.
- Liveness/readiness probes en el deployment K8s.
