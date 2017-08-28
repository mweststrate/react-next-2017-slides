# 3 ways to reason about what is happening with data

.inline_block[
1. Snapshots
2. Patches
3. Action middleware
]

---

# Taking a *

```javascript
    .actions(self => {
        function takeA____() {
            self.toilet.donate()
            self.wipe() // <- Don't want to get stuck here...
            self.wipe()
            self.toilet.flush()
        }

        return {
            takeA____
        }
    })
```

---


# Taking a *

```javascript
    .actions(self => {
        function takeA____() {
            self.toilet.donate()
            self.wipe() // <- Don't want to get stuck here...
            self.wipe()
            self.toilet.flush()
        }

        return {
            takeA____ : decorate(atomic, takeA____)
        }
    })
```
