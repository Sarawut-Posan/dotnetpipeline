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
                   def serverName = '20.2.250.65'
                   def siteName = 'azuretestproject'
                   def appPool = 'coreapp'

                   bat "net use \\\\${serverName}\\C\$ /user:sarawutadmin @Pond36802a55"

                   bat "C:\\Windows\\System32\\inetsrv\\appcmd stop apppool \"${appPool}\""
                   bat "C:\\Windows\\System32\\inetsrv\\appcmd delete site \"${siteName}\""
                   bat "C:\\Windows\\System32\\inetsrv\\appcmd add site /name:\"${siteName}\" /physicalPath:\"C:\\inetpub\\wwwroot\\${siteName}\" /bindings:http/*:80:"
                   bat "C:\\Windows\\System32\\inetsrv\\appcmd set app \"${siteName}/\" /applicationPool:\"${appPool}\""
                   bat "xcopy /Y \"publish\" \"\\\\${serverName}\\C\$\\inetpub\\wwwroot\\${siteName}\""
                   bat "C:\\Windows\\System32\\inetsrv\\appcmd start apppool \"${appPool}\""
               }
           }
       }
   }

}
