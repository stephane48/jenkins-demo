pipeline {
    agent any
    tools {
        nodejs "node"
    }
    stages {
        // stage('Cleanup workspace') {
        //     steps {
        //         cleanWs()
        //     }
        // }
        stage('increment version') {
            steps {
                script {
                    // enter app directory, because that's where package.json is located
                    dir("app") { 
                        sh "ls -la"
                        sh "node --version"
                        sh "npm --version"
                        // update application version in the package.json file with one of these release types: patch, minor or major
                        // this will commit the version update
                        sh "npm version minor"

                        // read the updated version from the package.json file
                        def packageJson = readJSON file: 'package.json'
                        def version = packageJson.version

                        // set the new version as part of IMAGE_NAME
                        env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                    }

                    // alternative solution without Pipeline Utility Steps plugin: 
                    // def version = sh (returnStdout: true, script: "grep 'version' package.json | cut -d '\"' -f4 | tr '\\n' '\\0'")
                    // env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
        }
        stage('Run tests') {
            steps {
               script {
                    // enter app directory, because that's where package.json and tests are located
                    dir("app") { 
                        // install all dependencies needed for running tests
                        sh "npm install"
                        sh "npm run test"
                    } 
               }
            }
        }
        stage('Build and Push docker image') {
            steps {
                sh "docker --version"
                sh "which docker"
                withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]){
                    sh "docker build -t stephane48/myapp:${IMAGE_NAME} ."
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                    sh "docker push stephane48/myapp:${IMAGE_NAME}"
                }
            }
        }
        stage('commit version update') {
            steps {
                script {
                    // withCredentials([usernamePassword(credentialsId: 'github-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    //     // git config here for the first time run
                    //     sh 'git config --global user.email "stephanekamdem09@gmail.com"'
                    //     sh 'git config --global user.name "stephane48"'
                    //     sh 'git remote set-url origin https://$USER:$PASS@github.com/stephane48/jenkins-demo.git'
                    //     sh 'git add .'
                    //     sh 'git commit -m "ci: version bump"'
                    //     sh 'git push origin HEAD:main'
                    // }
                }
            }
        }
    }
}

