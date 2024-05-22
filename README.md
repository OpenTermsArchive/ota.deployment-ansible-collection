# Open Terms Archive Ansible Collection

This repository contains the `opentermsarchive.deployment` Ansible collection. This Ansible collection provides playbooks to set up the infrastructure of and deploy Open Terms Archive applications.

To prevent confusion between the notion of Ansible [Collection](https://docs.ansible.com/ansible/latest/collections_guide/index.html) and an Open Terms Archive [Collection](https://docs.opentermsarchive.org/#collection), we will refer to Ansible Collection only as “Playbook”, as this is the main entry point to interact with it.

## Installation

[Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) is required to use this playbook.

This playbook can be installed from Ansible Galaxy manually with the `ansible-galaxy` command-line tool:

```sh
ansible-galaxy collection install opentermsarchive.deployment
```

It can also be included in a `requirements.yml` file using the format:

```yaml
---
collections:
- name: opentermsarchive.deployment
```

And installed with the command:

```sh
ansible-galaxy collection install -r requirements.yml
```

## Usage

Once installed, the playbook `deploy` allows to set up the two main Open Terms Archive applications: [Engine](https://github.com/OpenTermsArchive/engine) and [Federated API](https://github.com/OpenTermsArchive/federated-api).

The playbook can be executed using the `ansible-playbook` command-line tool:

```sh
ansible-playbook opentermsarchive.deployment.deploy
```

_It is possible to check a playbook execution without actually applying changes with `check` and `diff` options:_

```sh
ansible-playbook opentermsarchive.deployment.deploy --check --diff
```

> See “[Using collections](https://docs.ansible.com/ansible/latest/user_guide/collections_using.html)” in Ansible’s user guide for more information about Ansible collections.

- - -

### Configuration

Available [variables](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html) are listed below, along with their default values:

| Variable                       | Description                                                                                                                | Default Value          | Required |
|--------------------------------|----------------------------------------------------------------------------------------------------------------------------|------------------------|----------|
| `ota_source_repository`        | URL of the source repository                                                                                               | No default value       | ✔︎        |
| `ota_source_repository_branch` | [Git branch or tag](https://git-scm.com/docs/gitglossary#Documentation/gitglossary.txt-aiddeftree-ishatree-ishalsotreeish) of the source repository | `main`                 |          |
| `ota_directory`                | Directory path where the code will be deployed on the server                                                               | Name of the repository |          |

These variables can be defined in the inventory file, for example:

```yml
all:
  hosts:
    127.0.0.1:
      ansible_user: debian
      ota_source_repository: https://github.com/OpenTermsArchive/demo-declarations.git
      ota_directory: demo
```

#### Additional files

The `deploy` playbook requires additional files to be placed alongside the `inventory.yml` file. These files are necessary for deploying Open Terms Archive applications properly. Below are the required and optional files and their purposes:

| File                     | Description                                                                                             | Required | Encryption Required |
|--------------------------|---------------------------------------------------------------------------------------------------------|----------|---------------------|
| `pm2.config.cjs`         | Configuration file describing the processes to be started and managed by PM2                            | ✔︎        |                     |
| `github-bot-private-key` | Private SSH key for accessing SSH Git URLs                                                              | Required if `ota_source_repository` is an SSH Git URL or if the URLs for versions and/or snapshots repositories in the `config/production.json` file of the source repository are SSH Git URLs | ✔︎                 |
| `.env`                   | File defining environment variables required by the deployed application                                |          | ✔︎                   |

Here is an example of the directory structure:

```plaintext
ops/
  ├── inventory.yml
  ├── pm2.config.cjs
  ├── github-bot-private-key
  └── .env
```

#### Encrypting sensitive configuration files

Sensitive configuration files should be encrypted using [Ansible Vault](https://docs.ansible.com/ansible/latest/vault_guide/index.html).

Examples:

- Encrypt the `github-bot-private-key` file: `ansible-vault encrypt github-bot-private-key`
- Decrypt the `github-bot-private-key` file: `ansible-vault decrypt github-bot-private-key`
- Encrypting with a password stored in a file:
  - `echo 'your_password' > vault.key`
  - `ansible-vault encrypt --vault-password-file vault.key github-bot-private-key`

To run the playbook with encrypted files:

```sh
ansible-playbook playbook.yml --ask-vault-pass
```

Or with a password file:

```sh
ansible-playbook playbook.yml --vault-password-file vault.key
```

Please note that the data will be stored unencrypted on the deployment server.

### Refining playbook execution

Use [tags](https://docs.ansible.com/ansible/latest/user_guide/playbooks_tags.html) to refine playbook execution. Example commands:

| Tag             | Description                             | Command Example                                                                  |
|-----------------|-----------------------------------------|----------------------------------------------------------------------------------|
| `start`         | Start Open Terms Archive applications   | `ansible-playbook opentermsarchive.deployment.deploy --tags start`               |
| `stop`          | Stop Open Terms Archive applications    | `ansible-playbook opentermsarchive.deployment.deploy --tags stop`                |
| `restart`       | Restart Open Terms Archive applications | `ansible-playbook opentermsarchive.deployment.deploy --tags restart`             |
| `infrastructure`| Set up the infrastructure only          | `ansible-playbook opentermsarchive.deployment.deploy --tags infrastructure`      |
| `infrastructure`| Skip the infrastructure                 | `ansible-playbook opentermsarchive.deployment.deploy --skip-tags infrastructure` |

- - -

## Development

### Requirements

1. Install [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).
2. Install [Vagrant](https://www.vagrantup.com/downloads).
3. Install [VirtualBox](https://www.virtualbox.org/wiki/Downloads) to manage virtual machines. If Docker is prefered, or on an Apple Silicon machine, install [Docker](https://docs.docker.com/get-docker/) instead.
4. Create a dedicated SSH key with no password: `ssh-keygen -f ~/.ssh/ota-vagrant -q -N ""`. This key will be automatically used by Vagrant.

> VirtualBox is not compatible with Apple Silicon (M1…) processors. To use vagrant on this kind of machine, specifying the Docker provider will be required. Since MongoDB cannot be installed on ARM, it is skipped in the infrastructure installation process. This means the MongoDB storage repository cannot be tested with Vagrant with an Apple Silicon processor.

## Usage

For testing this collection, a virtual machine description file is provided, inside the `tests` folder, to be used with [Vagrant](https://www.vagrantup.com).

All following commands must be executed from the `tests` folder:

    cd tests

### Launch VM

```sh
vagrant up
```

> With an Apple Silicon processor or to use Docker instead of VirtualBox, use `vagrant up --provider=docker`.

Then the code can be deployed to the running machine with all the options described before.

### Test collection

Testing the Ansible collection locally is crucial to ensure that changes function properly before submitting them as a pull request.

The testing environment is preconfigured for Open Terms Archive maintainers. For other contributors, the inventory file `tests/inventory.yml` needs to be updated to specify repositories where they have authorizations. Additionally, the `github-bot-private-key` file should be updated.

Follow these instructions to test the collection in a local environment:

- Ensure you have a clean testing environment to prevent interference from previous configurations:
```sh
vagrant destroy
vagrant up
```

- Apply the changes to the virtual machine:
```sh
ansible-playbook ../playbooks/deploy.yml
```

- Connect to the virtual machine to verify that changes were applied successfully:
```sh
vagrant ssh # use "vagrant" as password
```

- Check that everything works as intended within the virtual machine. Depending on the nature of changes made, you can monitor logs or execute commands to validate functionality:
```sh
pm2 logs
```

#### Troubleshooting

If you encounter an error while running the playbook, such as:

```sh
PLAY [Deploy Open Terms Archive applications] ************************************************

TASK [Gathering Facts] ***************************************************************************
fatal: [127.0.0.1]: UNREACHABLE! => changed=false
  msg: 'Failed to connect to the host via ssh: vagrant@127.0.0.1: Permission denied (publickey,password).'
  unreachable: true
```

Do the following:

1. Run `vagrant ssh-config` and note the `IdentityFile` output:

```sh
vagrant ssh-config
```

Example output:

```plaintext
Host opentermsarchive_deployment
  HostName 127.0.0.1
  User vagrant
  Port 2222
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /path/to/your/vagrant/private_key
  IdentitiesOnly yes
  LogLevel FATAL
  PubkeyAcceptedKeyTypes +ssh-rsa
  HostKeyAlgorithms +ssh-rsa
```

2. Add the private key to your SSH agent:

```sh
ssh-add /path/to/your/vagrant/private_key
```

---

## License

The code for this software is distributed under the European Union Public Licence (EUPL) v1.2.
Contact the author if you have any specific need or question regarding licensing.
