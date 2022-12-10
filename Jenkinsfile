pipeline 
{
    
    
    agent 
    {
        label 'maven'
    }
    
    stages 
    {
        
        
        
        stage('Build')
        {
            steps
            {
                echo 'Building...'
                sh 'mvn clean package'
            }
        }
        
        
        stage('Create Container Image')
        {
            steps 
            {
                echo 'create container image..'
                script
                {
                    openshift.withCluster()
                    {
                        openshift.withProject("ci-cd")
                        {
                            def buildConfigExists = openshift.selector("bc", "codelikethewind").exists()
                            
                            if(!buildConfigExists)
                            {
                                openshift.newBuild("--name=codelikethewind", "--docker-image=registry.redhat.io/redhat-openjdk-18/openjdk18-openshift@sha256:db485b9aeb6ed953cb9b642061a7cd92b1b20cded591ffd4be14e00813274789", "--binary")
                            }
                            
                            openshift.selector("bc", "codelikethewind").startBuild("--from-file=target/simple-servlet-0.0.1-SNAPSHOT.war", "--follow")
                            
                        }
                    }
                }
            }
        }
        
        
        
        
        stage('Deploy')
        {
            steps
            {
                echo 'Deploying....'
                script
                {
                    openshift.withCluster()
                    {
                        openshift.withProject("ci-cd")
                        {
                            def deployment = openshift.selector("dc", "codelikethewind")
                            
                            if(!deployment.exists())
                            {
                                openshift.newApp('codelikethewind', "--as-deployment-config").narrow('svc').expose()
                            }
                            
                            timeout(5)
                            {
                                openshift.selector("dc", "codelikethewind").related('pods').untilEach(1)
                                {
                                    return (it.object().status.phase == "Running")
                                }
                            }
                            
                            
                        }
                    }
                }
            }
        }
        
        
        
        
    }
    
    
}
