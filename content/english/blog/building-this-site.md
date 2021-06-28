---
title: "Building This Site"
date: 2021-05-02T20:37:30-04:00
draft: false
---

# Overview

This blog posts serves as a welcome and a write up on the decisions and experience of setting up this site.

As you may have read on the main site, we at Bradypus take pleasure in taking the path of least resistance when it comes to technology success.  The setup of this site is no different.  This post details the decisions made when creating this site and how I got it all up
and running.

# Goals

I knew several things at the start.

* The vast majority of our content would be static.
* Hosting could be done for under $5 per month.
* Content creation should be easy.

# Background

From other sites that I had deployed recently, I knew that you could launch an AWS hosted Wordpress site for under $5 per month (not
counting the cost of the domain name).  I had deployed Wordpress using Lightsail, setup media content hosting in S3, fronted it all using
CloudFront, and managed the DNS with Route 53.  A basic setup could be created and configured using just Terraform (and a few manual step
on the Lightsail host).

However, there were several things I experienced in this setup that we less than ideal.  Though the overall setup works and is very cheap, it is worth noting that Lightsail doesn't allow for IAM Profile attachment.  So automating an SSL certificate renewal process wasn't
ideal.  Also, I used almost none of the advanced features of Wordpress for that site.  So it was a lot of effort wasted.

That experience led me to believe that for most use cases, a site that has almost nothing but static content can be done just as well in an
even simpler manner.

# Static Site Review

I started looking at several static site generators that I had heard about recently.  Namely, Gatsby, Hugo, and Jekyll.  All three appear to be modern and well supported site generation tools.  A quick search on the internet led to the usual posts claiming superiority of each.  Several things became apparent after reading dozens of articles:

* Hugo is fast to generate the entire site (not critical after the initial setup, but still nice).
* Local installation size of Hugo is small(-ish) since it doesn't involve NPM.
* Hugo is well supported with plugins, themes, and can support any JavaScript if you really want it.

In the end though, I think it came down to my general distaste for the JavaScript ecosystem that led me to choose Hugo.

DISCLAIMER: I am not a frontend developer.  I generally dispise all frontend ecosystems.  NodeJS and npm are terrible in particular.

# The Site

I'll skip over the AWS portion for now (and leave that for a future post).  Here, I'm going to detail the steps I took to get this site going in Hugo locally.  Most of this would be covered by the [Hugo Quickstart](https://gohugo.io/getting-started/quick-start/).

## First Install Hugo

I'm on Linux and Hugo has a Brew package.  So pretty simple:

```
$ brew install hugo
```

Next, create your new site locally.  Do this from the parent directory of where you want the new project/folder to be created:

```
$ hugo new site <sitename>
```

Quick, change into your new project/folder.  Then, you'll need to add a theme.  This involves searching the [Hugo Themes Site](https://themes.gohugo.io/).  Eventually, when you find one, you'll need to clone the theme to a local Git Submodule in you project.  The URL for the Git clone can be found on the Theme's page.  It is the 'Download' button's reference location:

```
$ cd <sitename>
$ git init
$ git submodule add <theme Git URL> themes/<themename>
```

Now, here's where its not so obvious what you need to do next.  I found that the theme that I'm using has an 'exampleSite' folder.  I basically started copying folders and files from there.  I really had no idea what I was doing and it took me a while to realize how much of it I needed (almost all of it) and that some of their folders had language subfolders.  Apparenly that was important too even though I'll only be supporting English.

Finally, startup the Hugo server:

```
$ hugo server -D
```

At this point, a local Hugo server should startup, generate all your pages, and tell you how to get to it at [http://localhost:1313/](http://localhost:1313/).  The site will stay up and running and hot reload as you save files in the project directory.

# AWS Setup

The goal here is cheap and quick.  I had originally thought about managing the site myself with an Instance or Lightsail again.  But I had heard about AWS Amplify a while back and wanted to give it a try.  Amplify is basically all about static site hosting.  Behind the scenes it is probably just S3 + CloudFront.  That and a little Route 53 is all you need to host a site.

However, Amplify adds to that with predesigned 'build packs' for different source types and site rendering engines.  And of course, a Hugo site hosted in GitHub is one of them.  So this should be super easy.

## GitHub

This one is super simple.  Create a new repository and push your code.  Done!

## Amplify

Also pretty straight forward.  Log into the Console and navigate to Amplify.  

Click the 'Get Started' button.  You'll be presented with the choice of 'Frontend + Backend' or 'Just Frontend'.  Now in my case, I'm just hosting a frontend.  So I chose that as the option.  

Next, I chose GitHub as my source location.   It then walked me through a standard OAuth based process so that I could grant AWS access to the respository.  Honestly, it wanted a lot more permissions than I thought it would need.  But I'm not going to fight it.  The entire GitHub organization is just for this site.  Once the OAuth is setup, Amplify will present you with an option to select the repository.

Once you select your repository, AWS must inspect the code base.  In my case, it came up with a 'build plan' for Hugo.  I just accepted it.

And off to the races!  The site started to build and was fully deployed to an Amplify generated domain name in less than two minutes!

## Site Verification

At this point, I navigated to the site link that Amplify gave me.  Immediately I noticed that half the site didn't render correctly.  Some quick inspections of the browser console and I noticed that half the URLs were to a placeholder 'baseURL' that I had put in the Hugo config.toml file.  I updated that parameter to the Amplify URL and pushed again.  In a few moments, the page was updated and working alright.

Which leads us to the next part.

## Domain Registration + DNS Setup

Head over to Route53.  Register a domain name.  Get an email.  Click the confirmation link.  Wait.  The AWS Invoice will show up in your email before the registration is complete.  This took about 20 minutes.  When you are used to everything in life being so quick, the process of getting a new domain seems to take forever (or about 20 minutes).  I should have made the domain registrations one of my first steps.

This is the first time I've registered a new domain in about four years.  I forgot how long it can take.

Once the registration completes go back to the Route53 dashboard and check to make sure there is a new Hosted Zone created for the domain.  Check the records.  Specifically, compare the NS records in the Hosted Zone with the Name Servers listed in the Registered Domain.  They should be the same. 

Head back over to the Amplify console.  In my case, I had a checklist of things in Amplify.  Step two (after the site roll out) was to setup the domain name and SSL.  I just clicked through and chose the domain that I had just added.  This kicks off another process by which AWS will generate the Route53 records in your new Hosted Zone.

You can then wait again.

Mine made it through the SSL Creation and made it past the domain ownership check and told me that they were waiting on verification of ownership.  And that it could take up to eight hours.

I thought it would be another email asking me to verify the ownership.  But eventually it just went through.  So I don't really know what the wait was for.

# Conclusion

In the end, it took me several nights to create the basic content of this site in Hugo.  Should it?  I don't know.  I'm sure someone
with more experience in developing websites in general could have knocked this out in a much shorter time.

As usual, interacting with AWS was easy.  I had a new organization (and new GMail account) in under 15 minutes.  The connection from my
machine to the new account in Terraform took about five minutes to configure.  And the creation of the Amplify site and Route 53 was done
in under an ???.