pipeline {
    agent { label 'iti-ubuntu-slave'}
//     parameters
//     {
//         choice(name: 'ENV', choices: ['dev', 'test', 'prod',"release"])
//     } 

    stages {

        stage('bulidd')
        {
            steps
            {
                script
                {
                    if (env.BRANCH_NAME == "release")
                    {
                        withCredentials([usernamePassword(credentialsId: 'Dockerhub-cred', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')])
                        {
                            sh """
                                docker login -u $USERNAME -p $PASSWORD
                                docker build -t moelzedy/iti-bakehouse-jenkins-proj:${BUILD_NUMBER} .
                                docker push moelzedy/iti-bakehouse-jenkins-proj:${BUILD_NUMBER}
                                echo ${BUILD_NUMBER} > ../proj-build-number.txt
                            """
                        }
                    }
                }
            }
        }
        
        stage('deploy')
        {
            steps
            {   
                script
                {
                    if (env.BRANCH_NAME == "dev" || env.BRANCH_NAME == "test" || env.BRANCH_NAME == "prod")
                    {
                        withCredentials([file(credentialsId: 'k8s-kubeconfig', variable: 'KUBECONFIG')])
                        {
                        sh """
                            export BUILD_NUMBER=\$(cat ../proj-build-number.txt)
                            mv Deployment/deploy.yaml Deployment/deploy.yaml.tmp
                            cat Deployment/deploy.yaml.tmp | envsubst > Deployment/deploy.yaml
                            rm -f Deployment/deploy.yaml.tmp
                            kubectl apply -f Deployment --kubeconfig=${KUBECONFIG}
                         """
                        }
                    }
                }
            }
        }
    }
}
