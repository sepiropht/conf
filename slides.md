name: none
layout: true

---

name: inverse
layout: true
class: left, middle, inverse

.footnote[[bit.ly/rust-and-wasm](https://bit.ly/rust-and-wasm)]

---

name: normal
layout: true
class: left, middle

.footnote[[bit.ly/rust-and-wasm](https://bit.ly/rust-and-wasm)]

---

class: middle, center

# Rust 🦀 and Wasm 🕸

#### William Mbotta
#### [@sepiropht](https://twitter.com/fitzgen) | [@rustwasm](https://twitter.com/rustwasm)

![rustwasmjs](public/img/rustwasmjs.png)

.footnote[[bit.ly/rust-and-wasm](https://bit.ly/rust-and-wasm)]

???

* I'm lead of the rust+wasm WG
* we are making wasm awesome via rust

---

template: inverse

# Use Rust-generated WebAssembly to speed up your performance-sensitive JavaScript

???

* This is the ideal we're aiming for

---

class: center

# Do <u>NOT</u> Rewrite &mdash; Integrate
<br/>
<img src="./public/img/lin_rustjs.png"/>

???

* Corollary:
  * since we are surgically replacing performance-sensitive code paths, the
    other code paths can remain the same
  * which means that Rust+Wasm is *augmenting* your JS
  * not replacing it
* furthermore, you should be able to leverage rust-generated wasm packages from
  npm transparently
  * nothing in your workflow changes
  * you just get faster dependencies

---

class: js-code

.filename[index.js]

```js
import { greet } from "./pkg/greet";

greet();
```

???

* what is "hello world" of integrating with JS?
  * importing a function with ES modules!
* Rust-generated wasm is consumable as an ES module
* just looking at this, we can't tell if the module is JS or wasm
  * this is the level of transparent, it-just-works integration we aim for

---

class: rust-code

.filename[src/greet.rs]

```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
extern {
    fn alert(s: &str);
}

#[wasm_bindgen]
pub fn greet() {
    alert("Hello, World!");
}
```

???

* Here is how we implement that JS interface with Rust and Wasm
* if you don't know any Rust, don't worry we'll go through this bit by bit

---

class: rust-code

.filename[src/greet.rs]

```rust
*use wasm_bindgen::prelude::*;

#[wasm_bindgen]
extern {
    fn alert(s: &str);
}

#[wasm_bindgen]
pub fn greet() {
    alert("Hello, World!");
}
```

???

* `wasm-bindgen` is the tool we use for facilitating communication between JS
  and wasm
  * more on this later in the talk
* `use` is bringing `wasm-bindgen`'s common functionality into scope

---

class: rust-code

.filename[src/greet.rs]

```rust
use wasm_bindgen::prelude::*;

*#[wasm_bindgen]
*extern {
*   fn alert(s: &str);
*}

#[wasm_bindgen]
pub fn greet() {
    alert("Hello, World!");
}
```

???

* importing the `window.alert` function
* `extern` = these functions exist, but I don't have the definition
* `#[wasm_bindgen]` on an `extern` block creates imports at the `.wasm` level

---

class: rust-code

.filename[src/greet.rs]

```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
extern {
    fn alert(s: &str);
}

*#[wasm_bindgen]
pub fn greet() {
    alert("Hello, World!");
}
```

???

* exporting a `greet` function
  * `#[wasm_bindgen]` on a `pub` function makes it an export in the `.wasm`
    binary

---

class: rust-code

.filename[src/greet.rs]

```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
extern {
    fn alert(s: &str);
}

#[wasm_bindgen]
pub fn greet() {
*   alert("Hello, World!");
}
```

???

* calling imported `alert` function like we would any normal Rust function!

---

class: center

[![](./public/img/hello-world.png)](./public/img/hello-world.png)

---

# <img src="./public/img/world-map.png" style="max-width:1em; max-height:1em"/>  Roadmap

<hr/>

### 1. 🤔 Why Rust and WebAssembly?
### 2. <img src="./public/img/woman-lifting.png" style="max-width:1em; max-height:1em"/> Using Rust and WebAssembly
### 3. 🌭 How the Sausage is Made

---

class: middle, center
exclude: true

<video src="./public/img/why.mp4" autoplay="" loop="" playsinline=""></video>

???

* Those of you following along closely will know that this next section is about
  "why rust and webassembly?"
  * dive into both:
      1. why **rust** and wasm?
      2. why rust and **wasm**?

---

template: none
class: middle, center

![](./public/img/oxidizingsourcemaps.png)

.footnote[[hacks.mozilla.org/2018/01/oxidizing-source-maps-with-rust-and-webassembly/](https://hacks.mozilla.org/2018/01/oxidizing-source-maps-with-rust-and-webassembly/)]

???

* back in january, we sped up the `source-map` library by integrating Rust+Wasm
* motivating example: speeding up the `source-map` library by integrating
  rust+wasm
* source maps are debug info format for the Web
  * map lines in minified JS back to the original, unminified JS
* this experience ended up informing a lot of our WG's efforts
* different direction from emscripten:
  * trying to integrate into JS ecosystem
  * not trying to port native applications to the Web

---

# Idiomatic vs. Fast

#### Idiomatic JavaScript

```js
function decodeVlq(input) {
  // ...
  return { result, rest };
}
```

#### Faster-but-less-idiomatic JavaScript

```js
function decodeVlq(input, out) {
  // ...
  out.result = result;
  out.rest = rest;
}
```

???

* over the years, the original JS accumulated convoluted code in the name of
  performance
* often found abstraction and idiomatic code at odds with run time performance
* this example: reuse `out` object on every call = no allocation on every call
  * in theory, JITs *could* optimize the allocation away with escape analysis,
    but we found that it was not consistently happening across all JS engines
* other examples:
  * `return -1;` instead of `throw new Error`
  * Custom Quick Sort implementation to allow inlining comparator function

---

template: none
class: middle, swing-crab-bg

## Low-level<br/>Control
<br/>
## High-level<br/>Ergonomics

.footnote[Image credit: [reddit.com/u/ConeCandy](https://www.reddit.com/user/ConeCandy)]

---

template: none
class: center, middle

<video src="./public/img/cake.mp4" autoplay="" loop="" playsinline=""></video>

???

* have our cake and eat it too

---

## Control Over
<br/>
#### ✔ Memory layout and indirections
#### ✔ Allocation and deallocation
#### ✔ Monomorphization vs. dynamic dispatch
#### ✔ Function inlining

???

* deterministic, reliable, consistent performance
* no GC pauses

---

## Zero-cost abstractions

<br/>

> What you don’t use, you don’t pay for. And further: What you do use, you
> couldn’t hand code any better.

– Bjarne Stroustrup

???

* Example: iteration protocol in JS vs rust
  * chained `.map(..)` and `.filter(..)`
  * with JS:
      * implicitly heap allocating intermediate protocol objects that say if
        iteration is done or not, and have the next element in the iteration
      * maybe inlining each mapped or filtered function, maybe not
      * generally faster to use a C-style `for (i=0; i<n; i++)` loop
  * on the other hand, Rust
      * is not heap allocating intermediate objects
      * can boil away the iterators into the same code generated for a C-style
        `for` loop or better

---

class: middle, center

<div style="white-space:pre-wrap; font-size: 80%; margin-top: -5%">
  <span style="width: 1em; height: 1em; background-color: #f8766d; display: inline-block"></span> = pure JavaScript &Tab; &Tab; <span style="width: 1em; height: 1em; background-color: #00bfc4; display: inline-block"></span> = w/ Rust and Wasm
</div>
<a href="./public/img/pause-at-exception.svg">
  <object data="./public/img/pause-at-exception.svg" type="image/svg+xml"></object>
</a>

???

* time it takes to query a source map for info needed when a debugger pauses at
  an exception the first time
  * lower is better
  * red = original, pure-JS implementation
  * blue = new hybrid implementation with rust+wasm for core, perf-sensitive
    code paths
  * chrome, firefox, safari
* straight port led to ~6x faster than original implementation
* further algorithmic improvements got it up to ~11x faster (not reflected in
  this graph)
* and this is *after* we had already spent time optimizing the pure-JS over the
  years
* relative standard deviations fell: samples are much tighter together now
* Ultimately:
  * still a JS library! just core compute-bound kernel in rust+wasm
  * still have to rely on profiling!
  * algorithms are still important!
  * JS performance tuning finicky
      * need to know JS engine implementation and JIT internals
      * working without abstractions
  * Rust and Wasm: speed without wizardry
      * idioms guide us towards performant code
      * don't give up abstraction to get speed

---

class: center

.tiny[![tooling](./public/img/tools.png)]
# Top-notch tooling
###e.g. `cargo`

???

* C++ also has zero-cost abstractions
* where Rust stands out:
  * top-notch tooling that JS devs expect; `cargo` for dependency management, easy to get wasm environment up and running
      * compare this to adding dep on external C++ library: `autoconf`,
        `configure` scripts, `make`. `cmake` if you're lucky

---

class: center

.tiny[![cartwheel](./public/img/cartwheel.png)]
# Strong safety net

???

* safety net that is Rust's ownership and lifetime type system lets you:
  * push the envelope
  * be more ambitious with your code: perform less copies/allocation
  * but at the same time: you won't spend lots of time debugging heap
    corruption b/c compiler has your back

---

class: center

.tiny[![hearteyes](public/img/hearteyes.png)]
.tiny[![nerd](public/img/nerd.png)]
.tiny[![stareyes](public/img/stareyes.png)]

# Welcoming, inclusive community

???

* the Rust community strives to be
  * welcoming
  * inclusive
  * empower developers from a variety of backgrounds who haven't done
    systems programming before
  * for example, JS developers

---

template: none
class: middle

<img src="./public/img/animal-bros.jpg" style="float:right; max-width: 50%; margin: .5em"/>

## Plays Well With Others
<br/>
#### ✔ Integrates with bundlers
#### ✔ Publish to NPM

.footnote[[instagram.com/henrythecoloradodog](https://www.instagram.com/henrythecoloradodog/)]

???

* final differentiator is playing well with others and integrating with the
  larger ecosystem
* Rust on native has core tenet of inter-operating with C code
* Rust on wasm has core tenet of inter-operating with JS
* Keep using bundlers like webpack you know and love
* Keep using NPM for dependencies
  * rust+wasm will create NPM dependencies and consume them

---

class: middle, center

# Using Rust and Wasm
<br/>
<video src="./public/img/dog-lifting.mp4" autoplay="" loop="" playsinline=""></video>

---

# `wasm-pack`
<br/>
### 👷 Building `.wasm`
### 🎁 Creating and publishing NPM packages
### <img src="./public/img/female-scientist.png" style="max-height: 1em; max-width: 1em"/> Testing in headless browsers

---

# `wasm-bindgen`
<br/>
## Wasm<sup style="font-size: 150%">💬 🗨</sup>JavaScript

???

* facilitates communication between wasm and JS
* "bindgen" = "bindings generator"
* raw wasm functions only take and return primitive number types
  * but we want to pass strings, objects, DOM nodes, etc
  * wasm-bindgen enables that, all with the zero-cost abstraction principle
    we keep revisiting
  * generates JS glue you would otherwise have to write by hand

---

class: rust-code

.filename[src/greet.rs]

```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
extern {
    fn alert(s: &str);
}

/// Make a greeting!
#[wasm_bindgen]
pub fn greet() {
    alert("Hello, World!");
}
```

???

* revisit "hello world" and look closer at workflow

---

class: center

# `wasm-pack build`
<br/>
![](./public/img/wasm-pack-build-pipeline.png)

???

* `wasm-pack build` generates:
  * a `.wasm` file, containing your Rust compiled to wasm
  * a JS glue file that provides nice JS APIs to your Rust code
  * a TypeScript interface definition file for type checking your JS/TS and
    getting autocompletions
  * a `package.json` for integrating with JS tooling, bundlers, publishing to
    NPM, etc...

---

class: center, middle

![](./public/img/wasm-bindgen-code-communication.png)

???

* your JS interacts with the wasm through the generated JS glue code
* the wasm interacts with DOM/HTML5/Node/etc APIs through the generated JS glue

---

class: js-code

.filename[pkg/greet.js]

```js
// ...

export function __wbg_alert_2c86be282863e459(arg0, arg1) {
    let varg0 = getStringFromWasm(arg0, arg1);
    alert(varg0);
}

// ...
```

???

* the Rust code imported `window.alert`
* so generated JS has glue for wrapping imported functions and translating their
  arguments from numbers to JS things

---

class: js-code

.filename[pkg/greet.js]

```js
// ...

export function __wbg_alert_2c86be282863e459(arg0, arg1) {
*   let varg0 = getStringFromWasm(arg0, arg1);
    alert(varg0);
}

// ...
```

???

* recall from earlier talks that strings aren't simple in wasm
* read a Rust string from wasm memory into a JS string
* arg0 = pointer to start of string
* arg1 = length of the string

---

class: js-code

.filename[pkg/greet.js]

```js
// ...

export function __wbg_alert_2c86be282863e459(arg0, arg1) {
    let varg0 = getStringFromWasm(arg0, arg1);
*   alert(varg0);
}

// ...
```

???

* then call the actual `window.alert` function

---

class: js-code

.filename[pkg/greet.js]

```js
import * as wasm from './greet_bg';

// ...

/**
 * Make a greeting!
 * @returns {void}
 */
export function greet() {
    return wasm.greet();
}
```

???

* the generated JS also wraps the raw wasm exports and provides a nice JS
  interface to them

---

class: js-code

.filename[pkg/greet.js]

```js
*import * as wasm from './greet_bg';

// ...

/**
 * Make a greeting!
 * @returns {void}
 */
export function greet() {
    return wasm.greet();
}
```

???

* this is importing the actual `.wasm` file as an ES module

---

class: js-code

.filename[pkg/greet.js]

```js
import * as wasm from './greet_bg';

// ...

/**
 * Make a greeting!
 * @returns {void}
 */
export function greet() {
*   return wasm.greet();
}
```

???

* the generated JS also wraps the raw wasm exports and provides a nice JS
  interface to them
* it converts and wraps arguments from JS into something that wasm can
  understand
  *  some combination of `i32`/`i64`/`f32`/`f64`
* we just don't have any arguments here yet, so it doesn't do anything
* what if we wanted to write our greeting into a DOM node of the user's choice?

---

class: js-code

.filename[index.js]

```js
import { greet2 } from "./pkg/greet2";

greet2(document.body);
```

???

* this version of hello world is putting its greeting into some DOM node's text
  content
* to call this version of the function, we need to pass a DOM node
* we don't have to do anything fancy to pass the DOM node to wasm, the generated
  JS takes care of that

---

class: rust-code

.filename[src/greet2.rs]

```rust
use wasm_bindgen::prelude::*;
use web_sys::Node;

#[wasm_bindgen]
pub fn greet2(node: &Node) {
   node.set_text_content(Some("Hello, World!"));
}
```

???

* no more importing common Web functions by hand
  * using `web_sys` instead

---

class: rust-code

.filename[src/greet2.rs]

```rust
use wasm_bindgen::prelude::*;
*use web_sys::Node;

#[wasm_bindgen]
*pub fn greet2(node: &Node) {
   node.set_text_content(Some("Hello, World!"));
}
```

???

* now we are taking a reference to a DOM node parameter
  * we know it is a reference because of the `&`

---

class: rust-code

.filename[src/greet2.rs]

```rust
use wasm_bindgen::prelude::*;
use web_sys::Node;

#[wasm_bindgen]
pub fn greet2(node: &Node) {
*  node.set_text_content(Some("Hello, World!"));
}
```

???

* and setting its `textContent` to "Hello, World!"

---

class: center

# `wasm-pack build`
<br/>
<video src="./public/img/excited-corgi.mp4" autoplay="" loop="" playsinline=""></video>

---

class: js-code

.filename[pkg/greet2.js]

```js
import * as wasm from './greet2_bg';

// ...

export function greet2(arg0) {
    try {
        return wasm.greet2(addBorrowedObject(arg0));
    } finally {
        stack.pop();
    }
}
```

???

* Not showing import wrappers
  * wrapping the `Node.prototype.textContent` setter is largely the same as
    wrapping `window.alert`

---

class: js-code

.filename[pkg/greet2.js]

```js
import * as wasm from './greet2_bg';

// ...

export function greet2(arg0) {
    try {
*       return wasm.greet2(addBorrowedObject(arg0));
    } finally {
*       stack.pop();
    }
}
```

???

* `addBorrowedObject` pushes an object onto the stack and returns its index
* `stack.pop()` removes it from the stack after the call is done

---

class: center

### `greet2` took a <u>borrowed</u> DOM node...

--

<br/>
### What if we took <u>ownership</u> the DOM node?

---

class: rust-code

.filename[src/greet3.rs]

```rust
thread_local! {
    static ALL_NODES: RefCell<Vec<web_sys::Node>> =
        Default::default();
}

#[wasm_bindgen]
pub fn greet3(node: web_sys::Node) {
    ALL_NODES.with(|all_nodes| {
        let mut all_nodes = all_nodes.borrow_mut();
        all_nodes.push(node);

        for n in all_nodes.iter() {
            n.set_text_content(Some("Hello, World!"));
        }
    });
}
```

???

* now we are taking ownership of DOM nodes, instead of borrowing them
* that means we can do tricky things like keep a list of every DOM node we were
  ever given
* and write our greeting into *all* of their `textContent`s

---

class: rust-code

.filename[src/greet3.rs]

```rust
thread_local! {
    static ALL_NODES: RefCell<Vec<web_sys::Node>> =
        Default::default();
}

#[wasm_bindgen]
*pub fn greet3(node: web_sys::Node) {
    ALL_NODES.with(|all_nodes| {
        let mut all_nodes = all_nodes.borrow_mut();
        all_nodes.push(node);

        for n in all_nodes.iter() {
            n.set_text_content(Some("Hello, World!"));
        }
    });
}
```

???

* no more `&`; that's how you know we are taking ownership of the argument

---

class: rust-code

.filename[src/greet3.rs]

```rust
*thread_local! {
*   static ALL_NODES: RefCell<Vec<web_sys::Node>> =
*       Default::default();
*}

#[wasm_bindgen]
pub fn greet3(node: web_sys::Node) {
    ALL_NODES.with(|all_nodes| {
        let mut all_nodes = all_nodes.borrow_mut();
        all_nodes.push(node);

        for n in all_nodes.iter() {
            n.set_text_content(Some("Hello, World!"));
        }
    });
}
```

???

* we have a thread-local vector we are saving the nodes we are given in

---

class: rust-code

.filename[src/greet3.rs]

```rust
thread_local! {
    static ALL_NODES: RefCell<Vec<web_sys::Node>> =
        Default::default();
}

#[wasm_bindgen]
pub fn greet3(node: web_sys::Node) {
    ALL_NODES.with(|all_nodes| {
        let mut all_nodes = all_nodes.borrow_mut();
*       all_nodes.push(node);

        for n in all_nodes.iter() {
            n.set_text_content(Some("Hello, World!"));
        }
    });
}
```

???

* we add this most recently given node

---

class: rust-code

.filename[src/greet3.rs]

```rust
thread_local! {
    static ALL_NODES: RefCell<Vec<web_sys::Node>> =
        Default::default();
}

#[wasm_bindgen]
pub fn greet3(node: web_sys::Node) {
    ALL_NODES.with(|all_nodes| {
        let mut all_nodes = all_nodes.borrow_mut();
        all_nodes.push(node);

*       for n in all_nodes.iter() {
*           n.set_text_content(Some("Hello, World!"));
*       }
    });
}
```

???

* and finally we set the `textContent` for all of the nodes we've ever been
  given

---

class: center

# `wasm-pack build`
<br/>
<video src="./public/img/breakdance.mp4" autoplay="" loop="" playsinline=""></video>

???

* run `wasm-pack build` again to regenerate the bindings

---

class: js-code

.filename[pkg/greet3.js]

```js
import * as wasm from './greet3_bg';

/// ...

export function greet3(arg0) {
    return wasm.greet3(addHeapObject(arg0));
}

/// ...
```

???

* how do the generated bindings for `greet3` differ from `greet2`?
* not doing a `stack.pop()` at the end anymore

---

class: js-code

.filename[pkg/greet3.js]

```js
import * as wasm from './greet3_bg';

/// ...

export function greet3(arg0) {
*   return wasm.greet3(addHeapObject(arg0));
}

/// ...
```

???

* instead of adding borrowed objects, we are adding heap objects
* these objects have dynamic lifetime, which requires more bookkeeping than
  stack objects, and are freed in the `web_sys::Node` destructor
* `wasm-bindgen` will look at ownership/borrowing in function signatures and
  generate more efficient bindings code when it is safe to do so

---

template: inverse
class: middle, center

## Rust's ownership model helps us optimize Wasm ↔ JavaScript

---

## Streaming Statistics in Rust
<br/>
### Don't keep every sample in memory
### `mean = sum(samples) / count(samples)`

???

* let's take a look at a slightly more involved example: exposing a Rust
  struct+methods to JS
* this example is going to track some statistics in streaming fashion
  * not keeping all samples around, hogging a bunch of memory
      * mean = sum of samples / number of samples
      * can just store
          1. sum of samples and
          2. number of samples
      * and then compute mean without every sample all at the same time
  * maybe useful for an FPS counter or something like that
  * we could also compute variance, stddev, min, max, etc

---

class: js-code

.filename[index.js]

```js
import { StreamingStats } from "./pkg/streaming_stats";

const stats = new StreamingStats();

for (let i = 0; i < 1000; i++) {
  stats.add(Math.random());
}

console.log(stats.mean());

stats.free();
```

???

* how would we ideally like to use a `StreamingStats` class?

---

class: js-code

.filename[index.js]

```js
*import { StreamingStats } from "./pkg/streaming_stats";

const stats = new StreamingStats();

for (let i = 0; i < 1000; i++) {
  stats.add(Math.random());
}

console.log(stats.mean());

stats.free();
```

???

* again, using ES modules to import the `StreamingStats` ES class

---

class: js-code

.filename[index.js]

```js
import { StreamingStats } from "./pkg/streaming_stats";

*const stats = new StreamingStats();

for (let i = 0; i < 1000; i++) {
  stats.add(Math.random());
}

console.log(stats.mean());

stats.free();
```

???

* constructing is just like any plain JS class

---

class: js-code

.filename[index.js]

```js
import { StreamingStats } from "./pkg/streaming_stats";

const stats = new StreamingStats();

for (let i = 0; i < 1000; i++) {
* stats.add(Math.random());
}

*console.log(stats.mean());

stats.free();
```

???

* calling the `add` or `mean` method is just liek any plain JS class as well

---

class: js-code

.filename[index.js]

```js
import { StreamingStats } from "./pkg/streaming_stats";

const stats = new StreamingStats();

for (let i = 0; i < 1000; i++) {
  stats.add(Math.random());
}

console.log(stats.mean());

*stats.free();
```

???

* the only different thing is that you have to remember to call `free` when
  you're done with it.

---

class: rust-code

.filename[src/streaming_stats.rs]

```rust
#[wasm_bindgen]
pub struct StreamingStats {
    count: u32,
    sum: f64,
}
```

???

* let's look at the Rust code for implementing `StreamingStats`

---

class: rust-code

.filename[src/streaming_stats.rs]

```rust
*#[wasm_bindgen]
pub struct StreamingStats {
    count: u32,
    sum: f64,
}
```

???

* make sure struct has `#[wasm_bindgen]` annotation on it

---

class: rust-code

.filename[src/streaming_stats.rs]

```rust
#[wasm_bindgen]
*pub struct StreamingStats {
    count: u32,
    sum: f64,
}
```

???

* and that it has `pub` visibility
* that's all that's needed to get an ES class that mirrors the Rust `struct`!

---

class: rust-code

.filename[src/streaming_stats.rs]

```rust
#[wasm_bindgen]
impl StreamingStats {
    // ...
}
```

???

* let's also add a constructor and some methods
* to expose methods:
  * throw `#[wasm_bindgen]` on an impl block
  * then any `pub` method inside will be exposed to JS as well
      * as a method on the generated ES class

---

class: rust-code

.filename[src/streaming_stats.rs]

```rust
// ...

#[wasm_bindgen]
impl StreamingStats {
*   pub fn add(&mut self, sample: f64) {
*       self.count += 1;
*       self.sum += sample;
*   }

    pub fn mean(&self) -> f64 {
        self.sum / (self.count as f64)
    }

    // ...
}
```

???

* to add a sample:
  * increment the `self.count`
  * add the sample to the rolling sum

---

class: rust-code

.filename[src/streaming_stats.rs]

```rust
// ...

#[wasm_bindgen]
impl StreamingStats {
    pub fn add(&mut self, sample: f64) {
        self.count += 1;
        self.sum += sample;
    }

*   pub fn mean(&self) -> f64 {
*       self.sum / (self.count as f64)
*   }

    // ...
}
```

???

* to get the current mean of all samples seen so far, we divide the rolling sum
  by the number of smaples seen

---

class: rust-code

.filename[src/streaming_stats.rs]

```rust
#[wasm_bindgen]
impl StreamingStats {
    // ...

    #[wasm_bindgen(constructor)]
    pub fn new() -> StreamingStats {
        StreamingStats {
            count: 0,
            sum: 0.0,
        }
    }
}
```

???

* also need to let JS create instances
* to add a constructor:

---

class: rust-code

.filename[src/streaming_stats.rs]

```rust
#[wasm_bindgen]
impl StreamingStats {
    // ...

    #[wasm_bindgen(constructor)]
*   pub fn new() -> StreamingStats {
        StreamingStats {
            count: 0,
            sum: 0.0,
        }
    }
}
```

???

* static `new` method that returns `StreamingStats`
* Rust doesn't have constructors, just static methods and `struct` literals

---

class: rust-code
.filename[src/streaming_stats.rs]

```rust
#[wasm_bindgen]
impl StreamingStats {
    // ...

*   #[wasm_bindgen(constructor)]
    pub fn new() -> StreamingStats {
        StreamingStats {
            count: 0,
            sum: 0.0,
        }
    }
}
```

???

* use the `constructor` attribute with `wasm-bindgen`
* this tells `wasm-bindgen` to create a JS constructor instead of a static
  method named `new`

---

class: center

# `wasm-pack build`
<br/>
<video src="./public/img/zoidberg.mp4" autoplay="" loop="" playsinline=""></video>

---

class: js-code

.filename[pkg/streaming_stats.js]

```js
import * as wasm from './streaming_stats_bg';

export class StreamingStats {
    constructor() {
        this.ptr = wasm.streamingstats_new();
    }
    free() {
        const ptr = this.ptr;
        this.ptr = 0;
        wasm.__wbg_streamingstats_free(ptr);
    }
    add(arg0) {
        wasm.streamingstats_add(this.ptr, arg0);
    }
    mean() {
        return wasm.streamingstats_mean(this.ptr);
    }
}
```

???

* I cleaned this up a little bit and hid unnecessary things

---

class: js-code

.filename[pkg/streaming_stats.js]

```js
import * as wasm from './streaming_stats_bg';

export class StreamingStats {
    constructor() {
        this.ptr = wasm.streamingstats_new();
    }
    free() {
        const ptr = this.ptr;
        this.ptr = 0;
        wasm.__wbg_streamingstats_free(ptr);
    }
*   add(arg0) {
        wasm.streamingstats_add(this.ptr, arg0);
    }
*   mean() {
        return wasm.streamingstats_mean(this.ptr);
    }
}
```

???

* each of the `pub` methods we created have a corresponding method on the ES
  class
* since we are using `f64`, the arguments don't need any special treatment
* same for the constructor

---

class: js-code

.filename[pkg/streaming_stats.js]

```js
import * as wasm from './streaming_stats_bg';

export class StreamingStats {
*   constructor() {
*       this.ptr = wasm.streamingstats_new();
*   }
    free() {
        const ptr = this.ptr;
        this.ptr = 0;
        wasm.__wbg_streamingstats_free(ptr);
    }
    add(arg0) {
        wasm.streamingstats_add(this.ptr, arg0);
    }
    mean() {
        return wasm.streamingstats_mean(this.ptr);
    }
}
```

???

* and the ES class constructor calls our `StreamingStats::new` constructor
* it saves a pointer into the wasm linear memory where the `StreamingStats`
  instance lives

---

class: js-code

.filename[pkg/streaming_stats.js]

```js
import * as wasm from './streaming_stats_bg';

export class StreamingStats {
    constructor() {
        this.ptr = wasm.streamingstats_new();
    }
*   free() {
*       const ptr = this.ptr;
*       this.ptr = 0;
*       wasm.__wbg_streamingstats_free(ptr);
*   }
    add(arg0) {
        wasm.streamingstats_add(this.ptr, arg0);
    }
    mean() {
        return wasm.streamingstats_mean(this.ptr);
    }
}
```

???

* there is also a `free` method
* Once we give `StreamingStats` to JS, it is JS's responsibility to manage its
  lifetime
* Rust compiler can't tell when it goes out of scope, like it normally can
* also encounter this with C and C++ compiled to wasm
* can also run into this when writing "C" in JS!

---

class: center

## Managing Lifetimes from JavaScript
<!-- <br/> -->
<!-- #### 🏗 &emsp; Component lifecycle hooks -->
<!-- #### 🥑 &emsp; `with` functions -->
<!-- #### 🤸 &emsp; *Future:* GC weak refs and finalizers -->

---

class: js-code

.filename[component-lifecycle-hooks.js]

```js
class StreamingStatsElement {
  connectedCallback() {
    this.stats = new StreamingStats();
  }

  // ...

  disconnectedCallback() {
    this.stats.free();
  }
}
```

???

* use webcomponents and custom elements component lifetime callbacks
* or `componentDidMount` and `componentWillUnmount` for React

---

class: js-code

.filename[with-functions.js]

```js
export function withStreamingStats(callback) {
  const stats = new StreamingStats();
  try {
    return callback(stats);
  } finally {
    stats.free();
  }
}
```

???

* similar to Python's `with` statement
* RAII in JS
* creates a resource
* invokes callback with the resource
* ensures that the resource is always cleaned up afterwards
  * as a user, you don't have to clean it up yourself
* can also have `async`/`await` version

---

class: js-code

.filename[with-functions.js]

```js
withStreamingStats(stats => {
  for (let i = 0; i < 1000; i++) {
    stats.add(Math.random());
  }

  console.log(stats.mean());
});
```

???

* can use `stats` inside the callback
* but `stats` is always cleaned up once the callback returns
* so you can't use it outside the callback

---

class: center

### *Future:* [TC39 Weak Refs and Finalizers](https://github.com/tc39/proposal-weakrefs)
<br/>
#### ✨ We'll get the garbage collector to clean up after us ✨

???

* register a cleanup function for our objects
* when the collector reclaims one of our objects, it will run the clean up
  function for us
* so we don't have to remember to do it ourselves

---

exclude: true

class: center

# How it's Made

<img src="./public/img/bigbird.gif" style="max-height: 50%; max-width: 50%"/>

???

* let's peak under the hood
* see how the sausage is made

---

exclude: true

template: inverse
class: center

<img src="./public/img/alert.png" style="max-width:30%; max-height:30%" class="inverse" />

# Numbers Only

???

* TODO

---

exclude: true

* wish that WebIDL would specify whether buffer source is const or not
* wish that TypeScript included whether a method throws or not

---

## In Conclusion:

<br/>

##### <img src="./public/img/high-voltage.png" style="max-width:1em; max-height:1em"/>  Use Rust and Wasm to speed up perf-sensitive JS

--

##### 💖 Rust and Wasm integrates with your JS toolchain

--

##### ✨ `wasm-pack` makes it easy to build, test, and publish to NPM

--

##### 💬 `wasm-bindgen` enables Wasm ↔ JavaScript communication

???

* that follows the zero-cost abstractions principle

---

class: center

# Learn more at [rustwasm.github.io/book](https://rustwasm.github.io/book)

???

* short book on rust+wasm development
* game of life tutorial
* designing code for rust+wasm
* debugging
* time profiling
* code size profiling
* publishing to NPM
* more

---

class: center

# [Join the Rust and Wasm Working Group!](https://github.com/rustwasm/team/blob/master/README.md#get-involved)

<img src="./public/img/wasmwg.png" style="max-width: 50%; max-height: 50%"/>

???

* we don't bite, but we do hug

---

template: inverse
class: center

# THANK YOU!
<br/>
### Nick Fitzgerald
<br/>
### [@fitzgen](https://twitter.com/fitzgen) | [@rustwasm](https://twitter.com/rustwasm)
