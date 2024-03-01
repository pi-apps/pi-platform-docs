# Security Policy
The site is at risk of being hacked by SQL injection This is due to a security vulnerability, which is the hidden link to WordPress located in "https://minepi.com/robots.txt" 
(disallow: /wp-login.php) The hacker can hack into WordPress ( https://minepi.com/wp-login.php) and control the pi network website settings To hide it, you can change the direction of the link so that if a hacker enters it, it leads to another site, and that is from the WordPress settings.

## Supported Versions

Use this section to tell people about which versions of your project are
currently being supported with security updates.

| Version | Supported          |
| ------- | ------------------ |
| 5.1.x   | :white_check_mark: |
| 5.0.x   | :x:                |
| 4.0.x   | :white_check_mark: |
| < 4.0   | :x:                |

## Reporting a Vulnerability

Use this section to tell people how to report a vulnerability.

Tell them where to go, how often they can expect to get an update on a
reported vulnerability, what to expect if the vulnerability is accepted or
declined, etc.
