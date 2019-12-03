---
layout: post
title:      "Programming is a Metaphor"
date:       2019-12-03 02:31:19 +0000
permalink:  programming_is_a_metaphor
---


Hey, I'm about to submit my JS project.

I created an app that will help users understand cities better in terms of wealth and value.

I've been looking into investing in real estate and wanted an app that will allow me to store and compare information about cities. Information like what was the median household property value? What was the GDP per capita in that area.. What are the schools like in the area.. What has the growth been like .. in property, population and income.

One thing that I found challenging about this app was all the different functionality that was required with one 'event'.

![](https://imgur.com/a/tJMLvse)

For example, here I have a lot of information loaded: the city Tucson, a zipcode in that city, and also the homes sold recently in that city. What would I want to happen when I select '-' for the city select???

Do I want to just hide everything OR do I want to clear all the elements with info and then hide everything?

I decided the best thing to do is to both hide the elements and actually clear all the information like the zipcode information, the city information and etc. I came to this decision because doing so accurately represents what the intent of the selected action is.. Like although just simply hiding all the elements would be effectively doing the same thing, I thought, metaphorically, it made more sense that the DOM, although hidden, reflected what the page was showing..

I thought this was important distinction because coding like Avi says is like creating metaphors. We want our objects to be a metaphor, a representation of what the object is in real life. In that same way, our actions, in this case selecting '-' and the consequences of that action should properly reflect the intent. In this case selecting a blank city should clear all the info in the page.

That was a fun and definitely challenging project that incorporated a lot of different things that I learned so far. I'm proud of myself and ready to move on to the next module!
