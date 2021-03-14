---
layout: note
title: Clean Code &#58; A Handbook of Agile Software Craftsmanship
categories: notes 
permalink: notes/books/clean-code/

hidden: true
---

Notes and quotes on Clean Code &#58; A Handbook of Agile Software Craftsmanship by Robert Cecil Martin

- toc
{: toc }

## Chapter 1: Clean Code

> You will not make the deadline by making the mess. Indeed, the mess will slow you down instantly, and will force you to miss the deadline. The only way to make the deadline—the only way to go fast—is to keep the code as clean as possible at all times.

## Chapter 2: Meaningful Names

Long but informative names are better than one letter ones. Make sure your code always shows what in intended even if it takes twice as space.  
Don't use misleading names. If two things are not similar, their names should not make you think so.
> Spelling similar concepts similarly is information. Using inconsistent spellings is  disinformation.  

Make sure to use **pronounceable** and **searchable** names.  

>When constructors are overloaded, use static factory methods with names that describe the arguments. For example,  
*Complex fulcrumPoint = Complex.FromRealNumber(23.0);*
is generally better than  
*Complex fulcrumPoint = new Complex(23.0);*  
Consider enforcing their use by making the corresponding constructors private.  

Pick one word per concept.
> It’s confusing to have  fetch, retrieve, and get as equivalent methods of different classes.  

**Example:** If you are using add for concatenating of two values, use append or insert for adding one item to a collection.

Add meaninful context.
Make sure it is understandable what the method or variable is for even when not used in context.

## Chapter 3: Functions
There are points in this chapter I thought was to strict. I agree that functions should not be long and do one thing in general. But keeping every function 2-3 lines long would make looking through code so much harder since to find the whole code so much navigation would be needed. Of course these are my opinions and does not depend on any real evidence.

> The first rule of functions is that they should be small. The second rule of functions is that *they should be smaller than that*.  

The indent level of a function should not be greater than one or two. If more lines needed create a new function with descriptive name.

**Only one thing** in a function. Functions should **not have side effects** and only do what is says it should. Also in a function there should be only **one abstraction level**.  

> FUNCTIONS SHOULD DO ONE THING. THEY SHOULD DO IT WELL. THEY SHOULD DO IT ONLY.

I rarely use switch statements and this is the quote on them:
> My general rule for switch statements is that they can be tolerated if they appear only once, are used to create polymorphic objects, and are hidden behind an inheritance   

**Arguments**  
- Number of arguments on a function should never exceed 3. 
- If there are related arguments, create a class for them. 
- Using so many arguments make testing a lot harder. 
- Don't use flag arguments, using them means you do two things in a function. Instead write two seperate functions.
- Avoid output arguments. If a function needs to change state of something, it should be its function.

> Anything that forces you to check the function signature is equivalent to a double-take. It’s a cognitive break and should be avoided.  

**Don't repeat yourself.**

## Chapter 4: Comments

Main point: If your code is clear enough you don't need comments.

>The proper use of comments is to compensate for our failure to express ourself in code.  

Comments can be used to: 
- express intention on a decision 
- clarify content  
- warn on consequences

>Any comment that forces you to look in another module for the meaning of that comment has failed to communicate to you and is not worth the bits it consumes. 

Make sure comment does not mislead and uses a clear language.  
Don't comment out code, version control is here for you.

>Don’t offer systemwide information in the context of a local comment.

