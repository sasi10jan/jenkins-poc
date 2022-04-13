// backup Dynamic jenkins file
def timeStamp = Calendar.getInstance().getTime().format('YYYYMMdd-hhmmss',TimeZone.getTimeZone('IST'))
def changedFiles=[]
pipeline {
    agent any
    environment{
        //App-name 
        APP_NAME = 'API'
        APP = "API-${timeStamp}"
    }
    stages{
        stage('Pre Stage'){
            steps {
                echo 'checking folders'
				script{

                    stdout = bat( script: 'git.exe log -1 --pretty="format:Commit Message: %%s% %%%n%% Author: %%an% %%n% Date: %%aD%"' ,returnStdout: true).trim()
          		    env.GIT_COMMENT = (stdout.readLines().drop(1).join("\n"))

                    if(env.GIT_BRANCH == "main")  {
                        mail to: 'gagan.verma@apisero.com',
                        subject: "Approve Production Pipeline: ${currentBuild.fullDisplayName}",
                        body: "Please click  here  ${BUILD_URL} input to approve or reject the start of prod pipeline"

                        echo 'Starting Production pipeline due to the latest code commit in Prod branch.'

                        timeout(time: 30, unit: 'MINUTES'){
                        input "Start of Production?"}
                    }

					// def subfolders = bat(script: '@dir /B /AD | @findstr /L /V "tmp" | @findstr /L /V ".git"', returnStdout: true).split(/\n\r/)
					def changeLogSets = currentBuild.changeSets
                    for (int i = 0; i < changeLogSets.size(); i++) {
                        def entries = changeLogSets[i].items
                        for (int j = 0; j < entries.length; j++) {
                            def entry = entries[j]
                            // echo "${entry.commitId} by ${entry.author} on ${new Date(entry.timestamp)}: ${entry.msg}"
                            def files = new ArrayList(entry.affectedFiles)
                            for (int k = 0; k < files.size(); k++) {
                                def file = files[k]
                                // echo "  ${file.editType.name} ${file.path}"
                                changedFiles+=("${file.path}").split('/')[0]
                            }
                        }
                    }
		        }
            }
        }

        stage('Initialize'){
            environment {
                ANYPOINT_CREDENTIALS = credentials('anypoint.credentials')
                ANYPOINT_CLIENT_ID = credentials('client_id')
                ANYPOINT_CLIENT_SECRET = credentials('client_secret')
            }
            steps{
                echo 'Creating Stages for each change project folder'
                script{

                    changedFiles.removeAll(Collections.singleton("Jenkinsfile")) // removing Jenkinsfile coz dont want a stage for Jenkinsfile
                    for(String i : changedFiles.unique()){
                        echo "${i}"
                        if(env.GIT_BRANCH == "dev")  {  

                            stage("Build ${i} for DEV") {
                                echo 'Building  mule project due to the latest code commits in Dev branch'
                                echo "Building ${i}"
                                dir("${i}"){
                                    bat "mvn clean package -Djar.name=${i}-%APP%"
                                }
                            }

                            stage("Deploying ${i} for DEV") {
                                echo 'Deploying to the Development environment.'
                                echo "Deploying ${i}"
                                dir("${i}"){
                                    bat "mvn deploy -DskipTests -DmuleDeploy -Danypoint.username=%ANYPOINT_CREDENTIALS_USR% -Danypoint.password=%ANYPOINT_CREDENTIALS_PSW% -Danypoint.platform.client_id=%ANYPOINT_CLIENT_ID% -Danypoint.platform.client_secret=%ANYPOINT_CLIENT_SECRET% -Danypoint.env=Sandbox -Danypoint.region=us-east-1 -Danypoint.workers=1 -Danypoint.name=${i}-api-dev -Djar.name=${i}-%APP% -Dmule.artifact=%WORKSPACE%\\${i}\\target\\${i}-%APP%-mule-application.jar"
                                }
                            }

                        }else if(env.GIT_BRANCH == "qa")  {

                            mail to: 'gagan.verma@apisero.com',
                            subject: "Approve QA Pipeline: ${currentBuild.fullDisplayName}",
                            body: "Please click  here  ${BUILD_URL} input to approve or reject the deployment in QA for the following GIT commment : \n\n${env.GIT_COMMENT}\n" 
                            
                            stage("Build ${i} for QA") {
                                echo 'Building  mule project due to the latest code commits in QA branch'
                                echo "Building ${i}"
                                dir("${i}"){
                                    bat "mvn clean package -Djar.name=${i}-%APP%"
                                }
                            }

                            stage("Deploying ${i} for QA") {
                                echo 'Deploying to the QA environment.'
                                echo "Deploying ${i}"
                                timeout(time: 30, unit: 'MINUTES'){
                                    input "Deploy to QA?" 
                                }
                                dir("${i}"){
                                    bat "mvn deploy -DskipTests -DmuleDeploy -Danypoint.username=%ANYPOINT_CREDENTIALS_USR% -Danypoint.password=%ANYPOINT_CREDENTIALS_PSW% -Danypoint.platform.client_id=%ANYPOINT_CLIENT_ID% -Danypoint.platform.client_secret=%ANYPOINT_CLIENT_SECRET% -Danypoint.env=Sandbox -Danypoint.region=us-east-1 -Danypoint.workers=1 -Danypoint.name=${i}-api-qa -Djar.name=${i}-%APP% -Dmule.artifact=%WORKSPACE%\\${i}\\target\\${i}-%APP%-mule-application.jar"
                                }
                            }
                            
                        }else if(env.GIT_BRANCH == "main")  {

                            mail to: 'gagan.verma@apisero.com',
                            subject: "Approve Production Pipeline: ${currentBuild.fullDisplayName}",
                            body: "Please click  here  ${BUILD_URL} input to approve or reject the deployment in prod"
                            
                            stage("Build ${i} for Prod") {
                                echo 'Building  mule project due to the latest code commits in Prod branch'
                                echo "Building ${i}"
                                dir("${i}"){
                                    bat "mvn clean package -Djar.name=${i}-%APP%"
                                }
                            }
                            
                            stage("Deploying ${i} for Prod") {
                                echo 'Deploying to the Production environment.'
                                echo "Deploying ${i}"
                                timeout(time: 30, unit: 'MINUTES'){
                                    input "Deploy to Production?"
                                }
                                dir("${i}"){
                                    bat "mvn deploy -DskipTests -DmuleDeploy -Danypoint.username=%ANYPOINT_CREDENTIALS_USR% -Danypoint.password=%ANYPOINT_CREDENTIALS_PSW% -Danypoint.platform.client_id=%ANYPOINT_CLIENT_ID% -Danypoint.platform.client_secret=%ANYPOINT_CLIENT_SECRET% -Danypoint.env=Sandbox -Danypoint.region=us-east-1 -Danypoint.workers=1 -Danypoint.name=${i}-api-prod -Djar.name=${i}-%APP% -Dmule.artifact=%WORKSPACE%\\${i}\\target\\${i}-%APP%-mule-application.jar"
                                }
                            }
                            
                        }else  {
                            echo "Branch not expected" 
                        }

                    }
                }
            }
        }
    }
    post {
		aborted {
            		mail to: 'gagan.verma@apisero.com',
			subject: "Aborted Pipeline: ${currentBuild.fullDisplayName}",
			body: "This ${env.BUILD_URL} pipeline was aborted for the following git comment:\n\n${env.GIT_COMMENT}\n"
        	}
		failure {
			mail to: 'gagan.verma@apisero.com',
			subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
			body: "Something is wrong with ${env.BUILD_URL} and found the following git comment:\n\n${env.GIT_COMMENT}\n"
		}
		success {
			mail to: 'gagan.verma@apisero.com',
			subject: "Successful Pipeline: ${currentBuild.fullDisplayName}",
				body: "Sucessfully deployed and found the following git comment:\n\n${env.GIT_COMMENT}\n"
		}
	}
}
