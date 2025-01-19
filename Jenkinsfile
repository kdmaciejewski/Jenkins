pipeline {
    agent any
    environment {
        AWS_ACCOUNT_ID="160227654080"
        AWS_DEFAULT_REGION="us-east-1"
	    CLUSTER_NAME="Pilot2Cluster"
	    SERVICE_NAME="Pilot22Service"
	    TASK_DEFINITION_NAME="Pilot2Task"
	    DESIRED_COUNT="1"
        IMAGE_REPO_NAME="pilot"
        //Do not edit the variable IMAGE_TAG. It uses the Jenkins job build ID as a tag for the new image.
        IMAGE_TAG="${env.BUILD_ID}"
        //Do not edit REPOSITORY_URI.
        REPOSITORY_URI = "160227654080.dkr.ecr.us-east-1.amazonaws.com/pilot"

//         REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
	    registryCredential = "Pilot"
        registryCredential2 = "Pilot2"

	    JOB_NAME = "PilotPipeline"
	    TEST_CONTAINER_NAME = "${JOB_NAME}-test-server"

}
   
    stages {

    // Building Docker image
    stage('Building image') {
      steps{
        script {
          dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
        }
      }
     }
    // Run container locally and perform tests
    stage('Running tests') {
      steps{
        sh 'docker run -i --rm --name "${TEST_CONTAINER_NAME}" "${IMAGE_REPO_NAME}:${IMAGE_TAG}" npm test -- --watchAll=false'
      }
    }

    // Uploading Docker image into AWS ECR
//     stage('Releasing') {
//      steps{
//          script {
// 			docker.withRegistry("https://" + REPOSITORY_URI, "ecr:${AWS_DEFAULT_REGION}:" + registryCredential2) {
//                     	dockerImage.push()
//             }
//          }
//        }
//      }
     stage('Releasing') {
     steps{
         script {
			docker.withRegistry("https://" + REPOSITORY_URI, registryCredential2) {
                    	dockerImage.push()
            }
         }
       }
     }

    // Update task definition and service running in ECS cluster to deploy
    stage('Deploy') {
     steps{
            withAWS(credentials: registryCredential, region: "${AWS_DEFAULT_REGION}") {
                script {
			sh "chmod +x -R ${env.WORKSPACE}"
			sh './script.sh'
                }
            }
         }
       }
     }
   // Clear local image registry. Note that all the data that was used to build the image is being cleared.
   // For different use cases, one may not want to clear all this data so it doesn't have to be pulled again for each build.
   post {
       always {
       sh 'docker system prune -a -f'
     }
   }
 }
