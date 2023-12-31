pipeline {
    
	agent any
	
	tools {
        maven "maven3"
    }
	
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "44.199.217.199:8081"
        NEXUS_REPOSITORY = "Maven-Demo"
	NEXUS_REPO_ID    = "Maven-Demo"
        NEXUS_CREDENTIAL_ID = credentials('nexuslogin')
	NEXUSIP   = "44.199.217.199"
	NEXUSPORT = "8081"
        NEXUS_USER = "admin"
	NEXUS_PASS = "Password"
        ARTVERSION = "${env.BUILD_ID}"
        RELEASE_REPO = "Maven-Demo"
    }
	
    stages{
        
        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.jar'
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
                nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                groupId: 'com.mt',
                version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                repository: "${RELEASE_REPO}",
                credentialsId: "${NEXUS_CREDENTIAL_ID}",
                artifacts: [
                    [artifactId: 'spring-boot-mongo',
                     classifier: '',
                     file: 'target/spring-boot-mongo.jar',
                     type: 'jar']
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
                extraVars         : [
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
