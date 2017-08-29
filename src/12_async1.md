---

class: timeline

```
[💩📃📃💦]      [🖼]    [🖼]    [🦆📃📃]
                                  💥
                OutOfToiletPaperException
```
<br/>

.timeline_bottom.appear[
```
Rewind:
👇
[💩📃📃💦]      [🖼]    [🖼]    [🦆📃📃]
```
.appear[
```
                👆      👆
                  LOST!
```
]
]

---

# Snapshots are too naive for time travelling...

---

# Proper rewinding of changes

1. Record patches (diffs) and revert them
1. Record start state, rewind, replay all other actions

---

Mendix Undo / redo

---

# Record patches & reverse apply

---

class: timeline

```
spawn #1                       resume #1
|                              |
[💩📃📃💦]   [🖼]    [🖼] [🖼]   [🦆📃📃]
                                  💥
+------+                       +-----+ 🔴 patches

```
<br/>
.timeline_bottom.appear[
```
Reverse apply:

[💩📃📃💦]   [🖼]    [🖼] [🖼]   [🦆📃📃]
[💩📃📃💦]                      [🦆📃📃] -
______________________________________ =
            [🖼]    [🖼] [🖼]
```
]

---

```javascript
        case "process_spawn": {
            const recorder = recordPatches(call.context)
            runningActions.set(call.rootId, recorder)
            break
        }
        case "process_yield":
        case "process_yield_error": {
            const recorder = runningActions.get(call.rootId)
            try {
                recorder.resume()
                return next(call)
            } finally {
                recorder.stop()
            }
        }
        case "process_throw":
            runningActions.get(call.rootId).undo()
            break
```

---

# Record actions & replay

---

class: timeline

```
🔴 act #1   #2      #3  #4     resume #1
|           |       |   |      |
[💩📃📃💦]   [🖼]    [🖼] [🖼]   [🦆📃📃]
                                 💥
```
<br/>
.timeline_bottom.appear[
```
Rewind & replay

👇
[💩📃📃💦]   [🖼]    [🖼] [🖼]   [🦆📃📃]

▶
            [🖼]    [🖼] [🖼]
```
]

---

.inline_block[
```javascript
const history = [] // { id, snapshot, call }[]

function atomic(call, next) {

    // record every 'root' action
    if (call.id === call.rootId) {
        history.push({
            id: call.id,
            snapshot: getSnapshot(call.context),
            call
        })
    }

    // ...
}
```
]

---

<div style="display:inline-block">
.boring[
```javascript
function atomic(call, next) {
    // ...
    switch (call.type) {
```
]

```
        case "process_throw":
            // find root call
            const idx = history.findIndex(item => item.id === call.rootId)

            // restore 'zero' state
            applySnapshot(history[idx].call.context, history[idx].snapshot)

            // replay all later action invocations
            history.splice(idx).slice(1).forEach(item => {
                item.call.context[item.call.name].apply(null, item.call.args)
            })
```

.boring[
```
    }
}
```
]
</div>
