# formazione_cm
Step 1: registry + esempi docker/podman  
Step 2: build immagini SSH (Ubuntu + Rocky)

## Comandi rapidi
ansible-galaxy collection install -r requirements.yml
ansible-playbook step1/container-playbook.yml
ansible-playbook step2/build-containers.yml

## Step 3: Creazione ruoli

1. Creazione dei file necessari: 
   - roles/build_images/tasks/main.yml
   - roles/build_images/defaults/main.yml
   - roles/registry/tasks/main.yml
   - roles/registry/defaults/main.yml


2. Parametrizzazione
il suolo è parametrizzato in quanto
    - Le porte non sono hardcoded → 2223, 2224 prese da variabili in defaults/main.yml. 
    - Il nome del registry è una variabile (registry_host, registry_port, registry_name).
    - Le immagini e i rispettivi Dockerfile sono in una lista (images: nel defaults). 
    - La chiave SSH è definita in una variabile (ssh_key). 

3. Dockerfile usati : 
   - formazione_cm/templates/Dockerfile.ubuntu
   - formazione_cm/templates/Dockerfile.rocky

   
