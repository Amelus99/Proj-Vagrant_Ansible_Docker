# Projeto - Provisionamento com Vagrant, Ansible e Docker

**Introdução**

Neste projeto, será implementado um ambiente automatizado para a implantação do WordPress utilizando ferramentas de provisionamento e conteinerização, como Vagrant, Ansible e Docker. O objetivo é criar um servidor configurado inteiramente via código, garantindo uma infraestrutura replicável e eficiente.

A implementação seguirá um fluxo estruturado em três etapas: primeiro, a criação de uma Máquina Virtual com Vagrant, seguida da configuração automatizada com Ansible, e, por fim, a construção de uma infraestrutura baseada em Docker para a hospedagem do WordPress.

A VM será provisionada com Ubuntu 22.04, configurada com um endereço IP privado e um nome específico. O Ansible será responsável por instalar o Docker e executar um docker-compose, que criará três containers interconectados: um proxy reverso Nginx, um servidor WordPress e um banco de dados MySQL.

Além disso, será desenvolvida uma imagem personalizada do Nginx, que atuará como balanceador de carga de camada 4, garantindo um melhor desempenho no encaminhamento de requisições. Todo o processo será testado acessando o WordPress por meio do endereço http://192.168.57.10:8080.

Esse projeto demonstra a aplicação prática de ferramentas de automação para o provisionamento de servidores e a gestão de aplicações conteinerizadas, reforçando conceitos essenciais de infraestrutura como código (IaC).

---

##Estrutura do Projeto
A estrutura do projeto segue uma organização clara para facilitar o provisionamento automatizado da infraestrutura. Os principais arquivos responsáveis pela configuração do ambiente são:

Vagrantfile – Define a criação e configuração da Máquina Virtual.
playbook.yml – Contém as instruções do Ansible para instalação e configuração do ambiente.
docker-compose.yml – Orquestra os containers do WordPress, MySQL e Nginx.
Além disso, há uma pasta nginx/, que originalmente conteria arquivos para a construção da imagem personalizada do Nginx. No entanto, como essa imagem já foi publicada no Docker Hub, o docker-compose.yml a utiliza diretamente, tornando desnecessário o uso do Dockerfile e do nginx.conf localmente.

```
projeto-samuel/
├── Vagrantfile
├── playbook.yml
├── docker-compose.yml
├── nginx/
│   ├── Dockerfile
│   └── nginx.conf

```

---

##1. Criação da Máquina Virtual com Vagrant

O arquivo Vagrantfile é responsável por definir e configurar a Máquina Virtual que servirá como base para o ambiente. Ele estabelece os seguintes parâmetros:

Box Base: roboxes/ubuntu2204 (Ubuntu 22.04)
Provider: VirtualBox
IP Privado: 192.168.57.10 (para acesso direto à aplicação)
Nome da VM: SamuelIsabel
Provisionamento Automatizado: Chama o Ansible para executar o playbook.yml, que realizará a configuração do sistema e a implantação dos containers.
Essa abordagem garante um ambiente padronizado e replicável, reduzindo a necessidade de configurações manuais e facilitando a implantação do WordPress.

### Exemplo do Vagrantfile

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "roboxes/ubuntu2204"
  config.vm.provider "virtualbox" do |vb|
    vb.name = "SamuelIsabel"
    vb.memory = "1024"
    vb.cpus = 1
  end
  config.vm.network "private_network", ip: "192.168.57.10"
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yml"
  end
end
```

---

### 2. Provisionamento com Ansible

O Ansible é responsável por:

- Alterar o hostname da VM.
- Instalar pacotes básicos.
- Instalar Docker e Docker Compose.
- Copiar os arquivos necessários (`docker-compose.yml`) para a VM.
- Subir a infraestrutura Docker.

O **playbook.yml** contém todas as tarefas:

```yaml
---
- name: Provisionar Servidor Completo
  hosts: all
  become: true

  tasks:
    - name: Atualizar cache do apt
      apt:
        update_cache: yes

    - name: Alterar hostname
      hostname:
        name: server.samuel.Isabel

    - name: Instalar pacotes básicos
      apt:
        name:
          - curl
          - vim
        state: present

    - name: Instalar dependências do Docker
      apt:
        name:
          - apt-transport-https
          - ca-certificates
          - curl
          - software-properties-common
        state: present

    - name: Adicionar chave GPG do Docker
      apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present

    - name: Adicionar repositório oficial do Docker
      apt_repository:
        repo: "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
        state: present
        filename: docker

    - name: Atualizar cache do apt após adicionar repositório
      apt:
        update_cache: yes

    - name: Instalar Docker e Docker Compose
      apt:
        name:
          - docker-ce
          - docker-ce-cli
          - containerd.io
          - docker-compose
        state: present

    - name: Iniciar e habilitar serviço do Docker
      systemd:
        name: docker
        state: started
        enabled: yes

    - name: Adicionar usuário vagrant ao grupo docker
      user:
        name: vagrant
        groups: docker
        append: yes

    - name: Copiar docker-compose.yml para a VM
      copy:
        src: docker-compose.yml
        dest: /home/vagrant/docker-compose.yml

    - name: Subir infraestrutura Docker
      command: docker-compose up -d
      args:
        chdir: /home/vagrant/
```

---

### 3. Stack Docker - WordPress + Nginx + MySQL

O arquivo `docker-compose.yml` define:

- **webproxy:** Container baseado em uma imagem personalizada do Nginx com balanceamento de carga de camada 4, publicada no Docker Hub.
- **webserver:** Container oficial do WordPress.
- **database:** Container oficial do MySQL 5.7.

A imagem personalizada do Nginx está publicada em:

[https://hub.docker.com/r/amelus99/nginx-lb](https://hub.docker.com/r/amelus99/nginx-lb)

---

## Teste Final

Após subir o ambiente com:

```bash
vagrant up
```

A aplicação WordPress estará disponível em:

[http://192.168.57.10:8080](http://192.168.57.10:8080)

---

## Observações Finais

- O provisionamento é totalmente automatizado.
- O Docker Compose é iniciado automaticamente durante o provisionamento.
- Toda a infraestrutura pode ser destruída com:

```bash
vagrant destroy
```

E recriada com:

```bash
vagrant up
```

---

## Autores

- **Samuel Silva**
- **Maria Isabel Saturnino**
- Projeto acadêmico para a disciplina **Administração de Sistemas Abertos**
