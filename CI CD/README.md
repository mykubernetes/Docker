解决jenkins自动关闭衍生的子进程问题
```
withEnv(['JENKINS_NODE_COOKIE=dontkillme']) {
    sh """
         ${tomcatHome}/bin/startup.sh
    """
}
```

PHP项目pipeline
```
node ("slave1-192.168.0.215") {
   stage('git checkout') { 
       checkout([$class: 'GitSCM', branches: [[name: '${branch}']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'git@192.168.0.216:/home/git/repos/wordpress']]])
   }
   stage('code copy') {
        sh '''rm -rf ${WORKSPACE}/.git
        mv /usr/share/nginx/html/wp.aliangedu.com /data/backup/wp.aliangedu.com-$(date +"%F_%T")
        cp -rf ${WORKSPACE} /usr/share/nginx/html/wp.aliangedu.com'''
   }
   stage('test') {
       sh "curl http://127.0.0.1/status.html"
   }
}

```  

JAVA项目pipeline
```
node ("192.168.0.216") {
   //def mvnHome = '/usr/local/maven'
   stage('git checkout') { 
        checkout([$class: 'GitSCM', branches: [[name: '${branch}']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'git@192.168.0.216:/home/git/repos/solo']]])
   }
   stage('maven build') {
        sh '''export JAVA_HOME=/usr/local/jdk1.8
        /usr/local/maven3.3/bin/mvn clean package -Dmaven.test.skip=true'''
   }
   stage('deploy') {
        sh '''export JAVA_HOME=/usr/local/jdk1.8
        JENKINS_NODE_COOKIE=dontkillme                #jenkins自动关闭衍生的子进程
        TOMCAT_NAME=tomcat
        TOMCAT_HOME=/usr/local/$TOMCAT_NAME
        WWWROOT=$TOMCAT_HOME/webapps/ROOT

        if [ -d $WWWROOT ]; then
           mv $WWWROOT /data/backup/${TOMCAT_NAME}-$(date +"%F_%T")
        fi
        unzip ${WORKSPACE}/target/*.war -d $WWWROOT
        PID=$(ps -ef |grep $TOMCAT_NAME |egrep -v "grep|$$" |awk \'{print $2}\')
        [ -n "$PID" ] && kill -9 $PID
        /bin/bash $TOMCAT_HOME/bin/startup.sh'''
   }
   stage('test') {
       sh "curl http://127.0.0.1/status.html"
   }
}
```  

