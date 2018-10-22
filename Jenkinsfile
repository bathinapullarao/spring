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
            sh "mvn clean install"
         
        }
	
}
