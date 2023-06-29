# EVSClient
###### tags: `CyLab`
###### tags: `Gazelle`

## Introduction
### What is Validation Services?
This application has been developed with the purpose of aggregating in a same user interface, the access to all validation services developed for IHE and its national extensions. Called services are the followings (non-exhaustive):

* Gazelle HL7 Validator for HL7v2.x and HL7v3 messages
* Schematron-based Validator (CDA, Audit messages, HL7v3, assertions…)
* Model-based validator for CDA
* Model-based validator for XD* messages
* Model-based validator for XDW
* Model-based validator for DSUB
* Model-based validator for SVS
* Model-based validator for HPD
* Certificates validation
* JHOVE for PDF files validation
* Dicom3Tools, DCMCHECK, Dcm4Che, Pixelmed for validating DICOM objects

### Dependency
Gazelle Proxy runs under Jboss AS 7.2.0 with a postgresql PostGresql 9.1 or higher database. 

## Installation
* Download
```
# find and download the last version ear and sql https://gazelle.ihe.net/nexus/index.html#nexus-search;gav~~EVSClient-ear~5.*~~
wget https://gazelle.ihe.net/nexus/service/local/repositories/releases/content/net/ihe/gazelle/EVSClient-ear/5.15.2/EVSClient-ear-5.15.2.ear
wget https://gazelle.ihe.net/nexus/service/local/repositories/releases/content/net/ihe/gazelle/EVSClient-ear/5.15.2/EVSClient-ear-5.15.2-sql.zip
unzip -d sql EVSClient-ear-5.15.2-sql.zip
cd sql/
```

* Database creation and initialization
```
# create database user and database
sudo -u postgres psql
CREATE DATABASE "evs-client-prod" OWNER gazelle ENCODING "UTF-8";
\q

psql -U gazelle evs-client-prod < schema-X.X.X.sql
psql -U gazelle evs-client-prod < init-X.X.X.sql
```

* Proxy file configuration
```
# add the following line to /opt/gazelle/cas/EVSClient.properties
# replace all occurence of the ${FQDN} with the FQDN
casServerUrlPrefix = https://${FQDN}/sso
casServerLoginUrl = https://${FQDN}/sso/login
casLogoutUrl = https://${FQDN}/sso/logout
service = https://${FQDN}/EVSClient
```

* Deployment
```
# Jboss configuration
${seamName} = EVSClientDS
${databaseName} = evs-client-prod

# stop jboss
sudo systemctl stop jboss7.service

# copy ear to jboss deployment folder
sudo cp EVSClient-ear-${version}.ear /usr/local/jboss7/standalone/deployments/EVSClient.ear

# start jboss
sudo systemctl start jboss7.service
```

* Configure Apache2
```
<Location /EVSClient>
   ProxyPass        ajp://localhost:8109/EVSClient
   ProxyPassReverse ajp://localhost:8109/EVSClient
</Location>
 
# Check that apache2 configuration is still ok
sudo apache2ctl configtest

#If everything is ok you can then restart apache2
sudo apache2ctl restart
```