---
layout: post
title: Perform Selector Leak
published: true
categories: []
tags: [warning, leak, pragma, clang]
---
Have you ever had a flood of warnings similar to "performSelector may cause a leak because its selector is unknown"?

### Solution

Insert the following somewhere (I usually have a Constants.h I inlude in my .pch file).

	#define SuppressPerformSelectorLeakWarning(Content) \
    do { \
        _Pragma("clang diagnostic push") \
        _Pragma("clang diagnostic ignored \"-Warc-performSelector-leaks\"") \
        Content; \
        _Pragma("clang diagnostic pop") \
    } while (0)


The usage is fairly straight forward.

	SuppressPerformSelectorLeakWarning(
		[_target performSelector:_action withObject:self]
	);
	// or;
	id result;
	SuppressPerformSelectorLeakWarning(
		result = [_target performSelector:_action withObject:self]
	);

**However**, warnings should be followed and corrected when appropriate. Try the following instead:

	SEL selector = NSSelectorFromString(@"aMethod");
	IMP imp = [_controller methodForSelector:selector];
	void (*func)(id, SEL) = (void *)imp;
	func(_controller, selector);

An excellent summary of the above can be found [here](http://stackoverflow.com/a/20058585)
