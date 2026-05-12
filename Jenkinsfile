pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        IMAGE_NAME = "subhashrokkala/bms:latest"
        CONTAINER_NAME = "bms"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from Git') {
            steps {

                checkout scmGit(
                    branches: [[name: '*/main']],
                    extensions: [],
                    userRemoteConfigs: [[
                        credentialsId: 'git-creds',
                        url: 'https://github.com/Subhash-Rokkala/Book-My-Show.git'
                    ]]
                )

                sh 'ls -la'
            }
        }

        stage('SonarQube Analysis') {
            steps {

                withSonarQubeEnv('sonar-server') {

                    sh """
                    \$SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=BMS \
                    -Dsonar.projectKey=BMS
                    """
                }
            }
        }

        // OPTIONAL
        // Comment this stage if SonarQube blocks pipeline

        stage('Quality Gate') {
            steps {

                script {

                    timeout(time: 2, unit: 'MINUTES') {

                        waitForQualityGate abortPipeline: false
                    }
                }
            }
        }

        stage('Install Dependencies') {
            steps {

                sh '''
                cd bookmyshow-app

                ls -la

                if [ -f package.json ]; then

                    rm -rf node_modules

                    npm install --legacy-peer-deps

                else
                    echo "package.json not found!"
                    exit 1
                fi
                '''
            }
        }

        stage('Trivy FS Scan') {
            steps {

                sh '''
                trivy fs . > trivyfs.txt
                '''
            }
        }

        stage('Docker Build & Push') {
            steps {

                script {

                    withDockerRegistry(
                        credentialsId: 'docker-creds',
                        toolName: 'docker'
                    ) {

                        sh '''
                        echo "Building Docker image..."

                        docker build --no-cache \
                        -t $IMAGE_NAME \
                        -f bookmyshow-app/Dockerfile \
                        bookmyshow-app

                        echo "Pushing Docker image..."

                        docker push $IMAGE_NAME
                        '''
                    }
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {

                sh '''
                trivy image $IMAGE_NAME > trivyimage.txt
                '''
            }
        }

        stage('Deploy Container') {
            steps {

                sh '''
                echo "Stopping old container..."

                docker stop $CONTAINER_NAME || true

                docker rm $CONTAINER_NAME || true

                echo "Removing old image..."

                docker rmi $IMAGE_NAME || true

                echo "Pulling latest image..."

                docker pull $IMAGE_NAME

                echo "Running new container..."

                docker run -d \
                --restart=always \
                --name $CONTAINER_NAME \
                -p 3000:3000 \
                $IMAGE_NAME

                echo "Checking running containers..."

                docker ps -a

                echo "Waiting for app startup..."

                sleep 15

                echo "Container logs..."

                docker logs $CONTAINER_NAME
                '''
            }
        }
    }

    post {

        success {

            emailext(
                attachLog: true,

                subject: "SUCCESS: ${env.JOB_NAME} Build #${env.BUILD_NUMBER}",

                body: """
                <h2>Build Successful</h2>

                <p><b>Project:</b> ${env.JOB_NAME}</p>

                <p><b>Build Number:</b> ${env.BUILD_NUMBER}</p>

                <p><b>Status:</b> SUCCESS</p>

                <p><b>Build URL:</b>
                <a href="${env.BUILD_URL}">
                ${env.BUILD_URL}
                </a></p>
                """,

                to: 'mr.siddu1432@gmail.com',

                attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
            )
        }

        failure {

            emailext(
                attachLog: true,

                subject: "FAILED: ${env.JOB_NAME} Build #${env.BUILD_NUMBER}",

                body: """
                <h2>Build Failed</h2>

                <p><b>Project:</b> ${env.JOB_NAME}</p>

                <p><b>Build Number:</b> ${env.BUILD_NUMBER}</p>

                <p><b>Status:</b> FAILED</p>

                <p><b>Check Console Output:</b>
                <a href="${env.BUILD_URL}">
                ${env.BUILD_URL}
                </a></p>
                """,

                to: 'mr.siddu1432@gmail.com'
            )
        }
    }
}
