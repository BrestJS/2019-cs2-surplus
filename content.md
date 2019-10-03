CoffeeScript(2)
-------------

### History

- first release 24 december 2009 (jashkena)
- lots of features made it into ECMAScript (`=>`, `class`, …)
- v2 = community release (2017): `await`, JSX, …
- new features = TC39 Stage 4 (`BigInt` → ``` `1n` ```)

### Syntax

```coffeescript
my_fun = (modes,{define},more...) ->
  for a in modes
    define a, more...
```

- Litterate CoffeScript (Markdown)
```coffeescript
Calcule un carré
----------------

    square = (x) -> x*x
```

### Iterators

```coffeescript
for a in [0..10] by 2
  assert a % 2 is 0
```

```coffeescript
cat =
  name: 'kitty'
  color: 'beige'
  
for own k,v of cat
  console.log "cat has #{v} #{k}"
```

### Generators

- generator: object with `.next()` → `{value,done}`
- generator function: returns a generator

```coffeescript
f = -> yield 42
v = f()

v.next()
→ { value: 42, done: false }
v.next()
→ { value: 42, done: false }
```

### Generators (2)

```coffeescript
natural = ->
  num = 0
  loop
    num += 1
    yield num
  return

perfectCubes = ->
  for num from natural()
    yield num*num*num
  return

do ->
  for v from perfectSquares()
    console.log v
```

### Promise (1)

- Promise: object with `.then( on_success, on_failure )`

```coffeescript
p = Promise.resolve 6
p.then (value) -> assert value is 6
```

```coffeescript
p = new Promise (resolve) -> resolve 7
p.then (value) -> assert value is 7
```

```coffeescript
p = new Promise.reject new Error 'oops'
p.catch (error) -> assert error.message is 'oops'
```

### Promise (2)

- `await` will interrupt execution and wait for the Promise to resolve

```coffeescript
f = ->
  value = await Promise.resolve 42
  assert value is 42
f()
```

### Promise (3)

- shortest useful Promise

```coffeescript
sleep = (timeout) -> new Promise (resolve) -> setTimeout resolve, timeout

do ->
  console.log 'Ready?'
  await sleep 1000
  console.log 'Done.'
```

### Async function (1)

- async function: function that returns a Promise

```coffeescript
f = -> Promise.resolve 42
f().then (value) -> assert value is 42
```

### Async function (2)

- a function containing `await` is itself an async function

```
f = -> Promise.resolve 7

g = (x) -> x * await f()

do ->
  assert 42 is await g()
```

### Async function (3)

```coffeescript
agent = require 'superagent'
do ->
  {text} = await agent.get 'http://stephane.shimaore.net/'
  console.log text.length
```

### Async function (4)

```coffeescript
fs = (require 'fs').promises
{mtime} = await fs.stat '/etc/motd'
```

### Async generators (1)

- async generator: generator that yields Promises

```coffeescript
f = (names) ->
  for name in names
    {mtime} = await fs.stat name
    yield [name,mtime]
  return
  
do ->
  for await mtime from f(['/etc/motd','/etc/passwd'])
    console.log mtime
# →
# [ '/etc/motd', 2018-01-13T23:36:34.000Z ]
# [ '/etc/passwd', 2019-09-09T17:41:15.752Z ]
```

### Async generators (5)

- example: transform an API with pagination into a continuous stream of documents

```coffeescript
findAsyncIterable = (params,limit = 10) ->
  until done      # iterate until we retrieve all data
    body = null
    until body?   # iterate until we get a valid response
      try
        {body} = await agent.post buildFindURL(params,limit)
      catch error
        if error.status is 404  # invalid query
          body = docs: []
        else
          await sleep 100

    {docs} = body
    for doc from docs
      yield doc

    done = docs.length < limit
  return

do ->
  asyncQuery = findAsyncIterable selector: account: 'bob'
  for await v from asyncQuery
    console.log v
```

### Async generators (6)

- migration Object-Stream → générateur

Surplus
-------

(blabla startup, S.js, JSX, JSX in CS2)
- `await` → exécution synchrone (avant) vs asynchrone
- `do` (`toe`) → keep track of rejected Promises
