---
date: '2013-03-19'
title: Using WRDS on a Linux terminal
categories: ['finance']
tags: [finance]
---

I'm currently a visiting graduate researcher at UCLA and one of the nice perks here is access to [WRDS](http://wrds-web.wharton.upenn.edu/wrds/). As they write on their site, they are the leading data research platform in the world. And why do they lead? Well, in my opinion one reason is the very easy access to great data sets. For instance, as soon as I had my account I checked out their awesome documentation sites and noticed that I can easily run scripts on their servers (apparently, this only applies to faculty staff and PhD students, so not everyone can do this). 

If you want to do that as well, just start a terminal session in `Linux` (don't ask me how this works on `Windows` or on a `Mac`, but you find great documentation on WRDS for that as well) and type:

```
$ ssh username@wrds.wharton.upenn.edu
```

(You also find a very good introduction [here](http://wrds-web.wharton.upenn.edu/wrds/support/Accessing%20and%20Manipulating%20the%20Data/_002Unix%20Access/Introduction%20to%20the%20WRDS%20Unix%20System.cfm). Note that this and the following links only help if you have an account for WRDS. However, if you don't, this post isn't relevant for you anyways...)


After that, you are asked to enter your password and BOOM!, you are already on their server! After that, you can start by typing in some [commands](http://wrds-web.wharton.upenn.edu/wrds/support/Accessing%20and%20Manipulating%20the%20Data/_002Unix%20Access/UNIX%20Quick%20Reference%20Sheet.cfm).

To exit, just type

```
$ exit
```

or 

```
$ logout
```

Now, it is quite easy to run a `SAS` file on their server. First, you need to know how to write a file...basically, you can start an editor -- such as `vi`, `Emacs`, or `pico` --  in the terminal. I use `pico` here, because it is a rather simple editor and since I copy/paste the text anyways, this is more than enough.

Let's start with a simple example (taken from [here](http://www.ats.ucla.edu/stat/sas/modules/intsas.htm)) to see that you can actually run `SAS` on the WRDS server. Type the following into the terminal:

```
$ pico first_script
```

This opens an editor window in the terminal. In this window, copy/paste the following `SAS` code:

```
DATA auto ;
  INPUT make $ price mpg rep78 weight length foreign ;
DATALINES;
AMC     4099 22  3     2930   186    0
AMC     4749 17  3     3350   173    0
AMC     3799 22  3     2640   168    0
Audi    9690 17  5     2830   189    1
Audi    6295 23  3     2070   174    1
BMW     9735 25  4     2650   177    1
Buick   4816 20  3     3250   196    0
Buick   7827 15  4     4080   222    0
Buick   5788 18  3     3670   218    0
Buick   4453 26  3     2230   170    0
Buick   5189 20  3     3280   200    0
Buick  10372 16  3     3880   207    0
Buick   4082 19  3     3400   200    0
Cad.   11385 14  3     4330   221    0
Cad.   14500 14  2     3900   204    0
Cad.   15906 21  3     4290   204    0
Chev.   3299 29  3     2110   163    0
Chev.   5705 16  4     3690   212    0
Chev.   4504 22  3     3180   193    0
Chev.   5104 22  2     3220   200    0
Chev.   3667 24  2     2750   179    0
Chev.   3955 19  3     3430   197    0
Datsun  6229 23  4     2370   170    1
Datsun  4589 35  5     2020   165    1
Datsun  5079 24  4     2280   170    1
Datsun  8129 21  4     2750   184    1
;
RUN;

PROC PRINT DATA=auto(obs=10);
RUN;
```

This creates a data set named auto with the columns *price*, *mpg*, *rep78*, *weight*, *length*, and *foreign* and some observations. Finally, it prints the first 10 observations.

After you have copied this file in the `pico` editor, press `CTRL + X`. Now you should see a dialogue asking you if you want to save the file. Press Yes. Now, to run the programm, just type

```
$ sas first_script
```

into the terminal. Now, a file named `first_script.lst` is created, which you can check out by typing

```
$ more first_script.lst
```

into the terminal. You just run your first script on a WRDS terminal.

Next, let's try a far more ambitious example: Let's run the CRSP/IBES matching programm (called `iclink`) on the terminal. You can find this file [here](http://wrds-web.wharton.upenn.edu/wrds/support/Data/_010Linking%20Databases/_000iclink.sas). A quick disclaimer here: I am a complete `SAS` noob, so everything I am writing with regard to `SAS` could be completely wrong. For me, the only thing that matters here is that I get the desired output and this is solely based on my feeling whether or not a copied script run correctly.

Unfortunately, when I copied that `iclink.sas` into the `pico` editor as before and let it run on the WRDS terminal, it failed with an error. Checking out the log `iclink.log` that is created, I saw that all started with the following warning: "WARNING: Apparent symbolic reference lt not resolved."

It turned out that something in those lines were off:

```
if (not ((ldate&lt;namedt) or (fdate&gt;nameenddt))) and name_dist < 30 then SCORE = 0;
    else if (not ((ldate&lt;namedt) or (fdate&gt;nameenddt))) then score = 1;
  else if name_dist &lt; 30 then SCORE = 2; 
```

Doing some googling, I found [this](http://support.sas.com/documentation/cdl/en/lrdict/64316/HTML/default/viewer.htm#a001157104.htm). So my explanation (again, I have no clue!) is that the script wrapped a `&` and a `;` around every `lt` (less then) and `gt` (greater then), something it shouldn't do (or maybe not anymore in `SAS 9`). Replacing `&lt;` with `<` and `&gt;` with `>` made it work for me and I was left with a `iclink.sas7bdat`file in my folder, which was the matching table. 

Final task? Downloading that file to my computer. This can be done as follows. Just log out of your `ssh` session by typing in `exit` into the terminal and type the following into your terminal:

```
$ sftp username@wrds.wharton.upenn.edu
```

`sftp` stands for **S**ecure **F**ile **T**ransfer **P**rotocol and allows you to transfer files between a *host* and a *client*. In this case, the former is the WRDS server and the latter is your computer. So after you established a connection by entering your password, you can use the standard `UNIX` commands such as `ls`, `pwd`, `exit`, etc. 

To download the file, just go the folder where `iclink.sas7bdat` is saved and type 

```
$ get iclink.sas7bdat [PATH]
```

into the terminal. `[PATH]` is optional and can be the path on your local machine, in my case for instance `/home/christophj/WRDS/IBES_CRSP_matching_table.sas7bdat`.

And that's how you run `SAS` files on the WRDS server and get those files (probably data sets) on your local machines.

As a side note: If you want to use another statistical software such as `R`, it is better to transform the `sas7bdat` file into a transport data set file. To do so, copy the following `SAS` code into your `pico` editor and run the script afterwards:

```
LIBNAME in_file  '~';
LIBNAME out_file XPORT '~/match_t.xpt';

PROC COPY IN=in_file OUT=out_file;
        SELECT dat;
RUN;

```

This script selects a file named *dat* (note that this file name cannot be larger than 8 characters) in the home folder and exports it into a file named *match_t.xpt*. Obviously, you have to replace *dat* with the name of the `SAS` data file. I renamed the file to *dat* instead of *iclink* because I had many files starting with that name. (The actual file I want to convert is *iclink.sas7bdat*, but apparently, you don't have to specify the file extension.)



