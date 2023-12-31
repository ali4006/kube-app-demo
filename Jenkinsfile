node {
    def app

    stage('Clone Repository') {

        checkout scm
    }

    stage('Build Image') {
        app = docker.build("salari/kube-app-demo", "./app")
    }

    stage('Test Image') {
        app.inside {
            sh 'echo "Test Passed!"'
        }
    }

    stage('Push Image') {
        docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
            app.push("${env.BUILD_NUMBER}")
        }
    }

    stage('Trigger Manifest Update'){
        echo "Update manifest file"

        script {
            catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                    //def encodedPassword = URLEncoder.encode("$GIT_PASSWORD",'UTF-8')
                    sh "git config user.email ali.salar007@gmail.com"
                    sh "git config user.name ali4006"
                    sh "cat deployment.yaml"
                    sh "sed -i 's+salari/kube-app-demo.*+salari/kube-app-demo:${env.BUILD_NUMBER}+g' deployment.yaml"
                    sh "cat deployment.yaml"
                    sh "git add ."
                    sh "git commit -m 'Done by Jenkins Job changemanifest: ${env.BUILD_NUMBER}'"
                    sh "git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${GIT_USERNAME}/kube-app-demo.git HEAD:main"
                }
            }
        }
    }
}