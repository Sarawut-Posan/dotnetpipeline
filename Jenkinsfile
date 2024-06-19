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
