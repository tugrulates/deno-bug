# Exponential work in `deno lint .` and `deno fmt .` with workspaces

## Bug

Passing “.” to either `deno lint` or `deno fmt` processes each file once for
each [workspace](https://docs.deno.com/runtime/fundamentals/workspaces/) member.
If the workspace has _N_ members, each file in the project is either linted or
formatted _N_ times.

Easy to observe on [std](https://github.com/denoland/std) repo.

```sh
deno fmt
```

```
Checked 1138 files
```

```sh
deno fmt .
```

```
…
thread 'tokio-runtime-worker' has overflowed its stack
fatal runtime error: stack overflow, aborting
[1]    10949 IOT instruction (core dumped)  deno fmt .
```

## Example

A minimal example is like the following.

<details>
<summary>
./deno.json
</summary><code>{
  "workspace": [
    "./member1",
    "./member2"
  ]
}
</code>
</details>

<details>
<summary>
./member1/deno.json
</summary><code>{
  "name": "member1",
  "exports": {
    ".": "./member1.ts"
  }
}
</code>
</details>

<details>
<summary>
./member1/member1.ts
</summary><code>const x = 1;
</code>
</details>

<details>
<summary>
./member2/deno.json
</summary><code>{
  "name": "member2",
  "exports": {
    ".": "./member2.ts"
  }
}
</code>
</details>

<details>
<summary>
./member2/member2.ts
</summary><code>// empty
</code>
</details>

### `deno lint` behavior

```sh
deno lint
```

```
error[no-unused-vars]: `x` is never used
 --> /Users/tugrul/Code/deno-bug/member1/member1.ts:1:7
  |
1 | const x = 1;
  |       ^
  = hint: If this is intentional, prefix it with an underscore like `_x`

  docs: https://docs.deno.com/lint/rules/no-unused-vars


Found 1 problem
Checked 2 files
```

```sh
deno lint .
```

```
error[no-unused-vars]: `x` is never used
 --> /Users/tugrul/Code/deno-bug/member1/member1.ts:1:7
  |
1 | const x = 1;
  |       ^
  = hint: If this is intentional, prefix it with an underscore like `_x`

  docs: https://docs.deno.com/lint/rules/no-unused-vars


error[no-unused-vars]: `x` is never used
 --> /Users/tugrul/Code/deno-bug/member1/member1.ts:1:7
  |
1 | const x = 1;
  |       ^
  = hint: If this is intentional, prefix it with an underscore like `_x`

  docs: https://docs.deno.com/lint/rules/no-unused-vars


Found 2 problems
Checked 4 files
```

### `deno fmt` behavior

```sh
deno fmt
```

```
Checked 6 files
```

```sh
deno fmt .
```

```
Checked 14 files
```
