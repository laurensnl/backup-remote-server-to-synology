remote-server-to-synology-backup
================================

A push backup scheme that creates snapshot "rsync" backup of your remote
server and stores it on your Synology NAS or other Linux based server.
After a backup is complete other rolling hard linked snapshots are created
generating cycles of month, week, day and hourly snapshots.

The script will perform a backup of the SOURCE directory on the remote server
using rsync and create the monthly, weekly, daily and hourly snapshots 
afterwards. In case of a failure, a notification will be send to the 
Synology dashboard and a log entry will be made in the Synology log.
A log of all performed backups can be viewed in the Synology Diskstation 
dashboard -> Log center -> Log Search.
 
A more detailed log called 'backup-log.txt' will be stored in the 'logs' directory.


# DIRECTORY STRUCTURE

/volumeX/vpsbackups (or any other name)
 ↳ backups
 ↳ logs
 ↳ scripts
    ↳ linkdups
    ↳ vps_backup
    ↳ vps_backup_exclude
    ↳ vps_backup_prep

The script can also be used to backup a remote server to a non-Synology device.
In that case, remove all references to Synology features (synolog and synodsmnotify)


TO PERFORM A MANUAL BACKUP

> /volumeX/vpsbackups/scripts/vps_backup

# OPTIONS
   -u   update only, do not 'roll' backup into snapshot backup cycles
   -r   roll snapshots only (as needed), do not update the primary backup

Extra script/data files used by this script...
  home_backup_exclude  # exclude (and include) list of files (data caches)
  home_backup_prep     # prepare backup on backup server (make linked copy)


# LINKDUPS

The actual backup method involves the way "rsync" updates files that change.
When that happens any hardlink that is attached to that file in the backup
directory is broken.  As such the older 'snapshot' of the file is preserved
while the "current" backup gets a new version of the file, (the whole file).

This also means any files which are unmodified from the last "snapshot" to
the next remain hard linked together, so only one physical copy of each file
version is kept on disk, resulting in enormous disk space savings.  The disk
cost (not counting inodes) is the directory structure, and any files which
are either new or had been modified.

A detail discussion and examples of this technique is provided on the web
 Easy Snapshot-Style Backups with Rsync
 http://www.mikerubel.org/computers/rsync_snapshots/

There is one caveat with this system.  When a file is moved, renamed, or
worse a whole directory is renamed, rsync will also break the hardlink, even
though the file itself has not changed.  As such a secondary method of
're-linking' moved or renamed files is needed. This is achieved using the
"linkdups" script, and performed by the "home_backup_roll" script after the
rsync is complete, and before it generates the various backup cycles from
the new "current" backup, just made.

As a side effect any copies of the same file may also become hard linked
together in the backup.  To prevent lots of small (empty) but otherwise
unrelated files being re-linking together, the "linkdups" limits its task
to files larger than a few disk blocks.

THE LINKDUPS SCRIPT IS DISABLED IN THIS VERSION OF THE SCRIPT
Performance was bad and questionable links were being made when enabled.
Suggestions for a workaround are more than welcome!


# CREDITS

Anthony Thyssen -- February 2004 
Original script called home_backup

Edited by Laurens Maarschalkerweerd -- December 2014
Modified to backup a remote server to a Synology NAS


# PRE-REQUIREMENTS AND DEPENDANCIES

SSH connection with the server should be made without password using SSH
public key authentication

Install using IPKG:
- bc (terminal command /opt/bin/ipkg install bc)

To install IPKG on a Synology DS414:
http://www.noobunbox.net/synology/installer-ipkg-sur-un-nas-synology-ds414/
