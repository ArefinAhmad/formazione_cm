FROM jenkins/inbound-agent:latest

USER root
# Installa Docker CLI
RUN apt-get update && \
    apt-get install -y docker.io && \
    apt-get clean


# Aggiungi utente jenkins al gruppo docker
#RUN groupadd -g 1 dockermac && usermod -aG dockermac jenkins

RUN groupdel docker || true
RUN groupadd -for -g 988 docker
RUN usermod -aG docker jenkins
# Installa kubectl
ENV KUBECTL_VERSION=v1.32.0
RUN curl -LO "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl" && \
    install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl && \
    rm kubectl

# Installa helm
RUN curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# installa ansible
RUN apt-get update && \
    apt-get install -y ansible && \
    rm -rf /var/lib/apt/lists/*

USER jenkins
# 4cc18731e974198be8a18fb017b6906fa2c3aa5daacd25f614400124f8cb07d2
