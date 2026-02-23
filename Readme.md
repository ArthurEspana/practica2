## Enfoque de la práctica
Esta práctica debe ser implementada, no solo diseñada.
El objetivo es aplicar DevSecOps de manera práctica, integrando:
        - - Front-end
        - - Back-end
        - - Inicio de sesión seguro
        - - Arquitectura de microservicios
        - - Automatización CI/CD con seguridad embebida
     

# 1. Adición del Front-end

[ Front-end ]
     |
     | Login / JWT
     v
[ users-service ]
     |
     | JWT
     v
[ api-gateway ]
     |
     v
[ academic-service ]


## Integración DevSecOps (obligatoria)
El Front-end y el inicio de sesión deben estar cubiertos por el pipeline DevSecOps existente:

- SAST: análisis del código de autenticación y manejo de inputs.
- SCA: análisis de dependencias relacionadas con seguridad.
- DAST: pruebas de acceso no autorizado a endpoints protegidos.

**El login no se asume seguro, se valida automáticamente

## Propósito de esta extensión
Consolidar una visión end-to-end DevSecOps, donde:
 - El diseño,
 - La seguridad,
 - La automatización,
  Y la experiencia de usuario,
se integran desde las primeras etapas del desarrollo.

## Pipeline 
Commit / Pull Request
   ↓
Tests automatizados
   ↓
SAST (Semgrep)
   ↓
Build (Docker)
   ↓
SCA (dependencias)
   ↓
Deploy automático
   ↓
DAST (aplicación en ejecución)

## Docker Compose
docker-compose down
docker-compose up --build

## Estructura del Pipeline
Push / Pull Request
   ↓
Install dependencies
   ↓
Tests (backend + frontend)
   ↓
SAST (Semgrep)
   ↓
Build Docker images
   ↓
SCA (Trivy)
   ↓
docker-compose up
   ↓
Smoke tests

## Kubernetes
kubectl apply -f k8s/users-service/
kubectl apply -f k8s/academic-service/
kubectl apply -f k8s/api-gateway/

kubectl get pods
kubectl get services

# Correr api-gateway
minikube service api-gateway
minikube start
# Trabajar con Docker
eval $(minikube docker-env -u)
# Trabajar Docker dentro Kubernetes
1. minikube start --driver=docker
   eval $(minikube docker-env)
2. minikube status
3. kubectl config current-context
4. kubectl get nodes
## Construir las imágenes
docker build -t frontend:latest ../frontend
kubectl get pods -n backend
docker build -t users-service:latest ../backend/users-service

## Desplegar en Kubernetes
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/users-service/
kubectl apply -f k8s/academic-service/
kubectl apply -f k8s/api-gateway/
kubectl apply -f k8s/frontend/

# eliminar cluster
minikube delete


minikube start
eval $(minikube docker-env)

docker build -t users-service backend/users-service
docker build -t academic-service backend/academic-service
docker build -t api-gateway backend/api-gateway
docker build -t frontend frontend

kubectl apply -f k8s/

# Justificación Técnica del Pipeline DevSecOps

Este documento detalla las decisiones técnicas adoptadas en la implementación del pipeline CI/CD en `.github/workflows/devsecops.yml`, cumpliendo con los lineamientos de DevSecOps descritos en la Práctica 2. Cada etapa del pipeline no solo asegura la funcionalidad del sistema, sino que integra controles de seguridad críticos desde las etapas más tempranas de desarrollo (*Shift Left*).

A continuación se analizan las fases del pipeline y las herramientas implementadas.

---

### 1. Instalación Reproducible
- **Herramienta usada:** `npm ci`
- **Fase de DevSecOps:** *Build Tooling & Configuration*
- **Riesgo mitigado:** Inconsistencias entre entornos (*"En mi máquina funciona"*). Prevención de la instalación inyectada accidental de dependencias no referenciadas o versiones incorrectas que podrían romper el sistema.
- **Necesidad:** Incluso cuando la aplicación ya es funcional localmente en entorno de desarrollo, generar *builds* limpias con dependencias exactas usando el `package-lock.json` garantiza que todos los entornos posteriores (CI, staging y producción) emplearán de forma estricta los mismos artefactos.

### 2. Análisis de Calidad de Código (Linter)
- **Herramienta usada:** `ESLint`
- **Fase de DevSecOps:** *Code / Local Analysis*
- **Riesgo mitigado:** Errores de sintaxis, anti-patrones de diseño y código "sucio" (*Code Smells*). Mitiga además bugs lógicos o fallas potenciales en la semántica del lenguaje.
- **Necesidad:** El código puede funcionar de manera feliz ("happy path") e ignorar malas prácticas. Mantener un estilo de código estricto reduce la deuda técnica, previniendo fallos futuros o problemas de mantenimiento, siendo vital para un desarrollo a largo plazo.

### 3. Testing Automático
- **Herramienta usada:** `Jest` (framework de test de React y Node) y scripts `npm test`.
- **Fase de DevSecOps:** *Test (Funcional / Unitario)*
- **Riesgo mitigado:** Errores de lógica y regresiones en código. Evita romper funcionalidades del negocio o alterar el comportamiento esperado luego de refactorizar.
- **Necesidad:** Aunque el desarrollador pruebe el sistema en su máquina, las pruebas manuales no escalan y omiten fácilmente flujos alternos. Las pruebas automatizadas ofrecen una red de seguridad innegociable (*gate*) que bloqueará el código inestable antes de que alcance usuarios reales.

### 4. Seguridad de Dependencias Externas (SCA)
- **Herramienta usada:** `npm audit --audit-level=critical`
- **Fase de DevSecOps:** *Code / Build (Software Composition Analysis)*
- **Riesgo mitigado:** Inclusión de paquetes maliciosos o vulnerabilidades conocidas (CVEs) de terceros. Mitiga ataques a la cadena de suministro (*Supply Chain Attacks*).
- **Necesidad:** Una aplicación moderna como este esquema de microservicios está compuesta hasta por un 80% de código de terceros (librerías públicas). Si una librería externa posee un CVE crítico, el sistema funcional se vuelve un blanco de ataques incluso cuando nuestro propio código sea inofensivo. Es imperativo detectar de forma automatizada estas debilidades. 

### 5. Análisis Estático de Seguridad del Código (SAST)
- **Herramienta usada:** `Semgrep`
- **Fase de DevSecOps:** *Code / Security Gate (Static Application Security Testing)*
- **Riesgo mitigado:** Presencia de credenciales hardcodeadas, llamadas a funciones inseguras, vectores de ataques genéricos como (Inyección SQL, XSS). Mitiga falencias OWASP de las primeras etapas.
- **Necesidad:** Realizar escaneos estáticos en PRs evita que patrones inseguros de código suban a la rama principal. Aunque el sistema operativo y entorno funcionen perfectos a nivel macro, vulnerabilidades como "Inyección SQL" sólo pueden descubrirse leyendo semánticamente el código antes de lanzar.

### 6. Build de Contenedores y Estructura Base
- **Herramienta usada:** `Docker` / `Docker Compose`
- **Fase de DevSecOps:** *Release / Deploy*
- **Riesgo mitigado:** Discrepancias directas de los entornos de ejecución (Sistema Operativo y Configuración de Red Base).
- **Necesidad:** Permite encapsular el ambiente exacto que necesitan los microservicios sin polucionar los nodos base.

### 7. Seguridad de Contenedores (Scanner de Imágenes)
- **Herramienta usada:** `Trivy` (Trivy Action)
- **Fase de DevSecOps:** *Release / Container Security*
- **Riesgo mitigado:** Despliegue de imágenes base desactualizadas con fallos en paquetes del OS, así como librerías expuestas del entorno y malas configuraciones del `Dockerfile`.
- **Necesidad:** Como DevOps, se construyen imágenes docker continuamente. La app puede ser 100% segura semánticamente, pero si se despliega sobre una base "Node:20" genérica sin parches y expuesta a CVEs en OS (`curl`, `openssl`), la seguridad final está comprometida. Trivy es nuestro último *gate* previo a la orquestación.

---

### Resumen
Estas herramientas consolidan un pipeline de tipo **fail-fast**: detiene y reporta de inmediato la compilación al encontrar violaciones funcionales o de seguridad, convirtiendo a DevSecOps en un habilitador para lanzamientos más ágiles y robustos, demostrando el ciclo virtuoso de integrar la seguridad "por defecto" (Shift-Left).