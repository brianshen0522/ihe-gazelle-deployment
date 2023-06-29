# Gazelle TestManagement
###### tags: `CyLab`
###### tags: `Gazelle`

## Introduction
### What is TestManagement?
The Gazelle Test Manager takes care of conformity and interoperability tests for IHE Profiles, according to their requirements. This tool also allows to manage test campaigns, generate test reports and add non-IHE tests to the set.

### Dependency
Gazelle Test Management runs under Jboss AS 7.2.0 with a postgresql PostGresql 9.1 or higher database. 

## Installation
* Download
```
# install graphviz
sudo apt install -y graphviz
sudo chown jboss -R /opt/
sudo apt install fonts-arphic-bkai00mp fonts-arphic-bsmi00lp fonts-arphic-gbsn00lp fonts-arphic-gkai00mp fonts-arphic-ukai fonts-arphic-uming fonts-cns11643-kai fonts-cns11643-sung fonts-cwtex-fs fonts-cwtex-heib fonts-cwtex-kai fonts-cwtex-ming fonts-cwtex-yen
sudo fc-cache -f -v

# find and download the last version ear and sql https://gazelle.ihe.net/nexus/index.html#nexus-search;gav~~gazelle-tm-ear*~~~
wget https://gazelle.ihe.net/nexus/service/local/repositories/releases/content/net/ihe/gazelle/tm/gazelle-tm-ear/6.10.0/gazelle-tm-ear-6.10.0.ear
wget https://gazelle.ihe.net/nexus/service/local/repositories/releases/content/net/ihe/gazelle/tm/gazelle-tm-ear/6.10.0/gazelle-tm-ear-6.10.0-sql.zip
unzip -d sql gazelle-tm-ear-6.10.0-sql.zip
cd sql/
```
* Database creation and initialization
```
# create database user and database
sudo -u postgres psql
CREATE USER gazelle;
CREATE DATABASE "gazelle" OWNER gazelle ENCODING "UTF-8";
ALTER USER gazelle WITH ENCRYPTED PASSWORD 'gazelle';
\q

psql -U gazelle gazelle < schema-X.X.X.sql
psql -U gazelle gazelle < init-X.X.X.sql
```

* SSO configuration
```
sudo mkdir -p /opt/gazelle/cas
sudo touch /opt/gazelle/cas/gazelle-tm.properties
sudo chown -R jboss:jboss-admin /opt/gazelle/cas
sudo chmod -R g+w /opt/gazelle/cas

# add the following line to /opt/gazelle/cas/gazelle-tm.properties
# replace all occurence of the ${FQDN} with the FQDN
casServerUrlPrefix = https://${FQDN}/sso
casServerLoginUrl = https://${FQDN}/sso/login
casLogoutUrl = https://${FQDN}/sso/logout
service = https://${FQDN}/gazelle
```

* etc
```
sudo mkdir -p /opt/gazelle/fileCache
sudo mkdir -p /opt/gazelle/cert
sudo chown -R jboss:jboss-admin /opt/gazelle
# move 643.jks and truststore.jks to /opt/gazelle/cert
```

* Deployment
```
# Jboss configuration
${seamName} = TestManagementDS
${databaseName} = gazelle

# stop jboss
sudo systemctl stop jboss7.service

# copy ear to jboss deployment folder
sudo cp gazelle-tm-ear-6.10.0.ear /usr/local/jboss7/standalone/deployments/gazelle-tm.ear

# start jboss
sudo systemctl start jboss7.service
```

* Configure Apache2
```
<Location /gazelle>
   ProxyPass        ajp://localhost:8109/gazelle
   ProxyPassReverse ajp://localhost:8109/gazelle
</Location>
 
# Check that apache2 configuration is still ok
sudo apache2ctl configtest

#If everything is ok you can then restart apache2
sudo apache2ctl restart
```