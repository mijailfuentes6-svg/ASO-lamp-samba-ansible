PROYECTO: DESPLIEGUE LAMP, SAMBA, FTP Y POSTGRESQL CON ANSIBLE
Sistema de destino: openSUSE Leap 16.0
Repositorio: https://github.com/mijailfuentes6-svg/ASO-lamp-samba-ansible

1. OBJETIVO
Automatizar con roles la instalacion y configuracion de Apache, PHP, phpMyAdmin, MariaDB, Samba, FTP (vsftpd) y PostgreSQL.

2. REQUISITOS
- Una maquina de control Linux con Ansible instalado y acceso de red por SSH.
- Una VM destino limpia con openSUSE Leap 16.0, SSH habilitado y un usuario con permisos sudo.
- Acceso a los repositorios de paquetes de openSUSE.
- En la maquina de control, instalar las colecciones: ansible-galaxy collection install -r requirements.yml
- Para crear la contrasena del usuario Linux/Samba/FTP, instalar passlib en el entorno de Python usado por Ansible (por ejemplo: python3 -m pip install passlib).

3. CONFIGURACION
1) Editar inventory/hosts.ini y reemplazar 192.0.2.10 por la IP actual de la VM openSUSE. No escribir alli usuarios, contrasenas ni claves privadas.
2) Ajustar group_vars/servidores.yml: postgresql_allowed_cidr debe ser la red autorizada para conectarse por PostgreSQL. Si se usa NAT en VMware, revisar tambien ftp_pasv_address y el reenvio de puertos.
3) El usuario SSH se proporciona al ejecutar Ansible con -u. Las contrasenas de SSH, sudo y servicios se solicitan durante la ejecucion y no se guardan en el repositorio.

4. EJECUCION
Desde la carpeta raiz del proyecto:

ansible-inventory --graph
ansible servidores -m ping -u USUARIO_SSH --ask-pass
ansible-galaxy collection install -r requirements.yml
ansible-playbook playbooks/site.yml -u USUARIO_SSH --ask-pass --become --ask-become-pass

Durante el playbook se solicitaran de forma oculta las contrasenas nuevas de PUBLICA, MariaDB y PostgreSQL. Usar valores seguros y guardarlos fuera del repositorio.

5. ROLES Y SERVICIOS
- apache: instala y habilita apache2.
- php: instala PHP y extensiones MariaDB/PostgreSQL.
- mariadb: instala MariaDB, establece clave root y crea base/usuario de aplicacion.
- phpmyadmin: instala phpMyAdmin.
- samba: crea el grupo/usuario Linux, el usuario Samba y el recurso compartido.
- ftp: instala vsftpd con acceso local de escritura y puertos pasivos 30000-30100.
- postgresql: instala PostgreSQL, define clave admin y usuario/base de aplicacion, habilita acceso remoto cifrado con TLS y SCRAM.

Recurso Samba: \\SERVIDOR\compartido (sustituir SERVIDOR por la IP/nombre del destino).
Ruta: /srv/samba/compartido. Usuario: PUBLICA. Mascara de archivos: 0660; directorios: 0770.
FTP: puerto 21/TCP y rango pasivo 30000-30100/TCP.
PostgreSQL: puerto 5432/TCP. La plantilla acepta solo conexiones TLS desde postgresql_allowed_cidr.

6. RED Y FIREWALL
Permitir en el firewall del destino y en la red de VMware segun corresponda: TCP 80 (HTTP), 445 (SMB), 21 y 30000-30100 (FTP), y 5432 (PostgreSQL). Para el acceso SMB antiguo por descubrimiento NetBIOS tambien pueden requerirse 137-139. Limitar estos puertos a la red de laboratorio.
Con NAT, crear reenvios de puertos a la IP privada de la VM. Para FTP pasivo, reenviar el rango completo y configurar ftp_pasv_address con la IP que alcanza el cliente. Si el cliente solo necesita red de VM, usar adaptador puente o host-only con conectividad entre cliente y servidor.

7. ARCHIVOS DEL EQUIPO Y ENTREGA
El archivo team_list.txt se entrega por separado en Classroom. No incluirlo en este repositorio publico ni poner telefonos o codigos de estudiante en README.

8. NOTAS DE SEGURIDAD
FTP sin TLS cifra usuario, contrasena y datos en transito; usarlo solo en el laboratorio solicitado. PostgreSQL usa certificado autofirmado de laboratorio, por lo que los clientes TLS deben confiar/configurarse para ese certificado. No reutilizar estas configuraciones para un servidor expuesto a Internet.
