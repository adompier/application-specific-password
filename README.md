# Application Specific Passwords for Roundcube
Application specific password let you sign in to your account securely when using third-party apps with your email account.
It's recommended to install this along side of Roundcube two-factor authentication.

This repo is a fork of [https://github.com/dweuthen/roundcube-application_passwords](https://github.com/dweuthen/roundcube-application_passwords).


License
-------
This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program. If not, see http://www.gnu.org/licenses/.

Installation
------------
- Clone from github:
    roundcube/plugins

```
 git clone https://github.com/adompier/application-specific-password.git application_passwords
```
- Activate the plugin into roundcube/config/config.inc.php:

```
    $rcmail_config['plugins'] = array('application_passwords');
```
Configuration
-------------
Copy *config.inc.php.dist* to *config.inc.php* and set the options as described
within the file.

The SQL queries in config.php.dist work with belows example setup.

## Example Setup

Table __vpopmail__ as used with Dovecot SQL driver

| Field     | Type       | Null | Key | Default | Extra |
| --------- | ---------- | ---- | --- | ------- | ----- |
| pw_name   | char(32)   | NO   | PRI | NULL    |       |
| pw_domain | char(96)   | NO   | PRI | NULL    |       |
| pw_passwd | char(255)  | YES  |     | NULL    |       |
| pw_uid    | int(11)    | YES  |     | NULL    |       |
| pw_gid    | int(11)    | YES  |     | NULL    |       |
| pw_gecos  | char(48)   | YES  |     | NULL    |       |
| pw_dir    | char(160)  | YES  |     | NULL    |       |
| pw_shell  | char(20)   | YES  |     | NULL    |       |
| 2FA       | tinyint(1) | NO   |     | 0       |       |

The 2FA field was added to trigger application specific password for remote hosts. All
local requests are assumed to be from roundcube.

The table used in this example is created with the 
following SQL statement:

```
 CREATE TABLE `applications` (
   `username` varchar(128) NOT NULL,
   `domain` varchar(128) NOT NULL,
   `application` varchar(128) NOT NULL,
   `password` varchar(255) DEFAULT NULL,
   `created` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP
 ) ENGINE=InnoDB DEFAULT CHARSET=latin1
```
In order to authenticate applications in Dovecot the following "password_query" is 
used:

```
password_query = SELECT CONCAT(pw_name, '@', '%d') AS user, \                                                                                                                                                                           
  if(2FA = '1' and ('%r' != '%l'),t2.password,pw_passwd) AS password, \                                                                                                                                                                 
  pw_dir as userdb_home, \                                                                                                                                                                                                                                                                                                                                                                                                                              
  FROM `vpopmail` \                                                                                                                                                                                                                     
  LEFT JOIN `applications` as t2 on t2.username = '%n' and t2.domain='%d' \                                                                                                                                                       
  WHERE pw_name = '%n' AND pw_domain = '%d' \                                                                                                                                                                                           
  AND ('%r' = '%l' or ('%r' != '%l' and 2FA='0') or (2FA='1' and ('%r' != '%l') and encrypt('%w', t2.password) = t2.password))  limit 1                                                                                                                                                                          D ('%r' = '%l' or ('%r' != '%l' and 2FA='0') or (2FA='1' and ('%r' != '%l') and encrypt('%w', t2.password) = t2.password))  limit 1 
```
The "user_query" is still set to query the user table:

```
 user_query = \                                                                                                                                                                                                                          
   SELECT pw_dir AS home, \                                                                                                                                                                                                                                                                                                                                                                                                                                   
   CONCAT('*:bytes=', REPLACE(SUBSTRING_INDEX(pw_shell, 'S', 1), 'NOQUOTA', '0')) AS quota_rule \                                                                                                                                        
   FROM vpopmail \                                                                                                                                                                                                                       
   WHERE pw_name = '%n' AND pw_domain = '%d' \                                                                                                                                                                                           
```

Notes
-----
Tested for Roundcube 1.5 on Centos 7 using PHP 7.3


