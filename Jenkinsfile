
pipeline{
    agent any
    tools{
        jdk 'jdk'
        maven 'maven'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/Shubh1am/Task-Master-Pro.git'
            }
        }
        stage("Compile"){
            steps{
                sh "mvn clean compile"
            }
        }
        
         stage("Test Cases"){
            steps{
                sh "mvn test"
            }
        }
        stage('File System Scan') {
            steps {
  //              sh "trivy fs --format table -o trivy-fs-report.html ."
                  sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Sonarqube Analysis"){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix \
                    '''
                }
            }
        }
        stage("Quality Gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            } 
        }
        stage("OWASP Dependency Check"){
            steps{
               sh' mvn org.owasp:dependency-check-maven:6.0.0:check'

                dependencyCheck additionalArguments: '--scan ./ --format HTML ', odcInstallation: 'DP'
               // dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
//        stage('Deploy to Nexus') {
//            steps {
//             withNexus(nexusUrl: 'http://65.2.150.34:32000/', credentialsId: 'nexus-cred') {
//                sh '''mvn deploy:deploy-file \
//                    -Dfile=target/todo-app-1.0-SNAPSHOT.jar \
//                    -Durl=http://65.2.150.34:32000/repository/maven-releases/ \
//                    -DrepositoryId=maven-releases \
//                    -DgroupId=http://65.2.150.34:32000/repository/nuget-group/ \
//                    -DartifactId=todo-app \
//                    -Dversion=${env.BUILD_NUMBER}'''
//                }
//            }
//        }
        
//        stage('Publish To Nexus') {
//            steps {
//               withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk', maven: 'maven', mavenSettingsConfig: '', traceability: true) {
//                    sh "mvn deploy"
//                }
//            }
//        }
        
         stage("Build"){
            steps{
                sh " mvn clean install"
            }
        }
        
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        
                        sh "docker build -t image1 ."
                        sh "docker tag image1 shubhammutkalwar/taskmaster:latest "
                        sh "docker push shubhammutkalwar/taskmaster:latest "
                    }
                }
            }
        }
        
        stage("TRIVY"){
            steps{
                sh " trivy image shubhammutkalwar/taskmaster:latest > trivyimage.txt "
            }
        }
    }
    post {
     always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
                "Build Number: ${env.BUILD_NUMBER}<br/>" +
                "URL: ${env.BUILD_URL}<br/>",
            to: 'shubhammutkalwar@gmail.com',
            attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
        }
    }
}
