# Solución para la Gestión de Paquetes

## 1. Problema

En InnovaCloud Solutions la instalación de software se realiza manualmente en cada máquina, sin un repositorio centralizado.

## 2. Riesgos

- **Inconsistencia de versiones:** cada técnico instala paquetes en momentos distintos, contra los repositorios oficiales de internet. Esto provoca que dos servidores "iguales" terminen con versiones distintas de la misma librería o paquete, generando bugs difíciles de reproducir, por ejemplo, "en mi máquina funciona".
- **Alto consumo de ancho de banda:** si 20 máquinas descargan el mismo paquete de forma independiente desde internet, se descarga 20 veces la misma información, saturando el enlace de salida a internet de la empresa.
- **Dependencia de conectividad externa:** si el proveedor de internet falla o el repositorio oficial está caído, no se puede instalar ni actualizar nada.
- **Sin control ni auditoría:** no hay forma centralizada de saber qué versiones están instaladas en la infraestructura ni de congelar una versión "aprobada" antes de un despliegue.

## 3. Solución propuesta: repositorio espejo local con `apt-mirror`

**¿Por qué un repositorio espejo local y no seguir usando los repositorios de internet directamente?**

Un mirror local descarga y almacena una copia completa (o filtrada) de los repositorios oficiales de Ubuntu/Debian una sola vez dentro de la red de InnovaCloud. A partir de ahí, todas las máquinas internas instalan paquetes contra ese servidor local en lugar de internet.

¿Qué beneficios tiene esto?

Primero, es eficaz, ya que el paquete se descarga una única vez desde internet, el resto ocurre en la red interna, que es mucho más rápida que internet. Esto también soluciona la inconsistencia, ya que todas las máquinas terminan instalando exactamente las mismas versiones, ya que se descargan del mismo repositorio congelado. 
El ancho de  banda ya no sería un problema, porque se reduce drásticamente el tráfico proveniente de internet, porque sólo el servidor mirror sincroniza los repositorios oficiales. Por último, si el internet falla momentáneamente, las máquinas internas pueden seguir instalando o actualizando paquetes con el mirror local.

## 4. Implementación

### 4.1. Instalar `apt-mirror` en el servidor que actuará como repositorio local

```bash
sudo apt update
sudo apt install apt-mirror -y
```

### 4.2. Configurar qué repositorios se van a espejar

```bash
sudo nano /etc/apt/mirror.list
```

Contenido de ejemplo:

```
set base_path    /var/spool/apt-mirror
set nthreads     20
set _tilde 0

deb http://archive.ubuntu.com/ubuntu noble main restricted universe multiverse
deb http://archive.ubuntu.com/ubuntu noble-updates main restricted universe multiverse
deb http://archive.ubuntu.com/ubuntu noble-security main restricted universe multiverse

clean http://archive.ubuntu.com/ubuntu
```

¿Por qué estos parámetros?
- `base_path`: define dónde se almacenará físicamente la copia del repositorio.
- `nthreads 20`: permite descargas en paralelo, acelerando la sincronización inicial.
- Se incluyen `main`, `restricted`, `universe` y `multiverse` para cubrir la totalidad de paquetes que el cliente podría necesitar, evitando tener que reconfigurar el mirror cada vez que falte un paquete.

### 4.3. Ejecutar la sincronización inicial

```bash
sudo apt-mirror
```
Este comando descarga por primera vez toda la estructura de paquetes definida en `mirror.list`. Es un proceso que puede tardar horas dependiendo del ancho de banda, pero se hace **una sola vez** (y luego incrementalmente).

### 4.4. Automatizar la sincronización periódica

```bash
sudo crontab -e
```
Agregar la línea:
```
0 2 * * * /usr/bin/apt-mirror
```
Esto programa la sincronización todas las noches a las 2 AM, cuando el consumo de red de la empresa es mínimo, manteniendo el mirror actualizado sin afectar la operación diurna.

### 4.5. Publicar el mirror vía HTTP

```bash
sudo apt install apache2 -y
sudo ln -s /var/spool/apt-mirror/mirror /var/www/html/ubuntu
```
Las máquinas cliente necesitan acceder al mirror mediante HTTP, tal como lo harían con un repositorio oficial de internet.

### 4.6. Configurar las máquinas cliente para usar el mirror local

```bash
sudo nano /etc/apt/sources.list
```
Reemplazar por:
```
deb http://IP_DEL_SERVIDOR_MIRROR/ubuntu noble main restricted universe multiverse
deb http://IP_DEL_SERVIDOR_MIRROR/ubuntu noble-updates main restricted universe multiverse
deb http://IP_DEL_SERVIDOR_MIRROR/ubuntu noble-security main restricted universe multiverse
```

```bash
sudo apt update
```
A partir de este cambio, todas las instalaciones y actualizaciones de paquetes en esa máquina se resuelven contra el servidor interno, cumpliendo el objetivo de consistencia y ahorro de ancho de banda.
