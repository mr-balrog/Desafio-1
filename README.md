# Aceleração SRE
Desafio 1

Criando instancia na AWS EC2

 - Selecionar a imagem: centos 7
 - Tipo da instancia: t2.micro
 - Selecionar uma chave 
 Iniciar(launch)
 
 
Instalando o software
 - yum -y update
 - yum -y install git wget curl nano unzip vim net-tools openssh-server htop
 
Instalando Nginx
- yum install epel-release
- sudo yum -y install nginx
- service nginx start && systemctl enable nginx
 
PHP-FPM

Instalando
  - rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
  - yum -y install php70w-fpm php70w-mcrypt php70w-curl php70w-cli php70w-mysql php70w-gd php70w-xsl php70w-json php70w-intl php70w-pear php70w-devel php70w-mbstring php70w-zip php70w-soap
  
Configurando
    
 vim /etc/php.ini
 Remova o comentário da linha cgi.fix_pathinfo e altere o valor para 0.
 memory_limit = 512M
 max_execution_time = 1800
 zlib.output_compression = On
 session.save_path = "/var/lib/php/session"
    
vim /etc/php-fpm.d/www.conf
    
 user = nginx
 group = nginx
 listen = /var/run/php/php-fpm.sock
 listen.owner = nginx
 listen.group = nginx
 listen.mode = 0660
         
 Descomentar as linhas
 env[HOSTNAME] = $HOSTNAME
 env[PATH] = /usr/local/bin:/usr/bin:/bin
 env[TMP] = /tmp
 env[TMPDIR] = /tmp
 env[TEMP] = /tmp
           
Criando um novo diretório para o caminho da sessão e alterando o usuário e o grupo
mkdir -p /var/lib/php/session/
chown -R nginx:nginx /var/lib/php/session/
     
Criando um novo diretorio php socket
mkdir -p /run/php/
chown -R nginx:nginx /run/php/
     
Concluido as configurações do PHP
systemctl start php-fpm && systemctl enable php-fpm

Instalando MySQL
yum localinstall https://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm
yum -y install mysql-community-server
systemctl start mysqld
systemctl enable mysqld
cat /var/log/mysqld.log | grep 'password'
mysql_secure_installation
mysqladmin -u root -p version
mysql -u root -p
 
Criando os bancos para o magento e wordpress;
create database magentodb;
create user magentouser@localhost identified by 'password';
grant all privileges on magentodb.* to magentouser@localhost identified by 'password';
flush privileges;

create database wordpressdb;
create user wordpressuser@localhost identified by 'password';
grant all privileges on wordpressdb.* to wordpressuser@localhost identified by 'password';
flush privileges;
   
Magento

Instalando o php composer
curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/bin --filename=composer
composer -V
        
Download e extração do arquivo
cd /var/www/html
wget https://github.com/magento/magento2/archive/2.1.zip
unzip 2.1.zip
mv magento2-2.1 magento2
       
Instalando dependencias php
cd magento2
composer install -v
	
Configurando virtual host magento
cd /etc/nginx/
vim conf.d/magento.conf
	
upstream fastcgi_backend {
 server  unix:/run/php/php-fpm.sock;
	}
 
server {
 
   listen 80;
   server_name site.lojamagento.cf;
   set $MAGE_ROOT /var/www/html/magento2;
   set $MAGE_MODE developer;
   include /var/www/html/magento2/nginx.conf.sample;
	}
	
nginx -t
systemctl restart nginx
      
Instalando Magento

cd /var/www/html/magento2

bin/magento setup:install --backend-frontname="adminlogin" \
--key="biY8vdWx4w8KV5Q59380Fejy36l6ssUb" \
--db-host="localhost" \
--db-name="magentodb" \
--db-user="magentouser" \
--db-password="password" \
--language="en_US" \
--currency="USD" \
--timezone="America/New_York" \
--use-rewrites=1 \
--use-secure=0 \
--base-url="http://site.lojamagento.cf" \
--admin-user=adminuser \
--admin-password=admin123@ \
--admin-email=admin@newmagento.com \
--admin-firstname=admin \
--admin-lastname=user \
--cleanup-database
      
Alterando usuário e dando permissão
chmod 700 /var/www/magento2/app/etc
chown -R nginx:nginx /var/www/magento2
 
 
Configurando cron magento

crontab -u nginx -e
 
* * * * * /usr/bin/php /var/www/html/magento2/bin/magento cron:run | grep -v "Ran jobs by schedule" >> /var/www/html/magento2/var/log/magento.cron.log
* * * * * /usr/bin/php /var/www/html/magento2/update/cron.php >> /var/www/html/magento2/var/log/update.cron.log
* * * * * /usr/bin/php /var/www/html/magento2/bin/magento setup:cron:run >> /var/www/html/magento2/var/log/setup.cron.log
 
 Configurando selinux e firewalld
 sestatus
 yum -y install policycoreutils-python
 
 cd /var/www/html
 
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/html/magento2(/.*)?'
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/html/magento2/app/etc(/.*)?'
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/html/magento2/var(/.*)?'
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/html/magento2/pub/media(/.*)?'
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/html/magento2/pub/static(/.*)?'
restorecon -Rv '/var/www/html/magento2/'
 
Acessando Magento

Abri a url site.lojamagento.cf no navegador
