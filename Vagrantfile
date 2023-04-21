Vagrant.configure("2") do |config|

# Uncomment this depending on the provider you want to use
ENV['VAGRANT_DEFAULT_PROVIDER'] = 'virtualbox'
# ENV['VAGRANT_DEFAULT_PROVIDER'] = 'vmware_desktop'

boxes = [
  # windows server 2022 : don't work for now
  #{ :name => "DC01",  :ip => "192.168.56.10", :box => "StefanScherer/windows_2022", :box_version => "2021.08.23", :os => "windows"},
  # windows server 2019
  { :name => "DC01",  :ip => "192.168.56.10", :box => "StefanScherer/windows_2019", :box_version => "2021.05.15", :os => "windows"},
#https://software-download.microsoft.com/download/pr/17763.737.190906-2324.rs5_release_svc_refresh_SERVER_EVAL_x64FRE_en-us_1.iso
  # windows server 2019
  { :name => "DC02",  :ip => "192.168.56.11", :box => "StefanScherer/windows_2019", :box_version => "2021.05.15", :os => "windows"},
  # windows server 2016
  { :name => "DC03",  :ip => "192.168.56.12", :box => "StefanScherer/windows_2016", :box_version => "2017.12.14", :os => "windows"},
# http://care.dlservice.microsoft.com/dl/download/1/4/9/149D5452-9B29-4274-B6B3-5361DBDA30BC/14393.0.161119-1705.RS1_REFRESH_SERVER_EVAL_X64FRE_EN-US.ISO
  # windows server 2019
  #{ :name => "SRV01", :ip => "192.168.56.21", :box => "StefanScherer/windows_2019", :box_version => "2020.07.17", :os => "windows"},
  # windows server 2019
  { :name => "SRV02", :ip => "192.168.56.22", :box => "StefanScherer/windows_2019", :box_version => "2020.07.17", :os => "windows"},
# https://software-download.microsoft.com/download/pr/17763.737.190906-2324.rs5_release_svc_refresh_SERVER_EVAL_x64FRE_en-us_1.iso
  # windows server 2016
  { :name => "SRV03", :ip => "192.168.56.23", :box => "StefanScherer/windows_2016", :box_version => "2019.02.14", :os => "windows"}
# https://software-download.microsoft.com/download/pr/Windows_Server_2016_Datacenter_EVAL_en-us_14393_refresh.ISO
  # ELK
# { :name => "elk", :ip => "192.168.56.50", :box => "bento/ubuntu-18.04", :os => "linux",
#   :forwarded_port => [
#     {:guest => 22, :host => 2210, :id => "ssh"}
#   ]
# }
]

# BUILD with a full up to date vm if you don't want version with old vulns 
# ansible versions boxes : https://app.vagrantup.com/jborean93
# boxes = [
#   # windows server 2019
#   { :name => "DC01",  :ip => "192.168.56.10", :box => "jborean93/WindowsServer2019", :os => "windows"},
#   # windows server 2019
#   { :name => "DC02",  :ip => "192.168.56.11", :box => "jborean93/WindowsServer2019", :os => "windows"},
#   # windows server 2016
#   { :name => "DC03",  :ip => "192.168.56.12", :box => "jborean93/WindowsServer2016", :os => "windows"},
#   # windows server 2019
#   { :name => "SRV02", :ip => "192.168.56.22", :box => "jborean93/WindowsServer2019", :os => "windows"},
#   # windows server 2016
#   { :name => "SRV03", :ip => "192.168.56.23", :box => "jborean93/WindowsServer2016", :os => "windows"}
# ]

  config.vm.provider "virtualbox" do |v|
    v.memory = 4000
    v.cpus = 2
  end

  config.vm.provider "vmware_desktop" do |v|
    v.vmx["memsize"] = "4000"
    v.vmx["numvcpus"] = "2"
  end

  # disable rdp forwarded port inherited from StefanScherer box
  config.vm.network :forwarded_port, guest: 3389, host: 3389, id: "rdp", auto_correct: true, disabled: true

  # no autoupdate if vagrant-vbguest is installed
  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end

  config.vm.boot_timeout = 600
  config.vm.graceful_halt_timeout = 600
  config.winrm.retry_limit = 30
  config.winrm.retry_delay = 10

  boxes.each do |box|
    config.vm.define box[:name] do |target|
      # BOX
      target.vm.provider "virtualbox" do |v|
        v.name = box[:name]
      end
      target.vm.box_download_insecure = box[:box]
      target.vm.box = box[:box]
      if box.has_key?(:box_version)
        target.vm.box_version = box[:box_version]
      end

      # issues/49
      target.vm.synced_folder '.', '/vagrant', disabled: true

      # IP
      target.vm.network :private_network, ip: box[:ip]

      # OS specific
      if box[:os] == "windows"
        target.vm.guest = :windows
        target.vm.communicator = "winrm"
        target.vm.provision :shell, :path => "vagrant/Install-WMF3Hotfix.ps1", privileged: false
        target.vm.provision :shell, :path => "vagrant/ConfigureRemotingForAnsible.ps1", privileged: false

        # fix ip for vmware
        if ENV['VAGRANT_DEFAULT_PROVIDER'] == "vmware_desktop"
          target.vm.provision :shell, :path => "vagrant/fix_ip.ps1", privileged: false, args: box[:ip]
        end

      else
        target.vm.communicator = "ssh"
      end

      if box.has_key?(:forwarded_port)
        # forwarded port explicit
        box[:forwarded_port] do |forwarded_port|
          target.vm.network :forwarded_port, guest: forwarded_port[:guest], host: forwarded_port[:host], host_ip: "127.0.0.1", id: forwarded_port[:id]
        end
      end

    end
  end
end
