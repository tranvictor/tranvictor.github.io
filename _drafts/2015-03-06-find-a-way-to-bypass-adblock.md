---
layout: post
title: "Plan for an internal hack night at ClickLion"
date: 2015-03-06
tags: hacknight adblock
---

Topic: Find a way to bypass AdBlock

How do we do it?

  * We should have a simple code which will be installed on publishers' website
  * We should create a simple website (adopting from trendsread might be a good idea)
  * We should create a simple web service to respond to the script with ad html
  * Install the code to that website to see if the AdBlock could block it
  * Change variables on the script to see if AdBlock can block it or not
  * Change ad html tags, classes, ids, structure to see if AdBlock can block it or not
  * Write a report on all the things we just harvested from the research
  * If we could find a way to bypass the AdBlock, we should find a way to automate the process so it could be applied to ClickLion

Who will do it?

  We will have totally 4 guys to do this hacknight (Vu, Nam, Thiem, Canh)

  * Vu will use a trendsread page as publisher website
  * Nam and Thiem will create a script to load the ad from web service
  * Canh will create a simple Ruby on Rails web service to response ad html
  * Everyone will try to change the script and ad html to bypass AdBlock
  * Vu will write a report on the hacknight result
