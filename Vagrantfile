# encoding: utf-8
# -*- mode: ruby -*-
# vi: set ft=ruby :

dir = File.dirname(File.expand_path(__FILE__))
file = "#{dir}/config.yaml"

require 'yaml'
configs = File.exist?(file) && YAML.load_file(file)

# environment overrides the config file
provider = ENV['VAGRANT_PROVIDER'] || (configs && configs['configs']['provider'])

if provider == ""
  provider = VAGRANT_DEFAULT_PROVIDER
end

if ARGV[0] == "up"
  puts "config file: #{file}"
  puts "provider: #{provider}"
end

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty64"

  # Workaround for annoying non-critical error during provisioning:
  # "==> default: stdin: is not a tty"
  config.vm.provision "fix-no-tty", type: "shell" do |s|
    s.privileged = false
    s.inline = "sudo sed -i '/tty/!s/mesg n/tty -s \\&\\& mesg n/' /root/.profile"
  end
  # end workaround

  # virtualbox only for now, other providers should be configured as needed
  if provider == "virtualbox"
    config.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--memory", 2048]
      v.name = "poco-build-trusty64"
    end
  else
    puts "Missing or not suported vagrant provider: #$ENV['VAGRANT_PROVIDER']"
    exit
  end #provider

  config.vm.provision :shell, inline: "apt-get update"
  config.vm.provision :docker
  # not using provisioned docker-compose, we install our own newer version able to create our own docker network
  config.vm.provision :shell, inline: "sudo curl -L https://github.com/docker/compose/releases/download/1.19.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose"
  config.vm.provision :shell, inline: "chmod +x /usr/local/bin/docker-compose"
  config.vm.provision :shell, inline: "docker pull phusion/baseimage:0.10.0"

  config.vm.provision :shell, inline: "cd /vagrant/docker/"
  # TODO: add capability to loop over various configs provided from outside
  config.vm.provision :shell, inline: "/vagrant/docker/configure"
  config.vm.provision :shell, inline: "COMPOSE_PROJECT_NAME=/vagrant/docker /vagrant/docker/run"

  #TODO: other boxen (windows, osx, android ...)
end # Vagrant.configure
