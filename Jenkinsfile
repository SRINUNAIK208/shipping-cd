pipeline {
    agent {
        label 'AGENT-1'
    }
    options {
        timeout(time:30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }
    environment{
      appVersion = ''
        REGION = 'us-east-1'
        ACC_ID = '388343452532'
        project = 'roboshop'
        component = 'shipping'
    }
    parameters {
        string(name: 'appVersion', description:' application version')
        choice(name: 'deploy_to', choices:['dev','qa','prod'], description: 'pick the env')
    }
    stages{
        stage('Deployment'){
            steps{
                withAWS(credentials: 'aws-cred', region: 'us-east-1'){
                    sh """
                      aws eks update-kubeconfig --region ${region} -n ${project}
                      kubectl get nodes
                      sed -i "s/IMAGE_VERSION/${params.appVersion}/g" values-${params.deploy}.yaml
                      helm upgrade --install ${component} -f values-${params.deploy}.yaml -n ${project}
                    """
                }
            }
        }
        stage('check deployment status'){
            steps{
                script {
                    withAWS(credentials: 'aws-cred', region: 'us-east-1'){
                        def DeploymentStatus = sh(returnStdout: true, script: 'kubectl rollout status deployment/${component} --timeout=30s -n ${project} || echo FAILED ').trim();
                        if (DeploymentStatus.contains('successfully rolled out'))
                        {
                            echo "Deployment is success"
                        }
                        else {
                            echo "deployment is failed, start proceding with rollback"
                            sh """
                            helm rollback ${component} -n ${project}
                            sleep 10
                            """
                            def RollbackStatus = sh(returnStdout: true, script: 'kubectl rollout status deployment/${component} --timeout=30s -n ${project} || echo FAILED ').trim();
                            if (RollbackStatus.contains('successfully rolled out'))
                            {
                                echo "Deployment is failed, rollback is success"
                            }
                            else {
                                error "Deployment failed and rollback also failed, its emergency"
                            }

                        }


                    }
                }
            }
        }
    }
}