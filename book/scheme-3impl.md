# Three Implementation Models for Scheme

---

http://www.cs.indiana.edu/~dyb/pubs/3imp.pdf

---

感觉看错资料了。
讲的是如何把 scheme 编译成 instruction，然后放到 VM 上执行。

---

## Introduction && The Scheme Language

---

介绍了三种模型

- heap-based model
- stack-based model
- string-based model

区别在于如何存储 `call frames and variable bindings`。

---

- lexically-scoped
- block-structured
- closures
- continuations
- λ-calculus
- weakly-typed

作者列举了一些 scheme 的特点

closures, assignments, and continuations

---

`(((call/cc (lambda (c) c)) (lambda (x) x)) 'HEY!)`

> This is probably the most confusing Scheme program of its size!

（其实我感觉下面这个更难懂
`((let ([comeback (call/cc (lambda (c) c))]) (comeback (lambda (x) x))) 'HEY!)`

---

## The Heap-Based Model

---

- caller build call frame
    - actual parameters
    - link to caller's stack frame
    - return address
- caller jump to callee
    - push local variables to stack
- callee return to caller
    - reset the frame link
    - remove the frame
    - jump to the return address

> the state of each active call is recorded on the stack, and this state is
> destroyed once the call has been completed.

stack-based 的一般流程。
每次函数调用都是创建一个新的 call frame。
函数返回就是销毁当前 frame 并跳转回之前的位置。

但是 scheme 要求支持 continuation，前面这种直接销毁 frame 的方式无法满足要求。
所以考虑用 heap，把 call frame 和 environment 都放到 heap 上。

> Recall that a continuation is a closure that, when invoked, returns control to
> the point where the continuation was obtained.

---

- data structure to support the core of scheme
    - environment
        - a list of pairs of lists, `env = (var list, val list) list`
        - 提及了一个优化，将 variables 干掉，只保留 values。eopl 里的 nameless 吗
    - call frame
        - pending computation
        - `call frame = (next_expression, env, rib, next_frame)`
        - 上面的 rib 是用于记录已经计算出的 argument 的值
    - control stack
        - `stack = frame list`
        - current -> previous -> previous previous
    - closure
        - `closure = (body, env, var list)` / `closure = (body, env)`
    - continuation
        - a special closure containing a reference to the current frame

---

> in order not to build up the control stack on a tail call, a new call frame is
> not added to the stack.

EOPL 里反复提及这点。
tail position 进行函数调用，可以不增加额外的 control context。

---

source
↓ compile
instruction
↓ VM
value

有价值的地方，大概是把 VM 和 compile 看懂。
关键大概是 instruction 的设计。
都是大概，因为还没看懂。
状态不好，读不下去。

---

第五节做的是优化

> compute for each variable its location in the environment, so that the virtual
> machine can go straight to this location to find the variable’s value without
> searching along the way.

想要达成的效果，是简化 environment 的查找。

> Scheme variables are statically scoped.
> This means that the binding of any variable is apparent in the static
> structure of the program.

支持 static scope 是进行这种优化的条件。

> The improvement given here is substantial and important
这种优化似乎效果很明显？
然后 stack-based 的模型好像还能更好。

---

## The Stack-Based Model

---

在 heap 模型下，一半以上的时间都在查找变量、执行函数调用。

---

> Functions are allowed as parameters, but they are not first class objects
> since they cannot be reliably retained and invoked after the defining scope
> no longer exists.

（可以作为参数，却不是 first-class 的函数

---

- call stack 的结构 （下面这种 push 顺序，便于 tail call 优化）
    - static link (pushed last)
        - frame pointer
        - point to the the called function
        - a pointer to the next rib of an environment
    - last argument ... first argument
    - next expression
        - return address
    - dynamic link (pushed first)
        - frame pointer
        - point to the caller's frame
        - a pointer to the next frame in the control stack

> Separation of the dynamic and static chains facilitates the support of
> closures and tail calls.

> call frames and variable bindings are placed on a true stack rather than in
> a heap-allocated linked structure.

---

## Conclusions

---

> principle: it is important to support the most commonly-used features with the
> greatest possible efficiency even if this means supporting the less common
> features with less than optimal efficiency.

> principle: if a language feature is difficult to implement efficiently, and if
> the choice is between supporting the feature inefficiently and leaving it out,
> the choice should be to support it inefficiently (assuming the feature is
> worth having at all).
