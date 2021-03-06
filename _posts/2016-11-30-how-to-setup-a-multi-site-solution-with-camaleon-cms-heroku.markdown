---
published: false
title: How to Setup a Multisite application with Camaleon CMS on Heroku  
layout: post
---


## The goal: Multiple Sites - & just one CMS-Installation to rule them all

Camaleon CMS is an awesome open source Rails-CMS which is released under the MIT License and supports multisite apps out of the box. A Camaleon multisite app allows you to create, run & manage multiple independent sites - each accessible via a dedicated web address - with just one CMS-Installation. This means you can utilize 1 Camaleon Rails App to serve `n` sites for `n` different customers (Corp A, Corp B... Corp N) - or to serve `n` sites for the same customer (e.g. one site for a corporation's regular website, 1 for their blog, 1 for their shop etc...).

### Quick n dirty: How does a Camaleon Multisite App work?

To make this possible, each site of a Camaleon multisite app reveives a unique site-key - a string that functions as humnan readable id (or "slug" in Rails-parlor) which is stored in the DB. This key is used to identify each site and to create a dedicated web address for each site, wherein the site key acts as virtual subdomain of your app's root domain:

Given your app is accessible via the url `www.[mydomain].com`, then a site with the site-key `fubar` would be accessible via the url `fubar.[yourdomain].com`, another site with the site-key `acme-corp` via `acme-corp.[yourdomain].com` and so forth... So if a Camelon Multisite app receives a  GET-request for `fubar.[yourdomain].com`, Camaleon extracts the virtual subdomain `fubar` from the request url and uses it as site-key / slug for a DB-lookup for this site. If the Site was found, its associated contents (DB-contents, view-templates, layouts, styles etc.) are retrieved and served to the client. The same goes for the request `acme-corp.[yourdomain].com`, which results in a DB-lookup for a site with the key `acme-corp` etc. All requests for the root domain `[mydomain].com` are routet to the default-/main-site that's created when you set up the CMS for the first time.  


## Why Multisite apps?

Before we dive into the details of setting up such an multisite app with Camaleon and heroku we have a quick glance at the reasons ...

Multisite apps came with numerous benefits:

- You avoid the overhead that's usually associated with the management and maintenance of n different Rails-apps / installs of Camaleon CMS; think of all the hustles that came with the frequent updates of all your apps' gems alone - a process that you would have to repeat for every app (be it a new version of Ruby, Rails, Camaleon CMS, Bootstrap, jQuery or whatever) - a Camaleon-Multisite app  ... plus the financial costs of having n different heroku-apps (assuming your're on a paid plan for each of your sites)
- quick and unified access to multiple client sites
- each site can use its own theme - or the can used shared themes
- reduced maintenance

- http://stackoverflow.com/questions/7003024/multi-tenant-rails-application-what-are-the-pros-and-cons-of-different-techniqu

- http://blog.elbowroomstudios.com/zero-to-multitenant-in-15-minutes-a-rails-walkthrough/
- https://www.elegantthemes.com/blog/resources/the-complete-guide-to-creating-a-wordpress-multisite-installation
- https://www.web-savvy-marketing.com/2012/02/wordpress-multisite/
- https://www.isaumya.com/advantages-of-wordpress-multisite-how-to-create-one/
- https://torquemag.io/2013/03/6-multisite/

# Setup a Camaleon multisite app

## Where we’re headed

We are going to create a Camaleon Multisite app that supports multiple independent customer-sites with dedicated urls and gives each customer the opportunity to edit their site's content via the CMS. For its ease of use (and the free plan ;-) ) we will utilize Heroku as application platform.

Before we deploy the app to Heroku we're going to perform the necessary steps to setup and test such an app locally. After successful local setup and deployment we will take care of the DNS- & Heroku-settings that are required to get a multisite app running on Heroku; this involves to map a custom domain to your Heroku app and enabling so called wildcard subdomains.

After that's accomplished we are going to fulfill another requirement: Each site shall be accessible not only via the default url (i.e. via the `virtualsubdomain.[yourdomain].com`-notation, e.g. `fubar.[yourdomain].com`) but also via a custom domain (e.g. `fubar-corp.com`). To accomplish this we will perform the necessary DNS- and Heroku-settings and implement a SiteHelper in your app that maps your clients custom domains to the correct Camaleon::Site-instance.      

Sidenote: Since there are excellent resources concerning installation and deployment of Camaleon Apps available (see below), this tutorial will set the focus on the multisite setup (i.e. DNS-, Heroku- & Appp-settings) rather than the installation and deployment details.

## Prerequisites

Camaleon CMS is distributed as Ruby gem ready to be installed in Rails-Applications. So this tutorial assumes that you are familiar with the Web Application Framework Ruby on Rails and the RubyGems-ecosystem - if that's not the case head over to [RailsGuides: Getting Started with Rails](http://guides.rubyonrails.org/getting_started.html) (or use another of the numerous excellent resources on this subject that are available online).

Camaleon CMS itself has the following requirements:

- Rails 4.1+
- MySQL 5+ or SQlite or PostgreSQL
- Ruby 1.9.3+
- Imagemagick

Make sure to check the [Camaleon CMS-Github-repo](https://github.com/owen2345/camaleon-cms) for possible updates on the requirements...

# 1. Setup a Camelon Multisite App Locally

## Create a new Rails-App

Open a terminal window and create a new rails app named `multisiteapp` (or whatever name you prefer):

**Terminal**

```sh  
~/projects$ rails new multisiteapp
```

After the new application was successfully created `cd` into the new app's dir and fire up the local web server:

**Terminal**
```sh  
~/projects$ cd multisiteapp
~/projects/multisiteapp$ rails server
```

Open a browser and visit http://localhost:3000/  - if you see the rails welcome page you are good...


##  Install Camaleon CMS & create the DB

Camaleon CMS can be easily installed as a Ruby gem, so the installation process is quite forward (check the [Camaleon CMS-Github-repo](https://github.com/owen2345/camaleon-cms) for details and updates): Simply add the gem to your new app's Gemfile (located in `[yourrailsapp]/Gemfile` - e.g. `multisiteapp/Gemfile`) and use the terminal to run `bundle install` from your apps root directory:


<u>Gemfile</u>
```ruby  
# Gemfile
source 'https://rubygems.org'

gem 'rails', '4.2.1'
# ...
# ...
gem "camaleon_cms"
# ...
# ...
```


<u>Terminal</u>

```sh  
~/multisiteapp$ bundle install
```

Now it's time to run the `camaleon_cms:install`-generator that's provided by the `camaleon_cms`-gem; among other essential CMS-specific stuff (like the new `app/apps/themes`-directory & a `lib/plugin_routes.rb`-file) it adds a certain config-file (`config/system.json`) to your Rails-app that's important for our multisite setup.  

<u>Terminal</u>

```sh  
~/multisiteapp$ rails generate camaleon_cms:install
```

On top of that the generator appends code to the end of your Gemfile to include all gems for Camaleon's plugins and themes - that's why you should run another `bundle install` here:

<u>Terminal</u>

```sh  
~/multisiteapp$ bundle install
```


Run `rake db:migrate` in your terminal to create the DB structure:      

<u>Terminal</u>

```sh  
~/multisiteapp$ rake db:migrates
```

## Edit config.json

Camaleon CMS offers advanced role-based User Management. You can create CMS-users of different roles & either share them across all sites of the CMS-installation - or you can assign CMS-users to specific sites only. The latter is meant for use cases like ours: Utilize Camaleon CMS to serve independent sites for different clients & allow those clients to use the CMS to update their sites (and their sites only!). To achieve this, you need to edit the previously created `config/system.json`-file accordingly. All you need to do here is to set the config-option `"users_share_sites"` to `false`:   

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

If your use case differs - say your goal is to create multiple sites for one customer only & allow them to edit each of those sites - simply go with the default setting here (`"users_share_sites": false`) and you are good. The same goes if no customer is involved in editing sites at all - i.e. you alone / your team edits all sites...


## Test your multisite app locally with `lvh.me`

Camaleon CMS has multisite support backed right in, so you don't need to take care of setting up subdomain based multi-tenancy in your Rails app. This means that the CMS takes care of mapping incoming requests to the correct site by extracting the virtual subdomain from the request url & looking up the DB for the site in question etc. Thanks to this and domains that enable us to use virtual subdomains locally - like `lvh.me` (the "successor" of `smackaho.st`) - setting up & testing multiple sites in your local development environment is a breeze. We just have to use http://lvh.me:3000/ instead http://localhost:3000.   

> `lvh.me` (lvh = local virtual host) is a domain which resolves to your local machine (i.e. `localhost` (`127.0.0.1`)) - and the same goes for all its potential subdomains (`*.lvh.me`). So everything from `lvh.me` itself to `[whateverfancysubdomainyoucanthinkof].lvh.me` points to `127.0.0.1`, which makes the usage of this domain the ideal solution for local subdomain testing - no need to mess around with etc/hosts anymore (the only "downside" is the need of a Internet-connection - but that should be a given nowadays).    

So fire up your local server by typing `rails s` in your terminal & navigate to http://lvh.me:3000/ with your browser:

<u>Terminal</u>
```sh  
~/projects/multisiteapp$ rails server
```

You should be redirected to http://lvh.me:3000/admin/installers and see the '1st time setup page', where you are asked to specify "Domain", "Name" and "Template" of your default-/main-site . The Domain-form-field is prepolutated with your localhost-alias  

**[SCREENSHOT]**






If you did setup everything fine, you should see <....>. <....>. Play a bit with Camaleon CMS to get aquainted with the CMS. Navigate to <....> to create a couple of sites and use sudomains as keys (slug) - let's say `fubar` and `barfu`. If you navigate to http://fubar.lvh.me:3000/ and http://barfu.lvh.me:3000/ you should see 2 different sites.    




# 2. Local: Push your new app to heroku

Now that you verified everything locally it's time to push your Rails app to heroku - again, see resources like if you get stuck  ...

# 3. Setup a custom domain for your Heroku app & enable wildcard subdomains

Setting up a custom domain for your heroku app allows ... need a custom domain not only to make your app accessible via default heroku domain name (like yourapp.herokuapp.com) , but to enable wildcard sudomains as well.  ...  
The following assumes that you have registered a domain already that you wish to use a custom doamin for your heroku app.   

## 3.1 DNS-Settings (Domain-Registrar / DNS Provider)   

We need to map your custom root domain and it's subdomains to your Heroku app (i.e. your app's default Heroku domain name which funtions as DNS Target - e. g. `yourapp.herokuapp.com`). The root (or naked) custom domain in our example is `yourdomain.com` - and the term subdomain covers everything that might be prepended to this root domain, be it `www` (like in `www.yourdomain.com`) or any other subdomain you can think of (in "wildcard notation": `*.yourdomain.com`). So all in all we have to create 3 DNS records: 1 for your custom root domain, 1 for its subdomain `www` and 1 "catch-all"/wildcard-record for all its other possible but non existing subdomains.

Depending on where you registered &/ host your domain or what service-plan you choose, the process of mapping your root domain and its subdomains to your Heroku app via DNS-records may differ - as well as the types of DNS-records you must use. Some DNS providers for example offer so called ALIAS-records for the mapping of root domains to a specific DNS-Target (like DNSimpel or PointDNS), others utilize ANAME- or CNAME-records for this task (see the Heroku Dev Center for an overiew: https://devcenter.heroku.com/articles/custom-domains#configuring-dns-for-root-domains or simply check with your provider). And to resolve www and wildcard subdomains to your Heroku-app, you need to create 2 CNAME-records, one of them a so called wildcard DNS record - a record type that's not supported by every Service. So in the worst case the creation of the desired mapping seems "impossible" on first sight, since not every Service / Provider permits you to perform the necessary DNS-Settings (see GoDaddy for instance) - but no worries: If that's the case a "little detour" should do the trick. Using DNSimple and GoDaddy as an example both scenarios are covered below :


### Sceanario 1: Your Domain Service / Registrar permits the necessary DNS-Settings


If you host your domain with an awesome service like DNSimple (which allows CNAME- incl. WildCard-records) the name really says it all, because setting up the necessary DNS-records is indeed pretty simple:

- Login to your DNSimple Account and navigate to your Dashboard
- from the dashboard open the record editor for your domain (by clicking the icon with the tooltip "Jump to your DNS records" next to your domain name)
- **Mapping root domain:** Map your custom root domain to your heroku-app: Hit the Add-button and select "ALIAS". In the appearing form just fill in the "Alias For"-form field with the your default heroku domain name (here in our example `yourapp.herokuapp.com`) and leave the "Name"-form field blank.   
- **Mapping `www`-subdomain domain:** ... A CNAME record simply aliases a subdomain to another host. .. Hit the "Add"-button and select "CNAME" from the appearing dropdown. To create a new CNAME record that maps `www.yourdomain.com` to `yourapp.herokuapp.com` simply enter `www` in the "Name"-form-field and the default heroku domain name  (here in our example `yourapp.herokuapp.com`) im the "Alias For"-form field; you can see the effects of your input in the little preview box on the right - if that box says something like "www.yourdomain.com resolves to youapp.herokuapp.com" everything's fine and you can hit "Create Record" (see illu).
- **Mapping wildcard subdomain:** Repeat the previous step to create another CNAME record that maps all potential / non existing subdomains of `yourdomain.com` (`*.yourdomain.com`) to your heroku default domain - but this time enter the wildcard subdomain notation `*` (asterisk) in the "Name"-form-field (the "Alias For"-field get's the same input than before: `yourapp.herokuapp.com`) - the preview box should now read something like "`*.yourdomain.com` resolves to `youapp.herokuapp.com`"


That's it. If you navigate to ... you should see now the following records:


Type | Name | Content/DNS-Target
------------ | ------------- | -------------
`ALIAS` | `[yourdomain].com` | `[yourapp].herokuapp.com`	 
`CNAME` | `www.[yourdomain].com` | `[yourapp].herokuapp.com`		  
`CNAME` | `*.[yourdomain].com` | `[yourapp].herokuapp.com`



ALIAS	ourschoolnet.de	example.com.herokudns.com
CNAME	www.ourschoolnet.de	example.com.herokudns.com


And if that wasn't easy enough, DNSimple offers a special "One click service"   .... that's handling this for you.




### Sceanrio 2: Your Domain Registrar doesn't permit the necessary DNS Settings (e.g. GoDaddy)

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





## 3.2 Add custom root & subdomain to your Heroku app

### Add Domains

Now all that's left to be done is to tell tell Heroku to route requests for your domain to your app - by adding your custom root domain and its subdomains (www + wildcard). Fire up a terminal and issue the following commands (alternatively you can log in to your Heroku-Account and add your domain via the web interface):  

**Add your cutom root domain**

`$ heroku domains:add yourdomain.com --app yourapp`

**Add www subdomain**

`$ heroku domains:add www.yourdomain.com --app yourapp`

**Add wildcard subdomain**

`$ heroku domains:add *.yourdomain.com  --app yourapp`

The first two commands ensure, that every request to youdomain.com or www.yourdomain.com is resolved to your heroku app - this equals your Camaleon app's main site. The last command makes sure you can use virtual subdomains with your Camaleon app and thus support multiple personalized client sites. But that's not the only reason why you should never forget to add widcard subdomains to your heroku app, once you created a corresponding wildcard DNS record via your DNS Provider: As the heroku dev center article stresses (https://devcenter.heroku.com/articles/custom-domains#add-a-wildcard-domain), "a malicious person could add" something like `baddomain.youdomain.com` "to their Heroku app and receive traffic intended for your application" if you forget to add the wildcard subdomain to your heroku app....  


### Check

Before you go on and try to create sites with Camaleon CMS on Heroku,  you should use the Terminal again to verify that everything is setup correctly:   

`$ heroku domains --app yourapp`

You should get something like this as response:

```ruby
Domain Name            DNS Target
────────────────────  ───────────────────
yourdomain.com         yourapp.herokuapp.com
www.yourdomain.com     yourapp.herokuapp.com
*.yourdomain.com       yourapp.herokuapp.com
```

And while you are on it, open a browser and navigate to `www.[yourdomain].com`: You see the start page of the Camaleon App Main Site? Good! Just to round things up navigate to `[yourdomain].com` as well & verify that you get the same result!

### Repair

If you messed up things you can remove domains from your Heroku app via the following commands:

**To remove a specific domain:**

`$ heroku domains:remove www.yourdomain.com  --app yourapp`

**To clear all domains at once:**

`$ heroku domains:clear --app yourapp`


# 4. Create multiple sites with your Heroku App


Now it's finally time to test your Multisite-Setup by creating a couple of Sites with your deployed Camaleon CMS App... let's suppose you want to set up Sites for your imaginary clients "ACME Corporation" and "FUBAR Inc.":

## Create a Site for your Client
- Navigate to `www.yourdomain.com/admin/login` and login into your app by entering your Admin-username and -password (should be "admin" + "admin" per default)
- Select **Settings > Sites** from the menu in the left sidebar
- You should see a list that contains just 1 entry: Your main-/default-site (recognizable by the value `true` in the "default"-column). To populate that list a bit, hit the **"+Add Site"**-button in the upper right.   
- In the appearing form, enter a "key" of your choosing into the **"Domain"**-form field that shall function as the subdomain - e.g. "acme" if you want this Site to be available for your client via `acme.yourdomain.com`. Also provide a **"Name"** for this Site/Account - e.g. "ACME Corporation" - plus a **"Description"** if you like.  Hit **"Submit"** if you're ready - you should be redirected to the Sites list again
- Find the new Site in the Sites list and select the associated button with the Eye-icon on it to open this Site's start page under http://acme.yourdomain.com/ in a new browser tab. If you see something like in figure x you're good.
- Repeat the above steps to create another Site - this time for your other pseudo-Client "FUBAR Inc." - and give that Site the subdomain-key "fubar" (or whatever you like).


You can be sure that you set everything up correctly if you can access
- the ACME Corporation's Site via `http://acme.[yourdomain].com/`,
- the FUBAR Inc.'s Site via `http://fubar.[yourdomain].com/`,
- and your main / default-site via `http://www.[yourdomain].com/` (and `http://[yourdomain].com/`).

If that's the case: Congrats - you got a working Multisite-solution with just a single CMS-Installation, that costs you nothing (apart from the fees for domain-registration & services).   


## Edit + Create User-Accounts for your Client

In order to enable your client to edit his own Site via Camaleon CMS, you can create one or more user accounts that are assigned and restricted to this Site only (thanks to our initial configuration of the `config/system.json`-file). Luckily Camaleon CMS already auto-created an Admin-Account when the new site was created - but again with the default login-credentials. We like to change those credentials into something less guessable and create another account without admin-rights that can be used by the client's staff to edit the copy text of the site - but not the layout or the user accounts etc.    

### Edit Default Admin-Account

- Visit http://acme.yourdomain.com/admin/login and enter the default  Admin-credentials ("admin" as login & password)
- Select **Users > All Users** from the menu in the left sidebar
- You find 2 default user accounts here: 1 Admin-Account+ 1 CLient-Account. To edit the Admin-Account select the associated edit-button  
- Select the button labeled "Change-Password" in the appearing Edit:Administrator-view and set a new password - e.g. "acmesecurehar" - hit "Process" and your're done.
- Logout and in again - but this time using the new Admin-Password... works? Great - let's create a new User Account, just for the fun of it

### Create new XY Account

- .... [check for details on user rolls]
- Test the new Account by logging out and in again - this time with the new XY-Accounts login-credentials of yourse. You should notice that the available menu-options in the left sidebar change depending on the account-type you used to login   

- Paranoia rocks: Just to ensure that everything works as expected visit http://fubar.yourdomain.com/admin/login as well and change the Admin-account's credential etc. ...  


# 5. Custom Domains for your clients ...

Chances are that your clients won't be too happy with their Site's default url (i.e. the `virtualsubdomain.[yourdomain].com`-notation) and ask you to map a custom domain of their choosing to their Site. So the question is: How to map a custom domain to a virtual subdomain of your Camaleon-CMS main-/default-Site on Heroku?  

Easy enough - let's say your imaginary client ACME Corporation asks you to register the domain `acme-corp.com` and map this to their virtual subdomain of your Camalaeon-Multisite-App `acme.[yourdomain].com/`.    


## 5.1 DNS-Records


Type | Name | Content/DNS-Target
------------ | ------------- | -------------
`URL` | `acme-corp.com` | `http://www.acme-corp.com`	 
`CNAME` | `www.acme-corp.com` | `acme.[yourdomain].com`		  



## 5.2 Add custom domain to your Heroku app


edmeda-verlag.de

Domain Name | DNS-Target
------------ | -------------
`acme-corp.com` | `[yourapp].herokuapp.com` 	 
`www.acme-corp.com` | `[yourapp].herokuapp.com`  	  


## 5.3 Add a SiteHelper to your app

Camaleon CMS offers a simple mechanism to map requests to your client's custom domain to the correct `CamaleonCms::Site`-instance: Simply create a new ruby file `camaleon_mapsites.rb` in `config/initializers` like the one below - just customize this file to your needs by replacing the custom domain - here `acme-corp.com` - with your client's actual domain and the slug - here `acme` - with the actual slug of your client's Camaleon-Site. Also make sure to check the [Camaleon-documentation]( http://camaleon.tuzitio.com/documentation/category/40756-uncategorized/how.html) for potential updates on this.



```ruby
# config/initializers/camaleon_mapsites.rb
#
# To better understand what's going on here you can use the following
# inside the helper method:
#
# puts "Cama::Site.all.pluck(:slug): '#{ Cama::Site.all.pluck(:slug)}'"
# puts "args[:site]: '#{args[:site]}'"
# puts "request.original_url: '#{request.original_url}'"
# puts "request.original_url.to_s.parse_domain: '#{request.original_url.to_s.parse_domain}'"

Rails.application.config.to_prepare do
  CamaleonCms::SiteHelper.module_eval do
    def cama_current_site_helper(args)    
      if !args[:site].present? && request.present?
        args[:site] =
          case request.original_url.to_s.parse_domain
          when 'acme-corp.com'
            CamaleonCms::Site.find_by_slug('acme').decorate             
          end
      end
    end
  end
end
```



This maps all requests to `acme-corp.com` to the `CamaleonCms::Site` with the slug `acme`.

That's it! Your client's Camaleon-Site is now accessible via the regular subdomain-notation (`acme.[yourdomain].de`) AND via the custom domain (`acme-corp.com`).

From here it's pretty easy to add more custom domains - e.g. for another client like FUBAR Inc. - or even multiple domain's per client-account / -slug (always provided you performed the necessary previous steps x - y for each domain):  


```ruby
Rails.application.config.to_prepare do
  CamaleonCms::SiteHelper.module_eval do
    def cama_current_site_helper(args)    
      if !args[:site].present? && request.present?
        args[:site] =
          case request.original_url.to_s.parse_domain
          when 'acme-corp.com', 'acme-corp.de'
            CamaleonCms::Site.find_by_slug('acme').decorate             
          when 'fubar-inc.com', 'fubar-inc.de'
            CamaleonCms::Site.find_by_slug('fubar').decorate             
          end
      end
    end
  end
end
```



## Sources



* http://matthewhutchinson.net/2011/1/10/configuring-subdomains-in-development-with-lvhme
* https://devcenter.heroku.com/articles/custom-domains#add-a-wildcard-domain
* https://en.wikipedia.org/wiki/Wildcard_DNS_record
* https://support.dnsimple.com/articles/delegating-dnsimple-hosted/
* http://camaleon.tuzitio.com/documentation/category/40756-uncategorized/how.html
* Another great resource to get you started (including CMS-usage instructions & Heroku deployment etc.) can be found here: [Up and Running with Camaleon CMS](https://www.sitepoint.com/up-and-running-with-camaleon-cms/).
