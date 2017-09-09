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

<img src="img/git.png" width="50" />

Like a git commit

_Describes the entire state at a specific moment in time_

---

# JSON Patch

Deltas describing updates that were applied to the tree.

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

<img src="img/git.png" width="50" />

Like a git patch

_Describes the modifications from one commit to the next_

---

# Middleware

Intercept action invocations

---

# Middleware

<img src="img/git.png" width="50" />

Like git hooks

_pre- / post process specific commands_

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

.timeline_top[
```
🦆  💩📃📃💦
```
.appear[
```
       💥
     OutOfToiletPaperException
```
<br/>
]
]
.timeline_bottom.appear[
```
Rewind:
👇
🦆
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
    // wait...
    self.toilet.donate()
    // wait...
    self.isRelaxing = true
    // wait...
    self.isRelaxing = false
    // wait...
    self.wipe()
    // wait...
    self.wipe()
    // wait...
    self.toilet.flush()
}
```
]

---

.inline_block[
```javascript
async function fullVisit() {
    await delay(1000)
    self.toilet.donate()
    await delay(1000)
    self.isRelaxing = true
    await delay(1000)
    self.isRelaxing = false
    await delay(1000)
    self.wipe()
    await delay(1000)
    self.wipe()
    await delay(1000)
    await self.toilet.flush()
}
```

.appear[
Nope!
]
]

---

# Asynchronous processes in MST

.inline_block[
1. Built-in concept
2. Based on generators
3. Why: can run middleware on every continuation
]

---

.inline_block[
```javascript
const fullVisit = process(function* fullVisit() {
    yield delay(1000)
    self.toilet.donate()
    yield delay(1000)
    self.isRelaxing = true
    yield delay(1000)
    self.isRelaxing = false
    yield delay(1000)
    self.wipe()
    yield delay(1000)
    self.wipe()
    yield delay(1000)
    yield self.toilet.flush()
})
```
]

`async function` &rarr; `process(function* `

`await` &rarr; `yield`

---

class: timeline

.timeline_top[
```
🦆        💩        📃          📃         💦
```
.appear[
```
                               💥
                    OutOfToiletPaperException
```
]
]
.timeline_bottom.appear[
```
Rewind:
👇
🦆
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