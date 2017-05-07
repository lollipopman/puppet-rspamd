# rspamd

#### Table of Contents

1. [Description](#description)
1. [Setup - The basics of getting started with rspamd](#setup)
    * [What rspamd affects](#what-rspamd-affects)
    * [Beginning with rspamd](#beginning-with-rspamd)
1. [Usage - Configuration options and additional functionality](#usage)
1. [Reference - An under-the-hood peek at what the module is doing and how](#reference)
1. [Limitations - OS compatibility, etc.](#limitations)
1. [Development - Guide for contributing to the module](#development)

## Description

This module installs and manages the Rspamd spam filter, and provides resources
and functions to configure the Rspamd system. It also optionally installs the 
rmilter daemon to integrate rspamd into an MTA that supports it, and manages the 
rspamd.com apt-stable repository.
It does, however, not configure any of those systems beyond the upstream defaults.

This module is intended to work with Puppet 4 and has been tested with 
Rspamd 1.5.7 and rmilter 1.10.0 on Ubuntu 16.04. Patches to support other setups are welcome.

## Setup

### What rspamd affects

By default, this package...
* adds rspamd.com/apt-stable to your APT repository list
* installs rspamd and rmilter packages
* recursively purges all custom rspamd config (e.g. local.d and override.d directories)
* recursively purges all custom rmilter config (e.g. rmilter.local.conf and rmilter.conf.d)

### Beginning with rspamd

The simplest way to use this module is:

```puppet
include rspamd
```

This will setup the rspamd service and the rmilter service with the upstream
default configuration (except that rmilter legacy features are disabled).

## Usage

The rspamd::config resource can be used to specify custom configuration entries.
The easiest way to use it, is to put both the file and the hierachical config
key into the resource title:

```puppet
class { 'rspamd':
  rmilter => false
}
rspamd::config {
  'classifier-bayes:backend': value => 'redis';
  'classifier-bayes:servers': value => '127.0.0.1:6379';
  'classifier-bayes:statfile[0].symbol': value => 'BAYES_HAM';
  'classifier-bayes:statfile[0].spam':   value => false;
  'classifier-bayes:statfile[1].symbol': value => 'BAYES_SPAM';
  'classifier-bayes:statfile[1].spam':   value => true;
}
```

This results the following config file `/etc/rspamd/local.d/classifier-bayes.conf`:
```
# This file is managed by Puppet. DO NOT EDIT.
backend = redis;
servers = "127.0.0.1:6379";
statfile {
  spam = false;
  symbol = 'BAYES_HAM';
}
statfile {
  spam = true;
  symbol = 'BAYES_SPAM';
}
```


The provided `rspam::create_config_resources` and `rspam::create_config_file_resources`
functions allow for a much more convenient usage, especially with values stored in hiera:
```puppet
class profile::rspamd {
  rspamd::create_config_file_resources(hiera_hash("${title}::config", {}))
}
```
```yaml
profile::rspamd::config:
  classifier-bayes:
    backend: redis
    servers: "127.0.0.1:6379"
    statfile:
      - symbol: BAYES_HAM
        spam: false
      - symbol: BAYES_SPAM
        spam: true
```


## Reference

Here be dragons. Once I get around to it.

The important classes are somewhat documented (at least manifests/config.pp).

## Limitations

OS Versions tested:
* Ubuntu 16.04

Rspamd versions tested:
* 1.5.7

Rmilter versions tested:
* 1.10.0

Feel free to let me know if it correctly works on a different OS/setup, or 
submit patches if it doesn't.

## Development

You're welcome to submit patches and issues to the issue tracker on Github.

