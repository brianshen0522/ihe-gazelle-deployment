# Gazelle SSO
###### tags: `CyLab`
###### tags: `Gazelle`

## Introduction
### What is SSO?
The purpose of this document is to guide you through the installation process of the Apereo CAS 5.1.X. This SSO is used by Gazelle tools to authenticate users.

### Dependency
Installation needs to be performed on a Debian linux Stretch or newer. This is know to not work on jessie. Apereo needs to be embedded into a Tomcat 8 server. Tomcat 8 runs under Java 8. Make sure you have zulu-8 installed.

## Installation
* Download and deploy Gazelle SSO
```
cd /var/lib/tomcat8/webapps
sudo wget https://gazelle.ihe.net/apereo-cas-gazelle/cas.war
sudo mv cas.war sso.war
```
* Access to user database
```
# download configuration sample
sudo su
cd /etc
wget https://gazelle.ihe.net/apereo-cas-gazelle/cas.tgz
tar zxvf cas.tgz
mkdir /etc/cas/log
chown -R tomcat8:tomcat8 cas
```
* edit /etc/cas/config/cas.properties
```
find
"cas.authn.attributeRepository.jdbc[0].url=jdbc:postgresql://localhost:5432/cas"
"cas.authn.jdbc.query[0].url=jdbc:postgresql://localhost:5432/cas"
replace "cas" with "gazelle"

replace all occurence of the string "stretch.localdomain" with the FQDN
```
* Configure logs
```
#  in /etc/cas/config/log4j2.xml, add the following code in the first <RollingFile>, below </Policies>

<DefaultRolloverStrategy max="5">
    <Delete basePath="${sys:cas.log.dir}">
      <IfFileName glob="*.log" />
      <IfLastModified age="7d" />
    </Delete>
</DefaultRolloverStrategy>
```
* Configure the applications to use this new SSO
```
mkdir -p /opt/gazelle/cas
touch /opt/gazelle/cas/file.properties
chown -R jboss:jboss-admin /opt/gazelle/cas
chmod -R g+w /opt/gazelle/cas

# add the following line to /opt/gazelle/cas/file.properties
# replace all occurence of the ${FQDN} with the FQDN
serverName=https://${FQDN}
casServerUrlPrefix=https://${FQDN}/sso
casServerLoginUrl=https://${FQDN}/sso/login
casLogoutUrl=https://${FQDN}/sso/logout
```

* Fix log4j vulnerability
```
# Add the following line in setenv.sh. Create the file if it does not exist in {CATALINA_HOME}/bin or {CATALINA_HOME}/bin : it is usually in /usr/share/tomcat8/bin if tomcat was installed via apt.
CATALINA_OPTS="$CATALINA_OPTS -Dlog4j2.formatMsgNoLookups=true"

# restart tomcat
sudo systemctl restart tomcat8.service
```

* Configure Apache2
```
<Location /sso>
   ProxyPass        ajp://localhost:8209/sso
   ProxyPassReverse ajp://localhost:8209/sso
 </Location>
 
# Check that apache2 configuration is still ok
sudo apache2ctl configtest

# apache2 enable modules
sudo a2enmod ssl headers rewrite proxy_ajp

#If everything is ok you can then restart apache2
sudo apache2ctl restart
```