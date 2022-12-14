Cassandra Backup and Restore with S3 Object Stores
====================
Shell script for creating and managing Cassandra Backups using S3 compatible object stores.
## Features
- Take snapshot backups
- Copy Incremental backup files
- Compress with gzip or bzip2 to save space
- Prune old incremental and snapshot files
- Execute Dry Run mode to identify target files

## Requirements
The `s3cmd` utility, available with 'pip install s3cmd'.
An existing S3 bucket 
Linux system with BASH shell. 
Cassandra 2+


## Usage
./cassandra-cloud-backup.sh [ options ] < command> 

### Examples
  - Take a full snapshot, gzip compress it with nice level 15,  use the /var/lib/cassandra/backups directory to stage the backup before
  uploading it to the S3 Bucket, and clear old incremental and snapshot files

 `./cassandra-cloud-backup.sh -b s3://cassandra-backups123/ -zCc -N 15 -d /var/lib/cassandra/backups backup`

  - Do a dry run of a full snapshot with verbose output and create list of files that would have been copied

  `./cassandra-cloud-backup.sh -b s3://cassandra-backups123/ -vn backup`

  - Backup and bzip2 compress copies of the most recent incremental backup files since the last incremental backup

`  ./cassandra-cloud-backup.sh -b s3://cassandra-backups123/ -ji -d /var/lib/cassandra/backups backup`

  - Restore a backup without prompting from given bucket path and keep the old files locally

 ` ./cassandra-cloud-backup.sh -b s3://cass-bk123/backups/host01/snpsht/2016-01-20_18-57/ -fk -d /var/lib/cassandra/backups restore`

  - List inventory of available backups stored in S3

 ` ./cassandra-cloud-backup.sh -b s3://cass-bk123 inventory`
 
  - List inventory of available backups stored in S3 for a different server

 ` ./cassandra-cloud-backup.sh -b s3://cass-bk123 inventory -a testserver01`

### Commands:

- backup        ---        Backup the Cassandra Node based on passed in options
- restore        ---      Restore the Cassandra Node from a specific snapshot backup  
- inventory      ---       List available backups
- commands      ---        List available commands
- options       ---        list available options

### Options:
 Flags:

  -a, --alt-hostname
    Specify an alternate server name to be used in the bucket path construction. Used
    to create or retrieve backups from different servers

  -B, backup
    Default action is to take a backup

  -b, --s3bucket
   S3 bucket used in deployment and by the cluster.

  -c, --clear-old-ss
    Clear any old SnapShots taken prior to this backup run to save space
    additionally will clear any old incremental backup files taken immediately
    following a successful snapshot. this option does nothing with the -i flag

  -C, --clear-old-inc
    Clear any old incremental backups taken prior to the the current snapshot

  -d, --backupdir
    The directory in which to store the backup files, be sure that this directory
    has enough space and the appropriate permissions

  -D, --download-only
    During a restore this will only download the target files from GCS

  -f, --force
    Used to force the restore without confirmation prompt

  -h, --help
    Print this help message.

  -H, --home-dir
    This is the $CASSANDRA_HOME directory and is only used if the data_directories, commitlog_directory,
    or the saved_caches_directory values cannot be parsed out of the yaml file. 

  -i, --incremental
    Copy the incremental backup files and do not take a snapshot. Can only
    be run when compression is enabled with -z or -j

  -j, --bzip
    Compresses the backup files with bzip2 prior to pushing to S3
    This option will use additional local disk space set the --target-gz-dir
    to use an alternate disk location if free space is an issue

  -k, --keep-old  
    Set this flag on restore to keep a local copy of the old data files
    Set this flag on backup to keep a local copy of the compressed backup and schema dump

  -l, --log-dir
    Activate logging to file 'CassandraBackup${DATE}.log' from stdout
    Include an optional directory path to write the file
    Default path is /var/log/cassandra

  -n, --noop
    Will attempt a dry run and verify all the settings are correct

  -N, --nice
    Set the process priority, default 10

  -p
    The Cassandra User Password if required for security

  -r,  restore
    Restore a backup, requires a --gcsbucket path and optional --backupdir

  -s, --split-size
    Split the resulting tar archive into the configured size in Megabytes, default 100M

  -S, --service-name
    Specify the service name for cassandra, default is cassandra use to stop and start service

  -T, --target-gz-dir
    Override the directory to save compressed files in case compression is used
    default is --backupdir/compressed, also used to decompress for restore

  -u
    The Cassandra User account if required for security

  -U, --auth-file
    A file that contains authentication credentials for cqlsh and nodetool consisting of
    two lines:
      CASSANDRA_USER=username
      CASSANDRA_PASS=password

  -v --verbose
    When provided will print additional information to log file

  -y, --yaml
    Path to the Cassandra yaml configuration file
    default: /etc/cassandra/cassandra.yaml

  -z, --zip
    Compresses the backup files with gzip prior to pushing to S3
    This option will use additional local disk space set the --target-gz-dir
    to use an alternate disk location if free space is an issue


###Cron Examples
- Full gzip compressed snapshot every day at 1:30 am with nice level 10

`30 1 * * * /path_to_scripts/cassandra-cloud-backup.sh -z -N10 -b s3://cass-bk123-vCcj -d /var/lib/cassandra/backups > /var/log/cassandra/$(date +\%Y\%m\%d\%H\%M\%S)-fbackup.log 2>&1`

- Incremental gzip compressed backups copied every hour nice level 10

`0 * * * * /path_to_scripts/cassandra-cloud-backup.sh -b -N10 s3://cass-bk123 -vjiz -d /var/lib/cassandra/backups > /var/log/cassandra/$(date +\%Y\%m\%d\%H\%M\%S)-ibackup.log 2>&1`

### Notes

The script must be run with sufficient privileges to be able to stop/start processes and create/delete directories and files within the working directories.

The restore command is designed to perform a simple restore of a full snapshot. In the event that you want to restore incremental backups you should start by restoring the last full snapshot prior to your target incremental backup file and manually move the files from each incremental backup in chronological order leading up to the target incremental backup file.  The schema dump is included in the snapshot backups, but if necessary it must also be restored manually.

Snapshots are taken at the system level, the script currently does not support backup or restore of an individual keyspace or columnfamily. 

In order to enable incremental backups, the `incremental_backups` option has to be set to true in the cassandra.yaml file.

###License
 Copyright 2016 Google Inc. All Rights Reserved.

 Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at
      http://www.apache.org/licenses/LICENSE-2.0
Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS-IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.  See the License for the specific language governing permissions and limitations under the License.

This is not an official Google product.
