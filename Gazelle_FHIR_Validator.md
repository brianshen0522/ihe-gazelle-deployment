# Gazelle FHIR Validator 
###### tags: `CyLab`
###### tags: `Gazelle`

## Introduction
### What is Gazelle FHIR Validator?
This tool owns the validator for the messages defined in the IHE profiles based on FHIR. Two types of messages are validated:

* The requests (URLs) are parsed and validating using an in-house algorithm, based on a template defined by the test designer from the user interface.
* The FHIR resources are validated in three steps:
* Is the message a valid XML or JSON document
* Does the message complies with the FHIR requirements
* Does the message complies with IHE requirements (uses the structure definitions published by IHE or other domains)

### Dependency
Gazelle Proxy runs under Jboss AS 7.2.0 with a postgresql PostGresql 9.1 or higher database. 

## Installation
* Download
```
# find and download the last version ear and sql https://gazelle.ihe.net/nexus/index.html#nexus-search;gav~~FhirValidator*~~~
wget https://gazelle.ihe.net/nexus/service/local/repositories/releases/content/net/ihe/gazelle/validator/FhirValidator-ear/4.1.7/FhirValidator-ear-4.1.7.ear
wget https://gazelle.ihe.net/nexus/service/local/repositories/releases/content/net/ihe/gazelle/validator/FhirValidator-ear/4.1.7/FhirValidator-ear-4.1.7-sql.zip
unzip -d sql FhirValidator-ear-4.1.7-sql.zip
cd sql/
```

* Database creation and initialization
```
# create database user and database
sudo -u postgres psql
CREATE DATABASE "gazelle-fhir-validator" OWNER gazelle ENCODING "UTF-8";
\q

psql -U gazelle gazelle-fhir-validator < schema-X.X.X.sql
psql -U gazelle gazelle-fhir-validator < init-X.X.X.sql
```

* Deployment
```
# Wildfly configuration
${seamName} = FhirValidatorDS
${databaseName} = gazelle-fhir-validator

# stop wildfly
sudo systemctl stop wildfly10.service

# copy ear to wildfly deployment folder
sudo cp FhirValidator-ear-${version}.ear /usr/local/wildfly10/standalone/deployments/FhirValidator.ear

# start wildfly
sudo systemctl start wildfly10.service
```

* Configure Apache2
```
<Location /GazelleFhirValidator>
   ProxyPass        ajp://localhost:8309/GazelleFhirValidator
   ProxyPassReverse ajp://localhost:8309/GazelleFhirValidator
</Location>
 
# Check that apache2 configuration is still ok
sudo apache2ctl configtest

#If everything is ok you can then restart apache2
sudo apache2ctl restart
```