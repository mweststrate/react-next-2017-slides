# Snapshots are too naive for time travelling...

---

class: timeline

.timeline_top[
```
💩   🖼    📃    🖼     📃           💦
                       💥
             OutOfToiletPaperException
```
]
.timeline_bottom.appear[
```
Rewind:
👇
💩   🖼    📃    🖼     📃           💦
```
.appear[
```
     👆   LOST!  👆
```
]
]

---

# Proper rewinding of changes

Time travelling in the face of concurrent updates

---

# Proper rewinding of changes

.inline_block[
1. .appear[Record patches and inverse apply them].appear[<br/><img src="img/git.png" width="50" />_Like git revert_<br/><br/>]
1. .appear[Record start state, rewind, replay all non-failing actions].appear[<br/><img src="img/git.png" width="50" />_Like git rebase_]
]

---

# Undo / Redo at Mendix

Patch based

---

# Record patches & reverse apply

---

class: timeline

.timeline_top[
```
🔴 Record patches

 💩     🖼      📃      🖼        📃          💦
                                 💥
^^^^           ^^^              ^^^         ^^^
```
]
.timeline_bottom.appear[
```
◀ Reverse apply:

 💩     🖼      📃      🖼        📃          💦
 💩             📃                📃          💦
 _____________________________________________ -
        🖼              🖼
```
]

---
```javascript
        case "process_spawn":
            const recorder = recordPatches(call.tree)
            runningActions.set(call.rootId, recorder)
            break

        case "process_resume":
        case "process_resume_error":
            const recorder = runningActions.get(call.rootId)
            recorder.resume()
            try {
                return next(call)
            } finally {
                recorder.stop()
            }

        case "process_throw":
            runningActions.get(call.rootId).undo()
            break
```

---

# Recorded patches

```javascript
[
    { op: "add", path: "/toilet/pile/1", value:
        { type: "💩", weight: 365, smell: 7 }
    },
    { op: "replace", path: "/isRelaxing", value: true },
    { op: "replace", path: "/isRelaxing", value: false },
    { op: "replace", path: "/amountOfToiletPaper", value: 0 }
]
```

---

# Inverse of recorded patches

```javascript
[
    { op: "remove", path: "/toilet/pile/1" },
    { op: "replace", path: "/isRelaxing", value: false },
    { op: "replace", path: "/isRelaxing", value: true },
    { op: "replace", path: "/amountOfToiletPaper", value: 1 }
]
```


---
# Record actions & replay

---

class: timeline

.timeline_top[
```
🔴 Record action invocations

 💩     🖼      📃      🖼       📃             💦
#1     #2              #3
                                💥
```
]
.timeline_bottom.appear[
```
⏪ Rewind & replay
👇
 💩     🖼      📃      🖼       📃             💦

▶ #2 #3
        🖼             🖼
```
]

---

.inline_block[
```javascript
const history = [] // { id, snapshot, call }[]

function atomicAsyncAction(call, next) {

    // record every 'root' action
    if (call.parentId === 0) {
        history.push({
            id: call.id,
            snapshot: getSnapshot(call.tree),
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
function atomicAsyncAction(call, next) {
    // ...
    switch (call.type) {
```
]

```
        case "process_throw":

            // find root call
            const idx = history.findIndex(item => item.id === call.rootId)

            // restore 'zero' state
            applySnapshot(history[idx].call.tree, history[idx].snapshot)

            // remove & replay all later action invocations
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
