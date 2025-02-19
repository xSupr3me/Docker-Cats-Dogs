Vagrant.configure("2") do |config|
  nodes = {
    'manager' => '11',
    'worker1' => '12',
    'worker2' => '13'
  }

  nodes.each do |node_name, ip_end|
    config.vm.define node_name do |node|
      node.vm.box = "bento/ubuntu-20.04"
      node.vm.hostname = node_name
      node.vm.network "private_network", ip: "192.168.33.#{ip_end}"
      
      if node_name == 'manager'
        node.vm.network "forwarded_port", guest: 8080, host: 8080
        node.vm.network "forwarded_port", guest: 8888, host: 8888
      end

      node.vm.provider "vmware_desktop" do |v|
        v.memory = 2048
        v.cpus = 2
      end

      node.vm.provision "shell", inline: <<-SHELL
        apt-get update
        apt-get install -y apt-transport-https ca-certificates curl software-properties-common
        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
        add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
        apt-get update
        apt-get install -y docker-ce docker-ce-cli containerd.io
        usermod -aG docker vagrant
      SHELL

      if node_name == 'manager'
        node.vm.provision "shell", inline: <<-SHELL
          docker swarm init --advertise-addr 192.168.33.11
          
          docker swarm join-token worker -q > /vagrant/worker_token
          
          cd /vagrant
          docker build -t worker:latest ./worker/
          docker build -t vote:latest ./vote/
          docker build -t result:latest ./result/
          
          docker network create --driver overlay app-net
          
          cp /vagrant/docker-compose.yml /home/vagrant/docker-compose.yml

          cat > /home/vagrant/deploy.sh <<'EOF'
#!/bin/bash
# Attendre que tous les nœuds rejoignent le cluster
while [ $(docker node ls --format '{{.Hostname}}' | wc -l) -lt 3 ]; do
  echo "En attente de tous les nœuds..."
  sleep 5
done

# Configurer les labels
docker node update --label-add type=manager manager
docker node update --label-add type=worker worker1
docker node update --label-add type=worker worker2

# Déployer la stack
docker stack deploy -c /home/vagrant/docker-compose.yml myapp
EOF

          chmod +x /home/vagrant/deploy.sh
        SHELL
      elsif node_name.start_with?('worker')
        node.vm.provision "shell", inline: <<-SHELL
          while [ ! -f /vagrant/worker_token ]; do sleep 1; done
          docker swarm join --token $(cat /vagrant/worker_token) 192.168.33.11:2377
          cd /vagrant
          docker build -t worker:latest ./worker/
          docker build -t vote:latest ./vote/
          docker build -t result:latest ./result/
        SHELL
      end

      if node_name == 'worker2'
        node.vm.provision "shell", run: "always", inline: <<-SHELL
          sleep 10
          
          ssh -i /vagrant/.vagrant/machines/manager/vmware_desktop/private_key \
              -o StrictHostKeyChecking=no \
              -o UserKnownHostsFile=/dev/null \
              vagrant@192.168.33.11 \
              "bash /home/vagrant/deploy.sh"
          
          rm -f /vagrant/worker_token
        SHELL
      end
    end
  end
end
