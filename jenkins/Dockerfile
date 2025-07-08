FROM jenkins/jenkins:lts

USER root

# Install Docker CLI
RUN apt-get update && apt-get install -y \
    docker.io \
    && apt-get clean

# Add Jenkins to Docker group (optional, but safer than running as root)
RUN usermod -aG docker jenkins

USER jenkins
