# Práctica 2: Desarrollo Front-end / Back-end con Integración DevSecOps

Este repositorio contiene la implementación y resolución de mi **Tarea 2**, cuyo objetivo fue diseñar, completar e integrar un **Pipeline CI/CD con enfoque DevSecOps**, priorizando herramientas de automatización, pruebas y seguridad de principio a fin.

Durante esta tarea, simulé un escenario profesional real en el que se exige que el código que desarrollé, estructurado en arquitectura de microservicios e integrado en contenedores, cumpla de forma continua tanto con criterios **funcionales** como con políticas severas de **seguridad**.

## 🛠 Entregables Cumplidos

- Pipeline Integrado en `devsecops.yml`.
- Arquitectura de microservicios original preservada y 100% funcional.
- Superación de rigurosas fases de Linter, Testing Unitario y Análisis Analítico de Seguridad sin fallos.
- Documento de Justificación Técnica consolidado.

---

# 📚 Documento de Justificación Técnica de DevSecOps

Para el diseño del pipeline `.github/workflows/devsecops.yml`, decidí abarcar todos los vectores recomendados por las mejores prácticas del ciclo de desarrollo seguro. A continuación, desglose mi razonamiento analítico para la implementación de cada herramienta.

### 1. Instalación Reproducible
* **Herramienta:** `npm ci`
* **Fase en DevSecOps:** *Build / Plan*
* **Riesgo Mitigado:** Modificaciones subrepticias o inconsistencias de versiones transitivas en mis entornos ("Phantom Dependencies" o el clásico "Funciona en mi máquina").
* **Justificación de su necesidad:** Aunque mi sistema funcione hoy, los paquetes Node cambian a diario. Decidí que, en ausencia de instalaciones `ci` bloqueadas en un `package-lock.json`, un despliegue futuro podría instalar un parche nuevo pero roto o inestable por parte del autor original de un paquete, tirando un sistema en producción aparentemente de la nada.

### 2. Análisis de Calidad de Código (Linter)
* **Herramienta:** `eslint`
* **Fase en DevSecOps:** *Verify / Create*
* **Riesgo Mitigado:** Deuda técnica prematura, variables no inicializadas que detonan *memory leaks*, y degradación generalizada de la mantenibilidad de mi código.
* **Justificación de su necesidad:** Un código desprolijo "funciona" lógicamente pero acarrea costos de mantenimiento severos a largo plazo. Implementé este Linter para homogenizar los patrones de desarrollo, detectar de forma pasiva redundancias peligrosas de memoria y obligarme a programar con un estándar de equipo riguroso.

### 3. Testing Automático (Pruebas Unitarias/Integrales)
* **Herramienta:** `jest` (a través de `npm test`)
* **Fase en DevSecOps:** *Verify*
* **Riesgo Mitigado:** Bugs de Regresión; roturas de funcionalidades nucleares provocadas por integraciones inadvertidas.
* **Justificación de su necesidad:** A la hora de añadir refactorizaciones complejas, podría dañar algo como el login que funcionaba la semana pasada; el testing unitario ejerce como escudo contra estos errores garantizando que no lleguen a producción, sustituyendo la necesidad de que yo de forma manual tenga que probar todos los flujos en cada commit.

### 4. Seguridad de Código Estático (SAST)
* **Herramienta:** `semgrep`
* **Fase en DevSecOps:** *Verify / Secure* (*Shift-Left Security*)
* **Riesgo Mitigado:** Inyecciones de código (SQLi, Command Injection), malas manipulaciones de rutas (Path Traversal), o secretos/llaves Hardcodeados en texto plano directamente a mi control de versiones.
* **Justificación de su necesidad:** Un registro web puede ser 100% funcional y servir a las necesidades operativas de negocio, pero si mi backend accidentalmente estuviese guardando contraseñas rudimentarias o un JWT inseguro, el código base expone por completo al sistema. Con Semgrep freno despliegues con malas practicas antes de un atacante las explote.

### 5. Análisis de Seguridad de Componentes (SCA)
* **Herramienta:** `npm audit` (configurado bajo `--audit-level=critical`)
* **Fase en DevSecOps:** *Verify / Secure*
* **Riesgo Mitigado:** Integración y exposición originada por *Vulnerabilidades y Exposiciones Comunes* (CVEs) en librerías open-source subyacentes proveídas por terceras partes.
* **Justificación de su necesidad:** Gran parte del proyecto no es código mío de cero; son paquetes terceros heredados (`express`, `axios`, etc). Aún siendo mi código primario robusto, una dependencia desactualizada manipulando mis peticiones HTTP me puede generar severas fallas lógicas u operativas. Esta capa automatiza que siempre use versiones seguras en mis sub-herramientas.

### 6. Integración y Construcción de Artefactos inmutables
* **Herramienta:** `docker compose build` & Versionado Semántico por Commit de GitHub (`docker tag`)
* **Fase en DevSecOps:** *Package / Release*
* **Riesgo Mitigado:** "Drift" de configuración. Diferencias fatales entre las librerías de SO en mi ambiente local frente a un servidor en la nube.
* **Justificación de su necesidad:** Al empaquetar mi código con una imagen autoconstruida en el ciclo mediante un hash, mis librerías binarias quedan congeladas. Esto garantiza que lo que me aprueba mi SAST/SCA en GitHub, es byte a byte idéntico a lo que luego desplegaré como artefacto definitivo en Docker, sin cambios ocultos en el intermedio.

### 7. Comprobación y Seguridad del Contenedor 
* **Herramienta:** `aquasecurity/trivy-action`
* **Fase en DevSecOps:** *Release / Secure*
* **Riesgo Mitigado:** Fallos o huecos de escalamiento de privilegios o exploits a librerías profundas y anticuadas del sistema operativo subyacente de mi contenedor Docker (`alpine`, `glibc`, `npm` instaladores globales, etc.), que son invisibles para Node.js y Semgrep.
* **Justificación de su necesidad:** Con una arquitectura funcional en mis Dockerfiles, rara vez habría cambiado la imagen base de NodeJS si el servidor ya compilaba bien. Sin embargo, logré integrarle Trivy para descubrir y mitigar vulnerabilidades altamente peligrosas que venían por default en `Node:20-alpine`; con lo cual Trivy blindó mi contenedor local asegurando que mis bases en Linux no acarreen Troyanos a futuros Clusters que orqueste.

---

## 🚀 Probar Localmente
Para probar funcionalmente que mis servicios no arrojan errores subyacentes:
1. `git clone <repo>`
2. `docker compose -f backend/docker-compose.yml up -d`
3. Monitoree su local en `http://localhost:3000/`.