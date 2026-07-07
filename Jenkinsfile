// pipeline {
//     agent any
    
//     tools {
//         jdk 'jdk17'
//         maven 'maven3'
//     }
    
//     stages {   
//         stage('Compile') {
//             steps {
//             sh 'mvn compile'
//             }
//         }
        
//         stage('Test') {
//             steps {
//                 sh 'mvn test'
//             }
//         }
        
//         stage('Build') {
//             steps {
//                 sh 'mvn package'
//             }
//         }
//     }
// }

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
                    #SCANNER_HOME/BIN/sonar-scanner -Dsonar.projectName=Devops-end-to-end-poc -Dsonar.projectKey=Devops-end-to-end-poc \
                    -Dsonar.java.binaries=.
                    '''
                }
            }
        }
         stage('Quality Gate') {
            steps {
                script{
                    waitForQualityGate abortPipeline: false(), credentials: 'sonar-token'
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
               sh'''
                    withMaven(globalMavenSettingsConfig: 'global-setting', jdk: 'jdk17', maven: 'Maven3', traceability: true) {
                sh 'mvn deploy'
}
               '''
                
            }
        }
           stage('Build in docker') {
            steps {
                script{
                // This step should not normally be used in your script. Consult the inline help for details.
                withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                // some block
                 sh'''
                 docker buil -t nehal76/nehal76-devops-end-to-end-poc:latest .
                 '''

                    }
                }
                
            }
        }
        stage('Scan Docker File') {
            steps {
                sh '''
                trivy image --format table -o trivy-fs-report.html nehal76/nehal76-devops-end-to-end-poc:latest
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
                
                }
            }
        }
    }
}
