# Webinoly ![CI status](https://img.shields.io/badge/build-passing-brightgreen.svg)

Optimized LEMP Web Server.

Linux Ubuntu + Nginx + MySQL (MariaDB) + PHP is one of the most reliable and powerful configurations to host your websites.

With Webinoly you can set up your web server in just one step.

Webinoly provides a set of tools and commands that facilitate the web server administration.
- Unique commands to create, delete, disable sites.
- Free SSL certificates for your sites with Let’s Encrypt and automatic server configuration.
- HTTP/2 dramatically increase the speed of serving your content.
- PHP v7.2 and support for previous versions if necessary (5.6, 7.0 and 7.1).
- FastCgi Cache and Redis Object Cache for your WordPress sites.
- Get an A+ grade on [Qualys (SSL Labs) Test](https://www.ssllabs.com/ssltest/).
- Log viewer in real time.

### Requirements
* Ubuntu 16.04 or 18.04

## Usage

```bash
# Install Webinoly and LEMP
wget -qO weby qrok.es/wy && sudo bash weby 3

# Create your first site.
sudo site example.com -wp
```

## Documentation
For complete documentation, please [visit our site](https://webinoly.com/en/).

## Contributing
Please open an issue first to discuss what you would like to change.

Also, you can visit our [Community Support Forum](https://webinoly.com/support/)

## Donations
If you like Webinoly, buy me a coffee or a beer to show support.

[![PayPal Donations](https://www.paypalobjects.com/webstatic/en_US/i/btn/png/gold-rect-paypal-60px.png)](https://www.paypal.me/qrokes)

## License
[GNU GPLv3](https://choosealicense.com/licenses/gpl-3.0/)