## SMSauth (2006)

This is an old project that I built in 2006 and I haven't updated it since 
that year.

SMSauth implements a two factor authentication mechanism through a set of 
applications. This application tries to solve the problems of the traditional 
authentication mechanism.


## Licence

GNU GPL version 2.


## Goals

SMSauth is composed by a PAM authentication module, which generates OTPs, and a
server that sends those OTPs through SMS to the user.


## Module services provided

The PAM service supported is auth.


## Requeriments

To install the pam_smsauth module and be able to run the SMSauth server, the 
following packets must be installed:


* PAM (Pluggable Authentication Modules):

	- libpam0g – PAM library.

	- libpam0g-dev – PAM development files.

	- libsqlite3 (sqlite3) – SQLite 3 shared library.

* Python:

	- simplejson - Simple, fast, extensible JSON encoder/decoder for Python.

	- python-gammu – Python module to communicate with mobile phones.

	- libgammu0 – Mobile phone management library.


## Installation

Uncompress the file pam_smsauth.tar.bz2 and execute as root:
```
~# make
~# make install
```

This will create and install the pam_smsauth.so in /lib/security.

Then, execute as root the install.sh script. This will create the database that 
pam_smsauth requires and the directories that will contain the configuration 
files and the database.


## Configuration

1) Configuring pam

Edit the /etc/pam.d/ssh or login file. Between "@include common-auccount" and 
"@include common-auth", you can add the following settings:

* For debugging and testing: debug on, relaxed to avoid setup errors, always 
exclude root:

```
auth	requisite	pam_smsauth.so debug relaxed except=root
```

* The most simple setting (using the default values):

```
auth	required	pam_smsauth.so
```


* The secure mode:

```
auth	required	pam_smsauth.so debug otptype=alfanum otplen=8
```


These are the parameters that can be passed to the module:

	- debug
	- otplen = digit
	- otptype= [num|alfa|alfanum]
	- relaxed
	- expir = digit
	- except = string1
	- sockpath = string2


Values:

	string1 = [a-z]+ {,24}
	string2 =  [a-zA-Z]+ {,256}
	digit = [0-9]+


These are the  default values for some parameters:

	- otplen = 5
	- otptype = num
	- sockpath = /var/run/pam_smsauth_sock
	- expir = 5


The expir parameter expresses the time in minutes.

Finally IP/SMS gateways must be configured for SMSauth server, editing the 
/etc/sms_auth/server.conf file.


2) Adding a new user

You must complete the GECOS field in the following way:

[user_name;code_area;cel_number;company;]  

Note: don't forget the final ";".

Example:

```
~# adduser pipo
Adding user `pipo'...
Adding new group `pipo' (1001).
Adding new user `pipo' (1001) with group `pipo'.
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
Changing the user information for pipo
Enter the new value, or press ENTER for the default
        Full Name []: [pipo;261;5010203;cti;]
        Room Number []: 
        Work Phone []: 
        Home Phone []: 
        Other []: 
Is the information correct? [y/N] y

~# cat /etc/passwd | grep pipo

pipo:x:1001:1001:[pipo;261;5010203;cti;],,,:/home/pipo:/bin/bash
```

3) Or edit the GECOS field of the existent user, adding the user info in the 
specified format.


4) For ssh add to sshd_config file the following option:

```
UsePAM yes
```

