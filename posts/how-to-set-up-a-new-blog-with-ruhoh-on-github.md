---
title: How to set up a new blog with ruhoh on github
date: '2013-02-10'
description:
categories: 'tutorial'
tags: [tutorial]
---

Since it took me quite some effort to get this blog running, I give a short summary of the steps I went through. Note, however, that I am really a beginner at this, so I only point you to those links that helped me. Don't even bother to ask me when something doesn't work. Not that I wouldn't want to help...I just couldn't. Oh, and just to be clear: this guide assumes that you are using Linux.

Get ruhoh running
-------------------------

[ruhoh](http://ruhoh.com/) is a static blogging platform. Why would you want to use that in the first place? Good question! The point is that I wanted to publish some of my replications of finance papers and for that I wanted to be able to write both formulas and code snippets. Turns out that this isn't so easy with normal blogging platforms. For instance, I signed up for Bloggers from Google and couldn't get it to work. I wasn't able to get `mathjax` to run.

So after some research, I found [Jekyll/Bootstrap](http://jekyllbootstrap.com/), which seemed to do what I wanted. However, on that webpage the maintainer wrote that he now focused his efforts on `ruhoh` instead, so I thought that I might as well just do that. Note that `ruhoh` is quite new though, so `Jekyll` is definitely better documented.

Anyways, to get `ruhoh` running, just follow the [installer guide](https://github.com/ruhoh/blog/tree/2.0.alpha#readme). It looks pretty straightforward, but I had to deal with the issue that there are a lot of dependencies. I can't all memorize them now, but my workflow was something like this:

1. Write command from installer guide into terminal.
2. Get message on why this failed (*xyz* is missing).
3. Googling message.
4. Dealing with it (mostly just installing *xyz*, which often also needed *abc*, etc.)

So after that was done, you should have a folder somewhere in your  folder structure with all the subfolder copied from Jade's github page. To check if it worked out, fire up a terminal, go to the folder you copied everything into, and type:

```
$ bundle exec rackup -p 9292
```

This starts a web server that hosts your blog here: [http://localhost:9292](http://localhost:9292). So basically, you can check out your blog in the browser. Now you can edit all the posts and play around.

Install mathjax
-------------------------

You have to install a `mathjax` widget, which sounds more complicated than it is. Your folder structure should have one folder named *widgets*. In this folder, add another folder named *mathjax*, and a subfolder named *layouts* and copy [this file](https://github.com/ramnathv/ramnathv.ruhoh.com/blob/master/widgets/mathjax/layouts/mathjax.html) into the *layouts* folder. Actually, if you check out that file online, you also see the folder structure it has to be into.

Finally, you have to put \{\{\{ mathjax \}\}\} in your *default.html* file in the `/themes/.../layouts` subfolder. 

Now you should be able to write equations in LaTex. In-text math should be surrounded with $ signs, equations in a separate line with double dollar signs. To check if it worked, copy this [sample file](https://gist.github.com/plusjade/2699636) into your pages folder and check it out in your localhost session. 

The only issue I had was that I couldn't use underscores, which are quite important in LaTex. However, this [SO post](http://stackoverflow.com/questions/10438937/is-there-a-markdown-parser-supported-on-jekyll-that-plays-nicely-with-mathjax) solved this problem.

Deploying your blog on github
-------------------------

If you are happy with how your blog looks locally, it's time to move it to the world wide web. You could find a good web hoster or you could decide to just use Github pages: it's for free and your already use github (otherwise you couldn't have installed `ruhoh`). Here's just my personal rant though: the Github pages documentation is really bad. Seriously, I wouldn't call it a documentation at all. It's a bunch of FAQs that don't give you any big picture at all. To be fair, it's probably written for the average github user who is already quite familiar with this service. Sadly, I'm not...

However, Ramnath Vaidyanathan rescued me again (he also posted the Mathjax-widget code) and posted instructions on how to deploy your blog on Github pages. However, the page is down now so I just post the code here.

First, you have to compile your blog:

    $ cd /PATH_OF_YOUR_BLOG
    $ ruhoh compile
    $ cd compiled
    $ cd REPO (in my case: cd replicating)
    $ git init .
    $ git add .
    $ git commit -m "update blog"
    $ git push https://github.com/$USER/$REPO.git master:gh-pages --force
    $ rm -rf .git
    $ cd ../..
    
So you basically deploy the compiled version of your blog on `github` and then get rid of all traces. As Ramnath writes, it is not a good idea to put generated files under version control.

Before I run those commands though, I pushed all my blog files to github as well. I'm not sure if this is necessary, but I did it anyways. To do so, you have to run the following commands in the working directory where your blog is saved:

    $ git init
    $ git add .
    $ git commit -m "First commit to blog."
    $ git remote add origin git@github.com:USER/REPO.git
    $ git push -u origin master
  
Obviously, you have to replace USER with your github username and REPO with the repository that you have created before on the github.com website. If you do this, you should see all the files pushed to github.com and it should look similar to my [repository](https://github.com/christophj).

Google analytics
-------------------------

Setting up Google Analytics is straightforward. Just set up Analytics on Google's [webpage](google.com/analytics). You should then get a tracking number that you have to enter into the *config.yml* file in the **widget** folder of your blog. 

Now it is important to push those updated to `github` again and wait. It took a day or so for me until Google started tracking my blog.

Next steps
-------------------------

The next steps I'm playing are the following:

* Change the look of the code snippets. I don't really like the gray, it looks too pale.

If I figure those things out, I update this post here.

