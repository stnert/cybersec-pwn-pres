# cybersec-pwn-pres

Uma exploração para Apache Struts CVE-2017-5638

Seguimos estas etapas para configurar uma instância vulnerável do Struts 2 para testes - CVE-2017-5638

As instruções são para Ubuntu Server, mas devem funcionar com outras distros. 

Passo 1 - Instale o Oracle Java JDK 8 

```
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

´´´
# Passo 2 - Instale o Tomcat

´´´
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


