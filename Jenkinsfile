pipeline {
    
	agent any
	
	tools {
        maven "maven3"
    }
	
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "34.232.105.51:8081"
        NEXUS_IP = "34.232.105.51"
        NEXUS_PORT = "8081"
        NEXUS_USER = "admin"
        NEXUS_PASS = "Password"
        NEXUS_REPOSITORY = "Demo"
	    NEXUS_REPO_ID    = "Demo"
        NEXUS_LOGIN = "nexuslogin"
        ARTVERSION = "${env.BUILD_ID}"
    }
	
    stages{
        
        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

	stage('UNIT TEST'){
            steps {
                sh 'mvn test'
            }
        }

	stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }
		
        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        

        stage("Publish to Nexus Repository Manager") {
           steps{
            nexusArtifactUploader(
                nexusVersion: 'nexus3',
                protocol: 'http',
                nexusURl: "${NEXUSIP}:${NEXUSPORT}",
                groupId: 'com.visualpathit',
                version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                repository: "${RELEASE_REPO}",
                credentialsId: "${NEXUS_LOGIN}",
                artifacts: [
                    [artifactId: 'vprofile',
                     classifier: '',
                     file: 'target/vprofile-v2.war',
                     type: 'war']
                ]
            )
           }
        }
        stage('Ansible Deploy to staging'){
            steps{
                ansiblePlaybook([
                inventory       : 'ansible/stage.inventory',
                playbook        : 'ansible/site.yml',
                installation     : 'ansible',
                colorized        : true,
                credentialsId    : 'applogin',
                disableHostKeyChecking: true,
                extravars         : [
                    USER: "admin",
                    PASS: "Password",
                    nexusip: "34.232.105.51",
                    reponame: "Demo",
                    groupid : "QA",
                    time    : "${env.BUILD_TIMESTAMP}",
                    build: "${env.BUILD_ID}",
                    artifactid: "Demo",
                    Demo: "Demo-${env.BUILD_ID}-${env.BUILD_TIMESTAMP}.war"


                ]
             ])
            }
        }


    }


}