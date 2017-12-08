# vi: set ft=ruby :
VAGRANTFILE_API_VERSION = '2'
Vagrant.require_version '>= 1.8.2'

CURRENT_DIR = File.expand_path(File.dirname(__FILE__))
DIRNAME     = File.basename(CURRENT_DIR)

hosts = [
    #10.10.10.1 is configured as bridged between the host and 10.10.1.x guests
    {
        :name  => "haproxy-01.example.com",
        :box   => "minos/core-16.04",
        :ram   => "256", :cpus  => "1",
        :ip    => "10.10.10.11",
    },
    {
        :name  => "haproxy-02.example.com",
        :box   => "minos/core-16.04",
        :ram   => "256", :cpus  => "1",
        :ip    => "10.10.10.12",
    },
    {
        :name  => "haproxy-03.example.com",
        :box   => "minos/core-16.04",
        :ram   => "256", :cpus  => "1",
        :ip    => "10.10.10.13",
    },
{
        :name  => "haproxy-04.example.com",
        :box   => "minos/core-16.04",
        :ram   => "256", :cpus  => "1",
        :ip    => "10.10.10.14",
    },
{
        :name  => "haproxy-05.example.com",
        :box   => "minos/core-16.04",
        :ram   => "256", :cpus  => "1",
        :ip    => "10.10.10.15",
    },
    {
        

        
{
        :name  => "nginx-03.example.com",
        :box   => "minos/core-16.04",
        :ram   => "256", :cpus  => "1",
        :ip    => "10.10.10.16",
    },

        :name  => "nginx-01.example.com",
        :box   => "minos/core-16.04",
        :ram   => "256", :cpus  => "1",
        :ip    => "10.10.10.17",
    },
    {
        :name  => "nginx-02.example.com",
        :box   => "minos/core-16.04",
        :ram   => "256", :cpus  => "1",
        :ip    => "10.10.10.18",
    },
    {
        :name  => "nginx-03.example.com",
        :box   => "minos/core-16.04",
        :ram   => "256", :cpus  => "1",
        :ip    => "10.10.10.19",
    },

{
        :name  => "nginx-03.example.com",
        :box   => "minos/core-16.04",
        :ram   => "256", :cpus  => "1",
        :ip    => "10.10.10.20",
    },

]

host_os  = RbConfig::CONFIG['host_os']
if host_os =~ /linux/
    all_cpus = `nproc`.to_i
elsif host_os =~ /darwin/
    all_cpus = `sysctl -n hw.ncpu`.to_i
else #windows?
    all_cpus = `wmic cpu get NumberOfCores`.split("\n")[2].to_i
end

default_ram  = '512' #MB
default_cpu  = '50'  #%
default_cpus = all_cpus || '1'

#raise "vagrant-hostmanager plugin must be installed: $ vagrant plugin install vagrant-hostmanager" unless Vagrant.has_plugin? "vagrant-hostmanager"

host_counter = 0; Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    hosts.each do |host|
        config.vm.define host[:name] do |machine|
            machine.vm.box      = host[:box]
            machine.vm.box_url  = host[:box_url] if host[:box_url]
            machine.vm.hostname = host[:name]

            machine.vm.network :private_network, ip: host[:ip]

            machine.vm.provider "virtualbox" do |vbox|
                vbox.name = host[:name]
                vbox.linked_clone = true
                vbox.customize ["modifyvm", :id, "--memory", host[:ram] || default_ram ]          #MB
                vbox.customize ["modifyvm", :id, "--cpuexecutioncap", host[:cpu] || default_cpu ] #%
                vbox.customize ["modifyvm", :id, "--cpus", host[:cpus] || default_cpus ]
            end

            #echo cmds, lambda syntax: http://stackoverflow.com/questions/8476627/what-do-you-call-the-operator-in-ruby
            #why not UPPERCASE?: https://ruby-doc.org/docs/ruby-doc-bundle/UsersGuide/rg/constants.html
            cmd_script_root        = -> (cmd) { machine.vm.provision 'shell', path:   cmd, name: cmd, privileged: true  }
            cmd_script             = -> (cmd) { machine.vm.provision 'shell', path:   cmd, name: cmd, privileged: false }
            cmd_inline_root        = -> (cmd) { machine.vm.provision 'shell', inline: cmd, name: cmd, privileged: true  }
            cmd_inline             = -> (cmd) { machine.vm.provision 'shell', inline: cmd, name: cmd, privileged: false }
            cmd_script_always_root = -> (cmd) { machine.vm.provision 'shell', path:   cmd, name: cmd, run: "always", privileged: false }
            cmd_script_always      = -> (cmd) { machine.vm.provision 'shell', path:   cmd, name: cmd, run: "always", privileged: false }

            #authorize default public ssh key
            cmd_inline_root.call("mkdir -p /root/.ssh/")
            cmd_inline.call     ("mkdir -p /home/vagrant/.ssh/")
            if File.file?("#{Dir.home}/.ssh/id_rsa.pub")
                ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
                cmd_inline_root.call("printf '\\n%s\\n' '#{ssh_pub_key}' >> /root/.ssh/authorized_keys")
                cmd_inline.call     ("printf '\\n%s\\n' '#{ssh_pub_key}' >> /home/vagrant/.ssh/authorized_keys")
            end

            #copy private ssh key
            if File.file?("#{Dir.home}/.ssh/id_rsa")
                machine.vm.provision "file",  source: "~/.ssh/id_rsa", destination: "/home/vagrant/.ssh/id_rsa"
                cmd_inline.call("chown vagrant:vagrant /home/vagrant/.ssh/id_rsa")
                cmd_inline.call("chmod 600 /home/vagrant/.ssh/id_rsa")
            end

            #copy gitconfig
            if File.file?("#{Dir.home}/.gitconfig")
                machine.vm.provision "file",  source: "~/.gitconfig", destination: "/home/vagrant/.gitconfig"
            end

            #provision
            if machine.vm.hostname =~ /haproxy/
                cmd_script_root.call("#{CURRENT_DIR}/provision/00-apt-update.sh")
                cmd_script_root.call("#{CURRENT_DIR}/provision/01-setup-haproxy.sh")
                cmd_script_root.call("#{CURRENT_DIR}/provision/01-setup-keepalived.sh")
            elsif machine.vm.hostname =~ /nginx/
                cmd_script_root.call("#{CURRENT_DIR}/provision/00-apt-update.sh")
                cmd_script_root.call("#{CURRENT_DIR}/provision/01-setup-nginx.sh")
            end

            host_counter = host_counter + 1
        end
    end
    config.hostmanager.enabled      = true
    config.hostmanager.manage_host  = true
    config.hostmanager.manage_guest = true
end
