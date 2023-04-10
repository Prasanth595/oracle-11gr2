# Prasanth595/oracle Dockerfile

Oracle Database 11gR2 with SSH key access on Oracle Linux 6.6 Dockerfile for trusted Docker builds.

This [**Dockerfile**](https://github.com/Prasanth595/oracle/blob/master/Dockerfile) is a [trusted build](https://registry.hub.docker.com/u/Prasanth595/oracle/) of [Docker Registry](https://registry.hub.docker.com/).

### Base Docker Image

* [oraclelinux:6.6](https://github.com/_/oraclelinux)

### Installation
```
sudo docker pull Prasanth595/oracle
```

### Run with external Database storage
```
sudo mkdir /oradata
sudo chmod -R 770 /oradata
sudo chgrp -R 501 /oradata                   # where 501 is number of dba group in the container
sudo chcon -Rt svirt_sandbox_file_t /oradata # instruction for the SELinux
sudo docker run --name orac -it -P \
    -v /oradata:/u02/oradata \
    Prasanth595/oracle
```

### Login into Container by SSH
```
eval `ssh-agent -s`
ssh-add ssh/id_rsa
ssh root@localhost -p `sudo docker port orac 22 | cut -d":" -f2`
```

### Create Database
```
dbca-create <SID>
```
### Print Oracle port
```
export ORACLE_PORT=`sudo docker port orac 1521 | cut -d":" -f2`
echo $ORACLE_PORT
```

### Configure tnsnames.ora
```
$ORACLE_SID =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = $HOSTNAME)(PORT = $ORACLE_PORT))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = $ORACLE_SID)
    )
  )
```

### Login into Database
```
sqlplus "sys/oracle123@<SID> as sysdba"
```

### Delete Database
```
dbca-delete <SID>
```
### Instructions for building image manually
Download end extract this repo
```
wget https://github.com/Prasanth595/oracle/archive/master.zip
unzip master.zip
cd oracle-master
```

Download the Oracle Database from 
[**linux.x64_11gR2_database_1of2.zip**](http://download.oracle.com/otn/linux/oracle11g/R2/linux.x64_11gR2_database_1of2.zip)
[**linux.x64_11gR2_database_2of2.zip**](http://download.oracle.com/otn/linux/oracle11g/R2/linux.x64_11gR2_database_2of2.zip)
and replace them to software directory.

Change the encoding in the Dockerfile
```
echo 'export NLS_LANG=AMERICAN_AMERICA.CL8MSWIN1251' >> /etc/rc.local; \
```

Change the encoding in the response/db_install.rsp
```
oracle.install.db.config.starterdb.characterSet=CL8MSWIN1251
```

Build image
```
sudo docker build -t="Prasanth595/oracle" . 
```

### Build the image with only last layer to compress
```
echo "Build the Docker image"
sudo docker build -t="Prasanth595/oracle:source" .

echo "Create the Container with only last layer to compress"
sudo docker run --name orac Prasanth595/oracle:source
sudo docker stop orac

echo "Export the Container to file"
sudo docker export orac > oracle.tar

echo "Import the Container from file to the raw Docker image (without metadata)"
cat oracle.tar | sudo docker import - Prasanth595/oracle:raw
rm oracle.tar

echo "Remove the Container"
sudo docker rm orac

echo "List images (Prasanth595/oracle:source can be deleted)"
sudo docker images
```

### To restore metadata rebuild image with following Dockerfile
```
FROM Prasanth595/oracle:raw
VOLUME /u02/oradata /u02/dump
EXPOSE 22 1521
CMD /etc/rc.local; bash
```
As a result, the size of the Docker image will be 5 GB
