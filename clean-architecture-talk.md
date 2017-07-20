Script for my Talk

Sides:

# Who am me
The who am I slide, or as I like to call it, the "why should you listen to me" slide.
Well, my name is Barry O Sullivan, I'm the lead developer and solutions architecture for DynamicReservations.
The focus of my job is removing developer friction, I make it easy for our developers to do their job. Everytime we encounter bugs, have to spend time working around technical debt, or even have friction between teams, my job is to figure out the source and architect a solution, be it software or process related. I've been designing software solutions for years and I employ an aggressive restructuring policy, if it causes consistent pain, deal with it. In my mind clarity and consistency are king, especially if you're working as part of a team. This is the lense through which I analyse and design our architecture. 


# What are we going to talk about
- The status quo of software design (focussed on web dev)
- Why it's difficult to write clean code
- An architectural pattern that is scaffolding for writing clean code
- A practical example of how you can refactor/restructure code to be cleaner


# The status Quo
[Picture of MVC and some popular frameworks, possible showing some mess in code]

Most of us know MVC, it's the foundation for pretty much every web framework. This is funny and strange, because MVC is not a system architectural pattern, it's a user interface pattern, so as soon as your business logic gets complex, MVC starts to fall apart. Even a relatively simple product can end up with a bloated and messy codebase. MVC is where we start, but what do you do when you need to evolve past it? And how do you do that?

## The problem 
Here's a common conversation (for developers anyway)

[Slide of talking heads]

devA: "Our codebase is really messy, how do we clean it up?"
devB: "We need to refactor it, move code into objects, separate the concerns."
devA: "Ok, great. How do we do that?"
devB: "We'll use design patterns and follow the SOLID principles."

This is where we normally stop, we seem to think that's a good enough answer for someone to get started. 

[Pciture of carpenter pointing at his tools]
It's like you asked a carpenter how to make a table, and he just points at his tools and says "use those". He doesn't show you his blueprints, or even discuss how he came up with them, he just hands you a saw and says "use this". The answer is technically correct, but it doesn't tell the whole story and it definitely isn't useful to someone learning to write software (or make tables). The tools are only a small part of the process.

## Learning the patterns isn't enough
[Design patterns dotted around the slide]
This is the part that's the most infuriating, learning the patterns isn't enough. Infact, you're probably in a worse place immediately after learning them, because suddenly you have power tools and no clue where to use then.

Learning the patterns opens up a whole new range of questions you have to answer before you can use them effectively.
- Where should I use design patterns?
- How do I decide which one to use?
- Where do I draw my lines of abstraction?
- When should I use interfaces?

[Picture of the wong pattern being used, and a red X]
Without some guiding principles on how to answer these questions, developers end up creating random boundaries between objects, using patterns wherever they can. This leads to inconsistent code that's worse than it was without all this "design".

## What is design? 
[Thinking head picture, desing keyword] 
What is design? The key point of design is often overlooked. When you design something, you're designing it to solve a specific problem, at the expense of other problems. It's a compromise. If you can't nail down the problem you're trying to solve, then no amount of "design" wil help. 


# Where to start
[Clean architecture picture]

Now I've thouroughly depressed you, how do you possible move past this? Well, for me, the moment it all clicked was when I learned about clean architectures.

A clean architecture is a guide to splitting up your code acros layers, giving direction on where certain concepts belong. There are many flavours but they each share the same core concepts.

- Separate your code by layers
- Put abstractions in the inner layers
- Put implementation details in the outer layers
- Dependencies can only point inward, never outward.

# Why is it structured this way?
So this is a probably a lot to take in, so I'll quickly try to explain why it's structured this way. 

The key to this pattern is change management and how changes in one place affect other places. In standard software, everything is coupled together, a change in one place requires a change in many places. This is what we mean by tight coupling. This is where software patterns come in, they're there to let us decouple our code, so that different parts of our code can change at different rates. 

Now, the clean architecture emerged from analysisng the type of changes that happen in business applications. Effectively they categorised code blocks by responsibilities, and look at how each one changed, depending on common changes to the system. 

They noticed that changes to the core (business logic) forced a change in pretty much every other aspect of the system. This makes sense, as you've changed something fundamental. They also noticed that changes to a controller, such as adding a new endpoint, really shouldn't affect the busines logic, if it does, this is sign that you've got some bad coupling. 

So this is why this type of architecture was created, to give us a framework for structuring and decoupling our code, based on the types of changes we encounter.

## Change 1
[Show layers, what is affected]
Think of it this way. A change in your core business rules, eg. A user's email address must be unqiue, requires a change in nearly all the layers, as it's a change to the core, everything else points at it. 

## Change 2
[Show layers, what is affected]
Let's take another example. A new usecases comes in, a user can now update their email address via text message. There are no changes to business logic, so that stays the same, we don't need to touch it. We only need to add a new usecase that uses existing functionality, and add a controller that uses it. Done. 

## Change 3
[Show layers, what is affected]
However, a change to implementation, eg. emails address are cached in memory for quicker lookup, should not require a change to any of the other layers. The business logic hasn't changed, the controllers haven't changed, only the implemnetation has, so it should be a localised change. 

## Eye opening
[Picture of mental clarity]
This is why this pattern is so great. It gives guidelines on how to separate your concerns and where to draw your boundaries. Once the boundaries are drawn, the patterns become obvious. It's great tool for separating the wheat from the chaff.

# A practical example
Ok, that's enough background and overview, it's time for the main event. How to apply this on real code.


# Code sample
[Code sample] 
Here we have a simple usecase for setting a user's profile image. This class is used internally by both a controller and a console command, so the code is already pretty condensed.

What's wrong with the above? Well, despite it's condensed nature, it actually has six concepts intermingled, muddying the water.

## Concepts
[Code with words highlighted by concept] 

- Domain
- Application
- Config
- AWS S3
- Flysystem
- Eloquent ORM

You have to understand each of those concepts in order to understand what the class is trying to do internally. That's a surprising numbers of concepts you need to keep in your head in order to understand such a simple class.

To make matters worse, they're not consistent. They each describe their solution in their own language, using details and concepts that do not mix well.

For example, ORM language such as find, where and update don't exist in the Flysystem language. Throw domain and application concepts into the mix and you can see why things can get confusing.

## Book
[Picture of a confusing book] 

It's like trying to read a book where it switches between French, English and Russian at random points, sometimes from word to word. Sure, eventually you could read it, but you'd make a lot of mistakes along the way and you'd be a frustrated mess at the end. So let's try to clean things up.

# Dividing the Layers
First we need to know what we're removing from the application layer, so let's use language as our guide.

Since we want the language to be application focussed (the problem being, our application is confusing, and this is the application layer), we need to remove any words unique to the following concepts.

- Config
- Flysystem
- AWS S3
- Eloquent ORM

Instead, we'll replace them with integration points that use the language of the application, rather than the implementation.

This is where design patterns come in. By looking at what you're trying to do and not how you're trying to do it, you can spot common patterns emerging.

## Extracting the concepts
[Image of separation of concerns] 

Looking at the code, image and user are core/domain concepts respectively, but some of the code that uses them is pure implementation. These are our integration points, so let's drill down into what we're really doing with "images" and "users".

In our example it's clear that we really want to do two things

store and retrieve users
store images
Everything else is an implementation detail. Storing and retrieving things is clearly the repository pattern. So let's create two new application concepts, UserRepository and ImageRepository, and we'll implement them as interfaces.

## Application Layer

[Example of clean usecase]
[Example of interfaces]

That's the application level restructured. We've refined the language and concepts down to the bare minimum that the application needs, expressing our intent. We also created two new concepts that were hidden in the old implementation and we've boiled them down a simple pattern, the repository pattern.

As an aside, we have no problem adding design pattern language to the application layer, because it's a shared language between developers, it aids clarity rather than obscuring it.

## Infrastructure Layer
[S3ImageRepository]
[EloquentUserRepostiory] 

And that's that, we've separated our usecase's code across two layers, application and infrastructure.

# Why is this better?
[Show bullet points]

### Easier to read
From a language perspective, this is much cleaner. It's focussed on the language of what our application wants to do, not how it intends to do it. This cohesive language lowers the barrier to comprehension. In short, there are fewer concepts to parse in order to understand it.

### Testing
It's far easier to test. The original has so much going on that testing it is difficult. Our acceptance tests would have to connect to S3 and the DB to prove that the code worked. This would couple our tests to our implementations, making the code brittle and expensive to change.
In the new version, each layer can be tested in isolation, with unit tests for our usecases and integration tests for our implementations. This makes our acceptance tests much easier to verify, we only need to check the output, rather than the side effects.

### We're using patterns effectively
In the above, we used the repository pattern in a way that made sense to the application, rather than using whatever pattern we liked at the time. This keeps things focussed and stops us going design pattern mad. After failing with design patterns for so long, it feels really nice to use them effectively.

### Loose coupling
The above is actually a SOLID*** example of the dependency inversion principle in action. We've inverted the dependencies of our code, ie. the application is no longer dependant on the database, instead it is the database that depends on the application. Simply put, this means that a change in the database will no longer affects the business logic, so there's less risk in changing it.

### Changing implementation
Another benefit, switching implementations is incredibly easy. Imagine we needed to switch storage mechanism, say from MySQL to MongoDB. Instead of gutting our usecase, we just create a new implementation of that interface with the MongoDB lib under the hood. Boom, we're done. No changes required at the application level.
This example highlights how useful language can be as a guiding hand when writing layered code. The newly introduced language of mongodb did not bleed into the language of the application layer, it stayed the same, no change was necessary. This is a sure sign that we've separated our layers correctly.

# Why is this worse?
[Example of funny class, bullet points for the issues]

Let's be honest, it's not all sunshine and roses, so let's address the const ELEPHANT = true; in the class Room;
For a start the codebase is larger, there are now more files and there is more code to read. It also takes longer to write. You have to spend time extracting the code into layers, taking the time to think about the design of your code, so there is an element of sacrifice there. This could get in the way if it's throwaway software that you just want to get out the door and never see again****.

I don't think the above are reasons enough not to do it though. Software changes, especially if it's core to a product, so it's worth the extra time now to save a lot of time later.

[Show slide of getting faster]

Plus, you get better at it the more you do it. Eventually you'll notice the same patterns again and again, and you'll be able to create the abstractions at the beginning, intuitively separating what you want to do from the tools you're using to do it.

# When to do it
The above raises an important point, this approach to separation of concerns is a technique to use when needed, sometime you don't need it. Above we made a judgement call on what to extract into it's own layer and what to make part of the core. There are other guidelines, but really it boils down to clarity.

If you find things getting confusing in a layer, consider moving code/concept into another layer, or creating the layer if needs be.

# Conclusion
[overview of what we did]
So in this talk we covered 
- why software design is hard
- why the patterns are enough
- Showed how the 
- A practical example showcausing the idea

To finish, writing clean code is hard, and using patterns haphazardly just makes things worse. That's why I recommend applying the clean architecture mindset, it's a simple technique for cleaning up your code. It helped me, and I hope it can help you. Thank you.

# Q&A
[Standard Q&A slide]




