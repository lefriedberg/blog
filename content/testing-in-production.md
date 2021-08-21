+++
title = "Testing in Production"
date = 2021-08-21
draft = true
[extra]
show_date = true
+++

Don't bury the lede.

Background, I watched Charity Majors here
<iframe 
  width="560" 
  height="315" 
  src="https://www.youtube.com/embed/b2oota_FhGY" 
  title="YouTube video player" 
  frameborder="0" 
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" 
  allowfullscreen>
</iframe>


I started building HouseAccount from scratch this April.

The first thing I did was put a rails site on Heroku.

One of the first things we built was payment processing functionality (of course with Stripe) 

We ran in stripe test mode in both development and production. 

(Although we use https://github.com/Betterment/webvalve for development work)

This gave us confidence of our initial build and we were able to launch credit card processing without issue.

The second thing I worked on was 


Testing in Production can mean a number of things:
1. Feature flagging
1. Canary Releases
1. ...

This decision was a tradeoff: 

Alternatives:
1. 
