# Instalación de zabbix  


## Instalción en ubuntu  

```
sudo wget https://repo.zabbix.com/zabbix/3.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_3.4-1+xenial_all.deb
dpkg -i zabbix-release_3.4-1+xenial_all.deb
apt update

apt install zabbix-server-mysql
apt install zabbix-proxy-mysql
apt install zabbix-frontend-php
```
### Creación de la base de datos  

```
- En mi caso no tenía instalado mysql `apt install mariadb-server`.



create database zabbix_server character set utf8 collate utf8_bin;
grant all privileges on zabbix_server.* to zabbix@localhost identified by 'secreto';
create database zabbix_proxy character set utf8 collate utf8_bin;
grant all privileges on zabbix_proxy.* to zabbix@localhost identified by 'secreto';

```
### Importamos los esquemas  

```
zcat /usr/share/doc/zabbix-server-mysql/create.sql.gz | mysql -uroot -pise zabbix_server
zcat /usr/share/doc/zabbix-proxy-mysql/schema.sql.gz | mysql -uroot -pise zabbix_proxy


```

## Conocimiento relevante sobre la práctica 

### Estrategias de pull y push en monitorización 
 
 *Zabbix* utiliza los dos modos, aunque por defecto lo hace en pull.  
#### Push

(Empujar). 
Ejemplo en `nagios`.  
- Paradigma tradicional
- Sistema centralizado
- Los clientes que quieren ser minitorizados, envían la información al servidor centralizado (zabbix) en este caso. - Reduce la carga del servidor principal (no está ocupado haciendo los push) .
- Parecido a estrategia interrucción (cuando se produce evento significativo se comunica). 
- Problema no tienes control en qué momento te llega la información (servidor central saturado).


#### Pull 
- muestreol, estrategia más antigua 
- va servidor a servidor preguntando por los datos, (muestreo en secuencia).
- La información se almacena en cada servidor hasta que el central lo consulta. 
Lo usa `prometheus` 


### Arquitectura de zabbix, procesos que intervienen  

#### Servidor  
Núcleo es el que recopila los datos, en nuestro caso será *sql* aunque pordrían ser otras como postgre, bases de datos memoria ...  En la actualidad la mayoría utilizan bases de datos especializadas (en series temporales o documentales)
 

####  Agentes  
- Procesos que corren en sistema a morizar.
- Por lo general un apache o un mysql en background.
- Cuando zabbix server le pida los datos se los devolverá. 
- Agente toma medida locales también: memoria, CPU, entradas/salidas...
- Agente tiene capacidad de almacenar métricas durante un tiempo. 
- Cuando zabbix se las pide las manda todas.

#### Proxy  

- Necesario para escalar: cuando un servidor no es capaz de muestrear a todos los servicios.   
- Es un server pequeñito, que muestrea a servicios locales; el server entonces lo que hace es consultar a los proxys.  
- Nos interesa ya que el tiempo de abertura en TCP/IP es muy grande y nos interesa que cuando esa conexión se realice nos de toda la información posible.  

#### Get  

- Implenta un interfaz de tipo rest, para consultar agentes de tipo zabbix.  
- Útil para depurar problemas; inlcuso podriamos desarrollar nuestros propios clientes.   

 
### Zabbit agent  

Es un paquete que se instala a parte. 
Con la orden `zabbix_get` indicamos contra que agente nos vamos a conectar y hacerle una petición.

Mirar variables principales que podemos recuperar con zabbix_get.

Los item son las variables que admites. 



## Ansible  

Gestor de la configuración, `vagran` es un ejemplo. 
Ansible se utiliza en los playbooks ficheros de texto plano para la configuración. 

- *Terraform* va más orientado a la nube.  

- necesita python y ssh para funcionar. 
### inventario  

Catálogo ordenadores a los que nos vamos a conectar.  
El inventario se encuentra en `etc/ansible/host`
`ansible.cfg` de configuración (hebras, donde se encuentra el ansible por defecto)

`.ansible.cfg` fichero en nuestro directorio home, la configuración de aquí domina a la otra. 


Para configurar un archivo: 

abrir fichero `/etc/absible/host`

Escribir línea con con servidores a configurar. 
 
`ubudef ansible_host=ip`  ubudef es un nombre simbólico dentro del catálogo. 

también podemos configurar el usuario con el que accede: 

`ubudef ansible_host= pi ansible_user=usuarioNombre`
Vamos a probar que funciona. 

`ansible -m ping ubudef`

Donde `-m` indica que es un módulo. 


`ansible all -m ping`

Ansible lanza varias hebras. 

 También podemos configurar nombres simbólicos para los alias
 
 ```
 [lamp]
 cenise
 ubuise
 ```
 
 y si hiciéramos ahora
 
 `ansible lamp -m ping`  
 Solo probaría el ping a las configuradas. 
 
 
###  

### Módulos  

Un módulo es una serie de comandos con cierta coherencia. 

Problemas: 

Tenemos que incluir un usuario `-u usuarioConEntrar`   

Ahora tenemso que permitir usuarios sin contraseña, (reducir fachada usando ssh)  

ejemplo de módulo: `ansible -m ping ubudef -u mjose -s 'data="hola que tal"' 

otros: 

`shell` ejecutar comandos remotos. (aunque es el módulo que se ejecuta por defecto y se puede omitir. 

ejemplo: 'ansible all -m shell -a 'echo $PATH' coge la configuración de profile 

(que es común a todos los usuario). 


SE suele configurar en dos pasos: 

PRimero usuario y luego 

### Módulos de servicio   
Se basan en hacer llamadas a programas ya instalados. 
Podemos configurar los nuestros propios. (tienen que ser idenpoentes)
(mismas acciones tienes mismos resultados)


Progrmación declarativa: describe e estado al que que queremos llegar, el estado final. (si ya lo tenía no hace nada) NO ES PROCEDURAL, NO SE DECLARAN LOS PASOS QUE HACER. 

`ansibke cenise -m service -a "name=httod state=stopped"`

Para parar el servicio 


Vamos a configurar usuario privilegiado sin contraseña, 

esto se encuentra en el fichero de sudo: 
`/etc/sudoers` 
editamos de la forma que se quede como: 

Creamos un grupo y que cualquier usuario que sea de ahí pueda 

`%admin ALL=(ALL) NOPASSWD: ALL`  

cualquier usuario que sea admin podrá acceder sin contraseña  

`usermod -a -G admin david`

### fuentes 
- bases de datos en series temporales: https://tienda.digital/2019/08/06/que-son-las-bases-de-datos-de-series-temporales/  

- base de datos en memoria: https://es.wikipedia.org/wiki/Base_de_datos_en_memoria  
- bases de datos documentales: https://es.wikipedia.org/wiki/Base_de_datos_documental

