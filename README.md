# Dockerfile Snippets

These snippets are tested on `debian:bookworm` based images.
They should help save time when creating Dockerfiles, especially for [VS Code Dev Containers](https://code.visualstudio.com/docs/devcontainers/containers).

### Contributing

If you have any suggestions or improvements, please feel free to open an issue or pull request.
Useful snippets are always welcome.

## Table of Contents

- [Dockerfile Snippets](#dockerfile-snippets)
  - [Table of Contents](#table-of-contents)
  - [Update and Upgrade Packages](#update-and-upgrade-packages)
  - [Install Common Packages](#install-common-packages)
  - [Install Zsh/Oh My Zsh](#install-zshoh-my-zsh)
  - [Install Node.js and NPM](#install-nodejs-and-npm)
  - [Install Docker with Host Socket](#install-docker-with-host-socket)

## Update and Upgrade Packages

```Dockerfile
##################################
### START # Update and Upgrade ###
##################################

RUN apt update \
    && apt upgrade -y

##################################
### END # Update and Upgrade #####
##################################
```

## Install Common Packages

```Dockerfile
#######################################
### START # Install Common Packages ###
#######################################

RUN apt install -y build-essential curl wget git vim nano unzip zip gnupg2 apt-transport-https ca-certificates lsb-release software-properties-common

#######################################
### END # Install Common Packages #####
#######################################
```

## Install Zsh/Oh My Zsh

```Dockerfile
#####################################
### START # Install Zsh/Oh My Zsh ###
#####################################

# Base install of Zsh with Oh My Zsh
RUN apt install -y zsh \
    && sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" --unattended

# Install Zsh plugins
RUN git clone https://github.com/zsh-users/zsh-autosuggestions $HOME/.oh-my-zsh/custom/plugins/zsh-autosuggestions \
    && git clone https://github.com/zsh-users/zsh-syntax-highlighting $HOME/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting \
    && git clone https://github.com/zsh-users/zsh-history-substring-search $HOME/.oh-my-zsh/custom/plugins/zsh-history-substring-search

# Configure Zsh to use plugins
RUN sed -i 's/plugins=(git)/plugins=(git zsh-autosuggestions zsh-syntax-highlighting zsh-history-substring-search history aliases sudo themes docker nmap kubectl)/' $HOME/.zshrc \
    && sed -i 's/ZSH_THEME="robbyrussell"/ZSH_THEME="af-magic"/' $HOME/.zshrc

# Set Zsh as default shell
RUN chsh -s /usr/bin/zsh

#####################################
### END # Install Zsh/Oh My Zsh #####
#####################################
```

## Install Node.js and NPM

You can change the `NODE_MAJOR` argument to the version you want to install.

```Dockerfile
#######################################
### START # Install Node.js and NPM ###
#######################################

ARG NODE_MAJOR=20

RUN set -uex; \
    mkdir -p /etc/apt/keyrings; \
    curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key \
    | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg; \
    echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_${NODE_MAJOR}.x nodistro main" \
    > /etc/apt/sources.list.d/nodesource.list; \
    apt -y update; \
    apt -y install nodejs;

#######################################
### END # Install Node.js and NPM #####
#######################################
```

## Install Docker with Host Socket

```Dockerfile
###############################################
### START # Install Docker with Host Socket ###
###############################################

# Set docker host socket
ENV DOCKER_HOST=unix:///var/run/docker-host.sock

# Install Docker
RUN apt install -y lsb-release \
    && curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add - \
    && echo "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null \
    && apt update \
    && apt install -y docker-ce docker-ce-cli containerd.io

# Add user to docker group if it doesn't exist and add user to it
RUN if getent group docker > /dev/null 2>&1; then \
    echo "Docker group exists"; \
    else \
    groupadd docker; \
    fi \
    && usermod -aG docker root

###############################################
### END # Install Docker with Host Socket #####
###############################################
```

If you are using this for a VS Code devcontainer, you should add the following to your `.devcontainer/devcontainer.json` file to allow the container to access the host socket:

```json
"mounts": [
    "source=/var/run/docker.sock,target=/var/run/docker-host.sock,type=bind"
]
```

If you are using the `docker-compose.yml` file to allow the container to access the host socket, you can instead add the following to your `docker-compose.yml` file under the `volumes` section if the container:

```yaml
volumes:
  - /var/run/docker.sock:/var/run/docker-host.sock
```