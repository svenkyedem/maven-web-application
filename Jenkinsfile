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
	
	
	  steps{
        script{
        MavenHome = tool 'maven 3.6.3'
        }
         sh "${MavenHome}/bin/mvn clean package"
         }
	}
stage("sonar")
  {
   when {
	    expression { 		 
		    echo 'In when for Sonar'
	        //    environment_name : "Dev"
		    return (params.Sonar_Analysis == true)		 
	           }
		}
   steps{
    script{
	scannerHome = tool 'Sonar'
	}
   // bat "mvn sonar:sonar"
   echo "${scannerHome}"
   withSonarQubeEnv('Sonar'){
   sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=maven-web-application -Dsonar.projectName=maven-web-application -Dsonar.projectVersion=1.1 -Dsonar.java.binaries=.  -Dsonar.sources=src/main"					
   echo "End test"
   }
  }
 }
 stage("Quality Gate"){
	  when {
	    expression { 		  
		    echo 'In when for QualityGate'	         
		    return (params.Sonar_Analysis == true || params.GIT_BRANCHES == 'Development');		   
	           }
		}
	    steps{
		    timeout(time: 10, unit: 'MINUTES') {
		  //Just in case something goes wrong, pipeline will be killed after a timeout
	    script{
		    if (params.Environment_Name == 'DEV') { 
		    sh 'sleep 60'
		    def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv	
		    
		    echo "${qg.status}"
		    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
		    echo 'Quality Gate Failed'
                    sh 'exit 0'
		    currentBuild.result = 'SUCCESS'
               	    } 
		    if (qg.status != 'OK') {
			 echo "${qg.status}"   
	   		 echo "Quality Gates failed"
     			   error "Pipeline aborted due to quality gate failure: ${qg.status}"
    			}
                 } else {
                     def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv	
		    echo "${qg}.status"
		    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
		    echo 'Quality Gate Failed'
                    sh 'exit 0'
		    currentBuild.result = 'SUCCESS'
               	    }  
		     //echo "Quality completed without if loop"
  	    	   }
	         }   
    	      }
   	}
   }	  
  } 


	
}
