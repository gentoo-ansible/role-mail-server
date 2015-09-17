Ansible role for a mail server
==============================

This role installs Postfix (MTA), Dovecot (IMAP), DSPAM (antispam), ClamAV (antivirus) and OpenDKIM on Gentoo Linux *.

If you need also a web interface (aka webmail), then try my [other role](https://github.com/jirutka/ansible-role-roundcube) to install [Roundcube](http://roundcube.net/).

TODO

_* When slightly modified it can be used also on other Linux distributions._


## Setup firewall

Open these ports:

*  TCP 25 ... SMTP for inbound e-mails.
*  TCP 465 ... SMTP over TLS for sending e-mails (authorized relay).
*  TCP 993 ... IMAP over TLS.
*  TCP 995 ... POP3 over TLS (disabled by default).


## Migrate mails from another IMAP server

Let’s say that you want to migrate all your mails from Gmail to your own mail server.

### Preparation

Gmail has labels, but IMAP knowns only folders.
If a message has multiple labels, it shows up in multiple IMAP folders, but it’s still the same message.
Dovecot currently doesn’t have such support, so the migration will copy the message to multiple folders and each instance will use up quota.

If you want to prevent mails duplication, then ensure that each message has at most one label.

### Let’s migrate!

Run the following command on the machine that hosts your mail server.
You have to provide your current Gmail address and password (the source) and email address of your new local mailbox (the target).

If you have another mail provider than Gmail, then replace `imap.gmail.com` with address of your actual IMAP server and change port number or SSL type if needed.

**WARNING: The _target_ mailbox must be empty! If not, the migration will probably fail and all mails in the _target_ mailbox will be lost!**

This command will _not_ modify your source mailbox (Gmail) in any way (the Dovecot authors say so).

```sh
doveadm \
  -o imapc_user=SOURCE-EMAIL@gmail.com \
  -o imapc_password=SOURCE-PASSWORD \
  -o imapc_host=imap.gmail.com \
  -o imapc_port=993 \
  -o imapc_ssl=imaps \
  -o ssl_client_ca_dir=/etc/ssl/certs \
  -o imapc_features="rfc822.size fetch-headers" \
  -o mail_prefetch_count=20 \
  -o mail_fsync=never \
  backup -R \
    -x '\All' -x '\Flagged' -x '\Important' \
    -u TARGET-EMAIL@example.org imapc:
```

Gmail has virtual folders: “All Mail”, “Starred” and “Important”. From the migration point of view this means that the migration should skip these folders, since their mails are in other folders anyway. That’s what these `-x` options do.

If the migration fails with _Error: Mailbox INBOX sync: mailbox_delete failed: INBOX can't be deleted_, then delete content of `$mailsrv_mailboxes_dir/$domain/$username` (e.g. `/var/lib/dovecot/encom.org/flynn`) and try it again.


## Links

Some useful resources I have been using to configure a mail server and migrate mails.

* [NSA-proof your e-mail in 2 hours](http://sealedabstract.com/code/nsa-proof-your-e-mail-in-2-hours/) by Drew Crawford
* [Sovereign playbooks](https://github.com/al3x/sovereign) by Alex Payne
* [Postfix, dovecot, dspam and dovecot antispam](http://www.owlfish.com/thoughts/dovecot-antispam-2011-03-21.html) by Colin Stewart
* [Migrating from any IMAP/POP3 server to Dovecot via dsync](http://wiki2.dovecot.org/Migration/Dsync) by Timo Sirainen
* [Migration from Gmail to Dovecot](http://wiki2.dovecot.org/Migration/Gmail)


## License

This project is licensed under [MIT license](http://opensource.org/licenses/MIT).
