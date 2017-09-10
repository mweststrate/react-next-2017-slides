<img src="img/paper1.jpg" style="
    position: absolute;
    left: -109px;
    top: 0px;
    height: 100%;
"    />

<div style="
    position: absolute;
    top: 0px;
    left: 525px;
    width: 50%;
    background: white;
    height: 700px;
    padding: 40px;
">
<br/><br/>
<img src="img/mobx2.png" height="80px" />
<img src="img/logo.png" style="height:80px"/>
## React, but for data

<small>
React Next 2017

Michel Weststrate - @mweststrate
</small>
<br/><br/>

<img src="img/mendix-logo.png" height="60px" /><br/>


<div>

---

# React: composes components

.number.bg_navy[
  1
]

.number_column[
```javascript
const Bathroom = () =>
    <div className="Bathroom">
        <Painting />
        <Buttons />
        <ToiletPaper />
        <Toilet />
    </div>
```
]

---

# React: composes components

.number.bg_navy[
  1
]

.number_column[
```javascript
const Toilet = observer(({ toilet }) =>
    <div>
        {toilet.pile.map((item, i) =>
            item.type === "🦆"
                ? <Duck />
                : <Sh_t />
        )}
        <Pos top={480} left={700}>
            <Emoji.toilet size={35} />
        </Pos>
    </div>
)
```
]

---

# MST: composes types

.number.bg_navy[
  1
]

.number_column[
```javascript
const Duck = types.model({
    name: types.string
})
```

.appear[
```javascript

const Sh_t = types.model({
    weight: 500,
    smell: 7
})
```
]
]

---

# MST: composes types

.number.bg_navy[
  1
]

.number_column[
```javascript
const Toilet = types.model({
    isFlushing: false,
    pile: types.array(types.union(Sh_t, Duck))
})
```

.appear[
```javascript

const Bathroom = types.model({
    amountOfToiletPaper: 0,
    toilet: Toilet,
    painting: Painting
})
```
]
]

---

.center[
<img src="img/react.svg" width="50" /><br/>
React composes a tree of component intances

.inline_block[
```
const instance = React.createElement(Component, props)
```
]

<br/>
<br/>
<img src="img/logo.png" width="50" /><br/>

MST composes a tree of rich type instances
.inline_block[
```
const instance = Type.create(snapshot)
```
]
]

---

class: columns

.left[
<img src="img/json.svg" width="100%" />
<br/>
<br/>
<center>Snapshots: Raw data</center>
]

.right[
<center><img src="img/types.svg" height="300px" /></center>
<br/>
<br/>
<center>Types: Data models</center>
]

---

.inline_block[
```javascript
const snapshot = {
    amountOfToiletPaper: 1,
    isRelaxing: false,
    toilet: {
        isFlushing: false,
        pile: [{ type: "💩", weight: 365, smell: 7 }],
        processed: 14145
    },
    painting: { painting: "🖼", anchor: { x: 264, y: 652 } }
}
```
]

---

# Constructing the state tree

.type_create[
<img src="img/types.svg" />
+
<img src="img/json.svg" />
=
<img src="img/mst_tree.svg" />
]

.inline_block[
```
const tree = Bathroom.create(snapshot)
```
]

---


# Actions transition tree to next state

.type_create[
<img src="img/mst_tree.svg" />
&rarr;
<img src="img/mst_tree_2.svg" />
]

---

# Getting the "props" out

.type_create[
<img src="img/mst_tree_2.svg" />
&rarr;
<img src="img/json_2.svg" />
]


.inline_block[
```
const snapshot = getSnapshot(tree)
```
]

---

.inline_block[

```javascript
const initialState = window.localStorage.getItem(JSON.parse("bathroom"))

const bathroom = BathroomModel.create(initialState)
```

.appear[
```

ReactDOM.render(
    <Bathroom bathroom={bathroom} />,
    document.getElementById("root")
)
```
]

.appear[
```

onSnapshot(bathroom, snapshot => {
    window.localStorage.setItem("bathroom", JSON.stringify(snapshot))
})
```
]
]

---


# React: contract based

.number.bg_orange[
  2
]

.number_column[
1. Only props are exposed
1. Internal state encapsulated
1. Design- & runtime type checking
]
---

# MST: contract based

.number.bg_orange[
  2
]

.number_column[
1. Define which props, views and actions are exposed
1. Internal state encapsulated
1. Design- & runtime type checking
]

---

.inline_block[
.boring[
```
const Bathroom = types
    .model(/* props */)
```
]
```
    .actions(self => {

        /* local state & funcs */
        let drainSocket

        function flush() {
            drainSocket.send(self.toilet.pile)
        }

        /* exposed actions */
        return { flush }
    })
```
]

---

.inline_block[
```
const bathroom = Bathroom.create(snapshot)

bathroom.flush()
```
]


---


# React: lifecycle & dependency injection

.number.bg_green[
  3
]

.number_column[
```javascript
export class Bathroom extends Component {
    static contextTypes = {
        drain: PropTypes.object
    }

    componentWillMount() {
        this.context.drain.connect()
    }

    componentWillUnmount() {
        this.context.drain.disconnect()
    }
}
```
]

---

# MST: lifecycle & dependency injection


.inline_block[
.boring[
```javascript
const Toilet = types.actions(self => {
```
]
```

    let drainSocket

    function postCreate() {
        drainSocket = getEnv(self).drain.getSocket
        drainSocket.connect()
    }

    function beforeDestroy() {
        drainSocket.disconnect()
    }

```
.boring[
```

    return { flush, beforeDestroy, postCreate }
})
```
]
```

const bathroom = Bathroom.create(data, { drain: somedrain })
```
]

---

# React: instance reconciliation

.number.bg_red[
  4
]

.number_column[
1. Performance
2. Preserve internal state<br/ ><input value="Test" style="margin-left: 7px;
    padding: 5px;
    font-size: 16px;
    border: 2px solid #ccc;" />
]

---

# MST: instance reconciliation

.inline_block[
```
applySnapshot(bathroom, someNewSnapshot)
```
]

.type_create[
<img src="img/mst_tree.svg" />
+
<img src="img/json_2.svg" />
&rarr;
<img src="img/mst_tree_2.svg" />
]


<small>
Local state like `drainSocket` is preserved / life-cycle hooks will run
</small>

---

class: columns

# It's React, but for data

.left[
<center>
<img src="img/react.svg" width="50" /><br/>
Component<br/>
Props<br/>
State<br/>
Instance<br/>
Context<br/>
Reconciliation<br/>
</center>
]

.right[
<center>
<img src="img/logo.png" width="50" /><br/>
Type<br/>
Snapshot<br/>
Volatile state<br/>
Instance<br/>
Environment<br/>
Reconciliation<br/>
</center>
]


