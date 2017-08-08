node("${env.SLAVE}") {
  stage("Clear"){
  /*
    cleanWs()
  */
    git branch: 'master', url: 'https://github.com/Forbear/ansible-exam.git'
  }
  stage("Build"){
    /*
        Update file src/main/resources/build-info.txt with following details:
        - Build time
        - Build Machine Name
        - Build User Name
        - GIT URL: ${GIT_URL}
        - GIT Commit: ${GIT_COMMIT}
        - GIT Branch: ${GIT_BRANCH}

        Simple command to perform build is as follows:
        $ mvn clean package -DbuildNumber=$BUILD_NUMBER
    */
    sh "echo build artefact"
    sh "echo \$(date) >> src/main/resources/build-info.txt"
    sh "cat src/main/resources/build-info.txt"
    sh "mvn clean package -DbuildNumber=$BUILD_NUMBER"
  }

  stage("Package"){
    /*
        use tar tool to package built war file into *.tar.gz package
    */
    sh "mntlab-exam/"
    sh "echo package artefact"
    sh "tar -czvf mnt-archive.tar.gz target/mnt-exam.war"
    sh "ls -la | grep tar.gz"
  }

  stage("Roll out Dev VM"){
    /*
        use ansible to create VM (with developed vagrant module)
    */
    sh "echo ansible-playbook createvm.yml ..."
    withEnv([    "ANSIBLE_FORCE_COLOR=true",     "PYTHONUNBUFFERED=1"]) {
      ansiColor('xterm') {
        sh "ansible-playbook createvm.yml -vv"
      }
    }
  }

  stage("Provision VM"){
    /*
        use ansible to provision VM
        Tomcat and nginx should be installed
    */
    sh "echo ansible-playbook provisionvm.yml ..."
    withEnv([    "ANSIBLE_FORCE_COLOR=true",     "PYTHONUNBUFFERED=1"]) {
      ansiColor('xterm') {
        sh "ansible-playbook provisionvm.yml -vv"
      }
    }
  }

  stage("Deploy Artefact"){
    /*
        use ansible to deploy artefact on VM (Tomcat)
        During the deployment you should create file: /var/lib/tomcat/webapps/deploy-info.txt
        Put following details into this file:
        - Deployment time
        - Deploy User
        - Deployment Job
    */
    sh "echo ansible-playbook deploy.yml -e artefact=... ..."
    withEnv([    "ANSIBLE_FORCE_COLOR=true",     "PYTHONUNBUFFERED=1"]) {
      ansiColor('xterm') {
        sh "ansible-playbook deploy.yml -vv"
      }
    }
  } 
  stage("Use tests"){
    withEnv([    "ANSIBLE_FORCE_COLOR=true",     "PYTHONUNBUFFERED=1"]) {
      ansiColor('xterm') {
        sh "ansible-playbook test.yml -vv"
      }
    }
  }
  stage("Test Artefact is deployed successfully"){
    /*
        use ansible to artefact on VM (Tomcat)
        During the deployment you should create file: /var/lib/tomcat/webapps/deploy-info.txt
        Put following details into this file:
        - Deployment time
        - Deploy User
        - Deployment Job
    */
    sh "echo ansible-playbook application_tests.yml -e artefact=... ..."
    input 'Wait to abort vm'
  }
  stage("DestroyVM Clear directory"){
    withEnv([    "ANSIBLE_FORCE_COLOR=true",     "PYTHONUNBUFFERED=1"]) {
      ansiColor('xterm') {
        sh "ansible-playbook destroy.yml -vv"
      }
    }
    cleanWs()
  }
}

