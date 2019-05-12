Installation
============

Install: apt-get install python-flask python-crypto 
Install: pip install uwsgi

---------------
1.    $ git clone https://github.com/khabib97/d-note-bcc /build   
$ cd /build
$ python setup.py install
$ cd /
$ rm -rf build 

2. Create d-note.ini and put /etc/dnote folder (create dnote folder)
d-note.ini  

[DEFAULT]
app = dnote
config_path = /dnote
data_dir = /dnote/db

[dnote]

-----------------
3. Create /dnote directory 
4. Then run /usr/local/bin/generate_dnote_hashes.sh file 
Or 
   $ generate_dnote_hashes
5.
   $ uwsgi --master --processes 1 --threads 1 --http-socket :8080 \  
       --manage-script-name \  
       --chdir /usr/local/lib/python2.7/dist-packages/dnote-1.0.1-py2.7.egg/dnote \
       --mount /=__init__:DNOTE

Now you can browse this in your internal network, like https://localhost:8080 , https://127.0.0.1:8080  ...

To browse this appliction form outsider network we can use vitual host

1. create a file name dnote-file.conf
2. Put it into /ect/apache2/sites-enabled/ directory
3. enable proxy, proxy_http, rewrite , ssl
    $ a2enmod proxy , proxy_http , rewirte ,  ssl
    $ a2ensite example.com

<VirtualHost *:80>

 ServerName example.com

 RewriteEngine On
 RewriteCond %{HTTPS} off
 RewriteRule (.*) https://%{SERVER_NAME}/$1 [R,L]

</VirtualHost>

<VirtualHost *:443>
    ProxyPreserveHost On
    ProxyRequests Off
    ServerName example.com
    ServerAlias example.com

    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
    SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
    SSLHonorCipherOrder On

    ProxyPass / http://localhost:8080/
    ProxyPassReverse / http://localhost:8080/

</VirtualHost>


Nginx Setup
-----------
Install uwsgi:

    # apt-get install uwsgi uwsgi-core uwsgi-extra uwsgi-plugin-python

Create a uwsgi.ini file in the directory with the application:

    # touch /var/www/dnote/uwsgi.ini

And add the following to that file (you can tweak these settings as required):

    [uwsgi]
    socket = 127.0.0.1:8081
    chdir = /python/path/site-packages/dnote-1.0.1-py2.7.egg/dnote
    plugin = python
    module = __init__:dnote
    processes = 4
    threads = 2
    stats = 127.0.0.1:9192
    uid = www-data
    gid = www-data
    logto = /var/log/dnote.log

You can now start the dnote application by running:

    # /usr/bin/uwsgi -c /var/www/dnote/uwsgi.ini

This will start uwsgi in the foreground.  To start it as a
daemon:

    # /usr/bin/uwsgi -d -c /var/www/dnote/uwsgi.ini

You may want to add this to an init or upstart script, see:
http://uwsgi-docs.readthedocs.org/en/latest/Management.html

Now lets configure nginx. A common example would be if you wanted it 
to be avaliable under http://yoursite.tld/dnote. To acheive this, add
the following to your sites config (again, you can tweak thsi as needed):

    location = /dnote { rewrite ^ /dnote/; }
    location /dnote/ { try_files $uri @dnote; }
    location @dnote {
        include uwsgi_params;
        uwsgi_param SCRIPT_NAME /dnote;
        uwsgi_modifier1 30;
        uwsgi_pass 127.0.0.1:8081;
    }

And tada, restart the Nginx server and you should have a working dnote setup.


Troubleshooting
---------------
If you are getting any internal service errors make sure to verify that the
files in /var/www/dnote/dnote are readable by the webserver, and that
/var/www/dnote/dnote/data/hashcash.db is writable as well.

If you have trouble getting the app to load using uwsgi, try setting up a
dnote.wsgi file (as in the Apache directions above) and using uwsgi-file
in the uwsgi.ini file instead of module, like this:

    [uwsgi]
    socket = 127.0.0.1:8081
    chdir = /python/path/site-packages/dnote-1.0.1-py2.7.egg/dnote
    plugin = python
    wsgi-file = /var/www/dnote.wsgi
    processes = 4
    threads = 2
    stats = 127.0.0.1:9192
    uid = www-data
    gid = www-data
    logto = /var/log/dnote.log
