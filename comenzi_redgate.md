########################INSTALL/CONFIGURE DOCKER################################
sudo yum install -y git
git init
git add README.md comenzi_redgate.md
systemctl enable docker.service
systemctl start docker.service
systemctl status docker.service

########################Create/Activate HR schema in docker hub Oracle database#########################
docker login
docker run -d -it --name dbmigoracle -p 1521:1521 -v OracleDatabase:/ORCL store/oracle/database-enterprise:12.2.0.1
docker logs dbmigoracle

docker exec -it dbmigoracle bash -c "source /home/oracle/.bashrc; sqlplus sys/Oradoc_db1@ORCLCDB as sysdba"

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

##################Redgate shadow control##############################


docker exec -it dbmigoracle bash -c "source /home/oracle/.bashrc; sqlplus sys/Oradoc_db1@ORCLCDB as sysdba"

# Use pluggable DB
alter session set container = ORCLPDB1;

CREATE USER HRSHADOW
  IDENTIFIED BY pwd4hrshadow
  DEFAULT TABLESPACE users
  TEMPORARY TABLESPACE temp;



GRANT ALL PRIVILEGES TO HRSHADOW;

###########################################################################

##################SQL developer connection#################################

# sys user
Username: sys
Password: Oradoc_db1 

# hr user
Username: hr
Password: hr123

Hostname: localhost
Port: 1521
Service name: ORCLPDB1.localdomain

######################################################################

###################REDGATE Change Control Prod Database#################

docker run -d -it --name dbprod -p 1522:1521 -v OracleDatabaseProd:/ORCL store/oracle/database-enterprise:12.2.0.1
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



Windows AWS Admin Password: &TKI$%rzYuMsvRG.7-fhIjsFsd2PjlI;






###########################FLYWAY ORACLE#############################
https://hub.docker.com/r/flyway/flyway - link documentation
docker run --rm -v /home/ec2-user/oracle_db_migration/flyway/sqldir:/flyway/sql -v /home/ec2-user/oracle_db_migration/flyway/confdir:/flyway/conf -v /home/ec2-user/oracle_db_migration/flyway/driverdir:/flyway/drivers flyway/flyway info

#######################################################################

###########################Redgate Change Control###########################

https://www.youtube.com/watch?v=ikfbGviIm0w
https://www.red-gate.com/products/oracle-development/source-control-for-oracle/

############################################################################




./buildContainerImage.sh -v 19.3.0 -s

docker run -i -t -d --hostname ora19s --name ora19s -p 1521:1521 -v ~/Docker/shared/:/shared oracle/database:19.3.0-se2