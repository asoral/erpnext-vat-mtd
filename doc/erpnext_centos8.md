# Install ERPNext on CentOS 8

Some quick notes on installing ERPNext on CentOS 8.

Just so you know:

 * This is a minimal working installation, not a production site.

 * We don't cover firewalls and making the box secure - that's up to you!

## Install CentOS8.

1) Install CentOS 8. We chose the "minimal" install for this guide. If you've
   picked another option you'll probably need to open 
   you're new or rusty with this stuff we recommend you pick one of the 
   "Server" installs because they include more helpful tools.

2) After install, login and ensure your installation is up to date
   by running :

```sh
  sudo yum update -y
```

3) Install the extra packages repository:

```sh
  sudo yum install -y epel-release
```

## Prepare OS for ERPNext

1) Install required packages:

```sh
  sudo yum install -y gcc make git mariadb mariadb-server nginx supervisor python3 python3-devel python2 python2-devel redis nodejs
  sudo npm install -g yarn
```

2) Create a user for ERPNext to run as, allowing it sudo access too :

```sh
  sudo useradd -m erp -G wheel
```

3) Configure sudo so it doesn't need a password:

You might want to cut'n'paste this one!

```sh
  sudo sed -i 's/^#\s*\(%wheel\s\+ALL=(ALL)\s\+NOPASSWD:\s\+ALL\)/\1/' /etc/sudoers
```

## Prepare MariaDB (mysql) for ERPNext

1) Edit the mariadb configuration to set the correct character set:

```sh
  cat <<EOF >/etc/my.cnf.d/erpnext.cnf
[mysqld]
innodb-file-format=barracuda
innodb-file-per-table=1
innodb-large-prefix=1
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

[mysql]
default-character-set = utf8mb4
EOF
```

2) Enable and start the mariadb service

```sh
  systemctl enable mariadb
  systemctl start mariadb
```

3) Secure the service

Start the secure script:

```sh
  mysql_secure_installation
```

This is an interactive script that will ask you questions.

Options are:
  * Current root password is none - just press enter.
  * Set the root password - remember it!
  * Remove anonymous users - Y
  * Disallow remote root - Y
  * Remove test database - Y
  * Reload priv tables now - Y

Done!

## Install ERPNext

1) Switch to the ERP user (or login as it) and change to home directory:

```sh
  su erp
  cd
```

2) Install frappe-bench with pip and initialise:

This step takes a while so get yourself a beer. It reaches out to the Internet
and downloads a bunch of stuff and then builds it.

```sh
  pip3 install --user frappe-bench
  bench init frappe-bench
```

For the second command, a red error message appears early on about an "editable
requirement." Ignore it.

When it's done you should get the message in green text:

```
  SUCCESS: Bench frappe-bench initialized
```

3) Create a new frappe site:

Prerequisites:
  * You need a name for your site. We called ours erpdev.softwaretohardware.com
  * You'll need your MariaDB root password from earlier.

First we temporarily start the frappe development server:

```sh
  cd frappe-bench
  sed -i '/web:/ s/$/ --noreload/' Procfile
  bench start >/tmp/bench_log &
```

Then we create a new site. Substitute your own name.

```sh
  bench new-site erpdev.softwaretohardware.com
```

You will be prompted for the mysql password and a bit later, for the
adminstrator password for your new site.

NOTE: Don't visit your new site with a browser just yet!

4) Install the ERPNext application

```sh
  bench get-app erpnext
  bench install-app erpnext
```

At the end of this step, the temporary server will stop and the exception
message looks bad. You can ignore it.

5) Bring back your temporary server

```sh
  bench start
```

You now have an ERPNext instance listening on port 8000.

Visit it with a browser to set it up.

When you're done, press "Ctrl+C" to stop the site. You can start it again at
any time.

