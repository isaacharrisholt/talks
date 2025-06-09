---
theme: default
title: Your backend is too complicated
info: _Isaac Harris-Holt, Lambda Days 2025_
class: bg-[#ffaff3]
transition: none
mdc: true
layout: cover
colorSchema: light
fonts:
  provider: coollabs
  sans: Lato
  mono: 'JetBrains Mono,Fire Code,IBM Plex Mono'
  local: JetBrains Mono
defaults:
  layout: center
---

# **Your backend<br>is too complicated**

Isaac Harris-Holt

<!--
- Welcome people
  - Last talk before keynote and beer, will try not to run too long
-->

---
---

<div class="flex flex-col gap-16 max-h-full">
<img src="/rover-logo-transparent.svg" class="w-1/4 mx-auto"/>

<div class="grid grid-cols-2 gap-24 text-xs place-items-center">
```gleam
fn handle_message(message: Message(error), state: State(error)) {
  let State(self:, barnacle:, timer:) = state
  case message {
    Refresh(return) -> {
      let refresh_result = refresh_nodes(barnacle)

      cancel_timer(timer)
      let timer =
        process.send_after(self, barnacle.poll_interval, Refresh(None))

      send_response(return, refresh_result)
      send_response(barnacle.listener, RefreshResponse(refresh_result))
      actor.continue(State(..state, timer: Some(timer)))
    }
  }
}
```

<img src="/argus-graph.png" class="rounded-lg h-80 object-contain">
</div>
</div>

<style>
  pre {
    --slidev-code-font-size: 8px;
    --slidev-code-line-height: 12px;
    --slidev-code-padding: 8px;
    --slidev-code-radius: 4px;
  }
</style>

<!--
- So, who am I?
- I'm the CTO at Rover, where we turn code like this into graphs like this and look
  for bad patterns to make sure bugs don't happen
-->

---
layout: cover
class: bg-slide-dark
---

# **Let's imagine...**

<!--
- If I told you, right now, to go and build me a backend API for a new project idea, what technology would you choose?
- Got one?
- Great
- But now, I also need you to cache external API calls, maintain multiple caches, and manage a long-running background process that reads and writes from these caches at regular intervals.
-->

---
class: max-w-1/2 mx-auto
---

![Redis logo](https://upload.wikimedia.org/wikipedia/commons/thumb/6/64/Logo-redis.svg/2560px-Logo-redis.svg.png)

<!--
- Suddenly, you've added Redis to your stack, along with a background processing library
-->

---
---

<img src="https://media.makeameme.org/created/we-replaced-our.jpg" class="place-self-center rounded-lg">

<!--
- And some of you have already started thinking about microservices, I can tell
-->

---
class: bg-slide-dark
---

<img v-click src="/rust-logo-white.png" class="h-80">

<!--
- Great, I have one last requirement
- This API must *never* crash
- Damn
[click]
- Time to rewrite it in Rust, I guess
-->

---
class: bg-[#ffaff3]
---

<img v-click src="/gleam-lucy.svg" class="h-80 -rotate-10">

<!--
- Well, there’s a better way
- And that way, is a type-safe, simplicity-first programming language expertly baked for massive concurrency, high availability and effortless scaling
[click]
- I am, of course, talking about Gleam
-->

---
layout: full
class: p-0
---

<img src="/gleam-website.png">

<!--
- Gleam describes itself as a friendly language for building type-safe systems that scale
- Let’s break that down a little
-->

---
---

```gleam
import gleam/io
import gleam/list

fn print_best_mons() {
  ["zorua", "umbreon", "lucario"]
  |> list.map(io.println)
}

fn main() {
  print_best_mons()
}
```

<!--
- It’s friendly, with a simple, familiar, C-like syntax
-->

---
class: 'bg-faff font-mono text-3xl text-center'
---

as assert auto case const<br><br>delegate derive echo else fn<br><br>if implement import let macro<br><br>opaque panic pub test todo<br><br>type use

<!--
- It’s also very small, with only 22 reserved keywords...
-->

---
class: 'bg-faff font-mono text-3xl text-center'
---

as assert <span class="opacity-50">auto</span> case const<br><br><span class="opacity-50">delegate derive</span> echo <span class="opacity-50">else</span> fn<br><br>if <span class="opacity-50">implement</span> import let <span class="opacity-50">macro</span><br><br>opaque panic pub <span class="opacity-50">test</span> todo<br><br>type use

<!--
- ...only 15 of which are actually in use!
-->

---
---

````md magic-move
```gleam
fn calculate_total_cost(
  quantity: Float,
  price: Float,
  discount: Float,
) -> Float {
  let subtotal = quantity *. price
  let discount = subtotal *. discount
  subtotal -. discount
}
```

```gleam
fn calculate_total_cost(
  quantity,
  price,
  discount,
) {
  let subtotal = quantity *. price
  let discount = subtotal *. discount
  subtotal -. discount
}
```
````

<!--
- And it’s type safe, so the compiler will do what it can to make sure your programs are correct
[click]
- Though type annotations can usually be dropped without hampering that crucial type safety
-->

---
layout: two-cols
class: bg-slide-dark
---

<div v-click class="grid place-items-center w-full h-full">
    <img src="/javascript.png" class="h-80 place-self-center my-auto">
</div>

::right::

<div v-click class="grid place-items-center w-full h-full">
    <img src="/erlang-white.png" class="h-80 place-self-center">
</div>

<!--
- Importantly, Gleam is for systems that scale
- Gleam doesn’t have its own runtime
[click]
- Gleam code can compile to JavaScript...
[click]
- ...but it’s most often compiled to Erlang to run on the BEAM virtual machine
- The BEAM is an incredible piece of technology designed to power global systems like comms networks, and Gleam can take full advantage of that to go planet scale
-->

---
layout: cover
class: bg-slide-dark
---

# **The project**

<!--
- This talk is going to be centred around a fictional web server project
- It’s a simple API for getting data about Pokemon
- You don’t need to know anything about Pokemon to understand this talk
-->

---
---

```http
GET /pokemon/:name

GET /battle/:name_1/:name_2
```

<!--
- We’re just building an API that has two endpoints
- `/pokemon/:name` for getting information about a Pokemon, its stats and its moves, and `/battle/:name_1/:name_2` to get the results of a battle between two Pokemon
-->

---
layout: cover
class: bg-slide-dark
---

# **Types**

<!--
- Let's talk types
-->

---
layout: two-cols
---

<div v-click class="grid place-items-center w-full h-full">
    <img src="/python.svg" class="h-80 place-self-center my-auto">
</div>

::right::

<div v-click class="grid place-items-center w-full h-full">
    <img src="/typescript-types-doom.jpg" class="w-fit place-self-center object-contain rounded-lg">
</div>

<!--
- Modern languages often have a static type system that lies somewhere on a scale of
[click]
- “type suggestions that are there but don’t really do anything”
[click]
- to “Turing-complete”
-->

---
class: bg-[#ffaff3]
---

<img src="/gleam-lucy-happy.svg" class="h-80 -rotate-10">

<!--
- Gleam is not like that
- For the academics in the room, Gleam uses a nominal type system based on Hindley-Milner
- For everyone else, we’re using cutting edge technology from the 1970s
-->

---
---

```gleam
let hp: Int = 64
let name: String = "Gible"
let types: #(String, String) = #("Dragon", "Ground")
let moves: List(Move) = [...]
```

<!--
- Gleam has all your standard primitives for things like strings, ints, floats, booleans, and so on
- And of course it has lists and tuples
-->

---
---

```gleam
type Pokemon {
	Pokemon(name: String, type_: PokemonType)
}
```

```gleam
let pikachu = Pokemon("Pikachu", type_: Electric)
```

<!--
- But if you want custom types, you’ll be writing code like this
- This is a record type
- In this example we have a type called `Pokemon` with a single constructor, also called `Pokemon`
-->

---
---

````md magic-move
```gleam
type Interactable {
  Character(name: String, dialogue: List(String))
  TrashCan(item: Item)
}
```

```gleam
type Interactable {
  Character(name: String, dialogue: List(String))
  TrashCan(item: Item)
  Tree
  MoveableRock
}
```
````

<!--
- Some types have multiple constructors, like an interactable object we might see in a Pokemon game
[click]
- And it’s possible for constructors to take no arguments
-->

---
---

```gleam {1,2,6|1,4,5,6|all}
type Interactable {
  Character(name: String, dialogue: List(String))
  TrashCan(item: Item)
  Tree
  MoveableRock
}
```

<!--
- So, generally, if you want to represent something akin to a struct in other languages, you’d use a record type with a single constructor

[click]

- If you want to represent a typical enum, you’d have multiple constructors with no parameters

[click]

- And you can mix and match the two to your heart’s content
-->

---
---

```gleam
case interactable {
	Character(name, dialogue) -> handle_character_interaction(name, dialogue)
	TrashCan(item) -> add_item_to_inventory(item)
	MoveableRock -> push_rock()
	Tree -> bonk()
}
```

<!--
- If you want to know which constructor was used to construct a value, you’d use pattern matching like this
- And what’s really nice is that the compiler will scream at you if you miss any patterns
-->

---
---

```gleam
type Result(a, b) {
	Ok(a)
	Error(b)
}
```

<!--
- Gleam also has a built-in record type called Result, which is a generic type defined like this
- And this is used for error handling throughout Gleam code, rather than throwing exceptions, so we know potential error sites at a glance
-->

---
---

```gleam
type Pokemon {
	Pokemon(
		name: String,
		types: PokemonTypes,
		moves: List(Move),
		stats: Stats,
	)
}
```

```json
{
	"name": "Murkrow",
	"types": ["dark", "flying"],
	"moves": [{...}, {...}],
	"stats": {"hp": 60, ...}
}
```

<!--
- Cool, so we can imagine we have a Pokemon type that looks a bit like this
- Which might be represented like this in our JSON API
-->

---
class: bg-slide-dark
---

<!--
- Moving on
- We’re building a web server, so we’re going to need some...
-->

---
layout: cover
class: bg-slide-dark
---

# **Concurrency**

<!--
- ...concurrency
- As I mentioned earlier, Gleam on the backend typically runs on the BEAM virtual machine
- Concurrency on the BEAM probably works a little differently to what you’re used to
-->

---
layout: default
class: bg-slide-dark
---

<img src="/javascript-event-loop.png" class="rounded-lg h-full place-self-center">

<!--
- Instead of a JavaScript-style event loop, or managing communication between threads with a mutex and a prayer...
-->

---
layout: default
class: bg-slide-dark
---

<img src="/erlang-beam-process.png" class="h-full place-self-center">

<!--
- ...the BEAM runs on a number of processes
- Processes don’t have shared memory or a shared callstack
- You interact with other processes by sending them messages
-->

---
---

````md magic-move
```gleam
process.spawn(fn() {
	do_some_work()
})
```

```gleam
let subj = process.new_subject()
process.spawn(fn() {
	process.send(subj, "Hello, Kraków!")
})

let receive_result = process.receive(subj, within: 100)
// Ok("Hello, Kraków!")
```
````

<!--
- In Gleam, spawning a new process looks like this
- If you want to receive messages, you create what’s called a subject
- Other processes can then send messages to that subject for you to receive

[click]

- That looks like this
- Processes on the BEAM are *really* cheap to create, and they’re incredibly lightweight
- You can have *thousands* of them running at any one time, and larger Erlang, Elixir or Gleam applications often do
-->

---
---

```json {all|7|all}
{
	"name": "ditto",
	"moves": [
		{
			"move": {
				"name": "transform",
				"url": "https://pokeapi.co/api/v2/move/144/"
			}
		}
	],
	...
}
```

<!--
- In our case, we’re getting data from an existing API called the PokeAPI
- It follows REST standards reasonably strictly, so when you GET a Pokemon, rather than getting all the details about its moves,
[click]
you get links to them instead

[click]

- We want more details about the move, like its type and base power, so we’re going to have to make an additional request per move returned
- Rather than waiting for each one to complete, we want to do that concurrently!
-->

---
---

```gleam {all|1,3}
let assert Ok(api_pokemon) = fetch_pokemon("zorua")

let assert Ok(moves) =
	api_pokemon.moves
	|> parallel_map(
		fetch_move,
		1000,
	)
```

<!--
- This code uses the a `parallel_map` function to quickly iterate over the list of moves in parallel
- Gleam will happily run all of these requests concurrently without breaking a sweat

[click]

- By the way, this `let assert` syntax just crashes if a variable doesn’t match a pattern
- In application code, that’s absolutely fine, and we’ll get into why later
-->

---
layout: cover
class: bg-slide-dark
---

# **Caching**

<!--
- Okay, I promised you you’d be able to replace a lot of your infrastructure, so let’s take a look at caching
-->

---
layout: default
class: bg-slide-dark
---

<img src="/ets-docs.png" class="max-h-full place-self-center rounded-lg">

<!--
- One of the other awesome features of the BEAM is Erlang Term Storage, or ETS
- It’s an in-memory database that allows you to store *any* Erlang value, and therefore any Gleam value, not just anything JSON serialisable
-->

---
---

```gleam
import carpenter/table

type Cache(value) = table,Set(String, value)
```

<!--
- We’re going to use the `carpenter` package, which is just a Gleam wrapper around Erlang’s ETS bindings
- A cache is just going to be a Carpenter table with string keys
- Carpenter includes functions to create new sets, insert and remove items, etc.
-->

---
---

```gleam {all|6|8-16|9|10-11|all}
pub fn read_through(
	cache: Cache(value),
	key: String,
	getter: fn() -> Result(value, error),
) -> Result(value, error) {
	case table.lookup(key) {
		[value, ..] -> Ok(value)
		[] -> {
			case getter() {
				Ok(value) -> {
					insert(cache, key, value)
					Ok(value)
				}
				Error(error) -> Error(error)
			}
		}
	}
}
```

<!--
- I’m going to add a `read_through` function that will try to

[click]

- get a value from the cache

[click]

- and in the case where it’s not present

[click]

- it will run a function to get that value

[click]

- caching it if successful

[click]

- This code works really well, but it’s so ugly
- Just look how many levels of indentation we have here!
- It’s time to make use of another fun Gleam trick
-->

---
class: 'bg-faff font-mono text-3xl text-center'
---

as assert auto case const<br><br>delegate derive echo else fn<br><br>if implement import let macro<br><br>opaque panic pub test todo<br><br>type use

<!--
- Remember those 22 keywords I mentioned earlier?
-->

---
class: 'bg-faff font-mono text-3xl text-center'
---

as assert auto case const<br><br>delegate derive echo else fn<br><br>if implement import let macro<br><br>opaque panic pub test todo<br><br>type <u>**use**</u>

<!--
- My favourite one is the use keyword
-->

---
---

````md magic-move
```gleam
fn my_func(args, fn(params) {
  operations
})
```

```gleam
use params <- my_func(args)
operations
```
````

<!--
- It’s just a bit of syntactic sugar, but what it allows us to do

[click]

- is flatten out our code when we have a function whose last argument is a callback
-->

---
---

````md magic-move
```gleam
pub fn read_through(
	cache: Cache(value),
	key: String,
	getter: fn() -> Result(value, error),
) -> Result(value, error) {
	case table.lookup(key) {
		[value, ..] -> Ok(value)
		[] -> {
			case getter() {
				Ok(value) -> {
					insert(cache, key, value)
					Ok(value)
				}
				Error(error) -> Error(error)
			}
		}
	}
}
```

```gleam
pub fn read_through(
	cache: Cache(value),
	key: String,
	getter: fn() -> Result(value, error),
) -> Result(value, error) {
	use _ <- result.try_recover(table.lookup(key) |> list.first)
	use value <- result.try(getter())
	insert(cache, key, value)
	Ok(value)
}
```
````

<!--
- If we combine this with some of the functions from the gleam/result module in the standard library, we can turn this

[click]

- Into this
- The `try` and `try_recover` functions allow you to update the Ok or Error value in a result by passing it into another result-returning function
- Combined with use, we’re immediately back to one level of nesting, and it’s much easier to grok the happy path of the code
-->

---
---

````md magic-move
```gleam
pub fn fetch_pokemon(cache: Cache(ApiPokemon), name: String) {
	cache.read_through(cache, name, fn() {
		do_pokeapi_request("/api/v2/pokemon/" <> name)
		|> decode_pokemon
	})
}

pub fn fetch_move(cache: Cache(Move), number: Int) {
	let move_number = int.to_string(number)
	cache.read_through(cache, move_number, fn() {
		do_pokeapi_request("/api/v2/move/" <> move_number)
		|> decode_move
	})
}
```

```gleam
pub fn fetch_pokemon(cache: Cache(ApiPokemon), name: String) {
	use <- cache.read_through(cache, name)

	do_pokeapi_request("/api/v2/pokemon/" <> name)
	|> decode_pokemon
}

pub fn fetch_move(cache: Cache(Move), number: Int) {
	let move_number = int.to_string(number)
	use <- cache.read_through(cache, move_number)

	do_pokeapi_request("/api/v2/move/" <> move_number)
	|> decode_move
}
```
````

<!--
- Now we can apply the `read_through` function to our `fetch_pokemon` and `fetch_move` functions

[click]

- And, of course, we can apply `use` here too
-->
