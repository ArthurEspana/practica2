# Práctica 2: Desarrollo Front-end / Back-end con Integración DevSecOps

Este repositorio contiene la implementación y resolución de la **Tarea 2**, cuyo objetivo es diseñar, completar e integrar un **Pipeline CI/CD con enfoque DevSecOps**, priorizando herramientas de automatización, pruebas y seguridad de principio a fin.

Esta tarea simula un escenario real en el que se exige que todo código, estructurado en arquitectura de microservicios e integrado en contenedores, cumpla de forma continua tanto con criterios **funcionales** como con políticas severas de **seguridad**.

## 🛠 Entregables Cumplidos

- Pipeline Integrado en `devsecops.yml`.
- Arquitectura de microservicios original preservada y 100% funcional.
- Superación de rigurosas fases de Linter, Testing Unitario y Análisis Analítico de Seguridad sin fallos.
- Documento de Justificación Técnica consolidado.

---

# 📚 Documento de Justificación Técnica de DevSecOps

El pipeline `.github/workflows/devsecops.yml` fue concebido abarcando todos los vectores recomendados por las mejores prácticas del ciclo de desarrollo seguro. A continuación se desglosa el razonamiento analítico de la implementación de cada herramienta.

### 1. Instalación Reproducible
* **Herramienta:** `npm ci`
* **Fase en DevSecOps:** *Build / Plan*
* **Riesgo Mitigado:** Modificaciones subrepticias o inconsistencias de versiones transitivas en los entornos ("Phantom Dependencies" o "Funciona en mi máquina").
* **Justificación de su necesidad:** Aunque el sistema funcione hoy, los paquetes Node cambian a diario. En ausencia de instalaciones `ci` bloqueadas en un `package-lock.json`, un despliegue mañana podría instalar un parche nuevo pero roto o inestable por parte del autor original de un paquete, tirando un sistema en producción aparentemente de la nada.

### 2. Análisis de Calidad de Código (Linter)
* **Herramienta:** `eslint`
* **Fase en DevSecOps:** *Verify / Create*
* **Riesgo Mitigado:** Deuda técnica prematura, variables no inicializadas que detonan *memory leaks*, y degradación generalizada de la mantenibilidad del código.
* **Justificación de su necesidad:** Un código desprolijo "funciona" lógicamente pero acarrea costos de mantenimiento severos a largo plazo. Un Linter homogeniza los patrones del equipo, detecta de forma pasiva redundancias peligrosas de memoria y obliga a estandarizar los estándares del equipo.

### 3. Testing Automático (Pruebas Unitarias/Integrales)
* **Herramienta:** `jest` (a través de `npm test`)
* **Fase en DevSecOps:** *Verify*
* **Riesgo Mitigado:** Bugs de Regresión; roturas de funcionalidades nucleares provocadas por integraciones inadvertidas.
* **Justificación de su necesidad:** A la hora de añadir refactorizaciones complejas, un desarrollador puede dañar el login que funcionaba la semana pasada; el testing unitario ejerce como escudo contra estos errores garantizando que no lleguen a producción, sustituyendo el engorroso e imperfecto testeo humano manual.

### 4. Seguridad de Código Estático (SAST)
* **Herramienta:** `semgrep`
* **Fase en DevSecOps:** *Verify / Secure* (*Shift-Left Security*)
* **Riesgo Mitigado:** Inyecciones de código (SQLi, Command Injection), malas manipulaciones de rutas (Path Traversal), o secretos/llaves Hardcodeados en texto plano directamente al control de versiones.
* **Justificación de su necesidad:** Un registro web puede ser 100% funcional y servir a las necesidades operativas de negocio, pero si está guardando contraseñas cifradas con algoritmos débiles internos, o tiene un error lógico de JWT, el código base expone a toda la compañía.

### 5. Análisis de Seguridad de Componentes (SCA)
* **Herramienta:** `npm audit` (configurado bajo `--audit-level=critical`)
* **Fase en DevSecOps:** *Verify / Secure*
* **Riesgo Mitigado:** Integración y exposición originada por *Vulnerabilidades y Exposiciones Comunes* (CVEs) en librerías open-source subyacentes proveídas por terceras partes.
* **Justificación de su necesidad:** El 90% de las aplicaciones modernas no es código escrito por el desarrollador; son paquetes terceros heredados. Aún siendo el código primario estricto, una dependencia desactualizada manipulando el Parser HTTP puede acarrear una crisis por denegación de servicio (ReDOS), que es bloqueada gracias al escaneo continuo SCA.

### 6. Integración y Construcción de Artefactos inmutables
* **Herramienta:** `docker compose build` & Versionado Semántico por Commit de GitHub (`docker tag`)
* **Fase en DevSecOps:** *Package / Release*
* **Riesgo Mitigado:** "Drift" de configuración. Diferencias fatales entre las librerías de SO en Staging y Producción.
* **Justificación de su necesidad:** Al empaquetar código con una imagen autoconstruida en el ciclo, las librerías binarias quedan congeladas. Asegura que lo que pasa el SAST/SCA en GitHub, se corre byte a byte igual que en Amazon AWS mediante un artefacto estático.

### 7. Comprobación y Seguridad del Contenedor 
* **Herramienta:** `aquasecurity/trivy-action`
* **Fase en DevSecOps:** *Release / Secure*
* **Riesgo Mitigado:** Fallos o huecos de escalamiento de privilegios o exploits a librerías profundas y anticuadas del sistema operativo del contenedor Docker base (`alpine`, `glibc`, `npm` root tools, etc.), invisibles para Node.js y Semgrep.
* **Justificación de su necesidad:** Con una arquitectura funcional en Docker, rara vez los programadores actualizan el `FROM node:X` de cada dockerfile si éste "funciona". Trivy es capaz de detener el pipeline si detecta que la imagen final porta backdoors graves dentro del Linux contenido antes de enviarse al orquestador real (Kubernetes).

---

## 🚀 Probar Localmente
Para probar funcionalmente que los servicios no arrojan errores subyacentes:
1. `git clone <repo>`
2. `docker compose -f backend/docker-compose.yml up -d`
3. Monitoree su local.