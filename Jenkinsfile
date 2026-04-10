pipeline{
    agent{
        label "docker-maven-trivy"
    }
    tools {
        maven 'maven3'
    }
    environment {
        SONAR_IP = '172.31.9.78'
    }
    stages{
        stage("Trivy FS Scan"){
            steps{
                sh 'trivy fs --exit-code 1 --severity HIGH,CRITICAL .'
            }
        }
        stage ("Build and Scan") {
            steps {
                withCredentials([string(credentialsId: 'SONAR-TOKEN', variable: 'SONAR-TOKEN')]) 
                {
                  sh 'mvn clean verify sonar:sonar \
                -Dsonar.projectKey=devsecops-project \
                -Dsonar.host.url="http://${SONAR_IP}:9000" \
                -Dsonar.token="${SONAR-TOKEN}" \
                -Dsonar.qualitygate.wait=true'
                }
            }
        }
    }
    }