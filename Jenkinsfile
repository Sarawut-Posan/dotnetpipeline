pipeline {
    agent {
        label 'built-in'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo 'Code checked out from SCM'
            }
        }

        stage('Restore') {
            steps {
                bat 'dotnet restore dotnetwebapp/dotnetwebapp.csproj'
                echo 'Project dependencies restored'
            }
        }

        stage('Build') {
            steps {
                bat 'dotnet build dotnetwebapp/dotnetwebapp.csproj --configuration Release'
                echo 'Project built in Release configuration'
            }
        }

        stage('Test') {
            steps {
                echo 'Skipping tests'
            }
        }

        stage('Publish') {
            steps {
                bat 'dotnet publish dotnetwebapp/dotnetwebapp.csproj --configuration Release --output publish'
                echo 'Project published to the publish directory'
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def appPool = 'coreapp'
                    def siteName = 'azuretestproject'
                    def localPath = "C:\\inetpub\\wwwroot\\${siteName}\\"

                    // Check if the app pool is running and stop it if it is
                    bat script: "C:\\Windows\\System32\\inetsrv\\appcmd list apppool \"${appPool}\" | find \"State: Started\" && C:\\Windows\\System32\\inetsrv\\appcmd stop apppool \"${appPool}\" || echo App pool already stopped", returnStatus: true

                    // Delete existing site
                    bat "C:\\Windows\\System32\\inetsrv\\appcmd delete site \"${siteName}\""
                    echo 'Site deleted'

                    // Add new site
                    bat "C:\\Windows\\System32\\inetsrv\\appcmd add site /name:\"${siteName}\" /physicalPath:\"${localPath}\" /bindings:http/*:80:"
                    echo 'New site added'

                    // Set application to use the new app pool
                    bat "C:\\Windows\\System32\\inetsrv\\appcmd set app \"${siteName}/\" /applicationPool:\"${appPool}\""
                    echo 'Application set to use new app pool'

                    // Copying files locally
                    bat "xcopy /Y /I \"publish\" \"${localPath}\""
                    echo 'Files copied to the new site directory'

                    // Start the application pool
                    bat "C:\\Windows\\System32\\inetsrv\\appcmd start apppool \"${appPool}\""
                    echo 'Application pool started'
                }
            }
        }
    }
}
