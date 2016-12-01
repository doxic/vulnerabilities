# Webserver with TLS and Heartbleed

## Base Install
Using ubuntu-14.04.4-server-amd64.iso
User: dominic
PW: access4bbw2get
Use entire disk LVM

    sudo sed -i 's|http://archive|http:ch.archive//|g' /etc/apt/sources.list
    sudo apt-get update
	sudo apt-get upgrade
	sudo apt-get install openssh-server

### Prep for building packages

    sudo apt-get install build-essential checkinstall git
	
Setting rights

    sudo chown $USER /usr/local/src
	sudo chmod u+rwx /usr/local/src
	
	
## Webserver

Installing Nginx with vulnerable OpenSSL [https://www.openssl.org/source/old/1.0.1/]

    cd /usr/local/src
    curl http://nginx.org/download/nginx-1.11.1.tar.gz | tar xvz
    curl https://www.openssl.org/source/old/1.0.1/openssl-1.0.1f.tar.gz | tar xvz
	git clone https://github.com/aperezdc/ngx-fancyindex.git ngx-fancyindex
	
	cd nginx
	
	./configure \
	--user=nginx \
	--group=nginx \
	--prefix=/etc/nginx \
	--sbin-path=/usr/sbin/nginx \
	--conf-path=/etc/nginx/nginx.conf \
	--pid-path=/var/run/nginx.pid \
	--lock-path=/var/run/nginx.lock \
	--error-log-path=/var/log/nginx/error.log \
	--http-log-path=/var/log/nginx/access.log \
	--with-debug  \
	--with-file-aio  \
	--with-http_realip_module  \
	--with-http_ssl_module \
	--with-openssl=/usr/local/src/openssl-1.0.1f \
	--without-mail_pop3_module \
	--without-mail_imap_module \
	--without-mail_smtp_module \
	--without-http_scgi_module \
	--without-http_uwsgi_module \
	--without-http_fastcgi_module \
	--without-http_split_clients_module \
	--without-http_map_module \
	--without-http_geo_module \
	--without-http_ssi_module \
	--without-http_rewrite_module \
	--without-http_gzip_module \
	--add-module=/usr/local/src/ngx-fancyindex
	
	make
	sudo make install

create system account if not existent

    id -u nginx &>/dev/null && echo "User exists" || sudo useradd -r nginx

download init.d script and make executable

	sudo wget https://raw.githubusercontent.com/JasonGiedymin/nginx-init-ubuntu/master/nginx -O /etc/init.d/nginx
	sudo chmod +x /etc/init.d/nginx
	
	sudo /usr/sbin/update-rc.d -f nginx defaults
	
Enable fancy index 
    vim /etc/nginx/def
	location / {
		fancyindex on;              # Enable fancy indexes.
		fancyindex_exact_size off;  # Output human-readable file sizes.
		root html/m184;
	}
	
	
## Setup Insecure env

created a self signed certificate

	openssl req -x509 -nodes -days 1825 -newkey rsa:1024 -keyout /etc/nginx/cert.key -out /etc/nginx/cert.pem
	
