---
layout: post
title: "Knowledge Engine (expert system)"
date: 2016-06-30 14:35:19
comments: true
description: "Knowledge Engine (expert system)"
keywords: "knowledge engine, pyke, expert system, python"
categories:
- development

tags:
- python

---

While developing backend service, I hit the concept of knowledge engine, specifically [PYKE](http://pyke.sourceforge.net/). After spending a few days with it, I would like to write down what I've learned so far.

# what is knowledge engine
Basically, you tell the engine a bunch of rules and factors. Based on those rules and factors, you can ask the engine different question and let the engine solve the puzzle on how to receive your goal.

Rule has two basic type:

- forward chaining: if ... then ...
- backward chaining: Then ... if...

# how do we use it
Each object has a current state, which gives us some basic factors. When the service receive a request, it set the target state accordingly. 

Now, we run forward chaining rule to add more factors based on initial ones. Then use backward chaining rule to find the steps we need to take to reach the target state.

# Pro. vs Con.

## Pros

### 1. easy to program complicated operations:

If the operation involve making calls to several different services and wait for the response, then based on the response do different actions. The engine can solve those cleanly. You only need to care about small steps and let the engine put them together. 

### 2. increase code reuse: 

Each rule(step) doesn't need to care about what happened before. It only cares about the current state of the objects and if it meet the rule, it just do the corresponding operations. So it is common that some operation share the same rules. for example, if all we need charge objects, we just check if the state of the object is 'uncharge' and send the request to our 'charge' service to do the work. Then push the state of the object to 'charged'.

### 3. easy for handling retry and cancel

If some operation failed and we need to do retry, we can just blindly throw the object back to the engine without modifying the state, so that the engine will pick the same rule again.
For cancel, we only need to change the target statue of the object and engine will nicely push the state to the 'cancel' statue.

## Cons

### 1. hard to debug

the rule is not written in code and not easy to debug through

### 2. unclear change scope

if you modify a rule, the scope of change is unclear as it will be shared by a lot of operations, which cannot be figured out easily. Some operation may accidentally enter the new rule and cause unexpected behavior.

### 3. need to be designed very carefully with everything in mind

since it is not very easy to change, if you don't design it correctly, i.e., you want to change the name of state, add/remove states, it will not be a easy task.

