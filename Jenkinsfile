pipeline {
    environment {
        IMAGE_NAME = "static-website"
        STAGING = "static-website-staging"
        PRODUCTION = "static-website-production"
    }
    agent none
    stages {
        stage('Build image') {
            agent any
            steps {
                script {
                    sh 'docker build -t $IMAGE_NAME:$BUILD_NUMBER .'
                }
            }
        }
        stage('Run container based on build image') {
            agent any
            steps {
                script {
                    sh '''
                        docker run --name $IMAGE_NAME -d -p 80:80 $IMAGE_NAME:$BUILD_NUMBER
                        sleep 5
                    '''
                }
            }
        }
        stage('Test container') {
            agent any
            steps {
                script {
                    sh '''
                        curl http://172.17.0.1 | grep 'Dimension'
                        if [ $? -eq 0 ]; then echo "Acceptance test succeed"; fi
                    '''
                }
            }
        }
        stage('Artifact') {
            agent any
            steps {
                script {
                    sh 'docker rm -f ${IMAGE_NAME}'
                }
            }
        }
        stage('Push image to Docker Hub') {
            agent any
            environment {
                DOCKER_CREDENTIALS = credentials('docker-credentials')
            }
            steps {
                script {
                sh '''
                    echo $DOCKER_CREDENTIALS_PSW | docker login -u $DOCKER_CREDENTIALS_USR --password-stdin
                    docker tag $IMAGE_NAME:$BUILD_NUMBER mozkadocker/$IMAGE_NAME:$BUILD_NUMBER
                    docker push mozkadocker/$IMAGE_NAME:$BUILD_NUMBER
                '''
                }
            }
        }
        stage('Push image in review and deploy it') {
            when { changeRequest () }
            agent {
                docker { image 'coxauto/aws-ebcli' }
            }
            environment {
                AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
                AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
            }
            steps {
                script {
                sh '''
                    aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                    aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                    aws configure set region us-east-1
                    eb init --region us-east-1 --platform Docker static-website-review_$BUILD_NUMBER
                    eb create review-env-$BUILD_NUMBER
                    eb status
                '''
                }
            }
        }
        stage('Delete review environmnent') {
            when { changeRequest () }
            agent {
                docker { image 'coxauto/aws-ebcli' }
            }
            environment {
                AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
                AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
            }
            steps {
                script {
                    timeout(time: 5, unit: "MINUTES") {
                        input message: "Do you want to remove the Review Environment ?", ok: 'Yes'
                }
                sh '''
                    aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                    aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                    aws configure set region us-east-1
                    aws elasticbeanstalk terminate-environment --environment-name review-env-$BUILD_NUMBER
                    aws elasticbeanstalk delete-application --application-name static-website-review_$BUILD_NUMBER
                '''
                }
            }
        }
        stage('Push image in staging and deploy it') {
            when {
                expression { GIT_BRANCH == 'origin/main' }
            }
            agent {
                docker { image 'coxauto/aws-ebcli' }
            }
            environment {
                AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
                AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
            }
            steps {
                script {
                sh '''
                    aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                    aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                    aws configure set region us-east-1
                    eb init --region us-east-1 --platform Docker static-website-staging_$BUILD_NUMBER
                    eb create staging-env-$BUILD_NUMBER
                    eb status
                '''
                }
            }
        }
        stage('Push image in production and deploy it') {
            when {
                expression { GIT_BRANCH == 'origin/main' }
            }
            agent {
                docker { image 'coxauto/aws-ebcli' }
            }
            environment {
                AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
                AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
            }
            steps {
                script {
                sh '''
                    aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                    aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                    aws configure set region us-east-1
                    eb init --region us-east-1 --platform Docker static-website-production_$BUILD_NUMBER
                    eb create production-env-$BUILD_NUMBER
                    eb status
                '''
                }
            }
        }
    }
    post {
        success {
            slackSend (color: "#028000", message: "Pipeline succeed")
        }
        failure {
            slackSend (color: "#c70039", message: "Pipeline failed")
        }
    }
}
