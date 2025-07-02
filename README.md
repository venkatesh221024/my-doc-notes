

.................===============================.............

# Tomcat installation on EC2 instance

### Follow this article in **[YouTube](https://www.youtube.com/watch?v=m21nFreFw8A)**
### Prerequisites
1. EC2 instance with Java v1.8.x 

### Install Apache Tomcat
Download tomcat packages from  https://tomcat.apache.org/download-80.cgi onto /opt on EC2 instance
```sh 
  # create tomcat directory
  cd /opt
  wget https://mirrors.estointernet.in/apache/tomcat/tomcat-9/v9.0.50/bin/apache-tomcat-9.0.50.tar.gz
  tar -xvzf /opt/apache-tomcat-9.0.50.tar.gz
```
give executing permissions to startup.sh and shutdown.sh which are under bin. 
```sh
   chmod +x /opt/apache-tomcat-8.5.35/bin/startup.sh shutdown.sh
```

create link files for tomcat startup.sh and shutdown.sh 
```sh
  ln -s /opt/apache-tomcat-9.0.29/bin/startup.sh /usr/local/bin/tomcatup
  ln -s /opt/apache-tomcat-9.0.29/bin/shutdown.sh /usr/local/bin/tomcatdown
  tomcatup
```
#### check point :
access tomcat application from browser on prot 8080  
http://<Public_IP>:8080

Using unique ports for each application is a best practice in an environment. But tomcat and Jenkins runs on ports number 8080. Hence lets change tomcat port number to 8090. Change port number in conf/server.xml file under tomcat home
```sh
cd /opt/apache-tomcat-8.5.35/conf
# update port number in the "connecter port" field in server.xml
# restart tomcat after configuration update
tomcatdown
tomcatup
```
#### check point :
access tomcat application from browser on prot 8090  
http://<Public_IP>:8090

now application is accessible on port 8090. but tomcat application doesnt allow to login from browser. changing a default parameter in context.xml does address this issue
```sh
#search for context.xml
find / -name context.xml
```
above command gives 3 context.xml files. comment (<!-- & -->) `Value ClassName` field on files which are under webapp directory. 
After that restart tomcat services to effect these changes
```sh 
tomcatdown
tomcatup
```
Update users information in the tomcat-users.xml file
goto tomcat home directory and Add below users to conf/tomcat-user.xml file
```sh
	<role rolename="manager-gui"/>
	<role rolename="manager-script"/>
	<role rolename="manager-jmx"/>
	<role rolename="manager-status"/>
	<user username="admin" password="admin" roles="manager-gui, manager-script, manager-jmx, manager-status"/>
	<user username="deployer" password="deployer" roles="manager-script"/>
	<user username="tomcat" password="s3cret" roles="manager-gui"/>
```
Restart serivce and try to login to tomcat application from the browser. This time it should be Successful


.........============================================.....................

..........
# Nexus Installation
Nexus is one a artifact repository which helps to store your build outcomes.  

### Follow this article in **[Youtube](https://www.youtube.com/watch?v=83AGz9huJGo)
### Prerequisites

1. EC2 instance with java 

### Implementation steps 

Download and setup nexus stable version
```sh 
cd /opt
wget https://sonatype-download.global.ssl.fastly.net/nexus/3/nexus-3.0.2-02-unix.tar.gz
tar -zxvf  nexus-3.0.2-02-unix.tar.gz
mv /opt/nexus-3.0.2-02 /opt/nexus
```

As a good security practice, it is not advised to run nexus service as root. so create new user called nexus and grant sudo access to manage nexus services 
```sh 
sudo adduser nexus
# visudo \\ nexus   ALL=(ALL)       NOPASSWD: ALL
sudo chown -R nexus:nexus /opt/nexus
```

Open /opt/nexus/bin/nexus.rc file, uncomment run_as_user parameter and set it as following.
```sh 
vi /opt/nexus/bin/nexus.rc
run_as_user="nexus" (file shold have only this line)
```

Add nexus as a service at boot time
```sh
sudo ln -s /opt/nexus/bin/nexus /etc/init.d/nexus
```
Login as a nexus user and start service
```sh
su - nexus
service nexus start
```

Login nexus server from browser on port 8081

http://<Nexus_server>:8081

Use default credentials to login 

username : admin  
password : admin123


### Troubleshooting

service is not starting?
 - make sure you are trying to start service with nexus user. 
- check java installation

Unable to access nexus URL?
- make sure port 8081 is opened in security group. 

### Next Steps
- [x] [Configure Users & Groups in Jenkins](https://youtu.be/jZOqcB32dYM)
- [x] [Secure your Jenkins Server](https://youtu.be/19FmJumnkDc)
- [x] [Jenkins Plugin Installation](https://youtu.be/p_PqPBbjaZ4)
- [x] [Jenkins Master-Slave Configuration](https://youtu.be/hwrYURP4O2k)

......................................



## SonarQube Integration with Jenkins

Integration SonarQube server with Jenkins is necessary to store your reports. Follow below steps to enable that.
### Follow this in **[YouTube](https://www.youtube.com/watch?v=k-3krTRuAFA)**

### Prerequisites
1. SonarQube Server **[Get Help here](https://www.youtube.com/watch?v=zRQrcAi9UdU)**
1. Jenkins Server  **[Get Help here](https://www.youtube.com/watch?v=M32O4Yv0ANc)**

### Implementation

Login to Jenkins server and install sonarqube scanner. 

- SonarQube scanner URL : https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner
- Package : https://sonarsource.bintray.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-3.2.0.1227-linux.zip
- 
```sh 
# wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.6.2.2472-linux.zip
# unzip sonar-scanner-cli-4.6.2.2472-linux.zip
# mv sonar-scanner-4.6.2.2472-linux /opt/sonar_scanner 
```

Set SonarQube server details in sonar-scanner property file 

 - Sonar properties file: /opt/sonar_scanner/conf/sonar-scanner.properties
   - sonar.host.url=http://`<SONAR_SERVER_IP>`:9000

Login to Jenkins GUI console and install " SonarQube scanner" plugin

 - `Manage Jenkins` > `Manage Plugins` > `Avalable` > `SonarQube scanner` 

Configure SonarQube scanner home path

- `Manage Jenkins` > `Global Tool Configuration` > `SonarQube Scanner` 
   - Name  : `sonar_scanner`
   - SONAR_RUNNER_HOME : `/opt/sonar_scanner`

Configure SonarQube server name and authentication token 
- `Manage Jenkins` > `Configure Systems` > `SonarQube Servers`
    - Name : `SonarQube`
	- ServerURL : `http://<Sonarqube_server>:9000/sonar`
	- Server `authentication token`
To Get Authentication code follow below steps.
	Login to SonarQube server as a admin  `My Account` > `Security` > `Generate Token`

Create a job to test SonarQube. Provide below sonar properties details in the job under build 
-SCM:
   ```
  - https://github.com/betawins/VProfile-1.git
   ```
- Build:
  - `Invoke top-level Maven targets`
          `Goals : clean install`
  - `Execute SonarQube Scanner` > `Analysis properties`  (it is mandatary). 
     ```sh 
    sonar.projectKey=Sabear
	sonar.projectName=Sabear
	sonar.projectVersion=1.0
	sonar.sources=/var/lib/jenkins/workspace/$JOB_NAME/src/
	sonar.binaries=target/classes/com/visualpathit/account/controller/
	sonar.junit.reportsPath=target/surefire-reports
	sonar.jacoco.reportPath=target/jacoco.exec
	sonar.java.binaries=src/main/java/com/visualpathit/account/
     ```
Execute job to get analysis report. 

.......................................................


# SonarQube Installation

SonarQube provides the capability to not only show health of an application but also to highlight issues newly introduced. With a Quality Gate in place, you can fix the leak and therefore improve code quality systematically.


### Prerequisites
1. EC2 instance with Java installed
2. Use t2.large with atleast 20gb memory to run sonarqube.
1. MySQL Database Server or MyQL RDS instance.

### Installation

Install java 1.8 version
```sh
yum install java-1.8*
```

Add mysql rpm Repository

 ```sh
yum update
sudo wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
sudo yum localinstall mysql57-community-release-el7-11.noarch.rpm
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
 
sudo yum install mysql-community-server
sudo systemctl start mysqld.service


 ```

Start MySQL and Enable Start at Boot Time
 
 ```sh
 systemctl start mysqld
 systemctl enable mysqld
 ```
 
 Check if mysql is running or not
 
  ```sh
netstat -na | grep 3306
 ```
 
 Configure the MySQL Root Password 
 You will see default MySQL root password
 
 ```sh
grep 'temporary' /var/log/mysqld.log
 ```
 Login to mysql using the default password
 ```sh
mysql -u root -p
 ```
 
 Now replace the default password with a new and strong password
 
 ```sh
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Admin@123';
flush privileges;
 ```
 Test Using new password
 
 ```sh
mysql -u root -p
 ```
  
Download stable SonarQube version from below website. 
- Website: https://www.sonarqube.org/downloads/
- Note: This Article written for SonarQube6.0  

Download & unzip SonarQube 6.0
```sh
# cd /opt
# wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-6.6.zip
# unzip sonarqube-6.6.zip
# mv /opt/sonarqube-6.6 /opt/sonar
```


Login to mysql
```sh 
mysql -u root -p
```
Create a new sonar database
```sh
CREATE DATABASE sonar CHARACTER SET utf8 COLLATE utf8_general_ci;
```

Create a local and a remote user
```sh
CREATE USER sonar@localhost IDENTIFIED BY 'Sonar@123';
CREATE USER sonar@'%' IDENTIFIED BY 'Sonar@123';
```

Grant database access permissions to users 
```sh
GRANT ALL ON sonar.* TO sonar@localhost;
GRANT ALL ON sonar.* TO sonar@'%';
```

check users and databases 
```sh
show databases;
SELECT User FROM mysql.user;
FLUSH PRIVILEGES;
QUIT
```
So for you have configured required database information on mysql. Let’s Jump back to your EC2 instance and enable SonarQube properties file to connect his Database.

### ON EC2 Instance
Edit sonar properties file to uncomment and provide required information for below properties. 

- File Name: /opt/sonar/conf/sonar.properties
  - sonar.jdbc.username=`sonar`
  - sonar.jdbc.password=`Sonar@123`
  - sonar.jdbc.url=jdbc:mysql://`localhost:3306`/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false
  - sonar.web.host=`0.0.0.0`
  - sonar.web.context=`/sonar`
  
# Sonar version 7 and higher cannot be run using root user,please switch to any other user and change the permissions to sonar directory and start sonar.

Create new user

```sh
# useradd sabear
# passwd sabear
```
Add user sabear to sudoers file
```sh
vi /etc/sudoers

## Same thing without a password
# %wheel        ALL=(ALL)       NOPASSWD: ALL
sabear          ALL=(ALL)       NOPASSWD: ALL
```
Restart the sshd service
```sh
# systemctl restart sshd
```
Switch to newly created user
```sh
# sudo su sabear
```
Start SonarQube service 
```sh
# cd /opt/sonar/bin/linux-x86-64/
# ./sonar.sh start
```

##### Run SonarQube as a default service 

Implement SonarQube server as a service
```sh
Copy sonar.sh to etc/init.d/sonar and modify it according to your platform.
# sudo cp /opt/sonar/bin/linux-x86-64/sonar.sh /etc/init.d/sonar
# sudo vi /etc/init.d/sonar
```

Add below values to your /etc/init.d/sonar file
```sh
Insert/modify below values
SONAR_HOME=/opt/sonar
PLATFORM=linux-x86-64

WRAPPER_CMD="${SONAR_HOME}/bin/${PLATFORM}/wrapper"
WRAPPER_CONF="${SONAR_HOME}/conf/wrapper.conf"
PIDDIR="/var/run"
```

Start SonarQube server
```sh
# service sonar start
```
SonarQube application uses port 9000. access SonarQube from browser
```sh
  http://<EC2_PUBLIC_IP>:9000/sonar
```
Default credentials for sonarqube.
Username : admin
Password: admin
###  NOTES
1) Check whether you enabled port 9000 in EC2 instance security group

### Important Points:

1) mysql port number : 3306
2) sonarqube port: 9000
3) sonarqube logs : /opt/sonar/logs
4) We can find four different logs
 a)access.log
 b)es.log aka elastic search
 c)sonar.log
 d)web.log
   
 
### Videos for reference: 
### sonarqube installation: 
https://www.youtube.com/watch?v=zRQrcAi9UdU
#### mysql rds instance aws: 
https://www.youtube.com/watch?v=vLaW6b441x0

