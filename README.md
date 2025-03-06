# Projeto - Provisionamento com Vagrant, Ansible e Docker

Este projeto tem como objetivo criar uma infraestrutura automatizada para hospedar uma aplicação **WordPress**, utilizando as seguintes tecnologias:

- **Vagrant** para criação e gestão da máquina virtual.
- **Ansible** para provisionamento e configuração da VM.
- **Docker e Docker Compose** para criar e orquestrar os containers necessários para a aplicação.

---

## Estrutura do Projeto

```
projeto-samuel/
├── Vagrantfile
├── playbook_ansible.yml
├── docker-compose.yml
├── nginx/
│   ├── Dockerfile
│   └── nginx.conf
└── roles/
    ├── common/
    │   ├── tasks/
    │   │   └── main.yml
    │   └── handlers/
    │       └── main.yml
    └── docker/
        ├── tasks/
        │   └── main.yml
        └── handlers/
            └── main.yml
```

---

## Fluxo do Provisionamento

### 1. Criação da VM com Vagrant

O arquivo `Vagrantfile` define:

- Box base: `roboxes/ubuntu2204`
- Provider: VirtualBox
- IP privado: `192.168.57.10`
- Nome da VM: `SamuelSilva`
- Provisionamento via Ansible, chamando o `playbook_ansible.yml`

### Exemplo do Vagrantfile

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "roboxes/ubuntu2204"
  config.vm.provider "virtualbox" do |vb|
    vb.name = "SamuelSilva"
    vb.memory = "1024"
    vb.cpus = 1
  end
  config.vm.network "private_network", ip: "192.168.57.10"
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook_ansible.yml"
  end
end
```

---

### 2. Provisionamento com Ansible

O Ansible é responsável por:
- Alterar o hostname da VM.
- Instalar pacotes básicos.
- Instalar Docker e Docker Compose.
- Copiar os arquivos necessários (`docker-compose.yml` e diretório `nginx`) para a VM.
- Subir a infraestrutura Docker.

### Organização do Playbook

O playbook principal `playbook_ansible.yml` é bem simples e apenas inclui **roles**.

```yaml
---
- name: Provisionar Servidor Samuel
  hosts: all
  become: true

  roles:
    - role: common
    - role: docker
```

#### Role `common`

Configura o sistema operacional básico, incluindo hostname e pacotes essenciais.

```yaml
- name: Atualizar cache do apt
  apt:
    update_cache: yes

- name: Alterar hostname
  hostname:
    name: server.samuel.silva

- name: Instalar pacotes básicos
  apt:
    name:
      - curl
      - vim
    state: present
```

#### Role `docker`

Instala Docker e Docker Compose, além de copiar os arquivos necessários e subir a stack com Docker Compose.

```yaml
- name: Copiar docker-compose.yml para a VM
  copy:
    src: ../../../docker-compose.yml
    dest: /home/vagrant/docker-compose.yml

- name: Copiar diretório nginx para a VM
  copy:
    src: ../../../nginx/
    dest: /home/vagrant/nginx/

- name: Subir infraestrutura Docker
  command: docker-compose up -d
  args:
    chdir: /home/vagrant/
```

---

### 3. Stack Docker - WordPress + Nginx + MySQL

O arquivo `docker-compose.yml` define:

- **webproxy:** Container baseado em uma imagem personalizada do Nginx com balanceamento de carga de camada 4.
- **webserver:** Container oficial do WordPress.
- **database:** Container oficial do MySQL 5.7.

```yaml
version: '3'

networks:
  wordpress:
    driver: bridge

volumes:
  app:
  my:

services:
  webproxy:
    build: ./nginx
    networks:
      - wordpress
    ports:
      - "8080:8080"
    depends_on:
      - webserver

  webserver:
    image: wordpress:latest
    networks:
      - wordpress
    environment:
      WORDPRESS_DB_HOST: database
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpresspassword
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - app:/var/www/html

  database:
    image: mysql:5.7
    networks:
      - wordpress
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpresspassword
      MYSQL_ROOT_PASSWORD: rootpassword
    volumes:
      - my:/var/lib/mysql
```

---

### 4. Dockerfile Personalizado do Nginx

```Dockerfile
FROM nginx:latest

RUN apt-get update && apt-get install -y iputils-ping curl && apt-get clean

COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 8080
```

---

### 5. Configuração Nginx (nginx.conf)

```nginx
events {}

stream {
    upstream web_backend {
        server webserver:80;
    }

    server {
        listen 8080;
        proxy_pass web_backend;
    }
}
```

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

## Autor

- **Samuel Silva**
- Projeto acadêmico para a disciplina **Administração de Sistemas Abertos**

