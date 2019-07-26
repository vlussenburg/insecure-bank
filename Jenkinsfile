pipeline {
    agent any
    
    stages {
        stage ('Build-Time SCA & Build') {
            parallel {
                stage('Package Evaluation') {
                    steps {
                        echo 'Running Package Manager SCA'
                        sh 'ls $(pwd)'
                        synopsys_detect '--detect.tools=DETECTOR --detect.project.name=InsecureBank-Packages --detect.project.version.name=branch-tomas'
                    }
                }
                stage('Build Artifact') {
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
            }
        }

        stage ('Post-Build SCA') {
            steps {
                echo 'Running Black Duck FileSystem and Binary Scans'
                unstash 'Source'
                unstash 'warfile'
                sh 'ls $(pwd)'
                synopsys_detect '--detect.project.name=InsecureBank --detect.project.version.name=App-Build-${BUILD_NUMBER} --detect.binary.scan.file.path=./target/insecure-bank.war'
            }
        }

        stage('Docker Image Build') {
            steps {
                unstash 'Source'
                unstash 'warfile'
                sh '''
                    #!/bin/bash
                    docker build -t vlussenburg/insecure-bank-web:1.0.${BUILD_NUMBER} .
                    docker push vlussenburg/insecure-bank-web:1.0.${BUILD_NUMBER}
                    '''
            }
        }

        stage('Container SCA - Base Image Packages') {
            steps  {
                echo 'Scanning Container Base Image Packages'
                synopsys_detect '--detect.tools=DOCKER --detect.project.name=InsecureBank --detect.project.version.name=Container-Build-${BUILD_NUMBER} --detect.docker.image=vlussenburg/insecure-bank-web:1.0.${BUILD_NUMBER}'
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
                xlrCreateRelease releaseTitle: 'Release for $BUILD_TAG', serverCredentials: 'XL Release', startRelease: true, template: 'Samples & Tutorials/Sample Release Template with XL Deploy', variables: [[propertyName: 'packageId', propertyValue: 'Applications/InsecureBank/1.0.$BUILD_NUMBER'], [propertyName: 'ACC environment', propertyValue: 'Environments/dev'], [propertyName: 'QA environment', propertyValue: 'Environments/dev']]
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
