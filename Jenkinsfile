pipeline {
    agent any

    stages {
        stage('SCM') {
            steps {
                git(branch: 'develop', 
                url: "https://github.com/trisngo/WebGoat.git")
            }
        }
        stage('Build && Test'){
            steps {
                withMaven(maven:'3.6.2', jdk:'jdk17') {
                    sh 'mvn clean install'
                }
            }
        }
        stage ('OWASP Dependency-Check Vulnerabilities') {
            steps {
                dependencyCheck additionalArguments: ''' 
                    -o "./" 
                    -s "./"
                    -f "ALL" 
                    --prettyPrint''', odcInstallation: 'dependency-check'

                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
            }
        }           
        stage('SonarQube analysis') {
            environment {
                scannerHome = tool 'SonarQubeScanner'
            }
            steps {
                withSonarQubeEnv('sonarqube') {
                    // Optionally use a Maven environment you've configured already
                    withMaven(maven:'3.6.2', jdk:'jdk17') {
                        sh 'mvn sonar:sonar -Dsonar.dependencyCheck.jsonReportPath=target/dependency-check-report.json -Dsonar.dependencyCheck.xmlReportPath=target/dependency-check-report.xml -Dsonar.dependencyCheck.htmlReportPath=target/dependency-check-report.html'
                    }
                }
            }
            
        }
        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        // stage('cleanup') {
        //     steps {
        //         sh 'docker system prune -a --volumes --force'
        //   }
        // }
        stage('build container image') { 
            steps {
        		sh '''
        			cd docker/
                    docker build --no-cache --build-arg webgoat_version=8.2.0-SNAPSHOT -t milestone/webgoat:latest .
        		'''
            }
        }
        stage("docker_scan"){
            steps {
                // docker run -d --name db arminc/clair-db
                // sleep 15
                // docker run -p 6060:6060 --link db:postgres -d --name clair arminc/clair-local-scan
                // sleep 1
                // wget -qO clair-scanner https://github.com/arminc/clair-scanner/releases/download/v8/clair-scanner_linux_amd64 && chmod +x clair-scanner
                sh '''
                    DOCKER_GATEWAY=$(docker network inspect bridge --format "{{range .IPAM.Config}}{{.Gateway}}{{end}}")
                    wget -qO clair-scanner https://github.com/arminc/clair-scanner/releases/download/v8/clair-scanner_linux_amd64 && chmod +x clair-scanner
                    ./clair-scanner --clair=172.17.0.1 --ip="$DOCKER_GATEWAY" milestone/webgoat:latest || exit 0
                '''
            }
        }
        stage('run container') {
            steps {
                sh '''
        			cd docker/
                    docker run -d --name webgoat -p 0.0.0.0:80:8888 -p 0.0.0.0:8080:8080 -p 0.0.0.0:9090:9090 -e TZ=Europe/Amsterdam milestone/webgoat:latest
        		'''
          }
        }
    }
}
