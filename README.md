PyClean.py - A Usenet spamfilter intended for use with the INN2 server.

PyClean performs a similar role to the original Perl-based Cleanfeed.  Its core
functionality can be adjusted through settings in the pyclean.cfg file, while
other config files control specific filter and logging behaviour.  Examples of
these files can be found in the pyclean/samples directory.

By default the configuration takes a cautious approach, assuming that
under-filtering is better than accidental removal of valid posts.  The
pyclean.log file provides a real-time overview of filter actions so rules can
be tweaked where required.  In addition, entire articles can be saved to files
via rules defined in the pyclean/etc/log_rules file.

Please remember to configure the local_hosts file to contain a list of
injecting-hosts that are under the operator's control.  This file determines
whether articles are matched against bad_* or local_bad_* rule files.


# INSTALL

For now the Debian INN2 package does not provide Python support.

To add Python support to the Debian INN2 compile it with the missing **--with-python** option. 

This is easier than it appears and package recompilation is well docummented [here](https://wiki.debian.org/BuildingAPackage)

Once Python support added to INN:

1. Clone the repository to your server
```shell
git clone https://github.com/crooks/PyClean.git
  Cloning into 'PyClean'...
  remote: Enumerating objects: 715, done.
  remote: Total 715 (delta 0), reused 0 (delta 0), pack-reused 715
  Receiving objects: 100% (715/715), 130.10 KiB | 0 bytes/s, done.
  Resolving deltas: 100% (452/452), done.
```
2. Move the newly created PyClean directory to your _<pathetc>/filter_ directory:
```shell
mv ./PyClean /etc/news/filter/pyclean
```
3. Rename _<pathetc>/filter/pyclean/pyclean.py_ to _<pathetc>filter/filter_innd.py_:
```shell
mv /etc/news/filter/pyclean/pyclean.py /etc/news/filter/filter_innd.py
```
4. Create the followinf 2 directories _<pathetc>/filter/pyclean/etc_ and _<pathetc>/filter/pyclean/lib_:
```shell
mkdir /etc/news/filter/pyclean/etc /etc/news/filter/pyclean/lib
```
5. Apply rights to user _news_ on the _<pathetc>/filter/pyclean_ directory:
```shell
chown -R news: /etc/news/filter/pyclean
```

# CONFIGURATION

PyClean will look for its configuration files in various places. One of them is the **Debian specific** _/etc/default/pyclean_.
You can make it this file the configuration file or just set some values that will later be overridden by another configuration file.  

Edit the _/etc/default/pyclean_ file:
```
[paths]
etc = /etc/news/filter/pyclean/etc
log = /var/log/news/pyclean
logart = /var/log/news/pyclean/articles
lib = /etc/news/filter/pyclean/lib

```

With such à _/etc/default/pyclean_ PyClean will look after a configuration file in _/etc/news/filter/pyclean/etc_

The default configuration file looks like this:
```
[logging]
level = info
format = %(asctime)s %(levelname)s %(message)s
datefmt = %Y-%m-%d %H:%M:%S
retain = 7
logart_maxlines = 20

[binary]
lines_allowed = 15
allow_pgp = true
reject_suspected = false
fasttrack_references = true

[emp]
ph_coarse = true
body_threshold = 5
body_ceiling = 85
body_maxentries = 5000
body_timed_trim = 3600
body_fuzzy = true
phn_threshold = 100
phn_ceiling = 150
phn_maxentries = 5000
phn_timed_trim = 1800
lphn_threshold = 20
lphn_ceiling = 30
lphn_maxentries = 200
lphn_timed_trim = 1800
phl_threshold = 20
phl_ceiling = 80
phl_maxentries = 5000
phl_timed_trim = 3600
fsl_threshold = 20
fsl_ceiling = 40
fsl_maxentries = 5000
fsl_timed_trim = 3600
ihn_threshold = 10
ihn_ceiling = 15
ihn_maxentries = 1000
ihn_timed_trim = 7200

[groups]
max_crosspost = 10
max_low_crosspost = 3

[control]
reject_cancels = false
reject_redundant = true

[filters]
newsguy = true
reject_html = true
reject_multipart = false

[hostnames]
path_hostname = true

[paths]
etc = /etc/news/filter/pyclean/etc
log = /var/log/news/pyclean
logart = /var/log/news/pyclean/articles
lib = /etc/news/filter/pyclean/lib

```

**Filter files** are stored in _<pathetc>/filter/pyclean/etc_ as defined sooner but are not created.
You may want to create them to avoid warnings in logfile:
```shell
for FILE in "/etc/news/filter/pyclean/etc/local_bad_groups \
              /etc/news/filter/pyclean/etc/bad_groups_dizum \
              /etc/news/filter/pyclean/etc/bad_posthost \
              /etc/news/filter/pyclean/etc/local_bad_from \
              /etc/news/filter/pyclean/etc/local_hosts \
              /etc/news/filter/pyclean/etc/bad_body \
              /etc/news/filter/pyclean/etc/local_bad_subject \
              /etc/news/filter/pyclean/etc/local_bad_body \
              /etc/news/filter/pyclean/etc/bad_cp_groups \
              /etc/news/filter/pyclean/etc/bad_groups \
              /etc/news/filter/pyclean/etc/bad_subject \
              /etc/news/filter/pyclean/etc/log_from \
              /etc/news/filter/pyclean/etc/ihn_hosts \
              /etc/news/filter/pyclean/etc/local_bad_cp_groups \
              /etc/news/filter/pyclean/etc/bad_crosspost_host \
              /etc/news/filter/pyclean/etc/good_posthost \
              /etc/news/filter/pyclean/etc/bad_from"
  do touch ${FILE};
  chown news: ${FILE};
done
```

To allow PyClean to apply specific filters on local articles, you must set your local hostnames in _<pathetc>/filter/pyclean/etc/local_hosts_:
```
/old\.example\.net/       20220101
/server\.example\.net/    20420101
```

Most of the filter files use the same syntax wich has 2 parts on each line:
* One regular expression between slashes
* An expire date in YYYYmmdd format

You may need to restart INN at first startup. In any case after you modified a filter file you can just reload the INN python filter:
```shell
sudo -u news /usr/lib/news/bin/ctlinnd reload filter.python "PyClean Initial load"
OK
```

Finally check you logs, all should be running as expected.
