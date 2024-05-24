# Programación paralela
Una guía para programación en paralelo, utilizando mpich y nfs en el entorno de openSUSE Leap.

## Dependencias

### Descargar openSUSE Leap 15.5

Para este caso se utilizó la distribución openSUSE Leap en su versión 15.5.

[openSUSE Leap 15.5](https://get.opensuse.org/leap/15.5/)

### Intalar mpich

### Instalar gcc

## Guía

### Configurar NFS

#### En la Máquina Servidora

1. Instalar NFS:
```
sudo zypper install nfs-kernel-server
```

2. Configurar el directorio a compartir:
Edita el archivo /etc/exports y coloca directorio que quieres compartir. Por ejemplo: /home/usuario/carpeta.
```
/home/usuario/carpeta <ip-máquina-cliente>(rw,sync,no_subtree_check)
```

3. Reiniciar el servidor NFS:
```
sudo systemctl restart nfsserver
```

4. Permitir el tráfico NFS a través del firewall:
```
sudo firewall-cmd --permanent
--add-service=nfs
sudo firewall-cmd --reload
```

#### En la Máquina Cliente

1. Instalar NFS:
```
sudo zypper install nfs-client
```

2. Crear un punto de montaje:
```
sudo mkdir -p /mnt/carpeta
```
Nota: Se debe utilizar el mismo nombre de la carpeta del directorio que compartió la máquina servidora.

3. Montar el directorio compartido:
```
sudo mount <ip-máquina-servidora>:/home/usuario/carpeta /mnt/carpeta
```

4. Agregar entrada al archivo /etc/fstab para montar automáticamente el directorio en el arranque:
Ir hasta el directorio /etc, editar el archivo fstab y colocar lo siguiente:
```
<ip-máquina-servidora>:/home/usuario/carpeta /mnt/carpeta nfs defaults 0 0
```

### Ejecutar el Programa con MPI

#### Compilar el programa

1. Escribir y compilar el programa en la máquina servidora:
```
/usr/lib64/mpi/gcc/mpich/bin/mpicc tu_programa.c -o /home/usuario/carpeta/tu_programa
```
Ahora el ejecutable estará disponible en el directorio compartido /home/usuario/carpeta.

#### Crear el Archivo de Hosts

1. Crear un archivo de hosts en el directorio compartido:
```
ip-máquina-servidora
ip-máquina-cliente
```

#### Ejecutar el Programa

1. Ejecutar el programa desde la máquina servidora:
```
/usr/lib64/mpi/gcc/mpich/bin/mpiexec -f /home/usuario/carpeta/hosts -n 2 /home/usuario/carpeta/tu_programa
```
* -f /home/usuario/carpeta/hosts especifica el archivo hosts.

* -n 2 indica que se ejecutarán 2 procesos en total, uno en cada máquina.

##### Nota sobre Control de Procesos

Si necesitas especificar más de un proceso por máquina, puedes controlar esto directamente desde la línea de comandos de mpiexec. Por ejemplo, si quieres ejecutar dos procesos en la máquina sevidora y uno en máquina cliente:
```
mpiexec -f /home/usuario/carpeta/hosts -n 3 --map-by ppr:2:node /home/usuario/carpeta/tu_programa
```

* -n 3: Especifica que se ejecutarán 3 procesos en total.
* --map-by ppr:2:node: Especifica que cada nodo recibirá 2 procesos, y el tercero se asignará automáticamente al siguiente nodo.
