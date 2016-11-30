---
published: false
title: How to setup a Multi-Site Solution with Camaleon CMS & Heroku  
layout: post
---

To setup a multi-site solution with Camaleon CMS on Heroku is a great way to manage & run multiple client sites from a single CMS-installation. Since this process can be a bit tricky I outline the important steps & configuration details below - focussing on the Multi-Site setup rather than the installation:  

1. Create a new Rails App, Install Camaleon CMS & push your new app to Heroku 

Camaleon CMS is an awesome open source CMS that can be easily installed as a ruby gem - the installation process is quite forward - simply head over to the repo for installation details: https://github.com/owen2345/camaleon-cms#installation. Another great resource to get you started (including Heroku deployment & CMS-usage instructions etc.) can be found here: https://www.sitepoint.com/up-and-running-with-camaleon-cms/

2. Edit config.json 

How do I create and assign users to specific sites only 

3. Local: Start server, setup Camaleon CMS + create a couple of Sites 

Camaleon CMS has MultiSite-Support backed in, so you don't need to take care of seting up multi-tenancy in your Rails app <>. Thanks to this and domains like `lvh.me` or `smackaho.st` setting up multiple sites locally is a breeze (though I think `smackaho.st` won't be accessible for ever because it won't be renewed by it's owner - we stick with `lvh.me` here)

So once installation is complete you simply `cd` into your app's dir, fire up your local server by typing `rails s`  in your terminal + navigate to http://lvh.me:3000/. `lvh.me` (lvh = local virtual host) is a domain which resolves to your local machine (i.e. `localhost` (`127.0.0.1`)) - the same goes for all its potential subdomains (`*.lvh.me` `[whateversubdomainyoucanthinkof].lvh.me`). Which makes it really great for local subdomain testing. 

 He set up a localhost wildcard, pointing *.smackaho.st at 127.0.0.1. Essentially this means that [anything].lvh.me will point to your local machine. 

 

It resolves itself and all its subdomains to 127.0.0.1. So you can easily test virtual subdomains as xxx.lvh.me without setting up your own DNS or touching /etc/hosts.




If you did setup everything fine, you should see <....>. <....>. Play a bit with Camaleon CMS to get aquainted with the CMS. Navigate to <....> to create a couple of sites and use sudomains as keys (slug) - e.g. fubar. http://fubar.lvh.me:3000/    

& use lvh.me for local subdomain testing

4. Push your new app to heroku 

Now it's time to push you Camaleon CMS app to heroku ...

5. Setup a custom domain for your heroku app & enable wildcard subdomains 

a) Domain-Registrar   

First of all we need to map your custom domain (f.i. ' www.mydomain.de' ) to your heroku app ('xy') and enable wildcard subdomains for this domain (' *.mydomain.de ' ). The latter will allow to    


(1 main-/default-site like www.mydomain.de+ n further sites accessible via subdomains/keys like fu.mydomain.de, bar.mydomain.de etc... )

- Create an DNSimple-Account & register your domain with DNSimple 
- Create 2 CNAME records for your freshly registered domain:

CNAME 	www.ourschoolnet.de 	 =>   ourschoolnet.herokuapp.com 	
CNAME 	*.ourschoolnet.de       =>    ourschoolnet.herokuapp.com 	


- if your domain is hosted elsewhere (godaddy for example): Log into godaddy and change the names of the nameservers for your domain to the following:

    ns1.dnsimple.com 
    ns2.dnsimple.com
    ns3.dnsimple.com
    ns4.dnsimple.com
  
b) Heroku
-  Add your domain to tell Heroku to route requests for your domain to your app:
$ heroku domains:add www.ourschoolnet.de --app ourschoolnet

- Set subdomains (wildcard to catch all subdomains)
$ heroku domains:add *.ourschoolnet.de  --app ourschoolnet

- verify: Check out domains
$ heroku domains --app ourschoolnet

- to remove a specific domain:
$ heroku domains:remove www.ourschoolnet.com --app school-net-upgrade-cedar

- or to clear all domains at once:
$ heroku domains:clear --app school-net-upgrade-cedar

done!


