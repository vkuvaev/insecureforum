pipeline {

   agent any


   triggers {
       // cron('H */8 * * *') //regular builds
        pollSCM('* * * * *') //polling for changes, here once a minute
    }

   stages{

       stage('Fetch Code') { // for display purposes
            steps {


                    // Get a code from the server
                    git 'https://github.com/vkuvaev/insecureforum.git'
                    
            }
        }

	
        stage('Build') {
            // Run the maven build
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    script {
                        // Get the Maven tool.
                        // ** NOTE: This 'M3' Maven tool must be configured
                        // **       in the global configuration.
                        def mvnHome
                        mvnHome = tool 'M3'
                        //  bat label: '', script: 'C:\\Program^ Files\\Fortify\\Fortify_SCA_and_Apps_18.20\\plugins\\maven\\mvnwin\\install.bat'

                        if (isUnix()) {
                            sh "'${mvnHome}/bin/mvn' -Drat.skip=true -Dmaven.test.failure.ignore clean package"
                        } else {
                            bat(/"${mvnHome}\bin\mvn" -Drat.skip=true clean package/)
                        }
                    }

                }
            }
        }

        stage('Clean'){
            steps{
              
               //clean fortify project folder
               fortifyClean addJVMOptions: '', buildID: 'insecureforum', logFile: '', maxHeap: ''
               
            }
        }

        stage('Fortify Build Integration'){
            steps{
               // Running build integration phase
               fortifyTranslate addJVMOptions: '', buildID: 'insecureforum', excludeList: '', logFile: '', maxHeap: '', projectScanType: fortifyOther(otherIncludesList: '', otherOptions: 'mvn -Dmaven.test.failure.ignore package')
           }
        }


        stage('Security Analysis'){
            steps{
                // Running analysis phase for the build
                fortifyScan addJVMOptions: '', addOptions: '', buildID: 'insecureforum', customRulepacks: '', logFile: '', maxHeap: '', resultsFile: 'InsecureForum.fpr'
            }
        }

        
        stage('Archive Test Results') {
            steps{
              junit '**/target/surefire-reports/TEST-*.xml'
              archiveArtifacts 'target/*.war'
            }
        }

        stage ('Upload Results to Fortify SSC Server'){
            steps{
                   script { try {
                         fortifyUpload appName: 'InsecureForum', appVersion: '1', failureCriteria: '[fortify priority order]:critical OR [fortify priority order]:major', filterSet: '', pollingInterval: '1', resultsFile: 'InsecureForum.fpr'
                         
                    }
                    catch(Exception e) {
                        echo 'Unstable Detected'
                        error "Security criteria breached"
                    }
                   }    
            }
        }
        
        stage ('Deploy Artifact & continue pipeline'){
              steps {
                  echo 'Continuing successfully'
          // something
              }
        }
   }
   
}

