def registry = 'https://cditsaif.jfrog.io'

pipeline {
    agent {
        node {
            label 'maven'
            // Set PATH directly in the node block
            env.PATH = "/opt/apache-maven-3.9.4/bin:$PATH"
        }
    }
    
    environment {
        // You can define other environment variables here if needed
    }
    
    stages {
        stage("Build") {
            steps {
                echo "----------- Build Started ----------"
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo "----------- Build Completed ----------"
            }
        }
        
        stage("Test") {
            steps {
                echo "----------- Unit Test Started ----------"
                sh 'mvn surefire-report:report'
                echo "----------- Unit Test Completed ----------"
            }
        }
        
        stage("Jar Publish") {
            steps {
                script {
                    echo '<--------------- Jar Publish Started --------------->'
                    
                    def server = Artifactory.newServer url: "${registry}/artifactory", credentialsId: "Jgrog"
                    def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}"
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "target/*.jar",
                                "target": "libs-release-local/",
                                "flat": "false",
                                "props": "${properties}",
                                "exclusions": ["*.sha1", "*.md5"]
                            }
                        ]
                    }"""
                    
                    def buildInfo = server.upload(uploadSpec)
                    buildInfo.env.collect()
                    server.publishBuildInfo(buildInfo)
                    
                    echo '<--------------- Jar Publish Ended --------------->'  
                }
            }   
        }  
    }
}
