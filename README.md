# Desafio-1
Aceleração SRE


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
    
    - vim /etc/php.ini
      - Remova o comentário da linha cgi.fix_pathinfo e altere o valor para 0.
      - memory_limit = 512M
      - max_execution_time = 1800
      - zlib.output_compression = On
      - session.save_path = "/var/lib/php/session"
    
    - vim /etc/php-fpm.d/www.conf
    
       - user = nginx
       - group = nginx
       - listen = /var/run/php/php-fpm.sock
       - listen.owner = nginx
       - listen.group = nginx
       - listen.mode = 0660
         
          * Descomentar as linhas
           - env[HOSTNAME] = $HOSTNAME
           - env[PATH] = /usr/local/bin:/usr/bin:/bin
           - env[TMP] = /tmp
           - env[TMPDIR] = /tmp
           - env[TEMP] = /tmp
           
* Crie um novo diretório para o caminho da sessão e altere o usuário e o grupo
     - mkdir -p /var/lib/php/session/
     - chown -R nginx:nginx /var/lib/php/session/
     
* Crei um novo diretorio php socket
     - mkdir -p /run/php/
     - chown -R nginx:nginx /run/php/
     
Concluido as configurações do PHP
systemctl start php-fpm && systemctl enable php-fpm

Instalando MySQL
 - yum localinstall https://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm
 - yum -y install mysql-community-server
 - systemctl start mysqld
	- systemctl enable mysqld
	- cat /var/log/mysqld.log | grep 'password'
	- mysql_secure_installation
	- mysqladmin -u root -p version
	- mysql -u root -p
 
 
 
 * Criando os bancos para o magento e wordpress;
   - create database magentodb;
   - create user magentouser@localhost identified by 'password';
   - grant all privileges on magentodb.* to magentouser@localhost identified by 'password';
   - flush privileges;

   - create database wordpressdb;
   - create user wordpressuser@localhost identified by 'password';
   - grant all privileges on wordpressdb.* to wordpressuser@localhost identified by 'password';
   - flush privileges;
   
   Magento
   
    - Instalando
     
        Instalando o php composer
           - curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/bin --filename=composer
           - composer -V
        
        Download e extração do arquivo
           - cd /var/www/html
           - wget https://github.com/magento/magento2/archive/2.1.zip
           - unzip 2.1.zip
           - mv magento2-2.1 magento2
           
      
      
 
 
 
 
 
 
 
 
 
