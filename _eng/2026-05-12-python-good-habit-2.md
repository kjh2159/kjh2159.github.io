---
title: (Python) Good Coding Habits - Part 2
date: 2024-08-13 15:10:00 +0900
categories: [python, habit]
tags: [python, habit, coding, programming]
---

# Good Coding Habits in Python - Part 2

The good Python coding habit I want to introduce today is ***writing assignment code in a single line***.

What does this mean? You might think, “Don’t we normally write assignment code in one line anyway? Is there anything different?” However, when conditional statements such as `if` or loops such as `for` are involved, multiple lines can become part of a single assignment operation.

In such cases, developing the habit of writing the code concisely in one line can help you write cleaner and better code.

## Example

First, it will be easier to understand what I mean by looking at examples. The following two code snippets show examples of assigning values to variables using a conditional statement and a loop.

```python
num: int = int(input('input a number'))
result: str = None

if num%2 == 0:
    result = 'even'
else:
    result = 'odd'
```

```python
lst = []
for i in range(1, 100, 3):
    lst.append(i)
```

Do you see what I am trying to say?

What I want to talk about in this post is changing code like this into a much shorter form, usually in just one or two lines.

## Conditional Statement

### Purpose

Why do we need to make an effort to rewrite code into one line? I want to ask this question first.

“Is long code necessarily good code?”

“Is the purpose of this habit only refactoring and maintainability?”

No.

When someone says they wrote thousands, tens of thousands, or even hundreds of thousands of lines of code, do you feel impressed and think, “How could they write that much code?” It may indeed be impressive code. However, in many cases, meaningless code lines are simply left behind.

In conclusion, the main purpose of this habit is to reduce the effort needed to interpret unnecessary code as much as possible, ultimately reducing the time spent on debugging.

### Concise Form

Let’s take a closer look.

The conditional statement example is actually not difficult to understand. It receives a number and determines whether the number is odd or even. However, to understand this simple logic, we have to read six lines of code.

This becomes a waste of text, code lines, and effort.

Now, let’s look at the concise version.

```python
num: int = int(input('input a number'))
result: str = 'even' if num%2 == 0 else 'odd'
```

Isn’t this much simpler and easier to understand?

All the unnecessary lines are gone. Writing code in this concise form can help you write better code. Of course, if you are not used to this style, it may not look simple at first and may even feel harder to read. However, once you get used to it, the first example may start to look messy.

### Pros and Cons

The pros and cons are clear.

The advantage is that it removes the unnecessary code mentioned above.

The disadvantage is that it is not suitable for multiple conditions using `elif`. If you need to use `if~elif~else`, then you have no choice but to avoid this concise form.

## Loop

### Concise Form

For loops, let’s look at the concise version first.

```python
lst = [i for i in range(1, 100, 3)]
```

This allows us to assign values much more easily and conveniently.

Of course, this method can be used for iterable objects that can be processed with `for`. It is also a good way to avoid repeatedly calling the `list.append` function, which can make the code slower.

Compared to the old method of repeatedly adding elements through a loop, this approach has many advantages in terms of both memory usage and execution speed.

> Of course, you could also write it like this:
>
> ```python
> list(range(1, 100, 3))
> ```
>
> But this is not the point I want to make here, so let’s just move on.

## Conclusion

Today, we looked at how to express assignment code in a concise form.

This habit may not feel natural at first. However, these days, if someone is reasonably good at coding, you will often see them using this kind of syntax. You can also find this style very often by looking into well-made modern frameworks, modules, or GitHub repositories.

Of course, this is just a habit, so I cannot force anyone to write code this way all the time. Still, I hope this habit helps you communicate more effectively and appropriately with others through code.

I hope we can all become people who write better code.
