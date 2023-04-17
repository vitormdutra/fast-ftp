# FastFTP -  VM Project

To run this project you need a new VM or a machine for your ftp-server and install mysql and proftpd

For this project i used a debianVM, but you can use ubutu-server, if you want to use a diferrent SO you need to adapt the instalation process.
```bash
# this is the template installation for debian/ubuntu

apt update
apt upgrade
apt install  wget
apt install  proftpd-basic  proftpd-mod-mysql
wget https://dev.mysql.com/get/mysql-apt-config_0.8.24-1_all.deb
dpkg -i  mysql-apt-config_0.8.24-1_all.deb
apt update
apt install  mysql-server
```

after finish the installation you need to create a new database and user in mysql for the ftp and grant the privileges.

i used the same name for database and user, you can change, but need to change in the configuration 

 ```bash
 create database  proftpd;
create user  proftpd@localhost  identified  by  'password';
grant all  privileges  on  proftpd.*  to  proftpd;
 ```

now need to create 2 tables in the database

```sql
CREATE TABLE  IF  NOT  EXISTS  `ftpgroup` (
`groupname` varchar(16) COLLATE utf8_general_ci  NOT  NULL,
`gid` smallint(6) NOT NULL DEFAULT '5500',
`members` varchar(16) COLLATE utf8_general_ci NOT NULL,
KEY `groupname` (`groupname`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8 COLLATE=utf8_general_ci COMMENT='proftpd group table';

CREATE TABLE  IF  NOT  EXISTS  `ftpuser` (
`id` int(10) unsigned NOT  NULL  AUTO_INCREMENT,
`userid` varchar(32) COLLATE utf8_general_ci NOT NULL DEFAULT '',
`passwd` varchar(32) COLLATE utf8_general_ci NOT NULL DEFAULT '',
`uid` smallint(6) NOT NULL DEFAULT '5500',
`gid` smallint(6) NOT NULL DEFAULT '5500',
`homedir` varchar(255) COLLATE utf8_general_ci NOT NULL DEFAULT '',
`shell` varchar(16) COLLATE utf8_general_ci NOT NULL DEFAULT '/sbin/nologin',
`count` int(11) NOT NULL DEFAULT '0',
`accessed` datetime NOT  NULL  DEFAULT  CURRENT_TIMESTAMP,
`modified` datetime NOT  NULL  DEFAULT  CURRENT_TIMESTAMP,
PRIMARY KEY (`id`),
UNIQUE KEY  `userid` (`userid`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8 COLLATE=utf8_general_ci COMMENT='proftpd user table';
```
And insert your first user, but you need to encrypt the password using this command

``/bin/echo "{md5}"`/bin/echo -n "passwd" | openssl dgst -binary -md5 | openssl enc -base64``

The result of this command you substitute in the "{md5}passwd" don't remove the {md5}

```sql
INSERT INTO  proftpd.ftpuser
(userid, passwd,  uid,  gid,  homedir,  shell,  count,  accessed,  modified)
VALUES('user', '{md5}passwd',  5500,  5500,  '/shared-dir',  '/sbin/nologin',  0,  CURRENT_TIMESTAMP,  CURRENT_TIMESTAMP);
```
Create the sql.conf 
```bash
nano /etc/proftpd/sql.conf

SQLBackend mysql
SQLAuthTypes OpenSSL  Crypt
SQLAuthenticate users  groups
SQLConnectInfo proftpd@localhost  proftpd  Trindade2010@
SQLUserInfo ftpuser  userid  passwd  uid  gid  homedir  shell
SQLGroupInfo ftpgroup  groupname  gid  members
SQLMinID 500
SQLLog PASS  updatecount
SQLNamedQuery updatecount  UPDATE  "count=count+1, accessed=now() WHERE userid='%u'"  ftpuser
```
change the main config file with this alterations (Comment the ipv6)

```bash
nano /etc/proftpd/proftpd.conf

# UseIPv6 on
Umask 0022  0022
PassivePorts 45000  48000
Include /etc/proftpd/sql.conf
RequireValidShell off
```

Now you can start and use your fpt-server 

```bash
/etc/init.d/proftpd stop
/etc/init.d/proftpd start
systemctl status  proftpd
```

to teste your ftp usa ftp-client like [filezilla](https://filezilla-project.org/)
