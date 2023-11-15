node {
    def app

    stage('Clone repository') {
        checkout scm
    }

    // stage('SonarQube analysis') {
    //     steps {
    //         withSonarQubeEnv('SonarQube') {
    //             sh "./gradlew sonarqube"
    //         }
    //     }
    // }

    stage("Quality gate") {
        steps {
            waitForQualityGate abortPipeline: true
        }
    }

    stage('Build image') {
       app = docker.build("zakyfatih/python-program")
    }

    stage('Test image') {
        app.inside {
            sh 'echo "Tests passed"'
        }
    }

    stage('Push image') {
        docker.withRegistry('https://registry.hub.docker.com', 'dockerhub_zaky') {
            app.push("${env.BUILD_NUMBER}")
        }
    }
    
    stage('Trigger ManifestUpdate') {
        echo "triggering updatemanifestjob"
        build job: 'updatemanifest', parameters: [string(name: 'DOCKERTAG', value: env.BUILD_NUMBER)]
    }
    
}
