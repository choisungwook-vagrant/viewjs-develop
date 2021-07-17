require 'yaml'

CONFIG = YAML.load_file(File.join(File.dirname(__FILE__), 'config.yml'))

Vagrant.configure("2") do |config|
  config.ssh.insert_key = false

  config.vm.define CONFIG['server']['name'] do |cfg|
    cfg.vm.box = CONFIG['server']['box']
    cfg.vm.network "public_network", ip: CONFIG['server']['ip']
    cfg.vm.network "forwarded_port", guest: 8080, host: 9090
    cfg.vm.hostname = CONFIG['server']['hostname']
    
    cfg.vm.provider "virtualbox" do |v|
      v.memory = CONFIG['server']['memory']
      v.cpus = CONFIG['server']['cpu']
      v.name = CONFIG['server']['name']
    end

    # ssh 비밀번호인증 활성화
    cfg.vm.provision "shell", inline: <<-SCRIPT
      sed -i -e 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
      systemctl restart sshd
    SCRIPT

    # nodejs, npm설치
    cfg.vm.provision "shell", inline: <<-SCRIPT
      apt update -y
      curl -sL https://deb.nodesource.com/setup_14.x | sudo bash -
      apt install -y \
        tmux \
        nodejs \
        net-tools \
        vim
    SCRIPT

    # vue, vue-cli 설치
    cfg.vm.provision "shell", privileged: false, inline: <<-SCRIPT
      mkdir /home/vagrant/.npm-global
      npm config set prefix /home/vagrant/.npm-global
      echo "export PATH=~/.npm-global/bin:$PATH" >> /home/vagrant/.bashrc
      npm install -g @vue/cli
      echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p
    SCRIPT
  end
end
