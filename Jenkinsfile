pipeline {
  agent any

  environment {
    scannerHome = 'sonar_scanner_dotnet'
    username = 'ganeshchandra'
    appName = 'NAGPProject'
  }

  options {
    timestamps()

    timeout(time: 1, unit: 'HOURS')

    buildDiscarder(logRotator(
      numToKeepStr: '3',
      daysToKeepStr: '5'
    ))
  }

  stages {

    stage("Nuget Restore") {
      steps {
        echo "Nuget restore started for - ${BRANCH_NAME} branch"
        bat "dotnet restore"
      }
    }

    stage("Start Sonarqube Analysis") {
      when {
        branch "master"
      }
      steps {
        echo "Start Sonarqube analysis step started"
        withSonarQubeEnv('Test_Sonar') {
          bat "dotnet ${scannerHome}\\SonarScanner.MSBuild.dll begin /k:sonar-${userName} /n:sonar-${userName} /v:1.0"
        }
      }
    }

    stage("Code Build") {
      steps {
        echo "Code build step started"
        echo "Clean Old Build"
        bat "dotnet clean"

        // Buildthe project and its dependencies
        echo "Code Build"
        bat 'dotnet build -c Release -o "ProductManagementApi/app/build"'
        bat 'dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=opencover -l:trx;LogFileName=ProductManagementApi.xml'
      }
    }

    stage("Test Case Execution") {
      when {
        branch "master"
      }
      steps {
        echo "Test Case Execution started"
      }
    }
    
    stage("Stop Sonarqube Analysis") {
      when {
        branch "master"
      }

      steps {
        echo "Stop sonarqube analysis step started"
        withSonarQubeEnv('Test_Sonar') {
          bat "dotnet ${scannerHome}\\SonarScanner.MSBuild.dll end"
        }
      }
    }

    stage("Release Artifact") {
      when {
        branch "develop"
      }

      steps {
        echo "Release artifact step started"
        bat "dotnet publish -c Release -o ${appName}/app/${userName}"
      }
    }

    // stage('Kubernetes Deployment') {
    // steps{
    // echo "Kubernetes Deployment step started"
    // bat "kubectl apply -f deployment.yaml"
    // }
    //}
  }
}
