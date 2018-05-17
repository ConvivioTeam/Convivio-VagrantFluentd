# -*- mode: ruby -*-
# vi: set ft=ruby :

require_relative 'lib/fluentdvm/vagrant'

# Absolute paths on the host machine.
host_project_dir = File.dirname(File.expand_path(__FILE__))
host_config_dir = ENV['FLUENTDVM_CONFIG_DIR'] ? "#{host_project_dir}/#{ENV['FLUENTDVM_CONFIG_DIR']}" : host_project_dir

# Absolute paths on the guest machine.
guest_project_dir = '/vagrant'
guest_config_dir = ENV['FLUENTDVM_CONFIG_DIR'] ? "/vagrant/#{ENV['FLUENTDVM_CONFIG_DIR']}" : guest_project_dir

fluentdvm_env = ENV['FLUENTDVM_ENV'] || 'vagrant'

default_config_file = "#{host_config_dir}/default.config.yml"
unless File.exist?(default_config_file)
  raise_message "Configuration file not found! Expected in #{default_config_file}"
end

vconfig = load_config([
  default_config_file,
  "#{host_config_dir}/config.yml",
  "#{host_config_dir}/#{fluentdvm_env}.config.yml",
  "#{host_config_dir}/local.config.yml"
])

provisioner = vconfig['force_ansible_local'] ? :ansible_local : vagrant_provisioner
if provisioner == :ansible
  playbook = "#{host_config_dir}/provisioning/playbook.yml"
  config_dir = host_config_dir

  # Verify Ansible version requirement.
  require_ansible_version ">= #{vconfig['fluentdvm_ansible_version_min']}"
else
  playbook = "#{guest_config_dir}/provisioning/playbook.yml"
  config_dir = guest_config_dir
end

# Verify Vagrant version requirement.
Vagrant.require_version ">= #{vconfig['fluentdvm_vagrant_version_min']}"

ensure_plugins(vconfig['vagrant_plugins'])

Vagrant.configure('2') do |config|
  # Set the name of the VM. See: http://stackoverflow.com/a/17864388/100134
  config.vm.define vconfig['vagrant_machine_name']

  # Networking configuration.
  config.vm.hostname = vconfig['vagrant_hostname']
  config.vm.network :private_network,
    ip: vconfig['vagrant_ip'],
    auto_network: vconfig['vagrant_ip'] == '0.0.0.0' && Vagrant.has_plugin?('vagrant-auto_network')

  unless vconfig['vagrant_public_ip'].empty?
    config.vm.network :public_network,
      ip: vconfig['vagrant_public_ip'] != '0.0.0.0' ? vconfig['vagrant_public_ip'] : nil
  end

  # SSH options.
  config.ssh.insert_key = false
  config.ssh.forward_agent = true

  # Vagrant box.
  config.vm.box = vconfig['vagrant_box']

  # Display an introduction message after `vagrant up` and `vagrant provision`.
  config.vm.post_up_message = vconfig.fetch('vagrant_post_up_message', get_default_post_up_message(vconfig))

  # Sync the project root directory to /vagrant
  unless vconfig['vagrant_synced_folders'].any? { |synced_folder| synced_folder['destination'] == '/vagrant' }
    vconfig['vagrant_synced_folders'].push(
      'local_path' => host_project_dir,
      'destination' => '/vagrant'
    )
  end

  # Synced folders.
  vconfig['vagrant_synced_folders'].each do |synced_folder|
    options = {
      type: synced_folder.fetch('type', vconfig['vagrant_synced_folder_default_type']),
      rsync__exclude: synced_folder['excluded_paths'],
      rsync__args: ['--verbose', '--archive', '--delete', '-z', '--copy-links', '--chmod=ugo=rwX'],
      id: synced_folder['id'],
      create: synced_folder.fetch('create', false),
      mount_options: synced_folder.fetch('mount_options', []),
      nfs_udp: synced_folder.fetch('nfs_udp', false)
    }
    synced_folder.fetch('options_override', {}).each do |key, value|
      options[key.to_sym] = value
    end
    config.vm.synced_folder synced_folder.fetch('local_path'), synced_folder.fetch('destination'), options
  end

  config.vm.provision provisioner do |ansible|
    ansible.playbook = playbook
    ansible.extra_vars = {
      config_dir: config_dir,
      fluentdvm_env: fluentdvm_env
    }
    ansible.raw_arguments = Shellwords.shellsplit(ENV['FLUENTDVM_ANSIBLE_ARGS']) if ENV['FLUENTDVM_ANSIBLE_ARGS']
    ansible.tags = ENV['FLUENTDVM_ANSIBLE_TAGS']
    # Use pip to get the latest Ansible version when using ansible_local.
    provisioner == :ansible_local && ansible.install_mode = 'pip'
  end

  # VirtualBox.
  config.vm.provider :virtualbox do |v|
    v.linked_clone = true
    v.name = vconfig['vagrant_hostname']
    v.memory = vconfig['vagrant_memory']
    v.cpus = vconfig['vagrant_cpus']
    v.customize ['modifyvm', :id, '--natdnshostresolver1', 'on']
    v.customize ['modifyvm', :id, '--ioapic', 'on']
    v.gui = vconfig['vagrant_gui']
  end


  # Cache packages and dependencies if vagrant-cachier plugin is present.
  if Vagrant.has_plugin?('vagrant-cachier')
    config.cache.scope = :box
    config.cache.auto_detect = false
    config.cache.enable :apt
    # Cache the composer directory.
    config.cache.enable :generic, cache_dir: '/home/vagrant/.composer/cache'
    config.cache.synced_folder_opts = {
      type: vconfig['vagrant_synced_folder_default_type'],
      nfs_udp: false
    }
  end

  # Allow an untracked Vagrantfile to modify the configurations
  [host_config_dir, host_project_dir].uniq.each do |dir|
    eval File.read "#{dir}/Vagrantfile.local" if File.exist?("#{dir}/Vagrantfile.local")
  end

  end