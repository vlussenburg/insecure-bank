pipeline {
    agent any
    
    stages {
        stage ('Build Application') {
            steps {
                sh './mvnw clean package -DskipTests'
            }
        }

        /*stage ('Post-Build SCA') {
            parallel {
                stage('Black Duck FileSystem and Package Scan') {
                    steps {
                        echo 'Scanning with Black Duck'
                        unstash 'Source'
                        unstash 'warfile'
                        sh 'ls $(pwd)'
                        synopsys_detect '--detect.project.name=InsecureBank --detect.project.version.name=App-Build-1.0.${BUILD_NUMBER} --detect.code.location.name=bois-application'

                    }
                }

                stage('Black Duck Binary Scan') {
                    steps {
                        echo 'Scanning with Black Duck Binary Analysis'
                        unstash 'warfile'
                        sh 'ls $(pwd)'
                        synopsys_detect '--detect.project.name=InsecureBank --detect.project.version.name=App-Build-1.0.${BUILD_NUMBER} --detect.tools=BINARY_SCAN --detect.binary.scan.file.path=./target/insecure-bank.war --detect.code.location.name=bois-warfile'
                    }
                }
            }
        }*/

        stage('Docker Image Build') {
            steps {
                sh 'docker build -t insecure-bank-web:${BUILD_NUMBER} .'
            }
        }

        /*stage('Container SCA - Base Image') {
            steps  {                     docker push vlussenburg/insecure-bank-web:1.0.${BUILD_NUMBER}
                echo 'Scanning Base Image Packages'
                synopsys_detect '--detect.tools=DOCKER --detect.project.name=InsecureBank --detect.project.version.name=App-Build-1.0.${BUILD_NUMBER} --detect.docker.image=vlussenburg/insecure-bank-web:1.0.${BUILD_NUMBER} --detect.code.location.name=bois-tomcat-base-image'
            }
        }*/

        
        stage ('XL Deploy') {
            steps {
                //sh "./xlw apply -v --values=PACKAGE_NAME=Applications/$BUILD_NUMBER --values=IMAGE=vlussenburg/backtrace-webapp:$BUILD_NUMBER --values=APPLICATION_VERSION=$BUILD_NUMBER --xl-release-password ${xlr_password} --xl-deploy-password ${xld_password} -f xebialabs.yaml"
                //xldCreatePackage artifactsPath: './target/', darPath: '$JOB_NAME-$BUILD_NUMBER.0.dar', manifestPath: './deployit-manifest.xml'
                //xldPublishPackage serverCredentials: 'XL Deploy', darPath: '$JOB_NAME-$BUILD_NUMBER.0.dar'
            }
        }
        
        stage ('XL Release') {
            steps {
                //sh './xlw apply -v --values=BUILD_NUMBER=${BUILD_NUMBER} --xl-release-url http://xl-release:5516 --xl-release-password ${xlr_password} --xl-deploy-url http://xl-deploy:4516 --xl-deploy-password ${xld_password} -f start-release.yaml'
                // xlrCreateRelease releaseTitle: 'Release for $BUILD_TAG', serverCredentials: 'XL Release', startRelease: true, template: 'Samples & Tutorials/Sample Release Template with XL Deploy', variables: [[propertyName: 'packageId', propertyValue: 'Applications/InsecureBank/1.0.$BUILD_NUMBER'], [propertyName: 'application', propertyValue: 'InsecureBank'], [propertyName: 'packageVersion', propertyValue: '1.0.$BUILD_NUMBER'], [propertyName: 'ACC environment', propertyValue: 'Environments/dev'], [propertyName: 'QA environment', propertyValue: 'Environments/dev']]
            }
        }
    }
    
    post {
        always {
            deleteDir()
        }
    }
}
