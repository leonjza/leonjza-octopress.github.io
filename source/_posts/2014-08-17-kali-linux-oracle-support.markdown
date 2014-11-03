---
layout: post
title: "Kali Linux Oracle Support"
date: 2014-08-17 15:46:49 +0200
comments: true
categories: 
- how to
- kali linux
- oracle
- metasploit
- OCI
- library
---

Recently I have had to get Oracle support sorted in my Kali Linux install. I will try not to rant about the reasons why it doesn't just work out of the box and just get the steps written down quickly. Typically, when you try to use a module such as `oracle_login`, metasploit may error out with:

```bash Metasploit Oracle Error
msf auxiliary(oracle_login) > run

[-] Failed to load the OCI library: cannot load such file -- oci8
[-] See http://www.metasploit.com/redmine/projects/framework/wiki/OracleUsage for installation instructions
[*] Auxiliary module execution completed
msf auxiliary(oracle_login) > run

```

The link provided seems a little out of date, so here is an updated guide.

<!--more-->

##getting started
First, if you dont have an account at oracle.com, you will _unfortunately_ need one here. If you don't want to create one, don’t worry, a temp email works as well as you luckily don’t have to confirm ownership of it before you can download.

##sorting oracle first
Anyways. Create yourself a directory `/opt/oracle` and `cd` there. In a browser, navigate to the [instant client downloads](http://www.oracle.com/technetwork/database/features/instant-client/index-097480.html) url and choose your architecture. If you hate 32bit Kali, choose [Linux x86](http://www.oracle.com/technetwork/database/features/instant-client/index-097480.html), and 64bit Kali choose [Linux x86-64](http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html).

Next, download the following packages and save them to `/opt/oracle`:

- instantclient-basic-linux-12.1.0.1.0.zip
- instantclient-sqlplus-linux-12.1.0.1.0.zip
- instantclient-sdk-linux-12.1.0.1.0.zip

With those downloaded, extract the archives with the `unzip` command. This should leave you with a directory `/opt/oracle/instantclient_12_1/`. This is all the files needed to at least get the `sqlplus` client going. However, when we build the ruby stuff needed for Metasploit, we will need to hax a .so. Do this with:

```bash so symlink
root@kali:/opt/oracle/instantclient_12_1# ln libclntsh.so.12.1 libclntsh.so

root@kali:/opt/oracle/instantclient_12_1# ls -lh libclntsh.so
lrwxrwxrwx 1 root root 17 Aug 17 15:37 libclntsh.so -> libclntsh.so.12.1
```

The last step is to configure the environment so that your shell is... _oracle aware_. We do this by setting a few environment variables. I have chosen to just append the lines to the end of my `.bashrc` file


```bash .bashrc
root@kali:/opt/oracle/instantclient_12_1# tail ~/.bashrc 

# ORACLE
export PATH=$PATH:/opt/oracle/instantclient_12_1
export SQLPATH=/opt/oracle/instantclient_12_1
export TNS_ADMIN=/opt/oracle/instantclient_12_1
export LD_LIBRARY_PATH=/opt/oracle/instantclient_12_1
export ORACLE_HOME=/opt/oracle/instantclient_12_1
```

Done! Log out and back and check that these are present in the output of `env`.

##sorting out metasploit
The last thing we need to do is setup the metasploit part. For this we need to download [ruby-oci8](https://github.com/kubo/ruby-oci8/archive/ruby-oci8-2.1.7.zip). In `/opt/oracle`, download it:

```bash download ruby-oci8
root@kali:/opt/oracle# wget https://github.com/kubo/ruby-oci8/archive/ruby-oci8-2.1.7.zip
[...] 
2014-08-17 16:11:31 (42.8 KB/s) - `ruby-oci8-2.1.7.zip' saved [278270]

root@kali:/opt/oracle# unzip ruby-oci8-2.1.7.zip 
Archive:  ruby-oci8-2.1.7.zip
fb913e32d8a09bd46e5bf549bd8e554f0870d384
   creating: ruby-oci8-ruby-oci8-2.1.7/
  inflating: ruby-oci8-ruby-oci8-2.1.7/.gitignore  
  inflating: ruby-oci8-ruby-oci8-2.1.7/.yardopts  
[...]
  inflating: ruby-oci8-ruby-oci8-2.1.7/test/test_rowid.rb  
root@kali:/opt/oracle#
```
Next, move to the extracted directory and install the `ruby-dev` package if you have not already done so:

```bash ruby-dev
root@kali:/opt/oracle# cd ruby-oci8-ruby-oci8-2.1.7/

root@kali:/opt/oracle/ruby-oci8-ruby-oci8-2.1.7# apt-get install ruby-dev
```

Lastly, we will `make` && `make install`. Logout and back in and test that the oracle tools in metasploit are functional :)