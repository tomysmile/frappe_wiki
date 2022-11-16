This setup will create a separate hosted MariaDB server to be used by frappe benches.

### Prerequisites

- VM or Server with Ubuntu LTS
- Block volume mounted at `/var/lib/mysql` (Optional)

### Install MariaDB

Mount volumes (Optional)

In the following example `/dev/sda` is attached as the block volume.

```shell
sudo mkfs.ext4 /dev/sda
sudo mkdir -p /var/lib/mysql
echo '/dev/sda /var/lib/mysql ext4 defaults 0 0' >> /etc/fstab
sudo mount -o defaults /dev/sda /var/lib/mysql
sudo rm -fr /var/lib/mysql/lost+found
```

Update Ubuntu

```shell
sudo apt update && sudo apt dist-upgrade -y
```

Set [AutomaticSecurityUpdates](https://help.ubuntu.com/community/AutomaticSecurityUpdates)

```shell
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

Install MariaDB Server

```shell
sudo apt install mariadb-server
sudo mysql_secure_installation
sudo systemctl restart mariadb
```

Configure MariaDB

- Open the `/etc/mysql/mariadb.conf.d/50-server.cnf` file and set `bind-address = 0.0.0.0`. This step is necessary for server to allow external access. We must protect the server using firewall rules instead.
- Add file named `/etc/mysql/mariadb.conf.d/50-server.cnf` with [additional configuration](https://github.com/frappe/bench/wiki/MariaDB-conf-for-Frappe) required create frappe sites.

### Allow external access to MariaDB

Enter mysql shell

```shell
sudo mysql -uroot
```

Replace the `PASSWORD` with a secure password and enter the following statements:

```SQL
GRANT ALL PRIVILEGES ON *.* TO 'admin'@'%' IDENTIFIED BY 'PASSWORD';
CREATE DATABASE admin;
FLUSH PRIVILEGES;
```

Make sure `Grant_priv` property of `mysql.user` `admin@%` is set to `Y`.

```shell
SELECT `User`, `Grant_priv` FROM `mysql`.`user` WHERE `User` = 'admin';
UPDATE `mysql`.`user` SET `Grant_priv` = 'Y' WHERE `User` = 'admin';
SELECT `User`, `Grant_priv` FROM `mysql`.`user`;
FLUSH PRIVILEGES;
``` 

### Setup Firewall Rules

- Setup rule to allow only access from VPC to port 3306 of the server
- Optionally use `ufw` if cloud or router level firewall is not used.
