---
layout: post
tags: php coding tips
date: 2015-10-16 12:28
<!-- thumbnail: /assets/img/goldstar.png -->
title: Tips for Writing Clean PHP Code
published: true
comments: true
---

Here are some tips that I feel help me write nice, clean, efficient and readable code.
 
<h4>Use Descriptive Variable Names</h4>
We've all seen it before: one letter variable names like $x, $y or $xx. Why do this, unless you're working with something like coordinates where variables like x and y make sense to the reader? Your IDE should auto-complete, so it's not like you're saving time. In fact, you're more likely going to end up <i>losing time</i> in the long run as whenever you re-visit such code you will be forced to read through whatever is using such variables in order to understand what is going on! Just make your variables descriptive and you will thank yourself later.

<h4>Avoid using the word "And" in Function/Method Names</h4>
You've probably heard of the <a href='https://en.wikipedia.org/wiki/Single_responsibility_principle'>Single Responsibility Principal</a> before, so you can understand that if you see the word "and" in a function or method call, that means it is likely doing more than one thing and violating SRP. Don't have a method called <b>registerAndEmailUser</b> instead, extract that functionality into two methods. You will thank yourself when it comes time perform either action of the method individually of the other!

<h4>Wrap Scalars</h4>
This is one of my favorite examples of helping your future-self or coworkers easily see what some method is doing. It always bugs me when I see some method being invoked with an ambiguous parameter being passed, like "sendInventoryReport(false)". There is no way you can surmise what that boolean "false" means without jumping to, and reading the DocBlock for the "sendInventoryReport" function and that is just a waste of time! Just for the sake of example, lets say that the sendInventoryReport function can optionally include data for out of stock items, which is what that boolean controls. Instead of passing a simple boolean, create a ValueObject called "IncludeOutOfStockItems" that simply wraps a boolean value. Now, your code would read "sendInventoryReport((new IncludeOutOfStockItems(false)))". Awesome, now the intent is clear!


