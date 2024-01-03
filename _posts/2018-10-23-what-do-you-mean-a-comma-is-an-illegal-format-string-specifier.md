---
title: What Do You Mean A Comma Is An Illegal Format String Specifier?
date: 2018-10-23
modified: 2018-10-23
tags: java university
published: true
excerpt: "Learning Java at university: life lessons. Sometimes it's the silly things that have you stuck for longer than you'd like to admit."
---

Overriding the `toString()` method in Java is (theoretically) a very simple thing to do. Most of the tutorials suggest doing basic string concatenation:

```java
return this.getName() + " has a monthly gross salary of \u00A3" + this.getMonthlyGrossSalary() + ", a taxrate of " +
    this.getTaxRate() + "%, and a monthly net salary of \u00A3" + this.getMonthlyNetSalary;
```

Unfortunately that kind of string formatting makes my head hurt after years of formatting strings by writing the whole thing out and then putting the variables at the end. So, despite the fact that I'm very new to using Java I decided that I wanted to do it the pretty way. As you can see this string includes both &pound; and &#37; signs.

[String formatting in Java](https://docs.oracle.com/javase/tutorial/java/data/numberformat.html) is pretty powerful, you can make a lot of formatting decisions when you add variables into strings for outputting (see [this stackoverflow answer](https://stackoverflow.com/a/40572459) for a clear explanation of the options). So I dutifully put in `%s` and `%.2f` in all the appropriate places, made sure that my pound signs and the single percent sign I needed were rendered via unicode. Thus:

```java
@Override
public String toString() {
    return String.format("%s has a monthly gross salary of \u00A3%.2f, a taxrate of %.2f\u0025, and a monthly" +
    "net salary of \u00A3%.2f.", this.getName(), this.getMonthlyGrossSalary(), this.getTaxRate(),
    this.getMonthlyNetSalary());
}
```

I went to hit compile and ...

![Illegal format string specifier: flag ',' not allowed in '%, a'](/assets/2018-10-23-formatting-error.png)

What the hell?

So I tried changing the comma to unicode... Nope. Remove the comma? Yeah, that works. Oh wait, no it's just changed the error message:

![Too few arguments for format string (found: 4, expected: 5)](/assets/2018-10-23-too-few-arguments.png)

Okaaay? So, it took me longer than it should have done to figure this one out, I counted the formatters (4), then the variables (4), and completely forgot that `%` whether or not it's rendered in unicode is a reserved character - in fact, my decision to render it in unicode obfuscated the whole issue. All I needed to do was escape the percent character by using another percent character and all my troubles went away:

```java
@Override
public String toString() {
    return String.format("%s has a monthly gross salary of \u00A3%.2f, a taxrate of %.2f%% and a monthly net" +
            "salary of \u00A3%.2f.", this.getName(), this.getMonthlyGrossSalary(), this.getTaxRate(),
            this.getMonthlyNetSalary());
}
```

## Output


>Bart Simpson has a monthly gross salary of &pound;2400.00, a taxrate of 40.00%, and a monthly netsalary of &pound;1440.00.
