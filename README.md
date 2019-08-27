# WordPress_LAMP_Ansible
## Ansible - This is an Ansible playbook which can be used to install Wordpress on a LAMP environment. 

### You may need to create a file named virtualhost.j2 with a default apache virtualhost entry and another file named wp-config.php.tmpl with a default WordPress wp-config entries. I have provided the model of both these files below.

```ansible
---
- name: 'installing wordpress'
  hosts: all
  become: yes
  vars:
    domain: www.example.com
    mysql_root: mysqlroot123
    mysql_database: wodpress
    mysql_user: wpuser
    mysql_password: wpuser
        
  tasks:
    
    - name: 'Lamp installation'
      yum:
        name:
            - php-mysql
            - httpd
            - mariadb-server
            - php
            - MySQL-python
            
    - name: 'creating virtual host'
      template:
        src: virtualhost.j2
        dest: /etc/httpd/conf.d/{{domain}}.conf
        
    - name: 'creating document root'
      file:
        path: /var/www/html/{{domain}}
        state: directory
        owner: apache
        group: apache
            
    - name: 'adding content to the document root'
      copy:
        content: '<h1> it Works...!!!</h1>'
        dest: /var/www/html/{{domain}}/index.html
        
    - name: 'Restarting services'
      service:
        name: "{{ item }}"
        state: restarted
      with_tems:
        - httpd
        - mariadb
        
    - name: 'mariadb root password change'
      ignore_errors: true
      mysql_user:
        login_user: root
        login_password: ''
        user: root
        password: "{{mysql_root}}"
        host_all: true
            
    - name: 'Removing anonymous users'
      mysql_user:
        login_user: root
        login_password: "{{mysql_root}}"
        user: ''
        state: absent
        host_all: true
            
    - name: 'Mariadb creating new database'
      mysql_db:
        login_user: root
        login_password: "{{mysql_root}}"
        name: "{{mysql_databse}}"
        state: present
            
    - name: 'Mariadb creating new user'
      mysql_user:
        login_user: root
        login_password: "{{mysql_root}}"
        user: "{{mysql_user}}"
        password: "{{mysql_password}}"
        state: present
        host: localhost
        priv: "{{mysql_database}}.*:ALL"
            
    - name: 'Downloading WordPress to /tmp'
      get_url:
        url: https://wordpress.org/wordpress-4.9.2.tar.gz
        dest: /tmp/wordpress.tar.gz
            
    - name: 'Extracting contents from wordpress'
      unarchive:
        src: /tmp/wordpress.tar.gz
        dest: /tmp/
        remote_src: yes
            
    - name: 'Copy wordpress content to document root'
      shell: "cp -r /tmp/wordpress/*  /var/www/html/{{domain}}"
        
    - name: 'Wordpress - Creating wp-config.php'
      template:
        src: wp-config.php.tmpl
        dest: /var/www/html/{{ domain }}/wp-config.php
          
    - name: 'Wordpress - Chaning Ownership'
      shell: 'chown -R apache:apache /var/www/html/{{domain}}'
        
    - name: 'Deleting wordpress from /tmp'
      file:
        path: "{{ item }}"
        state: absent
      with_items:
        - /tmp/wordpress.tar.gz
        - /tmp/wordpress/
        
    - name: "LampStack - Restarting Services"
      service:
        name: "{{ item }}"
        state: restarted
      with_items:
        - httpd
        - mariadb
```

### virtualhost.j2

```
<virtualhost *:80>
  servername {{ domain }}
  documentroot /var/www/html/{{domain}}
  directoryindex index.php index.html info.html info.php
</virtualhost>
```

### wp-config.php.tmpl

```
<?php

define( 'DB_NAME', '{{ mysql_database }}' );

define( 'DB_USER', '{{ mysql_user }}' );

define( 'DB_PASSWORD', '{{ mysql_password }}' );

define( 'DB_HOST', 'localhost' );

define( 'DB_CHARSET', 'utf8' );

define( 'DB_COLLATE', '' );


define( 'AUTH_KEY',         'put your unique phrase here' );
define( 'SECURE_AUTH_KEY',  'put your unique phrase here' );
define( 'LOGGED_IN_KEY',    'put your unique phrase here' );
define( 'NONCE_KEY',        'put your unique phrase here' );
define( 'AUTH_SALT',        'put your unique phrase here' );
define( 'SECURE_AUTH_SALT', 'put your unique phrase here' );
define( 'LOGGED_IN_SALT',   'put your unique phrase here' );
define( 'NONCE_SALT',       'put your unique phrase here' );


$table_prefix = 'wp_';

define( 'WP_DEBUG', false );

if ( ! defined( 'ABSPATH' ) ) {
	define( 'ABSPATH', dirname( __FILE__ ) . '/' );
}

require_once( ABSPATH . 'wp-settings.php' );
```

## You can call the domain using /etc/hosts file after running this playbook to get the WordPress install screen.
