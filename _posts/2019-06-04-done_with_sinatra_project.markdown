---
layout: post
title:      "Done with Sinatra Project!"
date:       2019-06-04 05:32:04 +0000
permalink:  done_with_sinatra_project
---


Hello,
I just finished my Sinatra Project, Lixer!
It's basically a web app that allows users to post their favorite videos. Users can like other people's videos and edit or destroy their own video posts.

![](https://i.ytimg.com/vi/JpBPkYBW9lo/hqdefault.jpg)
<br>
how to create billie eilish's "bad guy"

My favorite part or actually the part that was the most challenging was creating the associations between my User model and VideoPost model. It was challenging because I wanted a VideoPost to belong to a creator but also have many likers, BUT a liker and a creator are both Users!!!

First, I needed a join table => UserVideoPosts, which will have a user_id and a video_post_id because a User has many liked video posts, and conversely a VideoPost has many likers (A has many to has many relationship).

Now, creating the associations within the models.

```
class UserVideoPost < ActiveRecord::Base
  belongs_to :liker,
             :class_name => "User",
             :foreign_key => :user_id
  belongs_to :liked_video_post,
             :class_name => "VideoPost",
             :foreign_key => :video_post_id
end
```

So first I created the associations for the join table. It belongs to a liker and also a liked_video_post, BUT since the keys and models are not named using "like" => (liker, liker_id, liked_video_post, etc), I need to specify the class name and also the foreign key in the join table.

I think it's very important that I use "liker" and "liked_video_post" here, because there are two relationships going on between a User and a VideoPost. The creator and created_video_posts relationship and the likers and liked_video_posts relationship. Creating this new namespace(??) makes both these relationships possible all the while, making the code more readable. 

```
class VideoPost < ActiveRecord::Base
  belongs_to :creator, 
             :class_name => "User", 
             :foreign_key => :creator_id
             
  has_many :user_video_posts
  has_many :likers, 
           :through => :user_video_posts,
           :source => :liker,
           :inverse_of => :liked_video_posts
end
```

This is my VideoPost model.
First I have the creator relationship, which is a belongs_to creator using class name User, and also the foreign key.

Then I have the has many to has many relationship between likers and liked_video_posts.
First, I specify the join table. (has_many :user_video_posts) (This is straightforward, since the join table has foreign keys matching the models)
Then, I need to have many likers through the join table (has_many :likers, :through => :user_video_posts). 
To do this I must specify the **source**, which is telling ActiveRecord which association to use **SINCE my join table DOES NOT have "liker_id." ** 

FINALLY, I need to specify the inverse_of so that ActiveRecord knows to save the association that is the inverse.
So if I DIDN'T have the inverse_of, if I have 

```
a = User.create(username: Bob)
p = VideoPost.create(title: Video of Funny Thing)
a.liked_video_posts << p
a.save
```

**a.liked_video_posts would include p. BUT p.likers would NOT contain a.** This is because ActiveRecord does not know that liker and liked_video_posts have an inverse relationship where a user of a liked_video_post is also that videopost's liker.

--This part, creating aliases for associations and also linking 2 aliases together with the inverse_of property, was very interesting and challenging figuring out on my own.

```
class User < ActiveRecord::Base
  has_secure_password
  
  has_many :created_video_posts,
           :class_name => "VideoPost",
           :foreign_key => :creator_id
           
  has_many :user_video_posts
  has_many :liked_video_posts, 
           :through => :user_video_posts,
           :source => :liked_video_post,
           :inverse_of => :likers
end
```

And finally, this is my User model, which has the same thing basically.

Overall, I really enjoyed making my Sinatra Project, Lixer, which is webapp that I could see (maybe within a view more iterations) people actually using. It was fun creating the logic, designing the routes, and also designing a very basic view with little styling.

Now its time for Rails!!!!

![](https://media.istockphoto.com/photos/celebrationparty-with-colorful-confettistreamers-on-white-picture-id871727100?k=6&m=871727100&s=612x612&w=0&h=1_LY5CuSD27fTFlVPKYWT_5lkc--_dh9nskDFkMq8lY=)
