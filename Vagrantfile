# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANT_VERSION = "2"
Vagrant.require_plugin "vagrant-aws"

module LocalCommand
	class Config < Vagrant.plugin(VAGRANT_VERSION, :config)
		attr_accessor :labels
		attr_accessor :proxies
		attr_accessor :engine_opts
		attr_accessor :master
		attr_accessor :swarm
		attr_accessor :swarm_token
		attr_accessor :insecure_registries

		def initialize
			@options = []
		end
	end

	class Provisioner < Vagrant.plugin(VAGRANT_VERSION, :provisioner)
		def provision!
			ssh     = env[:vm].config.ssh
			env     = env[:vm].env
			machine = env[:machine]
			name    = env[:machine].box.name

			options = "--driver general"
			options << " --generic-ssh-user #{ssh.username}"
			options << " --generic-ssh-key #{env.default_private_key_path}"
			options << " --generic-ssh-port #{ssh.port}"
			options << " --generic-ip-address #{ssh.address}" 
			options << " --engine-opt log-driver=syslog"
			if config.engine_opts
				config.engine_opts.each do |opt|
					options << " --engine-opt #{opt}"
				end
			end
			if config.labels
				config.labels.each do |label|
					options << " --engine-label #{label}"
				end
			end
			options << " --engine-install-url #{config.install_url}" if config.install_url
			options << " --engine-storage-driver overlay"
			options << " --swarm" if config.swarm
			options << " --swarm-master" if config.master
			options << " --swarm-discovery token://#{config.swarm_token}" if config.swarm_token
			if config.insecure_registries
				config.insecure_registries.each do |url|
					options << " --engine-insecure-registry #{url}"
				end
			end
			if config.proxies
				config.proxies.each do |proxy|
					options << " --engine-env #{proxy.type}=#{proxy.url}"
				end
			end
			options = options + config.options unless config.options.empty?
			options << " #{name}"

			cmd    = (%w( docker-machine create --driver general ) << options).flatten
			safe_exec *cmd
		end
	end

	class Plugin < Vagrant.plugin(VAGRANT_VERSION)
		name "docker_machine"
		description 'Provision a machine with docker-machine'

		config(:docker_machine, :provisioner) do
			Config
		end

		provisioner(:docker_machine) do
			Provisioner
		end
	end

end

Vagrant.configure("2") do |config|

  config.vm.network "forwarded_port", guest: 80, host: 80, protocol: "tcp", auto_correct: true
  config.vm.network "forwarded_port", guest: 443, host: 443, protocol: "tcp", auto_correct: true
  config.vm.network "forwarded_port", guest: 8000, host: 8000, protocol: "tcp", auto_correct: true
  config.vm.network "forwarded_port", guest: 8080, host: 8080, protocol: "tcp", auto_correct: true
  config.vm.network "forwarded_port", guest: 2376, host: 2376, protocol: "tcp", auto_correct: true
  config.vm.network "forwarded_port", guest: 2377, host: 2377, protocol: "tcp", auto_correct: true
  config.vm.network "forwarded_port", guest: 3376, host: 3376, protocol: "tcp", auto_correct: true
  config.vm.network "forwarded_port", guest: 7946, host: 7946, protocol: "tcp", auto_correct: true
  config.vm.network "forwarded_port", guest: 164, host: 164, protocol: "udp", auto_correct: true
  config.vm.network "forwarded_port", guest: 4789, host: 4789, protocol: "udp", auto_correct: true
  config.vm.network "forwarded_port", guest: 7946, host: 7946, protocol: "udp", auto_correct: true

  config.vm.provider :virtualbox do |provider, override|
	  config.vm.box = "xenji/ubuntu-17.04-server"
  end

  config.vm.provider :aws do |aws, override|

	aws.access_key_id = ENV['AWS_ACCESS_KEY_ID']
	aws.secret_access_key = ENV["AWS_SECRET_ACCESS_KEY"]
	aws.keypair_name = ENV["AWS_KEYPAIR_NAME"]
	aws.ami = "ami-fb310580"
	aws.tags = {
		'Name' => env[:machine].config.box.name,
		'net.matrix.orgunit' => "Matrix NOC",
		'net.matrix.organization' => "Private Ops",
		'net.matrix.commonname' => "cloud",
		'net.matrix.locality' => "Dallas",
		'net.matrix.state' => "Texas",
		'net.matrix.country' => "USA",
		'net.matrix.environment' => "<nonprod|production|staging>",
		'net.matrix.application' => "infrastructure",
		'net.matrix.role' => "application services",
		'net.matrix.owner' => "FC13F74B@matrix.net",
		'net.matrix.customer' => "PVT-01",
		'net.matrix.costcenter' => "INT-01"
	}
	aws.instance_type = "t2.micro"
	aws.region = ENV["AWS_DEFAULT_REGION"]
	aws.subnet_id = ENV["AWS_SUBNET"]
	aws.security_groups = ENV["AWS_SECURITY_GROUPS"]
	override.nfs.functional = false

  end

  config.vm.provision :docker_machine do |dm|
	dm.swarm = true
	dm.labels = ["net.dapla.role='devlocal'"]
  end

end
