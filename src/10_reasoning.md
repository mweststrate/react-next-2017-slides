# 3 ways to reason about what is happening with state

.inline_block[
1. Snapshots
2. Patches
3. Action middleware
]

---

# Snapshots

Immutable, structurally shared representation of entire state

_Like a commit in the git history: the complete state at a specific moment in time_

---


# JSON Patch

Deltas describing updates applied to the tree.

.inline_block[
```
{ op: "add", path: "/toilet/pile/0", value: {
    smell: 7,
    type: "💩",
    weight: 500
} }
```

1. Fine grained observability
2. Reversable
3. RFC-6902
]

_Like a git patch: describes the modifications from one commit to the next_

---

# Middleware

.inline_block[
```
addMiddleware(tree, (call, next) => {
    /* some pre-processing */

    // invoke next middleware
    const res = next(call)

    /* some post-processing */
    return res
})
```

_Like git hooks: pre / post process specific events_

---

# Taking a 💩 atomically

.appear[
    <img src="img/np1.jpg" style="position: fixed;width: 100%;height: 100%;top: 0px;left: 0px;" />
]

---

.inline_block[
```javascript
    .actions(self => {
        function takeA____() {
            self.toilet.donate()
            self.wipe() // 💥 <- Don't want to get stuck here...
            self.wipe()
            self.toilet.flush()
        }

        return {
            takeA____
        }
    })
```
]

---

.inline_block[
.boring[
```javascript
    .actions(self => {
        function takeA____() {
            self.toilet.donate()
            self.wipe() // 💥 <- Don't want to get stuck here...
            self.wipe()
            self.toilet.flush()
        }
```
]

```

        return {
            takeA____ : decorate(atomic, takeA____)
        }
```
.boring[
```
    })
```
]]

---

class: timeline

```
[💩📃📃💦]
```
.appear[
```
     💥
     OutOfToiletPaperException
```
<br/>
]
.timeline_bottom.appear[
```
Rewind:
👇
[💩📃📃💦]
```
]

---

.inline_block[
```javascript
export function atomic(call, next) {

    // record a preState
    const preState = getSnapshot(call.context)
    try {
        return next(call)
    } catch (e) {

        // exception: restore snapshot..
        applySnapshot(call.context, preState)

        // ..and rethrow
        throw e
    }
}
```
]

---

# Be a good citizen...

---

class: fullscreenw

<img src="img/clean.jpg" />

---

<div style="display:inline-block">
.boring[
```javascript
    .actions(self => {
```
]

```

        const takeA____ = process(function*() {
            self.toilet.donate()
            self.wipe()
            self.wipe()
            yield self.toilet.flush()
            self.wipe() // ..clean the seat
            self.wipe() // might 💥
        })
```

.boring[
```

        return {
            takeA____ : decorate(atomic, takeA____)
        }
    })
```
]
</div>

---

class: timeline

```
[💩📃📃💦]                      [🦆📃📃]
                                   💥
                OutOfToiletPaperException
```
<br/>
.timeline_bottom.appear[
```
Rewind:
👇
[💩📃📃💦]              📚👾🎮🕹📱📳        [🦆📃📃]
```
]

---

.inline_block[

```javascript
const runningActions = new Map()
```

.boring[
```javascript

export function atomic(call, next) {
    switch (call.type) {
        case "action":
            return atomic(call, next)
```
]

```javascript
        case "process_spawn":
            runningActions.set(call.id, getSnapshot(call.context))
            break
        case "process_throw":
            applySnapshot(call.context, runningActions.get(call.id))
```
.boring[
```
            runningActions.delete(call.id)
            break
        case "process_return":
            runningActions.delete(call.id)
            break
    }
    return next(call)
}
```
]
]