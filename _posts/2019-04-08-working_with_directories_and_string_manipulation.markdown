---
layout: post
title:      "Working With Directories & String Manipulation"
date:       2019-04-08 07:23:02 +0000
permalink:  working_with_directories_and_string_manipulation
---


Hey so today, I spent a lot of time figuring out how to get a list of the files in a directory and also parsing a file name to proper format.

I think I spent almost an hour for this one line of code.
(Alright I can't seem to figure out how to post a pic of my code, so here's it written out :p...)

```
    @files = Dir.glob("#{@path}/*.mp3")
```

Here, I am setting instance variable @files equal to an array of files ending with ".mp3" located in my path.





```
    @files = Dir.glob("*.mp3")
```

This line of code will create an array of all files ending with ".mp3" in the CURRENT DIRECTORY.
By adding the string interpolation #{@path}, you are specifying which directory to look for the .mp3 files in.
@path is initialized during instantiation.
And then finally you're setting the @files instance variable equal to the list of filenames.
Below is the code that gets executed with path value.

```
    @files = Dir.glob("./spec/fixtures/mp3s*.mp3")
```

Finally you want to parse through each filename.
The filename you end up getting from the files method looks like

```
    ./spec/fixtures/mp3s/Action Bronson - Larry Csonka - indie.mp3
```

And you want it to just be

```
    Action Bronson - Larry Csonka - indie.mp3
```

```
    file[21..-1]
```
(file => long file name string)
I solved it by grabbing the string at indexes 21 through -1 (including).
I simply counted to the index where "Action Bronson ...." started and -1 will take you to last index of a string.

And finally you iterate through the list, with collect to collect the parsed filename for each long ugly file name.

```
    @files.collect do |file|
      file[21..-1]
    end
```

I hope someone found this helpful. Good luck peeps!



