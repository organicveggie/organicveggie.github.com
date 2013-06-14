---
layout: post
title: "Custom SSL Certificates with Chef 11 Server"
date: 2013-06-14 14:02
comments: true
categories: 
 - Chef
 - Nginx
 - SSL
---
While setting up [Chef](http://www.opscode.com/chef/) 11 Server on a [CentOS](http://www.centos.org/) 6.4 box, I realized I wanted that it defaults to using self-signed SSL certificates. Since we have existing wildcard certificates through GoDaddy, I wanted to switch over to those. It look a little bit of digging and some help from IRC #chef to get things working.
<!-- more -->
First, you need to your certificate and key. For completeness, you want to combine your certificate with the certificate chain provided by your signing authority. In the case of GoDaddy, they gave me two files:

- `example.com.crt`
- `gd_bundle.crt`

To create a combined bundle:

```bash
cat example.com.crt gd_bundle.crt > example.com.pem
```

In addition, you need to make sure that your keyfile does *not* have a passphrase associated with it, otherwise Nginx will refuse to start up. To remove the passphrase from an existing key file:

```bash
openssl rsa -in example.com.key -out example.com.nopassphrase.key
```

You need to copy the combined bundle and your key file to your server and place them in a secure location. On a typical CentOS installation, you want to dump the files in `/etc/pki/tls/private`.

Next, you need to tell Chef to use the custom certificates. Create a file named `/etc/chef-server/chef-server.rb`, if doesn't exist already. Add the following lines to it:

```ruby
nginx['ssl_certificate']        = "/etc/pki/tls/private/example.com.pem"
nginx['ssl_certificate_key']    = "/etc/pki/tls/private/example.com.nopassphrase.key"
```

Now, reconfigure Chef server:

```bash
chef-server-ctl reconfigure
```

That will reconfigure Nginx with the new settings (which are stored in `/var/opt/chef-server/nginx/etc`) and automatically restart Nginx. You can verify that it's using the correct certificate by running:

```bash
openssl s_client -connect localhost:443
```