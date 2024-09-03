# Installation Instructions

Pre-Requisites:

User that can control asterisk cli.
User that can access/execute scripts in the folder where you place the script.

1. Copy the 'autoconnect' script to a folder that can be accessed and is able to be executed from "/usr/local/bin for example"
```
cp <github_repo>/autoconnect /usr/local/bin/
```

1. Change the permissions of the file to be owned by the user that will be executing the script
```
chown user:user <script_folder>/autoconnect && chmod 755 <script_folder>/autoconnect
```

1. Set a cronjob to run the script at whatever time window that you would like. For example
```
crontab -e -u <user>

*/15 * * * * <asterisk_user> <script_folder>/autoconnect > /dev/null 2>&1
```
