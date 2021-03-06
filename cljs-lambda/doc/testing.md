# Testing

While it's strongly suggested that Lambda functions are minimal abstractions of
the execution environment (i.e. input/output processing only, delegating to
generic Clojurescript functions), there are times when it's going to make sense
to test the entry points off of EC2.

[[cljs-lambda.local/invoke]] invokes a Lambda handler with the supplied event
(optional, nil), supplying a context suitable for local evaluation.  The context
record'll contain superficially plausible values for the context's string keys,
and invoke will ensure that the eventual result of the invocation, however
delivered, will be conveyed to the caller via a promise.

[[cljs-lambda.local/channel]] behaves identically, with the exception of placing
the result on a `core.async` channel, in a vector tagged with either `:success`
or `:fail`.

### Example

```clojure
(deflambda testable [x ctx]
  (go
    ;; TODO Think hard
    (inc x)))

(invoke  testable 5) ;; => <Promise 6>
(channel testable 5) ;; => <Channel [:succeed 6]>
```

### Example

```clojure
(def fs (.promisifyAll promesa/Promise (nodejs/require "fs")))

(deflambda read-file [path ctx]
  (.readFileAsync fs path))

(invoke read-file "static/config.edn") ;; => <Promise "{:x ...">
```

### Example

```clojure
(deflambda identify [caller-name {my-name :function-name}]
  (str "Hi " caller-name " I'm " my-name))

;; The default context values aren't particularly exciting
(invoke identify "Mary") => ;; <Promise "Hi Mary, I'm functionName">
;; But custom values may be supplied
(invoke identify "Mary" (->context {:function-name "identify"}))
;; => ;; <Promise "Hi Mary, I'm identify">
```

## Environment Variables

When testing functions which retrieve environment variables via
[[cljs-lambda.context/env]], alternate values may be supplied in a test's
context object.

```clojure
(deflambda env [k ctx]
  (ctx/env ctx k))

(invoke read-file "USER" (->context {:env {"USER" "moe"}}))
```

When deployed, `ctx/env` would read the value from Node's `process.env`.
