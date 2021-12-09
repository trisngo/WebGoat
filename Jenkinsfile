pipeline {
    agent any

    stages {
        stage('Build') { 
            steps {
		sh '''
			java --version
			mvn --version
			mvn clean install
		'''
            }
        }
    }
}
