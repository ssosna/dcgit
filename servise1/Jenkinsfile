def gitCommit() {
    sh "git rev-parse HEAD > GIT_COMMIT"
    def gitCommit = readFile('GIT_COMMIT').trim()
    sh "rm -f GIT_COMMIT"
    return gitCommit
}

node {
    // Checkout source code from Git
    stage 'Checkout'
//    deleteDir()
    checkout scm

    // Build Docker image
    stage 'Build'
    dir 'service1'
    sh "sudo docker build -t ssosna/dcos:service1_${gitCommit()} ."

    // Log in and push image to GitLab
    stage 'Publish'
    withCredentials(
        [[
            $class: 'UsernamePasswordMultiBinding',
            credentialsId: 'dockerhub',
            passwordVariable: 'DOCKERHUB_PASSWORD',
            usernameVariable: 'DOCKERHUB_USERNAME'
        ]]
    ) {
        sh "sudo docker login -u '${env.DOCKERHUB_USERNAME}' -p '${env.DOCKERHUB_PASSWORD}' -e sergio.sosna@gmail.com"
        sh "sudo docker push ssosna/dcos:${gitCommit()}"
    }
 // Deploy
    stage 'Deploy'

    marathon(
        url: 'http://m1.dcos:8080',
        forceUpdate: false,
        credentialsId: 'dcos-token',
        filename: 'marathon.json',
        appid: 'nginx-mesosphere',
        docker: "ssosna/dcos:${gitCommit()}".toString()
    )
step([$class: 'WsCleanup'])
}