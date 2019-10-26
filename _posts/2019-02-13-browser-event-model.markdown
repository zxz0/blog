---
layout: post
title:  "【转】Browser events: bubbling, capturing, and delegation"
date:   2019-02-13 12:00:00 -0700
categories: Front-End
tags: Front-End Browser Event
description: DOM事件模型（冒泡和捕获），以及事件委托机制（委托父元素捕捉处理，从而可以一次绑定，并且可以应对动态的元素添加。需要添加冒泡完全模拟子元素的触发情况）
---
### 总结自：
- [Browser events: bubbling, capturing, and delegation](https://blog.meteor.com/browser-events-bubbling-capturing-and-delegation-14db28e924ae)
- [event delegation -- 事件委托](https://www.jianshu.com/p/2c68c8ceef1c)

Diff, incompatible ways of propagating events:
- Netscape: capture (up -> down)
- Internet Explorer: bubble (specific -> general)  
-> W3C: standardize both when addEventListener, can be configured. combining by event first downward to specific, visiting the capturers, then (assuming propagation isn’t stopped) bubbles back up visiting the non-capturing handlers. NO addEventListener support in IE 6/7/8.  

Bubbling: bottom -> top (specific -> general)  
In practice: bubbling. lower can stop bubbling before it reaches higher. so: lower: one thing, higher: another

Event object properties:
- event.target: where the event actually occurred. 最终目标，要捉到事件的元素
- event.currentTarget: where the event is currently being handled. 当前捕捉到事件的元素，DOM tree上级的都有可能。可以不断变化。

Event delegation: popular technique built into libraries like jQuery. 对于所有元素的行为处理，不是一个个绑定，而是bind a single event handler to the BODY (因为bubbling，所以一定会接收到所有event), and look for event.target to see if it is the element desired. the BODY element is the delegator that handles events on behalf of the A tags, just like: the event handler is on the A tags. 但是子元素触发的(target不是我们绑定的元素了)也可以认为是当前元素触发了，所以除了target判断，还需要模拟bubbling向上看是否有我们绑定的元素作为parent存在。jQuery等库为了完全模拟当前元素事件，set event.currentTarget (非BODY而是我们绑定的元素, simulated) and target（真正出发的元素，我们绑定的元素的本身或子元素），BODY不重要所以无所谓。
- Add an event listener on some element enclosing all the elements where you want to simulate event handling (BODY)
- In the handler, simulate bubbling and look for matching elements (all A tags)
- When you find a matching element, set event.currentTarget to it and call subsequent handling code
实际上问题多多：比如delegated和non-delegated event handler的顺序等等，但是抽象就是这样。
如何delegate non-bublling event？不做。大多数想要delegate的都是bubbling event。
传统的non-bubbling events：
- change / submit / rest: not bubbling in IE 6/7/8, but do in all other browsers. jQuery makes them bubble.
- focus / blur: not, ever. since the browser window can also be sent focus / blur events. jQuery defined new name: focusin and focusout, aliases of focus and blur. if use, focusout to avoid confusion.
- load / error / resize / readystatechange / …: fired on the window or other non-DOM objects like XHRs, so delegation isn’t really relevant. Image tags have a “load” event in some browsers, but it’s not reliable enough to be useful.