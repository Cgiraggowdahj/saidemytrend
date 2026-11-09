
pipeline {
    agent any

    environment {
        PATH = "/opt/maven/bin:$PATH"
    }

    stages {
        stage("Build") {
            steps {
                echo '------------ build started ------------'
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo '------------ build completed ------------'
            }
        }

        stage("test") {
            steps {
                echo '------------ unit test started ------------'
                sh 'mvn surefire-report:report'
                echo '------------ unit test completed ------------'
            }
        }

        stage('SonarQube analysis') {
            environment {
                scannerHome = tool 'sonar-scanner'
            }

            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        stage("Jar Publish") {
            steps {
                script {
                    echo '<------------ Jar Publish Started ------------>'

                    def server = Artifactory.newServer url: 'artifactory', credentialsId: 'artifact-cred'

                    def properties = "BuildId=${env.BUILD_ID}, commitId=${GIT_COMMIT}"

                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "jarstaging/(*)",
                                "target": "si-release-local/{1}",
                                "flat": "false",
                                "props": "${properties}",
                                "exclusions": ["*.sha1", "*.md5"]
                            }
                        ]
                    }"""

                    def buildInfo = server.upload(uploadSpec)
                    buildInfo.env.collect()
                    server.publishBuildInfo(buildInfo)

                    echo '<------------ Jar Publish Ended ------------>'
                }
            }
        }
    }
}
