---
title: 'Monotonic Stack: Introduction, Properties, Implementations and Applications'
date: 2022-09-01 11:12:14
tags: algorithms
---

(WIP)

# Introduction

## Definition

A monotonic stack is fairly a simple data structure per se. It's a stack with an additional rule: items in the stack have to be ordered monotonically. That is, be arranged in an increasing or decreasing order from bottom to top. If the items are in a increasing order, the stack will be a monotonically increasing stack, otherwise, it will be a monotonically decreasing stack. Here is some examples:

(An example of monotonically increasing stack)

(An example of monotonically decreasing stack)

This isn't a monotonic stack because one of the items breaks the rule: it's smaller than the items that is stacked below it.

(An invalid example of monotonic stack)

## Operation

Since it is a stack, it has same operation as a normal stack: `push(value)` and `pop()`, but in order to maintain the monotonicity, we restrict on how they are used:

For popping it acts as same as a normal stack, since it won't breaks breaks the monotonicity.

For pushing, we have to repeatly pop the item from the stack first, until stack's top item is smaller than (or equal to) the item being pushed onto. Of course, if initially the stack's top item has already met the condition, or all items have been popped out, you may directly push the item onto the stack.

(Examples on popping and pushing items)

# Implementations

## from the begin to the end

## from the end to the begin

# Properties

A common use case of the monotonic stack is to solve the *cloest smaller value problem*, and its variants. This takes advancetage of some properties of a monotonic stack.

> **The nearest smaller value problem**
> Given an array of values `arr`, and an index `i`, find the value that is smaller than `arr[i]`, and it's cloest to `arr[i]`, i.e., their distances (`abs(j - i)`) is the smallest among all items that are smaller than `arr[i]`.
> 
> For example: let `arr` to be `[3,1,6,8,2,7]`. Which values in the array are smaller than `arr[2] == 6`? Obviously they are `arr[0] == 3`, `arr[1] === 1` and `arr[4] === 2`. Which one is closest to `arr[2]`? That is `arr[1]`, which is right on its left.
> 
> If we solve this problem with the brute-force method, the time complexity will usually be `O(N^2)`, since for each item we have to iterate through the array once. But with the help of monotonic stack, the complexity can be decreased, usually become `O(N)`.

## Proof

# Applications

