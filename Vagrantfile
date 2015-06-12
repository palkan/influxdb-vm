# Use rbconfig to determine if we're on a windows host or not.
require 'rbconfig'

host = RbConfig::CONFIG['host_os']
is_windows = (host =~ /mswin|mingw|cygwin/)

Vagrant.configure("2") do |config|
  config.vm.box       = "ubuntu/trusty64"

  ## OS Tune

  # Give VM 1/4 system memory & access to all cpu cores on the host
  if host =~ /darwin/
    cpus = `sysctl -n hw.ncpu`.to_i
    # sysctl returns Bytes and we need to convert to MB
    mem = `sysctl -n hw.memsize`.to_i / 1024 / 1024 / 4
  elsif host =~ /linux/
    cpus = `nproc`.to_i
    # meminfo shows KB and we need to convert to MB
    mem = `grep 'MemTotal' /proc/meminfo | sed -e 's/MemTotal://' -e 's/ kB//'`.to_i / 1024 / 4
  else # sorry Windows folks, I can't help you
    cpus = 2
    mem = 1024
  end

  config.vm.provider "virtualbox" do |v|
    v.customize ["modifyvm", :id, "--memory", mem]
    v.customize ["modifyvm", :id, "--cpus", cpus]
  end

  if is_windows
    # Share folder only under windows
    config.vm.synced_folder ".", "/priv"
    
    # Provisioning configuration for shell script.
    config.vm.provision "shell" do |sh|
      sh.path = "priv/win.sh"
      sh.args = "dev.yml dev_hosts"
    end
  else
    config.vm.provision "ansible" do |ansible|
      ENV["ANSIBLE_CONFIG"] = "ansible.cfg"
      ansible.playbook = "dev.yml"
      ansible.inventory_path = "dev_hosts"
      ansible.tags = ENV["TAGS"]
      ansible.skip_tags = ENV["SKIP_TAGS"]
      ansible.verbose = 'v'
      ansible.limit = "all"
    end
  end

  config.vm.network :forwarded_port, host: 2209, guest: 22 

  # InfluxDB admin interface
  config.vm.network :forwarded_port, host: 8083, guest: 8083 

  # InfluxDB HTTP API endpoint
  config.vm.network :forwarded_port, host: 8086, guest: 8086 

  # InfluxDB UDP endpoint
  config.vm.network :forwarded_port, host: 9001, guest: 9001 

  config.vm.network "private_network", ip: "192.168.50.199"

  config.vm.synced_folder ".", "/vagrant", disabled: true
end