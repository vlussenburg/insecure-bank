pipeline {
    agent any
    
    stages {
        stage ('CodeBuild') {
            agent {
                docker {
                    image 'maven:3.5-jdk-8-alpine'
                }
            }
            
            steps {
                sh 'mvn clean package -DskipTests'
                stash includes:'**/target/insecure-bank.war', name:'warfile'
                stash includes: '**/**', name: 'Source'
            }
        }
        
        stage ('Black Duck SCA') {
            steps {
                echo 'Running Synopsys Detect SCA'
                unstash 'Source'
                unstash 'warfile'
                sh 'ls $(pwd)'
                synopsys_detect '--detect.tools=DETECTOR --detect.project.name=InsecureBank --detect.project.version.name=Jenkins-CI'
            }
        }
        
        stage ('XL Deploy') {
            steps {
                xldCreatePackage artifactsPath: './target/', darPath: '$JOB_NAME-$BUILD_NUMBER.0.dar', manifestPath: './deployit-manifest.xml'
                xldPublishPackage serverCredentials: 'XL Deploy', darPath: '$JOB_NAME-$BUILD_NUMBER.0.dar'
                /*xldDeploy environmentId: 'Environments/dev', packageId: 'Applications/InsecureBank/1.0.$BUILD_NUMBER', serverCredentials: 'XL Deploy'*/
            }
        }
        
        stage ('XL Release') {
            steps {
                xlrCreateRelease releaseTitle: 'Release for $BUILD_TAG', serverCredentials: 'XL Release', startRelease: true, template: 'Samples & Tutorials/Sample Release Template with XL Deploy'
            }
        }
        
        
        /* stage ('Test-time Deploy') {
            steps {
                print 'Deploy to Tomcat'
                unstash 'warfile'
                sh 'sudo /opt/deployment/tomcat/apache-tomcat-8.5.28/bin/shutdown.sh'
                sh 'sudo cp $(pwd)/target/insecure-bank.war /opt/deployment/tomcat/apache-tomcat-8.5.28/webapps'
              
                sh 'sudo /opt/deployment/tomcat/apache-tomcat-8.5.28/bin/startup.sh'
                sleep 45
                print 'Application Deployed to test'
            }
        }*/
    }
    
    post {
        always {
            deleteDir()
        }
    }
}
