# maven-project

Simple Maven Project Jenkins CICD DEMO

Steps to trigger CI-CD maven job and deploy artifact into tomcat server using Jenkins "Deploy to container Plugin" plugin.

1. Install jenkins server on any platform like say docker

   sudo docker run -d -p 8080:8080 --name jenkins-server jenkins/jenkins

2. Once jenkins server started go to Manage Jenkins --> Manage Plugins  & from available plugins tab, select and install "Deploy to container Plugin" plugin.

3. Now on separate server(Local Virtual Box VM) install JAVA 8 
 
   sudo apt update
   
   sudo apt-get install openjdk-8-jdk -y
   
   export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
    
   echo $JAVA_HOME
   
4. Now on same server install tomcat 8 using below commands
  
   wget https://mirrors.estointernet.in/apache/tomcat/tomcat-8/v8.5.65/bin/apache-tomcat-8.5.65.tar.gz
   
   sudo mv  apache-tomcat-8.5.65.tar.gz /opt/
   
   cd /opt
   
   sudo su
   
   tar -xvzf apache-tomcat-8.5.65.tar.gz
   
   nano apache-tomcat-8.5.65/webapps/host-manager/META-INF/context.xml and comment these  below lines 
   
   ```
     
       <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />  
   ```
  
   nano apache-tomcat-8.5.65/webapps/manager/META-INF/context.xml and comment these  below lines 
     
   ```
       <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />   
   ```         

   nano apache-tomcat-8.5.65/conf/tomcat-users.xml  and add these below lines just before "/tomcat-users" tag.
   
   ```
      
       <role rolename="manager-gui"/>
       <role rolename="manager-script"/>
       <role rolename="manager-jmx"/>
       <role rolename="manager-status"/>

       <user username="admin" password="admin" roles="manager-gui, manager-script, manager-jmx, manager-status"/>
       <user username="deployer" password="deployer" roles="manager-script"/>
       <user username="tomcat" password="s3cret" roles="manager-gui"/>
       
   ```       

5. Now start your tomcat server by running below commands
  
   cd /opt/apache-tomcat-8.5.65/bin/
   
   chmod +x start.sh shutdown.sh
   
   # To start tomcat server
     ./start.sh
   
   # To stop tomcat server
     ./shutdown.sh

6.  Now, once you start tomcat server, hit your browser at following address http://tomcat-server-ip:8080/manager/html to access tomcat server manager page and  when promted for credentials, give username and password provided in  tomcat-user.xml file like Username: tomcat and password: s3cret

7. Now in jenkins server go to Manage-Jenkins-->Manage-Credentials-->click on Global --> Add credentials and give user credentials (username: deployer & password: deployer) and give creds some ID like 'tomcat-ID' for deploying war into tomcat server added in tomcat-users.xml file.

8. Now from jenkins dashboard menu click on 'New Item' --> give some name to job/project like maven-CICD , select 'Maven Project' & click 'Ok'

9. On configure page, under 'Source Code' tab/section select 'Git' and add this repo url 'https://github.com/devopsenggineer/hello-world.git', 

10. Move to 'Build-Triggers' tab/section, select 'Poll SCM' and give appropriate scheduling like (*/10 * * * *) to check for code changes in this git repo every 10 minutes 

11. Move to 'Build' tab/section  & under "Goals and options" field add these values 'clean install package'.

12. Move to 'Post-build Actions' select 'Deploy war/ear to a container', in 'war/ear file' field enter this '**/*.war'.

13. Click 'Add a container' & select 'Tomcat 8.x Remote' and credetials field give tomcat server credentials ID provided at jenkins credentials page & in 'Tomcat Url' field give tomcat server url like "http://<tomcat-server-ip>:8080/"

14. Click "Aplly" & "Save"

15. Now build the job, once is succeeded hit this url "http://<tomcat-server-ip>:8080/webapp/" to see changes deployed to tomcat server.

16. Also, tomcat server can be checked to  view new files added at "/opt/apache-tomcat-8.5.65/webapps/" location using below command

     ls -ltrs /opt/apache-tomcat-8.5.65/webapps/ 

17. Now if you make any changes in this repo and commit code changes, jenkins will trigger CI-CD process.
    
  
      
  

