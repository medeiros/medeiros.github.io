---
layout: post
title: "Docker Introduction"
description: >
  Introduction of Docker: basic steps to understand, configure and run Docker 
categories: containerization
tags: [docker, containerization, cloud]
comments: true
image: /assets/img/blog/docker/docker.jpg
---
> This page explains the basic of Docker architecture and configuration. It 
aims to reinforce knowledge of this technology.
{:.lead}

- Table of Contents
{:toc}

## Docker Configuration

### Docker Login  

#### Creating an account

The first step is go to [Docker Hub](https://hub.docker.com) and create an account.

#### Connecting to Docker Hub 

Then, in your local terminal, execute the following command:

```
[user@host]$ docker login
```

You'll be prompted to inform the Docker Hub username and password.

After that, the following warning will appear:

```
WARNING! Your password will be stored unencrypted in /home/daniel/.docker
/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-s
tore
```

So lets improve credencial's security by adding `pass` along with `gpg` 
(which will be required by pass).

#### Installing pass and gpg

The first step is to install `pass` and `gpg` locally (to save 
docker password). So, execute the following command:

```
sudo pacman -S pass gnupg
```

#### Configuring gpg

Execute the following commands to configure gpg:

```
gpg --expert --full-gen-key
```

- Then select the option `9` (to create an ECC - Elliptic Curve Criptography).
- Next, choose option `1` (to create ed25519 keys).
- Next, choose `2y` (so the key will be valid for two years) and confirm.
- Now, provide identification for the key: your full name and email address. 
This information is important because it will be included in the key. The 
email address is considered an unique identifier.
- Select `O` option ("OK") to confirm
- After that, a passphrase will be asked to protect your private key. Inform 
it - and save this passphrase (that will be required if you want to recover
your private key).

After that, GPG will generate your keys in the `~/.gnupg` directory, with 
the following three files, at least:

- `./openpgp-revocs.d/<your key>.rev`: this is the public key
- `./private-keys-v1.d/<first file>.rev`: this is one of the files that 
composes your private key
- `./private-keys-v1.d/<second file>.rev`: this is the second file that 
composes your private key

Along with another files.

> Note: in case you want to export the public key (not required), it can be 
done by executing  the following command:

`[~/.gnupg/openpgp-revocs.d]$ gpg --armor --export <your email address> > pubkey.asc`

#### Configuring pass with gpg

Now that we have the public GPG key, lets get back and configure pass.

```
$ ~> pass init <your email address, which is the unique 
  identifier of the gpg key>

Password store initialized for <your email address>
```

The directory `~/.password-store` was created, containing the 
`.gpg_id` file. This file has only one line - your email address, as a 
unique identifier for gpg.

#### Configuring Docker Credential Store for pass

Since `pass` is properly configured, it is now necessary to configure 
Docker Credential store for `pass` [^1], [^2].

The steps are the following:

- Download the [Docker credentials pass binary release for Linux](https://github.com/docker/docker-credential-helpers/releases)
- Rename the downloaded file to `docker-credential-pass`

```
mv docker-credential-pass-v0.8.2.linux-amd64 docker-credential-pass

chmod +x docker-credential-pass
```

- Add the binary file path to the `PATH` env var, so Docker can 
find it:

```
nvim ~/.config/fish/config.fish

set -x PATH $PATH ~/data/app/docker-credential-helpers/
```

#### Going back to Docker Login

Now that we have `pass`, `gpg` and `docker-credential-helper for pass` 
properly configured, we need to configure Docker to use `pass`.

Edit the Docker config file, remove the previous content and make sure that 
is set as below:

```
nvim ~/.docker/config.json

{
  "credsStore": "pass"
}
```
After that, you can try to login again:

```
docker login
```

Enter your Docker user name and password.
After that, the message `Login Succeeded` is displayed.

You can check pass and see the Docker entry that was just added. 
Just type:

```
pass
```

And the following structure will be displayed:

```
Password Store
└── docker-credential-helpers
    └── <some hash>
        └── <your docker user login>
```

You can also check the docker config file:

```
bat ~/.docker/config.json

───────┬──────────────────────────────────────────────
       │ File: /home/daniel/.docker/config.json
───────┼──────────────────────────────────────────────
   1   │ {
   2   │     "auths": {
   3   │         "https://index.docker.io/v1/": {}
   4   │     },
   5   │     "credsStore": "pass"
   6   │ }
───────┴──────────────────────────────────────────────

```

And you can see that the "auths" entry was added.

Now your docker login credentials security were improved.


## Docker Basic Concepts

The following is a very simple Docker command:

```bash
docker run  debian      echo "hello world"
----------  ----------  ------------------
docker cmd  image name  container command
```

Explanation:

- `docker run`: create a new docker container and run it.
- `debian`: this is the image name of the container
  - if not exists, it will be downloaded;
  - one can verify the existing images by typing `docker image ls`
- `echo "hello world"`: this is the command to be executed 
  - a docker container run the command and then finalizes it

When executing this command, the `debian` image will be downloaded (if 
not yet in the environment), the container will be created and then 
the command `echo "hello world"` will be executed (the output will be 
the 'hello world' message in the terminal).

After the execution, the container still exists - it was created and is 
not running (status=Exited): 

```bash
daniel@ataraxia ~> docker ps -a

CONTAINER ID   IMAGE     COMMAND                CREATED              STATUS                          PORTS     NAMES
90d910e2614a   debian    "echo 'hello world'"   About a minute ago   Exited (0) About a minute ago             friendly_moser
```
If we try to run this container again, the output would be the name of the 
container:

```bash
daniel@ataraxia ~ [1]> docker start friendly_moser
friendly_moser
daniel@ataraxia ~>
```
This is because there is no STDIN usage defined for this container.
So we add the `-i` param, and now we can see the expected result:

```bash
daniel@ataraxia ~> docker start friendly_moser -i
hello world
```

It is not possible to login into this container because it was created 
without interactive tty mode, and because the main command is echo, instead 
of a bash shell. So, let's create a new container with this is mind:

```bash
daniel@ataraxia ~> docker run -it --name test debian /bin/bash
root@a13f7ed087d5:/# 
```

The `-it` param was added to create this as a STDIN container and attach to 
the current terminal. Also, the main process is a bash shell instead of a 
echo command. So now we gained access to the container via bash shell.
In addition to that, the container is now named `test`.

If a new terminal is opened and `docker ps` is run, we can see that this 
new container is running:

```bash
daniel@ataraxia ~> docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS         PORTS     NAMES
a13f7ed087d5   debian    "/bin/bash"   7 minutes ago   Up 7 minutes             test
```

And if we exit the container we can see now that it is no longer running:

```bash
root@a13f7ed087d5:/# exit
exit
daniel@ataraxia ~ [127]> docker ps -a
CONTAINER ID   IMAGE     COMMAND                CREATED          STATUS                       PORTS     NAMES
a13f7ed087d5   debian    "/bin/bash"            9 minutes ago    Exited (127) 8 seconds ago             test
90d910e2614a   debian    "echo 'hello world'"   27 minutes ago   Exited (0) 19 minutes ago              friendly_moser
d
```

So, the container runs while its main process is executing (in this case, `bash`).

todo: add log, inspect, add fortune, cowsay, commit e remove

## Conclusions

This article aimed to reinforce knowledge of Docker fundamentals. 

## References

- [A Practical Guide to GPG Part 1: Generate Your Public/Private Key Pair](https://www.linuxbabe.com/security/a-practical-guide-to-gpg-part-1-generate-your-keypair)

[^1]: [Docker login: credentials store](https://docs.docker.com/reference/cli/docker/login/#credentials-store)
[^2]: [Docker Credential Store](https://github.com/docker/docker-credential-helpers/releases)
