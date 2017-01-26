---
layout: default
title: greg's blog
date: 2017-01-26
---
# [](#header-1)Nested Brackets

It's the final year of my degree, so I'm doing a few job interviews at the moment. And I've run in to the following problem a few times:

> Given an input string $$ \alpha $$
>
> Determine if $$ \alpha $$ can be generated in the grammar:
>
> S $$ \rightarrow \epsilon $$
>
> S $$ \rightarrow $$ \{S\} S
>
> S $$ \rightarrow $$ \[S\] S
>
> S $$ \rightarrow $$ \(S\) S

In other words, we want to recognise any sequence of brackets that might be used to represent a branching structure like a tree.

So we know that strings like `{}()[]` and `{{}}([])[({})]` are valid. \\
But strings like `{]` and `{(})` are invalid.


How do we recognise these valid strings? \\
If you look up this question in Google, the [solutions you'll find](http://www.ardendertat.com/2011/11/08/programming-interview-questions-14-check-balanced-parentheses) will (almost) all use a **stack** to record the last seen opening bracket as we scan from left to right along the string.  

Now that makes sense, given that the above is a [context free grammar](https://en.wikipedia.org/wiki/Context-free_grammar), which can be recognised with a push-down automaton.
Push-down automata are like their less powerful cousins [finite-state automata](https://en.wikipedia.org/wiki/Finite-state_machine), but crucially they can employ a **stack** to push and pop from.

But with modern programming languages, we're provided with so many string processing tools for free that we don't need to explicitly allocate a stack, at least not for the purposes of our problem.

# [](#header-3)Towards a more elegant solution

What if, instead, we looked at our string and removed the sequences `{}`, `[]`, and `()` until the string is empty or no more can be removed?

So, after one pass this valid string: `{{}}([])[({})]` \\
will be transformed to: `{}()[()]` \\
and after another pass: `()` \\
until after one last pass we're left with the empty string $$ \epsilon $$.

But applying the same process to the invalid string: `{(})` will yield the same string again.

You can think of the above as repeatedly removing the leaves of a tree represented by our string, after each pass, our tree gets smaller until it can shrink no more: in which case it's empty - or it's not a tree :(

So in python, our method might look like this:

```python
	def brackets(s):
		brackets = ["{}", "()", "[]"]
		oldString = s
		for br in brackets:
			s = s.replace(br, "")
	     
		if len(s) == 0:
			return True
		else:
			if oldString == s:
				return False
			else:
				return brackets(s)
```

Now this recursive solution is OK, but now we're using another stack implicitly: the call stack. ugh.

We actually don't need this recursive feature if we update some bits of state ourselves:

```python
	def bracketsIter(s):
		brackets = ["{}", "()", "[]"]
		oldString = ""
		while(oldString != s):
			oldString = s
			for br in brackets:
				s = s.replace(br, "")
	     
			if len(s) == 0:
				return True
		return False
```

So this new iterative approach will terminate, returning `False`, if our string can't be reduced further. Or returning `True` if our string is ever empty.

Now this algorithm has a worse asymptotic time complexity than the standard approach with a stack. \\
But it can run in place on the string, whereas the method with a stack required O(n) space.