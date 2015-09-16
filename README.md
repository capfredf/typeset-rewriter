# typeset-rewriter
Simple, low level rewriters for PLT Redex typesetting.

See the PLT Redex docs on [typesetting](http://docs.racket-lang.org/redex/The_Redex_Reference.html?q=redex#%28part._.Typesetting%29) for more details. These macros are just a thin layer on top of these mechanics.


Provides the following syntax:

### rw

Allows for simple compound-rewriter definitions (refer to redex docs for specific requirements on
what a compound rewriter is). Basically, a compound-rewriter for a symbol 'name'
will be called for redex terms being typeset that look like `(name any ...)`

The `rw` macro builds rewriters from quasiquote like patterns of the form
```racket (rw [`(name pats ...) => output] ...) ```

Things not unquoted are matched as literal symbols in the redex term, while things
unquoted are treated like standard match patterns.

Example usage:
```racket
(define lambda-rw
    (rw [`(lambda ([,x : ,t]) ,body)
         => (list "" "λ" x ":" t ". " body)]
        [`(lambda ([,x : ,t]) ,body ,bodies ...)
         => (list* "" "λ" x ":" t ". (begin " body (append bodies (list ")")))]))
```

This defines a rewriter with two cases, which together allow this rewriter 
to successfully rewrite PLT Redex terms of the form 
`(lambda ([any : any]) any any ...)`.
Our particular definition removes the parens from around the lambda and
adds a begin if there is more than one body expression. In other words, 
`(lambda ([x : t]) e)` is typeset to look like `λx:t.e`, 
and `(lambda ([x : t]) e e)` is typeset to look like `λx:t.(begin e e)`.

Note that the sub-terms we are not altering but merely rearranging (_e.g. t_) 
will have any other applicablerewriters applied to them as well. So if we
had an atomic rewriter converting `t` to `τ`, that would still occur, etc...

For more info on compound rewriters, see the redex docs. They are supposed to be functions 
with signature `(listof lw) -> (listof (or/c lw? string? pict?))` which is
"rewritten appropriately."

### with-atomic-rewriters

Like `with-compound-rewriters` for atomic rewriters.

Example usage:
```racket
(with-atomic-rewriters (['lambda "λ"] ['-> "→"]) 
 body-expr)
```

### with-rewriters

Allows for atomic and/or compound rewriters to be specified.

Example usage:
```racket
(with-rewriters
 #:atomic (['Env "Γ"] ['exp "e"] ['ty "τ"] ['-> "→"] ['integer "n"])
 #:compound (['lambda lambda-rw]
             ['typeof typeof-rw]
             ['extend extend-rw]
             ['lookup lookup-rw]
             ['val-type val-type-rw])
 body-expr)
```

### define-rw-context

Defines a macro to reuse sets of rewriters.

Example usage:
```racket
(define-rw-context with-rws
  #:atomic (['Env "Γ"] ['exp "e"] ['ty "τ"] ['-> "→"] ['integer "n"])
  #:compound (['lambda lambda-rw]
              ['typeof typeof-rw]
              ['extend extend-rw]
              ['lookup lookup-rw]
              ['val-type val-type-rw]))
              
(with-rws render-expr1)
(with-rws render-expr2)
```

