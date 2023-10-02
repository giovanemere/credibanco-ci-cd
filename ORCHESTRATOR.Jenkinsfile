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
            
            // Clean
            cleanEnvironment = "1"

            // CodeReview
            disableCodeReview = "1"
            disableSonar = "1"
            disableFortify = "1"

            // Define environment variables for Docker Hub credentials
            registry = "edissonz8809/credibanco'" 
            registryCredential = credentials('credential')
            dockerImage = ''

            
    } 
    stages {
        stage ('Variables - Master'){
         steps {
          script {
            echo "clone aplicacion: $WorkSpaceTrigger"
            sh 'ls $WorkSpaceTrigger'
            sh 'git -C $WorkSpaceTrigger branch -r'
            sh 'git -C $WorkSpaceTrigger status'
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
            steps {
                dir('$WorkSpaceTrigger/perceptor/') {                
                    script {
                        // login
                        sh 'echo $registryCredential | docker login -u $registryCredential --password-stdin'
                        
                        // Construye la imagen de Docker
                        dockerImage = docker.build registry + ":$BUILD_NUMBER"

                        // Inicia sesión en Docker Hub
                        docker.withRegistry( '', registryCredential ) {
                            dockerImage.push()
                        }

                        // Clean Image local
                        sh "docker rmi $registry:$BUILD_NUMBER" 
                    }
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