#Setting up diverse software tools

##Install browsers

	sudo apt-get install chromium
	
##Change the RPi hostname

<http://www.howtogeek.com/167195/how-to-change-your-raspberry-pi-or-other-linux-devices-hostname/>	
	
##General advice
	
[Tutorial – Prepare your Raspberry Pi to become a web server](http://www.raspipress.com/2012/09/tutorial-prepare-your-raspberry-pi-to-become-a-web-server/)  
[How set up a secure Raspberry Pi server installation](https://www.pestmeester.nl/)

##General software

The full home automation requires quite a list of software to be installed - details are interspersed throughout the set of documentation files.  Software not listed elsewhere are as follows:

- geany editor
- vim editor (nano is actually quite useful - easier than vim)
- chromium browser

##Install samba

To access files on your RPi directory from Windows.

<http://www.homeautomationforgeeks.com/samba.shtml>

	sudo apt-get install samba samba-common-bin
	sudo nano /etc/samba/smb.conf

Follow the instructions on the web site.



##Install pip

pip is not installed by default on RPi.

	wget https://raw.github.com/pypa/pip/master/contrib/get-pip.py
	sudo python get-pip.py

##Install mail sending capability using gmail as a server

This was a hit-and-miss process. In the end I installed both ssmtp and exim4.

Set up a new gmail account (or use your existing if you want).

This procedure assumes you are **not** using two-factor authentication.

Set your gmail account to use less secure apps:  
https://support.google.com/accounts/answer/6010255?hl=en-GB  
https://www.google.com/settings/security/lesssecureapps  

The procedure used here is taken from:  
<http://www.sbprojects.com/projects/raspberrypi/exim4.php>

###ssmtp

The ssmtp package is used to transport your mails to your provider, i.e. an MTA.
Install ssmtp

	sudo apt-get install ssmtp mailutils mpack
	#sudo apt-get install heirloom-mailx

Now edit the file `/etc/ssmtp/ssmtp.conf` as root 

	sudo nano /etc/ssmtp/ssmtp.conf

and add the next lines. Please note that some of the lines already exist and may need to be changed. Others don't exist yet and need to be added to the end of the file.

	root=postmaster
	mailhub=smtp.gmail.com:587
	hostname=ENTER YOUR RPI'S HOST NAME HERE
	AuthUser=YOU@gmail.com
	AuthPass=PASSWORD
	#useTLS=Yes
	useSTARTTLS=YES
	
You can obtain the hostname by opening a terminal and typing the command `hostname`.

Replace YOU with your gmail login name and PASSWORD with your gmail password. 
After this you're done. You don't even have to restart the SSMTP server (in fact, there is none).


###exim4

Install and configure

	sudo apt-get install exim4
	sudo dpkg-reconfigure exim4-config
	
Answer the questions according to the detail shown in 	http://www.sbprojects.com/projects/raspberrypi/exim4.php, exept that you must clean out the line stating with 127.0.0.1.

These answers worked for me (the answer for each screen in sequence):

1. Move to second entry `mail sent by smarthost; received via SMTP or fetchmail`
2. Set system mail name as the current RPi hostname
3. Set the `IP addresses to listen on for incoming SMTP`, by removing the 127.0.0.1 entry. It seems that either an empty line or the gmail server `smtp.gmail.com:587` can be used here.  
4. Set `Other destinations for which mail is accepted` to your RPi hostname
5. `Machines to relay` for must be set to the gmail server `smtp.gmail.com:587`
6. `IP address or hostname for outgoung smarthost` must be set to the gmail server `smtp.gmail.com:587`
7. `Hide local mail name in outgoing mail` must be set to `No`
8. `Keep numnber of DNS...` must be set to `No`
9. 	`Delivery method...` must be set to the second option `Maildir format in home directory`
10. `Split configuration in to small files` must be set to `No`


If you get this message: `IPv6 socket creation failed: Address family not supported by protocol`, look at this [short detour](http://www.geeklee.co.uk/basic-exim-mta-relay-on-raspberry-pi-with-raspbian/).  I had to perform the following:

Edit the file /etc/exim4/update-exim4.conf.conf (e.g. with nano)

	sudo nano /etc/exim4/update-exim4.conf.conf
	
Remove ::1 from dc_local_interfaces section. Save and exit. Delete the paniclog:

	sudo rm /var/log/exim4/paniclog

Now you'll have to enter your account details. 

	sudo nano /etc/exim4/passwd.client
	
Add these lines to the end of the file:

	gmail-smtp.l.google.com:YOU@gmail.com:PASSWORD
	*.google.com:YOU@gmail.com:PASSWORD
	smtp.gmail.com:YOU@gmail.com:PASSWORD	

Replace YOU with your gmail login name and PASSWORD with your gmail password. 

Update and restart exim4

	sudo update-exim4.conf
	sudo /etc/init.d/exim4 restart

###Setting aliases

Some processes, for instance crontabs, can send mails to root or other system users. If you don't want to miss any of them you can setup some aliases. You can do that by editing the file /etc/aliases. 

	sudo nano /etc/aliases

Here's an example of its contents:

	# /etc/aliases
	mailer-daemon: postmaster
	postmaster: root
	nobody: root
	hostmaster: root
	usenet: root
	news: root
	webmaster: root
	www: root
	ftp: root
	abuse: root
	noc: root
	security: root
	root: pi
	pi: youremail@example.com

This tells the system to redirect all mail to root, while mail to root is redirected to the user pi, while mail to user pi is finally redirected to my own mail account. This way all system mail will eventually be sent to my own mail account, no matter to what local user the mail was originally sent.

The `change finger` command will setup a new linux finger name (this is like a nick name for the user) for the user name pi, with which you can identify the source of the e-mail.

	sudo chfn -f "myfinger(nick)name" pi


###Testing

Test from the command line with one of these commands: 


	mail -s "test the mail thing" youremail@gmail.com

	echo "Test" | sudo sendmail -v youremail@gmail.com

	(echo Subject: Your subject ; echo "Mail message here"  ) | sudo sendmail -v youremail@gmail.com
	
The `-v` flag in the `sendmail` command activates the verbose mode: you can use this for debugging.    It seems that only the root user can use `sendmail`

###Additional reading on setting up email

smtp and exim4:   
[Prepare Your Pi To Send Mail Through Gmail](http://www.sbprojects.com/projects/raspberrypi/exim4.php)  
[How to set up smtp and send emails?](http://raspberrypi.stackexchange.com/questions/12405/how-to-set-up-smtp-and-send-emails)  
[SMTP Mail Setup](https://rpi.tnet.com/project/faqs/smtp)  
[How to send email on RPi with Ssmtp](http://www.techrapid.co.uk/raspberry-pi/send-email-raspberry-pi-with-ssmtp/)  
[ssmtp-to-send-emails](http://www.raspberry-projects.com/pi/software_utilities/email/ssmtp-to-send-emails)  
[Sending mail with Raspberry Pi](http://www.a-netz.de/2012/12/sending-mail-with-the-raspberry-pi/)  

postfix:  
[raspberry pi email server](https://samhobbs.co.uk/raspberry-pi-email-server)  
[raspberry-pi-email-server-part-1-postfix](https://samhobbs.co.uk/2013/12/raspberry-pi-email-server-part-1-postfix)  
[raspberry-pi-email-server-part-2-dovecot](https://samhobbs.co.uk/2013/12/raspberry-pi-email-server-part-2-dovecot)  
[Building a Raspberry Pi Mail Server](http://cha.rlesthom.as/blog/building_a_raspberry_pi_mail_server_how.html)   


Sending mail from the command line:   
[Sending mail from command line](http://www.linuxquestions.org/questions/linux-software-2/command-line-sendmail-356234/)  
[Sendmail, specify the subject line](http://fixunix.com/unix/82876-how-specify-subject-line-sendmail-command.html)   


Sending mail from Python:   
http://trevorappleton.blogspot.co.uk/2014/11/sending-email-using-python.html  
http://rosettacode.org/wiki/Send_an_email#Python  
https://www.raspberrypi.org/forums/viewtopic.php?f=32&t=94023  


other unsorted pages:    
https://www.raspberrypi.org/forums/viewtopic.php?f=36&t=68044  
<http://iqjar.com/jar/sending-emails-from-the-raspberry-pi/>  
<http://raspberrypi.stackexchange.com/questions/8180/raspberry-pi-as-an-email-server>  
<https://help.ubuntu.com/community/MailServer>  
<http://www.raspipress.com/2012/09/tutorial-install-postfix-to-allow-outgoing-email-on-raspberry-pi/>  
[send email on startup](https://gist.github.com/johnantoni/8199088)  
[Easily connect Raspberry Pi to Gmail, Facebook, Twitter ](http://mitchtech.net/connect-raspberry-pi-to-gmail-facebook-twitter-more/)  
[sending email from the command line](https://www.raspberrypi.org/forums/viewtopic.php?f=36&t=32077)   
https://www.raspberrypi.org/forums/viewtopic.php?f=36&t=32077  
http://www.cyberciti.biz/tips/linux-use-gmail-as-a-smarthost.html   
https://www.raspberrypi.org/forums/viewtopic.php?t=27937&p=288233


##Install supervisord to control mqttwarn

Supervisor is a client/server system that allows its users to monitor and control a number of processes on UNIX-like operating systems. Supervisor launches programs, monitors them for failure, logs their output, and restarts them. 

<http://jpmens.net/2014/02/13/in-my-toolbox-supervisord/>  
<http://supervisord.org/>

###Installing supervisor

<https://serversforhackers.com/monitoring-processes-with-supervisord>

To install Supervisord, we can simply run the following:

	sudo apt-get install -y supervisor

Installing it as a package gives us the ability to treat it as a service:

	sudo service supervisor start
	
The configuration file is here

	sudo nano /etc/supervisor/supervisord.conf

In order to control supervisor as a regular user, at the top of the file add the chown line underneath the chmod line:

	chmod=0700                  ; socket file mode (default 0700)
	chown=pi:pi                 ; socket file uid:gid owner
	
Towards the bottom of this file is written

	[include]
	files = /etc/supervisor/conf.d/*.conf

which implies that process configuration files can be placed in `/etc/supervisor/conf.d/`, because these file will be included in the main config file.

Save the following lines to a file as `/etc/supervisor/conf.d/mqttwarn.conf`

	[program:mqttwarn]
	directory = /home/pi/mqttwarn
	command = /home/pi//mqttwarn/mqttwarn.py
	autostart=true
	autorestart=true
	;startretries=1000
	stderr_logfile=/home/pi/log/mqttwarn.err.log
	stdout_logfile=/home/pi/log/mqttwarn.out.log
	user = pi
	environment= MQTTWARNINI="/home/pi/mqttwarn/mqttwarn.ini"

The first line points to the directory where mqttwarn is installed, change to suit your installation.  The same holds for the second line. In the third and fourth line enter your user home directory (if different from pi).

After changing the file reload and update

	sudo supervisorctl reread
	sudo supervisorctl update

check that mqttwarn is running:

	sudo supervisorctl
	-> type exit to leave the tool

When in supervisor you can type `help` to get a list of commands.

Starting/stopping mqttwarn:

	sudo supervisorctl stop mqttwarn
	sudo supervisorctl start mqttwarn

see the status:

	sudo supervisorctl status
		
		
		
		
		
		
	
	
	


	