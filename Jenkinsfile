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
            string(name: 'release_version', description: 'Code Release Version (SNAPSHOTS NOT ALLOWED)')
            string(name: 'next_dev_version', description: 'Next Version for Development Version(ONLY SNAPSHOTS)')
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
        stage('Increment Version') {
               steps {
                    configFileProvider([configFile(fileId: "maven-settings-file", variable: 'MAVEN_SETTINGS')]) {
                        sh "mvn -B versions:set -DnewVersion=$params.release_version -s $MAVEN_SETTINGS"
                    }
                }
        }
        stage('Commit, Push and Tag Incremented Version') {
               steps {
                    configFileProvider([configFile(fileId: "maven-settings-file", variable: 'MAVEN_SETTINGS')]) {
                         withCredentials([usernamePassword(credentialsId: 'GitCredentials', passwordVariable: 'paswd', usernameVariable: 'username')]) {
                            sh "git remote add git http://${username}:${paswd}@github.com/shred22/spring-boot-oai3.git"
                            sh "git commit -am 'Comitting Incremented Versions from Jenkins $params.release_version'"
                            sh "git tag -a ${parmas.release_version} -m 'Tagged Version from Jenkins $params.release_version'"
                            sh "git push ${params.branch_name}"
                            sh "git push ${params.release_version}"
                        } 
                    }
                }
        }
                      
          
        stage('Deploy Incremented Versions to Nexus') {
               steps {
                    configFileProvider([configFile(fileId: "maven-settings-file", variable: 'MAVEN_SETTINGS')]) {
                        sh "mvn -B deploy -DskipTests -s $MAVEN_SETTINGS"
                    }
                      
                }
        }
    }
}
