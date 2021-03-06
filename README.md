# Ansible
Файлы к второму заданию
Ansible-tree:

/etc/ansible/
	- hosts
	- install-nginx-main.yml
	- install-php-main.yml
	- copy-php-file-main.yml
	- roles/
		- install-nginx/
			- tasks/
				- main.yml
		- install-php/
			- tasks/
				- install-php.yml
		- copy-php-file/
			- tasks/
				- copy-php-file.yml
	- files/ 
		- mail.php
		- default

# /etc/ansible/hosts
 
Creating the List of NGINX Servers

[nginx-php]
192.168.1.2 ansible_ssh_port=22 // Для подключению к серверу используеться SSH-ключ

Ansible playbook
# /etc/ansible/install-nginx-main.yml

- hosts: nginx-php  // Название хоста к которому будет обращаться Ansible в файле hosts
  sudo: yes // Права администратора
  tasks:  //В задании подключаем файл в котором описана последовательность действий
    - include: 'roles/install-nginx/tasks/install_nginx.yml'

$ nano etc/ansible/roles/install-nginx/tasks/install_nginx.yml

Содержимое файла install_nginx.yml взято с официального сайта NGINX для Ansible:

# etc/ansible/roles/install-nginx/tasks/install_nginx.yml
 
- name: NGINX | Adding NGINX signing key
  apt_key: url=http://nginx.org/keys/nginx_signing.key state=present
 
- name: NGINX | Adding sources.list deb url for NGINX
  lineinfile: dest=/etc/apt/sources.list line="deb http://nginx.org/packages/mainline/ubuntu/ trusty nginx"
 
- name: NGINX Plus | Adding sources.list deb-src url for NGINX
  lineinfile: dest=/etc/apt/sources.list line="deb-src http://nginx.org/packages/mainline/ubuntu/ trusty nginx"
 
- name: NGINX | Updating apt cache
  apt:
    update_cache: yes
 
- name: NGINX | Installing NGINX
  apt:
    pkg: nginx
    state: latest
 
- name: NGINX | Starting NGINX
  service:
    name: nginx
    state: started

Запуск ansible playbook:

$ ansible-playbook /etc/ansible/install-nginx-main.yml

Конфигурация файла настроек NGINX:

etc/ansible/files/default

server {
    listen 80 ;
    
    root /var/www/mail;
    index index.php index.html index.htm;

    server_name 192.168.1.99;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php5.0-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}

Копирование настроек для NGINX:

- name: copy-nginx-file in 
  copy:
    src: files/default  //Локальный путь к файлу для копирования на удаленный сервер
    dest: /etc/nginx/sites-available/default  //Удаленный абсолютный путь, в который должен быть скопирован файл

Установка PHP playbook:

$ nano php.yml

---
- hosts: nginx-php  // Название хоста к которому будет обращаться Ansible в файле hosts
  sudo: yes  // Права администратора
   tasks:  //В задании подключаем файл в котором описана последовательность действий
   - include: 'roles/install-nginx/tasks/install_nginx.yml'

# nano etc/ansible/roles/install-php/tasks/install_php.yml

  - name: install packages   
    apt: name={{ item }} update_cache=yes state=latest
    with_items:  // Список устанавливаемых пакетов
      - git
      - mcrypt
      - nginx
      - php5-cli
      - php5-curl
      - php5-fpm
      - php5-intl
      - php5-json
      - php5-mcrypt
      - php5-sqlite
      - sqlite3

  - name: ensure php5-fpm cgi.fix_pathinfo=0
    lineinfile: dest=/etc/php5/fpm/php.ini regexp='^(.*)cgi.fix_pathinfo=' line=cgi.fix_pathinfo=0
    notify:
      - restart php5-fpm
      - restart nginx

  - name: enable php5 mcrypt module
    shell: php5enmod mcrypt
    args:
      creates: /etc/php5/cli/conf.d/20-mcrypt.ini

  handlers:
    - name: restart php5-fpm
      service: name=php5-fpm state=restarted

    - name: restart nginx
      service: name=nginx state=restarted

Запуск ansible playbook:

$ ansible-playbook /etc/ansible/install-php-main.yml

Копирование файла mail.php:

$nano etc/ansible/roles/copy-php-file/tasks/copy-php-file.yml

- name: copy-php-file in 
  copy:
    src: /etc/ansible/files/mail.php  //Локальный путь к файлу для копирования на удаленный сервер
    dest: var/www/mail.php  //Удаленный абсолютный путь, в который должен быть скопирован файл
