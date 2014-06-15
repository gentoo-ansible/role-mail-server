Ansible role for a mail server
==============================

This role installs Postfix (MTA), Dovecot (IMAP), DSPAM (antispam), ClamAV (antivirus) and OpenDKIM on Gentoo Linux.

TODO

Setup firewall
--------------

Open these ports:

*  TCP 25 ... SMTP for inbound e-mails.
*  TCP 465 ... SMTP over TLS for sending e-mails.
*  TCP 993 ... IMAP over TLS.
