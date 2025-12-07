# Main Features:
 * Backs up Softwares on Ubuntu distros
 * Generates Bash script to restore(2) them back
All of this configured via JSON Config File

Accepts Config file path as a first argument ("./backup_config.json" used if no arguments sent)

By default runs in NOOP mode (does not copy files, just generate output)
If you want it to actually backup things:
 * send 'true' as a second argument
 * must be run with sudo
 
_Example: `sudo ruby backup.rb ~/backups/configs/servername.example.com.json true`_

(1) JSON Config File data model check in `vm-configs/_template.json`

(2) restore.sh example see in `backup.rb.output.example` file
