## Install NFS Server

- Install NFS Server on [Debian](https://wiki.debian.org/NFSServerSetup)/[Ubuntu](https://ubuntu.com/server/docs/service-nfs)
- mount /var/nfs/general
- add to /etc/exports: /var/nfs/general *(rw,sync,no_root_squash,no_subtree_check)
- Restart nfs-kernel-server

## Configure Security

- Deny access to NFS ports from all IP addresses
- Allow access to NFS from list of servers in the cluster.
- Lock the server from accidental deletion or restart using ISP Dashboard or API

## Usage

- Use it to mount `sites` directory from multiple benches
- Use it with Docker Swarm or Kubernetes
