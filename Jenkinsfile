pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonar'
        NVD_API_KEY = credentials('NVD_API_KEY')      }

    tools {
        maven 'mvn3'
        jdk 'jdk21'
    }

    stages {
        stage('git checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/AmulThantharate/Ekart1'
            }
        }

        stage('compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('unit tests') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }

        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh "${env.SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=EKART \
                        -Dsonar.projectName=EKART \
                        -Dsonar.java.binaries=target/classes"
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('Trivy File System Scan') {
            steps {
                sh 'trivy fs --format table -o trivy-fs-report.html .'
            }
        }

        stage('OSV Dependency Scan') {
            steps {
                sh 'docker run --rm -v "$PWD":/src -w /src ghcr.io/google/osv-scanner:v1 --recursive .'

            }
        }

        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }

        stage('deploy to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk21', maven: 'mvn3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }

        stage('build and Tag docker image') {
            steps {
                script {
                    sh "docker build -t claw4321/ekart:v2 ."
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html claw4321/ekart:v2"
            }
        }

        // stage('Push image to Hub'){
        //     steps{
        //         script{
        //             withCredentials([string(credentialsId: 'DOCKER', variable: 'dockerhubpwd')]) {
        //                 sh 'docker login -u claw4321 -p ${dockerhubpwd}'
        //             }
        //             sh 'docker push claw4321/ekart:v2'
        //         }
        //     }
        // }
        // stage('EKS and Kubectl configuration'){
        //     steps{
        //         script{
        //             sh 'aws eks update-kubeconfig --region ap-south-1 --name project-cluster'
        //         }
        //     }
        // }
        // stage('Deploy to k8s'){
        //     steps{
        //         script{
        //             sh 'kubectl apply -f deploymentservice.yml'
        //         }
        //     }
        // }
    }

}
