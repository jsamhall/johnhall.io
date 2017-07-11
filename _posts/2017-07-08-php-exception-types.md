---
layout: post
tags: php best-practices exceptions code-style readability
date: 2017-07-08 10:07
title: PHP Exception Types
thumbnail: /assets/img/php-logo-square.png
published: true
comments: true
---

Exceptions- you've probably seen them and maybe you've even `thrown` a few of your own- but are you making full use of custom Exception types for managing code flow, and improving readability?

I've had junior developers ask me the same question many-a-time: "Why would I want to use my own Exception type when PHP offers so many out of the box?". Good question! By writing our own Exception classes not only can we grant ourselves extra functionality (e.g., storing an array of messages) but we can also improve code stablity and readability by limiting a `catch` block to a certain type of `Exception`.

Take for example this code that you might find in a Controller class in your application:
{% highlight php %}
<?php
// class: ProductController
public function save(Request $request){
     try {
        $product = $this->repository->find($request->getParam('product_id'));

        $formData = $request->getParam('formdata');
        $this->validateFormData($formData);

        $product->setData($formData);

        return $this->repository->persist($product);
     } catch(Exception $e) {
        return $e->getMessage();
     }
}
?>
{% endhighlight %}

Now that you have written your controller action code, you submit your form and voila, all works as expected! Job done, right? I think not! What happens if for example, an unexpected database condition results in a `PDOException` being raised? Well, your code just output a scary database error to the client! Bad news! Lets see how custom Exception types might prevent this scenario.

{% highlight php %}
<?php
// class: ProductController
public function save(Request $request){
     try {
        // throws ProductNotFoundException
        $product = $this->repository->find($request->getParam('product_id'));

        $formData = $request->getParam('formdata');
        $this->validateFormData($formData); // throws ValidationFailedException

        $product->setData($formData);

        return $this->repository->persist($product);
     } catch(ProductNotFoundException $e) {
        return "That product no longer exists! Maybe it was deleted by another User?";
     } catch(ValidationFailedException $e) {
        $tpl = "Please fix the following issue(s) and try again: %s";
        $issues = implode(',', $e->getFailureMessages());

        return sprintf($tpl, $issues);
     }
}
?>
{% endhighlight %}

Much better! Not only is this code easier to read and understand, we've also granted ourselves additional functionality, better testability and improved the user's experience with more informative messages. However we aren't quite done yet! Our cleaner code has introduced an issue in that it no longer handles library `Exception` classes. So, what happens if (as-per our previous example) a `PDOException` was thrown? Assuming there are no `catch` blocks further up the stack, PHP will invoke the [Exception Handler Function](https://secure.php.net/manual/en/function.set-exception-handler.php) that you have (hopefully!) defined.

Your Exception Handler is application-specific, but you will probably require different behavior depending on the current execution context (e.g., HTTP, CLI) and application environment (e.g., development, product). For example, your Exception Handler for your Production environment might generate an error ID and display that to the user with instructions on how to contact your Support team, whereas your Exception Handler for your Development Environment might just dump the contents of the uncaught Exception. If a blog post on custom Exception Handlers is something you're interested in, tell me in the comment section!