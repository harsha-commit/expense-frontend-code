pipeline{
    agent{
        label 'AGENT-1'
    }

    options{
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }

    environment{
        def appVersion = ''
        def nexusUrl = 'nexus.harshadevops.site:8081'
    }

    stages{
        stage('Read the App Version'){
            steps{
                script{
                    def packageJSON = readJSON file: 'package.json'
                    appVersion = packageJSON.version
                }
            }
        }
        stage('Build'){
            steps{
                sh """
                    rm -rf frontend-${appVersion}.zip
                    zip -rq frontend-${appVersion}.zip * -x Jenkinsfile
                """
            }
        }
        stage('Nexus Artifact Upload'){
            steps{
                 nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${nexusUrl}",
                    groupId: 'com.expense',
                    version: "${appVersion}",
                    repository: 'frontend',
                    credentialsId: 'nexus-auth',
                    artifacts: [
                        [
                            artifactId: "frontend",
                            classifier: '',
                            file: 'frontend-' + "${appVersion}" + '.zip',
                            type: 'zip'
                        ]
                    ]
                )
            }
        }
        stage('Trigger Deploy Job'){
            steps{
                build job: 'frontend-cd', parameters: [string(name: 'appVersion', value: "${appVersion}")], wait: false
            }
        }
    }

    post{
        always{
            deleteDir()
        }
    }
}