---
layout: post
tags: php coding tips clean-code
date: 2015-10-16 12:28
title: Tips for Writing Clean PHP Code
published: true
comments: true
---

Here are some tips that I feel help me write nice, clean, efficient and readable code.

<h3>Wrap Primitives/Scalars (sometimes)</h3>
This is one of my favorite examples of helping your future-self or coworkers easily see what some method is doing. It always bugs me when I see some method being invoked with an ambiguous parameter being passed, like `sendInventoryReport(false)`. There is no way you can surmise what that boolean `false` means without jumping to the definition for the `sendInventoryReport` function and that is just a waste of time! Just for the sake of example, lets say that the `sendInventoryReport` function can optionally include data for out of stock items, which is what that boolean controls. Instead of passing a simple boolean, lets use a ValueObject:
{% highlight php %}
<?php
class IncludeOutOfStockItems {
    private $shouldInclude;
    
    public function __construct($shouldInclude = true)
    {
        if ( ! is_bool($shouldInclude)) {
            throw new InvalidArgumentException("IncludeOutOfStockItems constructor requires boolean value.");
        }
        
        $this->shouldInclude = $shouldInclude;
    }
    
    public function shouldInclude()
    {
        return $this->shouldInclude;
    }
}
?>
{% endhighlight %}

Now the code reads: `sendInventoryReport(new IncludeOutOfStockItems)`. The *intent* is clear! This is one of those "use your brain" situations. Obviously there isn't a need to wrap *everything* but for circumstances like the example above, a ValueObject is perfect for making the calling code clear and concise.


<h3>Use Descriptive Variable Names</h3>
We've all seen it before: one letter variable names like `$x` or `$y` Why do this, unless you're working with something like coordinates where variables like x and y make sense to the reader? Your IDE should auto-complete, so it's not like you're saving time. In fact, you're more likely going to end up <i>losing time</i> in the long run as whenever you re-visit such code you will be forced to read through whatever is using such variables in order to understand what is going on! The same goes for abbreviations, those should be avoided as well with the exception of "ID" as it is so universally recognized in the software world. Bottom line, just make your variables descriptive and you will thank yourself later.

<h3>Avoid using the word "And" in Function/Method Names</h3>
You've probably heard of the [Single Responsibility Principal](https://en.wikipedia.org/wiki/Single_responsibility_principle) before, so you can understand that if you see the word "and" in a function or method call, that means it is likely doing more than one thing and violating SRP. Don't have a method called `registerAndEmailUser()` instead, extract that functionality into two methods. You will thank yourself when it comes time perform either action of the method individually of the other and especially when writing tests! In fact, this is one bad pattern that will be extremely obvious if you write tests for your code (which you should!)

<h3>Avoid using 'else' / Return Early</h3>
Nothing emits code smell quite like a gigantic conditional block. Not only are they hard to read, but troubleshooting can be a nightmare. I feel like you can almost always avoid using `else` in favor of returning early in the function. As a rule of thumb, I like to have my functions "desired" path result with the entire method being run. Never should the expected outcome be hidden somewhere in a conditional. Obviously there will be situations where the else operator is necessary, but for the most part I think it can be avoided.

{% highlight php %}
<?php

// bad!!
public function getConditionWrong(){
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
public function getConditionRight(){
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
Comments are good right! Well yes, they are, but some times they are entirely unneeded! For the most part, SOLID, clean, readable code does a lot of speaking for itself which makes certain comments redundant and entirely unnecessary. The the following example (I'm definitely guilty of this one!):

{% highlight php %}
<?php
public function saveOrder(Order $order){
    // validate the order
    if ( ! $this->validator->isValid($order)) {
        throw new ValidationException($this->validator->getErrors());
    }
    
    // save the order
    $this->repository->save($order);
}
?>
{% endhighlight %}

Anyone looking at this code in a glance can immediately understand what is happening, the two comments serve absolutely no purpose because the method names clearly demonstrate what is going on. Documentation is important, but useless comments serve no purpose and just clutter your code and give your brain unnecessary information to process.

<h3>Use Whitespace, it is your friend!</h3>
Whether it is in a method body or in a conditional, white space really helps with code readability. I find it especially helpful when white space is left around an exclamation point (bang), as without white space the (very important!) bang operator is (in my opinion!) just a little bit too easy to miss, especially when skimming code. I know this is a violation of [PSR-2](http://www.php-fig.org/psr/psr-2/) but I'm also [not the only one who feels this way](https://github.com/php-fig/fig-standards/issues/102) when it comes to the trade-off between readability and conforming to standards.

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
 
 I will continue to add to this list (maybe in another post, maybe right in this one) as I come up with more stuff. I hope this is helpful to someone!