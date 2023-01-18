---
layout: post
title: 'PHP4/5: Object orientation and compatibility'
date: '2009-11-03 16:20:37'
tags:
- php
redirect_from:
- /php4-5-object-orientation-and-compatibility
---

Some compatibility problems were reported following the release of [SLOODLE](http://sloodle.org) 1.0, and they largely appear to centre around the object-oriented plugin system I developed for the Presenter module.

The idea is fairly simple. The SLOODLE core team or 3rd party developers can create plugin classes to extend the module’s functionality. These will all belong to a structured inheritance hierarchy, and will be automatically detected and loaded by the SLOODLE framework. However, it has been a big challenge maintaining compatibility for both PHP4 and PHP5 because the object orientated features of the language have changed so much in that time.

I think I’ve finally found the solution, and it’s infuriatingly inelegant.

## What needs to be done

Unfortunately, there is no sensible way (at least as far as I know) to detect where on disk a particular class was loaded from, without perhaps some rather laborious naming scheme. As such, all the relevant files are included by the framework, and then all the declared classes are scanned to determine which of them are SLOODLE plugins.

## The ideal solution

All the plugin classes are ultimately derived from a single base class called `SloodlePluginBase`. It would be nice if we could just pass the name of any given class to `is_subclass_of()` to determine if it’s a SLOODLE plugin. Sadly though, in PHP4, `is_subclass_of()` requires an instance of an object rather than the name of a class. Instantiating _every_ class just to test if it’s a SLOODLE plugin would be inefficient and probably unsafe.

## Other potential solutions

1. Use `get_parent_class(..)` to check if the base class is the parent of the given class. This doesn’t work because there can be multiple levels of inheritance.
2. Use a naming convention. This would be possible, but really confusing to program due to the different way class naming is handled between PHP4 and 5. (Not to mention the fact that the existing naming conventions in Moodle and SLOODLE use CamelCase).
3. Define a unique static class variable (such as `$is_sloodle_plugin`) and check its existence with `isset()`. This doesn’t work because PHP4 doesn’t support static member variables.
4. Define a unique static class method, and check for its existence with `method_exists()`. This doesn’t work because PHP4 (on some or all versions) requires an object instance to test, rather than just a class name.
5. Define a unique static class method, and check for its existence with `is_callable()`. This isn’t ideal because it’s only compatible with PHP \>= 4.0.6. Some of our users may be stuck with earlier incarnations of PHP4.

## The solution

You could probably see a theme emerging with ideas 3-5. In the same light, my final solution has been to define a unique static method in the base class, called `sloodle_get_plugin_id()`. It will always return a value of some kind. The SLOODLE module then attempts to call that function on each potential plugin class (using the class name rather than a class instance). If it gets a value back then we know it was a plugin class.

Here’s the code which does it:

```php
$result = @call_user_func(array($classname, 'sloodle_get_plugin_id'));
if (!empty($result))
{
    // It's a SLOODLE plugin!
}
```

Frankly, I think it’s a hideous solution. Executing this code on all declared classes is neither elegant nor efficient. However, for now, it seems to be effective and portable.
