pipeline {
    agent any
        environment {
            TOKENSONAR = 'squ_27920b448c22c6ca53d7dbae57a0b707c5113d7e'
        }
        stages {
        stage('Initialize') {
            steps {
                echo 'Esta es el inicio'
            }
        }
        stage('Sonarcan') {
            steps {
                sh '''/var/jenkins_home/sonar-scanner/bin/sonar-scanner \
                -Dsonar.projectName=prueba-de-todo \
                -Dsonar.projectKey=prueba-de-todo \
                -Dsonar.projectVersion=1 \
                -Dsonar.sources=src/main/java/ \
                -Dsonar.language=java \
                -Dsonar.java.binaries=./target/classes \
                -Dsonar.host.url=http://192.168.26.129:9000 \
                -Dsonar.login=${TOKENSONAR}'''
            }
        }
        stage('Build') {
            steps {
                sh 'mvn -B package'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn clean verify'
            }
        }
        stage('Publish to Nexus Repository Manager') {
            steps {
                script {
                    pom = readMavenPom file: 'pom.xml'
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path
                    artifactExists = fileExists artifactPath
                    if (artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}"
                        nexusArtifactUploader(
                            nexusVersion: 'nexus3',
                            protocol: 'http',
                            nexusUrl: '192.168.26.129:8081',
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: 'm3pj6',
                            credentialsId: 'admin',
                            artifacts: [
                                [artifactId: pom.artifactId,
                                        classifier: '',
                                        file: artifactPath,
                                        type: pom.packaging]
                            ]
                        )
                    } else {
                        error "*** File: ${artifactPath}, could not be found"
                    }
                }
            }
        }
        }
    post {
        always {
            script {
                echo 'Probando el posts'
                slackSend(channel: '@U05690FEL7P', message: " Proyecto Creado Perfectamente *${currentBuild.currentResult}:* build ${env.BUILD_NUMBER}, ${env.JOB_NAME}", color: '#00FF04')
            }
        }
    }
}
