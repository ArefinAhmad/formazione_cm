# README - Formazione CM

Questo progetto "formazione\_cm" è stato realizzato per guidare l'automazione nella creazione, gestione e deploy di container Linux tramite Ansible, Docker/Podman e Jenkins.
L'obiettivo è creare un workflow completo che va dalla configurazione di un registry locale, alla build di container, fino alla gestione di pipeline CI/CD con Jenkins.

## Struttura del progetto

formazione\_cm/
├── ansible.cfg                # Configurazione globale di Ansible
├── inventory.ini              # Inventario Ansible dei target e container
├── Jenkinsfile                # Pipeline Jenkins per build, push e deploy
├── README.md                  # Questo file
├── requirements.yml           # Dipendenze Ansible e ruoli esterni
├── StepbyStep.md              # Guida dettagliata step-by-step del progetto
├── vault.yml                  # Contiene password e credenziali criptate
├── files/                     # Chiavi SSH e risorse necessarie
│   ├── id\_rsa\_genericuser
│   └── id\_rsa\_genericuser.pub
├── jenkins/                   # Dockerfile e configurazioni per Jenkins master
│   ├── Dockerfile
│   └── password.txt
├── jenkins-agent/             # Dockerfile per Jenkins agent custom
│   └── Dockerfile
├── playbooks/                 # Playbook principali del progetto
│   ├── build-containers.yml
│   ├── container-deploy.yml
│   ├── container-playbook.yml
│   ├── id\_rsa\_genericuser.pub
│   └── step3.yml
├── roles/                     # Ruoli modulari Ansible
│   ├── build\_images/
│   │   ├── defaults/main.yml
│   │   └── tasks/main.yml
│   └── registry/
│       ├── defaults/main.yml
│       └── tasks/main.yml
└── templates/                 # Dockerfile dei container e chiavi
├── Dockerfile.agent
├── Dockerfile.almalinux
├── Dockerfile.ubuntu
└── files/id\_rsa\_genericuser.pub

## Step 1 - Creare il Primo Playbook

Obiettivo: configurare un Docker registry locale senza autenticazione.

* Playbook: playbooks/container-playbook.yml
* Include task per:

  * Creazione del registry
  * Gestione delle immagini con moduli Docker o Podman (docker\_image, podman\_image)

## Step 2 - Build di Container

Obiettivo: creare almeno due container con OS differenti (Ubuntu e AlmaLinux) con SSH attivo e utente configurato con chiave e sudo.

* Playbook: playbooks/build-containers.yml
* Container generati:

  * ubuntu-ssh (porta host 2223)
  * almalinux-ssh (porta host 2224)
* Compatibilità con Docker e Podman
* Chiave SSH pubblica copiata nel container (id\_rsa\_genericuser.pub)

## Step 3 - Creazione di Ruoli Ansible

Obiettivo: strutturare i task in ruoli modulari e riutilizzabili.

* Ruoli:

  * roles/registry -> gestione registry locale
  * roles/build\_images -> build delle immagini e run dei container
* Playbook: playbooks/step3.yml
* Rilevamento automatico dell'engine Docker o Podman
* Parametrizzazione di immagini e porte

## Step 4 - Ansible Vault

Obiettivo: protezione delle credenziali sensibili.

* File: vault.yml
* Contiene password e chiavi criptate
* Utilizzato da tutti i playbook per sicurezza

## Step 5 - Jenkins & Ansible

Obiettivo: creare un container Jenkins agent con Docker/Podman, configurare pipeline automatica di build, push e deploy.

* Dockerfile Jenkins agent: jenkins-agent/Dockerfile

  * Base: jenkins/inbound-agent
  * Installazione: Docker CLI, kubectl, helm, Ansible, sudo
  * Configurazione gruppo docker per utente jenkins

* Jenkinsfile

  * Stages:

    1. Checkout repository
    2. Build immagine Docker con Dockerfile.agent
    3. Tag progressivo con BUILD\_NUMBER e latest
    4. Push sul registry locale
    5. Deploy container target con Ansible (playbooks/container-deploy.yml)

* Container target deploy:

  * SSH attivo
  * Utente deployer
  * Docker installato

* Playbook deploy: playbooks/container-deploy.yml

  * Stop e rimozione container precedente
  * Avvio container con /usr/sbin/sshd -D
  * Controllo che Docker sia installato

## Inventory Ansible (inventory.ini)

\[local]
localhost ansible\_connection=local

\[ssh\_containers]
ubuntu   ansible\_host=localhost ansible\_port=2223 ansible\_user=genericuser ansible\_ssh\_private\_key\_file=../files/id\_rsa\_genericuser
almalinux ansible\_host=localhost ansible\_port=2224 ansible\_user=genericuser ansible\_ssh\_private\_key\_file=../files/id\_rsa\_genericuser

\[target]
my-container ansible\_host=127.0.0.1 ansible\_port=2222 ansible\_user=deployer ansible\_ssh\_private\_key\_file=files/id\_rsa\_genericuser ansible\_become=true

## Procedura di Esecuzione

1. Build dei container base (Step 2):
   ansible-playbook playbooks/build-containers.yml

2. Creazione ruoli e build automatica (Step 3):
   ansible-playbook playbooks/step3.yml

3. Deploy container target (Step 5):
   ansible-playbook playbooks/container-deploy.yml -e "image=host.docker.internal:5000/myapp\:latest" --ask-become-pass

4. Esecuzione pipeline Jenkins:

   * Avviare Jenkins master e agent
   * Eseguire la pipeline dal Jenkinsfile
   * La pipeline esegue build, tag, push e deploy automatico sul container target

## Note Aggiuntive

* Tutti i container SSH hanno porte mappate differenti (2222, 2223, 2224) per evitare conflitti.
* L'agent Jenkins necessita di sudo per eseguire Ansible con become: true.
* I playbook sono compatibili sia con Docker che con Podman.
* Chiavi SSH e credenziali sensibili sono memorizzate in files/ e vault.yml.
* Tutti i task sono parametrizzati per massima flessibilità.

   
