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

Define number from 0 to 9, + - _ /, if, and, or, not, and Y combinator and then calculate 6 _ 7:

> let succ = \n.\f.\x.f (n f x) in let pred = \n.\f.\x.n (\g.\h.h (g f)) (\u.x) (\u.u) in let 0 = \f.\x.x in let 1 = succ 0 in let 2 = succ 1 in let 3 = succ 2 in let 4 = succ 3 in let 5 = succ 4 in let 6 = succ 5 in let 7 = succ 6 in let 8 = succ 7 in let 9 = succ 8 in let + = \m.\n.\f.\x.m f (n f x) in let - = \m.\n.n pred m in let \* = \m.\n.m (+ n) 0 in let pow = \b.\n.n b in let true = \x.\y.x in let false = \x.\y.y in let and = \p.\q.p q p in let or = \p.\q.p p q in let not = \p.p false true in let if = \p.\a.\b.p a b in let zero? = \n.n (\x.false) true in let <= = \m.\n.zero? (- m n) in let Y = \g.(\x.g (x x)) (\x.g (x x)) in (\* 6 7)

  </div>
</details>

<style>
    .repl {
        /* padding: 10px; */
        font-family: monospace;
        overflow-y: auto;
        box-sizing: border-box;
        height: 100vh;
        font-size: 16px;

        /* Hide scrollbar (cross-browser) */
        scrollbar-width: none;
        /* Firefox */
        -ms-overflow-style: none;
        /* IE/Edge */
    }

    .repl::-webkit-scrollbar {
        display: none;
        /* Chrome, Safari */
    }

    .repl-output {
        font-family: monospace;
        white-space: pre-line;
        margin-bottom: 10px;
        font-size: 16px;
    }

    .repl-input-line {
        font-family: monospace;
        font-size: 16px;
        display: flex;
    }

    .prompt {
        font-family: monospace;
        font-size: 16px;
        margin-right: 10px;
    }

    #stdin {
        background: transparent;
        /* Make input background transparent */
        border: none;
        /* Remove default input border */
        font-family: monospace;
        font-size: 16px;
        flex-grow: 1;
        /* Input takes up remaining space */
        outline: none;
        /* Remove blue highlight on focus */
    }
</style>
<div class="repl" id="repl">
    <div class="repl-output" id="stdout">
    </div>
    <div class="repl-input-line">
        <span class="prompt">λ</span>
        <input type="text" id="stdin" autofocus>
    </div>
</div>

<script src="/assets/js/wasm_exec.js"></script>

<script>
    const go = new Go();
    WebAssembly.instantiateStreaming(fetch("/assets/wasm/la.wasm"), go.importObject).then((result) => {
        go.run(result.instance);
    });

    let repl = document.getElementById("repl")
    let stdin = document.getElementById("stdin")
    let stdout = document.getElementById("stdout")
    stdin.addEventListener("keydown", function (e) {
        if (e.key === "Enter") {
            e.preventDefault()

            const input = stdin.value
            const res = run(input)

            const entry = document.createElement("div")
            entry.innerHTML = `<span class="prompt">λ</span>${input}<br/><pre>${res}</pre>`

            stdout.appendChild(entry)

            stdin.value = ""

            const repl = document.querySelector(".repl")
            repl.scrollTo({
                top: repl.scrollHeight,
                behavior: "smooth"
            })
        }
    })

</script>
