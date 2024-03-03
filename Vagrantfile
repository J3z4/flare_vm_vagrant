require 'yaml'

config_file = YAML.load(File.read('config.yaml'))

STACK_NAME = config_file["stack_name"]

SAMPLE_SRC_PATH = config_file["sample_src_path"]
SAMPLE_DEST_PATH = config_file["sample_dest_path"]

FLARE_VM = {
  # "name" => "#{STACK_NAME}-FlareVM", 
  # "box" => "./Boxes/FlareVM-Win10Enterprise22H2vmware.box"
  # "version" => File.exist?("./Boxes/FlareVM-Win10Enterprise22H2vmware.box") ? nil : config_file["flare_vm_box_version"],
  "username" => "analyst", 
  "password" => "infected",
  "memory" => config_file["flare_vm_memory"],
  "cpus" => config_file["flare_vm_cpus"], 
  "vram" => 128
}


Vagrant.configure("2") do |config|
  # Flare VM
  config.vm.define FLARE_VM["name"] do |flare|
    flare.vm.box = FLARE_VM["box"]  
    if FLARE_VM["version"] != nil
      flare.vm.box_version = FLARE_VM["version"]
    end

    # flare.trigger.after [:up] do |trigger|
    #   trigger.info = "Disconnecting NAT network cable to #{FLARE_VM["name"]}..."
    #   trigger.run = {inline: "bash ./scripts/control-nat.sh \"#{FLARE_VM["name"]}\" \"off\""}
    # end

    # flare.vm.hostname = FLARE_VM["name"]
    # flare.vm.guest = :windows
    # flare.vm.synced_folder ".", "/vagrant", disabled: true  # disable synced folder
    # flare.vm.network :private_network, virtualbox__intnet: "flarenet", auto_config: false

    flare.vm.communicator = "winrm"
    flare.winrm.basic_auth_only = true    
    flare.winrm.transport = :plaintext
    flare.winrm.username = FLARE_VM["username"]
    flare.winrm.password = FLARE_VM["password"]
    flare.vm.boot_timeout = 1200
    flare.winrm.timeout = 1200
    flare.winrm.retry_limit = 20

    flare.vm.provision "shell", inline: "echo 'Provisioning Flare VM...'"
    flare.vm.provision "shell", path: "./res/flare/scripts/download-files.ps1", privileged: true, args: "-file_url #{SAMPLE_SRC_PATH} -file_path #{SAMPLE_DEST_PATH}"
    flare.vm.provision "shell", path: "./res/flare/scripts/install-chrome.ps1", privileged: false     
    flare.vm.provision "shell", path: "./res/flare/scripts/configure-windows-network.ps1", privileged: true, args: "-adapterName 'Ethernet 2' -ip '10.100.0.105' -gateway '10.100.0.1' -dns '10.100.0.1'"
    flare.vm.provision "file", source: "./res/flare/config/shutup10.cfg", destination: "C:\\Users\\analyst\\AppData\\Local\\Temp\\"
    flare.vm.provision "file", source: "./res/flare/config/MakeWindows10GreatAgain.reg", destination: "C:\\Users\\analyst\\AppData\\Local\\Temp\\"
    flare.vm.provision "shell", path: "./res/flare/scripts/MakeWindows10GreatAgain.ps1", privileged: false
    # flare.vm.provision "shell", path: "./scripts/install-sysinternals.ps1", privileged: false 
    flare.vm.provision "shell", inline: "echo 'Provisioning ended, rebooting...'"
    flare.vm.provision :reload

    flare.vm.provider "vmware_desktop" do |flare_vb, override|
      flare_vb.gui = true
      flare_vb.name = FLARE_VM["name"]

      flare_vb.customize ["modifyvm", :id, "--memory", FLARE_VM["memory"]]
      flare_vb.customize ["modifyvm", :id, "--cpus", FLARE_VM["cpus"]]
      flare_vb.customize ["modifyvm", :id, "--vram", FLARE_VM["vram"]]
      flare_vb.customize ["modifyvm", :id, "--clipboard", "bidirectional"]
      flare_vb.customize ["setextradata", "global", "GUI/SuppressMessages", "all" ]
    end
  end
end