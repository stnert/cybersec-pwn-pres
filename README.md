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

# Passo 3 - Instalando o Unzip
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

Adicione esta linha ao final do arquivo, conforme mostrado abaixo.

export PATH=$PATH:/opt/apache-maven-3.5.0/bin

Salve o arquivo (Ctrl+X) + Y + Enter.

Servidor Ubuntu, execute este comando para definir a nova variável de ambiente:
source .bashrc

Servidor Ubuntu, execute este comando para definir a nova variável de ambiente:
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

Na parte inferior do arquivo, na seção "dependências", adicione uma nova sessão "dependência", Incluir na sessão <dependências>:

<dependency>
<groupId>org.apache.struts</groupId>
<artifactId>struts2-core</artifactId>
<version>2.5.10</version>
</dependency>
Salve o arquivo (Ctrl+X) + Y + Enter.

Para tornar seu aplicativo web um servidor Ubuntu, execute este comando:
mvn clean package
Many pages of "Downloading" messages scroll by, ending with a green "BUILD SUCCESS" message
This has created a "war" file, ready to deploy, at this location:
~/myWebApp/target/basic_struts.war
However, we don't actually need that application. We'll deploy a different one later.
```

# Passo 7 - Configurando o Web-Based Deployment
```
cd
nano .bashrc
Adicione esta linha ao final do arquivo, conforme mostrado abaixo.
export CATALINA_HOME=/usr/local/tomcat

Salve o arquivo (Ctrl+X) + Y + Enter.

source .bashrc
Agora precisamos ajustar a configuração do Tomcat para permitir a administração de endereço remoto.

sudo nano $CATALINA_HOME/conf/tomcat-users.xml
The "tomcat-users" section contain only comments,
Insert these lines into the "tomcat-users" section,
<role rolename="manager-gui" />
<user username="admin" password="admin" roles="manager-gui"/>

Salve o arquivo com Ctrl+X, Y, Enter.
Ubuntu server, execute esses comandos:

sudo nano $CATALINA_HOME/conf/Catalina/localhost/manager.xml

Insira essas linhas no arquivo, conforme mostrado abaixo.

<Context privileged="true" antiResourceLocking="false"
docBase="${catalina.home}/webapps/manager">
<Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="^.*$" />
</Context>

Salve o arquivo (Ctrl+X) + Y + Enter.

Ubuntu server, execute these commands to restart Tomcat. It may take a few minutes to shut down the first time--that's OK.
sudo $CATALINA_HOME/bin/shutdown.sh
sudo $CATALINA_HOME/bin/startup.sh
Tomcat restarts,
```

# Passo 8 - Abra a página de Administração Web 
```
http://IP:8080/manager

Uma caixa aparece solicitando credenciais. Insira estas credenciais:
Username: admin
Password: admin

Na página "Tomcat Web Application Manager", role para baixo até a seção "Implantar"
```

# Passo 9 - Baixando o Web App Vulnerável 
```
No seu sistema, abra o navegador Web em:
https://github.com/nixawk/labs/blob/master/CVE-2017-5638/struts2_2.3.15.1-showcase.war
No lado direito,clique no botão de Download 
Você verá um arquivo chamado: struts2_2.3.15.1-showcase.war
```

# Fazendo o Deploying do Web App
```
Na página "Tomcat Web Application Manager", na sessão "Deploy", na sessão "WAR file to deploy", clique no botão "Choose File".

Navegue até a pasta Downloads e clique duas vezes no arquivo struts2_2.3.15.1-showcase.war.

Selecione para fazer o Deploy. 

A página do Tomcat agora mostra o aplicativo /struts2_2.3.15.1-showcase na parte inferior da seção Aplicativos, conforme mostrado abaixo
Clique em /struts2_2.3.15.1-showcase.
A página "Struts2 Showcase" deve aparecer.

http://blog.ud64.com/2017/09/apache-struts-with-cve-2017-5638-set-up.html
