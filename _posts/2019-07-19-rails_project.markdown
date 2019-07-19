---
layout: post
title:      "Rails Project"
date:       2019-07-19 04:41:31 +0000
permalink:  rails_project
---


Hi, what's up. I just got done with my rails project! 

I am pretty proud of finishing this project, as there were so many components to the project.
I basically created an app, FamilyTime, that allows users to create and join families. Families have boards where users can post and interact with each other.

I really enjoyed creating a program that was essentially a metaphor for what a family is. It was really interesting trying to come up with what I wanted my app to do, and also creating computer code that represents a dynamic in real life. Should I have a password for a family? Should a password for a family be optional?? Who can edit a family?? Can you leave a family? What different roles are there in a family?

It was also fun and challenging trying to figure out what models I would need, and how I wanted to relate the models. So in the beginning, I knew I wanted each family to have Mom(s), Dad(s), a Eldest child, and Youngest child, and I was having difficulty figuring out how to implement this. At first I thought I could achieve this through aliasing. For example, a family has_one mother whose class name is User. But what if I want a family to have more than 1 mom? I would have to have a join table for Moms, Dads, and so on. After a lot of scratchpadding, I realized the best way was to designate what 'title' a user had in a family in the join table. So in my join table, 'roles', my join table for a user and a family, I had the attribute 'title' which specified whether the user was a Mom or Dad or whatever.

After completing the Sinatra project and the Rails project, I think what I enjoy most about these projects, is designing the models and the underlying structure of the data for these apps. I mean the other parts are okay too, like creating the routes and the views, but a lot of it seems like busy work. It can get very repetitive validating the inputs for each of the models, creating basically the same views, or making sure each of the routes are protected. But, designing the models, I found the most engaging (trying to design my models, getting all the models to work the way I needed, creating the methods I need to be used in the controller or the views).

I generally knew that I wanted to work in the backside of development, but I'm glad I'm kind of seeing hands on which part of the app I enjoy most working in. 

And I'm done with 3/5 big projects!! :). See u again at javascript!!!! Pce.

