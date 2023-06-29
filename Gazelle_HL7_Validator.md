# Gazelle HL7 Validator
###### tags: `CyLab`
###### tags: `Gazelle`

## Introduction
### What is Gazelle HL7 Validator?
GazelleHL7Validator is the part of the Gazelle platform dedicated to the validation of HL7 messages.

### Dependency
Gazelle Proxy runs under Jboss AS 7.2.0 with a postgresql PostGresql 9.1 or higher database. 

## Installation
* Download
```
# find and download the last version ear and sql https://gazelle.ihe.net/nexus/#nexus-search;quick~GazelleHL7v2Validator-ear

unzip -d sql GazelleHL7v2Validator-ear-${version}-sql.zip
cd sql/
```

* Database creation and initialization
```
# create database user and database
sudo -u postgres psql
CREATE DATABASE "gazelle-hl7-validator" OWNER gazelle ENCODING "UTF-8";
\q

psql -U gazelle gazelle-hl7-validator < schema-X.X.X.sql
psql -U gazelle gazelle-hl7-validator < init-X.X.X.sql
```

* Deployment
```
# Jboss configuration
${seamName} = GazelleHL7v2ValidatorDS
${databaseName} = gazelle-hl7-validator

# stop jboss
sudo systemctl stop jboss7.service

# copy ear to jboss deployment folder
sudo cp GazelleHL7v2Validator-ear-${version}.ear /usr/local/jboss7/standalone/deployments/GazelleHL7v2Validator.ear


# start jboss
sudo systemctl start jboss7.service
```

* Configure Apache2
```
<Location /GazelleHL7Validator>
   ProxyPass        ajp://localhost:8109/GazelleHL7Validator
   ProxyPassReverse ajp://localhost:8109/GazelleHL7Validator
</Location>
 
# Check that apache2 configuration is still ok
sudo apache2ctl configtest

#If everything is ok you can then restart apache2
sudo apache2ctl restart
```