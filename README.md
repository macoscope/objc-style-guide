# Macoscope Objective-C Style Guide

This style guide describes a set of basic rules that we follow here at [Macoscope](http://macoscope.com). We strive to write better code every day, and we believe this document outlines the best practices that will help you do the same.

This document is based on, and inspired by, the awesome [NYTimes Objective-C Style Guide](https://github.com/NYTimes/objective-c-style-guide/).

Also, [we're hiring](http://macoscope.com/objective-c-developer).

## Mandatory Reading

If you haven't read the [Coding Guidelines for Cocoa](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html) yet, it's probably a good idea to stop right here and come back only after you're finished with it. You should pay special attention to:

* [Method naming](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CodingGuidelines/Articles/NamingMethods.html)
* [Property & type naming](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CodingGuidelines/Articles/NamingIvarsAndTypes.html)
* [Allowed abbreviations](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CodingGuidelines/Articles/APIAbbreviations.html#//apple_ref/doc/uid/20001285-BCIHCGAE)


## Table of Contents

* [Dot-Notation Syntax](#dot-notation-syntax)
* [Spacing](#spacing)
  * [Ternary Operator](#ternary-operator)
* [Error handling](#error-handling)
* [Methods](#methods)
* [Variables](#variables)
  * [Variable Qualifiers](#variable-qualifiers)
  * [Property attributes](#property-attributes)
* [Naming](#naming)
* [Comments](#comments)
* [Init and Dealloc](#init-and-dealloc)
* [Literals](#literals)
* [Categories](#categories)
* [CGRect Functions](#cgrect-functions)
* [Constants](#constants)
* [Enumerated Types](#enumerated-types)
* [Bitmasks](#bitmasks)
* [Private Properties](#private-properties)
  * [Custom getters](#custom-getters)
  * [Subclassing and Testing](#subclassing-and-testing)
* [Booleans](#booleans)
* [Singletons](#singletons)
* [Xcode Project](#xcode-project)
* [Refactoring](#refactoring)
* [Miscellaneous](#miscellaneous)

## Dot-Notation Syntax

Property? Dot-notation. **Always**.

Not a property? Brackets.

If Apple brings their frameworks up to date with modern ObjC syntax (looking at you, `NSArray.count`!), feel free to use dot-notation when targeting earlier releases.

**For example:**
```objc
array.count;
[UIApplication sharedApplication].delegate;
```

**Not:**
```objc
[array count];
UIApplication.sharedApplication.delegate;
```

## Spacing

* 4 spaces. Soft tabs. Period.
  * When working with older projects that used *wrong* number of spaces, adapt to the existing style. **Do not** convert existing projects to new guidelines or you'll break `git blame`.
* [1TBS](http://en.wikipedia.org/wiki/Indent_style#Variant:_1TBS) for control statements.
* [K&R](http://en.wikipedia.org/wiki/Indent_style#K.26R_style) for method declaration. (Opening brace appears in the following line.)
* **Never** omit braces in control statements, `goto fail;` should be a convincing enough argument for everyone.

**For example:**
```objc
- (void)describeCodingStyleWithCodingStyle:(MCSCodingStyle)style
{
    if (MCSCodingStyleKAndR == style) {
        [self doSomething];
    }
}
```
* Separate methods with **one** empty line. Use whitespace in methods as needed, but if you find yourself separating multiple groups of statements, there's a good chance they should be split into separate methods.

### Ternary Operator

If you're writing (or, for that matter, reading) multiple nested ternary operators, it means that something, somewhere went wrong. Write it in a different way (`if` statement, instance variable) and refactor existing occurrences.

**For example:**
```objc
goodThing = a > b ? x : y;
```

**Not:**
```objc
badThing = a > b ? x = c > d ? c : d : y;
```

## Error handling

If you haven't already, you should probably read @mattt's [excellent write-up of `NSError`](http://nshipster.com/nserror/) on [NSHipster](http://nshipster.com/).

TL;DR version: Check for returned value, not error variable.

**For example:**
```objc
NSError *error;
if (![self magicalOperationThatCanResultInError:&error]) {
    // whoops. Better handle this!
}
```

**Not:**
```objc
NSError *error;
[self magicalOperationThatCanResultInError:&error];
if (error) {
    // whoops. Better handle this!
}
```

But really, read that NSHipster article.

## Methods

Follow Apple API examples and docs. A space between `+`/`-` and method name, a space between method segments, a space between type and asterisk. No spaces anywhere else.

**For example:**
```objc
- (BOOL)tableView:(UITableView *)tableView canMoveRowAtIndexPath:(NSIndexPath *)indexPath;
```
**Not:**
```objc
- (BOOL) tableView: (UITableView *) tableView canMoveRowAtIndexPath: (NSIndexPath *) indexPath;
```

Multiargument methods like new lines between arguments, colon-aligned.

**For example:**

```objc
- (void)thisViewController:(ThisViewController *)thisViewController
        didFinishWithThing:(Thing *)thing
             somethingElse:(SomethingElse *)somethingElse
                     style:(Style)style;

- (void)prepareCakeWithIngredients:(NSArray *)ingredients
                           success:(void(^)(Cake *deliciousCake))success
                           failure:(void(^)(NSError *))failure;
```



## Variables

> Make things as simple as possible, but not simpler.

If you're unsure which side you fall on, it's better to err on the side of wordiness.

There shouldn't be any reason to use one-letter variable names apart from `for()` loops.

Asterisks hug the variable name, not the type.

**For example:**
```objc
NSString *macoscopeString;
```
**Not:**
```objc
NSString* macoscopeString;
```

Use properties over naked instance variables.

Generally avoid directly accessing instance variables, except in initializer methods (`init`, `initWithCoder:`, etc…), `dealloc` methods and within custom setters and getters. See [Advanced Memory Management Programming Guide](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmPractical.html#//apple_ref/doc/uid/TP40004447-SW6) for rationale.

**For example:**

```objc
@interface MCSObject: NSObject

@property (nonatomic, strong) NSString *title;

@end
```

**Not:**

```objc
@interface MCSObject : NSObject {
    NSString *title;
}
```

### Variable Qualifiers

When it comes to the variable qualifiers [introduced with ARC](https://developer.apple.com/library/ios/releasenotes/objectivec/rn-transitioningtoarc/Introduction/Introduction.html#//apple_ref/doc/uid/TP40011226-CH1-SW4), the qualifier (`__strong`, `__weak`, `__unsafe_unretained`, `__autoreleasing`) should be placed first, e.g., `__weak NSString *text`. This is technically incorrect, but much more clear since it immediately stands out (and is allowed by the compiler).

### Property attributes

Property attributes are declared in the following order:

1. Atomicity (`nonatomic`)
1. Storage type (`weak`, `strong`, `copy`, etc.)
1. Visibility (`readonly`, `readwrite`)
1. Custom getters and setters


**For example:**
```objc
@property (nonatomic, strong, readonly) NSManagedObjectContext *managedObjectContext;
@property (nonatomic, weak) IBOutlet UILabel *detailDescriptionLabel;
@property (nonatomic) BOOL selected;
```

## Naming

Apple’s naming conventions should be adhered to wherever possible, especially those related to [memory management rules](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/MemoryMgmt/Articles/MemoryMgmt.html) ([NARC](http://stackoverflow.com/a/2865194/340508)).

**For example:**

```objc
UIButton *magicButton;
```

**Not:**

```objc
UIButton *magBut;
```

Use MCS prefixes for internal code. Three-letter class prefixes are preferred, but Apple doesn't respect their own guidelines in this regard anyway (see [Mantle](https://github.com/Mantle/Mantle) and [Metal](https://developer.apple.com/library/prerelease/ios/documentation/Miscellaneous/Conceptual/MTLProgGuide/Cmd-Submiss/Cmd-Submiss.html#//apple_ref/doc/uid/TP40014221-CH3-SW1)). You can omit them for Core Data entity names.

Externally visible constants should be written in camel case with all words capitalized and prefixed by the related class name for clarity.

**For example:**

```objc
// MCSMagicalCollectionView.h
extern NSString * const MCSMagicalCollectionViewDidSparkleNotification;
```

**Not:**

```objc
static const NSTimeInterval animDuration = 0.5;
```

Properties and local variables should be in camel case with the leading word being lowercase.

Instance variables should be in camel case with the leading word being lowercase, and should be prefixed with an underscore. This is consistent with instance variables synthesized automatically by LLVM. **If LLVM can synthesize the variable automatically, then let it.**

**For example:**

```objc
@synthesize descriptiveVariableName = _descriptiveVariableName;
```

**Not:**

```objc
id varnm;
```

## Comments

Use common sense. Explain why you’re doing something, not the steps you’re taking.

Never commit code that has been commented out (take a second and go through `git diff` before applying your changes, make sure you're leaving codebase in better shape than before, no one likes to deal with a messy code).

[Documentation comments](http://nshipster.com/documentation/) are good. Use them when they're actually useful and needed. If you can refactor something so that you don't need to explain it, do that instead.

## Init and Dealloc

`init` methods go at the top. `dealloc` goes right below them. Consider wrapping them in `#pragma mark - Lifecycle` or something similar.

`init` methods should be structured like this:

```objc
- (instancetype)init
{
    self = [super init]; // or call the designated initializer
    if (self) {
        // Custom initialization
    }

    return self;
}
```

Where applicable, **always** return `instancetype`.


## Literals

> Then Apple said, "I give you `NSDictionary`, `NSArray`, `NSString`, and `NSNumber` literals." And Apple saw everything it had made, and behold, it was very good.

No, really. They're good for you. Use them.

**For example:**

```objc
NSArray *names = @[@"Agata", @"Ania", @"Bartek", @"Daniel", @"Dawid", @"Dominik", @"Jarek",
                   @"Karol", @"Klaudia", @"Maciej", @"Maciej", @"Marcin", @"Rafał", @"Wojtek",
                   @"Wojtek", @"Zbigniew", @"Janek"];
NSDictionary *animals = @{@"dogs" : @[@"Boomer"]};
NSNumber *officeIndoorSwimmingPool = @YES;
NSNumber *over9000 = @9001;
```

## Categories

Categories should be declared in separate files called ClassName+CategoryName.{h,m}.

There are no well-defined semantics for defining the same method in more than one category. In order to avoid undefined behavior, add a prefix to method names in categories on framework classes (e.g. `NSArray`, `UIImageView`, `AFHTTPRequestOperationManager`). Don't add a prefix to method names in categories on your own classes.

**For example:**

```objc
// NSArray+MCSCollectionUtility.h

@interface NSArray (MCSCollectionUtility)

- (void)mcs_each:(void(^)(id object))block;
- (void)mcs_eachWithIndex:(void(^)(id object, NSUInteger index))block;
...
```

```objc
// MCSMagicalCollectionView+AbraCadabra.h

@interface MCSMagicalCollectionView (AbraCadabra)

- (void)abraCadabra;
- (void)hocusPocus;
...
```

## CGRect Functions

When accessing the `x`, `y`, `width`, or `height` of a `CGRect`, always use the [`CGGeometry` functions](http://developer.apple.com/library/ios/#documentation/graphicsimaging/reference/CGGeometry/Reference/reference.html) instead of direct struct member access. From Apple's `CGGeometry` reference:

> All functions described in this reference that take CGRect data structures as inputs implicitly standardize those rectangles before calculating their results. For this reason, your applications should avoid directly reading and writing the data stored in the CGRect data structure. Instead, use the functions described here to manipulate rectangles and to retrieve their characteristics.

**For example:**

```objc
CGRect frame = self.view.frame;

CGFloat x = CGRectGetMinX(frame);
CGFloat y = CGRectGetMinY(frame);
CGFloat width = CGRectGetWidth(frame);
CGFloat height = CGRectGetHeight(frame);
```

**Not:**

```objc
CGRect frame = self.view.frame;

CGFloat x = frame.origin.x;
CGFloat y = frame.origin.y;
CGFloat width = frame.size.width;
CGFloat height = frame.size.height;
```

In most common scenarios, when you want to get the size of a view, refer to its `bounds` and not `frame`. You can read [this post](http://macoscope.com/blog/understanding-frame/) to know why it might not be safe to base off of `frame`.

```objc
CGRect bounds = self.view.bounds;

CGFloat width = CGRectGetWidth(bounds);
CGFloat height = CGRectGetHeight(bounds);
```

Try not to cut corners when you're adding a view which fills the entire superview. `UIScrollView` changes `bounds` when changing `contentOffset`.

```objc
CGRect superBounds = self.view.bounds;

UIView *subview = [[UIView alloc] initWithFrame:CGRect(0, 0, CGRectGetWidth(superBounds), CGRectGetHeight(superBounds)];
```

**Instead of:**

```objc
CGRect superBounds = self.view.bounds;

UIView *subview = [[UIView alloc] initWithFrame:superBounds];
```

Remember about `convertRect:toView:` and `convertRect:fromView:` methods when working with frames. They also handle the `UIScollView` `bounds` well.

```objc
CGRect frameWithoutTransformations = [scrollView convertRect:scrollView.bounds toView:scrollView.superview];
```

## Constants

Magic values should be avoided. If you do need to define a constant, it should be a `static` constant.

Avoid `#define`s. Unless you're writing an actual macro. Then, by all means, go ahead.

You may use Hungarian notation for implementation-level constants.

**For example:**

```objc
// MCSCompany.h
extern NSString * const MCSCompanyTagline;

// MCSCompany.m
NSString * const MCSCompanyTagline = @"We make award-winning apps for Apple devices.";

static NSString * const kOfficeLocation = @"Warsaw, Poland";

```


**Not:**

```objc
#define CompanyTagline @"We make award-winning apps for Apple devices."
#define maxNumberOfRows 2
```

## Enumerated Types

Use NS_ENUM(). Not sure why? NSHipster [has you covered.](http://nshipster.com/ns_enum-ns_options/)

NS_ENUM also provides [Swift interoperability](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/BuildingCocoaApps/InteractingWithCAPIs.html#//apple_ref/doc/uid/TP40014216-CH8-XID_15).

**For example:**

```objc
// MCSFishEye.h

typedef NS_ENUM(NSInteger, MCSFishEyeState) {
    MCSFishEyeStateCollapsed,
    MCSFishEyeStateExpandedActive,
    MCSFishEyeStateExpandedPassive
};
```

```swift
// SomeFile.swift

fishEyeView.state = .Collapsed
```


## Bitmasks

NS_OPTIONS(). For the same reasons as stated earlier with regards to NS_ENUM(). Again, read the article.

**For example:**

```objc
typedef NS_OPTIONS(NSUInteger, MCSCustomerHappiness) {
    MCSCustomerHappinessVeryHapy = 1 << 0,
    MCSCustomerHappinessExtremelyHappy = 1 << 1,
    MCSCustomerHappinessEnormouslyHappy = 1 << 2
};
```

## Private Properties

Private properties should be declared in class extensions (anonymous categories) in the implementation file of a class. Named categories (such as `MCSPrivate` or `private`) should never be used unless you’re extending another class.

**For example:**

```objc
@interface MCSFishEyeView()

@property (nonatomic, strong) NSArray *itemContainers;
@property (nonatomic, strong) NSArray *items;

@property (nonatomic, strong) UIView *transformView;

@end
```

### Custom getters

There's no real consensus here. But remember, custom getters are _custom_ for a reason. In most cases, when you have a property, the default getter should work just fine. That's what most developers expect. They see a header file and expect the default behaviour.

If you want to lazy load somethings, to have a singleton object or to have a `readonly` property whose result has to be calculated (ex. `count` of `NSArray`), feel free to create one. Just make sure you have a good reason to do so.

### Subclassing and Testing

Private properties and methods that need to be visible to some parts of the code for subclassing or testing purposes should be declared in class extensions in separate header files. To indicate the private nature of the header, the filename should follow the `ClassName_Description.h` format (notice an `_` used instead of a `+` to differentiate from category headers).

**For example:**

```objc
// MCSSomeObject_PrivateProperties.h

@interface MCSSomeObject ()

@property (nonatomic) NSInteger someProperty;

@end
```

```objc
// MCSSomeObject.m

#import "MCSSomeObject.h"
#import "MCSSomeObject_PrivateProperties.h"
```


## Booleans

Since `nil` resolves to `NO`, it is unnecessary to compare it in conditions. Never compare something directly to `YES`, because `YES` is defined to 1 and a `BOOL` can be up to 8 bits.

This allows for more consistency across files and greater visual clarity.

**For example:**

```objc
if (!someObject) {
}
```

**Not:**

```objc
if (someObject == nil) {
}
```

-----

**For a `BOOL`, here are two examples:**

```objc
if (isAwesome)
if (![someObject boolValue])
```

**Not:**

```objc
if ([someObject boolValue] == NO)
if (isAwesome == YES) // Never do this.
```

## Singletons

Singleton objects should use a thread-safe pattern for creating their shared instance.
```objc
+ (instancetype)sharedInstance
{
    static id sharedInstance = nil;

    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        sharedInstance = [[self alloc] init];
    });

    return sharedInstance;
}
```
This will prevent [possible and sometimes prolific crashes](http://cocoasamurai.blogspot.com/2011/04/singletons-your-doing-them-wrong.html).

Try not to create excessive singletons. Perhaps what you're trying to do could be accomplished much easier using dependency injection?

## Xcode Project

The physical files should be kept in sync with the Xcode project files in order to avoid file sprawl. Any Xcode groups created should be reflected by folders in the filesystem. Code should be grouped not only by type, but also by feature for greater clarity. You can use [Synx](https://github.com/venmo/synx) by @venmo to organize the files on your disk for you.

When possible, always turn on "Treat Warnings as Errors" in the target's Build Settings and enable as many [additional warnings](http://boredzo.org/blog/archives/2009-11-07/warnings) as possible. If you need to ignore a specific warning, use [Clang's pragma feature](http://clang.llvm.org/docs/UsersManual.html#controlling-diagnostics-via-pragmas).

## Refactoring

When you add/change code and you see that the nearby code is bad (e.g. doesn't comply with the guidelines you're reading right now), **change it**.

**Example**: you want to add a constant at the top of a file and you see that already present constants use #define instead of const. First, in a separate commit, change #define to const, then add your constant.

## Miscellaneous

* Given how long Objective-C identifiers can get, it's rather pointless to enforce the hard line length limit. However, it's also important to not get carried away, so turn on page guide at column 100. Don't be afraid to break it when it increases code clarity.
* Don't be afraid of multiple return statements. Your code should follow the [golden path](http://en.wikipedia.org/wiki/Happy_path). Multiple nested conditionals should be avoided.

# Other Objective-C Style Guides

If ours doesn't fit your tastes, have a look at some other style guides:

* [NYTimes Objective-C Style Guide](https://github.com/NYTimes/objective-c-style-guide/)
* [Google](http://google-styleguide.googlecode.com/svn/trunk/objcguide.xml)
* [GitHub](https://github.com/github/objective-c-conventions)
* [Adium](https://trac.adium.im/wiki/CodingStyle)
* [Sam Soffes](https://gist.github.com/soffes/812796)
* [CocoaDevCentral](http://cocoadevcentral.com/articles/000082.php)
* [Luke Redpath](http://lukeredpath.co.uk/blog/my-objective-c-style-guide.html)
* [Marcus Zarra](http://www.cimgf.com/zds-code-style-guide/)

# Acknowledgements

This document is largely based on the [NYTimes Objective-C Style Guide](https://github.com/NYTimes/objective-c-style-guide/), with sections we agreed with and felt we couldn't improve upon pulled verbatim from it.

It's also heavily influenced by and references [NSHipster](http://nshipster.com) in multiple places.
