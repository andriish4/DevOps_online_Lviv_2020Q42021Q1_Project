pipeline{
    agent any
    tools {
      maven 'Maven3'
    }
    
    stages{
        stage('Checkout'){
            steps{
                deleteDir()
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'GIT4_GLOBAL_JENKINS', url: 'https://github.com/andriish4/spring-petclinic.git']]])
            }
        }
        
        stage('Build'){
            steps{
                
                sh "mvn -B -DskipTests clean package"
            }
         
        }
        stage('Copy artifacts'){
            steps{
                
               sshPublisher(publishers: [sshPublisherDesc(configName: 'web1', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/home/ec2-user/dockerd', remoteDirectorySDF: false, removePrefix: 'target', sourceFiles: 'target/*.jar')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            }
         
        }
    
       // stage ('Docker build image') {
            
        
        
      //  }  
        
        
        stage('Ansible tasks'){
            steps{         
           
      // Create playbook
     writeFile encoding: 'utf8', file: "playbook.yml", text: """---
- hosts: all
  become: True
  tasks:
    - name: Install python pip
      yum:
        name: python-pip
        state: present
    - name: Install docker
      yum:
        name: docker
        state: present
    - name: start docker
      service:
        name: docker
        state: started
        enabled: yes
    - name: Install docker-py python module
      pip:
        name: docker-py
        state: present
   
"""
    // Create inventory
   writeFile encoding: 'utf8', file: "inventory", text: """
[dev]
172.31.27.159 ansible_user=ec2-user ansible_ssh_extra_args='-o ForwardAgent=yes' # Switch to deploy user and forward keys for remote access
"""
                
   ansiblePlaybook credentialsId: 'web1cred', disableHostKeyChecking: true, installation: 'ansible', inventory: "inventory", playbook: "playbook.yml", sudoUser: null
                      
                    //Cleanup
                    sh "rm playbook.yml -f"
                    sh "rm inventory -f"
                }
        }
        
        
        
        
        
        
        //stage('Docker Deploy'){
        //    steps{
        //      ansiblePlaybook credentialsId: 'web1cred', disableHostKeyChecking: true, installation: 'ansible', inventory: 'inventory', playbook: 'playbook.yml'
        //    }
       // }
       
 stage('Docker Build'){
            steps{
                writeFile encoding: 'utf8', file: "Dockerfile", text: """
                FROM tomcat:8
                COPY target/*.jar /usr/local/tomcat/webapps/webapp.jar
""" 
              sh "docker build . -t andriyandriy75/webapp"
            }
        }
        
        stage('DockerHub Push'){
           steps{
                withCredentials([usernamePassword(credentialsId: 'docker_hub', passwordVariable: 'docker_hub_pass', usernameVariable: 'docker_hub_login')]) {
    
 
                    sh "docker login -u ${docker_hub_login} -p ${docker_hub_pass}"
                }
                
                sh "docker push andriyandriy75/webapp "
           }
        }
        
   //     stage('Docker Deploy'){
  //          steps{
    //          ansiblePlaybook credentialsId: 'dev-server', disableHostKeyChecking: true, extras: "-e DOCKER_TAG=${DOCKER_TAG}", installation: 'ansible', inventory: 'dev.inv', playbook: 'deploy-docker.yml'
     //       }
     //   }   
        
        
        stage('Artifacts') {
            steps {
                
                sh 'echo'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true, followSymlinks: false
                  }
                            
                      
        
        }
    }
            
       
}      