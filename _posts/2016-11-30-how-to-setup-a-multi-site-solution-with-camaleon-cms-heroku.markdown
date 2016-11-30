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

(1 main-/default-site like www.mydomain.de+ n further sites accessible via subdomains/keys like fu.mydomain.de, bar.mydomain.de etc... )
