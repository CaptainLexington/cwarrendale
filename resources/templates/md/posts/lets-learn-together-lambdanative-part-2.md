{:title "Let's Learn Together: LambdaNative Part 2"
 :layout :post
 :date "2016-03-25"
 :draft? false
 :tags ["lets-learn-together" "programming" "functional-programming" "lambdanative" "bowlliards" "scheme"]}

Welcome back to the Let's Learn Together! Last time we installed LambdaNative from source and generated an empty template GUI application. Today we are going to examine their GUI library. But before we get to that, let's examine what kind of basic interface it is we want.

We are going to steal our UI from extant Android application [Touch de Score Bowlard](https://play.google.com/store/apps/details?id=jp.dcom.android.bowlard)—specifically, the paradigm of touching each specific ball as it is sunk. However, play around with the app a bit and you'll see it has several UI issues:

* It's impossible to undo any mistakes in input - if you accidentally hit the "G" button (which means "gutterball"), you've scrapped that whole frame and can't change your score.
* There's a separate button for "Spare" and "Strike" - the UI should just detect spares and strikes because it should know what turn you're on.
* You can only keep your own score - you can't keep score for an entire game of multiple players.

So, we're going to go our own way on these issues. Here's what we want out of our app:

* A diagram of the balls in play. You click on a ball to indicate you've sunk it, and your score is incremented. The ball gets grayed out to demonstrate that it is out of play, and if you tap it again it comes back into play - for instance, if you've made a mistake. There is additionally a button to advance to the next frame.
* A picture of a scorecard at the top of the screen indicates the current player, their running total score, and their current frame.
* There are arrow buttons near the scorecard that allow you to cycle back to earlier frames to alter those scores.
* It is possible to go to a separate screen to view the entire scorecard for the every player so far, like a traditional bowling alley scorecard.
* There is a label of text beneath the scorecard that instructs the player on their next move, according to the mildly complicated rules of the game.

I think that's everything we need to worry about for now. For this first example, we'll want our ten "ball" buttons on the screen.

Well, let's get started!

## Let's Learn Together: LambdaNative's GUI system 

All LambdaNative GUI applications need to make use of the module [ln\_glgui](https://github.com/part-cw/lambdanative/wiki/Index-of-Module-ln\_glgui), documented in the wiki.

It looks like we want a [`glgui-button`](https://github.com/part-cw/lambdanative/wiki/glgui-button). It puts an image texture at an x,y location on the screen and attaches a callback. First, we'll need image textures! I made up a simple suite of pool ball graphics, to be released under [CC-by-sa 4.0](http://creativecommons.org/licenses/by-sa/4.0/), like the text of this blog.

<figure>
<label for="pool-balls" class="margin-toggle"></label>
<input type="checkbox" id="pool-balls" class="margin-toggle">
<span class="marginnote">The SVG-based pool ball graphics we will use. CC-by-sa 2016 C Warren Dale</span>
<img src="/img/balls.png">
</figure>

I've made a complete pool set so people can use it for their own purposes, even though we'll only use balls one-through ten. I've also chopped them into separate files called `ballK.png` and `ballK-gray.png`, for 1 ≤ K ≤ 15. 

So, the first thing we want to do is load all our images. LambaNative has a wrapper around libpng, [png](https://github.com/part-cw/lambdanative/wiki/Index-of-Module-png), which looks promising; specifically the `png->img` function, which loads png images by filename and returns an image texture like our `glgui-button` is expecting.

So we'll want to take a list of the numbers one through ten, map it to a list of image textures, and and pass those to a function that creates buttons, along with a list of x,y coords. So the first thing we need is range.

```scheme
1> range
*** ERROR IN (console)@9.1 -- Unbound variable: range
```

Huh. Well, Google indicates Scheme thinks it makes sense to call the function we all know as range `iota`. Okay. 

```scheme
2> iota
*** ERROR IN (console)@10.1 -- Unbound variable: iota
```

Okay. So Gambit-C doesn't have an implementation of iota. We're going to do a terrible thing and hardcode the identifiers for the balls, justifying this by loading in the cue ball as well! 

At the top of `main.scm`:

```scheme
 ;; An app for keeping score in a game of bowlliards
 (define gui #f)
 (define balls
   '("1" "2" "3" "4" "5"
     "6" "7" "8" "9" "10" 
     "cue"))
```

And in the `initialization` function:
```scheme
;; initialization
  (lambda (w h)
            ;; We map the balls list to a list of image pairs corresponding to default and grayed-out balls
    (let ([ball-images (map
			(lambda (ball)
			  `((png->img (string-append "resources/ball" ball ".png"))
			    (png->img (string-append "resources/ball" ball "-gray.png"))))
			balls)])
    (make-window 320 480)
    (glgui-orientation-set! GUI_PORTRAIT)
    (set! gui (make-glgui)))
    ;; initialize gui here
    (for-each (lambda (imgs) 
		(glgui-button gui
			      150
			      150
			      50
			      50
			      imgs
			      (lambda () #f)))
	      ball-images))
```

This should stack 15 buttons on top of each other, with only the last visible. That's not useful to use but I just want to make sure it works how I think it will.

Okay, let's build!

```bash
$ make
  [... no errors in compilation ...]
  [... executable is ~/,cache/lambdanative/[target]/bowlliards/bowlliards ...]

$ ~/.cache/lambdanative/linux/bowlliards/bowlliards
```


<figure>
<img src="/img/bowlliards-screenshot-1.png">
</figure>


I don't know about you, but I don't see anything on the screen right now.

That's okay! There's a section on the wiki about [Debugging](https://github.com/part-cw/lambdanative/wiki/Debugging)! Let's check it out.

> Log file location: By default LambdaNative sets the `current-exception-handler` to [log:exception-handler](https://github.com/part-cw/lambdanative/blob/master/modules/ln\_core/log.scm). This results in all errors and additional desired logging information to be written to the applications `log/` folder.

This is not really what I was expecting. I want to use a REPL to see if my functions are doing what I think they are doing. So let's Google "[lambdanative repl](https://www.google.com/search?q=lambdanative+repl)".

The [first result](https://webmail.iro.umontreal.ca/pipermail/gambit-list/2013-September/006976.html) indicates that getting a REPL running involves learning more about Gambit-C applications than I know:

>Hi,
>I started a remote repl with the demoredsquare app but the repl get too slow, somehow the app takes forever to process the repl events
```scheme
main.scm:
(include "repl.scm")

(define gui #f)

(start-repl-server)
```

We'll need to learn this stuff eventually because we won't be able to make an entire LISP application without REPL access, but for now let's try more rudimentary forms of debugging.

The first thing that occurs to me is that I assumed, rather uncharitably, that `glgui-button` was a side-effectful function that mutated the gui object it was sent - and therefore I used `for-each` to create each button. Let's try making just a single button, but calling `glgui-button` in such a way that if it *returns* a GUI object of some kind, the initialization function will be able to handle it.

```scheme
   ;; initialize gui here
    (glgui-button gui
		  150
		  150
		  50
		  50
		  (png->img "resources/ball8.png")
		  (lambda () #f)))
```

```bash
$ ~/.cache/lambdanative/linux/bowlliards/bowlliards
 Segmentation fault (core dumped)
``` 

Okay. Maybe the images aren't being packaged up with the executable correctly. Let's see what LambdaNative thinks should happen.

The [Applications](https://github.com/part-cw/lambdanative/wiki/Applications) page on the wiki says I should call this directory `textures/`. Furthermore, the [page on the textures directory](https://github.com/part-cw/lambdanative/wiki/textures%28dir%29) seems to imply we don't need to manually load the images—this happens during the compilation step, and the image textures can be accessed as `<png name>.img`.  Okay, let's rename the directory and change the file we pass to the button!

<figure>
<img src="/img/bowlliards-screenshot-2.png">
</figure>

Looks, it's an image! We're so close to the end now I can almost taste it.

Okay, it looks like the library is not going to resize our images for us. We're just a few lines of ImageMagick away from making them smaller on our own!

Unfortunately it looks like we have to hardcode our calls to `glgui-button`, so I'm going to do that and get back to you when it looks sharp. Looks like we won't even be using out ball list.

Okay, I'm back! Our new `main.scm`:

```scheme
;; An app for keeping score in a game of bowlliards
(define gui #f)
(define balls
  '("1" "2" "3" "4" "5"
    "6" "7" "8" "9" "10" 
    "11" "12" "13" "14" "15"
    "cue"))

(define (ball-button x y img)
  (glgui-button gui
		x
		y
		64
		64
		img
		(lambda () #t)))

(main
;; initialization
  (lambda (w h)
            ;; We map the balls list to a list of image pairs corresponding to default and grayed-out balls
     
    (make-window 320 480)
    (glgui-orientation-set! GUI_PORTRAIT)
    (set! gui (make-glgui)) 
    ;; initialize gui here
    (ball-button 128
		 64
		 ball1.img)
    (ball-button 96
		 128
		 ball2.img)
    (ball-button 160
		 128
		 ball3.img)
    (ball-button 62
		 192
		 ball4.img)
    (ball-button 126
		 192
		 ball5.img)
    (ball-button 190
		 192
		 ball6.img)
    (ball-button 32
		 256
		 ball7.img)
    (ball-button 96
		 256
		 ball8.img)
    (ball-button 160
		 256
		 ball9.img)
    (ball-button 224
		 256
		 ball10.img))

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

Make it and run it, and:

<img src="/img/bowlliards-screenshot-3.png">

Well, that took longer than I expected, but we made it. Right now if you click on any of the balls, you get a segfault and the program crashes. Next time we'll hook up the buttons to some real functionality!
