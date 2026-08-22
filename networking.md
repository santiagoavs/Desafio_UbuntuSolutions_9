# Solución para la Configuración de Red

## 1. Problema

Las máquinas virtuales del entorno de desarrollo de InnovaCloud usan el modo de red **NAT por defecto** de VirtualBox, lo cual dificulta la comunicación entre máquinas virtuales entre sí y con otros recursos corporativos (servidores, otras VMs, equipos de la red interna).

## 2. ¿Por qué el modo NAT es una limitante en un entorno colaborativo?

En el modo NAT cada máquina virtual vive detrás de una red privada y aislada creada por VirtualBox:

- La VM puede salir a internet, pero no es alcanzable desde el host ni desde otras VMs, salvo que se configure manualmente la redirección de puertos, lo cual es un proceso confuso y poco escalable.
- Cada VM tiene su propia subred NAT aislada, así que dos VMs distintas ni siquiera pueden verse entre sí por defecto.
- Es inviable para un entorno de trabajo colaborativo donde los desarrolladores necesitan que sus VMs se comuniquen entre sí o accedan a recursos de la red corporativa (bases de datos internas, repositorios, etc.).

## 3. Modo recomendado: **Adaptador Puente (Bridged)**

**¿Por qué Bridged y no otro modo?**

- En modo **Puente**, la VM obtiene una IP dentro del mismo segmento de red física de la empresa (como si fuera un equipo más conectado al switch corporativo).
- Esto permite que la VM sea alcanzable **tanto por otras VMs como por cualquier recurso de la red corporativa** (servidores, otras estaciones de trabajo), cumpliendo exactamente la necesidad planteada: comunicación entre máquinas virtuales y recursos corporativos.
- A diferencia de **NAT Network** o **Red Interna**, que solo resuelven la comunicación *entre VMs* pero mantienen aislamiento del resto de la red corporativa, **Bridged** integra la VM de forma completa a la infraestructura existente, sin necesidad de reglas de redirección de puertos.

> **Cuándo NO usar Bridged:** si se necesitara aislar completamente un entorno de pruebas (por ejemplo, para no exponer una VM vulnerable a la red corporativa), **Red Interna** sería más apropiada. Pero para el caso de un entorno de *desarrollo colaborativo*, Bridged es la opción correcta.

## 4. Configuración de IP estática con Netplan

**¿Por qué usar una IP estática y no DHCP?**

En un entorno de desarrollo donde otras VMs o servicios necesitan conectarse de forma confiable a una máquina (por ejemplo, un servidor de base de datos de pruebas), una IP que cambia cada vez que se reinicia la VM (como ocurre con DHCP) rompe esas conexiones. Una IP estática asegura que la VM siempre sea alcanzable en la misma dirección.

### 4.1. Identificar el nombre de la interfaz de red

```bash
ip a
```
*¿Por qué?* Antes de configurar Netplan hay que confirmar el nombre exacto de la interfaz (por ejemplo `enp0s3`), ya que varía según el hardware virtual y la distribución.

### 4.2. Editar el archivo de configuración de Netplan

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Contenido de ejemplo:

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 192.168.1.50/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

¿Por qué cada campo?
- `dhcp4: no`: desactiva la asignación dinámica de IP, requisito para tener una dirección fija.
- `addresses`: define la IP estática y su máscara de red en notación CIDR (`/24` = `255.255.255.0`).
- `routes` con `to: default`: define la puerta de enlace (gateway) para que la VM pueda salir a otras redes/internet (es la sintaxis moderna recomendada por Netplan, en reemplazo del antiguo `gateway4`).
- `nameservers`: asegura resolución de nombres DNS aunque no se use DHCP.

### 4.3. Validar la sintaxis antes de aplicar

```bash
sudo netplan try
```
`netplan try` aplica el cambio de forma temporal y revierte automáticamente si no se confirma en unos segundos, evitando quedarnos sin acceso a la VM por un error de configuración (por ejemplo, un gateway mal escrito).

### 4.4. Aplicar la configuración

```bash
sudo netplan apply
```

### 4.5. Verificar

```bash
ip a
ip route
ping -c 4 192.168.1.1
```
Esto Confirma que la IP estática quedó asignada correctamente (`ip a`), que la ruta por defecto apunta al gateway esperado (`ip route`) y que hay conectividad real hacia la red (`ping`).