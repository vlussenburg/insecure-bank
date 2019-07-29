pipeline {
    agent any
    
    stages {
        stage ('Build Application') {
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

        stage ('Post-Build SCA') {
            parallel {
                stage('Black Duck FileSystem and Package Scan') {
                    steps {
                        echo 'Scanning with Black Duck'
                        unstash 'Source'
                        unstash 'warfile'
                        sh 'ls $(pwd)'
                        synopsys_detect '--detect.project.name=InsecureBank --detect.project.version.name=App-Build-1.0.${BUILD_NUMBER} --detect.code.location.name=application'

                    }
                }

                stage('Black Duck Binary Scan') {
                    steps {
                        echo 'Scanning with Black Duck Binary Analysis'
                        unstash 'warfile'
                        sh 'ls $(pwd)'
                        synopsys_detect '--detect.project.name=InsecureBank --detect.project.version.name=App-Build-1.0.${BUILD_NUMBER} --detect.tools=BINARY_SCAN --detect.binary.scan.file.path=./target/insecure-bank.war --detect.code.location.name=warfile'
                    }
                }
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

        stage('Container SCA - Base Image') {
            steps  {
                echo 'Scanning Base Image Packages'
                synopsys_detect '--detect.tools=DOCKER --detect.project.name=InsecureBank --detect.project.version.name=App-Build-1.0.${BUILD_NUMBER} --detect.docker.image=vlussenburg/insecure-bank-web:1.0.${BUILD_NUMBER} --detect.code.location.name=tomcat-base-image'
            }
        }

        
        stage ('XL Deploy') {
            steps {
                xldCreatePackage artifactsPath: './target/', darPath: '$JOB_NAME-$BUILD_NUMBER.0.dar', manifestPath: './deployit-manifest.xml'
                xldPublishPackage serverCredentials: 'XL Deploy', darPath: '$JOB_NAME-$BUILD_NUMBER.0.dar'
            }
        }
        
        stage ('XL Release') {
            steps {
                xlrCreateRelease releaseTitle: 'Release for $BUILD_TAG', serverCredentials: 'XL Release', startRelease: true, template: 'Samples & Tutorials/Sample Release Template with XL Deploy', variables: [[propertyName: 'packageId', propertyValue: 'Applications/InsecureBank/1.0.$BUILD_NUMBER'], [propertyName: 'application', propertyValue: 'InsecureBank'], [propertyName: 'packageVersion', propertyValue: '1.0.$BUILD_NUMBER'], [propertyName: 'ACC environment', propertyValue: 'Environments/dev'], [propertyName: 'QA environment', propertyValue: 'Environments/dev']]
            }
        }
    }
    
    post {
        always {
            deleteDir()
        }
    }
}
