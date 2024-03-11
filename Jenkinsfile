pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', credentialsId: 'git-threepoints-github', url: 'https://github.com/Marcko9/threepoints_devops_webserver.git'
            }
        }
        
        stage('Parallels'){
            parallel {
                stage('Echo SAST') {
                    steps{
                        script{
                            echo 'Pruebas de SAST'
                        }
                    }        
                }
                stage('Imprimir Env') {
                    steps {
                        echo "WORKSPACE: ${env.WORKSPACE}"
                    }
                }
            }
        }
        
        stage('Pruebas de SAST'){
            environment {
                scannerHome = tool 'sonar-scanner' 
            }
            steps{
                script{
                    echo 'Pruebas de SAST'
                    withSonarQubeEnv(credentialsId: 'sonar-token-new'){
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=sonarqube \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=http://localhost:9000 \
                            -Dsonar.token=sqa_ef6b785d0c53d831a09f0670e3bc76c390659a3e
                        """
                    }
                    timeout(time:1, unit: 'HOURS'){
                        waitForQualityGate abortPipeline: false
                        
                        def qg= waitForQualityGate()
                        if (qg.status!= 'OK'){
                          error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }
        
        stage('Configurar archivo'){
            steps{
                script {
                    userVar = null
                    passVar = null
                    withCredentials([usernamePassword(credentialsId: 'Credentials_Threepoints', passwordVariable: 'password', usernameVariable: 'username')]) {
                        userVar = username
                        passVar = password
                    }
                    echo "Username: ${userVar}"
                    echo "Password: ${passVar}"
                    
                    file './credentials.ini'
                    writeFile file: './credentials.ini', text: "[credentials]\nUser=${userVar}\nPassword=${passVar}"
                    
                    archiveArtifacts artifacts: 'credentials.ini', fingerprint: true
                }
            }
        }
        
        stage('Build') {
            steps {
                script{
                    sh 'docker build -t devops_ws .'
                }
            }
        }

        stage('Despliegue de servidor'){
            steps{
                script{
                    sh 'docker stop devops_ws || true'
                    sh 'docker run -d -p 8090:8090 --name devops devops_ws'
                }
            }
        }
    }
}
