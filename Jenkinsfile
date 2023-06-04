pipeline {
    agent any
      stages {
        stage('Build') {
            steps {
              sh 'mvn -B package'
            }
        }   

      stage("Publish to Nexus Repository Manager") {
        steps {
          script {
            pom = readMavenPom file: "pom.xml";
            filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
            echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
            artifactPath = filesByGlob[0].path;
            artifactExists = fileExists artifactPath;
            if(artifactExists) {
              echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
              nexusArtifactUploader(
                nexusVersion: "nexus3",
                protocol: "http",
                nexusUrl: "localhost:8081",
                groupId: pom.groupId,
                version: pom.version,
                repository: "RepositorioEjerM3",
                credentialsId: "NexusConexion",
                artifacts: [
                  [artifactId: pom.artifactId,
                  classifier: '',
                  file: artifactPath,
                  type: pom.packaging]
                ]
              );
            } else {
              error "*** File: ${artifactPath}, could not be found";
            }
          }
        }
        
      }
      stage('SonarQube analysis') {
          environment {
                SCANNER_HOME = tool 'SonarQubeConection'
            }
            steps {
              withSonarQubeEnv(credentialsId: 'SecretIdToken', installationName: 'SonarConexion') {
              sh '''$SCANNER_HOME/bin/sonar-scanner \
              -Dsonar.projectKey=projectKey \
              -Dsonar.projectName=projectName \
              -Dsonar.sources=src/ \
              -Dsonar.java.binaries=target/classes/ \
              -Dsonar.exclusions=src/test/java/****/*.java \
              -Dsonar.projectVersion=${BUILD_NUMBER}-${GIT_COMMIT_SHORT}'''
             }
            }
        
          }

    }
    post {
        failure {
            script {
                // Envía el mensaje de error a Slack
                slackSend  message: "¡Atención! Hubo un error durante la compilación.", color: 'danger',  teamDomain: 'sustantiva-sede', tokenCredentialId: 'Token-slack2', username: 'Juan Pablo Grover Pinto' // 
            }
        }

        success {
            script {
                // Envía el mensaje de felicitaciones a Slack
                slackSend  message: '¡Felicidades! No hubo errores durante la compilación.', color: 'good',  teamDomain: 'sustantiva-sede', tokenCredentialId: 'Token-slack2', username: 'Juan Pablo Grover Pinto' //
            }
        }
    }
}

En este ejemplo, el bloque catchError captura cualquier error que ocurra durante la compilación. Si no hay errores, no se ejecuta ninguna acción adicional en ese bloque. En la sección post, se utiliza un bloque failure para enviar un mensaje de error a Slack si la etapa falla, y un bloque success para enviar un mensaje de felicitaciones si la etapa tiene éxito.

Asegúrate de tener instalado el plugin de Slack en tu instancia de Jenkins y de haber configurado adecuadamente la integración con Slack en la configuración global de Jenkins.

Recuerda adaptar el código a tus necesidades específicas, como el nombre del canal de Slack y los mensajes que deseas enviar. También puedes agregar más etapas y lógica a tu Pipeline según tus requisitos.


} 

  def custom_msg()
  {
    def JENKINS_URL= "localhost:8080"
    def JOB_NAME = env.JOB_NAME
    def BUILD_ID= env.BUILD_ID
    def JENKINS_LOG= " FAILED: Job [${env.JOB_NAME}] Logs path: ${JENKINS_URL}/job/${JOB_NAME}/${BUILD_ID}/consoleText"
    return JENKINS_LOG
  }
