---
published: false
title: How to setup a Multi-Site Solution with Camaleon CMS & Heroku  
layout: post
---

# How to setup a multi site solution with Camaleon CMS on Heroku
------

Setting up a multi-site solution with Camaleon CMS on Heroku is a great way to manage & run multiple client sites from a single CMS-installation. Since this process can be a bit tricky I outline the important steps & configuration details below - focussing on the Multi-Site setup rather than the installation and deployment details:  

## Create a new Rails App & Install Camaleon CMS 

Camaleon CMS is an awesome open source Rails-CMS (released under the MIT License) that can be easily installed as a Ruby gem. The installation process is quite forward - simply head over to the Camaleon CMS-Github-repo for details: https://github.com/owen2345/camaleon-cms#installation. Another great resource to get you started (including CMS-usage instructions & Heroku deployment etc.) can be found here: https://www.sitepoint.com/up-and-running-with-camaleon-cms/. But read on before you push your app to Heroku:

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


Camaleon CMS has MultiSite-Support backed right in, so you don't need to take care of setting up virtual subdomain-based multi-tenancy in your Rails app <>. Thanks to this and domains like `lvh.me` (the "successor" of `smackaho.st`) setting up & testing multiple sites locally is a breeze.

So once installation is complete you simply `cd` into your app's dir, fire up your local server by typing `rails s` in your terminal + navigate to http://lvh.me:3000/ with your browser. `lvh.me` (lvh = local virtual host) is a domain which resolves to your local machine (i.e. `localhost` (`127.0.0.1`)) - and the same goes for all its potential subdomains (`*.lvh.me`). So everything from `lvh.me` itself to `[whateverfancysubdomainyoucanthinkof].lvh.me` points to `127.0.0.1`, which makes the usage of this domain the ideal solution for local subdomain testing - no need to mess around with etc/hosts etc. anymore (the only "downside" is the need of a internet-connection - but that should be a given nowadays).     

If you did setup everything fine, you should see <....>. <....>. Play a bit with Camaleon CMS to get aquainted with the CMS. Navigate to <....> to create a couple of sites and use sudomains as keys (slug) - let's say `fubar` and `barfu`. If you navigate to http://fubar.lvh.me:3000/ and http://barfu.lvh.me:3000/ you should see 2 different sites.    


## Local: Push your new app to heroku 

Now that you verified everything locally it's time to push your Rails app to heroku ...

## Setup a custom domain for your Heroku app & enable wildcard subdomains 

Setting up a custom domain for your heroku app allows ... need a custom domain not only to make your app accessible via default heroku domain name (like yourapp.herokuapp.com) , but to enable wildcard sudomains as well.  ...  
The following assumes that you have registered a domain already that you wish to use a custom doamin for your heroku app.   

### a) DNS-Settings (Domain-Registrar / DNS Provider)   

We need to map your custom root domain and it's subdomains to your Heroku app (i.e. your app's default Heroku domain name which funtions as DNS Target - e. g. `yourapp.herokuapp.com`). The root (or naked) custom domain in our example is `yourdomain.com` - and the term subdomain covers everything that might be prepended to this root domain, be it `www` (like in `www.yourdomain.com`) or any other subdomain you can think of (in "wildcard notation": `*.yourdomain.com`). So all in all we have to create 3 DNS records: 1 for your custom root domain, 1 for its subdomain `www` and 1 "catch-all"/wildcard-record for all its other possible but non existing subdomains. 

Depending on where you registered &/ host your domain or what service-plan you choose, the process of mapping your root domain and its subdomains to your Heroku app via DNS-records may differ - as well as the types of DNS-records you must use. Some DNS providers for example offer so called ALIAS-records for the mapping of root domains to a specific DNS-Target (like DNSimpel or PointDNS), others utilize ANAME- or CNAME-records for this task (see the Heroku Dev Center for an overiew: https://devcenter.heroku.com/articles/custom-domains#configuring-dns-for-root-domains or simply check with your provider). And to resolve www and wildcard subdomains to your Heroku-app, you need to create 2 CNAME-records, one of them a so called wildcard DNS record - a record type that's not supported by every Service. So in the worst case the creation of the desired mapping seems "impossible" on first sight, since not every Service / Provider permits you to perform the necessary DNS-Settings (see GoDaddy for instance) - but no worries: If that's the case you must take a "little detour", that's all. Using DNSimple and GoDaddy as an example both scenarios are covered below : 


#### Sceanario a) Your Domain Service / Registrar permits CNAME-Records inkl. WildCard-records (e.g. DNSimple)
------

If you host your domain with an awesome service like DNSimple the name really says it all, because setting up the necessary DNS-records is indeed pretty simple:

- Login to your DNSimple Account and navigate to your Dashboard
- from the dashboard open the record editor for your domain (by clicking the icon with the tooltip "Jump to your DNS records" next to your domain name)
- **Mapping root domain:** Map your custom root domain to your heroku-app: Hit the Add-button and select "ALIAS". In the appearing form just fill in the "Alias For"-form field with the your default heroku domain name (here in our example `yourapp.herokuapp.com`) and leave the "Name"-form field blank.   
- **Mapping `www`-subdomain domain:** ... A CNAME record simply aliases a subdomain to another host. .. Hit the "Add"-button and select "CNAME" from the appearing dropdown. To create a new CNAME record that maps `www.yourdomain.com` to `yourapp.herokuapp.com` simply enter `www` in the "Name"-form-field and the default heroku domain name  (here in our example `yourapp.herokuapp.com`) im the "Alias For"-form field; you can see the effects of your input in the little preview box on the right - if that box says something like "www.yourdomain.com resolves to youapp.herokuapp.com" everything's fine and you can hit "Create Record" (see illu).
- **Mapping wildcard subdomain:** Repeat the previous step to create another CNAME record that maps all potential / non existing subdomains of `yourdomain.com` (`*.yourdomain.com`) to your heroku default domain - but this time enter the wildcard subdomain notation `*` (asterisk) in the "Name"-form-field (the "Alias For"-field get's the same input than before: `yourapp.herokuapp.com`) - the preview box should now read something like "`*.yourdomain.com` resolves to `youapp.herokuapp.com`"


That's it. If you navigate to ... you should see now the following records:

Type	Name	Content/Target
ALIAS	  yourdomain.com	      => yourapp.herokuapp.com
CNAME 	www.yourdomain.com 	 =>   yourapp.herokuapp.com 	
CNAME 	*.yourdomain.de       =>    yourapp.herokuapp.com 	




ALIAS	ourschoolnet.de	example.com.herokudns.com
CNAME	www.ourschoolnet.de	example.com.herokudns.com


And if that wasn't easy enough, DNSimple offers a special "One click service"   .... that's handling this for you.

 


#### Sceanrio b) Your Domain Registrar doesn't permit necessary DNS Settings like CNAME- incl. Widlcard-Records etc. (e.g. GoDaddy)

If you registered your domain with a registrar that doesn't allow you to add the necessary DNS records (like GoDaddy), you can either transfer your domain completely to a DNS Provider that does (like DNSimple) - or you can just transfer the hosting of your domain to such a provider. The latter means: Your registrar stays your registrar but your domain will no longer be hosted by this registrar - instead it will be hosted by another DNS Provider, who's domain services you want to use. This can be done by switching to the name servers of the Provider who shall host your domain. If you'd like to to transfer hosting from GoDaddy to DNSimple for example, follow these steps: 
    
**GoDaddy**    
- Log into your GoDaddy-Account and select your domain from the dashboard ...  
- select ....
- and change the names of the nameservers for your domain to the following:

```
    ns1.dnsimple.com 
    ns2.dnsimple.com
    ns3.dnsimple.com
    ns4.dnsimple.com
```

- Hit OK and you're done here (but it could take some time - up to 24 hours - until the changes propagate ....)    


**DNSimple**

- Log into your DNSimple-Account and select "+ Add domain" from your dashboard ... 
- You are presented with 3 options - select "Use domain services" to use DNSimple's domain services while keeping your domain at your current registrar
- Enter the name of your domain in the appearing form and hit the button "Use domains services"
- Now check the Domain resolution of your freshly added domain - select the domain from your dashboard and check the entries under Domain > Domain resoulution (you may have to hit the "Refresh"-button): If you see the previously (via GoDaddy) entered name servers (ns1.dnsimple.com, ns2.dnsimpel.com etc.) followed by the message "Your domain is resolving with DNSimple’s name servers." you're all set. If that's not the case: Relax & remember - it make take some time until the name server changes you performed earlier propagate...  
- Once the Domain is resolving correctly you can start using DNSimple's Domain Services and create the necessary DNS-records for your domain as described above under Scenario a) ...    

 


  
### b) Add custom root & subdomains to Heroku app

Now all that's left to be donmme is to tell tell Heroku to route requests for your domain to your app - by adding your custom root domain and its subdomains (www + wildcard). Fire up a terminal and issue the following commands (alternatively you can log in to your Heroku-Account and add your domain via the web interface):  

**Add your cutom root domain**
`$ heroku domains:add yourdomain.com --app yourapp`

**Add www subdomain**
`$ heroku domains:add www.yourdomain.com --app yourapp`

**Add wildcard subdomain** 
`$ heroku domains:add *.yourdomain.com  --app yourapp`

The first two commands ensure, that every request to youdomain.com or www.yourdomain.com is resolved to your heroku app - this equals your Camaleon app's main site. The last command makes sure you can use virtual subdomains with your Camaleon app and thus support multiple personalized client sites. But that's not the only reason you should never forget to add widcard subdomains to your heroku app, if you created a corresponding wildcard DNS record via your DNS Provider: As the heroku dev center article stresses (https://devcenter.heroku.com/articles/custom-domains#add-a-wildcard-domain), "a malicious person could add" something like `baddomain.youdomain.com` "to their Heroku app and receive traffic intended for your application" if you forget to add the wildcard subdomain to your heroku app....  


Test


First you should use the terminal again to verify that everything is setup correctly:   

`$ heroku domains --app yourapp`

You should get something like this as response:

`Domain Name`           `DNS Target`
────────────────────  ───────────────────
`youdomain.com`             `yourapp.herokuapp.com`
`www.youdomain.com`         `yourapp.herokuapp.com`
`*.youdomain.com`           `yourapp.herokuapp.com`


If you messed up things you can remove domains from your heroku app again via the following comamnds:

**to remove a specific domain**
`$ heroku domains:remove www.yourdomain.com  --app yourapp`

**or to clear all domains at once**
`$ heroku domains:clear --app yourapp`


done!



## Sources

- http://matthewhutchinson.net/2011/1/10/configuring-subdomains-in-development-with-lvhme
- https://devcenter.heroku.com/articles/custom-domains#add-a-wildcard-domain
- https://en.wikipedia.org/wiki/Wildcard_DNS_record
- https://support.dnsimple.com/articles/delegating-dnsimple-hosted/ 

