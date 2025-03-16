# Projeto - Provisionamento com Vagrant, Ansible e Docker

Este projeto tem como objetivo criar uma infraestrutura automatizada para hospedar uma aplicação **WordPress**, utilizando as seguintes tecnologias:

- **Vagrant** para criação e gestão da máquina virtual.
- **Ansible** para provisionamento e configuração da VM.
- **Docker e Docker Compose** para criar e orquestrar os containers necessários para a aplicação.

---

## Estrutura do Projeto

> **Nota:** A pasta `nginx/` está listada na estrutura, mas não é utilizada no provisionamento atual. Isso ocorre porque a imagem personalizada do Nginx já está publicada no Docker Hub e é puxada diretamente pelo `docker-compose.yml`, eliminando a necessidade do Dockerfile e das configurações locais.

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

## Fluxo do Provisionamento

### 1. Criação da VM com Vagrant

O arquivo `Vagrantfile` define:

- Box base: `roboxes/ubuntu2204`
- Provider: VirtualBox
- IP privado: `192.168.57.10`
- Nome da VM: `SamuelIsabel`
- Provisionamento via Ansible, chamando o `playbook.yml`

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
