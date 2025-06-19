pipeline
{
    agent any

    environment 
    {
        APP_NAME = 'to-do-list'
        BUILD_ID_TAG = "${env.BUILD_NUMBER}"
        WORK_DIR = "backend/${APP_NAME}"
        IMAGE_NAME = "registry.local/${APP_NAME}:v${BUILD_ID_TAG}"
        CONTAINER_NAME = "${APP_NAME}_container_${BUILD_ID_TAG}"
        EXPOSED_PORT = "3001"
        
        // Docker Images

        MAVEN = "adoptopenjdk/maven-openjdk11:latest"
        openjdk = "openjdk:11-jre-slim"
    }
        

    options
    {
        timeout(time: 1, unit: 'HOURS')
        buildDiscarder(logRotator(numToKeepStr:'12'))
        skipDefaultCheckout(true)
        timestamps()
    }
    
    stages
    {
        stage("Cleanup Workspace & Checkout from SCM")
        {
            steps
            {
              echo "\n\tCleanning Workspace ..."
              cleanWs()
              checkout scm
            }
        }

        stage("Maven Build") 
        {
            agent 
            {
                docker 
                {
                    image "${MAVEN}"
                    args "--network jenkins"
                    reuseNode true
                }
            }
    
            steps 
            {
                dir("${WORK_DIR}") 
                {
                    sh """
                        pwd && ls -la
                        echo 'Building project using Maven...'
                        mvn clean package -DskipTests
                    """
                }
            }
        }

        stage("Build Docker Image") 
        {
            steps
            {
                dir("${WORK_DIR}")
                {
                    sh """
                        echo 'Building Docker image: ${IMAGE_NAME}'
                        docker build -t ${IMAGE_NAME} .
                    """
                }
            }
        }

        stage("Deploy Approval")
        {
            steps
            {
                script
                {    
                    input message: 'Do you approve Staging Setup ?', ok: 'OK'
                }
                // ToDo: Send Email Notification 
            }
        }

        stage("Deploy using Docker Container") 
        {
            steps 
            {
                sh """
                    echo 'Cleaning any existing container...'
                    docker rm -f ${CONTAINER_NAME} || true

                    echo ' Running Docker container: ${CONTAINER_NAME}'
                    docker run -d --name ${CONTAINER_NAME} \
                        --network jenkins \
                        -p ${EXPOSED_PORT}:${EXPOSED_PORT} \
                        ${IMAGE_NAME}
                """
            }
        }
    }

    post
    {
        // aborted 
        // {

        //     echo "Sending message to Slack"
        //     slackSend (color: "${env.SLACK_COLOR_WARNING}",
        //                channel: "${params.SLACK_CHANNEL_2}",
        //                message: "*ABORTED:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} by ${env.USER_ID}\n More info at: ${env.BUILD_URL}")
        // }
        success 
        {
            echo "CI/CD pipeline completed successfully for ${APP_NAME} #${BUILD_NUMBER}"
            // Example: send a notification with the success message
            // sendNotification("Text Success message")
        }

        failure
        {
            echo "CI/CD pipeline failed for ${APP_NAME} #${BUILD_NUMBER}"
            // Example: send a notification with the failure message
            // sendNotification("Text Failure Message")
        }
    }
}

//waitUntil
//quiet 
//test
