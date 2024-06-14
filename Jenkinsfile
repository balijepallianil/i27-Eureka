pipeline {
    agent {
        label 'k8s-slave'
    }

    tools{
        maven 'Maven-3.8.8'
        jdk 'JDK-17'
    }

    environment {
        APPLICATION_NAME = "Eureka"
        SONAR_URL = "http://34.46.21.82:9000"
        SONAR_TOKEN = credentials('sonar_creds')
    }

    stages{
       stage ('Build') {
            steps {
                echo "Bulding ${env.APPLICATION_NAME} application"
                sh 'mvn clean package -DskipTests=true'
                archiveArtifacts artifacts: 'target/*jar'
            }
        }
//        stage('Unit Test') {
//      steps {
//          echo "Executing UNIX test cases for ${env.APPLICATION_NAME} application"
//          sh 'mvn test'
//      }
//      post {
//          always {
//              junit 'target/surefire-reports/*xml'
//              jacoco execPattern: 'target/jacoco.exec'
//          }
//      }
// }
        stage ('Sonar') {
            steps {
                sh """
                echo "starting Sonar"
                mvn sonar:sonar \
                    -Dsonar.projectKey=i27-eureka \
                    -Dsonar.host.url=${env.SONAR_URL} \
                    -Dsonar.login=${SONAR_TOKEN}
                    """
            }
        }

    }
}
        

