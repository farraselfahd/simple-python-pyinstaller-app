


node{
    stage('Build'){
        docker.image('python:2-alpine').inside{
            checkout scm
            sh 'python -m py_compile ./sources/add2vals.py ./sources/calc.py'

        }
    }
    try{
        stage('Test'){
            docker.image('qnib/pytest').inside{
                checkout scm
                sh 'py.test --verbose --junit-xml test-reports/results.xml ./sources/test_calc.py'
            }
        }
    } finally {
        junit 'test-reports/results.xml'
    }  
    stage('Manual Approval'){
        input message: 'Lanjutkan ke tahap Deploy ?'

    }
    try{
        withCredentials([sshUserPrivateKey(credentialsId: '3dfbace7-3486-4fe9-81f7-f1aef58ae4e6', keyFileVariable: 'identity',usernameVariable: 'ubuntu')]){
            

            withEnv(['VOLUME=$(pwd)/sources:/src','IMAGE=cdrx/pyinstaller-linux:python2']){
                stage('Deploy'){
                dir(path: env.BUILD_ID) { 
                        checkout scm
                        // unstash(name: 'compiled-results') 
                        sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'" 

                    }
                    archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals" 
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"

                    sshagent(credentials: ['3dfbace7-3486-4fe9-81f7-f1aef58ae4e6']){
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@54.151.250.226 'test -f /home/ubuntu/add2vals && rm add2vals'"
                    sh "scp -v -o ForwardAgent=yes -o ServerAliveInterval=10 -o ServerAliveCountMax=1 ${env.BUILD_ID}/sources/dist/add2vals ubuntu@54.151.250.226:/home/ubuntu/add2vals"
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@54.151.250.226 './add2vals 20 6' "
                        
                   
                    }
                }
                

                // deploy to ec2
                // def remote = [:]
                // remote.name = "pyinstaller-app"
                // remote.host = "54.151.250.226"
                // remote.allowAnyHosts = true
                // remote.user = "ubuntu"
                // remote.identityFile = identity

                // sshCommand remote: remote, command: "ls -lS"
                // sshPut remote: remote, from: "${env.BUILD_ID}/sources/dist/add2vals", into: '.'
                // sshCommand remote: remote, command: "chmod +x add2vals"
                // sshScript remote: remote, script: "./add2vals 20 6"

                
                // sh "ssh-agent /bin/bash"
                
                // sh """
                //     eval \$(ssh-agent) && ssh-add ${identity} && ssh-add -l &&
                //     ./add2vals 20 6
                // // """


                sleep 60
            }
        }
        
    } catch (e){
        throw e
    }


    
}