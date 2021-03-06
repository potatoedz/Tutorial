# TUTORIAL KISI-KISI LKS Provinsi Jawa Barat
## SpeedRun Deploy Web App CodeIgniter ke AWS EC2 (AMAZON LINUX MACHINE IMAAGE)
---  
### EC2
    sudo yum update -y  
    sudo amazon-linux-extras enable php7.4 -y  
    sudo yum clean metadata  
    sudo yum install -y php-{pear,cgi,common,curl,mbstring,gd,mysqlnd,gettext,bcmath,json,xml,fpm,intl,zip,imap}        
    sudo yum install -y httpd mariadb-server
    
**Setting permission/izin file.**  

    sudo usermod -a -G apache ec2-user
    sudo su
    su ec2-user
    sudo chown -R ec2-user:apache /var/www  
    sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;  
    find /var/www -type f -exec sudo chmod 0664 {} \;    
    
**Download Web App anda menggunakan s3**

    cd /var/www/html 
    aws s3 cp s3://bucketanda/webanda.zip web
    unzip web  
    rm web
    mv web ../
    cd ../
    rmdir html
    mv web html
    cd html

**Download Web App anda menggunakan Git**

    cd /var/www
    rmdir html
    sudo yum install -y git
    git clone https://github.com/potatoedz/WebApp /var/www/html

**Install PHPMyAdmin**

    cd html
    wget https://www.phpmyadmin.net/downloads/phpMyAdmin-latest-all-languages.tar.gz
    mkdir phpMyAdmin && tar -xvzf phpMyAdmin-latest-all-languages.tar.gz -C phpMyAdmin --strip-components 1
    rm phpMyAdmin-latest-all-languages.tar.gz
    chmod -R 777 writable uploads
    sudo systemctl start mariadb
    sudo mysql_secure_installation
    sudo systemctl start httpd

**.env configurations.**  

    $ nano .env
    #--------------------------------------------------------------------
    # APP
    #--------------------------------------------------------------------
    app.baseURL = 'http://13.250.55.208/' (isi ip web server anda)
    #--------------------------------------------------------------------
    # DATABASE
    #--------------------------------------------------------------------
    database.default.hostname = localhost (atau) rds.ipaddress.host (bila menggunakan rds)
    database.default.database = database anda
    database.default.username = root (sesuai saat menjalankan mysql_secure_installation)
    database.default.password = root (sesuai saat menjalankan mysql_secure_installation)
    database.default.DBDriver = MySQLi (mariadb)

**Set host:**  

    $ sudo nano /etc/httpd/conf/httpd.conf   


**scroll ke bawah dan ganti **  

```blade
<Directory "/var/www/html">
    # The Options directive is both complicated and important.  Please see
    # http://httpd.apache.org/docs/2.4/mod/core.html#options
    # for more information.
    #
    Options Indexes FollowSymLinks

    #
    # AllowOverride controls what directives may be placed in .htaccess files.
    # It can be "All", "None", or any combination of the keywords:
    #   Options FileInfo AuthConfig Limit
    #
    AllowOverride All

    #
    # Controls who can get stuff from this server.
    #
    Require all granted
</Directory>
$ sudo systemctl restart httpd 
```  
---

### Setting database menggunakan phpMyAdmin
- Buka phpMyAdmin di browser (http://IP.WEB.SERVER.ANDA/phpMyAdmin)
  - Login menggunakan credential yang di konfigurasikan saat menjalankan mysql_secure_installation
  - Klik New di sidebar kiri untuk menambahkan database baru, nama harus disesuaikan dengan settingn database di .env
  - setelah itu impor database anda (sebelumnya harus sudah di export ke pc anda) 
### Menghubungkan phpMyAdmin Web Server ke RDS
- ```Sebelum menghubungkan RDS ke web server anda pastikan bahwa security group RDS mengizinkan inbound traffic type mysql/aurora (port 3306) dari Web Server anda```
- Buka Setup phpMyAdmin di browser (http://IP.WEB.SERVER.ANDA/phpMyAdmin/setup)
  - New Server
    - Basic setting -> `Server Hostname = database-1.crsgedzchyrc.ap-southeast-1.rds.amazonaws.com (endpoint RDS Yang sudah dibuat)`
    - Authentication -> `Config Authentication -> User = admin Password = admin123`
    - Apply
  - Display
    - Copy & paste ke EC2
  - Di Instance Web Server
  
  ```
  $ cd /var/www/html/phpMyAdmin
  $ sudo nano config.inc.php
  PASTE TEXT YANG DI COPY
  ```
- Ganti konfigurasi Database Web Server
  - sudo nano /var/www/html/.env
  ```
  database.default.hostname = databaseserver.crsgedzchyrc.ap-southeast-1.rds.amazonaws.com (sesuaikan dengan endpoint anda)
  database.default.database = database anda
  database.default.username = admin (sesuai saat menjalankan instance RDS Amazon)
  database.default.password = admin123 (sesuai saat menjalankan instanceRDS Amazon)
  database.default.DBDriver = MySQLi (jika menggunakan mariadb)
  ```
### Menerapkan Elastic File System (EFS)
  - jalankan EFS dan pastikan bahwa ada Mount Point yang bisa dimasuki oleh instance Web Server
  - Tambah security group baru yang menerima all incoming traffic dari Security-Group Web Server
  - Mount EFS di Instance Web Server
  ```
  cd /mnt
  sudo mkdir efs
  sudo mount -t efs -o tls fs-NOFSMILIKANDA:/ efs
  ```  

