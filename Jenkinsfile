pipeline {
    agent any
    stages {
        stage ( 'Build Backend'){
            steps {
                bat 'mvn clean package -DskipTests=true'
            }
        }
         stage ( 'Unit Tests'){
            steps {
                bat 'mvn test'
            }
        }
        stage ( 'Sonar Analysis'){
            environment {
                scannerHome = tool 'SONNAR_SCANNER'
            }
            steps {
                withSonarQubeEnv('SONAR_LOCAL'){
                  bat "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=f61ee5fc20db85cd4df2fee1e5bd248076ac58d7 -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/mvn/**,**/src/test/**,**/model/**,**Application.java"
                }
            }
        }
        stage ('Quality Gate') {
            steps {
                sleep(30)
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage ('Deploy Backend'){
            steps {
                deploy adapters: [tomcat9(credentialsId: 'Tomcatlogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        } 
        stage ('API Test'){
            steps {
                dir('api-test'){
                    git branch: 'main', credentialsId: 'GitHub', url: 'https://github.com/EliedersonLinhares/APITest'
                    bat 'mvn test'
                }    
            }
        } 
        stage ('Deploy Frontend'){
            steps {
                dir('frontend'){
                    git branch: 'master', credentialsId: 'GitHub', url: 'https://github.com/EliedersonLinhares/tasks-frontend'
                    bat 'mvn clean package'
                    deploy adapters: [tomcat9(credentialsId: 'Tomcatlogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
                }
            }
        }
        stage ('Functional Test'){
            steps {
                dir('functional-test'){
                    git branch: 'main', credentialsId: 'GitHub', url: 'https://github.com/EliedersonLinhares/APITest-Selenium'
                    bat 'mvn test'
                }    
            }
        } 
        stage ('Deploy Prod'){
            steps {
                bat 'docker-compose build'
                bat 'docker-compose up -d'
            }
        }
        stage ('Health check'){
            steps {
                sleep(30)
                dir('functional-test'){
                    bat 'mvn verify -Dskip.surefire.tests'
                }    
            }
        } 
    } 
    post {
        always{
           junit allowEmptyResults: true, testResults: 'target/surfire-reports/*.xml, api-test/target/surfire-reports/*.xml, functional-test/target/surfire-reports/*.xml, functional-test/target/failsafe-reports/*.xml'
        }
        unsuccessful {
            emailext attachLog: true, body: 'See the attached log below', subject: 'Build $BUILD_NUMBER has failed', to: 'eliederson250+jenkins@gmail.com'
        }
        fixed {
            emailext attachLog: true, body: 'See the attached log below', subject: 'Build is fine!!!', to: 'eliederson250+jenkins@gmail.com'
        }  
    } 
}

