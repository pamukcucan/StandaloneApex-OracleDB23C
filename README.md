Start with Linux 8 intance root user;

Sudo su


For Oracle Linux 8 Minimal or Server Install:
dnf install -y oraclelinux-developer-release-el8

For Oracle Linux 8 Oracle Cloud-Based Instances:
dnf config-manager --set-enabled ol8_developer 

dnf -y install oracle-database-preinstall-23c


Creating and Configuring an Oracle Database
The configuration script creates a container database (FREE) with one pluggable database (FREEPDB1) and configures the listener at the default port (1521). 
You can modify the configuration parameters by editing the /etc/sysconfig/oracle-free–23c.conf file. 

Sudo su

/etc/init.d/oracle-free-23c configure


Setting Oracle Database Free Environment Variables >> (Kalıcı hale getirmek için aşağıdaki ** bölüm ile devam edilebilir)
After you install and configure Oracle Database Free, set the environment before you use Oracle Database Free. 
Use the oraenv and coraenv scripts to set your environment variables. 
For example, to set your environment variables in Bourne, Bash, or Korn shell without being prompted by the script:


export ORACLE_SID=FREE 
export ORAENV_ASK=NO 
. /opt/oracle/product/23c/dbhomeFree/bin/oraenv

ORACLE_HOME = [] ? /opt/oracle/product/23c/dbhomeFree
The Oracle base has been set to /opt/oracle

$ cd $ORACLE_HOME/bin
$ ./sqlplus / as sysdba


SQL*Plus: Release 23.0.0.0.0 - Developer-Release on Wed Mar 15 22:37:14 2023
Version 23.2.0.0.0
Copyright (c) 1982, 2023, Oracle.  All rights reserved.
Connected to:
Oracle Database 23c Free, Release 23.0.0.0.0 - Developer-Release
Version 23.2.0.0.0


These commands connect you to the root container CDB$ROOT of the multitenant database (CDB) as database user SYS. This method of connecting to the database works even if the Net Services listener is not running. 


$ cd $ORACLE_HOME/bin
$ lsnrctl status

LSNRCTL for Linux: Version 23.0.0.0.0
Copyright (c) 1991, 2023, Oracle. All rights reserved.
Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=dbhost.example.com)(PORT=1521)))
STATUS of the LISTENER
------------------------
Alias                     LISTENER
Version                   TNSLSNR for Linux: Version 23.0.0.0.0 - Developer-Release
Start Date                21-MAR-2023 22:30:55
Uptime                    0 days 0 hr. 8 min. 3 sec
Trace Level               off
Security                  ON: Local OS Authentication
SNMP                      OFF
Default Service           FREE
Listener Parameter File /opt/oracle/product/23c/dbhomeFree/network/admin/listener.ora
Listener Log File /opt/oracle/diag/tnslsnr/dbhost/listener/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=dbhost.example.com)(PORT=1521)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1521)))
Services Summary...
Service "FREE.dbhost.example.com" has 1 instance(s).
  Instance "FREE", status READY, has 1 handler(s) for this service...
Service "FREEXDB.dbhost.example.com" has 1 instance(s).
  Instance "FREE", status READY, has 1 handler(s) for this service...
Service "dbhost.example.com" has 1 instance(s).
  Instance "FREE", status READY, has 1 handler(s) for this service...
Service "freepdb1.dbhost.example.com" has 1 instance(s).
  Instance "FREE", status READY, has 1 handler(s) for this service...
The command completed successfully


**Varıable'ları her sefer tek tek set etmek yerine root user ile env dediğimizde çıkan farkları cd home/oracle daki .bash_profile içerisine eklersek bunu kalıcı hale getirmiş oluruz!

Bu durumda farklar aşağıdakiler, bunları .bash_profile'a ekliyoruz:

export ORACLE_SID=FREE
export ORACLE_BASE=/opt/oracle
export ORACLE_HOME=/opt/oracle/product/23c/dbhomeFree
export OLDPWD=/opt/oracle
export PWD=/opt/oracle/product/23c/dbhomeFree/bin
export ORAENV_ASK=NO





Starting and Stopping Oracle Database
Shut Down and Start-Up Using SQL*Plus
You can shut down and start the database using SQL*Plus.
To shut down the database, log in as the oracle user with its environment variables set for access to Oracle Database Free, and run the following SQL*Plus command: 

$ sqlplus / as sysdba
SQL> SHUTDOWN IMMEDIATE

To start the database:

SQL> STARTUP
SQL> ALTER PLUGGABLE DATABASE ALL OPEN;


Listener ve DB Tek tek Stop Start etmek yerine hepsini Otomatize edelim;

Sudo su
# systemctl daemon-reload
# systemctl enable oracle-free-23c

Bu komutlar ile hem Listener hem de DB için stop ve start verebiliriz;

# systemctl start oracle-free-23c
# systemctl stop oracle-free-23c
# systemctl restart oracle-free-23c

Bu komut ile de Statusu görebiliriz;

# /etc/init.d/oracle-free-23c status

Status of the Oracle FREE 23c service:
LISTENER status: RUNNING
FREE Database status: RUNNING



Now Lets Install Apex

cd/home/oracle/ 
wget https://download.oracle.com/otn_software/apex/apex-latest.zip

unzip apex-latest.zip

cd apex

sqlplus sys@localhost:1521 as sysdba

In case this doesn't work, check your DB Environment Variables again

Execute the apexins.sql script to install Oracle APEX using the following command:

@apexins.sql SYSAUX SYSAUX TEMP /i/


Run the apxchpwd.sql script to change the APEX administrator password:

@apxchpwd.sql


Unlock the APEX_PUBLIC_USER account and set a new password using the following commands:

ALTER USER APEX_PUBLIC_USER ACCOUNT UNLOCK;
ALTER USER APEX_PUBLIC_USER IDENTIFIED BY <password>;

Finally, configure APEX RESTful services by executing the apex_rest_config.sql script:

@apex_rest_config.sql


Installation of ORDS
Verify your Java version.

Java -version


Next, execute the command below, which will initiate the installation process for ORDS.

yum install ords -y


Once the installation is finished, you can begin configuring and using ORDS.

ords --config /etc/ords/config install

(Bu çalışmayabiliyor bu sebeple ords komutunu alabilmesi için olduğu dizini bulmak gerek orada çalıştırınca sorun yok..)

You'll be asked a few questions, so just go ahead and answer them. No worries, you can likely press Enter/Return for most of them to accept the default options.

Type of installation > [2] Enter 
Database connection type> [1] Enter
Host name > [localhost] Enter
Listen port > [1521] Enter
Service name > FREEPDB1
Administrator username > sys
Password > 
Default tablespace > [SYSAUX] Enter
Temp tablespace > [TEMP] Enter
Additional features > [1] Enter
Start ORDS instandalone mode > [1] Enter
Protocol > [HTTP] Enter
Port > [8080] 8181
Static Resources > /home/oracle/apex/images


Once everything is complete, start ORDS using the command below.

/etc/init.d/ords start


Setting Up the Firewall
firewall-cmd --permanent --zone=public --add-port=1521/tcp
firewall-cmd --permanent --zone=public --add-port=8080/tcp
firewall-cmd --permanent --zone=public --add-port=8181/tcp
firewall-cmd --reload

And don't forget to setup your security list access rules accordingly;



The final Test

http://158.101.231.166:8181/ords



Congrats!!

http://158.101.231.166:8080/ords/sql-developer




References:
https://tm-apex.hashnode.dev/quick-and-easy-apex-and-ords-installation-for-oracle-database-23c-developer-release-on-oci

https://docs.oracle.com/en/database/oracle/oracle-database/23/xeinl/starting-and-stopping-oracle-database.html




