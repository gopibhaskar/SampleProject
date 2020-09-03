/* This script will provide basic idea to create jenkins file and deploy application on deployment environment.
Deployment environment can be anything like Docker, kubernetes or cloud.
Create 'deploy.properties' file at same location and save all the build and deployment properties. This file will be refered by this pipeline for build and deployment. */
def workspace;
def props='';
def tagName="""${JOB_NAME}-${BUILD_TIMESTAMP}"""
def branchName;
def commit_username;
def commit_Email;
def appDeployProcess;
def imageName;
def envMessage='';
node{
    stage('Checkout Code')
    {
        try
        {
            checkout scm
            props = readProperties  file: """deploy.properties"""
			workspace = pwd ()
			/*branchName=sh(returnStdout: true, script: 'git symbolic-ref --short HEAD').trim()
			commit_username=sh(returnStdout: true, script: '''username=$(git log -1 --pretty=%ae)
																echo ${username%@*} ''').trim();
			commit_Email=sh(returnStdout: true, script: '''Email=$(git log -1 --pretty=%ae)
																echo $Email''').trim(); */
			branchName= "master"
			commit_username="Papanaboina, Gopibhaskar"
			commit_Email="Gopibhaskar.Papanaboina@sabre.com"
			echo commit_username
			echo commit_Email
			echo branchName
        }
    	catch (e) {
    		currentBuild.result='FAILURE'
    		//logJIRATicket(currentBuild.result, "At Stage Checkout Code", props['JIRAprojectid'], props['JIRAissuetype'], commit_Email, props['JIRAissuereporter'])
    		//notifyBuild(currentBuild.result, "At Stage Checkout Code", "", commit_Email)
    		throw e
    	}
		catch (error) {
				currentBuild.result='FAILURE'
				//logJIRATicket(currentBuild.result, "At Stage Checkout Code", props['build.JIRAprojectid'], props['build.JIRAissuetype'], commit_Email, props['build.JIRAissuereporter'])
				//notifyBuild(currentBuild.result, "At Stage Checkout Code", "", commit_Email)
				throw error
			}
    }
	stage ('Check Environment')
    { 
    	try
		{
			//check if deployment environment is up and running
		}
	 catch (e) {
    		currentBuild.result='FAILURE'
    		//logJIRATicket(currentBuild.result,  "At Stage Check Environment", props['JIRAprojectid'], props['JIRAissuetype'], commit_Email, props['JIRAissuereporter'])
    		//notifyBuild(currentBuild.result, "At Stage Check Environment", "", commit_Email)
    		throw e
    	}
    }
    stage ('Static Code Analysis')
    { 
     try{
			//sh """echo ${workspace}"""
			def scannerHome = tool 'sonar-runner';
			withSonarQubeEnv('SonarQContainer')
			{
				sh 'mvn clean package sonar:sonar'
			}
        }
    	catch (e) {
    		currentBuild.result='FAILURE'
    		//logJIRATicket(currentBuild.result,  "At Stage Static Code Analysis", props['JIRAprojectid'], props['JIRAissuetype'], commit_Email, props['JIRAissuereporter'])
    		//notifyBuild(currentBuild.result, "At Stage Static Code Analysis", "", commit_Email)
    		throw e
    	}
     }
    stage ('Build')
    { 
		try
		{
			sh """mvn clean compile"""
		}
		catch (e) 
		{
    		currentBuild.result='FAILURE'
    		//logJIRATicket(currentBuild.result,  "At Stage Build", props['JIRAprojectid'], props['JIRAissuetype'], commit_Email, props['JIRAissuereporter'])
    		//notifyBuild(currentBuild.result, "At Stage Build", "", commit_Email)
    		throw e
    	}
	 
    }
    stage ('Unit Test Execution')
    { 
      try {
            sh """mvn clean test"""
        }
    	catch (e) {
    		currentBuild.result='FAILURE'
    		//logJIRATicket(currentBuild.result, "At Stage Unit Testing", props['JIRAprojectid'], props['JIRAissuetype'], commit_Email, props['JIRAissuereporter'])
    		//notifyBuild(currentBuild.result, "At Stage Unit Testing", "", commit_Email)
    		throw e
    	}
    }
    stage ('Build')
    { 
		try
		{
			sh """mvn clean install"""
		}
		catch (e) 
		{
    		currentBuild.result='FAILURE'
    		//logJIRATicket(currentBuild.result,  "At Stage Build", props['JIRAprojectid'], props['JIRAissuetype'], commit_Email, props['JIRAissuereporter'])
    		//notifyBuild(currentBuild.result, "At Stage Build", "", commit_Email)
    		throw e
    	}
	 
    }

    stage ('Code Coverage')
    { 
     try
        {
		def scannerHome = tool 'sonar-runner';
			withSonarQubeEnv('SonarQContainer')
			{
				codeCoverage(scannerHome, """${SonarQContainer}""")
			}				
        }
    	catch (e) {
    		currentBuild.result='FAILURE'
    		//logJIRATicket(currentBuild.result, "At Stage Code Coverage", props['JIRAprojectid'], props['JIRAissuetype'], commit_Email, props['JIRAissuereporter'])
    		//notifyBuild(currentBuild.result, "At Stage Code Coverage", "", commit_Email)
    		throw e
    	}
    }
 /*   stage ('Create Docker Image')
    { 
        try {
				imageName="""${props['docker.registry']}/${props['deploy.app']}:${props['api.version']}"""
                sh """mvn package
				sudo docker build -t ${imageName} ."""
        }
    	catch (e) {
    		currentBuild.result='FAILURE'
    		//logJIRATicket(currentBuild.result, "At Stage Create Package", props['JIRAprojectid'], props['JIRAissuetype'], commit_Email, props['JIRAissuereporter'])
    		//notifyBuild(currentBuild.result, "At Stage Create Package", "", commit_Email)
    		throw e
    	}
    }
    stage ('Push Image to Docker Registry')
    { 
       try {
	   
			sh """sudo docker push ${imageName}"""
        }
    	catch (e) {
    		currentBuild.result='FAILURE'
    		throw e
    	}
    }
    stage ('Deploy to Environment')
    { 
        try 
		{
			def helmChartValue = readYaml file: "helmchart/${JOB_NAME}/values.yaml"
			helmChartValue.microservice.port = props['app.port'].replaceAll("\'","");
			helmChartValue.microservice.image = "$imageName"
			helmChartValue.microservice.namespace = """${props['kubernetesnamespace']}"""
			helmChartValue.microservice.configServerURI = """${props['ConfigserverURL']}"""
			
			fileOperations([fileDeleteOperation(excludes: '', includes: "helmchart/${JOB_NAME}/values.yaml")])
			writeYaml file: "helmchart/${JOB_NAME}/values.yaml", data: helmChartValue
			//you can use any deployment tool here to deploy helm chart on kubernetes cluster 
			//or
			//you deploy container on docker environment
			sh """ssh user@deploymentserverhost 
			cd helmchart/${JOB_NAME}
			helm install --name ${props['deploy.app']} . """				
        }
    	catch (e) {
    		currentBuild.result='FAILURE'
    		throw e
    	}
    }
	stage ('Validate Microservice Deployment')
    { 
        try {
				sleep 120
				def chkmicroservice=sh(returnStdout: true, script: """curl -s http://${props['environment.URL']}:${props['app.port']}/health | jq '.status' | tr -d '"' """).trim();
				def chkDeployment='';
				 if(chkmicroservice != "UP")
				{
					chkDeployment = chkDeployment + """\n Microservice - ${JOB_NAME} connection failed (Status:${chkmicroservice})"""
				}
				
				if (chkDeployment != "")
				{
					error ("""\n Warning:\n Microservice deployment is unstable ${chkDeployment} \n """)
				}
			}
			catch (e) {
				currentBuild.result='FAILURE'
				throw e
			}
			catch (error) {
				currentBuild.result='UNSTABLE'
				//logJIRATicket(currentBuild.result, "At Stage Validate Microservice Deployment", props['JIRAprojectid'], props['JIRAissuetype'], commit_Email, props['JIRAissuereporter'])
				//notifyBuild(currentBuild.result, "At Stage Validate Microservice Deployment", "", commit_Email)
				echo """${error.getMessage()}"""
				//throw e
			}
    }*/
    /*stage ('Log JIRA Ticket for Code Promotion')
    {
        try {
            logJIRATicket('SUCCESS', "At Stage Log JIRA Ticket", props['JIRAprojectid'], props['JIRAissuetype'], commit_Email, props['JIRAissuereporter'])
    	}
    	catch (e) {
    		currentBuild.result='FAILURE'
    		logJIRATicket(currentBuild.result, "At Stage Log JIRA Ticket", props['JIRAprojectid'], props['JIRAissuetype'], commit_Email, props['JIRAissuereporter'])
    		notifyBuild(currentBuild.result ,"At Stage Log JIRA Ticket", """Version tag created with name '${tagName}'. but no JIRA ticket logged.""", commit_Email)
    		throw e
    	}
    }
    notifyBuild(currentBuild.result, "", """Version tag created with name '${tagName}' on '${branchName}' branch \n Build successfull, no JIRA ticket logged. """, commit_Email)
}
def notifyBuild(String buildStatus, String buildFailedAt, String bodyDetails, String commit_Email) 
{
	buildStatus = buildStatus ?: 'SUCCESS'
	def details = """Please find attahcment for log and Check console output at ${BUILD_URL}\n \n "${bodyDetails}"
	\n"""
	emailext attachLog: true,
	notifyEveryUnstableBuild: true,
	recipientProviders: [[$class: 'CulpritsRecipientProvider'], [$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']],
	body: details, 
	subject: """${buildStatus}: Job ${JOB_NAME} [${BUILD_NUMBER}] ${buildFailedAt}""", 
	to: """email@server.com,${commit_Email}"""
}
def logJIRATicket(String buildStatus, String buildFailedAt, String projectid, String issuetype, String assignTo, String issueReporter)
{
	buildStatus = buildStatus ?: 'SUCCESS'
	if (buildStatus == 'FAILURE' ){
	String Title="""${buildStatus} ${buildFailedAt} OF ${JOB_NAME}[${BUILD_NUMBER}]"""
	withEnv(['JIRA_SITE=Localhost']) {
		// Look at IssueInput class for more information.
	def Issue = [fields: [ project: [id: projectid],	
						summary: Title,
						description: 'New JIRA Created from Jenkins.',
						issuetype: [id: issuetype],
						assignee: [name: assignTo],
						reporter: [name: issueReporter]]]
		def Issues = [issueUpdates: [Issue]]
		response = jiraNewIssues issues: Issues
		echo """${response}"""
	}
	}
	else {
	echo "Build is successfull, no JIRA ticket logged."
	}
}*/
}

/*def staticCodeAnalysis(String scannerHome, String sonarHosturl)
{
	sh """	
	${scannerHome}/bin/sonar-runner -Dsonar.host.url=${sonarHosturl} -Dsonar.login=admin -Dsonar.password=admin"""
}*/
def codeCoverage(String scannerHome, String sonarHosturl)
{
	sh """	
	${scannerHome}/bin/sonar-runner -Dsonar.host.url=${sonarHosturl} -Dsonar.login=admin -Dsonar.password=admin -Dsonar.java.binaries=target/classes -Dsonar.jacoco.reportPaths=target/jacoco.exec"""
}
