

---

### Passo 1: Criar o `Vagrantfile`

1. Abra um terminal ou seu editor de texto favorito.
2. Crie um arquivo chamado `Vagrantfile` no diretório do projeto.
3. Insira o seguinte código no arquivo:

```ruby
Vagrant.configure("2") do |config|
  # Configuração geral
  config.vm.box = "ubuntu/focal64"
  
  # Definição da máquina master
  config.vm.define "master" do |master|
    master.vm.hostname = "master"
    master.vm.network "private_network", ip: "192.168.50.10"
    master.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.cpus = 1
    end
    master.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get install -y docker.io
      docker swarm init --advertise-addr 192.168.50.10
    SHELL
  end

  # Definição dos nodes
  ["node01", "node02", "node03"].each_with_index do |node_name, index|
    config.vm.define node_name do |node|
      node.vm.hostname = node_name
      node.vm.network "private_network", ip: "192.168.50.#{11 + index}"
      node.vm.provider "virtualbox" do |vb|
        vb.memory = "1024"
        vb.cpus = 1
      end
      node.vm.provision "shell", inline: <<-SHELL
        apt-get update
        apt-get install -y docker.io
      SHELL
    end
  end
end
```

---

### Passo 2: Configurar o Cluster Swarm

Para configurar o Swarm automaticamente:
1. Após inicializar o master, você precisa obter o comando de `docker swarm join` com o token.
2. Atualize o script no `Vagrantfile` para incluir os workers no cluster. Edite a seção dos nodes para incluir o seguinte comando:

```ruby
# Adicionar ao final do provisionamento de cada node
node.vm.provision "shell", inline: <<-SHELL
  apt-get update
  apt-get install -y docker.io
  docker swarm join --token [TOKEN] 192.168.50.10:2377
SHELL
```

Substitua `[TOKEN]` pelo token gerado na inicialização do master. Você pode automatizar isso com um script externo ou manualmente.

---

### Passo 3: Iniciar as Máquinas Virtuais

1. No terminal, execute o comando:

```bash
vagrant up
```

2. Espere até que todas as máquinas sejam criadas e configuradas.

---

### Passo 4: Verificar o Cluster

1. Acesse a máquina `master`:

```bash
vagrant ssh master
```

2. Execute o comando para verificar os nós do cluster:

```bash
docker node ls
```

Você deverá ver o `master` como líder e os `node01`, `node02` e `node03` como workers.

---
