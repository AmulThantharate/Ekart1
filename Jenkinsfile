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
                sh '''
                    docker run --rm \
                    -v "$PWD:/src" \
                    -w /src \
                    ghcr.io/google/osv-scanner:v2 \
                    scan source -r . || true
                '''
            }
        }

        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }

        stage('deploy to Nexus') {
            steps {
                configFileProvider([configFile(fileId: 'e0399204-16ad-45f0-82d5-54edab2afb7a', variable: 'MAVEN_SETTINGS')]) {
                    sh "mvn deploy -s $MAVEN_SETTINGS -DskipTests=true"
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

        stage('Push image to Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'DOCKER_CRED', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh 'docker push claw4321/ekart:v2'
                    }
                }
            }
        }
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
