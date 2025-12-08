# Main Features:
 * Backs up Softwares on Ubuntu distros based on JSON Config File(1)
 * Generates Bash script to restore(2) them back

```
Usage: sudo ruby backup.rb --conf PATH -v --cockpit-user-password PASSWORD
        --conf PATH                  Path to config file. (Default: ./softwares/.template_config.json)
    -n, --name SERVERNAME            Server Name, sent via this param has highest priority. Second Priority is CONFIG['title']
    -p STRING,                       Password for a cockpit user, sent via this param has highest priority. Second Priority is CONFIG['cockpitUserPassword'], 'someRanDomPhraze834587' if empty in both places
        --cockpit-user-password
    -c, --cockpit-username STRING    Username for a cockpit user, sent via this param has highest priority. Second Priority is CONFIG['cockpitUsername'], nil if empty in both places
    -u, --username STRING            Username for a ssh user, sent via this param has highest priority. Second Priority is CONFIG['username'], nil if empty in both places
    -o, --[no-]op                    Enable (--op) or disable (--no-op) actual files copying. (Default: --no-op)
    -v, --verbose                    Enable verbose output for files copying. (Default: false)
        --combine-megazord SOFTWARES Enable JSON combining from softwares list, use file names from ./softwares/ folder, f.e.: --combine-megazord '1-network.json,3-nginx.json'. (Default: false)
    -h, --help                       Prints this help message.
``` 

(1) JSON Config File data model check in `./softwares/.template_config.json`

(2) restore.sh example see in `backup.rb.output.EXAMPLE.md` file
