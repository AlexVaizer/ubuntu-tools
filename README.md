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

# JSON Config File example
```json
{
    "title": "vpsId.countryCode.example.com",
    "megazordCombined?": false,
    "username":"ubuntu", 
    "cockpitUsername": "vzr", 
    "cockpitUserPassword": "someRanDomPhraze834587", 
    // All all the above may be used  anywhere in config below, as 
    // $COCKPIT_USER_PASSWORD, $COCKPIT_USERNAME, $USERNAME, $TITLE
    // those be replaced with proper values from this config or 
    // command line params with run backup.rb (latter has higher priority)
    "softwares": [
        // Software: {
        //      name: String,
        //      customAptRepoCommands: [String, String...]
        //      aptPackages: [String, String...] <== Those will be installed on the beginning on restore procedure using 'apt install -y'
        //      backup: [
        //          {
        //              name: String,
        //              type: "DIR" or "FILE" or "COMMAND" <== COMMAND type of backup is not being restored!!! You should add it into a separate step into pre/post restore stages
        //              path: String 
        //          }
        //      ],
        //      preRestore: [
        //          these are commands that are being run before files/dirs from 'backup' section are restored
        //          {
        //              name: String,
        //              command: String 
        //          }
        //      ],
        //      postRestore: [
        //          these are commands that are being run after files/dirs from 'backup' section are restored
        //          {
        //              name: String,
        //              command: String 
        //          }
        //      ]
        // }
        {
            "name": "cockpit",
            "customAptRepoCommands": ["mkdir -p /etc/apt/keyrings;curl -fsSL https://packages.openvpn.net/packages-repo.gpg | tee /etc/apt/keyrings/openvpn.asc", "echo \"deb [signed-by=/etc/apt/keyrings/openvpn.asc] https://packages.openvpn.net/openvpn3/debian $(lsb_release -c -s) main\" | tee /etc/apt/sources.list.d/openvpn-packages.list; apt update"],
            "aptPackages": ["cockpit", "python3-openvpn-connector-setup"],
            "backup": [
                {
                    "name": "cockpit-user-ssh",
                    "type": "DIR",
                    "path": "/home/$COCKPIT_USERNAME/.ssh"
                },
                {
                    "name": "cockpit-override-conf",
                    "type": "FILE",
                    "path": "/etc/systemd/system/cockpit.socket.d/override.conf"
                },
                {
                    "name": "some-generic-command",
                    "type": "COMMAND",
                    "path": "iptables-save > $BACKUP_PATH/network/iptables.txt"
                }
            ],
            "preRestore": [
                {
                    "name": "add cockpit user",
                    "command": "useradd -m -p $(openssl passwd -6 $COCKPIT_USER_PASSWORD) -s /bin/bash $COCKPIT_USERNAME; usermod -aG sudo $COCKPIT_USERNAME; mkdir -p /home/$COCKPIT_USERNAME/.ssh; mkdir -p /etc/systemd/system/cockpit.socket.d"
                },
                {
                    "name": "restore thing that custom command did back up",
                    "command": "iptables-restore < $BACKUP_PATH/network/iptables.txt; netfilter-persistent save"
                }
            ],
            "postRestore": [
                {
                    "name":"Restart cockpit",
                    "command": "systemctl enable --now cockpit.socket; systemctl restart cockpit.socket"
                }
            ]
        }
    ]
}

```

# Autogenerated restore.sh example
```Bash
ruby backup.rb --combine-megazord "1-network.json,3-nginx.json" --name 'vps-3' --username USER1 --cockpit-user-password SOMEPASSWORD --cockpit-username USER2
###==========================================================================================
###------ Backing up files
###------ Script Configuration:
###------   * Combine Megazord Mode: true
###------   * Config File(s) path: ["./softwares/1-network.json", "./softwares/3-nginx.json"]
###------   * Operational mode: false
###------   * Verbose output: false
###------   * Backup root folder: /Users/machintoshhd/Documents/scripts/linux-tools/vps-3-2025-12-08_19-49-34
###==========================================================================================
###------ Backup Configuration:
###------   * Server Name: vps-3
###------   * Softwares: ["network", "nginx"]
###------   * SSH User: USER1
###------   * Cockpit User: USER2
###------   * Cockpit User Password: SOMEPASSWORD
###==========================================================================================
###------ BACKING UP STARTED
###==========================================================================================
mkdir -p /Users/machintoshhd/Documents/scripts/linux-tools/vps-3-2025-12-08_19-49-34/network
iptables-save > /Users/machintoshhd/Documents/scripts/linux-tools/vps-3-2025-12-08_19-49-34/network/iptables.txt
mkdir -p /Users/machintoshhd/Documents/scripts/linux-tools/vps-3-2025-12-08_19-49-34/network/vip-interface-settings; cp -v /etc/systemd/network/vip.* /Users/machintoshhd/Documents/scripts/linux-tools/vps-3-2025-12-08_19-49-34/network/vip-interface-settings
cp -r /home/USER1/.ssh/ /Users/machintoshhd/Documents/scripts/linux-tools/vps-3-2025-12-08_19-49-34/network
cp -r /root/.ssh/ /Users/machintoshhd/Documents/scripts/linux-tools/vps-3-2025-12-08_19-49-34/network
mkdir -p /Users/machintoshhd/Documents/scripts/linux-tools/vps-3-2025-12-08_19-49-34/nginx
cp -r /etc/nginx/ /Users/machintoshhd/Documents/scripts/linux-tools/vps-3-2025-12-08_19-49-34/nginx
###==========================================================================================
###------ BACKING UP FINISHED
###------ ACTUAL FILES COPYING WAS SKIPPED AS NOOP MODE ENABLED
###==========================================================================================

###==========================================================================================
###------ RESTORE SCRIPT BELOW
###==========================================================================================

#!/bin/bash
set -x
###==========================================================================================
###------ This is an autogenerated restore script built on ["./softwares/1-network.json", "./softwares/3-nginx.json"] config
###------ Be sure it is being run as ROOT
###==========================================================================================
###------ Stage1: Installing custom repos and packages
DEBIAN_FRONTEND=noninteractive apt update; apt upgrade

apt install -y iptables-persistent netfilter-persistent nginx
###==========================================================================================

###==========================================================================================
###------ Stage 2: Setting up other pre-requisites
### Restore VIP interface
cp -v ./network/vip-interface-settings/vip.* /etc/systemd/network/; systemctl restart systemd-networkd
### Restore iptables rules
iptables-restore < ./network/iptables.txt; netfilter-persistent save
### Make sure user and folders exists
useradd -m -s /bin/bash USER1; usermod -aG sudo USER1; mkdir -p /home/USER1/.ssh; mkdir -p /root/.ssh
###==========================================================================================

###==========================================================================================
###------ Stage 3: Restoring backup files
### user-ssh-keys
cp -vr ./network/.ssh/* /home/USER1/.ssh
### root-ssh-keys
cp -vr ./network/.ssh/* /root/.ssh
### nginx-configs
cp -vr ./nginx/nginx/* /etc/nginx
###==========================================================================================

###==========================================================================================
###------ Stage 4: Restarting and enabling services
### Remove default site
rm -f /etc/nginx/sites-enabled/default; rm -f /etc/nginx/sites-available/default
### Enable nginx
systemctl enable --now nginx.service; systemctl restart nginx.service
###==========================================================================================

###==========================================================================================
###------ !!! END OF RESTORE SCRIPT
###==========================================================================================


###==========================================================================================
###------ All needed files were archived. See below for hints on how to copy backup and restore it
### scp command: to copy file FROM this server:
scp vps-3:/Users/machintoshhd/Documents/scripts/linux-tools/vps-3-2025-12-08_19-49-34.tar.gz ~/Desktop
###==========================================================================================

###------ To Retore:
### copy file to this server
scp ~/Desktop/vps-3-2025-12-08_19-49-34.tar.gz vps-3:/home/ubuntu/
### untar command:
tar -xzvf /home/USER1/vps-3-2025-12-08_19-49-34.tar.gz
### run restore script
cd /home/ubuntu/vps-3-2025-12-08_19-49-34/; sudo bash ./restore.sh
###==========================================================================================
```


