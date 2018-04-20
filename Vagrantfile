## -*- mode: ruby -*-
## vi: set ft=ruby :

$boxes = [
  {
    :name => :web1,
    :group => "web",
    :ip => "192.168.33.54",
    :forwards => { 80 => 1080 }
  }
]

Vagrant.configure("2") do |config|
  $groups = { "all" => [] }
  $boxes.each do | opts |
    if ! $groups.has_key?(opts[:group])
      $groups[opts[:group]] = [ opts[:name] ]
    else
      $groups[opts[:group]].push(opts[:name])
    end
    $groups["all"].push(opts[:name])
  end

  config.vm.provider :virtualbox do |vb, override|
    override.vm.box = "ubuntu/xenial64"
    vb.memory = "1024"
    vb.customize [ "modifyvm", :id, "--uartmode1", "disconnected" ]
  end

  $boxes.each_with_index do | opts, index |
    config.vm.define(opts[:name]) do |config|
      config.vm.hostname =   "%s" % [ opts[:name].to_s ]
      config.vm.network "private_network", ip: "%s" % [ opts[:ip].to_s ]
      opts[:forwards].each do |guest_port,host_port|
        config.vm.network :forwarded_port, guest: guest_port, host: host_port
      end if opts[:forwards]

      # configure with ansible only on the last box for the all the hosts at once
      if index == $boxes.size - 1
        config.vm.provision :ansible do |ansible|
          # ansible.verbose = true
          ansible.playbook = "ansible/playbook.yml"
          ansible.groups = $groups
          ansible.become = true
          ansible.limit = "all"
          ansible.extra_vars = { ansible_ssh_user: 'vagrant' }
        end
      end
    end if ! opts[:disabled]
  end
end
