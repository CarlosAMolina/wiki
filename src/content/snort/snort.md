## Contents

- [Manual summary](#manual-summary)
  - [Files](#files)
  - [Work with Snort](#work-with-snort)
  - [Logs](#logs)
  - [Resources](#resources)

## Manual summary

### Files

PDF section 11

- Configurations and rules: `/etc/snort`.
- Alerts will be written to `/var/log/snort`.
- Compiled rules (.so rules) will be stored in `/usr/local/lib/snort_dynamicrules`.

PDF section 12

- Configuration file: `/etc/snort/snort.conf`.
- Detection rules: `/etc/snort/rules/local.rules`.
- Detection rules identifiers: `/etc/snort/sid-msg.map`.

PDF section 15

- Pulled Pork combines all the rules into one file: `/etc/snort/rules/snort.rules`.

### Work with Snort

PDF section 12

- Check configuration file: `sudo snort -T -c /etc/snort/snort.conf -i eth0`.
- Run Snort: `sudo /usr/local/bin/snort -A console -q -u snort -g snort -c /etc/snort/snort.conf -i eth0`.

PDF section 13

Snort in alert mode

```bash
sudo /usr/local/bin/snort -q -u snort -g snort -c /etc/snort/snort.conf -i eth0
# Events won't be showed, the option `-Alerts` is required
```

Test Barnyard2: `/usr/local/bin/barnyard2 -V`.

Configure Barnyard2 db: `mysql -u root -p`.

Run Barnyard2 with the following command

```bash
sudo barnyard2 -c /etc/snort/barnyard2.conf -d /var/log/snort -f snort.u2 -w /var/log/snort/barnyard2.waldo \
-g snort -u snort
#error: warning ignoring corrupt/truncated waldofile '/var/log/snort/barnyard2.waldo'
```

PDF section 15

Stop Snort: `kill -SIGHUP <snort pid>`.

PDF section 16

Check services up

```bash
service snort status
service barnyard2 status
```

PDF section 17

Restart Apache: `sudo service apache2 restart`.

### Logs

PDF section 12

- Detection logs: `/var/log/snort`. Name `snort.log.XXX`, another name is `snort.u2.XXX` processed by Barnyard2.

PDF section 13

See MySQL events

```bash
mysql -u snort -p -D snort -e "select count(*) from event"
```

### Resources

Manual

<https://s3.amazonaws.com/snort-org-site/production/document_files/files/000/000/122/original/Snort_2.9.9.x_on_Ubuntu_14-16.pdf>