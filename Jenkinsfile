pipeline {
    agent any
    
    parameters {
        string(name: 'PORT', defaultValue: '5000', description: 'Port number for the website (avoid 8080 as it is used by Jenkins)')
    }
    
    stages {
        stage('Checkout') {
            steps {
                script {
                    try {
                        checkout scm
                        echo 'Code checked out from SCM'
                    } catch (Exception e) {
                        echo "Failed to checkout code: ${e.message}"
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }
        
        stage('Restore') {
            steps {
                script {
                    try {
                        bat 'dotnet restore dotnetwebapp/dotnetwebapp.csproj'
                        echo 'Project dependencies restored'
                    } catch (Exception e) {
                        echo "Failed to restore dependencies: ${e.message}"
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }
        
        stage('Build') {
            steps {
                script {
                    try {
                        bat 'dotnet build dotnetwebapp/dotnetwebapp.csproj --configuration Release'
                        echo 'Project built in Release configuration'
                    } catch (Exception e) {
                        echo "Failed to build project: ${e.message}"
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }
        
        stage('Publish') {
            steps {
                script {
                    try {
                        bat 'dotnet publish dotnetwebapp/dotnetwebapp.csproj --configuration Release --output publish'
                        echo 'Project published to the publish directory'
                    } catch (Exception e) {
                        echo "Failed to publish project: ${e.message}"
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    def appPool = 'AzureTestProject'
                    def siteName = 'AzureTestProject'
                    def localPath = "C:\\inetpub\\wwwroot\\${siteName}"
                    
                    // Function to handle errors
                    def handleError = { String step, Exception e ->
                        echo "Error in ${step}: ${e.message}"
                        currentBuild.result = 'UNSTABLE'
                    }
                    
                    // Ensure the application pool exists and is in the correct state
                    try {
                        bat """
                        @echo off
                        echo Checking application pool "${appPool}"...
                        C:\\Windows\\System32\\inetsrv\\appcmd list apppool "${appPool}" > nul 2>&1
                        if %errorlevel% neq 0 (
                            echo Creating application pool "${appPool}"...
                            C:\\Windows\\System32\\inetsrv\\appcmd add apppool /name:"${appPool}"
                        ) else (
                            echo Application pool "${appPool}" already exists
                        )
                        """
                    } catch (Exception e) {
                        handleError('Application Pool Check', e)
                    }
                    
                    // Remove existing site if it exists
                    try {
                        bat """
                        @echo off
                        echo Checking for existing site "${siteName}"...
                        C:\\Windows\\System32\\inetsrv\\appcmd list site "${siteName}" > nul 2>&1
                        if %errorlevel% equ 0 (
                            echo Deleting existing site "${siteName}"...
                            C:\\Windows\\System32\\inetsrv\\appcmd delete site "${siteName}"
                        ) else (
                            echo Site does not exist, no need to delete
                        )
                        """
                    } catch (Exception e) {
                        handleError('Site Check and Delete', e)
                    }
                    
                    // Ensure the target directory exists and is empty
                    try {
                        bat """
                        @echo off
                        echo Preparing target directory "${localPath}"...
                        if exist "${localPath}" (
                            rmdir /S /Q "${localPath}"
                        )
                        mkdir "${localPath}"
                        """
                    } catch (Exception e) {
                        handleError('Directory Preparation', e)
                    }
                    
                    // Copy files to the server
                    try {
                        bat """
                        @echo off
                        echo Copying files to '${localPath}'...
                        xcopy /Y /E /I "publish\\*" "${localPath}"
                        """
                    } catch (Exception e) {
                        handleError('File Copy', e)
                    }
                    
                    // Create the site with the specified port
                    try {
                        bat """
                        @echo off
                        echo Creating new site "${siteName}" on port ${PORT}...
                        C:\\Windows\\System32\\inetsrv\\appcmd add site /name:"${siteName}" /physicalPath:"${localPath}" /bindings:http/*:${PORT}:
                        """
                    } catch (Exception e) {
                        handleError('Site Creation', e)
                    }
                    
                    // Set the application pool for the site
                    try {
                        bat """
                        @echo off
                        echo Setting application pool for site "${siteName}"...
                        C:\\Windows\\System32\\inetsrv\\appcmd set app "${siteName}/" /applicationPool:"${appPool}"
                        """
                    } catch (Exception e) {
                        handleError('Application Pool Assignment', e)
                    }
                    
                    // Start the application pool
                    try {
                        bat """
                        @echo off
                        echo Starting application pool "${appPool}"...
                        C:\\Windows\\System32\\inetsrv\\appcmd start apppool "${appPool}"
                        """
                    } catch (Exception e) {
                        handleError('Application Pool Start', e)
                    }
                    
                    echo "Deployment steps completed"
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        unstable {
            echo 'Pipeline completed with issues. Please check the logs for details.'
        }
        failure {
            echo 'Pipeline execution failed. Please check the logs for details.'
        }
    }
}
