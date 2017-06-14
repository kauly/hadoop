# Guia para instalar o Hadoop em modo multi cluster.


Criar usuário hadoop:

```shell
sudo adduser hadoop
```
Dar privilegios de root ao usuario:

```shell
sudo visudo
#includedir /etc/sudoers.d
hadoop ALL=(ALL) ALL
```

Logar com usuario hadoop:

```shell
su hadoop
```

Adicionar IP do MASTER/SLAVES no arquivo de hosts do master:

```shell
sudo vi /etc/hosts
127.0.0.1	localhost
127.0.1.1	PC-Kauly
192.168.1.103   master
192.168.1.104   slave-01
```

Baixar java:

```shell
sudo add-apt-repository ppa:openjdk-r/ppa
sudo apt-get update
sudo apt-get install openjdk-8-jdk
```

Verificar caminho onde o java foi instalado:

```shell
sudo update-alternatives --config java
java -version
```

Baixar ssh:

```shell
sudo apt-get install openssh-server
```

Gerar par de chaves:

```shell
ssh-keygen -t rsa -P ""
```

Habilitar acesso ssh no master:

```shell
cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys
```

Mandar chave aos slaves:

```shell
ssh-copy-id -i $HOME/.ssh/id_rsa.pub hadoop@slave
```


Testar acesso ssh sem senha, de preferencia em todos os nos:

```shell
ssh master
ssh slave-01
```

Baixar o hadoop:

```shell
cd ~
wget http://ftp.unicamp.br/pub/apache/hadoop/common/hadoop-2.8.0/hadoop-2.8.0.tar.gz
tar -xzf hadoop-2.8.0.tar.gz
```
Criar variaveis de ambiente:

```shell
vi ~/.bashrc
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
```
Atualizar arquivo .bashrc:

```shell
source ~/.bashrc
```

Criar arquivo "masters" com o ip do master(foi salvo como "master" no /etc/hosts):

```shell
cd /home/hadoop/hadoop-2.8.0/etc/hadoop/
vi masters
master
```

Setar caminho para o java:

```shell
vi hadoop-env.sh
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
ARRUMAR WARNING
export HADOOP_HOME_WARN_SUPPRESS=1
export HADOOP_ROOT_LOGGER="WARN,DRFA"
```

Arquivos de configuração que requerem atenção:

```shell
cd /home/hadoop/hadoop-2.8.0/etc/hadoop/
```

```shell
vi core-sites.xml
```

```
<configuration>
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://master:9000</value>
	</property>
</configuration>
```

```shell
vi hdfs-sites.xml
```

```
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
```

```shell
vi yarn-site.xml
```

```
<configuration>
   <property>
      <name>yarn.nodemanager.aux-services</name>
      <value>mapreduce_shuffle</value>
   </property>
</configuration>
```

```shell
cp mapred-site.xml.template mapred-site.xml
vi mapred-site.xml
```

```
<configuration>
   <property>
      <name>mapreduce.framework.name</name>
      <value>yarn</value>
   </property>
</configuration>
```

Enviar o hadoop para os slaves:
```shell
sudo rsync -avxP /home/hadoop/hadoop-2.8.0 hadoop@slave:/home/hadoop
```

Volte a pasta de config. e crie o arquivo "slaves":

```shell
cd /home/hadoop/hadoop-2.8.0/etc/hadoop/
vi slaves
master
slave-01
slave-02
etc..
```

Adicionar diretorio onde vai ser montado o sistema hdfs:

```shell
vi hdfs-site.xml
<property> //SO NO MASTER
      <name>dfs.name.dir</name>
      <value>file:///home/hadoop/hadoopinfra/hdfs/namenode</value>
</property>///
```


Criar/Formatar namenode:
```shell
hadoop namenode -format
```

Iniciar hdfs:
```shell
start-dfs.sh
```

Interface web:

http://master:50070/

## Exemplos

Criar diretorio:
```shell
hdfs dfs -mkdir /primeiro
```

Enviar arquivo de sistema local para o hdfs:
```shell
hdfs dfs -put data /primeiro
```
Executar exemplo de mapreduce:
```shell
hadoop jar hadoop-2.8.0/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.8.0.jar wordcount /primeiro/data /primeiro/data-output
```

Transferir arquivo do hdfs para o sistema local:
```shell
hdfs dfs -get /primeiro/data-output/part-r-00000 .
```
