---
published: false
title: How to setup a Multi-Site Solution with Camaleon CMS & Heroku  
layout: post
---

To setup a multi-site solution with Camaleon CMS on Heroku is a great way to manage & run multiple client sites from a single CMS-installation. Since this process can be a bit tricky I outline the important steps & configuration details below - focussing on the Multi-Site setup rather than the installation:  

1. Create a new Rails App, Install Camaleon CMS & push your new app to Heroku 

Camaleon CMS is an awesome open source CMS that can be easily installed as a ruby gem - the installation process is quite forward - simply head over to the repo for installation details: https://github.com/owen2345/camaleon-cms#installation. Another great resource to get you started (including Heroku deployment & CMS-usage instructions etc.) can be found here: https://www.sitepoint.com/up-and-running-with-camaleon-cms/

2. EDit config.json 

How do I create and assign users to specific sites only 


3. Setup a custom domain for your heroku app & enable wildcard subdomains 

a) Domain-Registrar   

(1 main-/default-site like www.mydomain.de+ n further sites accessible via subdomains/keys like fu.mydomain.de, bar.mydomain.de etc... )

- Create an DNSimple-Account & register your domain with DNSimple 
- Create 2 CNAME records for your freshly registered domain:

CNAME 	*.ourschoolnet.de       =>    ourschoolnet.herokuapp.com 	
CNAME 	www.ourschoolnet.de 	 =>   ourschoolnet.herokuapp.com 	

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


