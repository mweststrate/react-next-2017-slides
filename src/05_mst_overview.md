<img src="img/logo.png" style="height:200px"/>

# Mobx-State-Tree

---

class: columns

.left[
<center>Snapshots: Raw data</center>
<br/>
<br/>
<img src="img/json.svg" width="100%" />
]

.right[
<center>Types: Data models</center>
<br/>
<br/>
<center><img src="img/types.svg" height="300px" /></center>
]

---

# const tree = Type.create(data)
.type_create[
<img src="img/types.svg" />
+
<img src="img/json.svg" />
=
<img src="img/mst_tree.svg" />
]

.appear[
Types: Actions. Local state. Type-checking.
]

---

```javascript
const Sh_t = types.model({
    type: types.literal("sh_t"),
    weight: 500,
    smell: 7
})

const Duck = types.model({
    type: types.literal("duck"),
    name: "Donald"
})

const Toilet = types.model({
    isFlushing: false,
    pile: types.array(types.union(Sh_t, Duck))
})

const Bathroom = types.model({
    amountOfToiletPaper: 0,
    toilet: Toilet,
    painting: Painting
})
```

---

```javascript
const initialState = window.localStorage.getItem("bathroom")

const bathroom = BathroomModel.create(initialState)

ReactDOM.render(
    <Bathroom bathroom={bathroom} />,
    document.getElementById("root")
)
```

---

# tree.callAction()
.type_create[
<img src="img/mst_tree.svg" />
&rarr;
<img src="img/mst_tree_2.svg" />
]

.appear[
Actions modify node properties & local state
]

---

```javascript
const Toilet = types
    .model(/* ... */)
    .actions(self => {
        function donate() {
            if (Duck.is(self.pile[0])) self.pile.clear()
            self.pile.push({ type: "sh_t" })
        }

        const flush = process(function* flush() {
            if (self.isFlushing) return
            self.isFlushing = true
            yield delay(2000)
            self.pile = [{ type: "duck" }]
            self.isFlushing = false
        })

        return {
            donate,
            flush
        }
    })
```

---

```javascript
export const Bathroom = types
    .model(/* ... */)
    .actions(self => {
        function wipe() {
            if (self.amountOfToiletPaper <= 0)
                throw new Error("OutOfToiletPaperException")
            self.amountOfToiletPaper -= 1
        }

        function restock() {
            self.amountOfToiletPaper += 3
        }

        function takeA____() {
            self.toilet.donate()
            self.wipe()
            self.wipe()
            self.toilet.flush()
        }

        return { wipe, restock, takeA____ }
    })
```

---

# const data = getSnapshot(tree)
.type_create[
<img src="img/mst_tree_2.svg" />
&rarr;
<img src="img/json_2.svg" />
]

.appear[
Reactive. Immutable. Cheap. Structurally shared.
]

---

```javascript
const bathroom = BathroomModel.create(initialState)

onSnapshot(bathroom, snapshot => {
    window.localStorage.setItem("bathroom", JSON.stringify(snapshot))
})
```

---

# applySnapshot(tree, data)

.type_create[
<img src="img/mst_tree.svg" />
+
<img src="img/json_2.svg" />
&rarr;
<img src="img/mst_tree_2.svg" />
]

.appear[
Reconciliation. Minimal amount of changes. Preserve state.
]

---

```javascript
const Toilet = types
    .actions(self => {
        const drain = getEnv(self).drain

        function postCreate() {
            drain.connect()
        }

        function beforeDestroy() {
            drain.disconnect()
        }

        return {
            beforeDestroy, postCreate
        }
    })

const bathroom = Bathroom.create(data, { drain: somedrain })
```

---

# MST: control & flexibility

.inline_block[
1. Declarative ✅
2. Internal state hidden, type-checked api ✅
3. Model lifecycle ✅
4. Dependency injection ✅
5. Preserve internal state ✅
]

---

# It's React, but for data

