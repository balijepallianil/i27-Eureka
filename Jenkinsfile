pipeline {
    agent {
        label 'k8s-slave'
    }

    tools{
        maven 'Maven-3.8.8'
        jdk 'JDK-17'
    }

    environment {
        APPLICATION_NAME = "eureka"
        POM_VERSION = readMavenPom().getVersion()
        POM_PACKAGING = readMavenPom().getPackaging()
        SONAR_URL = "http://34.46.21.82:9000"
        SONAR_TOKEN = credentials('sonar_creds')
        DOCKER_HUB = "docker.io/i27anilb3"
        DOCKER_CREDS = credentials('docker_creds')
        MAHA_CREDS = credentials('maha_creds')
        // DOCKER_HOST_IP = "10.1.0.9"
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
            //sqa_a25af99d06b87a263ccf7aed9033cd9d80b97b36
            steps {
                sh """
                echo "Starting Sonar Scan"
                mvn sonar:sonar \
                    -Dsonar.projectKey=i27-eureka \
                    -Dsonar.host.url=${env.SONAR_URL} \
                    -Dsonar.login=${SONAR_TOKEN} 
                """
            }
        }

        stage ('Docker Build and push') {
            steps {
                  echo "Starting Docker build stage"
                sh """
                ls -la
                pwd
                cp ${WORKSPACE}/target/i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} ./.cicd/
                echo "Listing Files in .cicd folder"
                ls -la ./.cicd/
                echo "**************************** Building Docker Image ****************************"
                docker build --force-rm --no-cache --build-arg JAR_SOURCE=i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} -t ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT} ./.cicd
                docker images
                echo "********Docker login******"
                docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}
                echo "********Docker Push******"
                docker push ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}
                """
            }
        }

        stage ('Deploy To Dev') {
            steps {
                echo "*******Deploy to DEV********" 
                script {
                sh('sshpass -p $MAHA_CREDS_PSW ssh -o StrictHostKeyChecking=no $MAHA_CREDS_USR@$DOCKER_DEPLOY_HOST_IP docker pull $DOCKER_HUB/$APPLICATION_NAME:$GIT_COMMIT')
                try {
                    sh('sshpass -p $MAHA_CREDS_PSW ssh -o StrictHostKeyChecking=no $MAHA_CREDS_USR@$DOCKER_DEPLOY_HOST_IP docker stop $APPLICATION_NAME') 
                    sh('sshpass -p $MAHA_CREDS_PSW ssh -o StrictHostKeyChecking=no $MAHA_CREDS_USR@$DOCKER_DEPLOY_HOST_IP docker rm $APPLICATION_NAME')
                 } catch(err) {
                    echo "caught Error : $err"
                 }
                 sh('sshpass -p $MAHA_CREDS_PSW ssh -o StrictHostKeyChecking=no $MAHA_CREDS_USR@$DOCKER_DEPLOY_HOST_IP docker run -d -p 5671:8761 --name $APPLICATION_NAME -dev $DOCKER_HUB/$APPLICATION_NAME:$GIT_COMMIT')
                }
                
            }

        }
        stage ('Deploy To Test') {
            steps {
                echo "*******Deploy to TEST********" 
                script {
                sh('sshpass -p $MAHA_CREDS_PSW ssh -o StrictHostKeyChecking=no $MAHA_CREDS_USR@$DOCKER_DEPLOY_HOST_IP docker pull $DOCKER_HUB/$APPLICATION_NAME:$GIT_COMMIT')
                try {
                    sh('sshpass -p $MAHA_CREDS_PSW ssh -o StrictHostKeyChecking=no $MAHA_CREDS_USR@$DOCKER_DEPLOY_HOST_IP docker stop $APPLICATION_NAME-test') 
                    sh('sshpass -p $MAHA_CREDS_PSW ssh -o StrictHostKeyChecking=no $MAHA_CREDS_USR@$DOCKER_DEPLOY_HOST_IP docker rm $APPLICATION_NAME-test')
                 } catch(err) {
                    echo "caught Error : $err"
                 }
                 sh('sshpass -p $MAHA_CREDS_PSW ssh -o StrictHostKeyChecking=no $MAHA_CREDS_USR@$DOCKER_DEPLOY_HOST_IP docker run -d -p 6671:8761 --name $APPLICATION_NAME-test $DOCKER_HUB/$APPLICATION_NAME:$GIT_COMMIT')
                }
                
            }     
        }   

    }
    
}