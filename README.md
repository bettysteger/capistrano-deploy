# Capistrano actions
Github deploy action for Capistrano. Use this action to automate your capistrano deployment process.

## Dependencies
This action expects Ruby to be installed along with Capistrano, see below for a [basic workflow example](#workflow-example) that uses [ruby/setup-ruby](https://github.com/ruby/setup-ruby).

## Inputs
### `target`
Environment where deploy is to be performed to. E.g. "production", "staging". Default value is empty.

### `deploy_key`
**Required** Symmetric key to decrypt private 
key. Must be a string.

### `enc_rsa_key_pth`
Path to the encrypted key. Default `"config/deploy_id_rsa_enc"`. You have to use either `enc_rsa_key_pth` or `enc_rsa_key_val`.

### `enc_rsa_key_val`
Contents of the encrypted key. Best to use as repository secret. You have to use either `enc_rsa_key_pth` or `enc_rsa_key_val`.

### `working-directory`
The directory from which to run the deploy commands, including `bundle install`.

## Outputs
No outputs

## Setting up CD using this action
1. Generate SSH keys on the target machine
```bash
$ ssh-keygen -t ed25519
```
2. Export public key to the `authorized_keys` to allow the usage of this keypair to login
```bash
$ cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
```
3. Add public key from `~/.ssh/id_ed25519.pub` to your repository's deployment keys via *Settings / Deploy keys / Add*
4. Encrypt your private key with a strong password. **Please use these options**, otherwise this action may not be able to decrypt your key.
```bash
$ openssl enc -aes-256-cbc -md sha512 -salt -in ~/.ssh/id_ed25519 -out deploy_id_rsa_enc -k "PASSWORD" -a -pbkdf2
```
5. Add `deploy_id_ed25519_enc` file to your repository. Suggested path is `config/deploy_id_rsa_enc`
6. Save the password used in step 4 as a secret in repository settings via *Settings / Secrets / Add*
7. Create YAML configuration for your workflow (example below)

## Workflow example
```yaml
# This is a basic workflow to help you get started with Actions
name: Deploy with Capistrano

# Controls when the action will run. 
on:
  # Triggers the workflow on push or pull request events but only for the main branch
  push:
    branches: [ main ]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  deploy:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
    # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
    - uses: actions/checkout@v2
    - uses: ruby/setup-ruby@v1
      with:
        # ruby-version: 3.0.1 # Not needed with a .ruby-version file
        bundler-cache: true # runs 'bundle install' and caches installed gems automatically
    - uses: miloserdow/capistrano-deploy@master
      with:
        target: development # Defines the environment that will be used for the deployment
        deploy_key: ${{ secrets.DEPLOY_ENC_KEY }} # Name of the variable configured in Settings/Secrets of your github project
```
