# 生成 SSL 密钥 - Let's Encrypt

## 准备工作

* 熟悉命令行操作。
* 熟悉使用SSL证书来增强网络安全性是个加分项
* 了解命令行文本编辑器（本示例使用 _vi_）
* An already running web server open to the world on port 80 (http)
* 一台可通过80（HTTP）端口访问的服务器
* 熟悉 _SSH_ （安全外壳协议）并且可以使用 _SSH_ 访问服务器

# 简介

目前最流行的保护网站安全的方法是使用Let's Encrypt SSL证书，当然，这是免费的。

这些都是真正的证书，而非自签名或伪造证书，所以它非常适合作为低预算的安全解决方案。本文档将会引导您在运行Rocky Linux的Web服务器上安装并使用Let's Encrypt证书。

## 前提

* 所有命令均假定为您是root用户或者有权限使用 _sudo_ 来获取root权限。

## 安装 

首先，请使用 _ssh_ 登入你的服务器。如果服务器的DNS名称为www.myhost.com，则您可以使用一下命令登入：

`ssh -l root www.myhost.com`

如果你需要先以非root用户登入服务器，请使用你的用户名：

`ssh -l username www.myhost.com`

然后：

`sudo -s`

在这种情况下，你将会需要 _sudo_ 权限来获取系统的root权限

Let's Encrypt使用了一个叫做 _certbot_ 的包，而我们需要通过snap安装这个软件包。为了在Rocky Linux上安装 _snapd_，你将会需要安装EPEL仓库：

`dnf install epel-release`

依照您的系统，除了 _snapd_ 之外，您可能还需要安装 _fuse_ 和 _squashfuse_。我们还需要确保 _mod\_ssl_ 已经被安装。如果要安装它们，请使用：

`dnf install snapd fuse squashfuse mod_ssl`

_snapd_ 在安装时将会需要与其一起安装大量的依赖，因此，请在安装提示处回答yes。

当 _snapd_ 和所有依赖都被安装完成之后，使用以下命令启动 _snapd_ 服务：

`systemctl enable --now snapd.socket`

_certbot_ 需要传统 _snapd_ 支持，所以我们需要通过软链接启用它：

`ln -s /var/lib/snapd/snap /snap`

在进行其他操作之前，我们需要确保所有snap包都是最新版本。执行：

`snap install core; snap refresh core`

如果检测到了任何更新，那这些更新将会被安装。

如果你领先了一步，已经通过RPM安装了 _certbot_ （顺便说一下，这个版本无法正常工作），请确保您使用了一下命令移除它：

`dnf remove certbot`

最后，我们可以安装 _certbot_ 了，执行以下命令：

`snap install --classic certbot`

此命令将会安装 _certbot_。最后一步是将 _certbot_ 存放在一个Rocky Linux可以快速找到的路径下。这可以通过使用另一个软链接完成：

`ln -s /snap/bin/certbot /usr/bin/certbot`

## Getting The Let's Encrypt Certificate

There are two ways to retrieve your Let's Encrypt certificate, either using the command to modify the http configuration file for you, or to just retrieve the certificate. If you are using the procedure for a multi-site setup suggested for one or more sites in the procedure [Apache Web Server Multi-Site Setup](apache-sites-enabled.md), then you will only want to retrieve your certificate. 

We are assuming that you **are** using this procedure so we will only retrieve the certificate. If you are running a standalone web server using the default configuration, you can retrieve the certificate and modify the configuration file in one step using `certbot --apache`. 

To retrieve the certificate only, use this command:

`certbot certonly --apache`

This will generate a set of prompts that you will need to answer. The first is to give an email address for important information:

```
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator apache, Installer apache
Enter email address (used for urgent renewal and security notices)
 (Enter 'c' to cancel): yourusername@youremaildomain.com
```

The next asks you to read and accept the terms of the subscriber agreement. Once you have read the agreement answer 'Y' to continue:

```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server. Do you agree?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: 
```

The next is a request to share your email with the Electronic Frontier Foundation. Answer 'Y' or 'N' as is your preference:

```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing, once your first certificate is successfully issued, to
share your email address with the Electronic Frontier Foundation, a founding
partner of the Let's Encrypt project and the non-profit organization that
develops Certbot? We'd like to send you email about our work encrypting the web,
EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: 
```

The next prompt asks you which domain you want the certificate for. It should display a domain in the listing based on your running web server. If so, enter the number next to the domain that you are getting the certificate for. In this case there is only one option ('1'):

```
Which names would you like to activate HTTPS for?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: yourdomain.com
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel): 
```

If all goes well, you should receive the following message:

```
Requesting a certificate for yourdomain.com
Performing the following challenges:
http-01 challenge for yourdomain.com
Waiting for verification...
Cleaning up challenges
Subscribe to the EFF mailing list (email: yourusername@youremaildomain.com).

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/yourdomain.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/yourdomain.com/privkey.pem
   Your certificate will expire on 2021-07-01. To obtain a new or
   tweaked version of this certificate in the future, simply run
   certbot again. To non-interactively renew *all* of your
   certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

## The Site Configuration - https

Applying the configuration file to our site is slightly different than if we were using a purchased SSL certificate from another provider. 

The certificate and chain file are included in a single PEM (Privacy Enhanced Mail) file. This is a common format for all certificate files now, so even though it has "Mail" in the reference, it is just a type of certificate file. To illustrate the configuration file, we will show it in it's entirety and then describe what is happening:

```
<VirtualHost *:80>
        ServerName www.yourdomain.com 
        ServerAdmin username@rockylinux.org
        Redirect / https://www.yourdomain.com/
</VirtualHost>
<Virtual Host *:443>
        ServerName www.yourdomain.com 
        ServerAdmin username@rockylinux.org
        DocumentRoot /var/www/sub-domains/com.yourdomain.www/html
        DirectoryIndex index.php index.htm index.html
        Alias /icons/ /var/www/icons/
        # ScriptAlias /cgi-bin/ /var/www/sub-domains/com.yourdomain.www/cgi-bin/

	CustomLog "/var/log/httpd/com.yourdomain.www-access_log" combined
	ErrorLog  "/var/log/httpd/com.yourdomain.www-error_log"

        SSLEngine on
        SSLProtocol all -SSLv2 -SSLv3 -TLSv1
        SSLHonorCipherOrder on
        SSLCipherSuite EECDH+ECDSA+AESGCM:EECDH+aRSA+AESGCM:EECDH+ECDSA+SHA384:EECDH+ECDSA+SHA256:EECDH+aRSA+SHA384
:EECDH+aRSA+SHA256:EECDH+aRSA+RC4:EECDH:EDH+aRSA:RC4:!aNULL:!eNULL:!LOW:!3DES:!MD5:!EXP:!PSK:!SRP:!DSS

        SSLCertificateFile /etc/letsencrypt/live/yourdomain.com/fullchain.pem
        SSLCertificateKeyFile /etc/letsencrypt/live/yourdomain.com/privkey.pem
        SSLCertificateChainFile /etc/letsencrypt/live/yourdomain.com/fullchain.pem

        <Directory /var/www/sub-domains/com.yourdomain.www/html>
                Options -ExecCGI -Indexes
                AllowOverride None

                Order deny,allow
                Deny from all
                Allow from all

                Satisfy all
        </Directory>
</VirtualHost>
```

Here's what's happening above. You may want to review the [Apache Web Server Multi-Site Setup](apache-sites-enabled.md) to see the differences in the application of an SSL purchased from another provider and the Let's Encrypt certificate:

* Even though port 80 (standard http) is listening, we are redirecting all traffic to port 443 (https)
* SSLEngine on - simply says to use SSL
* SSLProtocol all -SSLv2 -SSLv3 -TLSv1 - says to use all available protocols, except those that have been found to have vulnerabilities. You should research periodically which protocols are currently acceptable for use.
* SSLHonorCipherOrder on - this deals with the next line that regarding the cipher suites, and says to deal with them in the order that they are given. This is another area where you should review the cipher suites that you want to include periodically
* SSLCertificateFile - this is the PEM file, that contains the site certificate **AND** the intermediate certificate. We still need the 'SSLCertificateChainFile' line in our configuration, but it will simply specify the same PEM file again.
* SSLCertificateKeyFile - the PEM file for the private key, generated with the _certbot_ request.
* SSLCertificateChainFile - the certificate from your certificate provider, often called the intermediate certificate, in this case exactly like the 'SSLCertificateFile' location above.

Once you have made all of your changes, simply restart _httpd_ and if it starts test your site to make sure you now have a valid certificate file showing. If so, you are ready to move on to the next step.

## Automating Let's Encrypt Certificate Renewal

The beauty of installing _certbot_ is that the Let's Encrypt certificate will be automatically renewed. There is no need to create a process to do this. We do need to test the renewal with:

`certbot renew --dry-run`

When you run this command, you'll get a nice output showing the renewal process:

```
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/yourdomain.com.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Cert not due for renewal, but simulating renewal for dry run
Plugins selected: Authenticator apache, Installer apache
Account registered.
Simulating renewal of an existing certificate for yourdomain.com
Performing the following challenges:
http-01 challenge for yourdomain.com
Waiting for verification...
Cleaning up challenges

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
new certificate deployed with reload of apache server; fullchain is
/etc/letsencrypt/live/yourdomain.com/fullchain.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations, all simulated renewals succeeded: 
  /etc/letsencrypt/live/yourdomain.com/fullchain.pem (success)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

The [_certbot_ documentation](https//certbot.eff.org/lets-encrypt/centosrhel8-apache.html) tells you in their step number 8, that the automatic renewal process could be in a couple of different spots, depending on your system. For a Rocky Linux install, you are going to find the process by using:

`systemctl list-timers`

Which gives you a list of processes, one of which will be for _certbot_:

```
Sat 2021-04-03 07:12:00 UTC  14h left   n/a                          n/a          snap.certbot.renew.timer     snap.certbot.renew.service
```

# Conclusions

Let's Encrypt SSL certificates are yet another option for securing your web site with an SSL. Once installed, the system provides automatic renewal of certificates and will encrypt traffic to your web site. 

It should be noted that Let's Encrypt certificates are used for standard DV (Domain Validation) certificates. They cannot be used for OV (Organization Validation) or EV (Extended Validation) certificates. 
