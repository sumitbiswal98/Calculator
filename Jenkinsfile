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
                        openshift.withProject("new-pro")
                        {
                            def buildConfigExists = openshift.selector("bc", "calculator").exists()
                            
                            if(!buildConfigExists)
                            {
                                openshift.newBuild("--name=calculator", "--docker-image=registry.redhat.io/jboss-eap-7/eap74-openjdk8-openshift-rhel7", "--binary")
                            }
                            
                            openshift.selector("bc", "calculator").startBuild("--from-file=target/Calculator-1.0.jar", "--follow")
                            
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
                        openshift.withProject("new-pro")
                        {
                            def deployment = openshift.selector("dc", "calculator")
                            
                            if(!deployment.exists())
                            {
                                openshift.newApp('calculator', "--as-deployment-config").narrow('svc').expose()
                            }
                            
                            timeout(5)
                            {
                                openshift.selector("dc", "calculator").related('pods').untilEach(1)
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
