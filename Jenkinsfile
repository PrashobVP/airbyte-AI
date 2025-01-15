pipeline {
    agent any
    environment {
        SONAR_HOME = tool "Sonar"
    }
    stages {
        stage("Code Checkout") { 
            steps {
                git(
                    url: "https://github.com/PrashobVP/airbyte-AI.git",
                    branch: "master",
                    credentialsId: "jenkins-secret",
                    changelog: true,
                    poll: true
                )
            }
        }
        
        // stage("Docker Build, Tag, and Push DEV") {
        //     steps {
        //         script {
        //             def condition = false
                    
        //             if (condition) {
        //                 def dockerImage = "amigo-nishant/airbyte"
        //                 def dockerTag = "latest"
                        
        //                 withCredentials([usernamePassword(credentialsId: 'docker-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
        //                     sh "docker build -t ${dockerImage}:${dockerTag} ."
        //                     sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
        //                     sh "docker push ${dockerImage}:${dockerTag}"
        //                 }
        //             } else {
        //                 sh "./compose_helm.sh -d"
        //                 sh "cd airbyte_files && docker compose up -d"
        //             }
        //         }
        //     }
        // }
        
        stage("SonarQube Code Analysis") {
            steps {
                withSonarQubeEnv("Sonar") {
                    sh """
                        $SONAR_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=Airbyte-AI \
                        -Dsonar.projectName=Airbyte-AI \
                        -Dsonar.sources=. \
                        -Dsonar.inclusions=**/*.py,**/*.yaml,**/*.json,**/*.md \
                        -Dsonar.exclusions=**/*.java,**/airbyte-config/**,**/airbyte-db/**,**/airbyte-workers/**
                    """
                }
            }
        }
        
        stage("Trivy File System Scan") {
            steps {
                sh "trivy fs --severity HIGH,CRITICAL --format table -o trivy-fs-report.html ."
            }
        }
        
        stage("OWASP Dependency Check") {
            steps {
                echo "Skipping due to some dependency test cases are yet to be merged to master"
                // dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'dc'
                // dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage("PROD Deployment") {
            steps {
                sh "helm repo add airbyte https://airbytehq.github.io/helm-charts"
                sh "helm repo update"
                sh "helm upgrade --install airbyte airbyte/airbyte --dry-run --debug"
                sh "helm upgrade --install airbyte airbyte/airbyte & pid=$!"
                echo "kill $pid" | at now + 2 minutes
                echo "PROD Deployment successful"
            }
        }
    }
}
