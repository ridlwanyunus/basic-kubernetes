# Integrating git, maven and tomcat using Jenkins Pipeline

## Install Jenkins

1. Install `jenkins` 
   ```bash
   $ sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
   $ sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
   $ sudo yum install jenkins
   $ sudo systemctl start jenkins
   $ sudo cat /var/lib/jenkins/secrets/initialAdminPassword > initialAdminPassword
   ```
   Setelah itu install Jenkins secara GUI dengan memilih `Recommended Installation`

2. Install `java`
   ```bash
   $ tar -zxvf jdk-8u221-linux-x64.tar.gz
   $ sudo mv jdk1.8.0_221/ /opt/java
   $ sudo update-alternatives --install /usr/bin/java java /opt/java/bin/java 10
   $ sudo update-alternatives --config java
   $ export JAVA_HOME=/opt/java
   ```   

3. Install `maven`
   ```bash
   $ tar -zxvf apache-maven-3.6.3-bin.tar.gz
   $ sudo mv apache-maven-3.6.3 /opt/maven
   $ sudo update-alternatives --install /usr/bin/mvn mvn /opt/maven/bin/mvn 10
   $ sudo update-alternatives --config mvn
   $ export M2_HOME=/opt/maven
   ```   

4. Install `git`
   ```bash
   $ sudo yum install git
   ```

5. Install Plugin `Pipeline: Multibranch with defaults`
   ```bash
   Dashboard > Manage Jenkins > Manage Plugins > Available > Pipeline: Multibranch with defaults
   ```

6. Install Plugin `SSH Agent Plugin`
   ```bash
   Dashboard > Manage Jenkins > Manage Plugins > Available > SSH Agent Plugin
   ```

7. Install Plugin `Maven Integration plugin`
    ```bash
   Dashboard > Manage Jenkins > Manage Plugins > Available > Maven Integration plugin
   ```
8. Creating pipeline 'first_pipeline'
   ```bash
   Dashboard > New Item > Pipeline > 'first_pipeline'
   ```

9. Insert the `Pipeline script`
   ```json
   pipeline {
        agent any 
        environment {
            PATH = "/opt/maven/bin:$PATH"
        }
        stages {
            stage("git") {
                steps {
                    git 'https://github.com/ravdy/hello-world.git'
                }
            }
            stage("maven") {
                steps {
                    sh "mvn clean install"
                }
            }
            stage("tomcat") {
                steps {
                    sshagent(['deploy_user']) {
                        sh "scp -o StrictHostKeyChecking=no webapp/target/webapp.war ec2-user@18.141.194.200:/opt/tomcat/webapps"
                    }
                }
            }
        }
    }
   ```
10. Build pipeline `first_pipeline`
    ```bash
    Dashboard > first_pipeline > Build Now
    ```

