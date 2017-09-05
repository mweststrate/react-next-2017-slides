# 3 ways to reason about what is happening with state

.inline_block[
1. Snapshots
2. Patches
3. Action middleware
]

---

# Snapshots

Immutable, structurally shared representation of entire state

---

# Snapshots

_Like a commit in the git history: the complete state at a specific moment in time_

---

# JSON Patch

Deltas describing updates that were applied to the tree.

---


# JSON Patch

_Like a git patch: describes the modifications from one commit to the next_

---

# JSON Patch

.inline_block[
```
{ op: "add", path: "/toilet/pile/0", value: {
    smell: 7,
    type: "💩",
    weight: 500
} }
```
]

---

# JSON Patch

.inline_block[
1. Fine grained observability
2. Reversable
3. RFC-6902
]


---

# Middleware

Intercept action invocations

---

# Middleware

_Like git hooks: pre- / post process specific events_

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
]

---

# Taking a 💩 atomically

.appear[
    <img src="img/np1.jpg" style="position: fixed;width: 100%;height: 100%;top: 0px;left: 0px;" />
]

---

.inline_block[
```javascript
    .actions(self => {

        function fullVisit() {
            self.toilet.donate()
            self.wipe() // 💥 <- Don't want to get stuck here...
            self.wipe()
            self.toilet.flush()
        }

        return {
            fullVisit
        }
    })
```
]

---

.inline_block[
.boring[
```javascript
    .actions(self => {

        function fullVisit() {
            self.toilet.donate()
            self.wipe() // 💥 <- Don't want to get stuck here...
            self.wipe()
            self.toilet.flush()
        }
```
]

```

        return {
            fullVisit : decorate(atomic, fullVisit)
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
    const preState = getSnapshot(call.tree)

    try {

        // run the action
        return next(call)

    } catch (e) {

        // exception: restore snapshot..
        applySnapshot(call.tree, preState)

        // ..and rethrow
        throw e
    }
}
```
]

---

# More realistically

---

class: fullscreenw

<img src="img/gaming.jpg" />

---

.inline_block[
```javascript
function fullVisit() {
    self.toilet.donate()
    delay(1000)
    self.wipe()
    delay(1000)
    self.wipe()
    delay(1000)
    self.toilet.flush()
}
```
]

---

# Asynchronous processes in MST

---

.inline_block[
```javascript
async function fullVisit() {
    self.toilet.donate()
    await delay(1000)
    self.wipe()
    await delay(1000)
    self.wipe()
    await delay(1000)
    self.toilet.flush()
}
```

.appear[
Nope!
]
]

---

.inline_block[
```javascript
const fullVisit = process(function* fullVisit() {
    self.toilet.donate()
    yield delay(1000)
    self.wipe()
    yield delay(1000)
    self.wipe()
    yield delay(1000)
    self.toilet.flush()
})
```
]


`async function` &rarr; `process(function* `

`await` &rarr; `yield`


---

.inline_block[
```javascript
const fullVisit = process(function* fullVisit() {
    self.toilet.donate()
    yield delay(1000)
    self.wipe()
    yield delay(1000)
    self.wipe() // might 💥 in some distant future...
    yield delay(1000)
    self.toilet.flush()
})
```
]

---

# Generators allow intercepting continuations

So we can run middleware every time the process resumes

---

class: timeline

```
[💩]           [📃]            [📃]           [💦]
                               💥
                   OutOfToiletPaperException
```
<br/>
.timeline_bottom.appear[
```
Rewind:
👇
[💩]           [📃]            [📃]           [💦]
```
]

---

.inline_block[

```javascript
const preActionSnapshots = new Map()
```

.boring[
```javascript

function atomicAsync(call, next) {
    switch (call.type) {
        case "action":
            return atomic(call, next)
```
]

```javascript

        case "process_spawn":
            preActionSnapshots.set(call.id, getSnapshot(call.tree))
```
.boring[
```javascript
            break
```
]

```javascript

        case "process_throw":
            applySnapshot(call.tree, preActionSnapshots.get(call.id))
```
.boring[
```

            preActionSnapshots.delete(call.id)
            break
        case "process_return":
            preActionSnapshots.delete(call.id)
            break
    }
    return next(call)
}
```
]
]