FROM jenkins
USER root
RUN apt-get update && apt-get install -y groovy
#COPY ./files/create-users.groovy /tmp/create-users.groovy
#RUN groovy /tmp/create-users.groovy
USER jenkins

RUN usr/local/bin/install-plugins.sh github:1.22.3
RUN /usr/local/bin/install-plugins.sh docker:0.16.2
RUN /usr/local/bin/install-plugins.sh saferestart:0.3

#COPY ./files/plugins.txt /usr/share/jenkins/plugins.txt
#RUN /usr/local/bin/plugins.sh /usr/share/jenkins/plugins.txt
#RUN echo 'jenkins.model.Jenkins.instance.securityRealm.createAccount("user1", "password123")' | java -jar jenkins-cli.jar -s http://localhost/ groovy =
EXPOSE 8080
