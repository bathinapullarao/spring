def CONTAINER_NAME="jenkins-pipelin"
def CONTAINER_TAG="latest"
def DOCKER_HUB_USER="bathinapullarao"
def HTTP_PORT="8087"

node 
{
	stage('declareEnvVariables')
	{
        def dockerHome = tool 'myDocker'
        def mavenHome  = tool 'myMaven'
        env.PATH = "${dockerHome}/bin:${mavenHome}/bin:${env.PATH}"
        }
stage('gitCheckout') 
	{
        checkout scm
    	}
  stage('Sonar')
	{
        try {
            sh "mvn clean install sonar:sonar -P sonar"
            } 
	catch(error)
	    {
            echo "The sonar server could not be reached ${error}"
            }
        }
   stage("Prune_deleteUnusedImages")
	{
        imagePrune(CONTAINER_NAME)
        }
    stage('buildDockerImage')
	{
        imageBuild(CONTAINER_NAME, CONTAINER_TAG)
        }
    stage('pushtoDockerRegistry')
	{
        withCredentials([usernamePassword(credentialsId: 'dockerHubAccount', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) 
		{
            pushToImage(CONTAINER_NAME, CONTAINER_TAG, USERNAME, PASSWORD)
                }
	}	


    stage('Run App'){
        runApp(CONTAINER_NAME, CONTAINER_TAG, DOCKER_HUB_USER, HTTP_PORT)
    }
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

