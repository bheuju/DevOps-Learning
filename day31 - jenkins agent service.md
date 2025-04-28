# Run jenkins node as a service

https://www.jenkins.io/blog/2022/12/27/run-jenkins-agent-as-a-service

Run jenkins agent (copied from jenkins after creating node)

```
curl -sO http://192.168.22.98:8080/jnlpJars/agent.jar
java -jar agent.jar -url http://192.168.22.98:8080/ -secret 53d1d2511bbf37efe01d9f5c6383515049b2bc0a60415665599e609f8079d47f -name jenkinsagent -webSocket -workDir "/home/bishal/jenkins-agent"
```

### 1. make a bash script to run the jenkins-agent

```bash
/home/bishal/jenkins-agent/start_jenkins.sh   # path should be the one specified when creating jenkins-node
```

```bash
#!/bin/bash
java -jar agent.jar -url http://192.168.22.98:8080/ -secret 53d1d2511bbf37efe01d9f5c6383515049b2bc0a60415665599e609f8079d47f -name jenkinsagent -webSocket -workDir "/home/bishal/jenkins-agent"
```

`chmod +x start_jenkins.sh`

### 2. create a service file

`/etc/systemd/system/jenkins-agent.service`

```ini
[Unit]
Description=Jenkins Agent

[Service]as
User=root
WorkingDirectory=/home/bishal
ExecStart=/bin/bash /home/bishal/jenkins-agent/start_jenkins.sh
Restart=always

[Install]
WantedBy=multi-user.target
```

## `Jenkinsfile` with Parameters

```groovy
pipeline {
  agent { label 'agent-name'}
  parameters {
    string(name: 'VERSION', defaultValue: '1.0.0', description: "Enter the version number"),
  }

  stages {
    stage('Checkout') {
      steps {
        // Get code from repository
        checkout scm
      }
    }

    stage('Build') {
      steps {
        // Build steps
        echo "Building.. ${params.VERSION} and ${env.BUILD_NUMBER}"
        // Add your build commands here
        sh "docker build -t app:${env.BUILD_NUMBER}"
        sh "docker tag app:${env.BUILD_NUMBER} nexus.devops.com:8082/app:${env.BUILD_NUMBER}"
        sh "docker login -u admin -p admin nexus.devops.com:8082"
        sh "docker push nexus.devops.com:8082/app:${env.BUILD_NUMBER}"
        sh "ip a"
      }
    }

    stage('Test') {
      steps {
        // Test steps
        echo 'Testing..'
        // Add your test commands here
      }
    }

    stage('Deploy') {
      agent { label 'newprodserver'}
      steps {
        // Deploy steps
        echo 'Deploying....'
        // Add your deployment commands here
        sh "docker login -u admin -p redhat nexus.devops.com:8082"
        sh "ls -al"
        sh "sed -i 's|webimage_value|nexus.devops.com:8082/app:${env.BUILD_NUMBER}|' docker-compose.yml"
        sh "docker-compose up -d"
      }
    }
  }

  post {
    success {
      echo 'Pipeline executed successfully!'
    }
    failure {
      echo 'Pipeline execution failed!'
    }
  }
}
```

### Credentials in jenkins

Manage > Credentials > add here.
