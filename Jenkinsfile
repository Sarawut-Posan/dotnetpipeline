pipeline {
    agent {
        label 'windows'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Restore') {
            steps {
                bat 'dotnet restore'
            }
        }

        stage('Build') {
            steps {
                bat 'dotnet build --configuration Release'
            }
        }

        stage('Test') {
            steps {
                bat 'dotnet test --logger trx --collect "Code coverage"'
            }
            post {
                always {
                    mstest testResultsFile: '**/*.trx'
                }
            }
        }

        stage('Publish') {
            steps {
                bat 'dotnet publish --configuration Release --output publish'
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def azureVM = '20.2.250.65'
                    def siteName = 'azuretestproject'
                    def appPool = 'coreapp'

                    // Stop the application pool
                    bat "C:\\Windows\\System32\\inetsrv\\appcmd stop apppool \"${appPool}\""

                    // Delete the existing site
                    bat "C:\\Windows\\System32\\inetsrv\\appcmd delete site \"${siteName}\""

                    // Create a new site
                    bat "C:\\Windows\\System32\\inetsrv\\appcmd add site /name:\"${siteName}\" /physicalPath:\"C:\\inetpub\\wwwroot\\${siteName}\" /bindings:http/*:80:"

                    // Set the application pool for the site
                    bat "C:\\Windows\\System32\\inetsrv\\appcmd set app \"${siteName}/\" /applicationPool:\"${appPool}\""

                    // Deploy the published files to the Azure VM
                    bat "xcopy /Y \"publish\" \"\\\\${azureVM}\\C\$\\inetpub\\wwwroot\\${siteName}\""

                    // Start the application pool
                    bat "C:\\Windows\\System32\\inetsrv\\appcmd start apppool \"${appPool}\""
                }
            }
        }
    }

    post {
        success {
            emailext (
                subject: "Deployment Succeeded",
                body: "The application was successfully deployed.",
                to: "sarwut.p@kkumail.com"
            )
        }
        failure {
            emailext (
                subject: "Deployment Failed",
                body: "The application deployment failed. Please check the Jenkins logs.",
                to: "sarwut.p@kkumail.com"
            )
        }
    }
}
