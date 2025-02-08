Vagrant.configure("2") do |config|
  nodes = [
    { name: "manager", ip: "192.168.56.100" },
    { name: "worker1", ip: "192.168.56.101" },
    { name: "worker2", ip: "192.168.56.102" }
  ]

  # Définir les machines
  nodes.each do |node|
    config.vm.define node[:name] do |node_config|
      node_config.vm.box = "bento/ubuntu-20.04"
      node_config.vm.provider "vmware_workstation"
      node_config.vm.network "private_network", ip: node[:ip]
      node_config.vm.provision "shell", inline: <<-SHELL
        apt-get update && apt-get install -y docker.io
        usermod -aG docker vagrant
        curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
        chmod +x /usr/local/bin/docker-compose
      SHELL
    end
  end

  # Initialisation du Swarm sur le manager
  config.vm.define "manager" do |manager_config|
    manager_config.vm.provision "shell", inline: <<-SHELL
      # Initialiser Docker Swarm
      docker swarm init --advertise-addr 192.168.56.100
      docker swarm join-token worker -q > /vagrant/worker_token

      # Construire et sauvegarder les images
      docker build -t result:latest /vagrant/result
      docker build -t vote:latest /vagrant/vote
      docker build -t worker:latest /vagrant/worker
      docker save -o /vagrant/result.tar result:latest
      docker save -o /vagrant/vote.tar vote:latest
      docker save -o /vagrant/worker.tar worker:latest
    SHELL
  end

  # Provisionnement des workers
  nodes.drop(1).each do |worker|
    config.vm.define worker[:name] do |worker_config|
      worker_config.vm.provision "shell", inline: <<-SHELL
        # Rejoindre le Swarm du manager
        docker swarm join --token $(cat /vagrant/worker_token) 192.168.56.100:2377

        # Charger les images sur les workers
        docker load -i /vagrant/result.tar
        docker load -i /vagrant/vote.tar
        docker load -i /vagrant/worker.tar
      SHELL
    end
  end

  # Provisionnement final sur le manager
  config.vm.define "manager" do |manager_config|
    manager_config.vm.provision "shell", inline: <<-SHELL
      sleep 30  # Attendre que les workers rejoignent

      # S'assurer que les nœuds sont disponibles
      until docker node ls | grep -q "worker2"; do
        sleep 5
      done

      # Mettre à jour les labels des nœuds
      docker node update --label-add type=worker worker1
      docker node update --label-add type=worker worker2
      docker node update --label-add type=manager manager

      # Créer le réseau overlay
      docker network create --driver overlay app-net || true

      # Déployer la stack
      docker stack deploy -c /vagrant/docker-compose.yml app

      # Vérifier le déploiement
      docker stack ps app
    SHELL
  end
end
