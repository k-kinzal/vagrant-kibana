# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  config.vm.hostname = "vagrant"

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = "CentOS 6.5 x86_64"

  # The url from where the 'config.vm.box' box will be fetched if it
  # doesn't already exist on the user's system.
  config.vm.box_url = "https://github.com/2creatives/vagrant-centos/releases/download/v6.5.1/centos65-x86_64-20131205.box"

  config.vm.provider "virtualbox" do |v|
    v.memory = 1024
  end

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  config.vm.network :forwarded_port, guest: 9200, host: 9200 # ElasticSearch
  config.vm.network :forwarded_port, guest: 5601, host: 5601 # Kibana4

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  config.vm.network :public_network

  # config.vm.provision "file", source: "./config/td-agent.conf", destination: "/etc/td-agent/td-agent.conf"
  config.vm.provision "file", source: "./config/.bigqueryrc", destination: "~/.bigqueryrc"
  config.vm.provision "file", source: "./config/.bigquery.v2.token", destination: "~/.bigquery.v2.token"

  # If true, then any SSH connections made will enable agent forwarding.
  # Default value: false
  # config.ssh.forward_agent = true

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider :virtualbox do |vb|
  #   # Don't boot with headless mode
  #   vb.gui = true
  #
  #   # Use VBoxManage to customize the VM. For example to change memory:
  #   vb.customize ["modifyvm", :id, "--memory", "1024"]
  # end
  #
  # View the documentation for the provider you're using for more
  # information on available options.

  config.vm.provision :shell, :inline => <<-EOS
    chef-solo --version | grep "Chef: 11.*.*" || curl -sS -L https://www.opscode.com/chef/install.sh | sudo bash
  EOS

  # The path to the Berksfile to use with Vagrant Berkshelf
  # config.berkshelf.berksfile_path = "./Berksfile"

  # Enabling the Berkshelf plugin. To enable this globally, add this configuration
  # option to your ~/.vagrant.d/Vagrantfile file
  config.berkshelf.enabled = true

  # Enable provisioning with chef solo, specifying a cookbooks path, roles
  # path, and data_bags path (all relative to this Vagrantfile), and adding
  # some recipes and/or roles.
  #
  config.vm.provision :chef_solo do |chef|
    chef.custom_config_path = "Vagrantfile.chef"

    chef.json = {
      :ark => {
        :package_dependencies => ["libtool", "autoconf"]
      },
      :java => {
        :install_flavor => "openjdk",
        :jdk_version => "7"
      },
      :elasticsearch => {
        :version => "1.4.0.Beta1",
        :path => {
          :conf => "/etc/elasticsearch",
          :data => "/var/data/elasticsearch",
          :logs => "/var/log/elasticsearch",
        },
        :pid_path => "/var/run",
        :plugins => {
          :'elasticsearch/kibana3' => {:url => "https://download.elasticsearch.org/kibana/kibana/kibana-3.1.1.zip"},
          # :'elasticsearch/kibana4' => {:url => "https://download.elasticsearch.org/kibana/kibana/kibana-4.0.0-BETA1.zip"},
        }
      },
      :td_agent => {
        :plugins => [
          "elasticsearch",
          "cloudwatch-logs"
        ]
      }
    }

    chef.run_list = [
        "recipe[yum]",
        "recipe[yum-epel]",
        "recipe[yum-remi]",
        "recipe[curl]",
        "recipe[curl::libcurl]",
        "recipe[java]",
        "recipe[elasticsearch]",
        "recipe[elasticsearch::plugins]",
        "recipe[td-agent]",
        "recipe[selinux::disabled]",
        "recipe[iptables::disabled]"
    ]
  end

  # TODO: export enviroment variable
  config.vm.provision "shell", path: "./config/.env"

  # TODO: install google cloud sdk
  config.vm.provision :shell, :inline => <<-EOS
    which bq | grep "/usr/bin/bq" || (
      sudo mkdir -p /opt/gcloud && cd /opt/gcloud && sudo curl -sSO https://google-bigquery-tools.googlecode.com/files/bigquery-2.0.17.zip && sudo unzip bigquery-2.0.17.zip && cd bigquery-2.0.17 && sudo python setup.py install && easy_install argparse)
  EOS

  # TODO: install kibana 4.0 beta
  config.vm.provision :shell, :inline => <<-EOS
    which kibana | grep "/usr/bin/kibana" || (
      sudo mkdir -p /opt/kibana && cd /opt/kibana && sudo curl -sSO https://download.elasticsearch.org/kibana/kibana/kibana-4.0.0-BETA1.1.zip && sudo unzip kibana-4.0.0-BETA1.1.zip && cd ./kibana-4.0.0-BETA1.1 && (sudo ./bin/kibana >/dev/null 2>&1 &) && sudo ln -s /opt/kibana/kibana-4.0.0-BETA1/bin/kibana /usr/bin/kibana)
  EOS

  config.vm.provision :shell, :inline => <<-EOS
    echo '
####
## Output descriptions:
##
<match **>
  type elasticsearch
  type_name fluentd
  include_tag_key true
  tag_key @log_name
  host localhost
  port 9200
  logstash_format true
  flush_interval 10s
</match>

####
## Source descriptions:
##

## built-in TCP input
## @see http://docs.fluentd.org/articles/in_forward
<source>
  type forward
  port 24224
  bind 0.0.0.0
</source>

## built-in UNIX socket input
#<source>
#  type unix
#</source>


## live debugging agent
<source>
  type debug_agent
  bind 127.0.0.1
  port 24230
</source>
    ' | sudo tee /etc/td-agent/td-agent.conf
  EOS

  config.vm.provision :shell, :inline => <<-EOS
    sudo service elasticsearch restart
    sudo service td-agent restart
  EOS
end
