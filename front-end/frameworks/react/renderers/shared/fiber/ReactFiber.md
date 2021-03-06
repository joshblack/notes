# `ReactFiber`

`ReactFiber` exports the `Fiber` type, as well as a variety of functions for building different kinds of Fibers.

#### `createFiber(tag: TypeOfWork, key: null | string): Fiber`

Given a tag and possible key, build a Fiber!

#### `cloneFiber(fiber: Fiber, priorityLevel: PriorityLevel): Fiber`

Used to create an alternate fiber to do work on. From the source:

```js
// We clone to get a work in progress. That means that this fiber is the
// current. To make it safe to reuse that fiber later on as work in progress
// we need to reset its work in progress flag now. We don't have an
// opportunity to do this earlier since we don't traverse the tree when
// the work in progress tree becomes the current tree.
// fiber.progressedPriority = NoWork;
// fiber.progressedChild = null;

// We use a double buffering pooling technique because we know that we'll only
// ever need at most two versions of a tree. We pool the "other" unused node
// that we're free to reuse. This is lazily created to avoid allocating extra
// objects for things that are never updated. It also allow us to reclaim the
// extra memory if needed.
```

#### `createHostRootFiber(): Fiber`

Creates a Fiber with the `HostRoot` `TypeOfWork`.

#### `createFiberFromElementType(type: mixed, key: null | string): Fiber`

Creates a Fiber based on the given `type`, or we've been given a continutation that is already a fiber.

As a result, we branch based loosely off of the given `type` into the following scenairos:

- if `typeof type === 'function'`:
  - We use a `shouldConstruct` utility to decide whether we've been given a `ClassComponent` or `IndeterminateComponent` and build a fiber with the corresponding `tag`/`TypeOfWork`.
- if `typeof type === 'string'`:
  - We create a Fiber with the `HostComponent` tag
- if `typeof type === 'object' && type !== null && typeof type.tag === 'number'`:
  - Assumed to be a continuation and is a fiber already
- else:
  - We haven't gotten a valid Element type, warn if in `__DEV__`

#### `createFiberFromElement(element: ReactElement, priorityLevel: PriorityLevel): Fiber`

Construct a fiber using `createFiberFromElementType`, and add `element.props` to the fiber's `fiber.pendingProps` field, as well as set the priority level on the fiber.

#### `createFiberFromFragment(elements: ReactFragment, priorityLevel: PriorityLevel): Fiber`

Constructs a fiber with `Fragment` as the `tag`. Sets the fiber's `pendingProps` property to `elements`, and sets its `pendingWorkPriority` property as well.

#### `createFiberFromText(content: string, priorityLevel: PriorityLevel): Fiber`

Creates a fiber with the `HostText` `tag`. Set `pendingProps` to `content`, and `pendingWorkPriority` to `priorityLevel`.

## Module

This module imports the following modules:

```js
var ReactTypeOfWork = require('ReactTypeOfWork');
var {
  NoWork
} = require('ReactPriorityLevel');
var {
  NoEffect
} = require('ReactTypeOfSideEffect');
var {
  cloneUpdateQueue
} = require('ReactFiberUpdateQueue');
var invariant = require('invariant');
var {
  getCurrentFiberOwnerName
} = require('ReactDebugCurrentFiber');
```

This module imports the following types:

```js
import type { ReactElement, Source } from 'ReactElementType';
import type { ReactInstance, DebugID } from 'ReactInstanceType';
import type { ReactFragment } from 'ReactTypes';
import type { ReactCoroutine, ReactYield } from 'ReactCoroutine';
import type { ReactPortal } from 'ReactPortal';
import type { TypeOfWork } from 'ReactTypeOfWork';
import type { TypeOfSideEffect } from 'ReactTypeOfSideEffect';
import type { PriorityLevel } from 'ReactPriorityLevel';
import type { UpdateQueue } from 'ReactFiberUpdateQueue';
```

Finally, this module exposes the following exports:

```js
// This is used to create an alternate fiber to do work on.
// TODO: Rename to createWorkInProgressFiber or something like that.
exports.cloneFiber = function (fiber: Fiber, priorityLevel: PriorityLevel): Fiber

exports.createHostRootFiber = function (): Fiber

exports.createFiberFromElement = function (element: ReactElement, priorityLevel: PriorityLevel): Fiber

exports.createFiberFromFragment = function (elements: ReactFragment, priorityLevel: PriorityLevel): Fiber

exports.createFiberFromText = function (content: string, priorityLevel: PriorityLevel): Fiber

exports.createFiberFromElementType = createFiberFromElementType;

exports.createFiberFromCoroutine = function (coroutine: ReactCoroutine, priorityLevel: PriorityLevel): Fiber

exports.createFiberFromYield = function (yieldNode: ReactYield, priorityLevel: PriorityLevel): Fiber

exports.createFiberFromPortal = function (portal: ReactPortal, priorityLevel: PriorityLevel): Fiber
```

And exports the following types:

```js
// A Fiber is work on a Component that needs to be done or was done. There can
// be more than one per component.
export type Fiber = {
  // __DEV__ only
  _debugID ?: DebugID,
  _debugSource ?: Source | null,
  _debugOwner ?: Fiber | ReactInstance | null, // Stack compatible

  // These first fields are conceptually members of an Instance. This used to
  // be split into a separate type and intersected with the other Fiber fields,
  // but until Flow fixes its intersection bugs, we've merged them into a
  // single type.

  // An Instance is shared between all versions of a component. We can easily
  // break this out into a separate object to avoid copying so much to the
  // alternate versions of the tree. We put this on a single object for now to
  // minimize the number of objects created during the initial render.

  // Tag identifying the type of fiber.
  tag: TypeOfWork,

  // Unique identifier of this child.
  key: null | string,

  // The function/class/module associated with this fiber.
  type: any,

  // The local state associated with this fiber.
  stateNode: any,

  // Conceptual aliases
  // parent : Instance -> return The parent happens to be the same as the
  // return fiber since we've merged the fiber and instance.

  // Remaining fields belong to Fiber

  // The Fiber to return to after finishing processing this one.
  // This is effectively the parent, but there can be multiple parents (two)
  // so this is only the parent of the thing we're currently processing.
  // It is conceptually the same as the return address of a stack frame.
  return: ?Fiber,

  // Singly Linked List Tree Structure.
  child: ?Fiber,
  sibling: ?Fiber,
  index: number,

  // The ref last used to attach this node.
  // I'll avoid adding an owner field for prod and model that as functions.
  ref: null | (((handle : ?Object) => void) & { _stringRef: ?string }),

  // Input is the data coming into process this fiber. Arguments. Props.
  pendingProps: any, // This type will be more specific once we overload the tag.
  // TODO: I think that there is a way to merge pendingProps and memoizedProps.
  memoizedProps: any, // The props used to create the output.

  // A queue of state updates and callbacks.
  updateQueue: UpdateQueue | null,
  // A list of callbacks that should be called during the next commit.
  callbackList: UpdateQueue | null,
  // The state used to create the output
  memoizedState: any,

  // Effect
  effectTag: TypeOfSideEffect,

  // Singly linked list fast path to the next fiber with side-effects.
  nextEffect: ?Fiber,

  // The first and last fiber with side-effect within this subtree. This allows
  // us to reuse a slice of the linked list when we reuse the work done within
  // this fiber.
  firstEffect: ?Fiber,
  lastEffect: ?Fiber,

  // This will be used to quickly determine if a subtree has no pending changes.
  pendingWorkPriority: PriorityLevel,

  // This value represents the priority level that was last used to process this
  // component. This indicates whether it is better to continue from the
  // progressed work or if it is better to continue from the current state.
  progressedPriority: PriorityLevel,

  // If work bails out on a Fiber that already had some work started at a lower
  // priority, then we need to store the progressed work somewhere. This holds
  // the started child set until we need to get back to working on it. It may
  // or may not be the same as the "current" child.
  progressedChild: ?Fiber,

  // When we reconcile children onto progressedChild it is possible that we have
  // to delete some child fibers. We need to keep track of this side-effects so
  // that if we continue later on, we have to include those effects. Deletions
  // are added in the reverse order from sibling pointers.
  progressedFirstDeletion: ?Fiber,
  progressedLastDeletion: ?Fiber,

  // This is a pooled version of a Fiber. Every fiber that gets updated will
  // eventually have a pair. There are cases when we can clean up pairs to save
  // memory if we need to.
  alternate: ?Fiber,

  // Conceptual aliases
  // workInProgress : Fiber ->  alternate The alternate used for reuse happens
  // to be the same as work in progress.
};
```
