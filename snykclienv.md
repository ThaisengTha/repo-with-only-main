# snykclienv.sh

This script will add the alias `snykclienv`  to your shell (zsh) which can start a temporary docker container from [snyk/snyk](https://hub.docker.com/r/snyk/snyk/tags) docker images. This is to help save time by providing a preconfigured environment for testing with CLI and to not have to change host environment/setup.

The command does the following:
- Passes the current directory the command is executed in as the working directory of the container so any files/subfolders present in the current directory will be accessible in the container.
- Copies the host's CLI API Key and sets it as SNYK_TOKEN env var in the container so reauthentication is not required.
- Starts `bash` shell session of active container so user can run CLI commands.

## Prerequisites
- Docker
- ZSH shell
- Snyk API Token
- Zscaler Certificates
- 1Password and its CLI (If you're going with the second option of storing env variables in 1Password)

## Certificate & Environemnt Setup
Run the zscaler certificate fix, if you haven’t already (see https://snyksec.atlassian.net/wiki/spaces/SE/pages/3630071840/Zscaler+First-Time+Login+Configuration+Guide)
This will add zscaler-root-ca.crt and zscaler-root-ca.pem to your ~/Documents folder.
Leave the certs where they are but copy the certs into a new folder called zscaler in your Documents folder.

```bash
mkdir -p ~/Documents/zscaler
cp ~/Documents/zscaler-root-ca.* ~/Documents/zscaler/

Create a file in the same folder, certs.env with the following content.

    AWS_CA_BUNDLE=/certs/zscaler-root-ca.crt
    REQUESTS_CA_BUNDLE=/certs/zscaler-root-ca.pem
    NODE_EXTRA_CA_CERTS=/certs/zscaler-root-ca.crt
    HOMEBREW_SSL_CERT_FILE=/certs/zscaler-root-ca.crt
    CURL_CA_BUNDLE=/certs/zscaler-root-ca.crt
    WGET_CA_BUNDLE=/certs/zscaler-root-ca.crt
 
## Installation
- Clone repo or download snykclienv.sh
- Make script executable: `chmod +x snykclienv.sh`
- Execute script and follow the prompt

## Option 1: Local Environment Token (Classic)
This option is simple, but requires local ENV of SNYK_TOKEN which also overrides the oauth auth for snyk. 
The env variable can be renamed to CLASSIC_SNYK_TOKEN in local environment  -e SNYK_TOKEN=${CLASSIC_SNYK_TOKEN} to avoid that, but still requires token in local env file. 

Create a local env var SNYK_TOKEN with your snyk api token.   
    export SNYK_TOKEN=<Your Snyk API Token>.

## Option 2: Env Variables Stored in 1Password
Ensure that you have the 1Password cli.  https://1password.com/downloads/command-line
Create an “API Credential” entry in your employee vault of 1password called ENV_GCP_SNYK_TOKEN and put your Snyk api token in that.  (you can create other env vars for the other environments too)
Alternatively, you can name your entry differently, but you need to adjust the name in the -e SNYK_TOKEN=op…. section below.

## Usage/Examples
Navigate into your local repo/test directory in terminal (Do not run from your home directory)
- To start a container:
    - `snykclienv <tag_name>`
    - `snykclienv python-3.8`
    - `snykclienv maven`
    - See https://hub.docker.com/r/snyk/snyk/tags for possible tags.

If you went with option 1, you’ll be asked to enter your password for sudo.
If you went with option 2, it will print signing in to onepassword for env vars and then you’ll be asked to authorize access (you may need to allow terminal to access other programs first), then you’ll be asked for your password for sudo.

- Once container has started, can use regular CLI commands as you normally would.
- To exit container, execute `exit` command, this will shutdown and kill the container.

## Notes
- This script is only designed for zsh shell (MacOS default). If using different shell, please add the following command as an alias:
```
f() { sudo docker run --rm -it --workdir $(pwd) -v $(pwd):$(pwd) -e SNYK_TOKEN=$(snyk config get api) --entrypoint bash snyk/snyk:$1 };f
```
