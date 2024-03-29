# cybersec-pwn-pres - Uma exploração para Apache Struts - CVE-2017-5638

Seguimos estas etapas para configurar uma instância vulnerável do Struts 2 para testes - CVE-2017-5638

As instruções são para Ubuntu Server, mas devem funcionar com outras distros. 

# O que é CVE-2017-5638?

Essa é uma vulnerabilidade presente na versão 2.5.10 do Apache Struts que permite realizar injeções de comando
por meio da análise incorreta do cabeçalho HTTP Content-Type de um invasor. Ela 
permite que esses comandos sejam executados sob privilégios do servidor Web.<br>
O código vulnerável está no analisador Jakarta Multipart. Se o valor Content-Type não for válido, ou seja, não corresponder a um tipo válido esperado, será lançada uma exceção que será usada para exibir uma mensagem de erro a um usuário.<br>
A vulnerabilidade ocorre porque o Content-Type é usado pela função LocalizedTextUtil.findText para criar a mensagem de erro. Esta função interpreta a mensagem fornecida, e qualquer coisa dentro de ${…} será tratada como uma expressão Object Graph Navigation Library (OGNL). O invasor pode aproveitar essas condições para executar expressões OGNL que, por sua vez, executam comandos do sistema.

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

Adicione esta linha ao final do arquivo, conforme mostrado abaixo.

export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_171

Salve o arquivo com :wq, Enter.

Servidor Ubuntu, execute este comando para definir a nova variável de ambiente:

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

# Passo 6 - Criando o projeto 
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
![Imagem do WhatsApp de 2024-01-25 à(s) 01 00 47_3963e635](https://github.com/stnert/cybersec-pwn-pres/assets/100847921/0fd399a8-8401-4c09-adf8-0bc35a897c0a)
<div align="center"> Aplicação WEB Vulnerável </div>

# Fazendo o Deploying do Web App
```
Na página "Tomcat Web Application Manager", na sessão "Deploy", na sessão "WAR file to deploy", clique no botão "Choose File".

Navegue até a pasta Downloads e clique duas vezes no arquivo struts2_2.3.15.1-showcase.war.

Selecione para fazer o Deploy. 

A página do Tomcat agora mostra o aplicativo /struts2_2.3.15.1-showcase na parte inferior da seção Aplicativos, conforme mostrado abaixo
Clique em /struts2_2.3.15.1-showcase.
A página "Struts2 Showcase" deve aparecer.

http://blog.ud64.com/2017/09/apache-struts-with-cve-2017-5638-set-up.html
```
![Imagem do WhatsApp de 2024-01-25 à(s) 01 01 07_bd4cb609](https://github.com/stnert/cybersec-pwn-pres/assets/100847921/7c371654-bde2-4ec1-a384-524c29f7bd24)
<div align="center"> Painel de Administração para o Deploy </div>

# Constatando a Vulnerabilidade
Para checar se há a vulnerabilidade, é só executar o comando:
```
python3 struts-pwn.py --check --url 'http://192.168.0.17:8080/struts2_2.3.15.1-showcase/showcase.action'
```
O script vai então montar o payload com uma string aleatória e verificar se está igual ao retornado na resposta para constatar a vulnerabilidade:
![Imagem do WhatsApp de 2024-01-24 à(s) 22 35 39_4b78cf13](https://github.com/stnert/cybersec-pwn-pres/assets/100847921/5659a79c-c9f7-442a-b2a8-9dac89dc1eaa)
<div align="center"> Sem Vulnerabilidade </div>

<br>

![Imagem do WhatsApp de 2024-01-25 à(s) 01 01 26_9dc3e774](https://github.com/stnert/cybersec-pwn-pres/assets/100847921/45335df1-926d-41ce-9d0a-42422fdefd9a)
<div align="center"> Vulnerabilidade Constatada </div>

# Verificando a Vulnerabilidade
Utilizando o script, é possível passar os comandos shell para serem explorados na aplicação:

![Imagem do WhatsApp de 2024-01-25 à(s) 21 24 35_1e0ca66f](https://github.com/stnert/cybersec-pwn-pres/assets/100847921/6a9631a6-f835-4b31-ae89-d797d22f5444)
<div align="center"> Execução do comando ls </div>

<br>

![Imagem do WhatsApp de 2024-01-25 à(s) 21 27 01_f14b27d0](https://github.com/stnert/cybersec-pwn-pres/assets/100847921/a4a89a0a-8db4-46e0-9e19-8d051b0bd99b)
<div align="center"> Execução do comando ls -la </div>

<br>

![Imagem do WhatsApp de 2024-01-25 à(s) 21 25 56_7f1f73ea](https://github.com/stnert/cybersec-pwn-pres/assets/100847921/fbf5bbd2-80c4-4eec-a81e-90e22df2ac0b)
![Imagem do WhatsApp de 2024-01-25 à(s) 21 26 06_07cd7c7b](https://github.com/stnert/cybersec-pwn-pres/assets/100847921/59b7033f-1a2d-4418-8b41-24e68df88b9b)
<div align="center"> Execução do comando mkdir teste2 </div>


