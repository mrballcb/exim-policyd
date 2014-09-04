exim-policyd
============

Exim Policy Daemon

The primary purpose of this daemon is to allow Exim to call out to check
a user's quota so Exim can reject at SMTP time, as opposed to accepting the
email and trying to generate a bounce (to the usually spoofed sender address).

## System Configuration:

1. Put the exim-policyd.sysconfig file in /etc/sysconfig, renaming it to
   */etc/sysconfig/exim-policyd*.  Adjust contents to suit your requirements.

2. Put the exim-policyd.init file in /etc/init.d, renaming it to
   */etc/init.d/exim-policyd*.

3. Put the exim-policyd script in /usr/local/bin.

4. Enable the service to start at boot: `chkconfig --add exim-policyd`

5. Create log and pid directories: `mkdir /var/log/helper /var/run/helper`

6. Change ownership of the directories to the user the daemon is configured
   to run as (default "vmail"): `chown vmail: /var/log/helper /var/run/helper`

7. Start it with: `service exim-policyd start`

8. Test like this with a valid user and domain:

```
$ telnet localhost 8049
SHOW_QUOTA todd example.com
using=1804761 limit=1073741824 available=1071937063
QUIT
```

## Exim configuration:

1. Define this macro in the global section of your exim.conf:
 
```
OVERQUOTA_CHECK = ${readsocket{inet:localhost:8049}{CHECK_QUOTA $local_part $domain}}
```

2. Put this in the RCPT acl, somewhere before your normal accept:

```
defer   domains        = +local_domains
        verify         = recipient
        condition      = ${if eq{OVERQUOTA_CHECK}{1} {yes}{no}}
        message        = The mailbox for $local_part@$domain is full, deferring.
        log_message    = Recipient mailbox is full, deferring.
```

You will see a log line that looks like this (line breaks added by me):

```
2014-08-31 17:56:31 H=(remote.bilboesgold.co.zw) [197.221.231.110] X=TLSv1:AES128-SHA:128
  F=<lujdpiu@gmail.com> temporarily rejected RCPT <service@example.com>: 
  Recipient mailbox is full, deferring.
```
