@title = 'Upgrade to 0.8'
@toc = false

LEAP Platform release 0.8 introduces several major changes that need do get taken into account while upgrading:

* Dropping Debian Wheezy support. You need to upgrade your nodes to jessie before deploying a platform upgrade.
* Dropping BigCouch support. LEAP Platform now requires CouchDB and therefore you need to migrate all your data from BigCouch to CouchDB.

Here's how to upgrade from Wheezy nodes running BigCouch to Jessie nodes using CouchDB:

1. Follow [["migrating from BigCouch to plain CouchDB" => services/couchdb#migrating-from-bigcouch-to-plain-couchdb]], but only until the step where you removed BigCouch.
2. Now upgrade to Jessie (see below)
3. Continue with [["migrating from BigCouch to plain CouchDB" => services/couchdb#migrating-from-bigcouch-to-plain-couchdb]] at the point where you stopped for the first step, and deploy to the couchdb node.

Upgrade from Debian Wheezy to Jessie
------------------------------------------------

There are the [Debian release notes on how to upgrade from wheezy to jessie](https://www.debian.org/releases/stable/amd64/release-notes/ch-upgrading.html). Here are the steps that worked for us, but please keep in mind that there is no bullet-proof method that will work in every situation. USE AT YOUR OWN RISK.

    # keep a log of the progress:
    screen
    script -t 2>~/leap_upgrade-jessiestep.time -a ~/upgrade-jessiestep.script

    # ensure you have a good wheezy install:
    apt-get autoremove
    apt-get update
    DEBIAN_FRONTEND=noninteractive apt-get -y -o DPkg::Options::=--force-confold dist-upgrade

    # if anything is held, you need to resolve it before continuing:
    dpkg --audit
    dpkg --get-selections | grep 'hold$'

    # switch sources to jessie
    sed -i 's/wheezy/jessie/g' /etc/apt/sources.list
    echo "deb http://deb.leap.se/0.8 jessie main" > /etc/apt/sources.list.d/leap.list

    # remove pinnings to wheezy
    rm /etc/apt/preferences
    rm /etc/apt/preferences.d/*

    # get jessie package lists
    apt-get update

    # test if there is enough disk space for the upgrade
    apt-get clean
    apt-get -o APT::Get::Trivial-Only=true dist-upgrade

    # do first stage upgrade
    DEBIAN_FRONTEND=noninteractive apt-get -y -o DPkg::Options::=--force-confold upgrade

    # repeat dist-upgrade until it makes no more changes:
    DEBIAN_FRONTEND=noninteractive apt-get -y -o DPkg::Options::=--force-confold dist-upgrade

    # resolve any apt issues if there are some
    apt-get -f install

    reboot


### Issues

**Failure restarting some services for OpenSSL upgrade**

If you get this warning:

    The following services could not be restarted for the OpenSSL library upgrade:
      postfix
    You will need to start these manually by running '/etc/init.d/<service> start'.

Just ignore it, it should be fixed on reboot/deploy.
