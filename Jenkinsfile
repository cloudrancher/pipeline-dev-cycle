pipeline() {
	agent any
	environment {
		buildConfig=null;
		devDeploy=null;
	}
	stages {
		stage('UpdateBuildConfig') {
			steps {
				echo '*****************  Creating S2I build config'
				script {
				   openshift.withCluster() {
				  		openshift.withProject() {
				    	    def buildObjList = openshift.process('-f','openshift/templates/build.yml','-p',
								  "SOURCE_REPOSITORY_URL=${params.SOURCE_REPOSITORY_URL}",
								  "SOURCE_REPOSITORY_REF=${params.SOURCE_REPOSITORY_REF}");
							buildConfig = openshift.apply(buildObjList).narrow('bc');

							echo "**** Created Build Config: ${buildConfig.object().metadata.name}";				
						}            
				   }              
				}
			}
		}
	}
	stage('Build') {
	  steps {
	    echo '*****************  Running S2I build'
	    script {
	       openshift.withCluster() {
	          openshift.withProject() {
	          def builds = buildConfig.related('build');
	          builds.untilEach(1) { 
	            echo "Watching ${it.object().metadata.name}"
	            return it.object().status.phase == "Complete"
	          }
	          
	          echo "**** All Builds Complete"
	        }
	       }
	    }
	  }
	}
	stage('DevDeploy') {
	  steps {
	    echo '*****************  Update Dev Deploy Pipeline'
	    script {
	       openshift.withCluster() {
	          openshift.withProject() {
	            def deployObjList = openshift.process('-f','openshift/templates/deployment.yml');
	            devDeploy = openshift.apply(deployObjList).narrow("dc")
	            echo "**** Created Deploy Pipeline: ${devDeploy.object().metadata.name}"
	          }
	       }
	    }
	  }
	}
}
