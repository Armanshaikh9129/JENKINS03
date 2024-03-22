pipeline {
    agent any
    
    environment {
        TAG = "${GIT_COMMIT}"
        
        // DEV
        DEV_DC_URL = "https://registry.hub.docker.com/armansk9129/dev-jenkins:latest"
        DEV_DC_CREDS = "DOCKER/DEV"
        DEV_TAG = "${env.TAG}"

        // QA
        QA_DC_URL = "https://registry.hub.docker.com/armansk9129/qa-jenkins:latest"
        QA_DC_CREDS = "DOCKER/qa"
        QA_TAG = "${env.TAG}"

        // Stage
        STAGE_DC_URL = "https://registry.hub.docker.com/armansk9129/dev-jenkins:latest"
        STAGE_DC_CREDS = "DOCKER/DEV"
        STAGE_TAG = "${env.TAG}"

        // PROD
        PROD_DC_URL = "https://registry.hub.docker.com/armansk9129/dev-jenkins:latest"
        PROD_DC_CREDS = "DOCKER/DEV"
        PROD_TAG = "${env.TAG}"
    }
    
    stages {
        stage('Docker Image Build IN Dev') {
            when {
                branch 'dev'
            }
            steps {
                echo "Building Docker Image and Push into Dockerhub" 
                script {
                    dockerBuildPush(${env.DEV_DC_URL}, ${env.DEV_DC_CREDS}, ${env.DEV_TAG})
                }
            }
        }
        
        stage('Docker Image pull tag push to qa') {
            when {
                branch 'qa'
            }
            steps {
                echo "Pulling Docker Image and Tagging and Push To qa" 
                script {
                dockerPullTagPush(${env.DEV_DC_URL}, ${env.DEV_DC_CREDS}, ${env.DEV_TAG}, ${env.QA_DC_URL}, ${env.QA_DC_CREDS}, ${env.QA_TAG})
                }
            }
        }
        
        stage('Docker Image pull tag push to stage') {
            when {
                branch 'stage'
            }
            steps {
                echo "Pulling Docker Image and Tagging and Push To stage" 
                script {
                    dockerPullTagPush(${env.QA_DC_URL}, ${env.QA_DC_CREDS}, ${env.QA_TAG}, ${env.STAGE_DC_URL}, ${env.STAGE_DC_CREDS}, ${env.STAGE_TAG})
                }
            }
        }
        
        stage('Docker Image pull tag push to prod') {
            when {
                branch 'prod'
            }
            steps {
                echo "Pulling Docker Image and Tagging and Push To stage" 
                script {
                    dockerPullTagPush(${env.STAGE_DC_URL}, ${env.STAGE_DC_CREDS}, ${env.STAGE_TAG}, ${env.PROD_DC_URL}, ${env.PROD_DC_CREDS}, ${env.PROD_TAG})
                }
            }
        }
    }
    
    post { 
        always { 
            echo 'Deleting Project now !! '
            deleteDir()
        }
    }
}

    // Function for Docker Build and Push for DEV
    def dockerBuildPush (String SRC_DH_URL, String SRC_DH_CREDS, String SRC_DH_TAG) {
    def app = docker.build(SRC_DH_TAG)
    docker.withRegistry(SRC_DH_URL, SRC_DH_CREDS) {
        app.push()
    }
}

    // Function for Docker Pull, Tag, and Push for QA, Stage, and Prod
    def dockerPullTagPush (String SRC_DH_URL, String SRC_DH_CREDS, String SRC_DH_TAG, String DEST_DH_URL, String DEST_DH_CREDS, String DEST_DH_TAG) {
   
    // Pull
    docker.withRegistry(SRC_DH_URL, SRC_DH_CREDS) {
        docker.image(SRC_DH_TAG).pull()
    }
    sh 'echo Image pulled successfully...'

    // Tag
    sh 'echo Tagging Docker image...'
    sh "docker tag ${SRC_DH_TAG} ${DEST_DH_TAG}" 

    // Push
    docker.withRegistry(DEST_DH_URL, DEST_DH_CREDS) {
        docker.image(DEST_DH_TAG).push()
    }

    sh 'echo Image Pushed successfully...'
    sh 'echo Deleting Local docker Images'
    sh "docker image rm ${SRC_DH_TAG}"  
    sh "docker image rm ${DEST_DH_TAG}" 
}
