# Verificación y Diagnóstico de Red

## 1. Problema

InnovaCloud Solutions no cuenta con un procedimiento estandarizado para verificar qué servicios están activos ni para diagnosticar la conectividad de red en sus servidores.

## 2. ¿Por qué esto es un problema?

- **Diagnósticos inconsistentes:** cada técnico revisa la red "a su manera", usando distintas herramientas o pasos, lo que genera reportes no comparables entre sí.
- **Tiempos de resolución más largos:** sin un checklist claro, identificar la causa raíz de un problema de red (¿es el servicio, el puerto, el firewall, el DNS, la ruta?) toma más tiempo del necesario.
- **Falta de trazabilidad:** sin un procedimiento documentado, no queda registro de qué se revisó y qué no ante un incidente, dificultando auditorías posteriores.
- **Riesgo de seguridad:** sin verificación periódica de puertos e interfaces activas, servicios innecesarios pueden quedar expuestos sin que nadie lo note.

## 3. Herramientas propuestas

`ip a` Para verificar interfaces de red activas y sus direcciones IP
`ip route` Para verificar la tabla de enrutamiento y la puerta de enlace
`ping` Para probar conectividad básica hacia un host
`traceroute` / `mtr` Identificar en qué salto de la red se pierde o retrasa la conexión `traceroute google.com`
`ss` Para ver puertos y servicios en escucha en el propio equipo `ss -tulpn`
`systemctl status` Para verificar el estado (activo/inactivo) de un servicio específico `systemctl status ssh`
`dig` / `nslookup` Para diagnosticar resolución de nombres DNS `dig innovacloud.com`
`nmap` Para auditar puertos abiertos y servicios expuestos en un host de la propia red interna `nmap -sV 192.168.1.50`
`curl -I` Para probar rápidamente si un servicio web responde en un puerto específico `curl -I http://192.168.1.50`

Cada una cubre una capa distinta del modelo de red: interfaces/routing (`ip`), conectividad extremo a extremo (`ping`, `traceroute`), servicios locales (`ss`, `systemctl`), resolución de nombres (`dig`) y auditoría de puertos remotos dentro de la propia red (`nmap`, `curl`). Usarlas en conjunto permite aislar en qué capa está el problema en lugar de "probar al azar".

## 4. Procedimiento estandarizado de diagnóstico

¿Por qué un checklist paso a paso? Porque establece un orden lógico (de lo local a lo remoto) que cualquier técnico de InnovaCloud puede seguir, garantizando que los diagnósticos sean repetibles y comparables entre sí.

### Paso 1 — Verificar interfaces de red locales
```bash
ip a
```
Confirmar que la interfaz de red esté `UP` y tenga una IP asignada.

### Paso 2 — Verificar la tabla de rutas
```bash
ip route
```
Confirmar que exista una ruta por defecto (`default via ...`) válida.

### Paso 3 — Probar conectividad básica
```bash
ping -c 4 <gateway>
ping -c 4 8.8.8.8
```
Primero al gateway local (descarta problemas de red local) y luego a un host externo (descarta problemas de salida a internet).

### Paso 4 — Verificar resolución DNS
```bash
dig google.com
```
Si el `ping` a una IP funciona pero a un nombre de dominio no, el problema está en el DNS, no en la conectividad.

### Paso 5 — Revisar servicios activos en el servidor
```bash
ss -tulpn
systemctl status <nombre-del-servicio>
```
Confirmar que el servicio esperado (por ejemplo, un servidor web o base de datos) esté efectivamente escuchando en el puerto correcto.

### Paso 6 — Auditar puertos expuestos desde otro equipo de la red
```bash
nmap -sV <ip-del-servidor>
```
Permite validar, desde una perspectiva externa (como la vería otra máquina de la red), qué puertos y servicios son realmente visibles, detectando servicios innecesarios que deberían cerrarse.

### Paso 7 — Documentar el resultado
Registrar en un reporte breve: interfaz verificada, resultado del ping, resultado de DNS, servicios encontrados y acciones tomadas.

## 5. Ejemplo de reporte de diagnóstico

```
Fecha: __________
Servidor: __________
Técnico: __________

1. Interfaz activa:        [OK / FALLA]  - IP: __________
2. Ruta por defecto:       [OK / FALLA]  - Gateway: __________
3. Ping a gateway:         [OK / FALLA]
4. Ping a internet:        [OK / FALLA]
5. Resolución DNS:         [OK / FALLA]
6. Servicios esperados activos: [OK / FALLA]  - Detalle: __________
7. Puertos expuestos detectados (nmap): __________
Observaciones: __________
```