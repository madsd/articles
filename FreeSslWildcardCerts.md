# Generate SSL certificates using LetsEncrypt
I often do PoC, MVP, hackathons or whatever they are called these days. In many scenarios, a real SSL certificate is required and makes the experience more realistic. LetsEncrypt has been around for a while offering free production grade certificates _for free_ with the only limitation, that the expiration is 90 days, which forces some sort of automation, and not just a calendar entry in Outlook with "Remember to order new Cert in 20NN+2", that is very easy to dismiss.
Until recently, you could only get specfic domain certs, but in March 2018, they opened the v2 endpoint, which also supports wildcard certs.

## Prerequisites
The only real prerequisite to getting a certificate is, that you have to own the domain. There are several ways to prove this, but in the scenario below, you prove it by adding a TXT record to the domain, which means you have to control the DNS of the domain. There are many low cost services, where you can obtain a domain and many of them also offer free DNS hosting.
You can also purchase your domain through Azure Portal with [App Service Domain](https://docs.microsoft.com/da-dk/azure/app-service/custom-dns-web-site-buydomains-web-app), you can host the DNS using [Azure DNS](https://docs.microsoft.com/en-us/azure/dns/dns-overview) (even if your domain was not bought through Azure), and you can purchase certificates using [App Service Certificates](https://docs.microsoft.com/en-us/azure/app-service/web-sites-purchase-ssl-web-site). These can be exported and used for all purposes, but are rather expensive in "use and throw away" scenarios.

## Lets Encrypt
This is why the focus of this article is to get you started on obtaining a standard or wildcard certificate from LetsEncrypt using a Windows 10 machine. You could alternatively use a small Linux box running in Azure. The examples are using Ubuntu 16.04.

## Installing Windows Subsystem for Linux
To get access to all the Linux goodness on Windows, you need to enable WSL. You do that under "Turn Windows features on or off" (hit Windows key and start typing)

![Windows Subsystem for Linux](https://github.com/madsd/articles/blob/master/images/windowsFeatures.png)

Next you need to install the subsystem. There are more options available such as SUSE, Debian and Ubuntu. I will use Ubuntu, but you can use your distribution of choice, but would need to adapt the scripts. Just open Microsoft Store and search for Ubuntu.

## Install CertBot
[CertBot](https://certbot.eff.org/) is the most commonly used client for LetsEncrypt. It has a wealth of features, and we will only use a subset. To install CertBot we need to open Bash on Ubuntu and run the following commands:
```
sudo add-apt-repository -y ppa:certbot/certbot
sudo apt-get update
sudo apt-get -y install certbot
```
## Initiate the certificate request
When this is done, we are ready to obtain the certificate. Before you run the next process, make sure that you can add a TXT record to your DNS Zone. When you run the certbot certonly command, it will generate a challenge and then pause. You must then add a TXT record to your DNS, and once that is done, you continue the script, and certbot will verify the TXT record to confirm your ownership of the domain.
Note: A lesson learned is that you should verify that the TXT record is "live" using e.g. nslookup before you continue the script. You should also choose a very low TTL for the record. Default is normally 24 hours, and if something goes wrong, you have to wait 24 hours for the next attempt. I usually use 60 seconds.

So you run the following commands, replacing the domain name with your domain name. The first time you run the command, you have to type in an email as well, and if you want to support the project.
Once you see the challenge, pause and switch to your DNS management tool and enter the TXT record.
```
domain=reddoglabs.com
sudo certbot certonly --manual -d *.$domain --agree-tos --no-bootstrap --manual-public-ip-logging-ok --preferred-challenges dns-01 --server https://acme-v02.api.letsencrypt.org/directory
```
![Cert challenge](https://github.com/madsd/articles/blob/master/images/txtRecordChallenge.png)

Enter the TXT record:
![Add TXT Record](https://github.com/madsd/articles/blob/master/images/addTxtRecord.png)

Check the TXT record using a different cmd window:
```
nslookup -q=TXT _acme-challenge.reddoglabs.com 8.8.8.8
```
![Check TXT Record](https://github.com/madsd/articles/blob/master/images/verifyTxtRecord.png)

And after you have verified the response, continue the certbot script:
![Verify challenge](https://github.com/madsd/articles/blob/master/images/verifyChallenge.png)

## Convert and export certificate
You know have a set of certificate files in .pem format. For some of the services I commonly use (App Services, Application Gateway, API Management) you need a .pfx file and .cer file. You can generate these using openssl. While generating the pfx file, you need to provide a password to protect it, as this contains your private key:
```
sudo openssl pkcs12 -export -out "star_$domain.pfx" -inkey "/etc/letsencrypt/live/$domain/privkey.pem" -in "/etc/letsencrypt/live/$domain/cert.pem" -certfile "/etc/letsencrypt/live/$domain/chain.pem"

sudo openssl x509 -outform der -in "/etc/letsencrypt/live/$domain/cert.pem" -out "star_$domain.cer"
```
![Export certs](https://github.com/madsd/articles/blob/master/images/exportCerts.png)

Finally you want to copy this to you "local machine". The Linux Subsystem has its own file system hidden in the Windows file system, but you have access to the Windows file system from /mnt/c|d|... In my case, I create a folder under C:\cert\reddoglabs.com, but you can change that in your script.
```
sudo mkdir --parents /mnt/c/cert/$domain
sudo cp "star_$domain.pfx" /mnt/c/cert/$domain
sudo cp "star_$domain.cer" /mnt/c/cert/$domain
```
![Copy certs](https://github.com/madsd/articles/blob/master/images/copyCerts.png)

### Enjoy!

![Explore certs](https://github.com/madsd/articles/blob/master/images/explorerCertificates.png)