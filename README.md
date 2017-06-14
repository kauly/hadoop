# Guia para instalar o Hadoop em modo multi cluster.


CRIAR USUARIO hadoop
´´´sudo adduser hadoop´´´

ADD USER HADOOP NO GRUPO SUDO
sudo visudo
#includedir /etc/sudoers.d
hadoop ALL=(ALL) ALL

LOGAR COM USUARIO HADOOP
su hadoop

ADICIONAR IP DO MASTER/SLAVES NO HOSTS
sudo vi /etc/hosts
127.0.0.1	localhost
127.0.1.1	PC-Kauly
192.168.1.103   master
192.168.1.104   slave-01

BAIXAR JAVA
sudo add-apt-repository ppa:openjdk-r/ppa
sudo apt-get update
sudo apt-get install openjdk-8-jdk

VERIFICAR DIRETORIO QUE O JAVA FOI INSTALADO
sudo update-alternatives --config java
java -version

BAIXAR SSH (mesmo se ja estiver instalado!)
sudo apt-get install openssh-server

GERAR PAR DE CHAVES
ssh-keygen -t rsa -P ""     *enter..enter..

HABILITAR ACESSO SSH NO MASTER
cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys

TESTAR ACESSO SSH SEM SENHA 
ssh localhost

MANDAR CHAVE PARA OS SLAVES
ssh-copy-id -i $HOME/.ssh/id_rsa.pub hadoop@slave

TESTAR ACESSO SSH SEM SENHA AOS SLAVES
ssh slave

BAIXAR HADOOP
cd ~
wget http://ftp.unicamp.br/pub/apache/hadoop/common/hadoop-2.8.0/hadoop-2.8.0.tar.gz
tar -xzf hadoop-2.8.0.tar.gz

CRIAR VARIAVEIS DE AMBIENTE
---- vi ~/.bashrc - --- VERIFICAR CAMINHOS!!
export HADOOP_HOME=/home/hadoop/hadoop-2.8.0
export PATH=$PATH:$HADOOP_HOME/bin
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$PATH:$JAVA_HOME/bin
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
export HADOOP_INSTALL=$HADOOP_HOME
---  source ~/.bashrc ----


CRIAR ARQUIVO MASTERS 
cd /home/hadoop/hadoop-2.8.0/etc/hadoop/
vi masters
master

SETAR PATH PARA O JAVA
vi hadoop-env.sh
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
ARRUMAR WARNING
export HADOOP_HOME_WARN_SUPPRESS=1
export HADOOP_ROOT_LOGGER="WARN,DRFA"

ARQUIVOS DE CONFIGURAÇÃO
vi core-sites.xml  //localização do master(namenode)
<configuration>
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://master:9000</value>
	</property>
</configuration>


vi hdfs-sites.xml //config do hdfs
<configuration>
   <property>
      <name>dfs.replication</name>
      <value>3</value>
   </property>
   <property>
   	<name>dfs.permissions</name>
	<value>false</value>
   </property>
   <property> //SO NO MASTER	   
      <name>dfs.name.dir</name>
      <value>file:///home/hadoop/hadoopinfra/hdfs/namenode</value>
   </property>///
   <property>
      <name>dfs.data.dir</name> 
      <value>file:///home/hadoop/hadoopinfra/hdfs/datanode</value> 
   </property>
</configuration>


vi yarn-site.xml
<configuration>
   <property>
      <name>yarn.nodemanager.aux-services</name>
      <value>mapreduce_shuffle</value> 
   </property>
</configuration>


cp mapred-site.xml.template mapred-site.xml
vi mapred-site.xml
<configuration>	
   <property> 
      <name>mapreduce.framework.name</name>
      <value>yarn</value>
   </property>
</configuration>

ENVIAR HADOOP PARA OS SLAVES
sudo rsync -avxP /home/hadoop/hadoop-2.8.0 hadoop@slave:/home/hadoop

CRIAR ARQUIVO SLAVES
vi slaves
master
slave
 
ADD LOC DO NAMENODE hdfs-site.xml
<property> //SO NO MASTER	   
      <name>dfs.name.dir</name>
      <value>file:///home/hadoop/hadoopinfra/hdfs/namenode</value>
</property>///

CRIAR/FORMATAR DIRETORIO NAMENODE
hadoop namenode -format

INICIAR
start-dfs.sh

INTERFACE WEB
http://master:50070/   hdfs
http://localhost:50030/ 
http://localhost:50060/ 

CRIAR DIRETORIO
hdfs dfs -mkdir /primeiro

ENVIAR PARA O HDFS
hdfs dfs -put data /primeiro

MAPREDUCE
hadoop jar hadoop-2.8.0/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.8.0.jar wordcount /primeiro/data /primeiro/data-output

PEGAR ARQ DO HDFS
hdfs dfs -get /primeiro/data-output/part-r-00000 .