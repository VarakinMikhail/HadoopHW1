Задание:

Необходимо задокументировать или реализовать автоматизированное развертывание кластера hdfs, включающего в себя 3 DataNode и обязательные для функционирования кластера сервисы: NameNode, Secondary NameNode.

Данные:

узел для входа **176.109.81.242**

jn **192.168.1.106**

nn **192.168.1.107** 

dn-00 **192.168.1.109** 

dn-01 **192.168.1.108**

Подключаемся:
```
ssh -v team@176.109.81.242
```

Устанавливаем tmux:
```
sudo apt install tmux
```

И запускаем его:
```
tmux
```

генерируем ssh:
```
ssh-keygen
```

И добавляем его в число авторизованных:
```
cat .ssh/id_ed25519.pub >> .ssh/authorized_keys
```

Можно проверить, что он есть (и совпадает), с помощью команд:
```
cat .ssh/id_ed25519.pub
```
```
cat .ssh/authorized_keys
```


Копируем ssh-ключ по всем узлам:
```
scp .ssh/authorized_keys 192.168.1.106
```
```
scp .ssh/authorized_keys 192.168.1.107
```
```
scp .ssh/authorized_keys 192.168.1.109
```
```
scp .ssh/authorized_keys 192.168.1.108
```

Изменим файл хоста
```
sudo vim /etc/hosts
```

На вот такой:
```
# 127.0.0.1 localhost
127.0.0.1 tmpl-jn

# 192.168.1.106   tmpl-jn
192.168.1.107   tmpl-nn
192.168.1.109   tmpl-dn-00
192.168.1.108   tmpl-dn-01

# The following lines are desirable for IPv6 capable hosts
# ::1     ip6-localhost ip6-loopback
# fe00::0 ip6-localnet
# ff00::0 ip6-mcastprefix
# ff02::1 ip6-allnodes
# ff02::2 ip6-allrouters
```

Проверим, что изменения вошли в силу (нажимаем ctrl+C, чтобы остановить):
```
ping tmpl-nn
```
```
ping tmpl-dn-00
```
```
ping tmpl-dn-01
```

Добавим пользователя, от имени которого будут выполняться сервисы Hadoop:
```
sudo adduser hadoop
```

Повторим описанные действия для всех остальных узлов:
```
ssh 192.168.1.107
```
```
sudo adduser hadoop
```
```
sudo vim /etc/hosts
```

И поправляем файл:
```
# 127.0.0.1 localhost
# 127.0.0.1 tmpl-nn

192.168.1.106   tmpl-jn
192.168.1.107   tmpl-nn
192.168.1.109   tmpl-dn-00
192.168.1.108   tmpl-dn-01

# The following lines are desirable for IPv6 capable hosts
# ::1     ip6-localhost ip6-loopback
# fe00::0 ip6-localnet
# ff00::0 ip6-mcastprefix
# ff02::1 ip6-allnodes
# ff02::2 ip6-allrouters
```

Идем к следующей ноде:
```
ssh 192.168.1.109
```
```
sudo adduser hadoop
```
```
sudo vim /etc/hosts
```
Вот файл:
```
# 127.0.0.1 localhost
127.0.0.1 tmpl-dn-00

192.168.1.106   tmpl-jn
192.168.1.107   tmpl-nn
#192.168.1.109   tmpl-dn-00
192.168.1.108   tmpl-dn-01

# The following lines are desirable for IPv6 capable hosts
# ::1     ip6-localhost ip6-loopback
# fe00::0 ip6-localnet
# ff00::0 ip6-mcastprefix
# ff02::1 ip6-allnodes
# ff02::2 ip6-allrouters
```
И последняя нода:
```
ssh 192.168.1.108
```
```
sudo adduser hadoop
```
```
sudo vim /etc/hosts
```
Файл:
```
# 127.0.0.1 localhost
127.0.0.1 tmpl-dn-01

192.168.1.106   tmpl-jn
192.168.1.107   tmpl-nn
192.168.1.109   tmpl-dn-00
#192.168.1.108   tmpl-dn-01

# The following lines are desirable for IPv6 capable hosts
# ::1     ip6-localhost ip6-loopback
# fe00::0 ip6-localnet
# ff00::0 ip6-mcastprefix
# ff02::1 ip6-allnodes
# ff02::2 ip6-allrouters
```

Переключимся на пользователя, который был создан
```
sudo -i -u hadoop
```

Скачаем hadoop:
```
wget https://dlcdn.apache.org/hadoop/common/hadoop-3.4.0/hadoop-3.4.0.tar.gz
```

От пользователя распространяем ключ на остальные ноды:
```
ssh-keygen
```
```
cat .ssh/id_ed25519.pub > .ssh/authorized_keys
```

Копируем ключ по узлам:
```
scp -r .ssh/ tmpl-nn:/home/hadoop
```
```
scp -r .ssh/ tmpl-dn-00:/home/hadoop
```
```
scp -r .ssh/ tmpl-dn-01:/home/hadoop
```

На этом этапе можно проверить, что можно подключиться к нодам по новым именам (на примере NameNode):
```
ssh tmpl-nn
```

Скопируем hadoop на все узлы:
```
scp hadoop-3.4.0.tar.gz tmpl-nn:/home/hadoop
```
```
scp hadoop-3.4.0.tar.gz tmpl-dn-00:/home/hadoop
```
```
scp hadoop-3.4.0.tar.gz tmpl-dn-01:/home/hadoop
```

Теперь можно приступить к настройке hadoop, но для начала его надо распаковать:
```
tar -xzvf hadoop-3.4.0.tar.gz
```

Переходим на NameNode:
```
ssh tmpl-nn
```
```
tar -xzvf hadoop-3.4.0.tar.gz
```

Переходим на DataNode0:
```
ssh tmpl-dn-00
```
```
tar -xzvf hadoop-3.4.0.tar.gz
```

Переходим на DataNode1:
```
ssh tmpl-dn-01
```
```
tar -xzvf hadoop-3.4.0.tar.gz
```

Переходим на JumpNode (проверяем, что переход работает):
```
ssh tmpl-jn
```

И возвращаемся назад:
```
exit
```
```
exit
```
```
exit
```
```
exit
```

Оказываемся снова на JumpNode.
Проверим версию java (есть ли совместимость с hadoop):
```
java -version
```
```
which java
```
```
readlink -f /usr/bin/java
```
(путь скопирован из which java)

Таким образом мы получили путь, где лежит bin:
```
/usr/lib/jvm/java-11-openjdk-amd64/bin/java
```

И редактируем профиль (добавляя переменные окружения):
```
vim .profile
```

Добавляем
```
export HADOOP_HOME=/home/hadoop/hadoop-3.4.0
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

И активируем:
```
source .profile
```

Теперь проверяем версию hadoop (чтобы проверить, что он работает):
```
hadoop version
```

Теперь это надо скопировать на все узлы:
```
scp .profile tmpl-dn-01:/home/hadoop
```
```
scp .profile tmpl-dn-00:/home/hadoop
```
```
scp .profile tmpl-nn:/home/hadoop
```

Переходим в папку с конфигами hadoop:
```
cd hadoop-3.4.0/etc/hadoop
```

Изменим скрипт, устанавливающий переменные окружения
```
vim hadoop-env.sh
```
Добавляем строку:
```
JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
```
(путь, который ранее получили readlink)

Правим другой конфиг:
```
vim core-site.xml
```
В configuration пишем следующее:
```
<property>
	<name>fs.defaultFS</name>
	<value>hdfs://tmpl-nn:9000</value>
</property>
```

И конфиг файловой системы:
```
vim hdfs-site.xml
```
В configuration пишем следующее:
```
<property>
	<name>dfs.replication</name>
	<value>3</value>
</property>
```

Также нужно поправить список workers
```
vim workers
```

Там комментируем localhost, и вместо него указываем имена узлов, на которых будут запущены сервисы DataNode:
```
tmpl-nn
```
```
tmpl-dn-00
```
```
tmpl-dn-01
```

Это нужно распространить на все остальные узлы. 
Для этого копируем текущий путь:
```
pwd
```
(Например, у меня это /home/hadoop/hadoop-3.4.0/etc/hadoop)
И выполняем копирование:
```
scp hadoop-env.sh tmpl-nn:/home/hadoop/hadoop-3.4.0/etc/hadoop
```
```
scp hadoop-env.sh tmpl-dn-00:/home/hadoop/hadoop-3.4.0/etc/hadoop
```
```
scp hadoop-env.sh tmpl-dn-01:/home/hadoop/hadoop-3.4.0/etc/hadoop
```
```
scp core-site.xml tmpl-nn:/home/hadoop/hadoop-3.4.0/etc/hadoop
```
```
scp core-site.xml tmpl-dn-00:/home/hadoop/hadoop-3.4.0/etc/hadoop
```
```
scp core-site.xml tmpl-dn-01:/home/hadoop/hadoop-3.4.0/etc/hadoop
```
```
scp hdfs-site.xml tmpl-nn:/home/hadoop/hadoop-3.4.0/etc/hadoop
```
```
scp hdfs-site.xml tmpl-dn-00:/home/hadoop/hadoop-3.4.0/etc/hadoop
```
```
scp hdfs-site.xml tmpl-dn-01:/home/hadoop/hadoop-3.4.0/etc/hadoop
```
```
scp workers tmpl-nn:/home/hadoop/hadoop-3.4.0/etc/hadoop
```
```
scp workers tmpl-dn-00:/home/hadoop/hadoop-3.4.0/etc/hadoop
```
```
scp workers tmpl-dn-01:/home/hadoop/hadoop-3.4.0/etc/hadoop
```

На всех нодах нужно установить соответствующее имя хоста (tmpl-jn, tmpl-nn, tmpl-dn-00, tmpl-dn-01), запустив vim командой: 
```
sudo vim /etc/hostname 
```

Теперь все готово для запуска hadoop!
Подключимся к NameNode:
```
ssh tmpl-nn
```
Перейдем в следующую директорию:
```
cd hadoop-3.4.0/
```

Нужно создать и отформатировать файловую систему:
```
bin/hdfs namenode -format
```

И запускаем кластер распределенной файловой системы hadoop:
```
sbin/start-dfs.sh
```

Можно проверить с помощью jps, что все демоны запущены
```
jps
```

Должны видеть DataNode, Jps, SecondaryNameNode, NameNode

На DataNode (обеих) должны видеть DataNode и Jps:
```
ssh tmpl-dn-00
```
```
jps
```
```
ssh tmpl-dn-01
```
```
jps
```

Можно также проверить логи
