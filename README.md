# Ubuntu Solutions

**Consultora de Infraestructura y Sistemas Linux**

Somos un equipo de consultoría especializados en diagnóstico, optimización y documentación de insfraestructura sobre entornos Linux. En este repositorio se detalla el estudio y plan de acción técnico desarrollado para InnovaCloud Solutions

---

## Descripción del Proyecto

InnovaCloud Solutions enfrentaba problemas críticos en su red e infraestructura interna que comprometían la continuidad de su operación. En base a las fallas que vimos en su organización, desarrollamos una metodología que podrá hacer posible a la empresa una mayor estabilidad y continuidad a largo plazo de su negocio.

### Problemáticas identificadas

- Almacenamiento: Fallos de disco en el servidor principal que causan pérdida de datos por falta de redundancia.
- Gestión de software: Instalación manual de paquetes que genera inconsistencias de versión y alto consumo de ancho de banda.
- Red de desarrollo: Configuración por defecto (NAT) en VirtualBox que dificulta la comunicación entre máquinas virtuales y otros recursos corporativos.
- Diagnóstico y control: Falta de estandarización en la verificación de servicios activos y diagnóstico de conectividad en la red del cliente.

---

## Objetivo del Repositorio

Documentar, de forma técnica y justificada, la solución integral propuesta para cada una de las problemáticas identificadas, de modo que el equipo de InnovaCloud Solutions pueda comprender qué se propone, por qué se propone y cómo implementarlo.

---

## Resumen de las Soluciones

Fallos de disco sin redundancia: Implementación de **RAID 1** por software con `mdadm`, garantizando continuidad ante el fallo de un disco [`storage.md`](./storage.md)
Instalación manual e inconsistente de paquetes: **Repositorio espejo local** con `apt-mirror`, centralizando versiones y reduciendo consumo de ancho de banda [`packages.md`](./packages.md)
VMs aisladas por NAT por defecto: Migración a **modo Puente (Bridged)** en VirtualBox + IP estática mediante Netplan [`networking.md`](./networking.md)
Falta de procedimiento de diagnóstico: **Checklist estandarizado** de herramientas y pasos para auditar interfaces, servicios y conectividad [`diagnostics.md`](./diagnostics.md)

---

## Equipo

Santiago Alejandro Ávila Vásquez. AV262350

---