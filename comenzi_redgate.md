########################INSTALL/CONFIGURE DOCKER################################
sudo yum install -y git
sudo yum install -y docker docker-compose
git init
git add README.md comenzi_redgate.md
git commit -m "redgate comnenzi"
git remote add origin https://github.com/pandelegeorge/redgate.git
git config --global user.name "pandelegeorge"
git config --global user.email "pandelegeorge@gmail.com"
git config --global --list
git config credential.helper store
git push -u origin master
systemctl enable docker.service
systemctl start docker.service
systemctl status docker.service


docker login
      Username: pandelegeorge
      Password: <keepass>

########################Create/Activate HR schema in Dev Database #########################
docker run -d -it --name dbdevelopment -p 1521:1521 -v OracleDatabaseDevelopment:/ORCL store/oracle/database-enterprise:12.2.0.1

docker logs dbdevelopment

docker exec -it dbdevelopment bash -c "source /home/oracle/.bashrc; sqlplus sys/Oradoc_db1@ORCLCDB as sysdba"

# Use pluggable DB
alter session set container = ORCLPDB1;

# kick start HR schema scripts
@?/demo/schema/human_resources/hr_main.sql;
specify password for HR as parameter 1:
Enter value for 1: hr123

specify default tablespeace for HR as parameter 2:
Enter value for 2: users

specify temporary tablespace for HR as parameter 3:
Enter value for 3: temp

specify log path as parameter 4:
Enter value for 4: log

# ALTER USER HR ACCOUNT UNLOCK IDENTIFIED BY password;
ALTER USER HR ACCOUNT UNLOCK IDENTIFIED BY hr123;
ALTER USER HR IDENTIFIED BY hr123;

GRANT ALL PRIVILEGES TO HR;

#######################################################################

##################Redgate shadow HR schema##############################

docker run -d -it --name dbshadow -p 1522:1521 store/oracle/database-enterprise:12.2.0.1

docker logs dbshadow

docker exec -it dbshadow bash -c "source /home/oracle/.bashrc; sqlplus sys/Oradoc_db1@ORCLCDB as sysdba"

# Use pluggable DB
alter session set container = ORCLPDB1;

CREATE USER HR
  IDENTIFIED BY hr123
  DEFAULT TABLESPACE users
  TEMPORARY TABLESPACE temp;

GRANT ALL PRIVILEGES TO HR;

###########################################################################

##################SQL developer connection#################################

# sys user
Username: sys
Password: Oradoc_db1 

# hr user
Username: hr
Password: hr123

Hostname: localhost
Port: 1522
Service name: ORCLPDB1.localdomain

######################################################################

###################REDGATE Prod Database#################

docker run -d -it --name dbprod -p 1523:1521 -v OracleDatabaseProd:/ORCL store/oracle/database-enterprise:12.2.0.1

docker logs dbprod

docker exec -it dbprod bash -c "source /home/oracle/.bashrc; sqlplus sys/Oradoc_db1@ORCLCDB as sysdba"

# Use pluggable DB
alter session set container = ORCLPDB1;

# kick start HR schema scripts
@?/demo/schema/human_resources/hr_main.sql;
specify password for HR as parameter 1:
Enter value for 1: hr123

specify default tablespeace for HR as parameter 2:
Enter value for 2: users

specify temporary tablespace for HR as parameter 3:
Enter value for 3: temp

specify log path as parameter 4:
Enter value for 4: log

# ALTER USER HR ACCOUNT UNLOCK IDENTIFIED BY password;
ALTER USER HR ACCOUNT UNLOCK IDENTIFIED BY hr123;
ALTER USER HR IDENTIFIED BY hr123;

GRANT ALL PRIVILEGES TO HR;

###########################FLYWAY ORACLE#############################
https://hub.docker.com/r/flyway/flyway - link documentation

docker run --rm -v /home/ec2-user/hr_redgate/sqldir:/flyway/sql -v /home/ec2-user/hr_redgate/confdir:/flyway/conf -v /home/ec2-user/hr_redgate/driverdir:/flyway/drivers flyway/flyway -url=jdbc:oracle:thin:@//172.31.29.27:1523/ORCLPDB1.localdomain -user=HR -password=hr123 -table="redgate_schema_history" info

docker run --rm -v /home/ec2-user/hr_redgate/sqldir:/flyway/sql -v /home/ec2-user/hr_redgate/confdir:/flyway/conf -v /home/ec2-user/hr_redgate/driverdir:/flyway/drivers flyway/flyway -url=jdbc:oracle:thin:@//172.31.29.27:1523/ORCLPDB1.localdomain -user=HR -password=hr123 -table="redgate_schema_history" -baselineOnMigrate="true" migrate

#######################################################################

###########################Redgate Change Control###########################

https://www.youtube.com/watch?v=ikfbGviIm0w
https://www.red-gate.com/products/oracle-development/source-control-for-oracle/

############################################################################




./buildContainerImage.sh -v 19.3.0 -s

docker run -i -t -d --hostname ora19s --name ora19s -p 1521:1521 -v ~/Docker/shared/:/shared oracle/database:19.3.0-se2
