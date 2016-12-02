---
published: false
title: How to setup a Multi-Site Solution with Camaleon CMS & Heroku  
layout: post
---

# How to setup a multi site solution with Camaleon CMS on Heroku
------

Setting up a multi-site solution with Camaleon CMS on Heroku is a great way to manage & run multiple client sites from a single CMS-installation. Since this process can be a bit tricky I outline the important steps & configuration details below - focussing on the Multi-Site setup rather than the installation and deployment details:  

## Create a new Rails App & Install Camaleon CMS 

Camaleon CMS is an awesome open source Rails-CMS (released under the MIT License) that can be easily installed as a Ruby gem. The installation process is quite forward - simply head over to the Camaleon CMS-Github-repo for details: https://github.com/owen2345/camaleon-cms#installation. Another great resource to get you started (including CMS-usage instructions & Heroku deployment etc.) can be found here: https://www.sitepoint.com/up-and-running-with-camaleon-cms/

Edit config.json
----

Camaleon CMS offers advanced role-based User Management. You can create CMS-users of different roles () & either share them across all Sites of the CMS-installation - or you can assign CMS-users to specific Sites only. The latter is meant for usecases like ours: Utilize Camaleon CMS to serve sites for different clients & allow those clients to use the CMS to update their sites. To achieve this, you need to edit the `config/system.json`-file (??that was created when you run the `rails generate camaleon_cms:install` generator during the CMS-installation process??) accordingly. All you need to do here is to set the config-option `"users_share_sites"` to `false`:    

```javascript
// config/system.json
// Note: after any change in this file, you need to restart your server to apply changes.

{
  "share_sessions": false, // (boolean) share user sessions between subdomains of base_domain (only relevant if users_share_sites = true)
  "default_user_role": "client", // default user role for all new users
  "users_share_sites": false, //(boolean) true: permit to share users between sites, false: All users are assigned to a unique site. (Only change before installation)
  "db_prefix": "cama_", // prefix name for database tables
  "relative_url_root": "", // URL prefix, for example to get http://localhost:3000/blog/, this should be "blog"
  "hooks": {}
}
```

## Local: Start server, setup Camaleon CMS + create a couple of Sites 


Camaleon CMS has MultiSite-Support backed right in, so you don't need to take care of setting up virtual subdomain-based multi-tenancy in your Rails app <>. Thanks to this and domains like `lvh.me` or `smackaho.st` setting up & testing multiple sites locally is a breeze (though I think `smackaho.st` won't be accessible for ever because it won't be renewed by it's owner - we stick with `lvh.me` here).

So once installation is complete you simply `cd` into your app's dir, fire up your local server by typing `rails s` in your terminal + navigate to http://lvh.me:3000/ with your browser. `lvh.me` (lvh = local virtual host) is a domain which resolves to your local machine (i.e. `localhost` (`127.0.0.1`)) - and the same goes for all its potential subdomains (`*.lvh.me`). So everything from `lvh.me` itself to `[whateverfancysubdomainyoucanthinkof].lvh.me` points to `127.0.0.1`, which makes the usage of this domain the ideal solution for local subdomain testing - no need to mess around with etc/hosts etc. anymore (the only "downside" is the need of a internet-connection - but that should be a given nowadays).     

If you did setup everything fine, you should see <....>. <....>. Play a bit with Camaleon CMS to get aquainted with the CMS. Navigate to <....> to create a couple of sites and use sudomains as keys (slug) - let's say `fubar` and `barfu`. If you navigate to http://fubar.lvh.me:3000/ and http://barfu.lvh.me:3000/ you should see 2 different sites.    


## Local: Push your new app to heroku 

Now that you verified everything locally it's time to push your Rails app to heroku ...

## Setup a custom domain for your Heroku app & enable wildcard subdomains 


a) DNS-Settings / Domain-Registrar   

First of all we need to map your custom domain (let's call it `www.yourdomain.com` ) to your Heroku app (`yourapp.herokuapp.com`) and enable wildcard subdomains for this domain (`*.yourdomain.com`) which must resolve to your Heroku-app as well. For this to work you need to change the DNS-settings of `yourdomain.com` by creating so called "CNAME" records that point to your Heroku-app. A CNAME record simply aliases a subdomain (but not the root domain itself!) to another host. The root (or naked) domain in ou example is yourdomain.com - and the term subdomain covers everything that might be prepended to this root domain, be it `www` (`www.yourdomain.com`) or any other subdomain you can think of (`*.yourdomain.com`). The process of mapping your domain's subdomains to your heroku app via CNAME records may differ depending on where you registered your domain, since not every Service permits you to perform the necessary DNS-Setttings (in this case you must take a little detour) . 

Sceanrio a) Domain Service / Registrar permits CNAME-Records
------

If you registered your domain with an awesome service like DNSimple the name says it all, because setting up the necessary CNAME records is pretty easy:

- Login to your DNSimple Account and navigate to your Dashboard
- from the dashboard open the record editor for your domain (by clicking the icon with the tooltoip "Jump to your DNS records" next to your domain name)   
- Hit the "Add"-button and select "CNAME" from the appearing dropdown. To create a new CNAME record that maps `www.yourdomain.com` to `yourapp.herokuapp.com` simply enter `www` in the "Name"-form-field and the name of your Heroku-app (here in our example `yourapp.herokuapp.com`) im the "Alias For"-form field; you can see the effects of your input in the little preview box on the right - if that box says "www.yourdomain.com resolves to youapp.herokuapp.com" everything's fine and you can hit "Create Record" (see illu).
- Repeat the last step to create another CNAME record for all potential subdomains of `yourdomain.com` (`*.yourdomain.de`)

(1 main-/default-site like www.mydomain.de+ n further sites accessible via subdomains/keys like fu.mydomain.de, bar.mydomain.de etc... )

for instance allwos you to make all the necessary settings 
Our name says it all: 
because  If you use an awsome service like DNSimple your are all set 

If you registered your domain with a registra taht doesn't alalow 

- Create an DNSimple-Account & register your domain with DNSimple 
- Create 2 CNAME records for your freshly registered domain:

Type	Name	Content
CNAME 	www.yourdomain.com 	 =>   yourapp.herokuapp.com 	
CNAME 	*.yourdomain.de       =>    yourapp.herokuapp.com 	


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



## Sources

- http://matthewhutchinson.net/2011/1/10/configuring-subdomains-in-development-with-lvhme
- http://tbaggery.com/2010/03/04/smack-a-ho-st.html
