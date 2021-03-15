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
        stage('Sync files') {
                   
          steps {         
                   
            
            // writeFile file: "output/uselessfile.md", text: "This file is useless, no need to archive it."
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
    - name: Start the container
      docker_container:
        name: hariapp
        image: "andriyandriy75/webapp"
        state: started
        published_ports:
          - 0.0.0.0:8080:8080
"""
                    // Create inventory
                    writeFile encoding: 'utf8', file: "inventory", text: """
[dev]
172.31.27.159 ansible_user=ec2-user ansible_ssh_extra_args='-o ForwardAgent=yes' # Switch to deploy user and forward keys for remote access
"""

                   // ansiColor('xterm') {
                        // We need ssh-agent for remote access
                       
                            ansiblePlaybook credentialsId: 'web1cred', disableHostKeyChecking: true, installation: 'ansible', inventory: "inventory", playbook: "playbook.yml", sudoUser: null
                       
                    }
                    //Cleanup
                    sh "rm playbook.yml -f"
                    sh "rm inventory -f"
            
        }
        
        
        
        
        
        
        //stage('Docker Deploy'){
        //    steps{
        //      ansiblePlaybook credentialsId: 'web1cred', disableHostKeyChecking: true, installation: 'ansible', inventory: 'inventory', playbook: 'playbook.yml'
        //    }
       // }
       
        stage('Artifacts') {
            steps {
                
                sh 'echo'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true, followSymlinks: false
                  }
                            
                      
        
        }
    }
            
       
}      
