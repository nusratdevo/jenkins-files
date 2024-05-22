# Final Pipeline Script for cicd docker image create & Image pull for update menifest fife on github
pipeline {
	agent any
	tools {
    	jdk 'jdk'
    	nodejs 'nodejs'
	}
	environment  {
    	SCANNER_HOME=tool 'sonar-scanner'
	}
	stages {
    	stage('Cleaning Workspace') {
        	steps {
            	cleanWs()
        	}
    	}
    	stage('Checkout from Git') {
        	steps {
            	git branch: 'master', url: 'https://github.com/AmanPathak-DevOps/End-to-End-Kubernetes-DevSecOps-Tetris-Project.git'
        	}
    	}
    	stage('Sonarqube Analysis') {
        	steps {
            	dir('Tetris-V2') {
                	withSonarQubeEnv('sonar-server') {
                    	sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=TetrisVersion1.0 \
                    	-Dsonar.projectKey=TetrisVersion1.0 '''
                	}
            	}
        	}
    	}
    	stage('Quality Check') {
        	steps {
            	script {
                	waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
            	}
        	}
    	}
    	stage('Installing Dependencies') {
        	steps {
            	dir('Tetris-V2') {
                	sh 'npm install'
            	}
        	}
    	}
    	stage('OWASP Dependency-Check Scan') {
        	steps {
            	dir('Tetris-V2') {
                	dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                	dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            	}
        	}
    	}
    	stage('Trivy File Scan') {
        	steps {
            	dir('Tetris-V2') {
                	sh 'trivy fs . > trivyfs.txt'
            	}
        	}
    	}
    	stage("Docker Image Build") {
        	steps {
            	script {
                	dir('Tetris-V2') {
                    	withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {   
                        	sh 'docker system prune -f'
                        	sh 'docker container prune -f'
                        	sh 'docker build -t tetrisv2 .'
                    	}
                	}
            	}
        	}
    	}
    	stage("Docker Image Pushing") {
        	steps {
            	script {
                	withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {   
                    	sh 'docker tag tetrisv2 avian19/tetrisv2:${BUILD_NUMBER}'
                    	sh 'docker push avian19/tetrisv2:${BUILD_NUMBER}'
                	}
            	}
        	}
    	}
    	stage("TRIVY Image Scan") {
        	steps {
            	sh 'trivy image avian19/tetrisv2:${BUILD_NUMBER} > trivyimage.txt'
        	}
    	}
    	stage('Checkout Code') {
        	steps {
            	git branch: 'master', url: 'https://github.com/AmanPathak-DevOps/End-to-End-Kubernetes-DevSecOps-Tetris-Project.git'
        	}
    	}
    	stage('Update Deployment file') {
        	environment {
            	GIT_REPO_NAME = "End-to-End-Kubernetes-DevSecOps-Tetris-Project"
            	GIT_USER_NAME = "AmanPathak-DevOps"
        	}
        	steps {
            	dir('Manifest-file') {
                	withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    	sh '''
                        	git config user.email "aman07pathak@gmail.com"
                        	git config user.name "AmanPathak-DevOps"
                        	BUILD_NUMBER=${BUILD_NUMBER}
                        	echo $BUILD_NUMBER
                        	imageTag=$(grep -oP '(?<=tetrisv2:)[^ ]+' deployment-service.yml)
                        	echo $imageTag
                        	sed -i "s/tetrisv2:${imageTag}/tetrisv2:${BUILD_NUMBER}/" deployment-service.yml
                        	git add deployment-service.yml
                        	git commit -m "Update deployment Image to version \${BUILD_NUMBER}"
                        	git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:master
                    	'''
                	}
            	}
        	}
    	}
	}
}
