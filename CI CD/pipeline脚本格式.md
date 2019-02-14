pipeline脚本格式
===============
```
#!groovy
pipeline {
       agent {node {lable 'node01'}}
       environment {
            PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin"
        }
       parameters {
            choice(
                choices: 'dev\nprod',
                description: 'choos deploy environment',
                name: 'deploy_env'
             )
             string (name: 'version', defaultValue: '1.0.0', description 'build version')
        }
       stages {
            stage("Checkout test repo") {
                steps{
                      sh 'git config --global http.sslVerify false'
                      dir ("${env.WORKSPACE}") {
                          git branch: 'master', credentialsID:"9aa11671-aab9-47c7-a5e1-a4be46bf587", url: 'https://root@gitlab.example.com/root/test-repo.git'
                      }
                 }
            }
            stage("Print env variable") {
               steps {
                  dir ("${env.WORKSPACE}") {
                      sh """
                      echo "[INFO] PRint env variable"
                      echo "Current deployment environment is $deploy_env >> test.properties
                      echo "The build is $version" >> test.proerties
                      echo "[INFO] Done..."
                      """
                  }
               }
            }
            stage("Check test properties") {
                steps{
                    dir ("${env.WORKSPACE}") {
                        sh """
                        echo "[INFO] Check test properties"
                        if [ -s test.properies ]
                        then
                            cat test.properties
                            echo "[INFO] Done..."
                        else
                            echo "test.properties is empty"
                        fi
                        """
                }
            }
```
