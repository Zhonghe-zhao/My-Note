

[CS61B](https://csdiy.wiki/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/CS61B/#_1)

[答案](https://github.com/InsideEmpire/CS61B-PathwayToSuccess)

[课程](https://sp24.datastructur.es/)


## 配置在wsl中java环境

`sudo apt install openjdk-11-jdk`

`sudo nano /etc/environment`

add: 
~~~
JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
PATH=$PATH:$JAVA_HOME/bin
~~~

~~~
source /etc/environment
~~~

编译：
`javac HelloWorld.java
`
执行：

`java HelloWorld
`