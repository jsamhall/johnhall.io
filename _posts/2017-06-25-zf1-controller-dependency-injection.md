---
layout: post
tags: zf1 zend-framework-1 dependency-injection ioc
date: 2017-06-25 12:28
title: ZF1 Controller Dependency Injection
thumbnail: /assets/img/zend-framework.jpg
published: true
comments: true
---

At my workplace we have an internal application built on Zend Framework 1 and naturally it my job to maintain it. One of the code smells of the project is the abundant use of the `new` keyword, largely due to the absence of any IOC container in the ZF1 framework. So I thought it would be a fun project to add dependency injection to the application.

Pretty much immediately I remembered that when working with ZF1 Action Controllers, you are encouraged to use the init() method as opposed to overriding the constructor, mainly due to the way the controller is instantiated by the Dispatcher. The standard dispatcher does so like this:

{% highlight php %}
// class: Zend_Controller_Dispatcher_Standard
public function dispatch(...){
        // ...
        $controller = new $moduleClassName($request, $this->getResponse(), $this->getParams());
        // ...
}
{% endhighlight %}

So yeah, so much for injecting into the constructor! In my research on the subject, I did happen upon the [PHP-DI project](http://php-di.org/doc/frameworks/zf1.html) but I'm not a huge fan of Annotation-based dependency injection, which is how the PHP-DI works with ZF1 controllers. So how to go about dependency injection without annotations, and without overriding the class constructor?

After reading Matthew Weier O'Phinney's blog post on [Resource Injection using an Action Helper](https://mwop.net/blog/235-A-Simple-Resource-Injector-for-ZF-Action-Controllers.html) I knew I could do something similar for dependency injection. After digging into Zend's base Action Controller's dispatch() method, I found that before running the controller's preDispatch() method, any assigned Action Helper's preDispatch() methods are run first allowing you to <i>inject</i> or decorate the controller with added functionality. Sounds like the perfect tool for dependency injection!

Rather than write my own dependency injection container, I opted to use the excellent [Container from The PHP League](http://container.thephpleague.com/). After installing container via composer, I defined a few empty classes to use in this example:

{% highlight php %}
<?php
interface TestAInterface {};
interface TestBInterface {};
interface TestCInterface {};
class TestA implements TestAInterface {}
class TestB implements TestBInterface {}
class TestC {

    public $a;
    public $b;

    public function __construct(TestAInterface $a, TestBInterface $b)
    {
        $this->a = $a;
        $this->b = $b;
    }
}
?>
{% endhighlight %}

Not only is the application bootstrap a logical place to configure Container, but Action Helpers can also access the Bootstrap via the Front Controller which will prove critical when actually injecting our dependencies. If a Bootstrapping method returns a value, Zend will store that value as a resource on the Bootstrap object-- this is how we will retrieve the configured Container in the Action Helper.

{% highlight php %}
<?php
class Bootstrap extends Zend_Application_Bootstrap_Bootstrap {
    protected function _initContainer()
    {
        $container = new League\Container\Container;
        $container->delegate(new League\Container\ReflectionContainer);

        $container->add('TestAInterface', 'TestA');
        $container->add('TestBInterface', 'TestB');

        // available via $bootstrap->getResource('container');
        return $container;
    }
}
?>
{% endhighlight %}

Next, I decided an Interface would be a suitable means for the Action Helper to ensure that the current Controller needs dependency injection. The interface has just one method which returns an array of dependencies. More on that in a second!

{% highlight php %}
<?php

namespace App\Domain\Controller;

interface Injectable
{
    /**
     * Return array of dependencies to resolve from IoC container and inject into controller 
     * Keys are the property to inject on (e.g., repository)
     * Values are class names to resolve from the Container (e.g., Domain\Whatever\Service)
     * @return array
     */
    public function getDependencies();
}
?>
{% endhighlight %}

Alright now for the fun part, the Dependency Injector Class! The code should speak for itself, but here is a basic overview:

1.  Grab the Application Bootstrap from the Front Controller, and the Controller that is currently dispatching.
2.  Ensure the Controller can be injected upon (i.e., implements the Injectable interface)
3.  Grab the Container from the Application Bootstrap (remember what I said earlier about returned values being stored as bootstrap resources!) and ensure it is an instance of Container (or whatever DI container you wish to use)
4.  Iterate over the array of dependencies using array keys as the controller property to inject on, and the array value as the class name to resolve from the DI container.
5.  If a requested class is not found in the Container then raise an Exception.
6.  Otherwise, grab the object from the DI container and inject on the controller!

Also, as seen below, I prefer to add spaces around my bangs. I think it makes them more obvious/way harder to miss when skimming over code. I know this isn't PSR-2 compliant but it is my code and therefore my preference :)

{% highlight php %}
<?php

namespace App\Domain\Controller;

use League\Container\Container;

class DependencyInjector extends \Zend_Controller_Action_Helper_Abstract
{
    public function preDispatch()
    {
        $bootstrap  = $this->getFrontController()->getParam('bootstrap');
        $controller = $this->getActionController();

        if( ! $controller instanceof Injectable) {
            return;
        }

        $container = $bootstrap->getResource('container');
        if ( ! $container instanceof Container ) {
            return;
        }

        foreach ($controller->getDependencies() as $property => $className) {
            if ( ! $container->has($className) ) {
                $message = sprintf("No class named %s found in configured DI Container", $className);
                throw new \OutOfRangeException($message);
            }

            $controller->$property = $container->get($className);
        }
    }
}
?>
{% endhighlight %}

I decided to throw an `OutOfRangeException` because to in my opinion, if a class cannot be retrieved from the Container that is a bad configuration on my behalf which fits nicely into [the documentation for OutOfRangeException](http://php.net/manual/en/class.outofrangeexception.php) definition of "errors that should be detected at compile time".

Ok, almost done! The last step is to add our new Helper to the dispatching chain which is done via a static method on Zend's HelperBroker class, as-so:

{% highlight php %}
<?php
class Bootstrap extends Zend_Application_Bootstrap_Bootstrap {
    
    // ... other bootstrap methods ...
    
    protected function _initDependencyInjector()
    {
        // explicitly bootstrap the container here. it will be automatically bootstrapped
        // by $app->getBootstrap()->bootstrap() method (usually in index.php) but this
        // ensures the container is bootstrapped *before* the helper's preDispatch method
        // asks for the 'container' bootstrap resource
        $this->bootstrap('container');

        Zend_Controller_Action_HelperBroker::addHelper(
            new \App\Domain\Controller\DependencyInjector
        );
    }
}
?>
{% endhighlight %}

Now lets see this in action!

{% highlight php %}
<?php

use App\Domain\Controller\Injectable;

class IndexController extends Zend_Controller_Action implements Injectable {
    /**
     * @var TestC
     */
    public $injectedDependency;

    /**
     * {@inheritdoc}
     */
    public function getDependencies()
    {
        return array(
            'injectedDependency' => 'TestC'
        );
    }
    
    public function indexAction(){
        var_dump($this->injectedDependency);
    }
}

?>
{% endhighlight %}

Output on page isn't very exciting, but dependency injection is working! Hurrah!

<pre><code>
object(TestC)[91]
  public 'a' => 
    object(TestA)[93]
  public 'b' => 
    object(TestB)[92]
</code></pre>




