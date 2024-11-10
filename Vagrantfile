# Configuración del Vagrantfile para crear 2 servidores Apache y 1 balanceador Nginx

Vagrant.configure("2") do |config|

    # Configuración de las máquinas
    servers = [
      { :name => "apache1", :ip => "192.168.33.10", :role => "web" },
      { :name => "apache2", :ip => "192.168.33.11", :role => "web" },
      { :name => "nginx",   :ip => "192.168.33.12", :role => "load_balancer" }
    ]
  
    # Configuración global para todas las máquinas
    config.vm.box = "ubuntu/jammy64"
  
    servers.each do |machine|
      config.vm.define machine[:name] do |node|
        node.vm.hostname = machine[:name]
        node.vm.network "private_network", ip: machine[:ip]
        node.vm.provider "virtualbox" do |vb|
          vb.cpus = 1
          vb.memory = 512
        end
  
        if machine[:role] == "web"
          # Instalación de Apache
          node.vm.provision "shell", inline: <<-SHELL
            apt-get update
            apt-get install -y apache2
            echo "<h1>#{machine[:name]}</h1>" > /var/www/html/index.html
          SHELL
  
          # Sincronización de un directorio compartido con la máquina local
          node.vm.synced_folder "./web_content", "/var/www/html", create: true
  
        elsif machine[:role] == "load_balancer"
          # Instalación de Nginx y configuración de balanceo de carga
          node.vm.provision "shell", inline: <<-SHELL
            apt-get update
            apt-get install -y nginx
            cat <<EOL > /etc/nginx/conf.d/load_balancer.conf
            upstream apache_servers {
                server 192.168.33.10;
                server 192.168.33.11;
            }
            server {
                listen 80;
                location / {
                    proxy_pass http://apache_servers;
                }
            }
            EOL
            systemctl restart nginx
          SHELL
        end
      end
    end
  end
  