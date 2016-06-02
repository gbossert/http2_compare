# Recipes: building HTTP2 servers

## SSL Certificates

HTTP2 relies on an SSL tunnel and thus on SSL certificates.
Certificates used as part of this analysis are stored in ```$GIT_REPO/confis/certs```.

The following command and parameters was used to issue the certificates.

```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout http2.key -out http2.crt
Country Name (2 letter code) [AU]:FR
State or Province Name (full name) [Some-State]:Bretagne
Locality Name (eg, city) []:Rennes
Organization Name (eg, company) [Internet Widgits Pty Ltd]:    
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:localhost
Email Address []:
```

## Apache 2.4.20


### nghttp 1.10.0

Sources are available in `$GIT_REPO/servers/nghttp2-1.10.0.tar.gz`.

```
cd $GIT_REPO
mkdir -p servers/compiled/nghttp
cd servers/compiles/nghttp
tar -zxvf ../../sources/nghttp2-1.10.0.tar.gz
cd nghttp2-1.100.0
./configure
make
sudo make install
```

### httpd 2.4.20
Sources are available in `$GIT_REPO/servers/httpd-2.4.20.tar.gz`

```
cd $GIT_REPO
mkdir -p servers/compiled/httpd
cd servers/compiled/httpd
tar -zxvf ../../sources/httpd-2.4.20.tar.gz
cd httpd-2.4.20.tar.gz
./configure --enable-http2 --prefix=$GIT_REPO/servers/compiled/httpd
make
make install
```

Finally, configuration file must be deployed in the install directory.
Paths in the configuration file must be adpated to your environment
```
#> ln -s $GIT_REPO/servers/configs/httpd/config/httpd.conf ..
```

The following command is used to start the process
```
./bin/httpd -DFOREGROUND
```

It listens to port *9901*

## h2o 1.7.1

Sources are available in ```$GIT_REPO/servers/sources/h2o-1.7.1.zip```

Commands shown below were used to obtain the h2o server binary

```
cd $GIT_REPO
mkdir -p servers/compiled/h2o
cd servers/compiled/h2o
unzip ../../sources/h2o-1.7.1.zip
cd h2o-1.7.1/
sudo apt-get install cmake
cmake -DWITH_BUNDLED_SSL=on .
make
```

The following command is used to start the process (Paths in the configuration file must be adpated to your environment)
```
./h2o -c ../../../configs/h2o/h2o.conf
```

It listens to port *9902*


## Tomcat 9.0.0.M4

Sources are available in ```$GIT_REPO/servers/sources/apache-tomcat-9.0.0.M4.tar.gz```

Commands shown below were used to obtain the server binaries well configured 

```
cd $GIT_REPO
mkdir -p servers/compiled/tomcat
cd servers/compiled/tomcat
tar -zxvf ../../sources/apache-tomcat-9.0.0.M4.tar.gz 
cd apache-tomcat-9.0.0.M4/
sudo apt-get install openjdk-8-jre-headless
ln -s ../../../../../website_sources/theuserisdrunk.com/ webapps/
ln -s ../../../../../website_sources/theuserismymom.com/ webapps/
rm conf/server.xml
ln -s ../../../../confis/tomcat/server.xml .

```

It also requires the APR JNI stuff so that ssl is provided by openssl (if i understood correctly).
Extract `tomcat-native-1.2.5-src` and install with the following commands the *native* content

```
sudo apt-get install openjdk-8-jdk-headless
cd $GIT_REPO
mkdir -p servers/compiled/tomcat-native
cd servers/compiled/tomcat-native
tar -zxvf ../../sources/tomcat-native-1.2.5-src.tar.gz
cd tomcat-native-1.2.5-src/native
./configure --with-apr=/usr/bin/apr-1-config --with-java-home=$JAVA_HOME --with-ssl=yes --prefix=/usr/
make
sudo make install

```

The following command is used to start the process
```
./bin/catalina.sh run
```

It listens to port *9903*


## Nginx 1.9.15

Sources are available in ```$GIT_REPO/servers/sources/nginx-1.9.15.tar.gz```

Commands shown below were used to obtain the server binaries well configured 
```
cd $GIT_REPO
mkdir -p servers/compiled/nginx
cd servers/compiled/nginx
tar -zxvf ../../sources/nginx-1.9.15.tar.gz
cd nginx-1.9.15/
./configure --with-http_ssl_module --with-http_v2_module --prefix=$GIT_REPO/servers/compiled/nginx/
make
make install
ln -s ../../../../confis/nginx/nginx.conf conf/nginx.conf

```

The following command is used to start the process
```
./sbin/nginx
```

It listens to port *9904*


## Testing the http2 connection


The following command is used to test web server HTTP2 h2c *without* prior-knowledge :
```
curl -v --http2 http://localhost:$PORT/theuserisdrunk.com/
```

The following command is used to test web server HTTP2 h2c *with* prior-knowledge :
```
nghttp -v http://127.0.0.1:9904/theuserisdrunk.com/
```



