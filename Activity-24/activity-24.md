# Actividad: Arquitectura y desarrollo de microservicios con Docker y Kubernetes

1. ¿Por qué microservicios?

- Monolitos: Son las aplicaciones donde todos los módulos está integrados en un solo despliegue.
- SOA: Introduce las ideas de componentes reutilizables y servicios distribuidos, pero era muy pesado y dependiendo. Estaba más pensado a sistemas empresariales.
- Microservicios: Es la evolución de SOA con un enfoque más ligero, centrado a servicios pequeños, independientes y enfocados en una única responsabilidad.

**Casos de uso Tipico:**

- Ecommerce con picos de demanda: Durante eventos donde la aplicación sufre alta demanda como puede ser un Black Friday, un monolito no puede escalar solo el servicio de pagos o inventario. Con microservicios, se puede escalar solo lo necesario.
- Aplicaciones SaaS multi-tenat: Al atender múltiples clientes con necesidades diferentes, los microservicios permiten adaptar y desplegar nuevas condiciones sin afectar a otros módulos.

2. Definiciones clave

- Microservicio: Unidad independiente que encapsula una funcionalidad unica, expone una API y se depliega de manera autónoma.
  - Características:
    - Despliegue independiente
    - Enfonca en una única responsabilidad
    - Comunicación via API
- Aplicación de microservicios: Conjunto de microservicios que colaborar para cumplir objetivos de negocio, generalmente orquestadores mediante gateways, balanceadores de carga y herramientas de observabilidad.

3. Críticas al Monolítico

- **Despliegue lento y global**: Cualquier cambio requiere construir y desplegar toda la aplicación, incluso si una parte fue modificada.
- **Alto acoplamiento**: Dificulta el escalado individual de componentes y entorpece el trabajo paralelo de equipos

4. Popularida y beneficios

- Compañías como Netflix y Amazon adoptaron microservicios para mejorar su crecimiento exponencial y ciclos de despliegue acelerados.
- Beneficios
  - Resiliencia: Fallos en un servicio no derriban todo el sistema
  - Escalabilidad granular: Cada servicio puede escalarse de forma independiente.
  - Equipos autónomos: Equipos pueden desarrollar, desplegar y mantener sus servicios sin depender de otros

5. Desventajas y retos

- Desafios comunes
  - Mayor complejidad en redes y seguridad
  - Orquestácion de multiples servicios
  - Dificultad para mantener consistencia en datos distribuidos
  - Testing complejo

- Estrategias de mitigación:
  - Uso de contratos de OpenApi para definir Apis claras
  - Pruebas contractuales para garantizar compatibilidad de servicios
  - Incorporación de herramientas de trazabilidad como Jaeger
  - Aplicación del patron Saga para manejar transacciones distribuidas

6. Principios de diseño

- DDD: Permite identificar bounded contexts para definir que funcionalidades se agrupan dentro de un microservicio
- DRY: Aunque se busca evitar la multiplicación, a veces es mejor duplicar lógicamente que compartir bibliotecas que acoplan servicios
