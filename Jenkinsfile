pipeline {
    agent any
    
    environment {
        TAG = "${GIT_COMMIT}"
        
        // DEV
        DEV_DC_URL = "https://registry.hub.docker.com/armansk9129/dev-jenkins"
        DEV_DC_CREDS = "DOCKER/DEV"
        DEV_TAG = "${env.TAG}"

        // QA
        QA_DC_URL = "https://registry.hub.docker.com/armansk9129/qa-jenkins"
        QA_DC_CREDS = "DOCKER/QA"
        QA_TAG = "${env.TAG}"

        // Stage
        STAGE_DC_URL = "https://registry.hub.docker.com/armansk9129/stage-jenkins"
        STAGE_DC_CREDS = "DOCKER/STAGE"
        STAGE_TAG = "${env.TAG}"

        // PROD
        PROD_DC_URL = "https://registry.hub.docker.com/armansk9129/prod-jenkins:latest"
        PROD_DC_CREDS = "DOCKER/PROD"
        PROD_TAG = "${env.TAG}"
    }

    parameters {
        choice(name: 'account', choices: ['dev', 'qa', 'stage', 'prod'], description: 'Select the environment.')
        string(name: 'commit_id', defaultValue: 'latest', description: 'provide commit id.')
    }
    
    stages {
        stage('Docker Image Build IN Dev') {
            when {
                expression {
                    params.account == 'dev'
                }
            }
            steps {
                echo "Building Docker Image Logging in to Docker Hub & Pushing the Image" 
                
                dockerBuildPush(env.DEV_DC_URL, env.DEV_DC_CREDS, env.DEV_TAG)

                echo 'Image Pushed to DEV'
                sh 'echo Deleting Local docker DEV Image'
            }
        }
        stage('Pull Tag push to QA') {
            when {
                expression {
                    params.account == 'qa'
                }
            }
            steps {
                dockerPullTagPush(env.DEV_DC_URL, env.DEV_DC_CREDS, env.DEV_TAG, env.QA_DC_URL, env.QA_DC_CREDS, env.QA_TAG)
            }
        }
        stage('Pull Tag push to STAGE') {
            when {
                expression {
                    params.account == 'stage'
                }
            }
            steps {
                dockerPullTagPush(env.QA_DC_URL, env.QA_DC_CREDS, env.QA_TAG, env.STAGE_DC_URL, env.STAGE_DC_CREDS, env.STAGE_TAG)
            }
        }
        stage('Pull Tag push to PROD') {
            when {
                expression {
                    params.account == 'prod'
                }
            }
            steps {
                dockerPullTagPush(env.STAGE_DC_URL, env.STAGE_DC_CREDS, env.STAGE_TAG, env.PROD_DC_URL, env.PROD_DC_CREDS, env.PROD_TAG) 
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
def dockerBuildPush(String SRC_DH_URL, String SRC_DH_CREDS, String SRC_DH_TAG) {
    def app = docker.build(SRC_DH_TAG)
    docker.withRegistry(SRC_DH_URL, SRC_DH_CREDS) {
        app.push()
    }
}

// Function for Docker Pull, Tag, and Push for QA, Stage, and Prod
def dockerPullTagPush(String SRC_DH_URL, String SRC_DH_CREDS, String SRC_DH_TAG, String DEST_DH_URL, String DEST_DH_CREDS, String DEST_DH_TAG) {
    // Pull
    docker.withRegistry(SRC_DH_URL, SRC_DH_CREDS) {
        docker.image(SRC_DH_TAG).pull()
    }
    echo 'Image pulled successfully...'

    // Tag
    echo 'Tagging Docker image...'
    sh "docker tag ${SRC_DH_TAG} ${DEST_DH_TAG}" 

    // Push
    docker.withRegistry(DEST_DH_URL, DEST_DH_CREDS) {
        docker.image(DEST_DH_TAG).push()
    }

    echo 'Image Pushed successfully...'
    echo 'Deleting Local docker Images'
    sh "docker image rm ${SRC_DH_TAG}"  
    sh "docker image rm ${DEST_DH_TAG}" 
}
