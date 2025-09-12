Step 1 – Creare il Primo Playbook
Struttura cartelle
mkdir -p ~/formazione_cm/{playbooks,roles,files,templates}
cd ~/formazione_cm

File richiesti

inventory.yml → definisce gli host (per ora localhost)

ansible.cfg → per usare localhost e non avere warning

playbooks/container-playbook.yml → playbook principale

inventory.yml
all:
  hosts:
    localhost:
      ansible_connection: local

ansible.cfg
[defaults]
inventory = ./inventory.yml
host_key_checking = False

playbooks/container-playbook.yml

Esempio che installa un registry senza autenticazione e un container hello-world:

---
- name: Step 1 - Playbook gestione container
  hosts: localhost
  become: false
  tasks:
    - name: Avvia un registry docker (senza autenticazione)
      community.docker.docker_container:
        name: registry
        image: registry:2
        state: started
        restart_policy: always
        published_ports:
          - "5000:5000"

    - name: Pull immagine hello-world con Docker
      community.docker.docker_image:
        name: hello-world
        source: pull

    - name: Run container hello-world
      community.docker.docker_container:
        name: hello-world-container
        image: hello-world
        state: started


👉 Su Mac con Docker Desktop questo funziona al volo.

✅ Step 2 – Build di Container con SSH

Qui creiamo due immagini (es. Ubuntu e CentOS/AlmaLinux) che:

ascoltano su porta 22

hanno SSH attivo

hanno un utente genericuser con accesso tramite chiave

1. Genera chiave SSH per il container
ssh-keygen -t rsa -b 4096 -C "genericuser@example.com" -f ~/formazione_cm/files/id_rsa_genericuser


Avrai due file:

id_rsa_genericuser (private key, NON va nel container)

id_rsa_genericuser.pub (public key, da copiare nell’immagine)

2. Crea due Dockerfile

📂 ~/formazione_cm/templates/Dockerfile.ubuntu

FROM ubuntu:22.04
RUN apt-get update && apt-get install -y openssh-server sudo && \
    mkdir /var/run/sshd && \
    useradd -m -s /bin/bash genericuser && \
    echo "genericuser ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers && \
    mkdir -p /home/genericuser/.ssh
COPY id_rsa_genericuser.pub /home/genericuser/.ssh/authorized_keys
RUN chown -R genericuser:genericuser /home/genericuser/.ssh && chmod 600 /home/genericuser/.ssh/authorized_keys
EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]


📂 ~/formazione_cm/templates/Dockerfile.almalinux

FROM almalinux:9
RUN dnf install -y openssh-server sudo && \
    mkdir /var/run/sshd && \
    useradd -m -s /bin/bash genericuser && \
    echo "genericuser ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers && \
    mkdir -p /home/genericuser/.ssh
COPY id_rsa_genericuser.pub /home/genericuser/.ssh/authorized_keys
RUN chown -R genericuser:genericuser /home/genericuser/.ssh && chmod 600 /home/genericuser/.ssh/authorized_keys
EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]

3. Playbook di build e run

📂 playbooks/build-containers.yml

---
- name: Step 2 - Build e run container SSH
  hosts: localhost
  tasks:
    - name: Copia la chiave pubblica nel contesto build
      copy:
        src: ../files/id_rsa_genericuser.pub
        dest: ./id_rsa_genericuser.pub

    - name: Build immagine Ubuntu SSH
      community.docker.docker_image:
        name: ubuntu-ssh
        build:
          path: ../templates
          dockerfile: Dockerfile.ubuntu
        source: build

    - name: Build immagine AlmaLinux SSH
      community.docker.docker_image:
        name: almalinux-ssh
        build:
          path: ../templates
          dockerfile: Dockerfile.almalinux
        source: build

    - name: Run container Ubuntu SSH
      community.docker.docker_container:
        name: ubuntu-ssh-container
        image: ubuntu-ssh
        state: started
        published_ports:
          - "2222:22"

    - name: Run container AlmaLinux SSH
      community.docker.docker_container:
        name: almalinux-ssh-container
        image: almalinux-ssh
        state: started
        published_ports:
          - "2223:22"


👉 Ora puoi connetterti così:

ssh -i ~/formazione_cm/files/id_rsa_genericuser -p 2222 genericuser@localhost   # Ubuntu
ssh -i ~/formazione_cm/files/id_rsa_genericuser -p 2223 genericuser@localhost   # AlmaLinux

✅ Step 3 – Creazione Ruoli

Da qui in poi userai:

ansible-galaxy init roles/build_images
ansible-galaxy init roles/registry


e sposterai i task degli step precedenti dentro roles/.../tasks/main.yml.
In questo modo il playbook principale diventa solo una chiamata ai ruoli.

✅ Step 4 – Vault

Su Mac puoi salvare la password in ~/.vault:

echo "mypassword" > ~/.vault
ansible-vault encrypt_string "mypassword" --vault-password-file ~/.vault

✅ Step 5 – Jenkins & Ansible

Ti serve un container Jenkins con Docker attivo (puoi usare Docker-in-Docker o montare la socket Docker Desktop).

La pipeline farà:

Build immagine con tag progressivo

Push sul registry locale (localhost:5000/...)

Deploy con Ansible playbook

👉 In sintesi:

Step 1: registry + hello-world

Step 2: build immagini SSH (Ubuntu + AlmaLinux) con utente genericuser

Step 3: ruoli (registry, build_images)

Step 4: vault per password e chiavi

Step 5: Jenkins pipeline con integrazione Ansible