pipeline {
    agent { label 'principal' }
    environment {
            WorkSpaceMPipeline = "$WORKSPACE"
            ServerSonar = "http://10.236.19.217:9000/sonarqube/dashboard?branch=release&id="
            SourceEncoding = "UTF-8"
            ServerFortify = "https://xxxx/ssc"
            ServerReportFortify = "${ServerFortify}/api/v1/projectVersions"
            ProjectName = "Angular"
            AppName = "Credibanco"
            
            // Clean
            cleanEnvironment = "1"

            // CodeReview
            disableCodeReview = "1"
            disableSonar = "1"
            disableFortify = "1"
            
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
                                           def scannerHome = tool 'sonarqube';
                    
                                            withSonarQubeEnv("sonarqube") {
                                                sh ("""${tool("sonarqube")}/bin/sonar-scanner \
                                                -Dsonar.projectKey=${projectName}-${appName} \
                                                -Dsonar.projectName=${projectName}-${appName} \
                                                -Dsonar.projectBaseDir=${workSpaceTrigger} \
                                                -Dsonar.sources=. \
                                                -Dsonar.java.binaries=. \
                                                -Dsonar.projectVersion=${BUILD_NUMBER}
                                                """) 
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
        stage ('Deploy - Build Docker'){
            stages {
                stage ('Deploy - Buld Variables'){
                 steps {
                  script {
                        echo "step pipeline CD"
                  }
                 }
                }
            }
        }
     
    }
    post {
        failure {
            script {
                
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