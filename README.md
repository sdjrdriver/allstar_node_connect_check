# Allstar Node Connection Check Script

**This script serves multiple purposes:**
- Check if your Allstar node that this script runs on (Origin Node) is connected to the node you want to remain connected to (Destination Node)
- If the scripts that the nodes are not connected, it will attempt to reconnect to the Destination Node.
- Logs the connection attempts and timestamp when they happen.

**For the script to run properly, 3 things must be set in the config variables**

- LOG_DIR: This is the folder where the script looks for a file called cron_check.txt
- ORIGIN_NODE_ID: This is your Allstar Node ID.
- DEST_NODE_ID: This is the Destination Node ID where you intend to be connected and for the script to check against.

The script can be run on it's own.

But it is best run via a cronjob if you are wanting to have it check every few minutes or more.

Here is the cronjob I use, this runs the command every 15 minutes and the autoconnect script located in a binary folder of my choice (often /usr/local/bin or /usr/bin depending on preferences) as the user that has the minimal amount of permissions to run the commands that are needed.

Unless it is absolutely required, I would not recommend doing this as root unless it is absolutely neccessary for security purposes.

```
*/15 * * * * <asterisk_user> <my_bin_folder>/autoconnect > /dev/null 2>&1
```

TO-DO:
Implement a retry system, so the script only attempts to reconned 3 times before completely failing to prevent overly aggressive connection attempts to destination nodes.
