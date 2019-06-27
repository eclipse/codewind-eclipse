#!groovy​

pipeline {
    agent any
    
    tools {
        jdk 'oracle-jdk8-latest'
    }
    
    options {
        timestamps() 
        skipStagesAfterUnstable()
    }
    
    stages {

        stage('Build') {
            steps {
                script {
                    println("Starting codewind-eclipse build ...")
                        
                    def sys_info = sh(script: "uname -a", returnStdout: true).trim()
                    println("System information: ${sys_info}")
                    println("JAVE_HOME: ${JAVA_HOME}")
                    
                    sh '''
                        java -version
                        which java    
                    '''
                    
                    dir('dev') { sh './gradlew --stacktrace' }
                }
            }
        } 
        
        stage('Deploy') {
            steps {
                sshagent ( ['projects-storage.eclipse.org-bot-ssh']) {
                  println("Deploying codewind-eclipse to downoad area...")
                  
                  sh '''
                  	if [ -z $CHANGE_ID ]; then
    					UPLOAD_DIR="$GIT_BRANCH/$BUILD_ID"
					else
    					UPLOAD_DIR="pr/$CHANGE_ID/$BUILD_ID"
					fi
 
                  	ssh genie.codewind@projects-storage.eclipse.org rm -rf /home/data/httpd/download.eclipse.org/codewind/codewind-eclipse/${UPLOAD_DIR}
                  	ssh genie.codewind@projects-storage.eclipse.org mkdir -p /home/data/httpd/download.eclipse.org/codewind/codewind-eclipse/${UPLOAD_DIR}
                  	scp -r ${WORKSPACE}/dev/ant_build/artifacts/* genie.codewind@projects-storage.eclipse.org:/home/data/httpd/download.eclipse.org/codewind/codewind-eclipse/${UPLOAD_DIR}
                  	
                  	if [[-z $CHANGE_ID] && [“$GIT_BRANCH” == “master”]]; then
                  		ssh genie.codewind@projects-storage.eclipse.org rm -rf /home/data/httpd/download.eclipse.org/codewind/codewind-eclipse/master/latest
                  		ssh genie.codewind@projects-storage.eclipse.org mkdir -p /home/data/httpd/download.eclipse.org/codewind/codewind-eclipse/master/latest
                  		scp -r ${WORKSPACE}/dev/ant_build/artifacts/* genie.codewind@projects-storage.eclipse.org:/home/data/httpd/download.eclipse.org/codewind/codewind-eclipse/master/latest
					fi
                  '''
                }
            }
        }       
    }    
}