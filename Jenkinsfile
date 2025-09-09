pipeline{
    agent {
        label 'AGENT-1'
    }
    options{
        timeout(time: 30 , unit: 'MINUTES')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }
    environment{
        def appVersion = ''
        nexusUrl= 'nexus.rajasekhar.store:8081'
    }
    stages{
         stage('Read the version'){
            steps{
                script{
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "application version is : $appVersion"
                }
            }
        }
       
        stage('Build'){
            steps{
                sh """
                    zip -r -q frontend-${appVersion}.zip * -x Jenkinsfile -x frontend-${appVersion}.zip
                    ls -ltr
                """
            }
        }
        stage('Nexus Artifact Upload'){
            steps{
                script{
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: "${nexusUrl}",
                        groupId: 'com.expense',
                        version: "${appVersion}",
                        repository: "frontend",
                        credentialsId: 'nexus-auth',
                        artifacts: [
                        [artifactId: "frontend",
                            classifier: '',
                            file: "frontend-" + "${appVersion}" + '.zip',
                            type: 'zip']
                        ]
                    )

                }
            }
        }
        stage('Deploy'){
            steps {
                script{
                    build job: 'frontend-deploy', parameters: [
                            string(name: 'appVersion', value: "${appVersion}")
                            ], wait: false 
                }
            }
        }
        
    }
    post{
        always{
            echo 'I will run always'
            deleteDir()
        }
        failure { 
            echo 'I will run when the build failed!'
        }
        success { 
            echo 'I will run when build is success!'
        }
    }

}