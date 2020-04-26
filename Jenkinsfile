imageName = 'mlabouardy/application'
resourceGroup = 'management'
registry = 'https://alldaydevops.azurecr.io'

node('workers'){
    stage('Checkout'){
        checkout scm
    }

    dir("pipeline") {
        stage('Test'){
            docker.build("${imageName}:${env.BUILD_ID}", '-f Dockerfile.test .')
            sh "docker run --rm ${imageName}:${env.BUILD_ID}"
        }

        stage('Build'){
            docker.build("${imageName}")
        }

        stage('Push'){
            docker.withRegistry(registry, 'registry') {
                docker.image(imageName).push("${commitID()}")

                if (env.BRANCH_NAME == 'master') {
                    docker.image(imageName).push('latest')
                }
                if (env.BRANCH_NAME == 'preprod') {
                    docker.image(imageName).push('preprod')
                }
                if (env.BRANCH_NAME == 'develop') {
                    docker.image(imageName).push('develop')
                }
            }
        }

        stage('Deploy'){
            if (env.BRANCH_NAME == 'master') {
                sh "az container restart --name application-production --resource-group ${resourceGroup}"
            }
            if (env.BRANCH_NAME == 'preprod') {
                sh "az container restart --name application-staging --resource-group ${resourceGroup}"
            }
            if (env.BRANCH_NAME == 'develop') {
                sh "az container restart --name application-sandbox --resource-group ${resourceGroup}"
            }
        }
    }
}

def commitID() {
    sh 'git rev-parse HEAD > ../.git/commitID'
    def commitID = readFile('../.git/commitID').trim()
    sh 'rm ../.git/commitID'
    commitID
}
