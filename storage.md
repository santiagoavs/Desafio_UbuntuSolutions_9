# Solución para el Almacenamiento

## 1. Problema

El servidor principal de InnovaCloud Solutions presenta fallos de disco que provocan pérdida de datos, debido a que actualmente no existe ningún mecanismo de redundancia, es decir, cada disco almacena una única copia de la información.

## 2. ¿Por qué es crítico este problema para el negocio?

Los fallos en el disco son críticos para la empresa, puesto que conllevan un gran impacto financiero, esto
ya que la interrupción del negocio causa una bola de nieve si este mismo tiene un entorno basado mayormente en tecnología que trabajo humano. De ser así, la empresa enfrentaría gastos en recuperación de datos y procesos fundamentales, además de los costos por hora que genera la inactividad del negocio.
Un mal manejo de esto puede llevar a problemas legales, ya que la péridida de datos de clientes puede 
llegar a violar las leyes, como también puede ser el impedimento de registros financieros, contratos y bases de datos.
Por último, un gran golpe que tendría la empresa sería en la reputación, al fallar a la confianza de sus clientes y la desventaja competitiva que este problema causaría.

## 3. Elección del tipo de RAID

En base al problema y prontitud de solución que la empresa presenta, nuestra propuesta de tipo de RAID es el RAID 1.

### Propuesta: **RAID 1: espejo**

**¿Por qué RAID 1 y no otro?**

- **Simplicidad y confiabilidad:** sólo requiere 2 discos, ideal para un servidor principal donde la prioridad es no perder datos, no maximizar espacio.
- **Redundancia inmediata:** cada disco es una copia exacta del otro. Si un disco falla, el sistema sigue funcionando sin interrupción con el disco espejo, mientras se reemplaza el dañado.
- **Reconstrucción rápida:** al reconstruir un RAID 1 solo se copia el contenido de un disco a otro, a diferencia de RAID 5 que debe recalcular paridad (proceso más lento y con mayor riesgo durante la reconstrucción).
- **Trade-off aceptado:** se sacrifica el 50% de la capacidad total (dos discos de 1TB = 1TB útil), pero para un servidor "principal" con datos críticos, la seguridad prima sobre el aprovechamiento de espacio.

> Si en el futuro InnovaCloud necesita escalar a más discos manteniendo buen rendimiento, RAID 10 sería la evolución natural ya que combina el espejo + striping, pero requiere mínimo 4 discos, por lo que se recomienda como plan de crecimiento, no como solución inicial.

## 4. Implementación

### 4.1. Instalación de la herramienta

```bash
sudo apt update
sudo apt install mdadm -y
```

### 4.2. Creación del arreglo RAID 1

```bash
sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
```
¿Por qué estos parámetros?
- `--level=1`: define el tipo de RAID (espejo).
- `--raid-devices=2`: indica que se usan 2 discos físicos (`/dev/sdb` y `/dev/sdc`), no el disco del sistema operativo (`/dev/sda`), para no interferir con el arranque.
- `/dev/md0`: es el nombre del dispositivo RAID resultante que el sistema tratará como un único disco lógico.

### 4.3. Verificar el estado del arreglo

```bash
cat /proc/mdstat
sudo mdadm --detail /dev/md0
```
Esto permite confirmar que el arreglo se creó correctamente, que ambos discos están activos (`active sync`) y que la sincronización inicial entre ambos discos finalizó.

### 4.4. Formatear y montar el arreglo

```bash
sudo mkfs.ext4 /dev/md0
sudo mkdir -p /mnt/raid1
sudo mount /dev/md0 /mnt/raid1
```
Un arreglo RAID recién creado no tiene sistema de archivos; hay que formatearlo (`ext4`, por ser estándar, estable y bien soportado en Linux) y montarlo en un punto de acceso.

### 4.5. Persistir la configuración

```bash
echo '/dev/md0 /mnt/raid1 ext4 defaults 0 0' | sudo tee -a /etc/fstab
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
sudo update-initramfs -u
```
Sin estos pasos, el RAID se perdería (o no montaría automáticamente) al reiniciar el servidor. `/etc/fstab` asegura el montaje automático, y `mdadm.conf` asegura que el sistema reconozca el arreglo en cada arranque.

## 5. Simulación de fallo y recuperación (prueba de concepto)

```bash
# Simular fallo de un disco
sudo mdadm --manage /dev/md0 --fail /dev/sdb
sudo mdadm --detail /dev/md0

# Remover el disco fallado
sudo mdadm --manage /dev/md0 --remove /dev/sdb

# Tras reemplazar físicamente el disco, agregarlo de nuevo al arreglo
sudo mdadm --manage /dev/md0 --add /dev/sdb
```
Esto permite demostrarle al cliente que, ante un fallo real de disco, el servicio **no se detiene**: el sistema sigue operando con el disco restante mientras se reemplaza el dañado, cumpliendo el objetivo de continuidad de negocio.

## 6. Monitoreo continuo

```bash
sudo mdadm --monitor --daemonise --mail=admin@innovacloud.com --delay=1800 /dev/md0
```
El detectar un disco fallado a tiempo es tan importante como tener redundancia. Este comando activa un demonio que revisa el estado del RAID cada 1800 segundos y envía una alerta por correo si detecta una degradación, evitando que un segundo fallo (sin haber reemplazado el primero) resulte en pérdida total de datos.