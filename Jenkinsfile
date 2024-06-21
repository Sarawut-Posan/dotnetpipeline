pipeline {
    agent {
        label 'built-in'
    }
    
    parameters {
        string(name: 'PORT', defaultValue: '5000', description: 'Port number for the website (avoid 8080 as it is used by Jenkins)')
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
                // Add your test commands here if needed
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
                    def appPool = 'AzureTestProject'
                    def siteName = 'AzureTestProject'
                    def localPath = "C:\\inetpub\\wwwroot\\${siteName}"
                    
                    // Ensure the application pool exists and is in the correct state
                    bat """
                    @echo off
                    echo Checking application pool "${appPool}"...
                    C:\\Windows\\System32\\inetsrv\\appcmd list apppool "${appPool}" > nul 2>&1
                    if %errorlevel% neq 0 (
                        echo Creating application pool "${appPool}"...
                        C:\\Windows\\System32\\inetsrv\\appcmd add apppool /name:"${appPool}"
                        if %errorlevel% neq 0 (
                            echo Failed to create application pool
                            exit /b %errorlevel%
                        )
                    ) else (
                        echo Application pool "${appPool}" already exists
                    )
                    """
                    
                    // Remove existing site if it exists
                    bat """
                    @echo off
                    echo Checking for existing site "${siteName}"...
                    C:\\Windows\\System32\\inetsrv\\appcmd list site "${siteName}" > nul 2>&1
                    if %errorlevel% equ 0 (
                        echo Deleting existing site "${siteName}"...
                        C:\\Windows\\System32\\inetsrv\\appcmd delete site "${siteName}"
                        if %errorlevel% neq 0 (
                            echo Failed to delete existing site
                            exit /b %errorlevel%
                        )
                    ) else (
                        echo Site does not exist, no need to delete
                    )
                    """
                    
                    // Ensure the target directory exists and is empty
                    bat """
                    @echo off
                    echo Preparing target directory "${localPath}"...
                    if exist "${localPath}" (
                        rmdir /S /Q "${localPath}"
                        if %errorlevel% neq 0 (
                            echo Failed to remove existing directory
                            exit /b %errorlevel%
                        )
                    )
                    mkdir "${localPath}"
                    if %errorlevel% neq 0 (
                        echo Failed to create directory
                        exit /b %errorlevel%
                    )
                    """
                    
                    // Copy files to the server
                    bat """
                    @echo off
                    echo Copying files to '${localPath}'...
                    xcopy /Y /E /I "publish\\*" "${localPath}"
                    if %errorlevel% neq 0 (
                        echo Failed to copy files
                        exit /b %errorlevel%
                    )
                    """
                    
                    // Create the site with the specified port
                    bat """
                    @echo off
                    echo Creating new site "${siteName}" on port ${PORT}...
                    C:\\Windows\\System32\\inetsrv\\appcmd add site /name:"${siteName}" /physicalPath:"${localPath}" /bindings:http/*:${PORT}:
                    if %errorlevel% neq 0 (
                        echo Failed to create site
                        exit /b %errorlevel%
                    )
                    """
                    
                    // Set the application pool for the site
                    bat """
                    @echo off
                    echo Setting application pool for site "${siteName}"...
                    C:\\Windows\\System32\\inetsrv\\appcmd set app "${siteName}/" /applicationPool:"${appPool}"
                    if %errorlevel% neq 0 (
                        echo Failed to set application pool for the site
                        exit /b %errorlevel%
                    )
                    """
                    
                    // Start the application pool
                    bat """
                    @echo off
                    echo Starting application pool "${appPool}"...
                    C:\\Windows\\System32\\inetsrv\\appcmd start apppool "${appPool}"
                    if %errorlevel% neq 0 (
                        echo Failed to start application pool
                        exit /b %errorlevel%
                    }
                    """
                    
                    echo "Deployment completed successfully!"
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline execution failed. Please check the logs for details.'
        }
    }
}
