pipeline {
    agent any
    tools {
        jdk 'JDK11'
        maven 'Maven3'
    }
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }
    stages {
        stage('Git CheckOut') {
            steps {
                git changelog: false, poll: false, url: 'https://github.com/moeltelawi/secretsanta-generator.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }
        
        stage('SonerQube Check') {
            steps {
                sh '''
                $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Santa \
                -Dsonar.java.binaries=. \
                -Dsonar.host.url=http://127.0.0.1:7778/ \
                -Dsonar.login=squ_0618b52c171384ef9fc22916edcb55a09b729dca \
                -Dsonar.projectName=Santa \
                -Dsonar.projectKey=Santa
                '''
            }
        }
        
        stage('OWASP check') {
            steps {
                dependencyCheck additionalArguments: ' --scan ./ ', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Build App JAR Package') {
            steps {
                sh 'mvn clean install'
            }
        }
                stage('Build app Image and push it to Docker Hub') {
            steps {
                    script {
                    docker.withRegistry('https://index.docker.io/v1/', 'Docker-Hub') {
                        def image = docker.build("eltelawi/santa:latest")
                        image.push()
                    }
                }
                // This step should not normally be used in your script. Consult the inline help for details.
//                withDockerRegistry(credentialsId: 'Docker-Hub', toolName: 'docker') {
                    // some block
//                    sh 'docker build -t santa .'
//                    sh 'docker tag santa eltelawi/santa:latest'
//                    sh 'docker push'
//                }
            }
        }
stage('Trivy Scan') {
    steps {
        sh """
        mkdir -p trivy-report
        trivy image --timeout 15m --format table --output trivy-report/report.txt eltelawi/santa:latest
        """
    }
}

        stage('Deploy Image') {
            steps {
                script {
                    sh '''
                    docker rm -f santa-app || true
                    docker run -d --name santa-app -p 8080:8080 eltelawi/santa:latest
                    '''
                }
            }
        }
    }
}
