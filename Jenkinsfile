pipeline { 
agent any 

parameters{
	    string(defaultValue: '', description: 'Enter Git Repo Name, build will fail if value is empty', name: 'GITREPO', trim: false) 
	    gitParameter branchFilter: 'origin.*/(.*)', 
					 defaultValue: '', 
					 name: 'GIT_BRANCHES', 
					 type: 'PT_BRANCH', 
		                         description: 'By default master branch is selected',
		    			 listSize: '1',
		                         useRepository: '.*${params.GITREPO}.git'					 
	    choice(choices: ['--None--','DEV','QA','STAGING','PRODUCTION'], description: 'Select An Environment Name', name: 'Environment_Name')
	    choice(choices: ['--None--','PLATFORM','DSS','XPONENT','IDX','AMERITRUST','FDS','RA','iGCB-CLO'], description: 'Select a Product name this service belongs to', name: 'PRODUCT_NAME')	  
	    choice(choices: ['--None--','JDK_11','JDK_8','JDK_6'], description: 'Select JDK version', name: 'SELECT_TECH')
	    choice(choices: ['--None--','Maven','Gradle'], description: 'Please select a build tool for project', name: 'BUILD_TOOL_SELECTION')
	    booleanParam(name: 'Sonar_Analysis', defaultValue: true, description: 'By default Sonar Analysis will be executed')
	    booleanParam(name: 'Veracode_Scan', defaultValue: true, description: 'By default Veracode Scan will be executed')
	    booleanParam(name: 'Twistlock_Scan', defaultValue: true, description: 'By default Image Scan will be executed')   
	    }
	//Environment has Environment variables
environment {
              //  PROJECT_VERSION = readMavenPom().getVersion()	
              //  VERSION = readMavenPom().getVersion() 			
		GIT_URL = "${params.GITREPO}"
	        //Environment_Name
    }
options {
		//skipStagesAfterUnstable()
		buildDiscarder(logRotator(numToKeepStr: '3'))
		timestamps()
}
stages {	
	tage('Read Properties'){
	steps{
		script{
					
		//User = ${BUILD_USER}			
		echo " Environment Selected is ${Environment_Name}"
		echo " PRODUCT NAME Selected is ${PRODUCT_NAME}"
		echo " Tool Selected is ${SELECT_TECH}"
		echo " Build Tool Selected: ${BUILD_TOOL_SELECTION}"
		toolSelection = "${SELECT_TECH}"
		echo "++++ ${toolSelection}"
		buildTool = "${BUILD_TOOL_SELECTION}"			
		jdkVersion = selectTool("${SELECT_TECH}")		
		echo "Jdk selected is: ${jdkVersion}"
		DOCKER_HUB_ACCOUNT = dockerHubSelector("${Environment_Name}")
		echo "Sonar Analysis Selected : ${params.Sonar_Analysis}"
		envAppend = appendHelmName("${Environment_Name}")		
		buildProps = readProperties file: 'build.properties'
		// Prinintg all build properties
		echo "Property file is : ${buildProps}"
		// getting all build properties into variables
		PROJECT_KEY = "${buildProps.project_key}"
		PROJECT_NAME = "${buildProps.project_name}"
		HELM_NAME = "${buildProps.helm_name}"
		YAMLFILE_NAME = "${buildProps.yamlfile_name}"
		FOLDER_NAME = "${buildProps.folder_name}"
		REPOSITORY_DOCKER = "${buildProps.repository_docker}"
		registry = "${DOCKER_HUB_ACCOUNT}/${REPOSITORY_DOCKER}"		
		newImage = "${DOCKER_HUB_ACCOUNT}/${REPOSITORY_DOCKER}:${envAppend}-${currentBuild.number}"		
			
			echo "Image Tag = ${newImage}"
			echo "Project_Key = ${PROJECT_KEY}"
			echo "Project_name = ${PROJECT_NAME}"
			echo "REPOSITORY_DOCKER = ${REPOSITORY_DOCKER}"
		}		
	}
   }
stage('SCM') { 	
	steps { 				
		echo "Pulling changes from branch ${params.GIT_BRANCHES} and from repo ${params.GITREPO}"		
		checkout poll: false, scm: [$class: 'GitSCM', branches: [[name: "${params.GIT_BRANCHES}"]], 
					    doGenerateSubmoduleConfigurations: false, 
		                            extensions: [[$class: 'CleanBeforeCheckout']], 
					    gitTool: 'Default', submoduleCfg: [], 
		                            userRemoteConfigs: [[credentialsId: 'GitHub', 
		                            url: "https://github.com/svenkyedem/${params.GITREPO}.git"]]
					   ]
		}
	}

stage('Maven BUILD') { 	
	when { 	
	    expression { 			    	
		    return params.BUILD_TOOL_SELECTION == 'Maven'
		    echo "In clean install, build tool selected is : ${buildTool}" 	           	    
	           }	
		}	
	tools {		
		jdk "${jdkVersion}"
  	}
	steps { 	
		echo "Starting Build"
		sh 'mvn clean install'
		echo 'Clean Install Completed'        					
		}
	}
stage('Gradle BUILD') { 	
	tools {
       	 gradle 'gradle-4.10'
   	 }
	steps { 		
			echo "In Build"
			//echo "Project_Key = ${buildProps.project_key} @#@#@#@#@# "
			echo "Starting Build"
			//sh 'mvn clean install'			
			//sh 'cd /opt/JenkinsHome/jobs/Initiations_Dev_Job/workspace/'
		 	sh 'gradle clean build'
			//echo 'Clean Install Complete'        
			//echo "${env.PROJECT_VERSION} --- PROJECT VERSION"
			//echo "$registry"
			//echo '###############################'		
     			echo '@@@ Starting Gradle Build @@@'
    		
      echo 'Completed Gradle Build'			
			}
	}
//Unit test Stage
stage('Maven JUNIT TEST'){
	when { 	   
	    expression { 		   
		     //return params.toolSelection != 'Python';	
		    return params.BUILD_TOOL_SELECTION == 'Maven'
		    echo "In unit test, build tool selected is : ${buildTool}" 	           	    
	           }	          
		}
	tools {
    	  	 jdk "${jdkVersion}"   	
  	   }
	steps {
		sh 'mvn test'
		echo 'Maven Test Completed'
		}
       }
stage('Gradle JUNIT TEST'){
	when { 	   
	    expression { 		   
		     //return params.toolSelection != 'Python';	
		    return params.BUILD_TOOL_SELECTION == 'Gradle'
		    echo "In unit test, build tool selected is : ${buildTool}" 	           	    
	           }	          
		}
	tools {
    	  	 gradle 'gradle-4.10' 	
  	   }
	steps {
		sh 'gradle test'
		echo 'Gradle Test Completed'
		}
       }
}
	
}
