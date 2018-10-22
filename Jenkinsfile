node 
{
	stage('declareEnvVariables')
	{
        def mavenHome  = tool 'myMaven'
        env.PATH = "${mavenHome}/bin:${env.PATH}"
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
	
}
