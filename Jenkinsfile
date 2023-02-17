def registry = 'https://devopshunger.jfrog.io/'
pipeline{
    agent {
        node {
            label "slave_java"
        }
    }
    environment {
        PATH = "/opt/maven/bin:$PATH"
    }
    stages {
        stage('build') {
            steps{
                echo "------------ build started ---------"
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo "------------ build completed ---------"
        }
      }

        stage('Unit Test') {
            steps {
                echo '<--------------- Unit Testing started  --------------->'
                sh 'mvn surefire-report:report'
                echo '<------------- Unit Testing stopped  --------------->'
            }
        }

       stage ("Sonar Analysis") {
            environment {
               scannerHome = tool 'valaxy-sonarscanner'
            }
            steps {
                echo '<--------------- Sonar Analysis started  --------------->'
                withSonarQubeEnv('valaxy-sonarqube-server') {    
                    sh "${scannerHome}/bin/sonar-scanner"
                echo '<--------------- Sonar Analysis stopped  --------------->'
                }         
            }   
        }
        stage("Jar Publish") {
        steps {
            script {
                    echo '<--------------- Jar Publish Started --------------->'
                     def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"Jfrog"
                     def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                     def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "jarstaging/(*)",
                              "target": "devopsintern-libs-release-local/{1}",
                              "flat": "false",
                              "props" : "${properties}",
                              "exclusions": [ "*.sha1", "*.md5"]
                            }
                         ]
                     }"""
                     def buildInfo = server.upload(uploadSpec)
                     buildInfo.env.collect()
                     server.publishBuildInfo(buildInfo)
                     echo '<--------------- Jar Publish Ended --------------->'  
            
            }
            def imageName = 'devopshunger.jfrog.io/devopsintern-docker-local/ttrend'
   def version   = '2.0.2'
    stage(" Docker Build ") {
      steps {
        script {
           echo '<--------------- Docker Build Started --------------->'
           app = docker.build(imageName+":"+version)
           echo '<--------------- Docker Build Ends --------------->'
        }
      }
    }

            stage (" Docker Publish "){
        steps {
            script {
               echo '<--------------- Docker Publish Started --------------->'  
                docker.withRegistry(registry, 'jfrogforjenkins'){
                    app.push()
                }    
               echo '<--------------- Docker Publish Ended --------------->'  
            }
        }
    }
        }   
    }   
    }
    }
