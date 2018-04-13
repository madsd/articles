# Generate SSL certificates using LetsEncrypt
I often do PoC, MVP, hackathons or whatever they are called these days. In many scenarios, a real SSL certificate is required and makes the experience more realistic. LetsEncrypt has been around for a while offering free production grade certificates _for free_ with the only limitation, that the expiration is 90 days, which forces some sort of automation, and not just a calendar entry in Outlook with "Remember to order new Cert in 20NN+2", that is very easy to dismiss.
Until recently, you could only get specfic domain certs, but in March 2018, they opened the v2 endpoint, which also supports wildcard certs.

# Prerequisites
The only real prerequisite to getting a certificate is, that you have to own the domain. There are several ways to prove this, but in the scenario below, you prove it by adding a TXT record to the domain, which means you have to control the DNS of the domain. There are many low cost services, where you can obtain a domain and many of them also offer free DNS hosting.
You can also purchase your domain through Azure Portal with [App Service Domain](https://docs.microsoft.com/da-dk/azure/app-service/custom-dns-web-site-buydomains-web-app), you can host the DNS using [Azure DNS](https://docs.microsoft.com/en-us/azure/dns/dns-overview) (even if your domain was not bought through Azure), and you can purchase certificates using [App Service Certificates](https://docs.microsoft.com/en-us/azure/app-service/web-sites-purchase-ssl-web-site). These can be exported and used for all purposes, but are rather expensive in "use and throw away" scenarios.

# Lets Encrypt
This is why the focus of this article is to get you started on obtaining a standard or wildcard certificate from LetsEncrypt using a Windows 10 machine. You could alternatively use a small Linux box running in Azure. The examples are using Ubuntu 16.04.

# Installing Windows Subsystem for Linux
To get access to all the Linux goodness on Windows, you need to enable WSL. You do that under "Turn Windows features on or off" (hit Windows key and start typing)
![Windows Subsystem for Linux](https://github.com/madsd/articles/blob/master/images/windowsFeatures.png)
