# Fundamental programming with ruby examples

This is repo keeps examples with description modern principles, patterns.

## Solid

In computer programming, SOLID (single responsibility, open-closed, Liskov substitution, interface segregation and dependency inversion) is a mnemonic acronym introduced by Michael Feathers for the "first five principles" named by Robert C. Martin in the early 2000s that stands for five basic principles of object-oriented programming and design. The intention is that these principles, when applied together, will make it more likely that a programmer will create a system that is easy to maintain and extend over time. The principles of SOLID are guidelines that can be applied while working on software to remove code smells by causing the programmer to refactor the software's source code until it is both legible and extensible. It is part of an overall strategy of agile and Adaptive Software Development.

####SRP - Single responsibility principle
A class should have only a single responsibility.
 
Every class should have a single responsibility, and that responsibility should be entirely encapsulated.
All its services should be narrowly aligned with that responsibility, this embrace the high cohesion.

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/solid/single_responsibility.rb)
     
####OCP - Open/closed principle
Software entities should be open for extension, but closed for modification.
That is, such an entity can allow its behaviour to be extended without modifying its source code.

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/solid/open_close.rb)
 
####LSP - Liskov substitution principle
Objects in a program should be replaceable with instances of their subtypes without altering the correctness of that program.

It states that, in a computer program, if S is a subtype of T, then objects of type T may be replaced with objects of type S (i.e., objects of type S may substitute objects of type T) without altering any of the desirable properties of that program (correctness, task performed, etc.)

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/solid/liskov_substitution.rb)

####ISP - Interface segregation principle
Many client-specific interfaces are better than one general-purpose interface.

States that no client should be forced to depend on methods it does not use.
ISP splits interfaces which are very large into smaller and more specific ones so that clients will only have to know about the methods that are of interest to them. Such shrunken interfaces are also called role interfaces. 
ISP is intended to keep a system decoupled and thus easier to refactor, change, and redeploy.

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/solid/interface_segregation.rb)

####DIP - Dependency inversion principle
One should Depend upon Abstractions, Do not depend upon concretions.

Refers to a specific form of decoupling software modules. When following this principle, the conventional dependency relationships established from high-level, policy-setting modules to low-level, dependency modules are inverted (i.e. reversed), thus rendering high-level modules independent of the low-level module implementation details. 

####Code and articles were taken from resources:

* [https://gist.github.com/khusnetdinov/9d8f50fdcaab197871b31578f2e14d5d] (https://gist.github.com/khusnetdinov/9d8f50fdcaab197871b31578f2e14d5d) 

* [https://robots.thoughtbot.com/back-to-basics-solid] (https://robots.thoughtbot.com/back-to-basics-solid) 

* [https://subvisual.co/blog/posts/19-solid-principles-in-ruby] (https://subvisual.co/blog/posts/19-solid-principles-in-ruby) 

* [http://blog.siyelo.com/solid-principles-in-ruby/] (http://blog.siyelo.com/solid-principles-in-ruby/)

## Threads

Note about parallelism and concurrency: The primary difference between using processes versus threads is the way that memory is handled. At a high level, processes copy memory, while threads share memory. This makes process spawning slower than thread spawning, and leads to processes consuming more resources once running. Overall, threads incur less overhead than processes. This Thread API is a Ruby API. I've hinted that the different Ruby implementations have different underlying threading behaviours.

#### Green Thread

Ruby 1.9 replaced green threads with native threads. However, the GIL is still preventing parallelism. That being said, concurrency has been improved through better scheduling. The new scheduler makes context-switch decisions more efficient, by essentially moving them to a separate native thread, known as the timer thread.

#### GIL - Global Interpreter Lock

MRI has a global interpreter lock (GIL). It's a lock around the execution of Ruby code. This means that in a multi-threaded context, only one thread can execute Ruby code at any one time.

So if you have 8 threads busily working on a 8-core machine, only one thread and one core will be busy at any given time. The GIL exists to protect Ruby internals from race conditions that could corrupt data. There are caveats and optimizations, but this is the gist.

This has some very important implications for MRI. The biggest implication is that Ruby code will never run in parallel on MRI. The GIL prevents it.

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/threads/gil.rb)

#### Mutex - Mutual Execution

Mutexes provide a mechanism for multiple threads to synchronize access to a critical portion of code. In other words, they help bring some order, and some guarantees, to the world of multi-threaded chaos.

The name 'mutex' is shorthand for 'mutual exclusion.' If you wrap some section of your code with a mutex, you guarantee that no two threads can enter that section at the same time.

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/threads/mutex.rb)

#### Fibers

Fibers are primitives for implementing light weight cooperative concurrency in Ruby. Basically they are a means of creating code blocks that can be paused and resumed, much like threads. The main difference is that they are never preempted and that the scheduling must be done by the programmer and not the VM. As opposed to other stackless light weight concurrency models, each fiber comes with a small 4KB stack. This enables the fiber to be paused from deeply nested function calls within the fiber block.

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/threads/fibers.rb)

#### Rails - Thread-safety in the framework

The problem with this is that there is no simple way to say with absolute certainty whether an app as a whole is thread-safe.

* Global variables are global. This means that they are shared between threads. If you weren’t convinced about not using global variables by now, here’s another reason to never touch them. If you really want to share something globally across an app, you are more than likely better served by a constant (but see below), anyway.

* Class variables. For the purpose of a discussion about threads, class variables are not much different from global variables. They are shared across threads just the same way.The problem isn’t so much about using class variables, but about mutating them. And if you are not going to mutate a class variable, in many cases a constant is again a better choice.

* Class instance variables. But maybe you’ve read that you should always use class instance variables instead of class variables in Ruby. Well, maybe you should, but they are just as problematic for threaded programs as class variables.

* Memoization by itself is not a thread safety issue. However, it can easily become one for a couple of reasons:

  - It is often used to store data in class variables or class instance variables (see the previous points).
  - The ||= operator is in fact two operations, so there is a potential context switch happening in the middle of it, causing a race condition between threads.
  - It would be easy to dismiss memoization as the cause of the problem, and tell people just to avoid class variables and class instance variables. However, the issue is more complex than that.
  - In this issue, Evan Phoenix squashes a really tricky race condition bug in the Rails codebase caused by calling super in a memoization function. So even though you would only be using instance variables, you might end up with race conditions with memoization.

  Make sure memoization makes sense and a difference in your case. In many cases Rails actually caches the result anyway, so that you are not saving a whole lot if any resources with your memoization method.
  Don’t memoize to class variables or class instance variables. If you need to memoize something on the class level, use thread local variables (Thread.current[:baz]) instead. Be aware, though, that it is still kind of a global variable. So while it’s thread-safe, it still might not be good coding practice.

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/threads/rails.rb)

#### Code and articles were taken from resources:

* [http://www.jstorimer.com/blogs/workingwithcode/8085491-nobody-understands-the-gil] (http://www.jstorimer.com/blogs/workingwithcode/8085491-nobody-understands-the-gil)

* [https://en.wikipedia.org/wiki/Global_interpreter_lock] (https://en.wikipedia.org/wiki/Global_interpreter_lock)

* [http://www.csinaction.com/2014/10/10/multithreading-in-the-mri-ruby-interpreter/] (http://www.csinaction.com/2014/10/10/multithreading-in-the-mri-ruby-interpreter/)

## Design patterns

### Creational pattern

In software engineering, creational design patterns are design patterns that deal with object creation mechanisms, trying to create objects in a manner suitable to the situation. The basic form of object creation could result in design problems or in added complexity to the design. Creational design patterns solve this problem by somehow controlling this object creation. Creational design patterns are composed of two dominant ideas. One is encapsulating knowledge about which concrete classes the system uses. Another is hiding how instances of these concrete classes are created and combined.

[Read wiki] (https://en.wikipedia.org/wiki/Creational_pattern)

#### Abstract factory pattern

The abstract factory pattern provides a way to encapsulate a group of individual factories that have a common theme without specifying their concrete classes. In normal usage, the client software creates a concrete implementation of the abstract factory and then uses the generic interface of the factory to create the concrete objects that are part of the theme. The client doesn't know (or care) which concrete objects it gets from each of these internal factories, since it uses only the generic interfaces of their products. This pattern separates the details of implementation of a set of objects from their general usage and relies on object composition, as object creation is implemented in methods exposed in the factory interface.

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/patterns/creational/abstract_factory.rb)

#### Builder pattern

The builder pattern is an object creation software design pattern. Unlike the abstract factory pattern and the factory method pattern whose intention is to enable polymorphism, the intention of the builder pattern is to find a solution to the telescoping constructor anti-pattern[citation needed]. The telescoping constructor anti-pattern occurs when the increase of object constructor parameter combination leads to an exponential list of constructors. Instead of using numerous constructors, the builder pattern uses another object, a builder, that receives each initialization parameter step by step and then returns the resulting constructed object at once.

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/patterns/creational/builder.rb)

#### Factory pattern

In class-based programming, the factory method pattern is a creational pattern that uses factory methods to deal with the problem of creating objects without having to specify the exact class of the object that will be created. This is done by creating objects by calling a factory method—either specified in an interface and implemented by child classes, or implemented in a base class and optionally overridden by derived classes—rather than by calling a constructor.

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/patterns/creational/factory.rb)

#### Prototype pattern

The prototype pattern is a creational design pattern in software development. It is used when the type of objects to create is determined by a prototypical instance, which is cloned to produce new objects. This pattern is used to:

* avoid subclasses of an object creator in the client application, like the abstract factory pattern does.
* avoid the inherent cost of creating a new object in the standard way (e.g., using the 'new' keyword) when it is prohibitively expensive for a given application.

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/patterns/creational/prototype.rb)

#### Singleton pattern

Ensure a class only has one instance, and provide a global point of access to it. This is useful when exactly one object is needed to coordinate actions across the system. The concept is sometimes generalized to systems that operate more efficiently when only one object exists, or that restrict the instantiation to a certain number of objects.

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/patterns/creational/singleton.rb)

#### Not covered patterns:

* [Lazy initialization] (https://en.wikipedia.org/wiki/Lazy_initialization)

* [Multiton] (https://en.wikipedia.org/wiki/Multiton_pattern)

* [Object pool] (https://en.wikipedia.org/wiki/Object_pool_pattern)

* [Resource acquisition is initialization] (https://en.wikipedia.org/wiki/Resource_Acquisition_Is_Initialization)

* [Utility] (https://en.wikipedia.org/wiki/Utility_pattern)

### Structural pattern

In software engineering, structural design patterns are design patterns that ease the design by identifying a simple way to realize relationships between entities.

[Read wiki] (https://en.wikipedia.org/wiki/Structural_pattern)

#### Adapter pattern

In software engineering, the adapter pattern is a software design pattern that allows the interface of an existing class to be used as another interface. It is often used to make existing classes work with others without modifying their source code.

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/patterns/structural/adapter.rb)

#### Composite pattern

The composite design pattern is a structural pattern used to represent objects that have a hierarchical tree structure. It allows for the uniform treatment of both individual leaf nodes and of branches composed of many nodes.

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/patterns/structural/composite.rb)

#### Decorator pattern

In object-oriented programming, the decorator pattern (also known as Wrapper, an alternative naming shared with the Adapter pattern) is a design pattern that allows behavior to be added to an individual object, either statically or dynamically, without affecting the behavior of other objects from the same class. The decorator pattern is often useful for adhering to the Single Responsibility Principle, as it allows functionality to be divided between classes with unique areas of concern.

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/patterns/structural/decorator.rb)

#### Facade pattern

The Facade design pattern is often used when a system is very complex or difficult to understand because the system has a large number of interdependent classes or its source code is unavailable. This pattern hides the complexities of the larger system and provides a simpler interface to the client. It typically involves a single wrapper class which contains a set of members required by client. These members access the system on behalf of the facade client and hide the implementation details.

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/patterns/structural/facade.rb)

#### Flyweight pattern

In computer programming, flyweight is a software design pattern. A flyweight is an object that minimizes memory use by sharing as much data as possible with other similar objects; it is a way to use objects in large numbers when a simple repeated representation would use an unacceptable amount of memory. Often some parts of the object state can be shared, and it is common practice to hold them in external data structures and pass them to the flyweight objects temporarily when they are used.

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/patterns/structural/flyweight.rb)

#### Proxy pattern

Protection Proxy: a protection proxy protects an object from unauthorized access. To ensure methods can only be run by authorized users, we can run an authorization check before messages are passed to the underlying object.

Remote Proxy: remote proxies provides access to objects which are running on remote machines. A great example of a remote proxy is Distributed Ruby (DRb), which allows Ruby programs to communicate over a network. With DRb, the client machines runs a proxy which handles all of the network communications behind the scenes.

Virtual Proxy: virtual proxies allow us to delay the creation of an object until it is absolutely necessary. This is useful when creating the object is computationaly expensive.

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/patterns/structural/proxy.rb)

#### Not covered patterns:

* [Annotated Callback] (http://c2.com/cgi/wiki?AnnotatedCallback)

* [Bridge] (https://en.wikipedia.org/wiki/Bridge_pattern)

* [Data Bus] (http://c2.com/cgi/wiki?DataBusPattern)

* [Role Object] (http://c2.com/cgi/wiki?RoleObjectPattern)

### Behavioral pattern 

In software engineering, behavioral design patterns are design patterns that identify common communication patterns between objects and realize these patterns. By doing so, these patterns increase flexibility in carrying out this communication.

[Read wiki] (https://en.wikipedia.org/wiki/Behavioral_pattern)

#### Chain of responsibility pattern

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/patterns/behavioral/chain_of_responsibility.rb)

#### Command pattern

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/patterns/behavioral/command.rb)

#### Interpreter patter

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/patterns/behavioral/interpreter.rb)

#### Iterator patter

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/patterns/behavioral/iterator.rb)

#### Mediator patter

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/patterns/behavioral/mediator.rb)

#### Momento pattern

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/patterns/behavioral/momento.rb)

#### Observer pattern

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/patterns/behavioral/observer.rb)

#### State pattern

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/patterns/behavioral/state.rb)

#### Strategy patter

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/patterns/behavioral/strategy.rb)

#### Template method pattern

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/patterns/behavioral/template_method.rb)

#### Visitor patter

[See example] (https://github.com/evncom/ruby.fundamental/blob/master/patterns/behavioral/visitor.rb)

#### Not covered patterns:

* [Hierarchical visitor] (http://c2.com/cgi/wiki?HierarchicalVisitorPattern)

####Code and articles were taken from resources:

* [https://ru.wikipedia.org/wiki/Design_Patterns] (https://ru.wikipedia.org/wiki/Design_Patterns)

* [https://en.wikipedia.org/wiki/Creational_pattern] (https://en.wikipedia.org/wiki/Creational_pattern)

* [https://en.wikipedia.org/wiki/Structural_pattern] (https://en.wikipedia.org/wiki/Structural_pattern)

* [https://en.wikipedia.org/wiki/Behavioral_pattern] (https://en.wikipedia.org/wiki/Behavioral_pattern)

* [http://c2.com/cgi/wiki?CategoryPattern] (http://c2.com/cgi/wiki?CategoryPattern)


