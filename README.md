Yet another rsync based incremental backup utility.

# Purpose

An utility to maintain incremental backups in differnet directories.

The backups in each directory will contain entire image, but the incremental
nature comes from the fact that unchanged files will be hard linked sharing the
same space across backups.

# Quick Start

```bash
# Install.
sudo pip3 install yaribak

# Make a directory for backups.
mkdir -p /path/to/abc_backups

# Create a new backup in a subdir of destination.
# If run again, will hard link unchanged files from previous backups.
yaribak /path/to/abc /path/to/abc_backups
```

# Setup and Invocation

## Option 1: pip

This is the recommend way to install for general purpose usage.

```bash
# Installation.
sudo pip3 install yaribak

# Invocation.
yaribak --help
```

You can leave out the `sudo` in most distributions, or if you don't want to
backup with super-user privileges.

## Option 2: git clone

Cloning the git repo is necessary for development.

It can also be used as an alternative means to quickly test the functionality,
without installing it on the system with `pip install`.

```bash
# Clone repository.
mkdir -p /path/to/git
git clone https://github.com/hirak99/yaribak

# Invocation.
/path/to/git/yaribak.sh --help
```

# Usage

## Common Invocation and Arguments

```bash
yaribak \
  --source /path/to/source \
  --backup-path /path/to/backups

# Optional args (run with --help to view description) -
# --dryrun
# --verbose
# --only-if-changed
# --max-to-keep=3
# --exclude source_subdir1 --exclude source_subdir2 ...
```

Note: Care must be taken to use different backup directories for different source directories.

## Example Usage

Maintain backups of the home directory -

```bash
# Create the backup directory.
mkdir -p /path/to/homdir_backups

# Call this every time you want to backup, or put it in cron.
# Past backups will be kept.
yaribak \
  --source ~ \
  --backup-path /path/to/homedir_backups \
  --verbose \
  --exclude .cache
```

The following structure will be generated in the backup directory (for this
example, after 3 calls) -
```
$ ls path/to/homedir_backups
_backup_20220306_232441
_backup_20220312_080749
_backup_20220314_110741
```

Each directory will have a full copy of the source.

# Yaribak features

## Conserving Space

The primary reason to use this over a simple `cp -r` is that it saves space.

Any file that remains unchanged will be hard linked, effectively resulting in very little space consumption for multiple invocation if the source remains largely unchanged.

```bash
$ # Size of source directory.
$ du -sh ~
6.5G /home/user1

$ # Say following backups were created by multiple invocations -
$ ls path/to/homedir_backups
_backup_20220314_110741
_backup_20220314_110815
_backup_20220314_110903

$ # Assuming multiple backups were created with no change in source,
$ # total size of backup will not increase (by much).
$ du -h /path/to/homedir_backups
6.5G /path/to/homedir_backups
```

## Fault Tolerance

If a backup is stopped abruptly in the middle, yaribak will recover next time
you run it.

# Testing

From the package root, run -
```python
./runtests.sh
```

# Alternatives

## Comparison with Timeshift
[Timeshift](https://github.com/teejee2008/timeshift) is an excellent utility to
do one thing: backing up system.

It however cannot be used to backup any other directory, or to back up in user-selected destination.

This is an alternative to allow more control, but without conveniences like a
GUI and out-of-the-box automated backups.

# FAQ

## How do I set up periodic backups?
You will need to add the backup command to cron. There are [many excellent
tutorials](https://www.google.com/search?q=setting+up+cron+job+linux+tutorial),
for example [this one](https://opensource.com/article/17/11/how-use-cron-linux).

## Does it create hard links to original files?
No, even if they are on the same filesystem, backups will never be hard linked
to source by design. Creating hard link to original files will result in the
backups being modified if the original file is modified - that's something we do
not want.

## Can I use the same destination for different sources?
Using the same destination for different sources will result in the algorithm
not evaluating unchanged files correctly across invocations. So it is strongly
discouraged to use the same directory for different sources. Instead, assign
different destination directories for different sources.

## How do I restore?
You will need to manually copy all files from any of the backups to the source directory.

This can be done either by `cp -ar` -
```bash
# Save the existing files.
mv /path/to/source /path/to/source_old

# Restore.
cp -ar /path/to/backups/ysnap_20220314_110903/payload /path/to/source
```

Or using `rsync` -
```bash
# Warning: Existing files will be irrevokably modified or deleted.
rsync -aAXHv --delete \
  /path/to/backups/ysnap_20220314_110903/payload \
  /path/to/source
```
