# Gazelle
###### tags: `CyLab`
###### tags: `Gazelle`
![](https://i.imgur.com/P9IIZuO.png)

Gazelle is a test bed aimed at testing the interoperability of eHealth information systems.

### Gazelle components
* [SSO](Gazelle SSO.md)
* [Gazelle Test Management](Gazelle Test Management.md)
* [Proxy](Gazelle Proxy.md)
* [EVSClient](EVSClient.md)
* [Demographic Data Server](Demographic Data Server.md)
* [Gazelle Fhir Validator](Gazelle FHIR Validator.md)
* [Gazelle HL7 Validator](Gazelle HL7 Validator.md)

### How do Gazelle works
all gazelle subsystem run's on AP servers (application servers),including Tomcat Jboss Wildfly

## Setup

### Basic dependency & tools
* Installation

```
deb [trusted=yes] http://archive.debian.org/debian/ stretch main non-free contrib 
deb-src [trusted=yes] http://archive.debian.org/debian/ stretch main non-free contrib 
deb [trusted=yes] http://archive.debian.org/debian-security/ stretch/updates main non-free contrib
```
```
sudo apt install -y curl git vim unzip gnupg apt-transport-https postgresql-9.6 dirmngr --install-recommends
```

### Java
* Installation
```
# add Azul public key
sudo apt-key adv \
  --keyserver hkp://keyserver.ubuntu.com:80 \
  --recv-keys 0xB1998361219BD9C9
  
# download Azul repo
curl -O https://cdn.azul.com/zulu/bin/zulu-repo_1.0.0-3_all.deb

# install repo
sudo apt-get install -y ./zulu-repo_1.0.0-3_all.deb

sudo apt-get update

sudo apt-get install -y zulu7-ca-jre-headless zulu8-ca-jre-headless openjdk-8-jdk
```

### Tomcat8
* Installation
```
sudo apt install -y tomcat8
```
* Set java profile
```
# set JAVA_HOME in /etc/init.d/tomcat8
JAVA_HOME=/usr/lib/jvm/zulu8-ca-amd64/jre
```
* Set port and protocol
```
#add the following line in /var/lib/tomcat8/conf/server.xml (comment original)
<Connector port="8209"
           protocol="AJP/1.3"
           redirectPort="8443"
           secretRequired="false" />
```
* Delete default content
```
sudo rm -rf /var/lib/tomcat8/webapps/ROOT
```
* Reload service
```
sudo systemctl daemon-reload
```

### Jboss
* Download
```
# download jboss
wget -nv -O /tmp/jboss-as-7.2.0.Final.zip https://gazelle.ihe.net/jboss7/jboss-as-7.2.0.Final.zip

# download script
wget -nv -O /tmp/init.d_jboss7 https://gazelle.ihe.net/jboss7/init.d_jboss7
```
* Set user and group
```
sudo useradd jboss
sudo groupadd jboss-admin
sudo adduser jboss jboss-admin
```
* Install jboss to /usr/local
```
cd /usr/local
sudo mv /tmp/jboss-as-7.2.0.Final.zip .
sudo unzip ./jboss-as-7.2.0.Final.zip
sudo ln -s jboss-as-7.2.0.Final jboss7
sudo chown -R jboss:jboss-admin /usr/local/jboss7/
sudo chmod -R 755 /usr/local/jboss-as-7.2.0.Final/
sudo mkdir /var/log/jboss7
sudo chown -R jboss:jboss-admin /var/log/jboss7/
sudo chmod -R g+w /var/log/jboss7/
```
* Install the init script and make it start at system startup
```
sudo mv /tmp/init.d_jboss7 /etc/init.d/jboss7
sudo chmod +x /etc/init.d/jboss7
sudo update-rc.d jboss7 defaults
```
* Set java profile
```
# set JAVA_HOME in /etc/init.d/jboss7
JAVA_HOME=/usr/lib/jvm/zulu7-ca-amd64

# reload service
sudo systemctl daemon-reload
```
* Deployment
```
# add the following line in /usr/local/jboss7/standalone/configuration/standalone.xml
# replace ${seamName} ${databaseName} ${jdbc.user} ${jdbc.password}

<datasource jta="true" enabled="true" use-java-context="true" pool-name="${seamName}" jndi-name="java:jboss/datasources/${seamName}">
    <connection-url>jdbc:postgresql:${databaseName}</connection-url>
    <driver>postgresql</driver>
    <security>
        <user-name>${jdbc.user}</user-name>
        <password>${jdbc.password}</password>
    </security>
</datasource>
```
### Wildfly
* Download
```
# download wildfly
wget -nv -O /tmp/wildfly-10.0.0.Final.zip http://gazelle.ihe.net/wildfly10/wildfly-10.0.0.Final.zip

# download script
wget -nv -O /tmp/init.d_wildfly10 https://gazelle.ihe.net/wildfly10/init.d_wildfly10
```
* Install jboss in the /usr/local folder
```
cd /usr/local
sudo mv /tmp/wildfly-10.0.0.Final.zip .
sudo unzip ./wildfly-10.0.0.Final.zip
sudo ln -s wildfly-10.0.0.Final wildfly10
sudo rm -rf wildfly-10.0.0.Final.zip
sudo chown -R jboss:jboss-admin /usr/local/wildfly-10.0.0.Final
sudo chmod -R 755 /usr/local/wildfly-10.0.0.Final
sudo mkdir /var/log/wildfly10/
sudo chown -R jboss:jboss-admin /var/log/wildfly10/
sudo chmod -R g+w /var/log/wildfly10/
```
* Install the init script and make it start at system startup
```
sudo mv /tmp/init.d_wildfly10 /etc/init.d/wildfly10
sudo chmod +x /etc/init.d/wildfly10
sudo chown root:root /etc/init.d/wildfly10
sudo update-rc.d wildfly10 defaults
```
* Update Wildfly 10 postgresql jdbc driver
```
# stop wildfly
sudo systemctl stop wildfly10.service

# create driver folder
sudo mkdir -p /usr/local/wildfly10/modules/system/layers/base/org/postgresql/main
cd /usr/local/wildfly10/modules/system/layers/base/org/postgresql/main

# download driver
sudo wget https://gazelle.ihe.net/wildfly10/postgresql-42.2.5.jar
sudo wget https://gazelle.ihe.net/nexus/service/local/repositories/central/content/com/fasterxml/jackson/core/jackson-core/2.10.0/jackson-core-2.10.0.jar
sudo wget https://gazelle.ihe.net/nexus/service/local/repositories/central/content/com/fasterxml/jackson/core/jackson-annotations/2.10.0/jackson-annotations-2.10.0.jar
sudo wget https://gazelle.ihe.net/nexus/service/local/repositories/central/content/org/wso2/orbit/com/fasterxml/jackson/core/jackson-databind/2.11.0.wso2v1/jackson-databind-2.11.0.wso2v1.jar
sudo chown jboss:jboss-admin *.jar
sudo chmod 775 *.jar

# create & edit module.xml
<module xmlns="urn:jboss:module:1.1" name="org.postgresql">
     <resources>
        <resource-root path="postgresql-42.2.5.jar"/>
        <resource-root path="jackson-annotations-2.10.0.jar"/>
        <resource-root path="jackson-core-2.10.0.jar"/>
        <resource-root path="jackson-databind-2.11.0.wso2v1.jar"/>
     </resources>
     <dependencies>
         <module name="javax.api"/>
         <module name="javax.transaction.api"/>
     </dependencies>
</module>

# clean jboss file
sudo rm -rf /usr/local/wildfly10/standalone/tmp/
sudo rm -rf /usr/local/wildfly10/standalone/data/

# start wildfly
sudo systemctl start wildfly10.service
```
* Deployment
```
# add the following line in /usr/local/wildfly10/standalone/configuration/standalone.xml
# replace ${seamName}
${databaseName} ${jdbc.user} ${jdbc.password}

<datasource jndi-name="java:jboss/datasources/${seamName}" pool-name="${seamName}" enabled="true" use-java-context="true">
    <connection-url>jdbc:postgresql://localhost:5432/${databaseName}</connection-url>
    <driver>postgresql</driver>
    <security>
        <user-name>${jdbc.user}</user-name>
        <password>${jdbc.password}</password>
    </security>
</datasource>

# add postgresql dirver under
<driver name="postgresql" module="org.postgresql">
    <driver-class>org.postgresql.Driver</driver-class>
    <xa-datasource-class>org.postgresql.xa.PGXADataSource</xa-datasource-class>
</driver>
```

### Apache2
* Installation
```
sudo apt install -y apache2 apache2-dev libcurl4-openssl-dev dh-autoreconf libpcre3-dev libssl-dev
```
* Compile mod_auth_cas
```
git clone https://github.com/apereo/mod_auth_cas.git
cd mod_auth_cas/
sudo autoreconf -iv 
sudo ./configure --with-apxs=/usr/bin/apxs2 
sudo make  
sudo make install
su
mkdir /var/cache/apache2; mkdir /var/cache/apache2/mod_auth_cas; chown www-data:www-data /var/cache/apache2/mod_auth_cas/
```

* Apache2 conf example
```
LoadModule auth_cas_module /usr/lib/apache2/modules/mod_auth_cas.so

<IfModule mod_ssl.c>
<VirtualHost *:443>
    ServerAdmin webmaster@localhost
    ServerName FQDN
    Header always set Strict-Transport-Security "max-age=63072000; includeSubdomains;"

    DocumentRoot /var/www/html
    CASCookiePath /var/cache/apache2/mod_auth_cas/
    CASLoginURL https://FQDN/sso/login
    CASValidateURL https://FQDN/sso/serviceValidate
<Location /index.html>
    Authtype CAS
    require valid-user
</Location>
...
</VirtualHost>
</IfModule>
```



https://biteeniu.github.io/ssl/convert_pem_to_jks/