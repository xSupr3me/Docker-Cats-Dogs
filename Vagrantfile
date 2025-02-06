Vagrant.configure("2") do |config|
  # Common configuration
  config.vm.box = "bento/ubuntu-20.04"
  
  # Array of node names
  nodes = ["manager", "worker1", "worker2"]

  # Configure nodes
  nodes.each_with_index do |node_name, i|
    config.vm.define node_name do |node|
      # Network configuration
      node.vm.hostname = node_name
      node.vm.network "public_network", ip: "192.168.250.#{i + 1}"
      
      # Port forwarding for applications
      node.vm.network "forwarded_port", guest: 8080, host: "#{8080 + i}"
      node.vm.network "forwarded_port", guest: 8888, host: "#{8888 + i}"

      # VM configuration for VMware
      node.vm.provider "vmware_desktop" do |vmw|
        vmw.memory = 2048
        vmw.cpus = 2
        vmw.gui = true
      end

      # Install Docker
      node.vm.provision "shell", inline: <<-SHELL
        apt-get update
        apt-get install -y apt-transport-https ca-certificates curl software-properties-common
        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
        add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
        apt-get update
        apt-get install -y docker-ce docker-ce-cli containerd.io
      SHELL

      # Configure Swarm
      if node_name == "manager"
        node.vm.provision "shell", inline: <<-SHELL
          # Initialize Swarm
          docker swarm init --advertise-addr 192.168.250.1
          docker swarm join-token worker > /vagrant/join-token.sh

          # Add vagrant user to docker group
          usermod -aG docker vagrant

          # Copy project files and build images
          mkdir -p /home/vagrant/app
          cp -r /vagrant/* /home/vagrant/app/
          cd /home/vagrant/app

          # Build images
          docker build -t voting-worker ./worker/
          docker build -t voting-vote ./vote/
          docker build -t voting-result ./result/

          # Wait a bit for the swarm to stabilize
          sleep 10

          # Deploy stack
          docker stack deploy -c docker-compose.yml votingapp
        SHELL
      else
        node.vm.provision "shell", inline: <<-SHELL
          # Add vagrant user to docker group
          usermod -aG docker vagrant
          
          # Join Swarm
          bash /vagrant/join-token.sh
        SHELL
      end
    end
  end
end
