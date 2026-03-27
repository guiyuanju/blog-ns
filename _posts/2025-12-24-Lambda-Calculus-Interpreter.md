---
title: "Lambda Calculus Interpreter"
date: 2025-12-24
layout: post
slug: lambda-calculus-interpreter
---

A Lambda Calculus Interpreter written in Go compiled to WASM, see source on [Github](https://github.com/guiyuanju/lamb).

<details>
  <summary>Click to Show Syntax</summary>
  <div markdown="1">
Function/abstraction: `(\x.x)`, `(\x.(\y.y x))`

Application: `((\x.x y) y)` => `(y y)`.

Let syntax sugar: `let x = y in ...`, replace all y with x, can be nested like `let a = A in let b = B in (a b)` => `(A B)`

  </div>
</details>

<details>
  <summary>Click to Show Examples</summary>
  <div markdown="1">
A function that returns its argument:

> (\x.x) y

Infinite self-application function, but interpreter detected and prevent the infinite evaluation:

> (\x.x x) (\x.x x)

Define number zero in Church encoding and successor function, then calculate the successor of 0:

> let 0 = \f.\x.x in let succ = \n.\f.\x.f (n f x) in succ 0

Define number from 0 to 9, + - \* /, if, and, or, not, and Y combinator and then calculate 6 \* 7:

> let succ = \n.\f.\x.f (n f x) in let pred = \n.\f.\x.n (\g.\h.h (g f)) (\u.x) (\u.u) in let 0 = \f.\x.x in let 1 = succ 0 in let 2 = succ 1 in let 3 = succ 2 in let 4 = succ 3 in let 5 = succ 4 in let 6 = succ 5 in let 7 = succ 6 in let 8 = succ 7 in let 9 = succ 8 in let + = \m.\n.\f.\x.m f (n f x) in let - = \m.\n.n pred m in let \* = \m.\n.m (+ n) 0 in let pow = \b.\n.n b in let true = \x.\y.x in let false = \x.\y.y in let and = \p.\q.p q p in let or = \p.\q.p p q in let not = \p.p false true in let if = \p.\a.\b.p a b in let zero? = \n.n (\x.false) true in let <= = \m.\n.zero? (- m n) in let Y = \g.(\x.g (x x)) (\x.g (x x)) in (\* 6 7)

  </div>
</details>

<br/>

<style>
/* Container fills available space */
.repl {
    display: flex;
    flex-direction: column;
    height: 50vh;
    width: 100%;
    background: #0f0f0f;
    color: #eaeaea;
    font-family: monospace;
    font-size: 14px;
}

/* Scrollable output area */
.repl-output {
    flex: 1;
    overflow-y: auto;
    overflow-x: auto; 
    padding: 12px;
    white-space: pre; 
    /* white-space: pre-wrap; */
}

/* Each entry */
.repl-entry {
    margin-bottom: 10px;
}

/* Input line fixed at bottom */
.repl-input-line {
    display: flex;
    padding: 10px;
    border-top: 1px solid #333;
    background: #111;
    color: black;
}

/* Prompt */
.prompt {
    margin-right: 8px;
    color: white;
}

/* Input field */
#stdin {
    flex: 1;
    background: transparent;
    border: none;
    color: inherit;
    font-family: inherit;
    font-size: inherit;
    outline: none;
}
</style>

<div class="repl">
    <div class="repl-output" id="stdout"></div>

    <div class="repl-input-line">
        <span class="prompt">λ</span>
        <input type="text" id="stdin" autofocus />
    </div>

</div>

<script src="/assets/js/wasm_exec.js"></script>

<script>
const go = new Go();

WebAssembly.instantiateStreaming(
    fetch("/assets/wasm/la.wasm"),
    go.importObject
).then((result) => {
    go.run(result.instance);
});

const stdin = document.getElementById("stdin");
const stdout = document.getElementById("stdout");

/* Scroll to bottom */
function scrollToBottom() {
    stdout.scrollTop = stdout.scrollHeight;
}

/* Append a REPL entry */
function appendEntry(input, result) {
    const entry = document.createElement("div");
    entry.className = "repl-entry";

    const line = document.createElement("div");
    line.innerHTML = `<span class="prompt">λ</span>${input}`;

    const output = document.createElement("div");
    output.textContent = result.replace(/\n+$/, ""); // trim trailing newlines

    entry.appendChild(line);
    entry.appendChild(output);

    stdout.appendChild(entry);
}

/* Handle input */
stdin.addEventListener("keydown", function (e) {
    if (e.key === "Enter") {
        e.preventDefault();

        const input = stdin.value;
        if (!input) return;

        let result;
        try {
            result = run(input);
        } catch (err) {
            result = String(err);
        }

        appendEntry(input, result);

        stdin.value = "";
        scrollToBottom();
    }
});

/* Keep focus on input */
// document.addEventListener("click", () => {
//     stdin.focus();
// });
</script>
