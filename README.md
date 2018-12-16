# -Howto-setup-back-ups-for-your-files-and-server-folders
Use one simple script named ./execute_backup to securely copy website folders from one server to another, followed through here in entirety.





This is a simple guide on how to use rsync, your system crontab, and simple key based authentication between machines for server to server back up of files and folders. I like this method because of it's low overhead and simple learning curve. We're going to be using commands and utilities most machines will come with preinstalled. High reliability and / rate limiting transfers is one tradeoff, which I and most others can live with for a LEB backup. I am starting with a clean 200 gig backup VM running Debian 6 32bit w/FTP and very few services running on her.




/etc/crontab




Note the location above. It is a standard path for your system crontab. I like to make use of this because changes made here will be automatically picked up which I have found cuts down on commands.




What I like to do first is to come up with a location to store backed up files and folders. And there's no better place like tilda. So login to your server and find yourself at ~/ which is your root home folder.




Now Let's create some executable files so our crontab will look cleaner when we ultimately have 5,10,15 backups running /day.




#root@node:~$ touch execute_backup 
#root@node:~$ nano execute_backup




Now you are working with the file that will execute your backup command. For simplicity sake I get away with most backups this way, using rsync and simple arcfour (Alleged RC4) encryption. As with all things this can be expanded upon including the use of a -v verbose flag But that can result in a lengthy Email confirmation. Also beyond the scope of this tutorial is your MTA. In general your backup server may be using images that don't start you off with any installed mailing agent. In my case a simple aptitude install sendmail / and aptitude install mailutils will get you the mail command below. I actually prefer heirloom-mailx because it is the smallest package for install.




rsync -ax --timeout=30 --rsh="ssh -p22 -c arcfour -l root" serverhostaddr:/var/www/vhosts/ /root/vhosts_backup | mail -s "Backup DONE for your-server (hostname)" MAIL@DOMAIN.COM




Above is the only command that really NEEDS to go in this executable file. What we're doing here is basically executing rsync -a recursively -x bounded with a timeout that helps if networks are slow, using a remote shell line that lets you specify port using arcfour encryption and to login as root. Then it connects to your server, backs up the folder vhosts/ to a renamed backup folder and ends with an Email that contains little more than a subject and your backup node. After getting this line into your executable you can ctrl+x, save your changes and test out the file using the following commands:




#root@node:~$ chmod +x execute_backup 
#root@node:~$ ./execute_backup




So by this point your backup executable seems like it may be working fine But how to automate this so it runs while you're catching some winks? Easily done using key based server authentication that will effectively allow the involved machines to "trust eachother" for SSH.




#root@node:~$ ssh-keygen 
#root@node:~$ [Enter]
#root@node:~$ [Enter] 
#root@node:~$ [Enter] 
#root@node:~$ ssh-copy-id -i ~/.ssh/id_rsa.pub serverhostaddr 
#root@node:~$ [Pass] 
#root@node:~$ ssh serverhostaddr




The commands above use some simple utilities to generate SSH keys and copy them over to the server you are backing up, defined by serverhostaddr. You ofcourse want this to be the Ip of the machine. And if your ports differ that is alright. Just make sure you have the appropriate port in your executable, and modify the above command to be:




#root@node:~$ ssh-copy-id -i ~/.ssh/id_rsa.pub '-p porthere serverhostaddr' 
#root@node:~$ [Pass] root@node:~# ssh -pporthere serverhostaddr




Note that the final line to ssh to your server should result in a good connection. Don't leave yourself logged in there! But that is useful to confirm the connection and perhaps the folders you want to backup, Just remember to exit to get you back to your backup server CLI.




Now open up your system crontab by running the following command:


root@node:~$ nano /etc/crontab


And under your environment variables, before other cron jobs let's do:




14 03 * * * root /root/execute_backup




All we did with the line above was to setup a daily cron job running at 03 14, executing as root the file in your root home that is well...executable. This shall execute EVERY DAY. From this simple line you have setup your cron job, you can exit, save changes, and not have to worry about any more complications or mail address setups through cron commands or the crontab file itself. I prefer this method and some variations of it because now you can have a cleaner crontab, multiple addresses for mailing the backups, and just a simpler setup overall. This method can be improved on in a few ways for reliability and reconciliation, But I do actively use it to backups not only hundreds of gigs per day of changing files, But also all my own sites and files. Enjoy!




As a final thought if the end user is having issues receiving mail notifications this is not a fault of the script, But one may wish to check whether the simplest of server sendmail utils is installed, and in most cases a simple command such as eg. aptitude install heirloom-mailx or mailutils will solve this. Check to make sure you have a service listening for mail using netstat or netcat after and nothing blocking or restricting a basic MTA such as a firewall. You need a basic server mail transport agent for the mail alerts, a simple MTA with no web configuration necessary, although if you have dovecot, exim4, Postfix etc you're already well past considerations for outside mail delivery, and if you are then having problems you may wish to try configuring a catch all account to see where the message is ending up.
