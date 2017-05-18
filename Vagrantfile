# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.box = "bento/ubuntu-16.04"
  config.vm.box_check_update = false

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  #config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #

  # Set number of CPUs assigned to the VM
  num_cpus = (ENV["VAGRANT_NUM_CPUS"] || "2").to_i
  ram_mb = ENV["VAGRANT_RAM"] || "4096"

  config.vm.provider "virtualbox" do |vb|
    # cableconnected on is needed for nested virtualization build environment.
    vb.customize ["modifyvm", :id, "--cableconnected1", "on"]

    # If you want to run vagrant up on e.g. a fast build server over ssh, comment
    # out the line below to run headless.
    # vb.gui = true

    vb.customize ["modifyvm", :id, "--memory", ram_mb]
    vb.cpus = num_cpus
  end

  # These are applied to all the software compiled from the Vagrantfile
  makeflags = "-j" + String(num_cpus + 1)

  #
  # Start setting up the machine as an SDE
  #

  # Install a desktop environment
  config.vm.provision "shell" do |s|
    s.inline = "apt-get update -y"
  end

  config.vm.provision "shell" do |s|
    s.inline = "apt-get install lubuntu-desktop -y"
  end

  config.vm.provision "shell" do |s|
    s.inline = "apt-get install cmake -y"
  end

  # Enable deb-src repos and install qt5 build dependencies
  config.vm.provision "shell", path: "cookbook/system-config/enable-all-deb-src.sh"
  config.vm.provision "shell", path: "cookbook/deps/qt5-dependencies.sh"

  # Define where to install Qt
  qtinstallprefix = "/opt/qt-5.8-git"
  qmakepath = qtinstallprefix + "/bin/qmake"
  qtcmakepath = qtinstallprefix + "/lib/cmake/"

  # Build Qt5 from git
  config.vm.provision "shell", privileged: false,
    path: "cookbook/build/qt5-git-builder.sh",
    env: {
      "MAKEFLAGS" => makeflags,
      "SRC_DIR" => "qt5",
      "GIT_REPO" => "https://github.com/qt/qt5.git",
      "GIT_REVISION" => "5.8",
      "CONFIGURE_ARGS" => "--prefix=" + qtinstallprefix + " -opensource -confirm-license",
    }

  # Build QtIVI
  config.vm.provision "shell", privileged: false,
    args: ["qtivi", "http://code.qt.io/qt/qtivi.git", qmakepath, "dev", "KEEP_OLD_BUILD"],
    path: "cookbook/build/qmake-git-builder.sh",
    env: {
      "MAKEFLAGS" => makeflags,
      "SRC_DIR" => "qtivi",
      "GIT_REPO" => "http://code.qt.io/qt/qtivi.git",
      "QMAKE_PATH" => qmakepath,
      "QMAKE_ARGS" => "",
      "GIT_REVISION" => "465f64cdd9a6f97893e77c9b46bfd1a2f830084d"
    }

  # Build QtApplicationManager
  config.vm.provision "shell", privileged: false,
    path: "cookbook/build/qmake-git-builder.sh",
    env: {
      "MAKEFLAGS" => makeflags,
      "QMAKE_PATH" => qmakepath,
      "SRC_DIR" => "qtapplicationmanager",
      "GIT_REPO" => "http://code.qt.io/qt/qtapplicationmanager.git",
      "QMAKE_PATH" => qmakepath,
      "GIT_REVISION" => "5.8"
      }

  # Install template configuration for QtApplicationManager
  config.vm.provision "shell" do |s|
    s.inline = "cp -r qtapplicationmanager/template-opt/am /opt/"
  end
  config.vm.provision "shell" do |s|
    s.inline = "mkdir -p /opt/am/docs"
  end

  config.vm.provision "shell" do |s|
    s.inline = "chown -R vagrant:vagrant /opt/am"
  end

  # Build neptune-ui
  config.vm.provision "shell", privileged: false,
    path: "cookbook/build/qmake-git-builder.sh",
    env: {
      "MAKEFLAGS" => makeflags,
      "SRC_DIR" => "neptune-ui",
      "GIT_REPO" => "http://code.qt.io/qt-apps/neptune-ui.git",
      "QMAKE_PATH" => qmakepath,
      "GIT_REVISION" => "5.8"
    }

  config.vm.provision "shell", privileged: false,
    args: [qtinstallprefix],
    path: "sde-cookbook/misc/perform-extra-neptune-setup.sh"

  # Build dlt-viewer
  config.vm.provision "dlt-viewer", type: "shell", privileged: false,
    path: "cookbook/build/qmake-git-builder.sh",
    env: {
      "MAKEFLAGS" => makeflags,
      "SRC_DIR" => "dlt-viewer",
      "GIT_REPO" => "http://github.com/GENIVI/dlt-viewer.git",
      "QMAKE_PATH" => qmakepath,
      "PRO_FILE" => "../BuildDltViewer.pro"
    }

  # Build qmllive
  config.vm.provision "qmllive", type: "shell", privileged: false,
    path: "cookbook/build/qmake-git-builder.sh",
    env: {
      "MAKEFLAGS" => makeflags,
      "SRC_DIR" => "qmllive",
      "GIT_REPO" => "http://code.qt.io/qt-apps/qmllive.git",
      "QMAKE_PATH" => qmakepath,
      "GIT_REVISION" => "5.8"
    }

  # Configure qtchooser
  config.vm.provision "shell", args: [qtinstallprefix], :inline => <<-SHELL
    QTINSTALLPREFIX=$1
    mkdir -p /etc/xdg/qtchooser
    echo "$QTINSTALLPREFIX/bin" > /etc/xdg/qtchooser/qt5.8-git.conf
    echo "$QTINSTALLPREFIX/lib" >> /etc/xdg/qtchooser/qt5.8-git.conf
  SHELL

  # Build gammaray
  config.vm.provision "shell", privileged: false,
    path: "cookbook/build/cmake-git-builder.sh",
    env: {
      "SRC_DIR" => "gammaray",
      "MAKEFLAGS" => makeflags,
      "QT_SELECT" => "qt5.8-git",
      "GIT_REPO" => "https://github.com/KDAB/GammaRay.git",
      "CMAKE_ARGS" => "-DGAMMARAY_BUILD_DOCS=off -DCMAKE_PREFIX_PATH=" + qtcmakepath,
      "GIT_REVISION" => "2.7"
      }

  # Download and install target SDK
  config.vm.provision "sdk", type: "shell", privileged: false,
    args: ["minnowboard"],
  path: "sde-cookbook/sdk/download-sdk.sh"

  # Install and configure qtcreator for use with target SDK
  config.vm.provision "shell" do |s|
    s.inline = "apt-get install qtcreator -y"
  end

  config.vm.provision "shell", privileged: false,
    args: ["Minnowboard"],
    path: "sde-cookbook/sdk/configure-qtcreator.sh"

  # Copy over some skeleton files into users home dir
  config.vm.provision "file", source: "files/vagrant", destination: "/home/"

  # Install some example media for music player app
  config.vm.provision "shell", privileged: false,
  path: "sde-cookbook/misc/install-example-media.sh"
end
