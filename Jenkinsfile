pipeline {
    agent none
    stages {
        stage('Get Version') {
            agent {
                label 'build && java8 && centos6'
                }
               steps {
               checkout scm
                sh '''
                    #!/bin/bash +x
                     ./gradlew printVersion
                      ./gradlew | grep VERSION | sed  "s/VERSION=//" > pipeline.properties
                       cat pipeline.properties
                      '''
                script{
                       VERSION=readFile('pipeline.properties')
                       }
                      }

        }
        stage('Build Distributables') {
            parallel {

                stage('Build Windows') {
                    agent {
                        label 'build && java8 && windows10 && msbuild'
                    }
                    steps {
                        checkout scm
                        bat  "gradlew.bat clean publishDistributablePublicationToMavenRepository -Pversion=${VERSION}"
                }
                }
                stage('Build Linux') {
                    agent {
                        label 'build && java8 && centos6'
                    }
                    steps {
                        checkout scm
                        sh  """ #!/bin/bash +x
                            ./gradlew clean publishDistributablePublicationToMavenRepository -Pversion=${VERSION}
                            """
                    }
                }
                stage('Build Darwin') {
                      agent {
                           label 'build && java8 && osx-10.12'
                             }
                             steps {
                                    checkout scm
                                    sh """ #!/bin/bash +x
                                     ./gradlew clean publishDistributablePublicationToMavenRepository -Pversion=${VERSION}
                                     """
                                    }
                                }

        }
        }
        stage('Build Jar') {
            agent {
                label 'build && java8 && centos6'
        }
        steps {
            checkout scm
            sh  " #!/bin/bash +x; ./gradlew clean publishAllPlatformsJarPublicationToMavenRepository -Pversion=${VERSION}"
        }
        }

        stage("Promote to RC") {
                    agent {
                        label 'build && linux && gradle'
                    }
                    steps {
                    dir ('promotion'){
                    git credentialsId: 'f5d48fb8-f02a-4b63-afbf-ce46c50d9363', url: 'https://stash.caplin.com/scm/releng/promotionscripts.git'
                    sh """ #!/bin/bash +x
                      ./gradlew PromoteToCaplinRC -Dversion=${VERSION} -DconfigFile=Platform/JavaDev/NanoTime.json -Pbranch=master
                      """
                    }}
                }
    }
}