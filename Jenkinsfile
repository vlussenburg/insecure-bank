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
                echo 'Running BlackDuck'
                unstash 'Source'
                unstash 'warfile'
                sh 'ls $(pwd)'
                synopsys_detect '--blackduck.url="https://bizdevhub.blackducksoftware.com" --blackduck.api.token=YmIzNzVmYmEtMGRkZi00YjMzLWIwYzYtZjVmOWYyNmU5OWJmOmFkMGVlY2NjLTQ1M2UtNDQ2NS04NGYzLTQ4ZTJkZWM3YzM4MA== --detect.policy.check.fail.on.severities="CRITICAL" --detect.project.name=InsecureBank" --detect.project.version.name=1.0.$BUILD_NUMBER'
				archiveArtifacts '**/BlackDuck-Report.json'
            }
        }
        
        stage ('XL Deploy') {
            steps {
                xldCreatePackage artifactsPath: '$(pwd)/target/insecure-bank.war', darPath: '$JOB_NAME-$BUILD_NUMBER.0.dar', manifestPath: '$(pwd)/deployit-manifest.xml'
                xldPublishPackage serverCredentials: 'admin', darPath: '$JOB_NAME-$BUILD_NUMBER.0.dar'
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
