# !!!!!I take no responsibility if this destroys your Pi or eats your mail !!!!!!

Now onto the interesting bits

# Running Zimbra on ARM64 / RPi4

I personally have this running in docker on https://github.com/GriffinPlus/docker-zimbra

## Setup private repo with all the ARM64 packages (or build yourself at your own peril, took me two weeks)

//build zimbra installer
./build.pl --build-no=1713 --build-ts=`date +'%Y%m%d'` \
--build-release=YOLO --build-release-no=9.0.0 \
--build-release-candidate=GA --build-type=FOSS \
--build-thirdparty-server=zimbra-files.coding.kiwi.nz --no-interactive


... fill in how to create private repo
Scan the packages to create your repo
`dpkg-scanpackages -m . > Packages && apt update`

and add the following extra source to sources.list.d
`deb [trusted=yes] file:/home/git/packages /`

## Getting Java to work
build java 13 for ARM64
SOLVED - build provided.

```bash
rm /opt/zimbra/common/lib/jvm/java/lib/security/cacerts
cp /etc/ssl/certs/java/cacerts /opt/zimbra/common/lib/jvm/java/lib/security/cacerts
chown zimbra:zimbra /opt/zimbra/common/lib/jvm/java/lib/security/cacerts
```

Fixing mariaDB

need to set the root localhost password to blank

`ALTER USER 'root'@'localhost' IDENTIFIED VIA mysql_native_password USING PASSWORD('');`

then run 
`$ sudo -Hiu zimbra /opt/zimbra/libexec/zmmyinit` 

might need to run `./libexec/zmsetup.pl` a few times.


##Setting up letsencrypt for certification
I recommend using an API key over dns verification as its much easier e.g cloudflare

### cron
need to setup cron for letsencrypt

### syslog
For some reason the syslog configuration for zimbra has changed or wasn't updated in github, you'll need to update the
syslog/zimbra file to be owned by syslog and not zimbra otherwise the zmcontrol incorrectly states things aren't 
running and logging in general doesn't work, e.g monitoring.

## setuid
need to copy setuid-linux.so and delete the macos one from /opt/zimbra/common/jetty_home/lib/setuid/

check version of setuid in /opt/zimbra/jetty/start.d/setuid.ini matches /opt/zimbra/common/jetty_home/lib/setuid
otherwise it won't start

## Choosing to build yourself
I've uploaded my commits to mikey0000/zm-build but you'll need to update zm-zcs-lib to pull in jna-4.3.0 instead of 3x
I've also got commits to 

## building syslog4j
you will need to set the java home to `export JAVA_HOME=/opt/zimbra/common/lib/jvm/java`
so that you can build it against the jvm in zimbra
