---
layout: post
title: "Backing up PostgreSQL with backup and whatever gems on VPS"
date: 2012-10-10 07:12
comments: true
categories: [backup, postgresql, whatever, gem, vps]
---

## Mission

A few months ago I've created a [simple Rails app](https://github.com/xajler/mi-guest) (after 6 years!!!)
that essentially stores guests and keeping track of accommodations for my friend's villa.

After a month in production I've started thinking about backup of the database. Code is on
the Github so I have only need to worry about database.

My first initial thoughts were on Dropbox, I've installed the Dropbox client on my no GUI
Ubunte Server, but when I ran the Dropbox the memory usage got doubled and I remove it all together!

Then I started thinking about good old FTP backup, I was sure that I want backup somewhere else
than the VPS the database is running. I have one  account at [Arvixe](http://arvixe.com) Unlimited Linux hosting
and I've created the FTP user and even found [backup gem](https://github.com/meskyanichi/backup) back then
but I just drop all together until today.

I have VPS at [BuyVM (ex Frantech)](http://buyvm.net) and they were offering a free storage
VPS of 5Gb which is more than enough for my backup. I get one account and idea came again to use
it as backup...

...and the idea expanded with every day schedule of backup.

Incidentally, to day I finally get started my Octopress blog deployed via Github Pages, so I'm
writting all the steps as notes for futre usage, or if someone can be useful...

## backup Gem

The [backup gem](https://github.com/meskyanichi/backup) is one great gem with a lot of
backup options like databases, storages, compressions, encrypt-ors,...

### Install

Installation is a straight forward like any Ruby Gem:

{% codeblock lang:bash %}
$ gem install backup
{% endcodeblock %}

### Backup Model Generator

backup gem have its own backup "model" generator that creates backup config files, this is a comprehensive usage:

{% codeblock lang:bash %}
Usage:
  backup generate:model --trigger=TRIGGER

Options:
  --trigger=TRIGGER
  [--config-path=CONFIG_PATH]  # Path to your Backup configuration directory
  [--databases=DATABASES]      # (mongodb, mysql, postgresql, redis, riak)
  [--storages=STORAGES]        # (cloudfiles, dropbox, ftp, local, ninefold, rsync, s3, scp, sftp)
  [--syncers=SYNCERS]          # (cloud_files, rsync_local, rsync_pull, rsync_push, s3)
  [--encryptors=ENCRYPTORS]    # (gpg, openssl)
  [--compressors=COMPRESSORS]  # (bzip2, gzip, lzma, pbzip2)
  [--notifiers=NOTIFIERS]      # (campfire, hipchat, mail, presently, prowl, twitter)
  [--archives]
  [--splitter]                 # use `--no-splitter` to disable
                              # Default: true
{% endcodeblock %}

Generates a Backup model file

Let's create simple config file with backup generator:

{% codeblock lang:bash %}
$ backup generate:model --trigger miguest_backup --databases="postgresql" \
  --storages="ftp" --compressors="bzip2"
{% endcodeblock %}

This will create a `config.rb` and `models/miguest_backup.rb` files in `~/Backup` folder.

### Refine backup Configuration

Backup generator will create boilerplate model template needing more refinement. It looks something like this:

{% codeblock Boilerplate backup model template - miguest_backup.rb %}
    # encoding: utf-8

    ##
    # Backup Generated: miguest_backup
    # Once configured, you can run the backup with the following command:
    #
    # $ backup perform -t miguest_backup [-c <path_to_configuration_file>]
    #
    Backup::Model.new(:miguest_backup, 'Description for miguest_backup') do
      ##
      # Split [Splitter]
      #
      # Split the backup file in to chunks of 250 megabytes
      # if the backup file size exceeds 250 megabytes
      #
      split_into_chunks_of 250

      ##
      # PostgreSQL [Database]
      #
      database PostgreSQL do |db|
        db.name               = "my_database_name"
        db.username           = "my_username"
        db.password           = "my_password"
        db.host               = "localhost"
        db.port               = 5432
        # db.socket             = "/tmp/pg.sock"
        # db.additional_options = ["-xc", "-E=utf8"]
        # Optional: Use to set the location of this utility
        #   if it cannot be found by name in your $PATH
        # db.pg_dump_utility = "/opt/local/bin/pg_dump"
      end

      ##
      # FTP (File Transfer Protocol) [Storage]
      #
      store_with FTP do |server|
        server.username     = "my_username"
        server.password     = "my_password"
        server.ip           = "1.2.3.4"
        server.port         = 21
        server.path         = "~/backups/"
        server.keep         = 5
        server.passive_mode = false
      end

      ##
      # Bzip2 [Compressor]
      #
      compress_with Bzip2

    end
{% endcodeblock %}

Fairly straight forward refinement are needed, usernames, passwords,... After adding all
the secret stuff, the backup can be called as easy as:

{% codeblock lang:bash %}
$ backup perform --trigger miguest_backup
[2012/10/10 12:21:26][message] Performing Backup for 'Description for miguest_backup (miguest_backup)'!
[2012/10/10 12:21:26][message] [ backup 3.0.25 : ruby 1.9.3p194 (2012-04-20 revision 35410) [x86_64-linux] ]
[2012/10/10 12:21:26][message] Database::PostgreSQL started dumping and archiving 'miguest'.
[2012/10/10 12:21:26][message] Using Compressor::Bzip2 for compression.
[2012/10/10 12:21:26][message]   Command: '/bin/bzip2'
[2012/10/10 12:21:26][message]   Ext: '.bz2'
[2012/10/10 12:21:28][message] Database::PostgreSQL Complete!
[2012/10/10 12:21:28][message] Packaging the backup files...
[2012/10/10 12:21:28][message] Splitter configured with a chunk size of 250MB.
[2012/10/10 12:21:28][message] Packaging Complete!
[2012/10/10 12:21:28][message] Cleaning up the temporary files...
[2012/10/10 12:21:29][message] Storage::FTP started transferring '2012.10.10.12.21.26.miguest_backup.tar' to 'ftp.services.buyvm.net'.
[2012/10/10 12:21:29][message] Storage::FTP: Cycling Started...
[2012/10/10 12:21:29][message] Storage::FTP: Cycling Complete!
[2012/10/10 12:21:29][message] Cleaning up the package files...
[2012/10/10 12:21:29][warning] Backup for 'Description for miguest_backup (miguest_backup)' Completed Successfully (with Warnings) in 00:00:03
{% endcodeblock %}

### But there is more to backup gem

Really, what I'm using for backup is just minimal what backup gem can offer, here is detailed
impressive list of backup features:

#### Databases

MySQL, PostgreSQL, MongoDB, Redis, Riak

#### Compression

bzip2, gzip, lzma, pbzip2

#### Storages

cloudfiles, Dropbox, FTP, local, ninefold, rsync, S3, scp, SFTP

#### Notifiers

campfire, hipchat, mail, presently, prowl, twitter

#### Encryption

GPG, OpenSSL

## Scheduling the backups with whenever Gem

[Whenever](https://github.com/javan/whenever) is a Ruby gem that provides a clear syntax for
writing and deploying cron jobs. And this is what I need to schedule the backups to run everyday.

### Install

{% codeblock lang:bash %}
$ gem install whenever
{% endcodeblock %}

### Configure (Wheneverize)

Configure the whenever, but first make sure that we are in backup folder `~/Backup`

{% codeblock lang:bash %}
$ cd ~/Backup
{% endcodeblock %}

Then create config folder:

{% codeblock lang:bash %}
$ mkdir config
{% endcodeblock %}

And then run the command:

{% codeblock lang:bash %}
$ wheneverize .
~ [add] writing './config/schedule.rb'
~ [done] wheneverized!
{% endcodeblock %}

Open the `config/schedule.rb` file and add the following:

{% codeblock whenever schedule config - config/schedule.rb %}
every 1.day, :at => '4:30 am' do
  command "backup perform --trigger miguest_backup"
end
{% endcodeblock %}

### Schedule to crontab

Run `whenever` with no arguments see the `crontab` entry this will create:

{% codeblock lang:bash %}
$ whenever
30 4 * * * /bin/bash -l -c 'backup perform --trigger miguest_backup'

## [message] Above is your schedule file converted to cron syntax; your crontab file was not updated.
## [message] Run `whenever --help' for more options.
{% endcodeblock %}

To write (or update) this job in your `crontab`, use:

{% codeblock lang:bash %}
$ whenever --update-crontab
~ [write] crontab file written
{% endcodeblock %}

And that's it, so what is included:

* Backup of postgres database.
* Sending compressed backup via FTP.
* Scheduled backup every day at 4:30 am.
