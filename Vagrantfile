# -*- mode: ruby -*-
# # vi: set ft=ruby :
# https://github.com/hashicorp/vagrant/issues/8058

require 'fileutils'

# Defaults for config options defined in CONFIG

# Log the serial consoles of VMs to log/
# Enable by setting value to true, disable with false
# WARNING: Serial logging is known to result in extremely high CPU usage with
# VirtualBox, so should only be used in debugging situations
$serial_logging = true
$num_instances = 2
$instance_name_prefix = "ubuntu"
$enable_serial_logging = false
$share_home = false
$vm_gui = false
$vm_memory = 2048
$vm_cpus = 2
$forwarded_ports = {}
$flavor = "trusty"
$ansible_version = "2.4.2.0"

Vagrant.require_version ">= 1.6.0"

# Vagrant VMs
host_cache_path = File.expand_path('../.cache', __FILE__)
guest_cache_path = '/tmp/vagrant-cache'

::Dir.mkdir(host_cache_path) unless ::Dir.exist?(host_cache_path)

default = {
  :user => ENV['OS_USER'] || 'vagrant',
  :project => File.basename(Dir.getwd),
  :ansible_project_name => ENV['_ANSIBLE_PROJECT_NAME'] || '{{cookiecutter.role_name}}'
}

VM_NODENAME = "#{default[:project]}-#{$flavor}"

puts "DEBUGGER - VM_NODENAME: #{VM_NODENAME}"

$fix_perm = <<SHELL
sudo chmod 600 /home/vagrant/.ssh/id_rsa
SHELL

# SOURCE: https://github.com/bossjones/docker-swarm-vbox-lab/blob/master/Vagrantfile
$docker_script = <<SHELL
if [ -f /vagrant_bootstrap ]; then
   echo "vagrant_bootstrap EXISTS ALREADY"
   exit 0
fi
export DEBIAN_FRONTEND=noninteractive
sudo apt-get autoremove -y && \
sudo apt-get update -yqq && \
sudo apt-get install -yqq software-properties-common \
                   python-software-properties && \
sudo apt-get install -yqq build-essential \
                   libssl-dev \
                   libreadline-dev \
                   wget curl \
                   openssh-server && \
sudo apt-get install -yqq python-setuptools \
                   perl pkg-config \
                   python python-pip \
                   python-dev
sudo easy_install --upgrade pip && \
sudo easy_install --upgrade setuptools; \
sudo pip install setuptools --upgrade; \
sudo add-apt-repository -y ppa:git-core/ppa && \
sudo add-apt-repository -y ppa:ansible/ansible && \
sudo add-apt-repository ppa:chris-lea/python-urllib3 && \
sudo pip install urllib3[secure] && \
sudo add-apt-repository -y ppa:git-core/ppa && \
sudo add-apt-repository -y ppa:ansible/ansible && \
sudo add-apt-repository ppa:chris-lea/python-urllib3 && \
sudo apt-get update -yqq && \
sudo apt-get install -yqq python-urllib3 && \
sudo apt-get install -yqq git lsof strace && \
sudo pip install ansible==#{$ansible_version} && \
sudo chown -R #{default[:user]}:#{default[:user]} /home/#{default[:user]}/ && \
sudo touch /vagrant_bootstrap && \
sudo chown #{default[:user]}:#{default[:user]} /vagrant_bootstrap
SHELL

$git_clone_dotfiles = <<SHELL
git clone https://github.com/bossjones/linux-dotfiles.git /home/#{default[:user]}/.dotfiles
cd /home/#{default[:user]}/.dotfiles/;
git checkout feature-new-dotfiles-layout
sudo chown -R #{default[:user]}:#{default[:user]} /home/#{default[:user]}/
SHELL

# Use old vb_xxx config variables when set
def vm_gui
  $vb_gui.nil? ? $vm_gui : $vb_gui
end

def vm_memory
  $vb_memory.nil? ? $vm_memory : $vb_memory
end

def vm_cpus
  $vb_cpus.nil? ? $vm_cpus : $vb_cpus
end

# ---------------------------------------------------------------------------------------

Vagrant.configure("2") do |config|

  config.ssh.insert_key = false

  # config.vm.box = "ubuntu/xenial64"
  config.vm.box = "ubuntu/trusty64"

  # plugin conflict
  if Vagrant.has_plugin?("vagrant-hostmanager") then
     # enable hostmanager
     config.hostmanager.enabled = true

     # configure the host's /etc/hosts
     config.hostmanager.manage_host = true

    #  config.hostmanager.include_offline = true
  else
    puts "Missing vagrant-hostmanager plugin, please install: "
    puts "https://github.com/devopsgroup-io/vagrant-hostmanager"
    puts
    exit
  end

  # # NOTE(berendt): This solves the Ubuntu-specific Vagrant issue 1673.
  # #                https://github.com/mitchellh/vagrant/issues/1673
  # config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"

  # if Vagrant.has_plugin?("vagrant-hostmanager")
  #   config.hostmanager.enabled = true
  #   config.hostmanager.manage_host = true
  #   config.hostmanager.ignore_private_ip = false
  #   config.hostmanager.include_offline = true
  #   if conf["use_bridge"] == false || conf["use_ip_resolver"] == true
  #     config.hostmanager.ip_resolver = proc do |machine|
  #       result = ""
  #       begin
  #         machine.communicate.execute("ifconfig eth1") do |type, data|
  #           result << data if type == :stdout
  #         end
  #       # NOTE(jerryz): This catches the exception when host is still
  #       # not ssh reachable.
  #       # https://github.com/smdahlen/vagrant-hostmanager/issues/121
  #       rescue
  #         result = "# NOT-UP"
  #       end
  #       (ip = /inet addr:(\d+\.\d+\.\d+\.\d+)/.match(result)) && ip[1]
  #     end
  #   end
  # end

  # SOURCE: https://github.com/openstack-dev/devstack-vagrant/blob/master/Vagrantfile
  # TODO: Enable this?
  # if Vagrant.has_plugin?("vagrant-cachier")
  #   config.cache.scope = :box
  #   # see https://github.com/fgrehm/vagrant-cachier/issues/175
  #   config.cache.synced_folder_opts = {
  #     owner: "_apt",
  #     group: "ubuntu",
  #     mount_options: ["dmode=777", "fmode=666"]
  #   }
  # end


  # TODO: Enable this?
  # SOURCE: https://github.com/openstack-dev/devstack-vagrant/blob/master/Vagrantfile
  # if Vagrant.has_plugin?("vagrant-proxyconf") && conf['proxy']
  #   config.proxy.http     = conf['proxy']
  #   config.proxy.https    = conf['proxy']
  #   config.proxy.no_proxy = "localhost,127.0.0.1,`facter ipaddress_eth1`,#{conf['hostname_manager']},#{conf['hostname_compute']},#{conf['ip_address_compute']},#{conf['ip_address_manager']},#{conf['user_domains']}"
  #   config.vm.provision :shell, :inline => $git_use_https
  # end

  config.vm.hostname = VM_NODENAME

  # ssh
  # config.ssh.max_tries = 40
  # config.ssh.timeout   = 120

#   # Enable SSH agent forwarding for git clones
#   config.ssh.forward_agent = true

  if $enable_serial_logging
    logdir = File.join(File.dirname(__FILE__), "log")
    FileUtils.mkdir_p(logdir)

    serialFile = File.join(logdir, "%s-serial.txt" % VM_NODENAME)
    FileUtils.touch(serialFile)
  end

  config.vm.provider :virtualbox do |vb, override|
    # Give enough horsepower to build without taking all day.
    # NOTE: uart1: serial port
    # INFO: http://wiki.openpicus.com/index.php/UART_serial_port
    vb.customize ["modifyvm", :id, "--uart1", "0x3F8", "4"]
    # INFO: --uartmode<1-N> <arg>: This setting controls how VirtualBox connects a given virtual serial port (previously configured with the --uartX setting, see above) to the host on which the virtual machine is running. As described in detail in Section 3.10, “Serial ports”, for each such port, you can specify <arg> as one of the following options:
    # SOURCE: https://www.virtualbox.org/manual/ch08.html
    # vb.customize ["modifyvm", :id, "--uartmode1", serialFile]
    # ******************************************************************************
    # Fix docker not being able to resolve private registry in VirtualBox
    # ******************************************************************************
    # SOURCE: https://github.com/iflowfor8hours/coreos-jenkins-spike/blob/6da5769c50a98ade62fcf163b98cc100402511da/Vagrantfile
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
    # ******************************************************************************
    vb.customize ["modifyvm", :id, "--ioapic", "on"]
    vb.customize ["modifyvm", :id, "--chipset", "ich9"]
    vb.customize ["modifyvm", :id, "--memory", "2048"]
    vb.customize ["modifyvm", :id, "--cpus", "2"]

    # you need this for openstack guests to talk to each other
    # SOURCE: https://github.com/openstack-dev/devstack-vagrant/blob/master/Vagrantfile
    # vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
    # if specified assign a static MAC address
    # if conf["mac_address_#{name}"]
    #   vb.customize ["modifyvm", :id, "--macaddress2", conf["mac_address_#{name}"]]
    # end

    ip = "172.17.10.101"
    config.vm.network :private_network, ip: ip

    # SOURCE: https://github.com/bossjones/oh-my-fedora24/blob/master/Vagrantfile
    # # network
    # config.vm.network "public_network", :bridge => 'en0: Wi-Fi (AirPort)'
    # config.vm.network "forwarded_port", guest: 19360, host: 1936
    # config.vm.network "forwarded_port", guest: 139, host: 1139
    # config.vm.network "forwarded_port", guest: 8081, host: 8881
    # config.vm.network "forwarded_port", guest: 2376, host: 2376
    # config.vm.network "forwarded_port", guest: 1999, host: 19999

    # # ssh
    # config.ssh.username = "vagrant"
    # config.ssh.host = "127.0.0.1"
    # config.ssh.guest_port = "2222"
    # config.ssh.private_key_path = ENV['HOME'] + '/dev/bossjones/oh-my-fedora24/keys/vagrant_id_rsa'
    # config.ssh.forward_agent = true
    # config.ssh.forward_x11 = true
    # config.ssh.insert_key = false
    # config.ssh.keep_alive  = 5
  end


  # TODO: Enable this??
  # SOURCE: https://github.com/openstack-dev/devstack-vagrant/blob/master/Vagrantfile
  # # NOTE(berendt): This solves the Ubuntu-specific Vagrant issue 1673.
  # #                https://github.com/mitchellh/vagrant/issues/1673
  # config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"

  # *****************
  $forwarded_ports.each do |guest, host|
    config.vm.network "forwarded_port", guest: guest, host: host, auto_correct: true
  end

  config.vm.provider :virtualbox do |vb|
    vb.gui = vm_gui
    vb.memory = vm_memory
    vb.cpus = vm_cpus
  end

  # config.vm.network "private_network", ip: box_ip

  # # set auto_update to false, if you do NOT want to check the correct
  # # additions version when booting this machine
  # config.vbguest.auto_update = true

  # # do NOT download the iso file from a webserver
  # config.vbguest.no_remote = false

  # FIXME: Set this to a real path
  #   public_key = ENV['HOME'] + '/dev/vagrant-box/fedora/keys/vagrant.pub'

  config.vm.provision :shell, inline: $docker_script

  # shared folder setup
  config.vm.synced_folder ".", "/home/vagrant/#{VM_NODENAME}", SharedFoldersEnableSymlinksCreate: false

  # TODO: Investigate if we need these other options
  # SOURCE: https://github.com/iflowfor8hours/coreos-jenkins-spike/blob/6da5769c50a98ade62fcf163b98cc100402511da/Vagrantfile
  # Uncomment below to enable NFS for sharing the host machine into the coreos-vagrant VM.
  # config.vm.synced_folder ".", "/home/core/share", id: "core", :nfs => true, :mount_options => ['nolock,vers=3,udp']

  # copy private key so hosts can ssh using key authentication (the script below sets permissions to 600)
  config.vm.provision :file do |file|
    file.source      = './keys/vagrant_id_rsa'
    file.destination = '/home/vagrant/.ssh/id_rsa'
  end

  # fix permissions on private key file
  config.vm.provision :shell, inline: $fix_perm
  # ******************

  # config.vm.provision "ansible" do |ansible|
  #     ansible.playbook = "playbook.yml"
  #     ansible.verbose = "-v"
  #     ansible.sudo = true
  #     ansible.host_key_checking = false
  #     ansible.limit = 'all'
  #     # ansible.inventory_path = "provisioning/inventory"
  #     ansible.inventory_path = "ubuntu-inventory"
  #     # ansible.sudo = true
  #     # ansible.extra_vars = {
  #     #   public_key: public_key
  #     # }
  #     # Prevent intermittent connection timeout on ssh when provisioning.
  #     # ansible.raw_ssh_args = ['-o ConnectTimeout=120']
  #     # gist: https://gist.github.com/phantomwhale/9657134
  #     # ansible.raw_arguments = Shellwords.shellsplit(ENV['ANSIBLE_ARGS']) if ENV  ['ANSIBLE_ARGS']
  #     # CLI command.
  #     # ANSIBLE_ARGS='--extra-vars "some_var=value"' vagrant up
  # end  # config.vm.provision "ansible" do |ansible|

  # git clone dotfiles repo
  config.vm.provision :shell, inline: $git_clone_dotfiles

  ansible_inventory_dir = "ansible/hosts"

end

# ---------------------------------------------------------------------------------------

