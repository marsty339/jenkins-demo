node('jnlp') {
    stage('Prepare') {
        echo "1.Prepare Stage"
        checkout scm
        script {
            build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
            if (env.BRANCH_NAME != 'master') {
                build_tag = "${env.BRANCH_NAME}-${build_tag}"
            }
        }
    }
    stage('Test') {
      echo "2.Test Stage"
    }
    stage('Build') {
        echo "3.Build Docker Image Stage"
        sh "docker build -t ty/jenkins-demo:${build_tag} ."
    }
    stage('Push') {
        echo "4.Push Docker Image Stage"
        withCredentials([usernamePassword(credentialsId: 'Hub', passwordVariable: 'HubPassword', usernameVariable: 'HubUser')]) {
            sh "docker login -u ${HubUser} -p ${HubPassword}  http://10.0.2.15:5000"
            sh "docker tag ty/jenkins-demo:${build_tag} 10.0.2.15:5000/ty/jenkins-demo:${build_tag}"
            sh "docker push 10.0.2.15:5000/ty/jenkins-demo:${build_tag}"
        }
    }
    stage('Deploy') {
        echo "5. Deploy Stage"
        if (env.BRANCH_NAME == 'master') {
            input "确认要部署线上环境吗？"
        }
        sh "sed -i 's#cnych/jenkins-demo:<BUILD_TAG>#10.0.2.15:5000/ty/jenkins-demo:${build_tag}#' k8s.yaml"
        sh "sed -i 's#<BRANCH_NAME>#${env.BRANCH_NAME}#' k8s.yaml"
        sh "kubectl create -f k8s.yaml --record"
    }
}
