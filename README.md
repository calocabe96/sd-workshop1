# Sistemas Distribuidos - Taller 1
## Carlos Ocampo -  A00052216

### **Requerimientos**

Para este taller se utilizaron los siguientes componentes:

* Centos i386
* centos1706_v0.2.0
* Vagrant

### **Desarrollo**

Los comandos necesarios para instalar todos los servicios necesarios en las máquinas son los siguientes:



### **Web Server 1 y Web Server 2**
#### Instalación de paquetes
```ruby
yum_package 'httpd' do
 action :install
end

yum_package 'php' do
    action :install
end

yum_package 'php-mysql' do
    action :install
end

yum_package 'mysql' do
    action :install
end

service 'httpd' do
    action [ :enable, :start ]
end

bash 'open port' do
	user 'root'
	code <<-EOH
	iptables -I INPUT 5 -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
	service iptables save
	EOH
end

template '/var/www/html/index.html' do
	source 'web_servers.erb'
	mode 0644
	owner 'root'
	group 'wheel'
  variables(
  :apaches => node[:apaches]
)
end

cookbook_file '/var/www/html/info.php' do
	source 'info.php'
	mode 0644
	owner 'root'
	group 'wheel'
end


cookbook_file '/var/www/html/select.php' do
	source 'select.php'
	mode 0644
	owner 'root'
	group 'wheel'
end
```
---
### **Database Server**
```ruby
yum_package 'mysql-server' do
 action :install
end

service 'mysqld' do
    action [ :enable, :start ]
end

bash 'open port' do
	user 'root'
	code <<-EOH
	iptables -I INPUT 5 -p tcp -m state --state NEW -m tcp --dport 3306 -j ACCEPT
	service iptables save
	EOH
end

yum_package 'expect' do
 action :install
end

cookbook_file '/tmp/configure_mysql.sh' do
	source 'configure_mysql.sh'
	mode 0711
	owner 'root'
	group 'wheel'
end

bash 'configure mysql' do
 user 'root'
 group 'wheel'
 cwd '/tmp'
 code <<-EOH
 ./configure_mysql.sh
 EOH
end

cookbook_file '/tmp/create_schema.sql' do
	source 'create_schema.sql'
	mode 0644
	owner 'root'
	group 'wheel'
end

bash 'create schema' do
	user 'root'
	cwd '/tmp'
	code <<-EOH
	cat create_schema.sql | mysql -u root -pdistribuidos
	EOH
end
```

A continuacón los comandos para la creación de una tabla

```sql
CREATE database database1;
USE database1;
CREATE TABLE example(
id INT NOT NULL AUTO_INCREMENT,
PRIMARY KEY(id),
 name VARCHAR(30),
 age INT);
INSERT INTO example (name,age) VALUES ('flanders',25);

-- http://www.linuxhomenetworking.com/wiki/index.php/Quick_HOWTO_:_Ch34_:_Basic_MySQL_Configuration
GRANT ALL PRIVILEGES ON *.* to 'icesi'@'192.168.56.101' IDENTIFIED by '12345';
GRANT ALL PRIVILEGES ON *.* to 'icesi'@'192.168.56.103' IDENTIFIED by '12345';
```

---

### **Load Balancer**
#### Instalación de paquetes

### **Vagrantfile**
```ruby
yum_package 'haproxy'

template '/etc/haproxy/haproxy.cfg' do
	source 'haproxy.erb'
	mode 0644
	owner 'root'
	group 'wheel'
	variables(
		:web_servers => node[:web_servers]
	)
end

service 'haproxy' do
  action [ :start, :enable ]
end

```

### **RECETAS**

**Ubicación** | **Descripción**
--- | ---
/cookbooks/apache | En este directorio se encuentra la información de instalación y configuración para los 2 servidor web en el que en la carpeta files están los archivos de la página web que se va a mostar y en recipies la configuración que se le hará al servicio. Se utiliza la misma receta para los dos servidores web, solo que usando un archivo de Ruby se mostrará un HTML diferente, dependiendo de la variable que le pase a través del Vagrantfile. 
/cookbooks/mysql | En este directorio se encuentran los archivos de configuración y aprovicionamiento para la máquina del servidor de base de datos, en el que en su archivo de configuración se le dan los permisos de acceso a ambos servidores web.
/cookbooks/haproxy | En este directorio se encuentran los archivos para el aprovicionamiento del balanceador de carga, que se encargará de distribuir las peticiones de acceso a los servidores web.


--- 
### **FUNCIONAMIENTO**
![][1]
![][2]
![][4]
![][3]



[1]: A00052216/Imagenes/web1.jpeg
[2]: A00052216/Imagenes/web2.jpeg
[3]: A00052216/Imagenes/web3.jpeg
[4]: A00052216/Imagenes/web4.jpeg
