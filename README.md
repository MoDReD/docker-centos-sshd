# CentOS SSHD

A simple, extensible, [dockerized SSH service][centos_sshd_docker] built on top of the [official CentOS 7][centos_docker] Docker image.

The root password is "root".

SSH2 host keys (RSA, ECDSA, and ED25519) are generated when the container is started, unless already present.

## Basic usage

```bash
$ docker run -dP --name=sshd sickp/centos-sshd
7b7579858c25bf07b48d6442bf4d346a140b86d0405809af9efde14ba4377d2a

$ docker logs sshd
Generated SSH2 RSA host key.
Generated SSH2 ECDSA host key.
Generated SSH2 ED25519 host key.
Server listening on 0.0.0.0 port 22.
Server listening on :: port 22.

$ docker port sshd
22/tcp -> 0.0.0.0:32768

$ ssh root@localhost -p 32768 # or $(docker-machine ip default) on Mac OS X / Windows
# The root password is "root".
```

You can also append additional arguments to SSHD if required. For example, to enable debug output:

```
$ docker run -dP --name=sshd sickp/centos-sshd -d
```

## Customization through extension

This image doesn't attempt to be "the one" solution that suits everyone's needs. It's actually pretty useless in the real world. But it is easy to extend via your own `Dockerfile`. See the [examples][examples_github] directory.

### Change root password

Change the root password to something more fun, like "password" or "sunshine":

```dockerfile
FROM sickp/centos-sshd:latest
RUN echo "root:sunshine" | chpasswd
```

### Use authorized keys

Disable the root password completely, and use your SSH key instead:

```dockerfile
FROM sickp/centos-sshd:latest
RUN usermod -p "!" root
COPY identity.pub /root/.ssh/authorized_keys
```

### Create multiple users

Disable root and create individual user accounts:

```dockerfile
FROM sickp/centos-sshd:latest
RUN \
  usermod -p "!" root && \
  useradd sickp && \
  mkdir ~sickp/.ssh && \
  curl -o ~sickp/.ssh/authorized_keys https://github.com/sickp.keys && \
  useradd afrojas && \
  mkdir ~afrojas/.ssh && \
  curl -o ~afrojas/.ssh/authorized_keys https://github.com/afrojas.keys
```

### Embed SSH host keys

Embed SSH host keys directly in your private image, so you can treat your containers like cattle.

```dockerfile
FROM sickp/centos-sshd:latest
RUN \
  usermod -p "!" root && \
  ssh-keygen -q -t RSA -N "" -f /etc/ssh/ssh_host_rsa_key && \
  ssh-keygen -q -t ECDSA -N "" -f /etc/ssh/ssh_host_ecdsa_key && \
  ssh-keygen -q -t ED25519 -N "" -f /etc/ssh/ssh_host_ed25519_key && \
  useradd sickp && \
  mkdir ~sickp/.ssh && \
  curl -o ~sickp/.ssh/authorized_keys https://github.com/sickp.keys
```

## History

- 2015-11-11 Allow passing of arguments to SSHD.
- 2015-11-03 Collapse layers, setuid ping, .dockerignore, more examples.
- 2015-11-02 Initial version.

[centos_docker]:      https://hub.docker.com/_/centos/
[centos_sshd_docker]: https://hub.docker.com/r/sickp/centos-sshd/
[examples_github]:    https://github.com/sickp/docker-centos-sshd/tree/master/examples
