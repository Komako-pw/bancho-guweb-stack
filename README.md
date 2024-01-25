# guweb + bancho.py stack
Automatically set up guweb, bancho.py, mysql and caddy using Docker.

Works locally and in production.

## Setup

- Install docker and docker-compose
- Clone this repo **recursively** (to get the submodules)
> IMPORTANT! If you don't use --recursive or download a zip from the web, you will be missing the submodules and it won't work.
```
git clone --recursive https://github.com/Komako-pw/stack
```
- Copy `config.example` to `config`
- Configure
    - Place relevant certificates in `config/tls`
        - If you're using this on LAN, look at the [custom CA](#custom-ca) section
        - If you're using this on the open web, you may not need this at all
    - Adjust `config/bancho.py.env` and `config/guweb.py` configs to your needs
        - Make sure to set your domain
    - Select the correct tls mode in `config/proxy.Caddyfile`
    - Set your domain in `config/proxy.env`
    - Setting a custom mysql password in `config/mysql.env` is not necessary
- If on Linux (skip on Windows):
    ```
    mkdir -p data/bancho-guweb data/mysql data/redis data/proxy
    sudo chown -R 1000:1000 ./data
    ```
- Start the stack
```
docker-compose up -d
```

## Custom CA
You need a custom certificate authority (CA) to use HTTPS **locally**. Since Caddy is running in a container, it can't access your system's trusted CAs.

> IMPORATNT! If you're running this on the open web, you can use `tls_auto` and skip this section.

- Download [easyrsa](https://github.com/OpenVPN/easy-rsa/releases)
- Make a folder somewhere
- Generate a CA
> It will ask for a Common Name (CN). Choose anything you want.

```
easyrsa init-pki
easyrsa build-ca nopass
```

- Give it to caddy
    - `pki/ca.crt` to `config/tls/custom_ca/ca.crt`
    - `pki/private/ca.key` to `config/tls/custom_ca/ca.key`
- Make your system trust it
    > IMPORTANT! Make sure you put it into `Trusted Root Certification Authorities` and not use automatic placement.
- Use the `tls_custom_ca` mode in `proxy.Caddyfile`

## Alias
To avoid typing `docker-compose` every time, you can add an alias to your shell config.

`~/.bashrc` (bash) or `~/.zshrc` (zsh)
```sh
alias dc="docker-compose"
# restart your terminal
```

fish:
```sh
alias dc="docker-compose"
funcsave dc
```

powershell (not permanent because idk how to do that):
```ps1
Set-Alias -Name dc -Value docker-compose
```

## Hosts
You must add the following to your hosts file for local development:

- Linux `/etc/hosts`
- Windows `C:\Windows\System32\drivers\etc\hosts`

```
127.0.0.1 meow.nya a.meow.nya assets.meow.nya api.meow.nya osu.meow.nya b.meow.nya c.meow.nya c1.meow.nya c2.meow.nya c3.meow.nya c4.meow.nya c5.meow.nya c6.meow.nya ce.meow.nya
```

## Relevant commands

- Start/stop
```sh
dc up -d
dc down
```

- Show status
```sh
dc ps
```

- Restart
```sh
dc restart # all services

dc restart guweb
dc restart bancho
dc restart mysql
dc restart proxy
```

- Rebuild guweb or bancho (need to do this after updating the code)
```sh
dc build guweb
dc build bancho

dc up -d
```

- Update mysql, redis and caddy to the latest versions
```sh
dc pull
dc up -d
```

- See logs
```sh
dc logs -f # all services

dc logs -f guweb
dc logs -f bancho
dc logs -f mysql
dc logs -f proxy
```

- Open a shell (go inside the container)
```sh
dc exec mysql mysql -u root -p # use MYSQL_ROOT_PASSWORD password from ./config/mysql.env
dc exec bancho bash
dc exec guweb ash
```

- Clean old images to free up disk space
```sh
docker image prune -a
```

- Pull upstream (will cause merge conflicts)
```sh
cd guweb
git pull upstream main

cd bancho.py
git pull upstream master
```

- Reset data
```sh
dc down
rm -rf ./data
```

## License
[MIT](./LICENSE)
