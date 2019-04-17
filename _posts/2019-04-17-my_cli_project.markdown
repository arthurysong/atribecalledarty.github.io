---
layout: post
title:      "My CLI Project"
date:       2019-04-17 05:10:30 +0000
permalink:  my_cli_project
---


Hello. How's it going.

For my CLI project I decided it was going to be about UFC. I've been trying to get more into the UFC recently, thanks to Joe Rogan's podcast, and so I decided it would be a good idea to make a program about the different divisions in the UFC and the title holders.

![](https://sportshub.cbsistatic.com/i/r/2018/10/10/5f5eb99f-399d-4c2f-b61e-ee4181387933/thumbnail/770x433/a9da16edb644a8fca0115f273d00730c/poirier.jpg)

So basically, I wanted my program to scrape data from this website http://www.espn.com/mma/story//id/14947566/current-all-ufc-champions, and display the champions (and other info) for each division in the UFC.

First thing, I wanted to do was create the CLI interface. Like Avi does in his tutorial of creating his Daily_Deal project, I created the CLI class (everything that was involved with interfacing with the user) in a very simple way.
I hardcoded everything that I wanted the program to output, like 

Heavyweight (Up to 265 pounds)<br>
Daniel Cormier<br>
Won title: July 7, 2018<br>
Outcome: R1 TKO oover Stipe Miocic (UFC 226)<br>
Defenses: 1

All of this info (that I later wanted to scrape from the website) I hardcoded into my CLI class just to lay out a foundation of what I wanted my program to do. I really like the way Avi does this because it really allows you to break down what you want your program to do in multiple steps.

Then once, I got my CLI class to work, I worked on creating the classes I would need for my program. At first I thought I would only need a Champion class, but if I wanted my program to more accurately model real life, I realized it would be better to have a Division class as well. And I think this is a better way (your code should more accurately model real life) because if I wanted my program to do more complicated things later on, it would make more sense.

So I created a Division class and Champion class, and Champion classes will belong to Division classes. For instance, I will have a Champion class with name "Daniel Cormier" that will belong to Division class "Heavyweight". So now once I have my classes working, I hardcoded all the info on the website into my class objects.

For example, I would have an instance of the Division class which had name = "Heavy weight", weight = "Up to 265 pounds", champion = instance of Champion class with the following attributes: name = "Daniel Cormier", title_won = "July 7, 2018", Outcome = etc etc etc...

So now I want my CLI to refer to these instances instead of just outputting data that I wrote into the code that way all I have to do now is scrape the data into these instances and my CLI will be outputting data from the website, not just whatever I write into my program!

So then, on to scraping data from the website. 
Scraping was pretty straightforward, except trying set the attributes for my Champion instances. This was tricky because a lot of the info on the website didn't have classes and most of them simply had paragraph tags. So I had to parse through a lot of other text to get the data that I wanted. I found the #split very helpful in dividing up a lot of text by keywords like in my case "Won title:". And then using more #split and #chomp I was able to narrow down the text to the exact texts that I wanted.

Another thing that was kind of tricky was the fact that one of the divisions did not have a title holder at the time. TJ Dillashaw who was up until recently the Men's bantamweight division title holder was suspended? for testing positive for EPO's. I think THE BEST WAY to solve this problem was with some if statement to control for an vacant division, so that if the website were to change, my program would be able to account for change. But as this was only an article with the current division title holders (which will be updated in the future by other articles), I thought it would be okay to hard code into my program, a blank champion for that division.

So for index 6 for my class variable @@all for my division class, the champion had name = "no champion", etc etc.

Finally, the last things I did was refactor all my code so it was more efficient and more organized. For example, initially I had a case statement for each choice in my program.

1. Heavyweight (Up to 265 pounds)
2. Light heavyweight (205)
3. Middleweight (185)
4. Welterweight (170)
5. Lightweight (155)
6. Men's featherweight (145)
7. Men's bantamweight (135)
8. Men's flyweight (125)
9. Women's featherweight (145)
10. Women's bantamweight (135)
11. Women's flyweight (125)
12. Strawweight (115)

So for each choice, I had a bunch of #puts statements to output the info of division and champion attributes.
After refactoring I was able to have one block of code that simply takes the users input and displays that division plus its corresponding champion, which I was able to access using the @@all class variable in division.

And finally finally, I added in a couple of "\n" to make my program outputs look more readable.

I think the biggest thing I learned from this project was being able to break down a project into simpler easier to code steps. Like first creating the simple CLI, then creating the classes, then scraping, then refactoring. I think the most important thing is just writing code whether it's the most efficient or even the correct way because at first it can be overwhelming to create a project. But yeah, just write code and try to get 1 thing working, and then just correct and make things more efficient as you go.

And BOOM, I'm done with my first FlatIron project!

![](https://i.ytimg.com/vi/wCc7xP_Pzsc/maxresdefault.jpg)

