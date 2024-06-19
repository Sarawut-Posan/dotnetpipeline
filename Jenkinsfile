pipeline {
   agent {
       label 'built-in'
   }

   stages {
       stage('Checkout') {
           steps {
               checkout scm
           }
       }

       stage('Restore') {
           steps {
               bat 'dotnet restore dotnetwebapp/dotnetwebapp.csproj'
           }
       }

       stage('Build') {
           steps {
               bat 'dotnet build dotnetwebapp/dotnetwebapp.csproj --configuration Release'
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
           }
       }

       stage('Deploy') {
           steps {
                script {
                  def appPool = 'coreapp'
                  def siteName = 'azuretestproject'
                  def localPath = 'C:\\inetpub\\wwwroot\\${siteName}'

                  // Commands to stop and manage IIS
                  bat "C:\\Windows\\System32\\inetsrv\\appcmd stop apppool \"${appPool}\""
                  bat "C:\\Windows\\System32\\inetsrv\\appcmd delete site \"${siteName}\""
                  bat "C:\\Windows\\System32\\inetsrv\\appcmd add site /name:\"${siteName}\" /physicalPath:\"${localPath}\" /bindings:http/*:80:"
                  bat "C:\\Windows\\System32\\inetsrv\\appcmd set app \"${siteName}/\" /applicationPool:\"${appPool}\""

                  // Copying files locally
                  bat "xcopy /Y \"publish\" \"${localPath}\""

                  // Restarting the application pool
                  bat "C:\\Windows\\System32\\inetsrv\\appcmd start apppool \"${appPool}\""
              }
           }
       }
   }

}
