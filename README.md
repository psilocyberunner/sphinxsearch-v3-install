# sphinxsearch-v3-install
 
 *Contains several helper-config files to make it easier to install and run SphinxSearch v3+.*

 *Repository doesn't contain Docker files or other ways to run search engine inside container.*

 ***I strongly recommend to use https://manticoresearch.com/ engine as an alternative to SphinxSearch due to 
 available source code, APT/YUM repos and Windows installer, more features, more stable.***

---

When your /var/run folder is located on tmpfs file system (ex: ubuntu/debian with systemd), it is often a surprise 
when your created folders in /var/run disappear after the system reboot. 

So below is an example to install SphinxSearch v3.5+ on such OS.

 **1. Get desired version from http://sphinxsearch.com**

```shell
wget http://sphinxsearch.com/files/sphinx-3.5.1-(XXX-some-hash-XXX)-linux-amd64.tar.gz
```

You version hash will be different from my one.

 **2. Create user**

```shell
useradd -r -U -c 'Sphinxsearch system user' sphinx
```

 **3. Unarchive repo contents**
 
 You'll get something like this as directory structure:
 
 ```text
sphinx-3.5.1
├── api
│   └── ...
├── bin
│   ├── indexer
│   ├── indextool
│   ├── searchd
│   └── wordbreaker
├── doc
│   └── ...
├── etc
│   ├── example.sql
│   ├── sphinx-min.conf.dist
│   └── sphinx.conf.dist
├── misc
│   └── ...
└── src
     └── ...
```
 We are interested in ./bin folder contents only. Just copy ./bin files into your /usr/bin folder.
 
 ```shell
 cp sphinx-3.5.1/bin/* /usr/bin
 ```
 
 Test that searchd - SphinxSearch daemon binary now exists in your system
 
 ```shell
 whereis searchd
 ```
 
 You'll get 
 
> searchd: /usr/bin/searchd

**4. Configure our installation**

I have my own example config, you may take yours.

First - create paths we need to store indexes, config files, logs and etc.

```shell
mkdir -p /etc/sphinx /var/run/sphinx /var/log/sphinx /var/lib/sphinx/data
```

Let our created sphinx user to deal with new paths.

```shell
chown -R sphinx:sphinx /etc/sphinx /var/run/sphinx /var/log/sphinx /var/lib/sphinx
```

Move config files from this repository /etc folder to their places.

General config:
> /etc/sphinx/sphinx.conf

Systemd service file:
> /etc/systemd/system/sphinx.service

File, indicating our OS to restore /var/run/sphinx folder with write permissions for sphinx user:
> /usr/lib/tmpfiles.d/sphinx.conf
 
**5. Enable systemd service**
 Enter this to enable system service:
 
```shell
systemctl enable sphinx
```
 
 You'll get something like:
```text
Created symlink /etc/systemd/system/sphinx.service → /lib/systemd/system/sphinx.service.
```
 
**5. Start the service**

Now you can start SphinxSearch daemon as

```shell
systemctl start sphinx
```

**6.Check installation**

To check the service you can use:

```shell
ps ax | grep searchd
```

results in similar output:

```text
  3744 ?        S      0:00 /usr/bin/searchd --config /etc/sphinx/sphinx.conf
  3746 ?        Sl     0:00 /usr/bin/searchd --config /etc/sphinx/sphinx.conf
```

SphinxSearch supports connections via mysql-like interface, so to perform additional checks we need:

**Note: for this check you'll have to install mysql-client library (ex: mysql-client-core-8.0)**

```shell
mysql -uroot -h 127.0.0.1 -P 9306
```

Port 9306 is the port you selected in _searchd_ section of your sphinx.conf file under parameter _listen_


In opened console you can see that SphinxSearch created our test news index

```test
mysql> show tables;
+-------+------+
| Index | Type |
+-------+------+
| news  | rt   |
+-------+------+
1 row in set (0.00 sec)
```

Reboot your system/virtual machine and see the service running.

To stop running search engine use

```shell
systemctl stop sphinx
```

