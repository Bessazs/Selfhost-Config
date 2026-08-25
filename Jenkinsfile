pipeline {
    agent any

    environment {
        REGISTRY = 'ghcr.io'
        IMAGE_NAME = 'bessazs/selfhostConfig'
        IMAGE_TAG = "homolog-${BUILD_NUMBER}-${GIT_COMMIT[0..7]}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

//        stage('Run Tests') {
//            when {
//                branch 'homol'
//            }
//            steps {
//                sh '''
//                    docker build -t app-test:${IMAGE_TAG} .
//                    docker run --rm app-test:${IMAGE_TAG} pytest
//                '''
//            }
//      }

        stage('Build & Push Image (Homolog)') {
            when {
                branch 'homol'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'REGISTRY_CREDS', usernameVariable: 'REG_USER', passwordVariable: 'REG_PASS')]) {
                    sh '''
                        echo "$REG_PASS" | docker login $REGISTRY -u "$REG_USER" --password-stdin
                        docker build -t $REGISTRY/$IMAGE_NAME:$IMAGE_TAG -t $REGISTRY/$IMAGE_NAME:homolog-latest .
                        docker push $REGISTRY/$IMAGE_NAME:$IMAGE_TAG
                        docker push $REGISTRY/$IMAGE_NAME:homolog-latest
                    '''
                }
            }
        }

        stage('Deploy Homolog via Ansible') {
            when {
                branch 'homol'
            }
            steps {
                ansiblePlaybook(
                    playbook: 'ansible/deploy.yml',
                    inventory: 'ansible/inventory_homolog.ini',
                    credentialsId: 'SSH_ANSIBLE_KEY',
                    extraVars: [
                        registry_image: "${REGISTRY}/${IMAGE_NAME}",
                        image_tag: "${IMAGE_TAG}",
                        env_target: "homologation"
                    ]
                )
            }
        }
    }

    post {
        always {
            sh 'docker image prune -f'
        }
    }
}
