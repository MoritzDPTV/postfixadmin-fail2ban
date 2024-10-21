# Fail2Ban Jail and Filter Configuration for PostfixAdmin
In order to secure the login interfaces from [PostfixAdmin](https://github.com/postfixadmin/postfixadmin) (setup.php, login.php, users/login.php) against brute-force attacks, [Fail2Ban](https://github.com/fail2ban/fail2ban) can be used. The following configurations are tested with Fail2Ban v0.11.1 and PostfixAdmin v3.3.11.

## Configure the Local Jail
Append the new jail to the end of the local jail file */etc/fail2ban/jail.local*:
```
[postfixadmin]
enabled  = true
port     = http,https
filter   = postfixadmin
logpath  = /var/log/apache2/postfixadmin_error.log
```

Note: The path of the log file must be adjusted if your PostfixAdmin is logging elsewhere, e.g. in the NGINX log.

## Create the According Filter
Create a new filter file */etc/fail2ban/filter.d/postfixadmin.conf* and insert:
```
[Definition]
failregex = \[client <HOST>(:\d{1,5})?\] PostfixAdmin (.*) login failed

ignoreregex =
```

## Restart and Check Fail2Ban
Restart the Fail2Ban instance and check if it is running and if the new jail is listed:
```sh
sudo systemctl restart fail2ban
sudo systemctl status fail2ban
fail2ban-client status
fail2ban-client status postfixadmin
```

## Additional for PostfixAdmin Versions <= v3.3.11
PostfixAdmin v3.3.11 does not include logging for failed login attemps in the setup interface yet. Therefore, this error log must be added manually to the setup.php of the PostfixAdmin installation (e.g. at /var/www/postfixadmin/public/setup.php), as shown in this official [commit](https://github.com/postfixadmin/postfixadmin/commit/9fbb2fcc14dce919eb4f73af4f2cb5a31baa461a). Furthermore, the error loggings of the other two login interfaces should be compared with those of v3.3.11. In case they differ from one another, use those of v3.3.11 to ensure that the filter matches the loggings. No restart needed after these adjustments.
