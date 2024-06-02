# -*- mode: ruby -*-
# vi: set ft=ruby :

boxVersion = {
  "vagrant-box" => "12.20240503.1",
  "starship" => "v1.19.0",
  "chezmoi" => "v2.48.1"
}

boxConf = {
  "memory" => "8192",
  "cpus" => "4",
  "git-email" => "philipp@sanwald.com",
  "git-username" => "Philipp Sanwald",
  "github-username" => "backphlip",
  "ssh-keys" => "id_ed25519_github"
}

boxVar = {
  "hostmachine-homepath" => "C:/Users/Phi"
}

Vagrant.configure("2") do |config|
  # Official Debian boxes at https://app.vagrantup.com/debian
  config.vm.define "debianbox"
  config.vm.hostname = "debianbox"
  config.vm.box = "debian/bookworm64"
  config.vm.box_version = boxVersion["vagrant-box"]
  #config.vm.box_check_update = false

  #config.vm.network "forwarded_port", guest: 80, host: 8080

  config.vm.synced_folder boxVar["hostmachine-homepath"], "/mnt/host/home"
  config.vm.synced_folder ".", "/vagrant"

  # Provider-specific configuration
  config.vm.provider "virtualbox" do |vb|
    vb.memory = boxConf["memory"]
    vb.cpus = boxConf["cpus"]
  end

  config.vm.provision "shell", privileged: false,
    env: {
      "BOXVERSION_STARSHIP" => boxVersion["starship"],
      "BOXVERSION_CHEZMOI" => boxVersion["chezmoi"],
      "BOXCONF_GIT_EMAIL" => boxConf["git-email"],
      "BOXCONF_GIT_USERNAME" => boxConf["git-username"],
      "BOXCONF_GITHUB_USERNAME" => boxConf["github-username"],
      "BOXCONF_SSH_KEYS" => boxConf["ssh-keys"]
    },
    inline: <<-SHELL
    set -o errexit
    set -o nounset
    set -o pipefail
    #set -o xtrace

    cd $HOME

    # Software and tool installation
    # ------------------------------

    echo "INFO ===> Upgrading apt packages"
    sudo apt-get update
    sudo apt-get upgrade --yes

    echo "INFO ===> Installing additional apt packages"
    sudo apt-get install --yes \
      curl \
      wget \
      git \
      zsh \
      vim-nox
      # ...

    echo "INFO ===> Making zsh default shell for user $USER"
    sudo usermod --shell $(which zsh) "$USER"
    mkdir -v "${HOME}/.zsh_completions"

    echo "INFO ===> Installing oh-my-zsh"
    sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
    rm -vf \
      "${HOME}/.bash_history" \
      "${HOME}/.bash_logout" \
      "${HOME}/.bashrc"

    echo "INFO ===> Installing starship"
    sh -c "$(curl -fsSL https://starship.rs/install.sh)" -- \
      --verbose \
      --yes \
      --platform unknown-linux-gnu \
      --version "$BOXVERSION_STARSHIP"

    echo "INFO ===> Installing chezmoi"
    sudo sh -c "$(curl -fsSL get.chezmoi.io)" -- \
      -b /usr/local/bin \
      -t "$BOXVERSION_CHEZMOI"
    chezmoi completion zsh > "${HOME}/.zsh_completions/chezmoi.sh"

    # User configurations
    # -------------------

    echo "INFO ===> Configuring Git user email and name"
    git config --global user.email "$BOXCONF_GIT_EMAIL"
    git config --global user.name "$BOXCONF_GIT_USERNAME"

    echo "INFO ===> Setting up SSH keys"
    # Start ssh-agent
    eval $(ssh-agent -s)
    for key in $(echo $BOXCONF_SSH_KEYS | sed "s/,/ /g"); do
      # Copy ssh-key from host machine
      cp -v "/mnt/host/home/.ssh/${key}" "${HOME}/.ssh/${key}"
      chmod -v 600 "${HOME}/.ssh/${key}"
      cp -v "/mnt/host/home/.ssh/${key}.pub" "${HOME}/.ssh/${key}.pub"
      chmod -v 644 "${HOME}/.ssh/${key}.pub"
      # Add ssh-key to ssh-agent
      ssh-add -v "${HOME}/.ssh/${key}"
    done

    echo "INFO ===> Setting up dotfiles with chezmoi"
    # Avoid interactive host key verification prompt during git clone
    # More info: https://unix.stackexchange.com/questions/724693/how-to-disable-host-key-checking-check-on-git-over-ssh
    ssh-keyscan github.com >> "${HOME}/.ssh/known_hosts"
    chezmoi -v --force init "git@github.com:${BOXCONF_GITHUB_USERNAME}/dotfiles.git"
    chezmoi -v --force --no-pager apply
  SHELL
end
