# Jacob Van Order Cocoa Design Standards

- [Cocoa Design Standards](#cocoa-design-standards)
  - [Introduction](#introduction)
  - [Large, Overarching Concepts](#large-overarching-concepts)
    - [Before You Start, Search](#before-you-start-search)
    - [Documentation](#documentation)
    - [Hacks and Work Arounds](#hacks-and-work-arounds)
    - [Feature Flags](#feature-flags)
    - [Testing](#testing)
    - [MVC](#mvc)
      - [Models](#models)
      - [Views](#views)
      - [Controllers](#controllers)
  - [Basics](#basics)
    - [Naming](#naming)
      - [Methods:](#methods)
        - [Delegate Methods:](#delegate-methods)
      - [Properties:](#properties)
      - [Classes:](#classes)
      - [Miscellaneous:](#miscellaneous)
    - [isEqual:](#isequal)
    - [Block Checking](#block-checking)
    - [Categories](#categories)
    - [`NSError`](#nserror)
    - [Checking Against `NSError`](#checking-against-nserror)
        - [Domain](#domain)
        - [Code](#code)
        - [User Info](#user-info)
    - [Localized String](#localized-string)
    - [Colors & Fonts](#colors-&-fonts)
    - [Images](#images)
    - [Stringly-Typed Values](#stringly-typed-values)
      - [Other Stringly-Typed cases](#other-stringly-typed-cases)
        - [NSStringFromSelector](#nsstringfromselector)
          - [NSPredicate](#nspredicate)
        - [NSStringFromClass](#nsstringfromclass)
    - [Private Class Extensions](#private-class-extensions)
    - [Immutable vs. Mutable](#immutable-vs-mutable)
  - [Advanced](#advanced)
    - [Singletons](#singletons)
    - [Delegate Pattern](#delegate-pattern)
        - [When to Use a Block](#when-to-use-a-block)
    - [A More Complicated Table/Collection View](#a-more-complicated-tablecollection-view)
        - [Example](#example)
    - [Sectioning View Controllers using Categories](#sectioning-view-controllers-using-categories)
    - [Network Interaction](#network-interaction)
        - [Keep the requests to an absolute minimum.](#keep-the-requests-to-an-absolute-minimum)
        - [Try not to retain state.](#try-not-to-retain-state)
        - [If you do have to make your own url, NSURLComponents are your friend](#if-you-do-have-to-make-your-own-url-nsurlcomponents-are-your-friend)
  - [In Conclusion…](#in-conclusion%E2%80%A6)

## Introduction

When leading a team of up to 9 iOS Developers, it became clear that certain rules needed to be placed in order to contribute to a harmonious code base. I wrote this document in order to give some guidelines based on my iOS Development experience stemming from 2010.

This document's purpose is to give a sense of what best practices can be kept in mind in order to keep all of the people working on it focused towards creating code that is consistent and cohesive. Ultimately, the goal is a seamless experience for the developer.

The code you write should value:

* Consistency with the existing code.
* Communicating as much **intent** as possible.
* Documentation to give insight to functionality as well as context.
* Simple external APIs with internal complexity kept to a minimum.
* Sticking as close as you can to Apple-approved MVC Cocoa patterns that might be described as “vanilla”.

## Large, Overarching Concepts

### Before You Start, Search

There you are. You have a ticket to create what UX has determined to be a wheel. You think to yourself, _“Excellent, I implemented one of those at my last job but I have a couple ideas on how to really do that rolling thing just a little bit different because I'm a reasonable human being who is constantly improving.”_.

**Stop right there.** Do a search for “Wheel” within the project and see what comes up. Odds are that it's been done. Twice. Hopefully, it's beautifully written prose that makes you weep with its clarity and impact. But if it's garbage, do everyone a favor by sitting down and actually looking at what is terrible about it. Write it down. Look at its public API and determine what can be improved upon. Write it down. Look at the logic involved and ask the BSA about when this was first implemented and what was the context? Is it still the same? Write it down.

At that point, determine if you'd save time by rewriting all of the existing business logic that works with your new functionality or if it'd be better to bolt on what you need with a note to come back and fix it. For more information, see: [Hacks and Work Arounds](#hacks-and-work-arounds).

### Documentation

We've all been there. You're working on something deep in thought when someone from another area of business comes over to quiz you about that esoteric feature you or someone you vaguely know worked on 8 months ago. Because you're the doof with Xcode open, you must be an expert on that so, pop quiz; what's the business logic around OMS and what happens if the Forwarder was zapped to outer space?

Luckily, the developer that previously worked on that feature did you a solid and clearly documented the crazy logic that's involved with this edge-case feature that you're being asked about.

**At a very minimum, header files should be documented.** Methods are mandatory, properties are optional if they go out of the norm (e.g., an array of mixed objects). If a feature is particularly confusing, requires a more than normal amount of client-side logic, or has required more than a normal amount of bug-fixes or development time, then at the very least please include the Epic ticket number. Preferred is a concise explanation of what the thinking was in the creation of the code. Remember, you're communicating to complete stranger so keep it short, express your intent, and do your best to explain the context.

### Hacks and Work Arounds

`//I know this is a dirty hack but…`

Duct tape is a wonderful tool for quick patches and emergency fixes but a solid foundation it does not make. Being pragmatic, there may be a need to get something fixed quickly or to overcome a problem that you don't quite understand. That being said, once you have one of these solutions that doesn't _feel_ right or match up with what this document outlines, then it's up to you do two things:

1. Get up and talk to your project facilitator to create a ticket to do technical analysis as to why this is happening and what you can do about it.
2. Make a comment with your initials so that you can find it at that point.

Don't be afraid if this is a symptom of a larger architectural change. Scope out the issue and make a case as to why it needs to be addressed during storytime. Odds are that your project manager will want the issue to be resolved and it never hurts to try.

### Feature Flags

If a feature is going to take more than *one* sprint to complete or has the possibility of being halfway done while candidate could be cut, you need to hide the feature behind a feature flag. This is to say that in the project's constants file, you'll put something along the lines of `#define kUsingNewGreatFeature  1`. Within the code, you will delineate the old feature from the new feature with a preprocessor macro, e.g.:

```(objective-c)
#if kUsingNewGreatFeature
//Your new code
#else
//Your old code
#endif
```

Files that need to be deleted should have a comment with a `TODO:` combined with the constant for easily being able to search and then delete unneeded files, e.g., `//TODO: delete after kUsingNewGreatFeature is enabled permanently.`

### Testing

In combination with QA, code review, and assertions, testing is another tool in our toolbox towards achieving  fidelity and stability in our code. Some may treat testing as a panacea but, again, it's another tool in our toolbox.

As a baseline,  you should be testing Model objects. Not only with their initialization in order to check our mapping but also any methods or properties that are attached to the Model object. For instance, let's say you have a `Person` class with `firstName` and `lastName` coming from the server. We'd check that those were parsed correctly but also a convenience method that we'd make for `fullName` that combines the two with a space in between. Not only would it check for, let's say, “John Smith” but also “Testy Von Testerson” in order to make sure that last name isn't truncating something or losing data.

What **shouldn't** you test? Testing `Bool`s or wasting your time with setting properties. Again, think of something that can broken by a side effect, state change, or unforeseen scenario. Odds are that `@synthesize` doesn't need any testing around it.

### MVC

There are many [Software Architectural Patterns](https://en.wikipedia.org/wiki/Architectural_pattern) but Model-View-Controller is the one sanctioned by [Apple](https://developer.apple.com/library/ios/documentation/General/Conceptual/DevPedia-CocoaCore/MVC.html). That last link gives a good overview but here is how you should think about it in a high-level:

#### Models

are the object that largely hold data (usually from the server). Usually, this comes from the API in the form of json payload which we convert into Cocoa objects via an Object Mapping service.

#### Views

are how that data is presented visually. This includes `UILabel`, `UIButton`, `UITableViewCell`, etc… They should not hold state or reference to the Model.

#### Controllers

are the logic classes that take the models and determine how to slot them into which views and when.  They also handle interaction with the View and modify the Model accordingly.

As with any architectural pattern, there are sometimes when not every scenario can be covered by those three structures. What would a network call be under? Doesn't that lead to huge View Controllers? How about instances like `UIImageView` which holds a reference to it's `UIImage`?

You're right, there are instances where the pattern isn't perfect but it's the most conventional one we have. When you're in doubt or if you need clarification, here's some rules of thumb:

**_A view should not have a reference to its model but can be loaded by it._**

This means that a `UITableViewCell` can have a method like `- (void)readModelObject:(JVOModelObject *)modelObject;` but not store that Model object in a property.

**_A Model object, conversely, should not know about views._**

If you were to set up a Model object that parses an array of objects, sorts them by some criteria, and distributes them out at the right index path. That sounds like it would be a great `dataSource` for something like a `UITableView`, right? The only problem is that `UITableViewDataSource` has that method `- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath` which is tasked with the creation of a cell. Is that testable? Kind of. Is it dependent on the View class? Most definitely. Those two facts kind of throw off my smell tests so just keep it in the controller. For more information see: [A More Complicated Table/Collection View](#a-more-complicated-tablecollection-view).

**_If you are not able to swap out a component, step back and reconsider._**

You could make the case that a `UITableViewCell` subclass' method like `- (void)readModelObject:(JVOModelObject *)modelObject;` makes it dependent on `JVOModelObject`. It does but it doesn't have to be the only method that reads a Model object. You could have a method that reads `JVOOtherModelObject` or `JVOSuperModelObject`. Ultimately, the goal here is flexibility and the ability to easily maintain code. Use your best judgement.

## Basics

### Naming

Before you start this section, please read [Apple's Cocoa Conventions](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Conventions/Conventions.html)

Got it?

Objective-C, Swift, and Cocoa have their own prefixes that are not allowed to be used by properties, methods, and internal implementations. For instance, Swift has it's list [here](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/LexicalStructure.html) under “Keywords and Punctuation”. [Here's](https://www.binpress.com/tutorial/objective-c-reserved-keywords/43) Objective-C. Lastly, [Cocoa](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CodingGuidelines/Articles/NamingMethods.html).

So what's a developer supposed to use? A rule of thumb is to consider the following:

#### Methods:

For network calls, use [HTTP request method](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#Request_methods) verbs (Get, Post, Put, Delete) e.g.:

* `- (void)getShippingAddressesWithCallback:(void (^)())callback;`

For everything else, _try_ to use [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) verbs (Read, Update, Delete) e.g.

* `- (void)readModelObject:(JVOModelObject *)modelObject;`

Notice how “Create” isn't in there? `alloc` and `init` largely take that over so please follow convention.

##### Delegate Methods:

For Delegate methods, follow [convention](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CodingGuidelines/Articles/NamingMethods.html#//apple_ref/doc/uid/20001282-1001803-BCIDAIJE) by starting the method with the sending object and “did”, “will”, or “should”, e.g.:

* `- (void)someObject:(SomeObject *)object didUpdateSomethingWithOtherObject:(OtherObject *)otherObject;`
* `- (void)someObjectDidUpdateSomething:(SomeObject *)object;`  

#### Properties:

Indicate what it stores, what its purpose is, how it's used, etc…

_Good:_

* `shippingAddresses`
* `isShowingDefaultLabel`
* `canBeAltered`

_Bad:_

* `shipAddr`
* `defaultLabelIsShowing`
* `alterable`

#### Classes:

For **Models**, it should be the real life object that it represents (e.g. `ShippingAddress`). **Controllers** and **Views** should be around what feature it represents combined with what its superclass is (e.g. `JVOHybridCatalogCollectionViewController`).

#### Miscellaneous:

* A block that's used for clean up after an asynchronous task is called a callback e.g. `- (void)getShippingAddressesWithCallback:(void (^)())callback;`
* Constants start with `JVO` followed by a descriptive name e.g. `JVOAddressFieldLineHeight`

### isEqual:

This seems obvious but you should not be using `==` for checking equality of objects. Instead, use the `isEqual:` or variants for each class (e.g.: `isEqualToString:`, `isEqualToNumber:`, etc…)

Yes, `==` compares the two objects' pointers which is usually what `isEqual:` checks but there might be special implementations for that class that you're not using.

For more information: [NSHipster](http://nshipster.com/equality/) has a great overview.

### Block Checking

If a block is `nil` when invoked, the app will crash. Until we transition to Swift, we can start using [Nullability Annotations](https://developer.apple.com/swift/blog/?id=25) to help make sure that a block passed is not nil but at this point it will only give a warning.

In order to resolve this, a parameter assertion should be used at the beginning of the method where a block could be used, e.g.:
```
- (void)someMethodWithCallback:(void (^)())callback {
    NSParameterAssert(callback, @"The call back needs to not be nil")
    …
    callback();
}
```

### Categories

In order to communicate that this is a app-specific change to an existing class, prefix any category methods with the same prefix you use for classes but lowercase, e.g., `JVO` -> `jvo_`.

### `NSError`

If you need an overview of what `NSError` is or what the pieces of it are, be sure to read the excellent overview from [NSHipster](http://nshipster.com/nserror/).

### Checking Against `NSError`

While we are talking about it, `NSError` in Objective-C is usually implemented as an inout argument in a method. As a result, it can be tempting to use that as the determining factor whether the task was successful. The problem is that since the argument is inout, you have no guarantee that the pointer to a pointer was written to mistakenly or that the memory was scribbled on. As Mark Dalrymple states in his [excellent post](https://www.bignerdranch.com/blog/an-nserror-error/), “You can think of the error details as out-of-band data, a secondary information stream coming from the method.”

Instead check the expected return value, e.g.:

```
NSError *error = nil;
BOOL success = [someMethodThatReturnsBoolWithError:&error];
if (!success) {
    NSLog(@"Error: %@", error);
}
```

Even if you're not using a pointer to a pointer (`*` `*`) implementation, you should still test for the desired outcome instead of the presence of an error.

##### Domain

This should be a reverse-domain url of `com.sushiGrass.<#App Name#>`.  This should be defined as a `FOUNDATION_EXPORT`ed string constant in the `JVOErrorCodes.h` and `.m` files.

##### Code

The projects should have a file called `JVOErrorCodes.h` that has an enum with the codes with brief but descriptive values.

```
typedef NS_ENUM(NSInteger, JVOErrorCode) {
    JVOErrorSomethingBadHappened = 1,
    JVOErrorSomethingElseHappened,
    …
};
```

##### User Info

In Cocoa, User Info is a dictionary object that acts as the arbitrary storage object that you can throw anything into. As NSHipster puts it, “As a convention throughout Cocoa, userInfo is a dictionary that contains arbitrary key-value pairs that, whether for reasons of subclassing or schematic sparsity, are not suited to full-fledged properties in and of themselves.”

Try to insert as much information as you can into these dictionaries including any objects that might be pertinent to solving why the issue is happening.

If possible, include more information using `NSLocalizedDescriptionKey`, `NSLocalizedFailureReasonErrorKey`, `NSLocalizedRecoverySuggestionErrorKey` to give your fellow developer a hand in trying to figure out why the issue is happening at all.

### Localized String

Although not yet implemented, we should be shooting for the day where we **can** export the strings for a translator to parse. There should be an category on `NSString` with a class method for that string.

In `NSString+Localization.h`:

```
+ (NSString *)jvo_okay;
```

In `NSString+Localization.m`:

```
+ (NSString *)jvo_okay {
    return NSLocalizedString(@"Okay", @"The text for Okay");
}
```

### Colors & Fonts

Similarly, colors and fonts should also have a category made for `UIColor` and `UIFont` respectively that has app-specific class methods for styles we are using, e.g.:

`[UIColor jvo_redDestructiveColor];`
`[UIFont jvo_systemFontOfSize:16];`

### Images

In the same vein, `UIImage` should have a category with class methods for images that are used within the app, e.g.:

 In `UIImage+Images.h`:

```
+ (UIImage *)jvo_catPerson;
```

In `UIImage+Images.m`:

```
+ (UIImage *)jvo_cartPerson {
    return [UIImage imageNamed:@"catPerson"];
}
```

### Stringly-Typed Values

One of the larger weaknesses of some APIs in Cocoa is that they rely on [Stringly-Typed](http://c2.com/cgi/wiki?StringlyTyped) values.

I'm going to refer to the solution to this as “structly-typed”.

If there is a case where only one file will be using this structly-typed value, let's say a `UITableViewController` subclass that needs to register a `UITableViewCell` class and then dequeue it, then put this struct in the `.h` and `.m` file for that `UITableViewController`.

If there's a case where the structly-typed value needs to be shared across files, let's say a `NSNotification` name or even a `UITableViewCell` that's used in more than one `UITableViewController` subclass, then each project should have an `.h` and `.m` file for each object outlined below that resembles something like this (in this case for `UITableViewCell` identifiers).

In `IdentifiersForTableViewCells.h`:

```
FOUNDATION_EXPORT const struct JVOSomeIdentifierForACertainTableView {
    __unsafe_unretained NSString *descriptiveTitleCellOne;
    __unsafe_unretained NSString *descriptiveTitleCellTwo;
} JVOSomeIdentifierForACertainTableView;
```

In `IdentifiersForTableViewCells.m`:

```
const struct JVOSomeIdentifierForACertainTableView JVOSomeIdentifierForACertainTableView = {
    .descriptiveTitleCellOne = @"descriptiveTitleCellOne",
    .descriptiveTitleCellTwo = @"descriptiveTitleCellTwo",
};
```

That way, when you need to call `- (id)dequeueReusableCellWithIdentifier:(NSString *)identifier`, it will look like this: `[tableView dequeueReusableCellWithIdentifier:JVOSomeIdentifierForACertainTableView.descriptiveTitleCellOne]`. Verbose? Yes. Sorry.

These files can have many structs that are grouped in ways that make sense, e.g., `NSNotification` names around logging in or out. There shouldn't be one really big struct with everything in it.

As to how you name the elements, they will vary slightly from type to type:

* **Cell identifiers**: What the parent TableView is and what kind of cell it is, e.g.:
  - `personCell`
  - `productCell`
  - `locationCell`

* **Segues**: What the destination view controller is and how it is being presented, e.g.:
  - `trainMapViewControllerEmbed`
  - `videoGameControllerViewControllerPresent`
  - `clockSettingsViewControllerPush`

* **Storyboard Files**: Pretty straight-forward. Be descriptive of what feature within which tab the Storyboard is part of.
* `NSNotification` name: Be clear as to when this notification is being called. e.g., `userDidLogin`, `omsCartWasProcessed`, `straightShooterWasIdentified`.

#### Other Stringly-Typed cases

##### NSStringFromSelector

Instead of typing that selector which could be incorrect or possibly be missing a colon, use this instead.  **Note!** This doesn't cover the Refactoring feature of Xcode which is to say that the selector name you type in will not be picked up by the tool but odds that you misspell it are less.

###### NSPredicate

It's tempting to have `[NSPredicate predicateWithFormat:@"someProperty == YES"]` and be done with it. But, again, that's a string that could have issues with the way it's typed. Instead, try to use that format you're given to be `[NSPredicate predicateWithFormat:@"%K == %@", NSStringFromSelector(@selector(someProperty)), @YES]`.

##### NSStringFromClass

Same kind of thing here. Instead of stringly-typing a class, use `NSStringFromClass` to allow for changes down the road. The Refactor tool does pick this up.

### Private Class Extensions

When thinking about a public API for your class, think about the public interface in the `.h` file being the absolute-must-know details of the code you wrote. The rest can be kept as private interface within the `.m` file's class extension.

But what if you want to subclass? Instead of moving everything over to the `.h` file which would lessen the impact of the public API or importing an `.m` file which would cause the app to not compile, make a new file with the class extension in it.

Name the file as the file it is extending but with `+Private.h` at the end. From there, move your class extension at the top of your `.m` file to this new file and work from there. Don't forget to import this file in your `.m` file on your class and all subclasses.

### Immutable vs. Mutable

Generally, it is better to keep container properties held as immutable objects in order to prevent undesired side effects, changes, or weird race conditions. Even though they are quite convenient, mutable objects are harder to debug issues with when you can add an object arbitrarily from anywhere where as a mutable object must be set when it is a property. And even though mutable containers are usually subclasses of the immutable version, keep your return types on methods consistent to what you are actually returning. That is to say that if you are declaring `- (NSArray *)someArray;` do not actually return an NSMutableArray.

## Advanced

These are the areas where we might swing away from convention slightly.

### Singletons

> Singletons need to die in a fire.
> ~ [Saul Mora](https://alpha.app.net/saulmora/post/1148057)

> Nothing is absolute. Everything changes, everything moves, everything revolves, everything flies and goes away.
> ~ Frida Kahlo

Nothing can inspire a good ol' fashioned debate like Singletons. But like anything, there is gray area when it comes to it's validity. If anything, it's just another tool in your [toolbox](https://en.wikipedia.org/wiki/Design_Patterns#Creational). People who advocate against the pattern end up making an instance of something (let's say a networking class) and attach them onto `UIApplicationDelegate` which is a, wait for it, Singleton. If not, then that networking class uses `[NSURLSession sharedSession]` which is a… well, you know.

Here's the rule of thumb: try to avoid it. If it doesn't need to retain state in a property then a Class with Class methods can fit the bill. If it does need to retain state, then see if having multiple instances of it would be detrimental. If those criteria are met, then yes, use a Singleton. Just don't tell Saul Mora.

For even greater detail see [this excellent post](https://sandofsky.com/blog/singletons.html)

### Delegate Pattern

Probably the most used software pattern in Cocoa is the [Delegate Pattern](https://en.wikipedia.org/wiki/Delegation_pattern).

If an object, let's call it a “Parent”, holds reference to another object (“Child”), it can directly send messages to the public methods of the child. The child, though, should not be able to send messages back up to the parent unless it is through delegate methods.

##### When to Use a Block

Largely, the main reason to use a block instead of delegation is when asynchronous network calls are being made by the child object. The other exception is for animations and their completion.  

### A More Complicated Table/Collection View

Some say that “MVC” doesn't stand for Model-View-Controller but instead “Massive View Controller” due the fact that Models and Views tend to be relatively light where as most of the logic, setup, and interactive handling tend to be in the View Controller.

This is especially true in `UITableViewController` and `UICollectionViewController` subclasses. Not only do you have to deal with the normal `UIViewController` setup and loading of data but also the delegate and datasource methods from the `UITableView` and `UICollectionView`. To make matters worse, what we get from the API in the order, style, configuration that we receive it isn't exactly what Product or UX might want.

This is where creating an object that handles that reordering and configuration of the Model objects comes in. For the sake of convention, this object should be named with `DataSource` and primarily be tasked with the following responsibilities:

* Taking in a collection or collections of Model objects.
* Reordering, transforming, distributing, or parsing Model objects.
* Responding to the number of sections and number of items per section.
* Vending the Model object in the correct order that corresponds to the index path requested by the `UITableView` or `UICollectionView`.

All of these responsibilities should be part of the object's API and anything else should be moved to private implementation details.

The benefit is that all of these responsibilities are able to be Unit Tested, reused at another location if needed, and easily swapped out.

What it **should not be doing** is:

* Touching the `UITableViewCell`s or knowing anything about what kind of cells will be displaying these Model Objects.
* Handling any user interaction.
* Be set as the `delegate` or `dataSource` of the `UITableView` directly.

This is due to the fact that we start getting into view lifecycle or setup territory which complicates Unit Tests as view state is involved. Plus, we might not even _want_ to display in a `UITableView`. We could have a comma separated list of all of the nicknames. This still works and allows for that flexibility. Just don't call it a ViewModel.

##### Example

Let's say that from the API, we get an array of Addresses. These are in the order closest to the user's location. UX wants a User Interface where a `UITableView` shows Addresses ordered with the Address marked “default” first in it's own section, then the Address marked “current” second in it's own section. The remaining Addresses should be split into sections based on States within the United States. Repeat for Mexico and Canada. Those sections should be alphabetically sorted. The remaining Addresses should be split into sections based on which country they are in with those sections also alphabetically sorted.

That right there is a whole bunch of Unit Testable logic that would be tied to a View Controller's view lifecycle if it happened during `- (void)viewDidLoad;`. Instead throw all of that code into a Data Source object. We Unit Test it under different scenarios and rest assured that when the `UITableViewController` subclass calls `[self.addressDataSource modelObjectForIndexPath:indexPath]` that it will all be in order.

### Sectioning View Controllers using Categories

Another tactic for making massive View Controllers more managable is breaking features of that View Controller into Extensions. This way, a feature can be added and subtracted with little entanglement with other features.

There is one major consideration in that properties can only be declared on the main `.h` and `.m` file. If you do need to add properties to the View Controller, try to add it to the private category extension as outlined in [Private Class Extensions](#private-class-extensions).

### Network Interaction

So much of what we do on our apps is go get information from the server, have the user interact with it in some way, and record that change to the server.

There are a couple concepts that are important to keep in mind concerning these interactions.

##### Keep the requests to an absolute minimum.

Mobile devices generally are using cellular data that has improved in recent years but can still have a good deal of latency. If the API is lacking information in one area and the only solution is to do yet another network call, stop and ask yourself why it wasn't included in the original request.

##### Try not to retain state.

A good API should do a good job keeping state and although it's tempting to add another property that indicates what the last action might have been, it more often than not causes issues where it's out of sync with what the service representation is. Before you add another property of the last request or navigation path, look for that information in your current request. If not found, ask to see if what you're looking for could be added to the response. Only as a last resort should another property be defined. Keep in mind that our View Controllers should try to be entered and left with as little context as possible.

##### If you do have to make your own URL, NSURLComponents are your friend

Instead of using a `[NSString stringWithFormat:…];` method to make your URL string, consider using [NSURLComponents](https://developer.apple.com/library/prerelease/ios/documentation/Foundation/Reference/NSURLComponents_class/index.html). This class was introduced in iOS 8 and allows you to construct a URL in a less error prone way, allows for conditional elements of the URL, and handles throwing it all together for you.

## In Conclusion…

That was a lot of information, I know. Review it, discuss it, and try to keep it in mind while writing your code. Overall, remember:

* Consistency
* Communicating Intent
* Documentation
* Simplicity
* Minimizing Human-Error

This document is a living piece of documentation. If something doesn't sit right or if it doesn't work well in practice, bring it up, and it will be evaluated.
