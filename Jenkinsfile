pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    
    stages {   
        stage('Compile') {
            steps {
            sh 'mvn compile'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }
    }
}




pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }
    
    stages {   
        stage('git checkout') {
            steps {
            git branch: 'main', credentialsId: 'git', url: 'https://github.com/nehal76/nehal76-Devops-end-to-end-poc.git'
            }
        }
        
        stage('compile') {
            steps {
                sh 'mvn compile'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('File-system-scan') {
            steps {
                sh ' trivy fs --format table -o trivy-fs-report.html .'
            }
        }
        stage('Sonar_Qube Analysis') {
            steps {
                withSonarQubeEnv('sonar'){
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Devops-end-to-end-poc -Dsonar.projectKey=Devops-end-to-end-poc \
                    -Dsonar.java.binaries=.
                    '''
                }
            }
        }
         stage('Quality Gate') {
            steps {
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
                
            }
        }
           stage('Build') {
            steps {
                sh ' mvn package'
            }
        }

           stage('publish artifects to nexus') {
            steps {
               
                    withMaven(globalMavenSettingsConfig: 'global-setting', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh 'mvn deploy'
            }
            
                
            }
        }
           stage('Build in docker') {
            steps {
                script{
                
                    withDockerRegistry([credentialsId: 'docker-cred']) {
                // some block
                 sh'''
                    docker build -t nehal76/nehal76-devops-end-to-end-poc:latest .
                 '''

                    }
                }
                
            }
        }
        stage('Scan Docker File') {
            steps {
                sh '''
                trivy image --format table -o trivy-image-report.html nehal76/nehal76-devops-end-to-end-poc:latest
                '''
            }
        }
        stage('Docker image into Docker hub') {
            steps {
               withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                // some block
                 sh'''
                    docker push nehal76/nehal76-devops-end-to-end-poc:latest
                 '''

                    }
            }
        }

        stage('Deploy to K8') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.16.247:6443') {
                // some block
                sh '''
                kubectl apply -f deployment-service.yaml
                '''

                }
            }
        }
        stage('verify deployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.16.247:6443') {
                // some block
                sh '''
                kubectl get pods -n webapps
                kubectl get svc -n webapps
                '''

                }
            }
        }
        
    }
    post {
    always {
        script {
            def jobName = env.JOB_NAME
            def buildNumber = env.BUILD_NUMBER
            def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
            def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'

            def body = """
                <html>
                <body>
                <div style="border: 4px solid ${bannerColor}; padding: 10px;">
                <h2>${jobName} - Build ${buildNumber}</h2>
                <div style="background-color: ${bannerColor}; padding: 10px;">
                <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                </div>
                <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                </div>
                </body>
                </html>
            """

            emailext (
                subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                body: body,
                to: 'nehal.spn786@gmail.com',
                from: 'nehal.spn786@gmail.com',
                replyTo: 'nehal.spn786@gmail.com',
                mimeType: 'text/html',
                attachmentsPattern: 'trivy-image-report.html'
            )
        }
    }
}

}
