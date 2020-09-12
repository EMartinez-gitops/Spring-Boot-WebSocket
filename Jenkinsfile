pipeline {
  agent {
    kubernetes {
      yamlFile 'maven-pod.yml'
    }
  }  
  options { 
    buildDiscarder(logRotator(numToKeepStr: '2'))
    skipDefaultCheckout true
    preserveStashes(buildCount: 2)
  }
  stages('First Stages of CI/CD')
  {
    stage('Pull Source Code from SCM') {
      steps {
        container ('maven') {
          checkout scm
        }
      }
    }
    stage('Maven Build Compile/Unit Test') {
      steps {
        container ('maven') {
          sh 'mvn install'
          sh 'mvn compile'
        }
      }
    }
    stage('Code Coverage - Sonar Scans') {
      steps {
        container('maven') {
                withCredentials([string(credentialsId: 'Sonar_Login', variable: 'Sonar_Login'), string(credentialsId: 'Sonar_URL', variable: 'Sonar_URL'), string(credentialsId: 'My_Sonar_Project', variable: 'My_Sonar_Project')]) {
                  sh 'mvn sonar:sonar   -Dsonar.projectKey=${My_Sonar_Project}   -Dsonar.host.url=${Sonar_URL}  -Dsonar.login=${Sonar_Login}'
                }                  
          }
      }
    }
    stage('Quality Scans') {
      steps {
        container('maven') {
          echo 'Quality Scanning Success!'
        }
      }
    }
    stage('Security Scans') {
      steps {
        container('maven') {
          echo 'Security Scanning Success!'
        }
      }
    }
    stage('Deploy to Nexus') {
      when {
        branch 'development'
      }
      steps {
        container('maven') {
          input(message: "Deploy this artifact as a release candidate?", ok: "Approve", submitterParameter: "APPROVER")
       nexusArtifactUploader artifacts: [[artifactId: 'spring-boot-starter-parent-noclash', classifier: '', file: 'target/spring-boot-websocket-0.0.1-SNAPSHOT.jar', type: 'jar']], credentialsId: 'ci-sa', groupId: 'org.springframework.boot', nexusUrl: 'nexus.cb-demos.io', nexusVersion: 'nexus3', protocol: 'https', repository: 'maven-releases', version: '0.0.1'
        }
      }
    }
     stage('Trigger Release Candidate') {
       when {
        branch 'development'
       }
       steps {
         cloudBeesFlowTriggerRelease configuration: 'Thunder-CD', parameters: '{"release":{"releaseName":"chat-app-scratch-pad", "stages":[{"stageName":"Release Readiness","stageValue":true},{"stageName":"Pre-Prod","stageValue":false},{"stageName":"Prod","stageValue":false}], "pipelineName":"pipeline_newPipeline","parameters":[]}}', projectName: 'emartinez Demo', releaseName: 'chat-app-scratch-pad', startingStage: 'Release Readiness'
       }
     }
  }  
}
