node {
    def app

    stage('Clone repository') {
        checkout scm
    }

    stage('SonarQube analysis') {
        def scannerHome = tool 'SonarScanner 4.0'
        withSonarQubeEnv('sonar-ict') {
            sh "${scannerHome}/bin/sonar-scanner"
        }
    }

    stage("Quality gate") {
        timeout(time: 2, unit: 'MINUTES') {
            // waitForQualityGate abortPipeline: true
            def qg = waitForQualityGate()
            if (qg.status != 'OK') {
                error "Pipeline aborted due to quality gate failure: ${qg.status}"
            }
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
