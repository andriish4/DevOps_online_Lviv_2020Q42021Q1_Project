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
               sh "mvn -B -DskipTests clean install"
            }
         
        }
        
   //      stage('Test'){
   //         steps{
   //            sh "mvn test"
   //         }
   //       }
  
    //      stage('Copy artifacts'){
  //          steps{
                
  //             sshPublisher(publishers: [sshPublisherDesc(configName: 'web1', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/home/ec2-user/dockerd', remoteDirectorySDF: false, removePrefix: 'target', sourceFiles: 'target/*.jar')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
  //          }
         
  //      }
        stage('Docker Build'){
            steps{
                writeFile encoding: 'utf8', file: "Dockerfile", text: """
                FROM openjdk:8-jre-alpine3.9
                COPY target/*.jar /home/webapp.jar
                CMD ["java","-jar","/home/webapp.jar"]
""" 
              sh "docker build . -t andriyandriy75/webapp:$BUILD_ID"
            }
        }
        
        stage('DockerHub Push'){
           steps{
                withCredentials([usernamePassword(credentialsId: 'docker_hub', passwordVariable: 'docker_hub_pass', usernameVariable: 'docker_hub_login')]) {
                  sh "docker login -u ${docker_hub_login} -p ${docker_hub_pass}"
                }
                
                sh "docker push andriyandriy75/webapp:$BUILD_ID"
           }
        }
        
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
    - name: Start the container
      docker_container:
        name: web-app
        image: "andriyandriy75/webapp:$BUILD_ID"
        state: started
        published_ports:
          - 0.0.0.0:9995:8080 
      """
    // Create inventory file
   writeFile encoding: 'utf8', file: "inventory", text: """
[dev]
172.31.27.159 ansible_user=ec2-user ansible_ssh_extra_args='-o ForwardAgent=yes' # Switch to deploy user and forward keys for remote access
[qa]
172.31.31.119 ansible_user=ec2-user ansible_ssh_extra_args='-o ForwardAgent=yes'

"""
                
   ansiblePlaybook credentialsId: 'web1cred', disableHostKeyChecking: true, installation: 'ansible', inventory: "inventory", playbook: "playbook.yml", sudoUser: null
   ansiblePlaybook credentialsId: 'qacred', disableHostKeyChecking: true, installation: 'ansible', inventory: "inventory", playbook: "playbook.yml", sudoUser: null
           
            }
        }
       stage('Docker Deploy'){
            steps{
              ansiblePlaybook credentialsId: 'web1cred', disableHostKeyChecking: true, extras: "-e DOCKER_TAG=$BUILD_ID", installation: 'ansible', inventory: 'inventory', playbook: 'playbook.yml'
              ansiblePlaybook credentialsId: 'qacred', disableHostKeyChecking: true, extras: "-e DOCKER_TAG=$BUILD_ID", installation: 'ansible', inventory: "inventory", playbook: "playbook.yml", sudoUser: null           
            }
        }   
              
 //       stage('Artifacts') {
 //           steps {
 //               archiveArtifacts artifacts: 'target/*.jar', fingerprint: true, followSymlinks: false
                //Cleanup
 //               sh "rm playbook.yml -f"
 //               sh "rm inventory -f"

  //                }
                            
                      
        
  //      }
    }
            
       
}      
