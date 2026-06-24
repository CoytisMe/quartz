---
category: troubleshooting
tags:
  - wordpress
  - debug
  - php
  - errors
---
if you are interested the way to find source of critical error is to set wp debug to true in the wp-config file in the public root of the website. i generally use the file manager in cpanel for that as having ftp logins for each account i cumbersome. then there is verbose output on the webpage which normally indicated the failing code, which in turn is in a plugin or them file that needs updating...