# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

$script = <<SCRIPT
GOVERSION="1.7.3"
SRCROOT="/opt/go"
SRCPATH="/opt/gopath"

# Get the ARCH
ARCH="$(uname -m | sed 's|i686|386|' | sed 's|x86_64|amd64|')"

# Install Prereq Packages
export DEBIAN_PRIORITY=critical
export DEBIAN_FRONTEND=noninteractive
export DEBCONF_NONINTERACTIVE_SEEN=true
APT_OPTS="--yes --force-yes --no-install-suggests --no-install-recommends"
echo "Upgrading packages ..."
apt-get update ${APT_OPTS} >/dev/null
apt-get upgrade ${APT_OPTS} >/dev/null
echo "Installing prerequisites ..."
apt-get install ${APT_OPTS} build-essential curl git-core libpcre3-dev mercurial pkg-config zip >/dev/null

# Install Go
echo "Downloading go (${GOVERSION}) ..."
wget -P /tmp --quiet "https://storage.googleapis.com/golang/go${GOVERSION}.linux-${ARCH}.tar.gz"
echo "Setting up go (${GOVERSION}) ..."
tar -C /opt -xf "/tmp/go${GOVERSION}.linux-${ARCH}.tar.gz"
chmod 775 "$SRCROOT"
chown vagrant:vagrant "$SRCROOT"

# Setup the GOPATH; even though the shared folder spec gives the working
# directory the right user/group, we need to set it properly on the
# parent path to allow subsequent "go get" commands to work.
mkdir -p "$SRCPATH"
chown -R vagrant:vagrant "$SRCPATH" 2>/dev/null || true
# ^^ silencing errors here because we expect this to fail for the shared folder

install -m0755 /dev/stdin /etc/profile.d/gopath.sh <<EOF
export GOPATH="$SRCPATH"
export GOROOT="$SRCROOT"
export PATH="$SRCROOT/bin:$SRCPATH/bin:\$PATH"
EOF

cat >>/home/vagrant/.bashrc <<EOF

## After login, change to terraform directory
cd /opt/gopath/src/github.com/hashicorp/terraform
EOF

SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "bento/ubuntu-14.04"
  config.vm.hostname = "terraform"

  config.vm.provision "prepare-shell", type: "shell", inline: "sudo sed -i '/tty/!s/mesg n/tty -s \\&\\& mesg n/' /root/.profile", privileged: false
  config.vm.provision "initial-setup", type: "shell", inline: $script
  config.vm.synced_folder '.', '/opt/gopath/src/github.com/hashicorp/terraform'

  config.vm.provider "docker" do |v, override|
    override.vm.box = "tknerr/baseimage-ubuntu-14.04"
  end

  ["vmware_fusion", "vmware_workstation"].each do |p|
    config.vm.provider p do |v|
      v.vmx["memsize"] = "4096"
      v.vmx["numvcpus"] = "2"
    end
  end

  config.vm.provider "virtualbox" do |v|
    v.memory = 4096
    v.cpus = 2
  end

  config.vm.provider "parallels" do |prl|
    prl.memory = 4096
    prl.cpus = 2
  end
end
