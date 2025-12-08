# Main Features:
 * Backs up Softwares on Ubuntu distros based on JSON Config File(1)
 * Generates Bash script to restore(2) them back

```
Usage:
Usage: sudo ruby backup.rb --conf PATH -v --cockpit-user-password PASSWORD
    -c, --conf PATH                  			Path to config file. (Default: ./backup_config.json)
    -p STRING, --cockpit-user-password STRING   Password for a cockpit user. (Default: 'someRanDomPhraze834587')
        --[no-]op                    			Enable (--op) or disable (--no-op) actual files copying. (Default: --no-op)
    -v, --verbose                    			Enable verbose output for files copying. (Default: false)
    -h, --help                       			Prints this help message.
``` 

(1) JSON Config File data model check in `vm-configs/_template.json`

(2) restore.sh example see in `backup.rb.output.example` file
