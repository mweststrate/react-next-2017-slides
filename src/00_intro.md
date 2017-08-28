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


class: fullscreen

![tree](img/bathroom.jpg)

---

I <i class="em em-heart"></i> React

---

# React: declarative composition

.number.bg_navy[
  1
]

.number_column[
```javascript
const Bathroom = () =>
    <div className="Bathroom">
        <BathroomIcon />
        <FlushingIcon />
        <Painting />
        <Buttons />
        <ToiletPaper />
        <Toilet />
    </div>
```
]

---

# React: contract based

.number.bg_orange[
  2
]

.number_column[
1. Only props are exposed
1. Internal state encapsulated
1. Runtime type checking
]
---

# React: contract based

.number.bg_orange[
  2
]

.number_column[
```javascript
export class Bathroom extends Component {
    static.propTypes = {
        drain: PropTypes.object,
        electricity: PropTypes.object
    }
}
```
]

---

# React: explicit lifecycle

.number.bg_green[
  3
]

.number_column[
```javascript
export class Bathroom extends Component {
    componentWillMount() {
        this.connectToDrain()
    }

    componentWillunmount() {
        this.disconnectFromDrain()
    }
}
```
]

---

# React: dependency injection

.number.bg_purple[
  4
]

.number_column[
```javascript
class Lightbulb extends React.Component {
  static contextTypes = {
    electricity: PropTypes.object
  }
}
```
]

---

# React: instance reconciliation

.number.bg_red[
  5
]

.number_column[
1. Performance
2. Preserve internal state<br/ ><input value="Test" style="margin-left: 7px;
    padding: 5px;
    font-size: 16px;
    border: 2px solid #ccc;" />
]

---

# React: control & flexibility

.inline_block[
1. Declarative
2. Internal state hidden, type-checked api
3. Model lifecycle
4. Dependency injection
5. Preserve internal state
]