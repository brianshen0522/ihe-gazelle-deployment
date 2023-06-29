# Gazelle Proxy
###### tags: `CyLab`
###### tags: `Gazelle`

## Introduction
### What is Proxy?
Proxy is the part of the Gazelle testbed which is used to capture the messages exchanged between two systems under test. This tool is also bind to the EVSClient in order to validate the messages stored in the Proxy in a very simple way.

### Dependency
Gazelle Proxy runs under Jboss AS 7.2.0 with a postgresql PostGresql 9.1 or higher database. 

## Installation
* Download
```
# install dcmtk
sudo apt install -y dcmtk

# find and download the last version ear and sql https://gazelle.ihe.net/nexus/index.html#nexus-search;quick~gazelle-proxy.ear
wget https://gazelle.ihe.net/nexus/service/local/repositories/releases/content/net/ihe/gazelle/proxy/gazelle-proxy-ear/5.0.6/gazelle-proxy-ear-5.0.6.ear
wget https://gazelle.ihe.net/nexus/service/local/repositories/releases/content/net/ihe/gazelle/proxy/gazelle-proxy-ear/5.0.6/gazelle-proxy-ear-5.0.6-sql.zip
unzip -d sql gazelle-proxy-ear-5.0.6-sql.zip
cd sql/
```

* Database creation and initialization
```
# create database user and database
sudo -u postgres psql
CREATE DATABASE "gazelle-proxy" OWNER gazelle ENCODING "UTF-8";
\q

psql -U gazelle gazelle-proxy < schema-X.X.X.sql
psql -U gazelle gazelle-proxy < init-X.X.X.sql
```

* Proxy file configuration
```
sudo mkdir /opt/proxy
sudo mkdir /opt/proxy/DICOM
sudo chown -R jboss:jboss-admin /opt/proxy
sudo chmod -R 775 /opt/proxy
```

* Deployment
```
# Jboss configuration
${seamName} = gazelle-proxyDS
${databaseName} = gazelle-proxy

# stop jboss
sudo systemctl stop jboss7.service

# copy ear to jboss deployment folder
sudo cp gazelle-proxy-ear-5.0.6.ear  /usr/local/jboss7/standalone/deployments/gazelle-proxy.ear

# start jboss
sudo systemctl start jboss7.service
```

* Configure Apache2
```
<Location /proxy>
   ProxyPass        ajp://localhost:8109/proxy
   ProxyPassReverse ajp://localhost:8109/proxy
</Location>
 
# Check that apache2 configuration is still ok
sudo apache2ctl configtest

#If everything is ok you can then restart apache2
sudo apache2ctl restart
```