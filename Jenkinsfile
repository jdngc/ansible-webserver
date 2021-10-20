pipeline{
    agent{label "agentfarm"}
    environment {
    	KEY_FILE = '/home/ubuntu/ .ssh/vm-instancekey.pem'
    	USER = 'ubuntu'
    } 
    stages {
        stage('Delete the workspace') {
	    steps {
	        cleanWs()
		}
	    }
	    stage('Installing Ansible') {
	        steps {
		    script {
		        def ansible_exists = fileExists '/usr/bin/ansible'
			if (ansible_exists) {
			    echo "Skipping Ansible install -already exists"
		        } else {
		            sh 'sudo apt-get update -y && sudo apt-get upgrade -y'
		            sh 'sudo apt install -y wget tree unzip ansible python3-pip python3-apt'
		        }
		    }
		}
	    }
		stage('Download Ansible Code') {
		    steps {
		            git credentialsId: 'git-repo-creds', url: 'git@github.com:jdngc/ansible-webserver.git'
	            }
		}
		stage('Run ansible-lint against playbooks') {
		    steps{
		        sh 'docker run --rm -v $WORKSPACE/playbooks:/data cytopia/ansible-lint apache-install.yml'
			    sh 'docker run --rm -v $WORKSPACE/playbooks:/data cytopia/ansible-lint website-update.yml'
			    sh 'docker run --rm -v $WORKSPACE/playbooks:/data cytopia/ansible-lint website-test.yml'
		    }
		}
		stage('Install apache & update website'){
			steps{
				sh 'ansible-playbook -u $USER --private-key $KEY-FILE -i $WORKSPACE/host_inventory $WORKSPACE/playbooks/apache-install.yml'
				sh 'ansible-playbook -u $USER --private-key $KEY-FILE -i $WORKSPACE/host_inventory $WORKSPACE/playbooks/website-update.yml'
		}
		stage('Test website'){
			steps{
				sh 'ansible-playbook -u $USER --private-key $KEY-FILE -i $WORKSPACE/host_inventory $WORKSPACE/playbooks/website-test.yml'
		    }
		}		
    }
}
