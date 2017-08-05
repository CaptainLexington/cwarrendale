{:title "Let's Learn Together: LambdaNative, Part 1"
 :layout :post
 :tags ["programming" "lets-learn-together" "functional-programming" "scheme" "lambdanative" "bowlliards"]}


<div class="epigraph">
	<blockquote>
	MARGE: But I could... give piano lessons.

	LISA: But you don't play the piano.

	MARGE: I just got to stay one lesson ahead of the kid.	
	<footer>The Simpsons, <cite>S14E12, "<a href="http://simpsons.wikia.com/wiki/I'm\_Spelling\_as\_Fast\_as\_I\_Can">I'm Spelling as Fast as I Can</a>"</cite></footer>
	</blockquote>
</div>

## Let's Learn Together

I am going to kick off my new blog with a new series of posts, called *Let's Learn Together*. I suspect most "Getting Started" tutorials you see online are written more-or-less just after the author has learned how to start off themselves, but they have already learned from their beginners' mistakes and the finished posts make them look like fonts of wisdom. I am going to continue in this proud tradition, only I am going to keep in all of my mistakes, for your reading pleasure. **Don't follow along with this "tutorial" as you read it, because you will certainly do unnecessary work. Read through the whole post to a get big-picture understanding of what to do - and what not to do!** 

## Motivation

Being, as I have written, an *opinionated* technologist, I choose my technology stack according to my preferences for development characteristics. I prefer functional programming to imperative (by a wide margin), and as such I haven't done much in the way of mobile development. I did aid in porting a native Java application to Android a few years back, but the experience was not ideal and it did not serve to entice me further down the path of native mobile development. The mere prospect of coding an entire project in the Android SDK or - shudder to think it - in Objective-C thrilled me, filled me with fantastic terrors never felt before. So I never pursued it.

I did work with PhoneGap on an iOS project, and in fact, as a front-end developer by day, I found the experience quite pleasant. I never returned to it, though, because I wouldn't *choose* to work in JavaScript on a personal project; I wanted a cross-platform experience a little more suited to my idiosyncratic tastes.

Enter [LambdaNative](http://lambdanative.org)! LambdaNative is a cross-platform (mobile AND desktop) development environment written in Scheme, the language that, when I learned it freshman year of college, was my first breatheless introduction to the world of functional programming. Sounds great! Now, what to do with it?

## The Project

What mobile app should we make? A lame Twitter clone? Some worthless Bejeweled-style casual game? Uber for dogs? I think...*no*.

I really like pool. In fact, I *love* pool. My brother has a pool table, and although we play together as often as we can, he doesn't always have someone else to play with. Looking for decent skill-building solo games, we discovered [bowlliards](https://en.wikipedia.org/wiki/Bowlliards). Bowling scoring is famously a little bit complicated, however, and we soon looked for automated ways of keeping score. Although several high-quality bowling score-keeping apps exist, they are all too focused on bowling. Existing apps in the bowlliards space are buggy, narrowly tailored to their platforms, and unmaintained. A vacuum exists, ready for a cross-platform, open-source solution!

## Getting Started with LambdaNative

LambdaNative has extensive documentation, but how to actually get the environment set up is not entirely clear.

### First attempt.

My first attempt at getting LambdaNative set up involved [cloning the GitHub repo](https://github.com/part-cw/lambdanative). Then I was sitting in the repo, looking at its contents. What now?

```bash
$ ls
  apps	        docs	   libraries	  loaders   modules	      README.md	      targets    VERSION
  config.guess  fonts	   LICENSE	  Makefile  plugins	      scripts	      templates
  configure     languages  LNCONFIG.h.in  make.sh   PROFILE.template  SETUP.template  tools
```

Well, this is the source repository. Probably you have to build the package if you want to use it to develop against. Let's try `make`ing the project.

```bash
$ make
  Error: ~/.lambdanative/config.cache: file does not exist
```

Hmm. Well, the [wiki](https://github.com/part-cw/lambdanative/wiki) says something about "[Compilation](https://github.com/part-cw/lambdanative/wiki/Compilation)", that sounds promising!

>The configure command follows the format:
```bash
>./configure <app> [android|ios|bb10|macosx|win32|linux|linux486|openbsd] [debug|release] [verbose]
```	
>`<app>` is the name of the application, and must match a directory name in the `./apps` framework subdirectory.	

Okay, so this page is actually about compiling projects we make *with* LambdaNative. Fair enough, that will be useful later! But from here I see another page, about "[System-wide Installation](https://github.com/part-cw/lambdanative/wiki/System-wide-installation)". Let's have a look!

### Second Attempt  
  
>The LambdaNative package, obtained from [LambdaNative Releases](https://github.com/part-cw/lambdanative/releases) should be placed in a system-wide location for third-party programs, such as `/usr/local` or `/opt`. Next, the LambdaNative initialization script, located in scripts/lambdanative needs to be placed in system path, e.g. `/bin` or `/usr/bin`. Finally, the `LAMBDANATIVE=` variable in this script needs to be overwritten with actual full LambdaNative path. The system is now ready for use.

Okay, so the package is available in this release! Let's download one and see what we have.

\*downloads\*

```bash
$ tar xvf lambdanative-1.0.7.tar.gz
[lots of unzipping]
$ cd lambdanative-1.0.7/
$ ls
  apps	        docs	   libraries	  loaders   modules	      README.md	      targets    VERSION
  config.guess  fonts	   LICENSE  	  Makefile  plugins	      scripts	      templates
  configure     languages  LNCONFIG.h.in  make.sh   PROFILE.template  SETUP.template  tools
```

Hmm. That looks familiar. The major release isn't a build, it's just a canonical package of the source code. Again, this makes sense in retrospect! So now what? The Wiki insists that all I need to do is put the "package" in some globally-accessible directory and then I can use the scripts. Maybe they mean...the source repository? In `/opt`? Well, let's give it a shot.

```bash
# cp lambdanative/scripts/lambdanative /usr/bin
# chmod +x /usr/bin/lambdanative
# mv lambdanative /opt
```

Then we set `LAMBDANATIVE= /opt/lambdanative` like the wiki says, and...

```bash
$ lambdanative init
  lambdanative: line 5: /opt/lambdanative/: Is a directory
  ERROR: this script has not been configured.
```

Okay, that's not it.

### Final Attempt

While Googling for solutions, I discovered that [LambdaNative is packed up in Arch Linux's "Arch User Repository"](https://aur.archlinux.org/packages/lambdanative/), and it's totally up-to-date. Luckily, I am an Arch Linux user! I install the package with yaourt and leave you to struggle along on your own.

Actually, now that the installation worked, let's see what `LAMBDANATIVE` *actually* refers to.

```bash
$ head /usr/bin/lambdanative
  #!/bin/sh
  # LambdaNative initialization script to be placed in system path

  # Override this with actual lambdanative location
  LAMBDANATIVE=/usr/share/lambdanative

  if [ "X$LAMBDANATIVE" = "X" ]; then
    echo "ERROR: this script has not been configured."
    exit
  fi
```

It's set to `/usr/share/lambdanative`. Let's see what's that all about!

```bash
$ ls /usr/share/lambdanative
  apps	        docs	   libraries	  Makefile  plugins	      scripts	      targets    VERSION
  config.guess  fonts  	   LNCONFIG.h.in  make.sh   PROFILE.template  SETUP	      templates
  configure     languages  loaders	  modules   README.md	      SETUP.template  tools
```

What! That IS the source repo after all. What went wrong???

Oops. When I tried editing the `lambdanative` script, I put a space between the `=` and the path to lambdanative. So everything died, but instead of throwing a syntax error, it set `LAMBDANATIVE` to nothing (hence the second error message) and tried to execute the directory (hence the first). I hate bash. Moving on.  

## Creating the foundation of our application

Now that that's set up, get ready to make our LambdaNative development directory. The environment is designed so all your LambdaNative projects are developed in the same directory tree, with their sources in the `lambdanative/apps` directory.

```bash
$ mkdir lambdanative
$ cd lambdanative
$ lambdanative init
$ lambdanative create bowlliards gui
$ ls
  apps	     Makefile  PROFILE
  configure  modules   README.md
```

Now, check out the template source file, `apps/bowlliards/main.scm`:

```scheme
;; LambdaNative gui template

(define gui #f)

(main
;; initialization
  (lambda (w h)
    (make-window 320 480)
    (glgui-orientation-set! GUI_PORTRAIT)
    (set! gui (make-glgui))

    ;; initialize gui here

  )
;; events
  (lambda (t x y) 
    (if (= t EVENT_KEYPRESS) (begin 
      (if (= x EVENT_KEYESCAPE) (terminate))))
    (glgui-event gui t x y))
;; termination
  (lambda () #t)
;; suspend
  (lambda () (glgui-suspend) (terminate))
;; resume
  (lambda () (glgui-resume))
)

;; eof
```

This is a simple event-loop driven program that listens for key events and quits when you hit the Escape key. You can compile it right now and try it out! Next time, we'll learn - together! - how to create GUI controls.
