title: install jdk from tar
date: 2016-08-09 03:02:12
tags:
---


place jdk1.8.0_101 to /opt

modify .bashrc:

``` bash
#set oracle jdk environment
export JAVA_HOME=/opt/jdk1.8.0_101
export JRE_HOME=${JAVA_HOME}/jre  
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib  
export PATH=${JAVA_HOME}/bin:$PATH

```

execute:

``` bash
source .bashrc

sudo update-alternatives --install /usr/bin/java java /opt/jdk1.8.0_101/bin/java 300  
sudo update-alternatives --install /usr/bin/javac javac /opt/jdk1.8.0_101/bin/javac 300  
sudo update-alternatives --install /usr/bin/jar jar /opt/jdk1.8.0_101/bin/jar 300   
sudo update-alternatives --install /usr/bin/javah javah /opt/jdk1.8.0_101/bin/javah 300   
sudo update-alternatives --install /usr/bin/javap javap /opt/jdk1.8.0_101/bin/javap 300
sudo update-alternatives --config java

```

verify java installation:

```
java -version
```
