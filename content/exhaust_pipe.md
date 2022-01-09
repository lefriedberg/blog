+++
title = "Exhaust Pipe"
date = 2022-01-09
draft = false
[extra]
show_date = true
+++

A few months ago, I wrote a little library [Exhaust Pipe](https://github.com/RowanMcDonald/exhaust_pipe) to help out with a side project. Exhaust pipe is a helper library for managing tailwind token lists.

### The behavior in question

A common issue that you encounter using tailwind in a component based application is class conflict. Say you have a component that has a default margin and you allow passing in classes that will be applied to your component. 

What happens today is that the rule with the highest specificity is applied. Thus, if we have a default margin, passing in another margin class and concatenating it to the token list may or may not apply the rule.

This behavior is described in a github issue [here](https://github.com/tailwindlabs/tailwindcss/issues/1010)

Side note, in Tailwind play, a warning will alert you to the cssConflict in the div above. I wish there was a tool to surface this via static analysis. (Although this is tricky b/c whatever tool you use has to understand your component system.) They also wrote a [VS Code extension](https://tailwindcss.com/blog/introducing-linting-for-tailwindcss-intellisense) that will sometimes surface these conflicts.

### What is the problem with this behavior?
1. The order of css rule definitions is opaque to developers.
2. Conflicts fail silently.
   1. Unnecessary classes make it past code review
   2. Developers have to root out the conflict themselves.
3. Silent failures and unexpected effects make this behavior unfriendly to component based applications. 

### My goal with this project

In component based applications, we often want to allow developers to merge style lists. We want to be able to support two behaviors:
   1. Merge and raise errors about conflicts.
   2. Merge and override the style.

Exhaust pipe is still a proof of concept.  I'm not planning on fleshing it out unless I have a pressing need. My current team is too small to justify the investment. Maybe someday.
