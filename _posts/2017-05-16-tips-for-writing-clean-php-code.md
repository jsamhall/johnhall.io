---
layout: post
tags: php coding tips clean-code
date: 2017-05-16 12:28
title: Tips for Writing Clean PHP Code
published: true
comments: true
---

Here are some tips that I feel help me write nice, clean, efficient and readable code.

<h3>Name Variables used in Method Calls</h3>
This is one of my favorite examples of helping your future-self or coworkers easily see what some method is doing. It always bugs me when I see some method being invoked with an ambiguous parameter being passed, like `sendInventoryReport(false)`. How can you surmise what that boolean `false` means without jumping to the definition for the `sendInventoryReport` function? That's a waste of time! Just for the sake of example, lets say that the `sendInventoryReport` function can optionally include data for out of stock items, which is what that boolean controls. For improved readability just declare a variable and call the method like this:
{% highlight php %}
<?php
$includeOutOfStockItems = false; // readability
$results = sendInventoryReport($includeOutOfStockItems);
?>
{% endhighlight %}

Now the *intent* is of your code is crystal-clear! Your future self and your colleagues will thank you!


<h3>Use PHP7's Null Coalesce Operator</h3>
One of my [favorite features introduced by PHP7](https://secure.php.net/manual/en/migration70.new-features.php) is the long awaited null coalesce operator (hey, better late than never, right?!). If you're unsure what that means or how to use it, take for example this bit of code that you've probably seen something very similar to:
{% highlight php %}
<?php
$username = isset($_POST['username']) ? $_POST['username'] : null;

// or even more worse...
if (isset($_POST['username']) {
   $username = $_POST['username'];
} else {
   $username = null;
}
?>
{% endhighlight %}

What we're saying here is if the `username` key is set in the `$_POST` superglobal, assign that to `$username`, otherwise set `$username` to null. With the glory of the null coalesce operator introduced by PHP7, this assignment can be shortened to:

{% highlight php %}
<?php
$username = $_POST['username'] ?? null;
?>
{% endhighlight %}

Much better, right?! Of course, we know better than to trust data sent from the client, but that's a topic for another day :)

<h3>Use Descriptive Variable Names</h3>
We've all seen it before: one letter variable names like `$x` or `$y` Why do this, unless you're working with something like coordinates where variables like x and y make sense to the reader? Your IDE should auto-complete, so it's not like you're saving time. In fact, you're more likely going to end up <i>losing time</i> in the long run as whenever you re-visit such code you will be forced to read through whatever is using such variables in order to understand what is going on! The same goes for abbreviations, those should be avoided as well with the exception of "ID" as it is so universally recognized in the software world. Bottom line, just make your variables descriptive and you will thank yourself later.

<h3>Avoid using the word "And" in Function/Method Names</h3>
You've probably heard of the [Single Responsibility Principal](https://en.wikipedia.org/wiki/Single_responsibility_principle) before, so you can understand that if you see the word "and" in a function or method call, that means it is likely doing more than one thing and violating SRP. Don't have a method called `registerAndEmailUser()` instead, extract that functionality into two methods. You will thank yourself when it comes time perform either action of the method individually of the other and especially when writing tests! In fact, this is one bad pattern that will be extremely obvious if you write tests for your code (which you should!)

<h3>Avoid using 'else' / Return Early</h3>
Nothing emits code smell quite like a gigantic conditional block. Not only are they hard to read, but troubleshooting can be a nightmare. I feel like you can almost always avoid using `else` in favor of returning early in the function. As a rule of thumb, I like to have my function's "desired" outcome at the end of the method. Of course there are exceptions to this (e.g., the result was previously cached) however I feel it's safe to say that the expected outcome of a method should never be hidden somewhere in a conditional. Obviously there will be situations where the else operator is necessary, but for the most part I think it can be avoided.

{% highlight php %}
<?php

// bad!!
public function getCondition(){
    if ($conditionA) {
        return 'condition a';
    } else {
        if ($conditionB) {
           return 'condition b';
        } else {
            return 'unknown condition';
        }
    }
}

// good!!
public function getCondition(){
    if ($conditionA) {
        return 'condition a';
    }
    
    if ($conditionB) {
       return 'condition b';
    }
    
    return 'unknown condition';
}
?>
{% endhighlight %}

<h3>Write Meaningful Comments</h3>
Comments are good right! Well yes, they are, but at times they are entirely unnecessary! For the most part, SOLID, clean, readable code does a lot of speaking for itself which makes comments like what you're about to see redundant, resulting in bloated code. Are the comments in this code block necessary? I think not!

{% highlight php %}
<?php
/**
 * Saves the order
 */
public function saveOrder(Order $order){
    // validate the order
    if ( ! $this->validator->isValid($order)) {
        throw new InvalidOrderException($this->validator->getErrors());
    }
    
    // save the order
    $this->repository->save($order);
}
?>
{% endhighlight %}

Anyone looking at this code in a glance can immediately understand what is happening, the two comments serve absolutely no purpose because the method names clearly demonstrate what is going on. Documentation is important, but useless comments serve no purpose and just clutter your code and give your brain unnecessary information to process.

<h3>Whitespace is your friend!</h3>
Whether it is in a method body or in a conditional, white space really helps with code readability. I find it especially helpful when white space is left around an exclamation point (bang), as without white space the (very important!) bang operator is (in my opinion!) just a little bit too easy to miss, especially when skimming code. I know this is a violation of [PSR-2](http://www.php-fig.org/psr/psr-2/) but I'm also [not the only one who feels this way](https://github.com/php-fig/fig-standards/issues/102) when it comes to the trade-off between readability and conforming to standards. Most modern editors have settings that allow you to fine-tune how your code is auto-formatted; make use of those features!

{% highlight php %}
<?php
// example 1) lack of white space makes the bang easy to miss
public function doSomethingImportant(SuperImportant $objectOfGreatImportance){
    if (!$this->allowSuperImportantStuffToHappen()) {
        return false;
    }
    
    // ... important stuff...
}

// example 2) white space makes the bang very much "in your face"
public function doSomethingImportant(SuperImportant $objectOfGreatImportance){
    if ( ! $this->allowSuperImportantStuffToHappen()) {
        return false;
    }
    
    // ... important stuff...
}
?>
{% endhighlight %}

Did you miss the exclamation point in example 1? Maybe you didn't now, but if you're working late trying to make a deadline, you certainly just might miss it then :)
 
I think that does it for today! I hope you found this helpful. If you agree or disagree, or perhaps have tips or nightmare stories of your own to share, I'd love to hear from you in the comment section!