# Demographic Data Server
###### tags: `CyLab`
###### tags: `Gazelle`

## Introduction
### What is Demographic Data Server?
The Demographic Data Server (aka DDS) is a tool which generates random demographic informations, and make them available to tools for testing purposes.

The Demographic Data Server tries to respond to the following use cases : 

* Generate realistic data set to fill out a database with data for testing purpose. 
* Request data on the demand through web services
* Transfert data through HL7 V2 or HL7 V3 messages (using multibyte character encoding when necessary)
* Support for different kind of character encoding. 
* Support for many languages and countries
* Usage through web interface (for humans) or web services (for machines)

### Dependency
Gazelle Proxy runs under Jboss AS 7.2.0 with a postgresql PostGresql 9.1 or higher database. 

## Installation
* Download
```
# find and download the last version ear and sql https://gazelle.ihe.net/nexus/index.html#nexus-search;quick~DemographicDataServer-ear
wget https://gazelle.ihe.net/nexus/service/local/repositories/releases/content/net/ihe/gazelle/maven/DemographicDataServer-ear/4.3.3/DemographicDataServer-ear-4.3.3.ear
wget https://gazelle.ihe.net/nexus/service/local/repositories/releases/content/net/ihe/gazelle/maven/DemographicDataServer-ear/4.3.3/DemographicDataServer-ear-4.3.3-sql.zip
unzip -d sql DemographicDataServer-ear-4.3.3-sql.zip
cd sql/
```

* Database creation and initialization
```
# create database user and database
sudo -u postgres psql
CREATE DATABASE "demographic-data-server" OWNER gazelle ENCODING "UTF-8";
\q

psql -U gazelle demographic-data-server < schema-X.X.X.sql
psql -U gazelle demographic-data-server < init-X.X.X.sql
```

* Deployment
```
# Jboss configuration
${seamName} = DemographicDataServerDS
${databaseName} = demographic-data-server

# stop jboss
sudo systemctl stop jboss7.service

# copy ear to jboss deployment folder
sudo cp DemographicDataServer-ear-${version}.ear /usr/local/jboss7/standalone/deployments/DemographicDataServer.ear

# start jboss
sudo systemctl start jboss7.service
```

* Configure Apache2
```
<Location /DDS>
   ProxyPass        ajp://localhost:8109/DDS
   ProxyPassReverse ajp://localhost:8109/DDS
</Location>
 
# Check that apache2 configuration is still ok
sudo apache2ctl configtest

#If everything is ok you can then restart apache2
sudo apache2ctl restart
```