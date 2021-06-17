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

1. Create development oracle database instance

  ```docker run -d -it --name dbdevelopment -p 1521:1521 -v OracleDatabaseDevelopment:/ORCL store/oracle/database-enterprise:12.2.0.1```

1. `docker logs dbdevelopment` - execute this command multiple time untill you'll see below output
   ```Done ! The database is ready for use .
      # ===========================================================================  
      # == Add below entries to your tnsnames.ora to access this database server ==  
      # ====================== from external host =================================  
      ORCLCDB=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=<ip-address>)(PORT=<port>))
          (CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ORCLCDB.localdomain)))     
      ORCLPDB1=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=<ip-address>)(PORT=<port>))
          (CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ORCLPDB1.localdomain)))```   

1. Login in SQL
   ```docker exec -it dbdevelopment bash -c "source /home/oracle/.bashrc; sqlplus sys/  Oradoc_db1@ORCLCDB as sysdba"```

1. Use pluggable DB
  ``` SQL> alter session set container = ORCLPDB1;```

1. kick start HR schema scripts
    ```SQL> @?/demo/schema/human_resources/hr_main.sql;
    specify password for HR as parameter 1:
    Enter value for 1: hr123

    specify default tablespeace for HR as parameter 2:
    Enter value for 2: users

    specify temporary tablespace for HR as parameter 3:
    Enter value for 3: temp

    specify log path as parameter 4:
    Enter value for 4: log```

1. Unlock HR acount and set password
  ```ALTER USER HR ACCOUNT UNLOCK IDENTIFIED BY hr123;
     GRANT ALL PRIVILEGES TO HR;```

#######################################################################

##################Redgate shadow HR schema##############################

1. Create shadow oracle database instance this is specially for Redgate

   ```docker run -d -it --name dbshadow -p 1522:1521 store/oracle/database-enterprise:12.2.0.1```

1. `docker logs dbshadow` - execute this command multiple time untill you'll see below output
    ```Done ! The database is ready for use .
      # ===========================================================================  
      # == Add below entries to your tnsnames.ora to access this database server ==  
      # ====================== from external host =================================  
      ORCLCDB=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=<ip-address>)(PORT=<port>))
          (CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ORCLCDB.localdomain)))     
      ORCLPDB1=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=<ip-address>)(PORT=<port>))
          (CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ORCLPDB1.localdomain)))``` 
   
1. Login in SQL 
    ```docker exec -it dbshadow bash -c "source /home/oracle/.bashrc; sqlplus sys/Oradoc_db1@ORCLCDB as sysdba"```

1. Use pluggable DB
    ```alter session set container = ORCLPDB1;```

1. Create an HR user schema. This schema name should be the same as in development and production database
  ```
    CREATE USER HR
    IDENTIFIED BY hr123
    DEFAULT TABLESPACE users
    TEMPORARY TABLESPACE temp;
  ```
1. Grant all privleges to HR user
    ```GRANT ALL PRIVILEGES TO HR;```

###########################################################################

###################REDGATE Prod Database#################

1. Create production oracle database instance
  ```docker run -d -it --name dbprod -p 1523:1521 -v OracleDatabaseProd:/ORCL store/oracle/database-enterprise:12.2.0.1```

1. docker logs dbprod - execute this command multiple time untill you'll see below output
  ```Done ! The database is ready for use .
    # ===========================================================================  
    # == Add below entries to your tnsnames.ora to access this database server ==  
    # ====================== from external host =================================  
    ORCLCDB=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=<ip-address>)(PORT=<port>))
        (CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ORCLCDB.localdomain)))     
    ORCLPDB1=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=<ip-address>)(PORT=<port>))
        (CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ORCLPDB1.localdomain)))```

1. Login in SQL 
    ```docker exec -it dbprod bash -c "source /home/oracle/.bashrc; sqlplus sys/Oradoc_db1@ORCLCDB as sysdba"```

1. Use pluggable DB
    ```alter session set container = ORCLPDB1;```

1. kick start HR schema scripts

    ```@?/demo/schema/human_resources/hr_main.sql;
    specify password for HR as parameter 1:
    Enter value for 1: hr123

    specify default tablespeace for HR as parameter 2:
    Enter value for 2: users

    specify temporary tablespace for HR as parameter 3:
    Enter value for 3: temp

    specify log path as parameter 4:
    Enter value for 4: log```

1.  Unlock HR acount and set password 
    ```SQL> ALTER USER HR ACCOUNT UNLOCK IDENTIFIED BY hr123;
       SQL> ALTER USER HR IDENTIFIED BY hr123;
       SQL> GRANT ALL PRIVILEGES TO HR;```


##################SQL developer connection#################################

1. Create three connections in SQL developer for these port 1521 (development instance), 1522 (shadow instance), 1523 (production instance). Use for all hr user.

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


###########################FLYWAY ORACLE#############################
https://hub.docker.com/r/flyway/flyway - link documentation

docker run --rm -v /home/ec2-user/apanovasqlmigration:/flyway/sql -v /home/ec2-user/redgate/confdir:/flyway/conf -v /home/ec2-user/redgate/driverdir:/flyway/drivers flyway/flyway -url=jdbc:oracle:thin:@//172.31.28.182:1523/ORCLPDB1.localdomain -user=HR -password=hr123 -table="redgate_schema_history" info

docker run --rm -v /home/ec2-user/apanovasqlmigration:/flyway/sql -v /home/ec2-user/redgate/confdir:/flyway/conf -v /home/ec2-user/redgate/driverdir:/flyway/drivers flyway/flyway -url=jdbc:oracle:thin:@//172.31.28.182:1523/ORCLPDB1.localdomain -user=HR -password=hr123 -table="redgate_schema_history" -baselineOnMigrate="true" migrate

#######################################################################

###########################Redgate Change Control###########################

https://www.youtube.com/watch?v=ikfbGviIm0w
https://www.red-gate.com/products/oracle-development/source-control-for-oracle/

############################################################################




./buildContainerImage.sh -v 19.3.0 -s

docker run -i -t -d --hostname ora19s --name ora19s -p 1521:1521 -v ~/Docker/shared/:/shared oracle/database:19.3.0-se2
