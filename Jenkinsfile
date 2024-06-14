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
        POM_VERSION = readMavenPom().getVersion()
        POM_PACKAGING = readMavenPom().getPackaging()
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
                // Code Quality needs to be implemented 
                echo "Starting Sonar Scans with Quality Gates"
                // before we go to next step, install sonarqube plugin 
                // next goto manage jenkins > configure > sonarqube > give url and token for sonarqube
                withSonarQubeEnv('SonarQube'){ // SonarQube is the name that we configured in manage jenkins > sonarqube 
                    sh """
                        mvn sonar:sonar \
                            -Dsonar.projectKey=i27-eureka \
                            -Dsonar.host.url=${env.SONAR_URL} \
                            -Dsonar.login=${SONAR_TOKEN} 
                        """
                }
                timeout (time: 2, unit: 'MINUTES') { // NANOSECONDS, SECONDS, MINUTES, HOURS, DAYS
                    script {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }

        stage ('Docker Format') {
            steps {
                //the this current format
                echo "the actual format is i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING}"
                //custom format
                echo "The Custom Format is ${env.APPLICATION_NAME}-${currentBuild.number}-${BRANCH_NAME}.${env.POM_PACKAGING}"
            }
        }

    }
}
    
