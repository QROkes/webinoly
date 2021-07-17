# Webinoly ![CI status](https://img.shields.io/badge/build-passing-brightgreen.svg)

**Optimized LEMP Web Server**

A powerful set of commands for doing just about anything you could wish.

Linux Ubuntu + Nginx + MariaDB (MySQL) + PHP is one of the most reliable and powerful configurations to host your websites. **With Webinoly you can set up your web server in just one step.**

Webinoly provides a set of tools and commands that facilitate web server administration.
- Unique commands to create, delete, disable sites.
- Free SSL certificates for your sites with Let’s Encrypt and automatic server configuration.
- HTTP/2 dramatically increases the speed of serving your content.
- PHP v8.0 and support for earlier versions if needed (7.4 or 7.3).
- FastCGI Cache and Redis Object Cache for your WordPress sites.
- Install WP in any subfolder and support for external databases, multisite and domain mapping.
- Reverse Proxy for any app (Java, React, Node, Angular, Vue, etc) or to use your own domain with any external file repo like S3.
- A complete suite of tools for backups. Moving a site or the whole server has never been so easy.
- Protect your site, folder or file with HTTP Authentication.
- Advanced support for cloning your sites for testing.
- Custom FastCGI Cache support for any dynamic site.
- Nginx redirection manager, native SMTP for emails and a lot more features.
- Datadog native integration for monitoring and analytics.
- Get an A+ grade on [Qualys (SSL Labs) Test](https://www.ssllabs.com/ssltest/).
- Log viewer in real time.

### Requirements
* Ubuntu 20.04 or 18.04

## Usage

```bash
# Install Webinoly and LEMP
wget -qO weby qrok.es/wy && sudo bash weby 3

# Create your first site.
sudo site example.com -wp

# Authentication
sudo httpauth -add
```

## Documentation
For complete documentation, please [visit our site](https://webinoly.com/en/).

Also, you can visit our [Community Support Forum](https://webinoly.com/support/)

## Contributing
Please open an issue first to discuss what you would like to change.

You don't need developer skills to help, visit our site to know [How to Contribute to Webinoly](https://webinoly.com/en/contribute/).

## Donations

[![PayPal Donations](https://cdn.qrokes.com/media/paypal-webinoly-donate.png)](https://www.paypal.me/qrokes)
[![GitHub Sponsors](https://cdn.qrokes.com/media/github-sponsors.png)](https://github.com/sponsors/QROkes)
[![Bitcoin Donations](https://cdn.qrokes.com/media/bitcoin-webinoly-donate.png)](https://www.blockchain.com/en/btc/address/1E3Ybo5UcvaAr1MoK4nBnMRFFY9aEMiku3)

If you like Webinoly, buy me a coffee or a beer to show support.

**_Your regular donations keep this project moving forward._**

## License
[GNU GPLv3](https://choosealicense.com/licenses/gpl-3.0/)