pipeline {
    agent any
    options { 
    	buildDiscarder(logRotator(numToKeepStr: '5'))
    	skipDefaultCheckout(true)

     }

     environment {
     	git_Credentials = credentials('GitCredentials')
     }

    parameters {
            string(name: 'branch_name', defaultValue: 'master', description: 'The branch from which Code is to be released')
    }
    /* tools {
            maven 'apache-maven-3.0.1'
    } */
    stages {

        stage('Checkout Branch for Release') {
                steps {
                        echo "The Credentials  I got are : $git_Credentials and pipeline param is : $params.branch_name other way ${params.branch_name}"
                    	deleteDir()
                     	git branch: "$params.branch_name", credentialsId: 'GitCredentials', url: 'https://github.com/shred22/spring-boot-oai3.git'
                }               
        }
        
        stage('Build Code') {
               steps {
                    configFileProvider([configFile(fileId: "maven-settings-file", variable: 'MAVEN_SETTINGS')]) {
                        sh "mvn clean install -DskipTests -s $MAVEN_SETTINGS"
                    }
                      
               }
        }
        stage('Release Code') {
               steps {
                    configFileProvider([configFile(fileId: "maven-settings-file", variable: 'MAVEN_SETTINGS')]) {
                        withCredentials([usernamePassword(credentialsId: 'GitCredentials', passwordVariable: 'paswd', usernameVariable: 'username')]) {
                            sh "git remote add git http://${username}:${paswd}@github.com/shred22/spring-boot-oai3.git"
                            sh """mvn -B release:clean release:prepare release:perform -Dusername=${username} -Dpassword=${paswd}
                              -Dtag=reg-service -DreleaseVersion=1.2 -DdevelopmentVersion=2.0-SNAPSHOT
                              -s $MAVEN_SETTINGS"""
                        }   
                        
                    }
                      
               }
        }
    }
}
