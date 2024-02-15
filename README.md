# MegaJS

With all these new Javascript runtimes going around, we decided to put our heads together to create the ultimate one: MegaJS.

## Features

### Top-Notch Permissions

The best Javascript runtimes currently available offer permissions to allow or deny access to certain OS APIs. MegaJS obviously implements those... and more.

```bash
$ megajs run --allow-net ./example.js
```

You can specify permissions not only per process or per file but per specific lines of code. Permissions get as fine-grained as allowing only specific tokens.

```bash
$ megajs run --in-lines=1:10->{allow-tokens:[console,.,log]} ./example.js
```

We can also automatically define permissions by reading your `LICENSE` file. Quite handy!

```bash
$ megajs run --license ./LICENSE ./example.js
```

### Cross-Runtime APIs

Current runtimes are cross-compatible enough. I should be able to use APIs from any runtime at any time. Node APIs into Deno APIs into Bun APIs? Why not. All JS code should be valid and I see no reason why to pick and choose what APIs are valid.

```ts
const fs = require('node:fs');
const file = await fs.readFileSync("tohash.txt", { encoding: "utf8" });
await Deno.writeTextFileSync("hashed.txt", Bun.hash(file).toString());
```

### Cross-Engine

Current runtimes are stuck to the engines they were designed for. Node and Deno are both stuck to v8. Bun is stuck to JavascriptCore. [Spiderfire](https://github.com/Redfire75369/spiderfire) is stuck to Spidermonkey.

This limitation makes us weak. It also means that if we choose a runtime, we are stuck dealing with the performance limitations of that runtime.

This will be really slow in Bun for example:
```ts
// regex is slow in JSC
const a = /[[^\(\)]*|(\(.*?\))]*/;
for(let i = 0; i < 100000; i++) {
	if(a.exec("foo(bar)"+i)) {
		console.log("works!")
	}
}
```

Running this in a v8 runtime would make it run a lot faster. We can do this by simply running our program with the `--runtime="v8"` flag.

Alternatively, we might want to do this at runtime (haha, get it?). This is where we include our first set of APIs, being the ability to change the runtime DYNAMICALLY at runtime.

Speeding up the above code would be trivial

```ts
// Speed up execution of this bit
mega.runtime("v8");
const a = /[[^\(\)]*|(\(.*?\))]*/;

for(let i = 0; i < 100000; i++) {
	if(a.exec("foo(bar)"+i)) {
		console.log("works!")
	}
}
// Return to low memory of JSC
mega.runtime("JavscriptCore");
```

This trick isn't particularly new, as we've seen from my previous experiments with SporkJS. What is new is that ALL runtimes are supported. That's right, all of them.

```ts
mega.runtime("v8"); // https://v8.dev
mega.runtime("JavascriptCore") // https://developer.apple.com/documentation/javascriptcore
mega.runtime("spidermonkey") // https://spidermonkey.dev/
mega.runtime("quickjs") // https://bellard.org/quickjs/
mega.runtime("duktape") // https://duktape.org/
```

We even support runtimes that don't fully work yet!

```ts
mega.runtime("nova") // https://github.com/trynova/nova
mega.runtime("boa") // https://github.com/boa-dev/boa
mega.runtime("starlight") // https://github.com/Starlight-JS/Starlight
```

**New in update 0.1.1**

After being inspired by NextJS, we've decided to add an alterative syntax to make it easier for you, the user.

```ts
"use v8";
console.log("hi from v8");
"use duktape";
console.log("hi from duktape");
```

### Cross-Runtime APIs (again)

Did you think I was joking the first time? Nope. MegaJS has TRUE cross-runtime API support, including browser APIs.

What about the DOM? Absolutely. Even super-discouraged APIs? WHY NOT!?!? Our core principles state that all code should be valid.

```ts
// JUST WORKS!
document.open();
document.write("<h1>Out with the old, in with the new!</h1>");
document.close();
```

Wanted to see this on your screen though? This is where we introduce yet another built-in API. `electron_2`!

```ts
// display our page in a browser
mega.electron_2.show();
// TODO: Show more functionality
```

We can of course do all of the things one would expect with `electron_2`, and more! Wanted to change what browser engine was being used? By default it's [servo](https://servo.org/), but we could of course switch to any engine you want, all at runtime of course.

```ts
// chromium engine
mega.electron_2.engine("blink");
// safari engine
mega.electron_2.engine("webkit");
// TODO: Document all 60 options
```

### Imports

One common source of confusion from developers we here about is how to import things. Do I use require? Do I use ECMAScript imports? Are HTTP imports allowed? Can I import from the blockchain :tm:? How about the AUR? "I actually like python imports better can I use those"?

We're here to support all of your needs, and refuse none of them. The consumer is always right as I like to say.

```ts
const express = require("express");
import thing from "./thing.js";
import oak from "https://deno.land/x/oak@v13.2.5/mod.ts";
import money from "1HB5XMLmzFVj8ALj6mfBsbifRoD4miY36v";
from foo import Foo
import kvdex from "jsr:@olli/kvdex@0.34";
```

You might say, "but what if I want to import one thing but use another API for it?". Don't worry, we have you covered. You can of course import anything however you like.

```ts
const oak = require("oak"); // oak isn't on npm? no problem.
import oak from "oak"; // guesses what you mean here
import oak from "http://oak.com"; // isn't a real URL? no worries, megajs figures it out
import "jquery" // let me find the latest version of JQuery hosted on a CDN for you real quick
```

What if you try to import a file and we can't guess what you mean? No worries there either. If your machine is powerful enough, we'll download the best open-source LLM and generate the file for you. If not, we'll make an OpenAI call. We're privacy first!

```ts
import { copyFile } from "./file_that_exports_copy_file.js";
copyFile("x.ts", "y.ts"); // just works!
```

### Compatibility

One common issue we hear is the churn of APIs. As time goes on, certain APIs get deprecated and removed and certain other APIs come into favor. We believe all code is valid code and thusforth never remove any APIs ever. We will always modify the runtime to make old code just work :tm: (even if it doesn't make sense).

For instance, we undeprecated the javascript [with statement](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/with). I thought it was useful so I brought it back.

```ts
with ([1, 2, 3]) {
  console.log(toString()); // 1,2,3
}
```

Our deep compatibility goes further than just bringing back old APIs (or implementing weird ones like [document.all](https://developer.mozilla.org/en-US/docs/Web/API/Document/all)), we even support stuff like Typescript decorators... and ECMAScript standard decorators. In the same file. With no configuration. How? We just guess.

```ts
// PERFECTLY VALID code as it SHOULD BE
//
// Stolen from: https://blog.logrocket.com/practical-guide-typescript-decorators/
@WithFuel
class Rocket {
  fuel: number = 75
  @deprecatedMethod
  isReadyForLaunch(): Boolean {
    return !(this as any).isEmpty()
  }
}

const rocket = new Rocket()
console.log(`Is the rocket ready for launch? ${rocket.isReadyForLaunch()}`)

// https://github.com/tc39/proposal-decorators
class Example {
  @reactive accessor myBool = false;
}
```

### Import Everything

This is where we run back to our import system. You can import any file, however you like.

```ts
// JSON value of the import
import J from "./x.json";
```

We can even import things that aren't really web-spec.

```ts
import C from "./x.csv";
import X from "./x.xml";
```

Want to import a dynamic library? Why not?

```ts
import lib from "./x.so";
```

> Implementation Note: For the dll case we just return a proxy object so that all methods can be existing at the same time.

We can obviously import from other languages and MegaJS will compile the program to run automagically.

```ts
import { hello } from "./x.rs";
hello(); // prints hello world, in rust
```

### Cross-language support

You may have caught a hint of this from last few sections but we support Typescript as well. Actually, we support [Flow, CoffeeScript and friends, ClojureScript, Elm, PureScript, ReasonML, Rescript, Gleam](https://linolevan.com/blog/transpiling_to_javascript)... actually just every language. Even [Blight](https://linolevan.com/blog/js_engines_do_too_much). All of them. Even in the same file if you want. No configuration needed, we just guess what you meant.

```ts
// typescript
console.log("hello world!");

// PureScript
import Prelude
import Effect.Console (log)
greet :: String -> String
greet name = "Hello, " <> name <> "!"
main = log (greet "World")

// rust
fn main() {
    println!("Hello World!");
}

// zig
const std = @import("std");
pub fn main() !void {
    const stdout = std.io.getStdOut().writer();
    try stdout.print("Hello, {s}!\n", .{"world"});
}
```
