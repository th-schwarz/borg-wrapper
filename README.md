# borg-wrapper

It's a very simple wrapper to [BorgBackup](https://borgbackup.readthedocs.io/en/stable/), short borg. The project is very flexible to customize, as it has few settings for different borg repository. Borg-wrapper automatically generates data for monitoring during backup, which is used by the bundled munin plugin.

Side note: I started the project for two reasons. I wanted to learn bash scripting and I needed a munin plugin for borg!

It is important to understand borg in order to use the borg-wrapper correctly!!!

The state of the project is beta! It runs stable on the author's server, but wasn't tested on other servers yet.

## Disclaimer

I'm not responsible for any data loss, hardware damage or broken keyboards. This guide comes without any warranty!

## Configuration

Here is the suggested directory / file structure:

```bash
├── /opt/borg-wrapper
│   ├── borg-wrapper   (the bash script)
│   ├── borg-wrapper-conf  (sets the configuration directory)
│   ├── borg-wrapper-munin  (the munin plugin)
├── /etc/borg-wrapper
│   ├── conf   (basic settings for borg and borg-wrapper)
│   ├── conf-vmail  (configuration named 'vmail')
```

### Basic Settings

The configuration file ```/etc/borg-wrapper/conf``` contains basic settings:

```bash
# basic variables for borg
export BORG_BASE_DIR="/var/lib/borg"
export BORG_CONFIG_DIR="/etc/borg"
export BORG_CACHE_DIR="/var/lib/borg/cache"
export BORG_SECURITY_DIR="/var/lib/borg/security"
export BORG_KEYS_DIR="/etc/borg/keys"

# settings for borg-wrapper
# if it is empty, it's disabled
DIR_MONITORING=/var/lib/borg-wrapper/monitoring

DIR_MOUNT=/mnt/tmp

# munin valus could be overwritten in the repository settings
MUNIN_WARNING=25
MUNIN_CRITICAL=30
```

### Repository Settings

The configuration file ```/etc/borg-wrapper/conf-vmail```:

```bash
export BORG_REPO='ssh://user@your-storage.com:23/./vmail-repo/'
export BORG_PASSPHRASE='securepassword'

BACKUP_CMD="borg create $BORG_REPO::'backup-{now:%Y-%m-%d_%H:%M}'    \
    /var/vmail     \
    --exclude-caches > /dev/null

borg prune --force $BORG_REPO --prefix 'backup' --keep-daily=2"
```

## Commands

borg-wrapper CONFIG-NAME COMMAND [BORG-PARAMS] [OPTION]
<pre>
CONIG_NAME: The name of the repository configuration.
OPTION:
  -d       Print some debug info.
COMMAND:
  backup   Start the backup of the named configuration, e.g.
           - borg-wrapper vmail backup
  info     Print infos of a repo or a backup, if specified.
           Archive filters can be use, eg.
           - borg-wrapper vmail info "--last 1"
           - borg-wrapper vmail info "vmail-2021-10-31_16:40"
  list     List all backups of a repo.
  mount    Mount a borg backup, which can be selected from a list.
  umount   Unmount a borg mount point.
</pre>

## Munin Plugins

There are two different munin plugins. One shows how many hours have passed since the last backup and the other shows the duration of a backup. Of course this is displayed for each defined backup.<br>
To activate the plugins, the following static links must be set:

```bash
/etc/munin/plugins/borg-wrapper-munin -> /usr/local/sbin/borg-wrapper-munin*
/etc/munin/plugins/borg-wrapper-munin-duration -> /usr/local/sbin/borg-wrapper-munin*
```

![borg-wrapper_screenshot](https://user-images.githubusercontent.com/1237847/157925469-6c4a8052-ac30-4362-bb47-d9acc150b45e.png)
