# Vagrant debianbox

## Custom installed software and tools

- git
- zsh
- oh-my-zsh
- starship
- chezmoi
- ... (see TODOs)

## Troubleshooting

### `vagrant ssh` fails in PowerShell

```PowerShell
> vagrant ssh
vagrant@127.0.0.1: Permission denied (publickey).
```

This seems to be happening only with PowerShell.
To get around the issue, we have to configure vagrant to use the embedded ssh utility: `$Env:VAGRANT_PREFER_SYSTEM_BIN = 0` (PowerShell).
Then try `vagrant ssh` again.

More info:
- https://stackoverflow.com/questions/51437693/permission-denied-with-vagrant
- https://developer.hashicorp.com/vagrant/docs/other/environmental-variables#vagrant_prefer_system_bin

A better option is to use Git Bash on Windows.

## TODOs

- Split up into two Vagrantfiles
  - A first one with the for installing software and tools and doing general configurations
  - A second one for applying user specific configuration (e.g. ssh-keys, dotfiles, etc.)

  Parameterize the second Vagrantfile e.g. by [using Ruby getoptlong package](https://stackoverflow.com/questions/14124234/how-to-pass-parameter-on-vagrant-up-and-have-it-in-the-scope-of-vagrantfile).
  Candidates for options are, amongst others, the GitHub username and the names of the ssh-keys to copy over from the host machine's `.ssh` directory
- Install and, if necessary, configure following packages and tools:
  - kubectl, kubectx, kubens
  - docker or podman
  - go
  - java, maven, sdkman
- Consider using external shell scripts instead of inline scripts for provisioning inside the Vagrantfile(s)
  - Possibly a library of scripts as done by e.g. Bitnami [here](https://github.com/bitnami/containers/tree/main/bitnami/nginx/1.26/debian-12/prebuildfs/opt/bitnami/scripts) could be helpful
- Find out how to connect to vagrant box using VS code's remote development features (without running into issues) instead of using Vagrant's synced folders

  **EDIT:** this somehow worked when using Git Bash instead of PowerShell; follow [this blog](https://medium.com/@lopezgand/connect-visual-studio-code-with-vagrant-in-your-local-machine-24903fb4a9de).
  A possible reason could be that setting `VAGRANT_PREFER_SYSTEM_BIN=0`, as is necessary with PowerShell (see workaround in [Troubleshooting](#troubleshooting) section above), is not needed in Git Bash for running `vagrant ssh`, and having `VAGRANT_PREFER_SYSTEM_BIN=1` may be required for VS code's SSH functionality to work
- Set up neovim instead of vim-nox
