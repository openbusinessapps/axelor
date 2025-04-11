# Installation of Axelor ERP with Tomcat, PostgreSQL, and Java on Ubuntu 22.04

This project provides a guide to install and configure Axelor ERP on an Ubuntu 22.04 server using PostgreSQL 14.5 as the database, Tomcat v9.0.102 as the application server, and Java 1.11.0 to run the services.

The tutorial covers the following steps:
- Installation of PostgreSQL 14.5 and creation of the database for Axelor.
- Installation and configuration of Tomcat v9.0.102 to run the Axelor ERP application.
- Downloading and deploying the latest version of Axelor ERP.
- Configuration of the database and necessary permissions for the Axelor user.
- Starting and managing Tomcat as a system service.

### Objective

The goal of this tutorial is to enable users to install Axelor ERP in a production environment on an Ubuntu 22.04 server, with an optimized configuration for PostgreSQL 14.5, Tomcat v9.0.102, and Java 1.11.0.
**Technologies used:**
- **Ubuntu 22.04** : operating system.
- **PostgreSQL 14.5** : relational database used by Axelor ERP.
- **Tomcat v9.0.102** : application server for deploying Axelor ERP.
- **Java 1.11.0** : runtime environment required for Tomcat and Axelor ERP.

This guide is designed for system administrators who wish to deploy Axelor ERP on an Ubuntu server with the specified versions of the software components..


----------------------------------------------------------------------------------------------------------------


# Installation d'Axelor ERP avec Tomcat, PostgreSQL et Java sur Ubuntu 22.04

Ce projet fournit un guide pour installer et configurer **Axelor ERP** sur un serveur Ubuntu 22.04 en utilisant **PostgreSQL 14.5** comme base de données, **Tomcat v9.0.102** comme serveur d'applications, et **Java 1.11.0** pour exécuter les services.

Le tutoriel couvre les étapes suivantes :
- Installation de PostgreSQL 14.5 et création de la base de données pour Axelor.
- Installation et configuration de **Tomcat v9.0.102** pour exécuter l'application Axelor ERP.
- Téléchargement et déploiement de la dernière version d'Axelor ERP.
- Configuration de la base de données et des permissions nécessaires pour l'utilisateur Axelor.
- Démarrage et gestion de Tomcat comme service système.

### Objectif

L'objectif de ce tutoriel est de permettre aux utilisateurs d'installer Axelor ERP dans un environnement de production sur un serveur Ubuntu 22.04, avec une configuration optimisée pour **PostgreSQL 14.5**, **Tomcat v9.0.102** et **Java 1.11.0**.

**Technologies utilisées :**
- **Ubuntu 22.04** : système d'exploitation.
- **PostgreSQL 14.5** : base de données relationnelle utilisée par Axelor ERP.
- **Tomcat v9.0.102** : serveur d'applications pour déployer Axelor ERP.
- **Java 1.11.0** : environnement d'exécution nécessaire pour Tomcat et Axelor ERP.

Ce guide est conçu pour les administrateurs système souhaitant déployer Axelor ERP sur un serveur Ubuntu avec les versions spécifiées des composants logiciels.


----------------------------------------------------------------------------------------------------------------

## GAIN SUPER ADMIN
sudo su

# INSTALL UNZIP
sudo apt install unzip


## INSTALL POSTGRESQL
sudo apt install postgresql

# CONNECT AS POSTGRES
sudo su postgres
 
# CREATE DATABASE USER CONNECTED AS POSTGRES
createuser --no-superuser --username postgres --pwprompt axelor

#CREATE DB
createdb --owner=axelor axelor

#START POSTGRE INTERACTIVE COMMAND
psql

#GRANT PRIVILEGES
GRANT ALL PRIVILEGES ON DATABASE axelor TO axelor;  
GRANT ALL PRIVILEGES ON SCHEMA public TO axelor;  
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO axelor; 
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL PRIVILEGES ON TABLES TO axelor;   
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO axelor; 
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL PRIVILEGES ON SEQUENCES TO axelor; 
GRANT ALL PRIVILEGES ON ALL FUNCTIONS IN SCHEMA public TO axelor; 
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL PRIVILEGES ON FUNCTIONS TO axelor;

#QUIT INTERACTIVE
\q

#BACK TO USER
exit

#INSTALL JDK
sudo apt install default-jdk

# CHECK JDK
java -version

# CREATE DIRECTORY
mkdir /opt/src

# HEAD TO DIRECTORY
cd /opt/src

# DL
wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.102/bin/apache-tomcat-9.0.102.tar.gz --no-check-certificate


#EXTRACT
tar -xzf apache-tomcat-9.0.102.tar.gz

#SYMBOLIC LINK FROM SRC to OPT
ln -s /opt/src/apache-tomcat-9.0.102 /opt/default-tomcat

#TOMCAT LOGS CONF
sed -z -r -i 's/(<Valve className="org.apache.catalina.valves.AccessLogValve".*\/>)/<!--\1-->/' /opt/default-tomcat/conf/server.xml


# CHECK JAVA INSTALLATION PATH
sudo update-java-alternatives -l
# java-1.11.0-openjdk-amd64      1111       /usr/lib/jvm/java-1.11.0-openjdk-amd64



# CREATE TOMCAT SERVICE FILE
sudo nano /etc/systemd/system/tomcat.service

 # APPEND LINES TO TOMCAT SERVICE FILE
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target
[Service]
Type=forking
User=tomcat
Group=tomcat
Environment="JAVA_HOME=/usr/lib/jvm/java-1.11.0-openjdk-amd64"
Environment="CATALINA_HOME=/opt/default-tomcat"
Environment="CATALINA_PID=/opt/default-tomcat/temp/tomcat.pid"
Environment="CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC"
ExecStart=/opt/default-tomcat/bin/startup.sh
ExecStop=/opt/default-tomcat/bin/shutdown.sh
[Install]
WantedBy=multi-user.target


# SETUP TOMCAT GROUP and USER
sudo groupadd tomcat
sudo useradd -s /bin/false -g tomcat -d /opt/default-tomcat tomcat

# GIVE TOMCAT USER PERMISSION ON TOMCAT FOLDER
sudo chown -RH tomcat: /opt/default-tomcat

# RELOAD SERVICES
sudo systemctl daemon-reload
sudo systemctl enable tomcat

# START TOMCAT

sudo systemctl start tomcat
sudo systemctl status tomcat

# AXELOR WAR
cd /opt/src/
wget https://github.com/axelor/open-suite-webapp/releases/download/v8.3.3/axelor-erp-v8.3.3.war
unzip axelor-erp-v8.3.3.war -d axelor
ln -s /opt/src/axelor /opt/default-tomcat/webapps/
nano /opt/src/axelor/WEB-INF/classes/axelor-config.properties
#change db name and password
#set demo to “true”


# GIVE PERMISSION ON AXELOR FOLDER TO TOMCAT USER

chown -R tomcat:tomcat /opt/src/axelor
chown -R tomcat:tomcat /opt/default-tomcat/webapps/axelor


# STOP TOMCAT
systemctl stop tomcat

# START TOMCAT 
systemctl start tomcat

# LOGS
tail -f /opt/default-tomcat/logs/catalina.out


#IF ADMIN ACCOUNT IS DISABLED
sudo -u postgres psql

#connect to db
\c axelor

#list all tables
\dt

#list columns in table
\d auth_user

#show value of field
SELECT code,blocked FROM auth_user;

#UPDATE
UPDATE auth_user SET blocked = 'f' WHERE code = 'demo';

