pipeline {
    agent { label 'principal' }
    environment {
            WorkSpaceMPipeline = "$WORKSPACE"
            ServerSonar = "http://172.178.98.221:9000/sonarqube/dashboard?branch=main&id="
            SourceEncoding = "UTF-8"
            ServerFortify = "https://xxxx/ssc"
            ServerReportFortify = "${ServerFortify}/api/v1/projectVersions"
            ProjectName = "Angular"
            AppName = "Credibanco"
            BranchName = "main"
            AgentLabel = "Docker"
            
            // Clean
            cleanEnvironment = "1"

            // CodeReview
            disableCodeReview = "1"
            disableSonar = "1"
            disableFortify = "1"

            // Define environment variables for Docker Hub credentials
            registry = "edissonz8809/credibanco" 
            registryCredential = credentials('credential')
            dockerImage = ''

            
    } 
    stages {
        stage ('Variables - Compile Agent Slaves'){
            agent { label "${AgentLabel}" }
            options { skipDefaultCheckout(true) }
            stages { 
                stage('Environtment - Remote'){
                    steps { script { env.remoteDirPath = "$WORKSPACE" } }
                }
            }
        }
        stage ('Variables - Master'){
         steps {
          script {
            echo "clone aplicacion: $WorkSpaceTrigger"
            sh 'ls $WorkSpaceTrigger'
            sh 'git -C $WorkSpaceTrigger branch -r'
            sh 'git -C $WorkSpaceTrigger status'
            
            // Copiar archivos
            def sshHost = getSSHHost('serverDocker')
            def host = [host: sshHost.hostname, user: sshHost.username, password: sshHost.password]
            sshHost = null
            sh("""
            set +x
            sshpass -p "${host.password}" scp -o StrictHostKeyChecking=no ${host.user}@${host.host}:$WorkSpaceTrigger/perceptor/* ${remoteDirPath}/
            set -x
            """)

          }
         }
        }
        stage ("Code Review"){
            when { expression { "${disableCodeReview}" == '1' } }
            options { skipDefaultCheckout(true) }
            stages {
                stage ('Identity Code'){           
                    parallel {
                        stage('SonarQube'){
                            when { expression { "${disableSonar}" == '1' } }
                            steps {  
                                script {       
                                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                                        script {
                                            env.VersionSonarQube = "${ProjectName}-${AppName}"
                                            def scannerHome = tool  'sonarqube';
                                            withSonarQubeEnv("sonarqube") {
                                                sh ("""${tool("sonarqube")}/bin/sonar-scanner \
                                                    -Dsonar.projectKey=${VersionSonarQube} \
                                                    -Dsonar.projectName=${VersionSonarQube} \
                                                    -Dsonar.projectBaseDir=${workSpaceTrigger} \
                                                    -Dsonar.sources=. \
                                                    -Dsonar.sourceEncoding=${SourceEncoding} \
                                                    -Dsonar.java.binaries=. \
                                                    -Dsonar.branch.name=${BranchName} \
                                                    -Dsonar.projectVersion=${BUILD_NUMBER} """
                                                )  
                                            }
                                        }
                                    }
                                } 
                            }
                        }
                        stage('Fortify'){
                            options { timeout(time: 60, unit: "MINUTES") }
                            when { expression { "${disableFortify}" == '1' } }
                            steps { 
                                script{ 
                                    
                                    echo "step for forty analizer"
                                    // Credentials Fortify
                                  //  withCredentials([
                                  //          string(credentialsId: "${CredentialsTokenCIFortify}", variable: 'VarTokenCIFortify'),
                                  //          string(credentialsId: "${CredentialsTokenFortify}", variable: 'VarTokenFortify'),
                                  //          string(credentialsId: "${CredentialsKeyFortify}", variable: 'VarKeyFortify')
                                  //      ]) 
                                  //  {
                                  //      env.VersionFortify = "${buildAnalyzer}${projectName}-${appName}"
                                  //      env.FileFPR = "${WorkSpaceTrigger}/${VersionFortify}${extFileAnalyzer}"
                                    //
                                  //      // Inicial el analisis
                                  //          sh ('${SourceAnalyzer} -b ${VersionFortify} ${WorkSpaceTrigger} -verbose')
                                  //      // Genera el resultado del reporte en fpr
                                  //          sh ('${SourceAnalyzer} -b ${VersionFortify} -scan -f ${FileFPR} -verbose')
                                  //      // Finaliza con limpieza de fortify
                                  //          sh ('${SourceAnalyzer} -b ${VersionFortify} -clean ${WorkSpaceTrigger} -verbose')
                                  //      // Subir reporte
                                  //          sh ('${FortifyClient} -url ${ServerFortify} -authtoken $VarTokenCIFortify uploadFPR -file ${FileFPR}  -project ${projectName} -version "${VersionFortify}"')
                                  //      // Verified Exist Vulnerability
                                  //          sh ('${ExecCheckSSCIssues} $VarTokenFortify $VarKeyFortify ${FileOutSscApiJson} ${FileOutPrettyJson} ${FileOutGrepJson} ${FortifyClient} ${ServerFortify} ${VersionFortify} ${ServerReportFortify} ${projectName} $VarTokenCIFortify ') 
                                  //  }
                                }
                            }
                        }
                    }
                }
            }
        }
        stage('Build Docker Image') {
            agent { label "${AgentLabel}" }
            options { skipDefaultCheckout(true) }
            steps {          
                script {
                    
                    
                    
                    // Construye la imagen de Docker
                    docker.build("${registry}:$BUILD_NUMBER", "-f Dockerfile .")
                    //sh 'docker build -t ${registry}:$BUILD_NUMBER .'     
	                echo 'Build Image Completed'

                   

                    //  Inicia sesión en Docker Hub
                    sh 'echo $registryCredential | docker login -u $registryCredential --password-stdin'               		
	                echo 'Login Completed'                     

                    docker.withRegistry('https://registry.hub.docker.com', registryCredential) {
                        // Sube la imagen a Docker Hub
                        docker.image("${registry}:$BUILD_NUMBER").push()
                    }

                    // Clean Image local
                    sh "docker rmi ${registry}:$BUILD_NUMBER" 

                    sh 'docker logout' 
                }
            }
        }
     
    }
    post {
        failure {
            script {
                 echo "La construcción o la carga de la imagen de Docker ha fallado."
                // Limpieza Ambiente
                    if ( "${cleanEnvironment}" == "1")  {
                        deleteDir()
                        dir("${WorkSpaceMPipeline}") { deleteDir() }    // Limpieza WorkSpace Pipeline
                        dir("${WorkSpaceTrigger}") { deleteDir() }      // Limpieza WorkSpace Trigger
                    } else { echo "Eliminar Carpetas desactivado" }     // Desactivado
            }
        }
        success {
            script {

                echo "La imagen de Docker se ha construido y subido a Docker Hub con éxito."
                // Limpieza Ambiente
                 if ( "${cleanEnvironment}" == "1")  {               
                        deleteDir()
                        dir("${WorkSpaceMPipeline}") { deleteDir() }    // Limpieza WorkSpace Pipeline
                        dir("${WorkSpaceTrigger}") { deleteDir() }      // Limpieza WorkSpace Trigger
                    } else { echo "Eliminar Carpetas desactivado" }     // Desactivado
            }   
        }
        aborted {
            script {
                // Limpieza Ambiente
                    if ( "${cleanEnvironment}" == "1")  {               
                        deleteDir()
                        dir("${WorkSpaceMPipeline}") { deleteDir() }    // Limpieza WorkSpace Pipeline
                        dir("${WorkSpaceTrigger}") { deleteDir() }      // Limpieza WorkSpace Trigger
                    } else { echo "Eliminar Carpetas desactivado" }     // Desactivado
            }   
        }
    }
}

import jenkins.plugins.publish_over_ssh.*

@NonCPS
def getSSHHost(name) {
  def found = null
  Jenkins.instance.getDescriptorByType(BapSshPublisherPlugin.Descriptor.class).each{
    it.hostConfigurations.each{host ->
      if (host.name == name) {
        found = host
      }
    }
  }

  found
}