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
                    def port = params.PORT
                    
                    // Validate port number
                    if (port == '8080') {
                        error "Port 8080 is used by Jenkins. Please choose a different port."
                    }
                    
                    // Ensure the application pool exists and is in the correct state
                    bat """
                    C:\\Windows\\System32\\inetsrv\\appcmd list apppool "${appPool}" > nul 2>&1
                    if %errorlevel% neq 0 (
                        C:\\Windows\\System32\\inetsrv\\appcmd add apppool /name:"${appPool}"
                        echo Application pool created
                    ) else (
                        echo Application pool already exists
                    )
                    if %errorlevel% neq 0 exit /b %errorlevel%
                    """
                    
                    // Check application pool state and stop if running
                    bat """
                    C:\\Windows\\System32\\inetsrv\\appcmd list apppool "${appPool}" /text:state | findstr "Started" > nul
                    if %errorlevel% equ 0 (
                        C:\\Windows\\System32\\inetsrv\\appcmd stop apppool "${appPool}" /commit:apphost
                        if %errorlevel% neq 0 (
                            echo Failed to stop application pool
                            exit /b %errorlevel%
                        )
                        echo Application pool stopped
                    ) else (
                        echo Application pool is already stopped
                    )
                    """
                    
                    // Remove existing site if it exists
                    bat """
                    C:\\Windows\\System32\\inetsrv\\appcmd list site "${siteName}" > nul 2>&1
                    if %errorlevel% equ 0 (
                        C:\\Windows\\System32\\inetsrv\\appcmd delete site "${siteName}"
                        if %errorlevel% neq 0 (
                            echo Failed to delete existing site
                            exit /b %errorlevel%
                        )
                        echo Existing site deleted
                    ) else (
                        echo Site does not exist, no need to delete
                    )
                    """
                    
                    // Ensure the target directory exists and is empty
                    bat """
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
                    echo "Deploying files to '${localPath}'..."
                    bat """
                    xcopy /Y /E /I "publish\\*" "${localPath}"
                    if %errorlevel% neq 0 (
                        echo Failed to copy files
                        exit /b %errorlevel%
                    )
                    """
                    echo 'Files copied to the new site directory'
                    
                    // Create the site with the specified port
                    bat """
                    C:\\Windows\\System32\\inetsrv\\appcmd add site /name:"${siteName}" /physicalPath:"${localPath}" /bindings:http/*:${port}:
                    if %errorlevel% neq 0 (
                        echo Failed to create site
                        exit /b %errorlevel%
                    )
                    """
                    echo "New site added on port ${port}"
                    
                    // Set the application pool for the site
                    bat """
                    C:\\Windows\\System32\\inetsrv\\appcmd set app "${siteName}/" /applicationPool:"${appPool}"
                    if %errorlevel% neq 0 (
                        echo Failed to set application pool for the site
                        exit /b %errorlevel%
                    )
                    """
                    echo 'Application set to use new app pool'
                    
                    // Start the application pool
                    bat """
                    C:\\Windows\\System32\\inetsrv\\appcmd start apppool "${appPool}"
                    if %errorlevel% neq 0 (
                        echo Failed to start application pool
                        exit /b %errorlevel%
                    )
                    """
                    echo 'Application pool started successfully'
                    
                    // Ensure web.config is in the correct location
                    bat """
                    if exist "${localPath}\\web.config" (
                        echo web.config found in the correct location
                    ) else (
                        echo ERROR: web.config not found in ${localPath}
                        exit /b 1
                    )
                    """
                }
            }
        }
    }
    post {
        failure {
            echo 'The pipeline failed. Please check the logs for more information.'
        }
        success {
            echo 'The pipeline completed successfully!'
        }
    }
}
