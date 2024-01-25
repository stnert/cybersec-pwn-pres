# cybersec-pwn-pres - Uma exploração para Apache Struts - CVE-2017-5638

Seguimos estas etapas para configurar uma instância vulnerável do Struts 2 para testes - CVE-2017-5638

As instruções são para Ubuntu Server, mas devem funcionar com outras distros. 



# Passo 1 - Instalando o JDK8

```

Abra o link: http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html

Download jdk-8u171-linux-x64.tar.gz copy to /tmp

cd /tmp

tar -xvf jdk-8u171-linux-x64.tar.gz

sudo mkdir -p /usr/lib/jvm

sudo mv ./jdk1.8.0* /usr/lib/jvm/

sudo update-alternatives --install "/usr/bin/java" "java" "/usr/lib/jvm/jdk1.8.0_171/bin/java" 1

sudo update-alternatives --install "/usr/bin/javac" "javac" "/usr/lib/jvm/jdk1.8.0_171/bin/javac" 1

sudo update-alternatives --install "/usr/bin/javaws" "javaws" "/usr/lib/jvm/jdk1.8.0_171/bin/javaws" 1

sudo chmod a+x /usr/bin/java

sudo chmod a+x /usr/bin/javac

sudo chmod a+x /usr/bin/javaws

sudo chown -R root:root /usr/lib/jvm/jdk1.8.0_171

sudo update-alternatives --config java

sudo update-alternatives --config javac

sudo update-alternatives --config javaws

If you see "nothing to configure" that's OK.
java -version

```

# Passo 2 - Instalando o TomCat

```
Download...
https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.0.M26/bin/apache-tomcat-9.0.0.M26.tar.gz

tar xvzf apache-tomcat-9.0.0.M26.tar.gz

sudo mkdir /usr/local/tomcat

sudo mv apache-tomcat-9.0.0.M26/* /usr/local/tomcat

Ubuntu server, execute these commands:

cd

vi .bashrc

Add this line to the bottom of the file, as shown below.

export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_171

Save the file with :wq, Enter.

Ubuntu server, execute this command to set the new environment variable:

source .bashrc

Ubuntu server, execute this command to start Tomcat:

/usr/local/tomcat/bin/startup.sh

Tomcat starts

On your host system, in a Web browser, open this URL, replacing the IP.

http://System_IP:8080/

You see an Apache Tomcat page.

```

# Passo 3 - Instaland oo Unzip
```
sudo apt update
sudo apt install unzip
```

# Passo 4 -  Instale Struts2 (Old, Vulnerable Version)
```
cd
wget http://archive.apache.org/dist/struts/2.5.10/struts-2.5.10-all.zip
unzip struts-2.5.10-all.zip
mv struts-2.5.10 struts2
```

# Passo 5 - Instalando o Maven
```
cd /tmp
wget https://repo.maven.apache.org/maven2/org/apache/maven/apache-maven/3.5.0/apache-maven-3.5.0-bin.tar.gz
sudo tar xvzf apache-maven*.tar.gz -C /opt/
cd
nano .bashrc
Add this line to the bottom of the file, as shown below.
export PATH=$PATH:/opt/apache-maven-3.5.0/bin
Save the file with Ctrl+X, Y, Enter.
Ubuntu server, execute this command to set the new environment variable:
source .bashrc
In the SSH session controlling your Ubuntu server, execute this command:
mvn -version
Verique a versão
```

# Passo 6: Criando o projeto 
Ubuntu server, execute esses comandos:
```
cd
mvn archetype:generate \
-DgroupId=com.tutorialforlinux \
-DartifactId=myWebApp \
-DarchetypeArtifactId=maven-archetype-webapp
Many pages of "Downloading" messages scroll by.
When you see the message: "Define value for property 'version' 1.0-SNAPSHOT: :", press Enter.
When you see the message: "Y: :", press Enter.
You see a "BUILD SUCCESS" message

Ubuntu server, execute esses comando:
cd myWebApp
nano pom.xml
The file opens, as shown below. This is an XML configuration file.
At the bottom of the file, in the "build" section, change myWebApp to basic_struts,
<build>
<finalName>basic_struts</finalName>
</build>
At the bottom of the file, in the "dependencies" section, add a new "dependency" section, Include in the <dependencies> Section:

<dependency>
<groupId>org.apache.struts</groupId>
<artifactId>struts2-core</artifactId>
<version>2.5.10</version>
</dependency>
Save the file with Ctrl+X, Y, Enter.

To make your web app,Ubuntu server, execute this command:
mvn clean package
Many pages of "Downloading" messages scroll by, ending with a green "BUILD SUCCESS" message
This has created a "war" file, ready to deploy, at this location:
~/myWebApp/target/basic_struts.war
However, we don't actually need that application. We'll deploy a different one later.
```

# Passo 6 - Configurando o Web-Based Deployment
```
cd
nano .bashrc
Add this line to the bottom of the file, as shown below.
export CATALINA_HOME=/usr/local/tomcat

Save the file with Ctrl+X, Y, Enter.

source .bashrc
Now we need to adjust the tomcat configuration to allow administration from remote addresses.
Ubuntu server, execute this command:
sudo nano $CATALINA_HOME/conf/tomcat-users.xml
The "tomcat-users" section contain only comments,
Insert these lines into the "tomcat-users" section,
<role rolename="manager-gui" />
<user username="admin" password="admin" roles="manager-gui"/>

Save the file with Ctrl+X, Y, Enter.
Ubuntu server, execute esses comandos:

sudo nano $CATALINA_HOME/conf/Catalina/localhost/manager.xml

Insert these lines into the file, as shown below.

<Context privileged="true" antiResourceLocking="false"
docBase="${catalina.home}/webapps/manager">
<Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="^.*$" />
</Context>
Save the file with Ctrl+X, Y, Enter.
Ubuntu server, execute these commands to restart Tomcat. It may take a few minutes to shut down the first time--that's OK.
sudo $CATALINA_HOME/bin/shutdown.sh
sudo $CATALINA_HOME/bin/startup.sh
Tomcat restarts,
```

# Passo 7 - Abra a página de Administração Web 
```
http://IP:8080/manager
A box pops up asking for credentials. Enter these credentials:
Username: admin
Password: admin

In the "Tomcat Web Application Manager" page, scroll down to the "Deploy" section
```

# Passo 8 - Baixando o Web App Vulnerável 
```
No seu sistema, abra o navegador Web em:
https://github.com/nixawk/labs/blob/master/CVE-2017-5638/struts2_2.3.15.1-showcase.war
No lado direito,clique no botão de Download 
Você verá um arquivo chamado: struts2_2.3.15.1-showcase.war
```

# Passo 9: Fazendo o Deploying the Vulnerable Web App
```
Na página "Tomcat Web Application Manager", na sessão "Deploy", na seção "WAR file to deploy", clique no botão "Choose File".

Navegue até a pasta Downloads e clique duas vezes no arquivo struts2_2.3.15.1-showcase.war.

Selecione para fazer o Deploy. 

A página do Tomcat agora mostra o aplicativo /struts2_2.3.15.1-showcase na parte inferior da seção Aplicativos, conforme mostrado abaixo
Clique em /struts2_2.3.15.1-showcase.
A página "Struts2 Showcase" deve aparecer.

http://blog.ud64.com/2017/09/apache-struts-with-cve-2017-5638-set-up.html


