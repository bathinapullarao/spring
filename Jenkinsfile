@Library('jenkins-shared-library')_
def CONTAINER_NAME="jenkins-pipeline"
def CONTAINER_TAG="latest"
def DOCKER_HUB_USER="bathinapullarao"
def HTTP_PORT="8085"

node {

    stage('declareEnvVariables'){
        def dockerHome = tool 'myDocker'
        def mavenHome  = tool 'myMaven'
        env.PATH = "${dockerHome}/bin:${mavenHome}/bin:${env.PATH}"
    }
stage('gitCheckout') {
        checkout scm
    }

stage('Build'){
        sh "mvn package"
    }
  stage('Junit'){
        try {
            sh "mvn test"        } catch(error){
            echo "The Maven can not perform Junit ${error}"
        }
     }
  stage('Sonar'){
        try {
            sh "mvn sonar:sonar"
        } catch(error){
            echo "The sonar server could not be reached ${error}"
        }
     }

    stage("Prune_deleteUnusedImages"){
        imagePrune(CONTAINER_NAME)
    }

    stage('buildDockerImage'){
        imageBuild(CONTAINER_NAME, CONTAINER_TAG)
    }
    stage('pushtoDockerRegistry'){
        withCredentials([usernamePassword(credentialsId: 'dockerHubAccount', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
            pushToImage(CONTAINER_NAME, CONTAINER_TAG, USERNAME, PASSWORD)
        }
    }

    stage('Run App'){
        runApp(CONTAINER_NAME, CONTAINER_TAG, DOCKER_HUB_USER, HTTP_PORT)
    }

    
    stage('approvalOfQA'){
    input "Deploy to QA?"
    }
    node {
    stage('deploy to QA'){
        dipQA(CONTAINER_NAME, CONTAINER_TAG, DOCKER_HUB_USER, 8086)
        }
    }
    stage('approvalOfUAT'){
    input "Deploy to UAT?"
    }
    node {
    stage('deploy to UAT'){
        dipUAT(CONTAINER_NAME, CONTAINER_TAG, DOCKER_HUB_USER, 8087)
        }
    }
    stage('approvalOfProd'){
    input "Deploy to Prod?"
    }
    node {
    stage('deploy to Prod'){
        dipProd(CONTAINER_NAME, CONTAINER_TAG, DOCKER_HUB_USER, 8088)
        }
    }
 post {
        always {
	    /* Use slackNotifier.groovy from shared library and provide current build result as parameter */   
            slackNotifier(currentBuild.currentResult)
          cleanWs()    }    }
}



    def imagePrune(containerName){
    try {
        sh "docker image prune -f"
        sh "docker stop $containerName"
    } catch(error){}
}

def imageBuild(containerName, tag){
    sh "docker build -t $containerName:$tag  -t $containerName --pull --no-cache ."
    echo "Image build complete"
}

def pushToImage(containerName, tag, dockerUser, dockerPassword){
    sh "docker login -u $dockerUser -p $dockerPassword"
    sh "docker tag $containerName:$tag $dockerUser/$containerName:$tag"
    sh "docker push $dockerUser/$containerName:$tag"
    echo "Image push complete"
}

def runApp(containerName, tag, dockerHubUser, httpPort){
    sh "docker pull $dockerHubUser/$containerName"
    sh "docker run -d --rm -p $httpPort:$httpPort --name $containerName $dockerHubUser/$containerName:$tag"
    echo "Application started on port: ${httpPort} (http)"
}



def dipQA(containerName, tag, dockerHubUser, httpPort){
    sh "docker pull $dockerHubUser/$containerName"
    sh "docker run -d --rm -p $httpPort:$httpPort --name $containerName $dockerHubUser/$containerName:$tag"
    echo "Application started on port: ${httpPort} (http)"
}
def dipUAT(containerName, tag, dockerHubUser, httpPort){
    sh "docker pull $dockerHubUser/$containerName"
    sh "docker run -d --rm -p $httpPort:$httpPort --name $containerName $dockerHubUser/$containerName:$tag"
    echo "Application started on port: ${httpPort} (http)"
}
def dipProd(containerName, tag, dockerHubUser, httpPort){
    sh "docker pull $dockerHubUser/$containerName"
    sh "docker run -d --rm -p $httpPort:$httpPort --name $containerName $dockerHubUser/$containerName:$tag"
    echo "Application started on port: ${httpPort} (http)"
}
