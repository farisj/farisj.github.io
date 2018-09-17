---
layout: single
title: "Go Tips & Tricks"
date: 2018-09-19T10:22:36-04:00
excerpt: Some fun things I've learned while learning Go
---

After a few years of programming almost exclusively in Ruby on Rails and Javascript, I've been fortunate enough to expand my skillset and get involved with some projects that are written in Golang. Golang, also known as [Go](https://golang.org/), is Google's open source programming language, and has been growing in popularity as a resilient, opinionated backend language. Some of the more interesting features in my opinion are:

* Compiled
* Statically Typed
* Strong concurrency support
* Documentation & Testing Tools built in

There are many awesome things about Go, and after developing projects in Go for a little while I've found a couple of neat things about the language that I wanted to share. It's been a while since I've blogged, so hopefully this post will put that fire under me and encourage me to pick technical blogging up as a hobby again. As for this post, this is not an exhaustive list of awesome features and libraries (there are too many to name!), but it's a start. So, let's get started!

![Starting Party]({{ site.baseurl }}/assets/images/go-tips-tricks/party.gif)

__NOTE:__ This blog post assumes some basic knowledge of Go.

# Forced Randomization When Iterating On Maps

Suppose your code has a `map` in it and you want to display the contents of that map. A common way to do so is to iterate over a `range` of that map and `fmt.Println()` the keys/values.

```go
func main() {
  m := map[int]bool{
    1:   true,
    5:   true,
    99:  true,
    101: false,
    14:  true,
  }

  for k, v := range m {
    fmt.Printf("%d: %t\n", k, v)
  }
}
```

It's more or less commonly accepted that relying on ordered maps is a bad idea, because you're depending on a functionality that isn't a primary purpose of that data structure. Golang agrees with this and enforces it aggressively. In fact, it _actively_ randomizes the map's ordering based on the random seed that's generated at runtime.

Try it out yourself on this [Go Playground](https://play.golang.org/p/8IdNVrPsleU) (which is a GREAT tool by the way!). In that playground, we're manually setting the `rand.Seed()`, which affects the pseudo-random algorithm used by the language. As you re-run the function with the same seed, the map will print out the same way each time. However, when the seed is changed, the ordering changes.

The seed changes with every `go run` (except on the Go Playground where it's a fixed seed by default, that's why the above code manually changes the seed), so you wouldn't get the same order regularly in production ever. This is Go's way of enforcing this good behavior!

# Embedding and Composing structs

I was impressed to see Go's methods of Composition and how methods can be shared by "Embedding" structs.

Consider that we have two types, `User` and `Admin`:

```go
type User struct {
  Name string
}

type Admin struct {
  User User
  Permissions map[string]string
}
```

In the above example, there's a `User` and an `Admin`, where the `Admin` has a `User` property along with some additional `Permissions.`

Here, one could access the `Name` of the `Admin` via `admin.User.Name` (assuming there's an instantiated `Admin` called `admin`).

However, consider the removal of _one_ word in the previous example which changes the `Admin`'s method surface completely:

```go
type User struct {
  Name string
}

type Admin struct {
  User
  Permissions map[string]string
}
```

Did you notice it? The second `User` in the `Admin` struct was removed. This means that the `User` is "embedded" within the `Admin`, and all its methods are "promoted" to the `Admin` as well. This means that one can reference the name of the user via `admin.Name` - no intermediate call to the `User` needed!

I found this to be pretty neat - it can definitely be used to share methods and help keep code clean.

Another additional use of this technique is known as "Composing" types, which consists of embedding various types to create other types/interfaces. A good example is in the `io` library, the `ReadWriter`, whose code I lifted directly from the [Go Docs](https://godoc.org/io#ReadWriter):

```go
type ReadWriter interface {
    Reader
    Writer
}
```

What I can see from the above definition is that a `ReadWriter` is an interface which must contain all the functions defined on both `Reader` and `Writer`, which are defined elsewhere. Knowing the way composition is set up in Go, that makes a lot of sense! It also allows the `Reader` and `Writer` to be used independently of each other, which is a plus.


# Testing

## Use the Test flags to your advantage

The `testing` package provides methods to give the test suite itself a way to know how it is being run. The `-v` and `-short` flags that can be used with `go test` are hooked into the package itself, and the user can write tests that behave differently depending on if those flags are present.

Consider a package and test that look like the following:

```go
import "fmt"

func main() {
  return fmt.Println(helloWorld())
}

func helloWorld() string{
  return fmt.Sprintf("Hello world")
}
```

```go
import (
  "testing"
  "fmt"
)

func TestHelloWorld(t *testing.T) {
  if testing.Short() {
    t.Skip("Short test suite, skipping test.")
  }

  if testing.Verbose() {
    fmt.Println("About to test helloWorld()...")
  }

  exp := "Hello world"
  got := helloWorld()
  if exp != got {
    t.Errorf("Expected %s, got %s", exp, got)
  }
}
```

In the above example, you will see that the `testing` package exposes a `testing.Short()` and `testing.Verbose()` method which can be used to dictate testing logic. These correspond directly to `go test -short` and `go test -v`. Even though this is more of a minor feature, it allows programmers more expressive control over their test suite, which I appreciate!

## Stubbing Tests & Keeping Code Loosely Coupled

One common problem that Go programs suffer from is an inflexibility to stub things out in testing due to Go's strict type-checking at compilation time. What Go is doing when it complains about typing issues in tests is really surfacing a common problem known as "tightly coupled" code, where functions and structs are explicitly relying on other constructs internally, which prevents them from being tested in isolation.

Here's an example: Suppose you are building a Go app that hits an API and then does some work. Let's call the struct that does this a `Worker`:

```go
type Worker struct {
  APIClient *Client
}

func (w Worker) Work(input string) {
  // Some work here...
  result, err := w.APIClient.Update(input)
  if err != nil {
    return nil, result
  }
  return result, nil
}
```

This seems great, but consider what a test for this struct might look like:

```go
import (
  "testing"
)

func TestWork(t *Testing.T) {
  // Setup test...
  worker := Worker(&Client{})
  result, err := worker.Work(input)

  if err != nil {
    t.Fatalf("Error: %s", err)
  }
  // Other assertions...
}
```

The problem with actually calling the `Work()` function above is that it internally uses the `Client.Update()` method and that's not the thing we're trying to test here! We just want to make sure that the Client is being called with `Update(input)`. In Rails' `rspec` tests, stubbing this out would be pretty easy, as we could say something like

```ruby
expect_any_instance_of(Client).to receive(:update).with(input).and_return(result, nil)
```

But we can't stub so easily in Go, as the Go compiler would be very unhappy with us if we tried to instantiate a `Worker` with anything other than a real life `Client` struct.

Because the `Worker` and the `Client` are tightly coupled, we can't test one without building out the other. Let's try to edit the `Worker` struct so that it isn't so directly dependent on the `Client` struct. Instead, we can give the `Worker` an `interface` that looks like the `Client` and has an `Update` method, like so:

```go
type Worker struct {
  APIClient DataLayer
}

type DataLayer interface {
 Update(string) (string, err)
}
```

This way, we can make a _fake_ Client in our test and bypass all of the logic that takes place in the real `Client` struct. (Obviously, we'll have to test that separately :wink:) Our updated test could look something like this:

```go
import (
  "testing"
)

type FakeClient struct {
  updateCalled bool
}

func (f *FakeClient) Update (string, error) {
  f.updateCalled = true
  return "", nil
}

func TestWork(t *testing.T) {
  // Setup test...
  client := &FakeClient{}
  worker := Worker(client)
  _, err := worker.Work(input) // Goes out an hits an API

  if client.updateCalled != true {
    t.Fatal("Expected Client.Update() to be called")
  }
  // Other assertions...
}
```

Now, we can use our stubbed `Client` class to verify that the `Worker`'s internals are behaving correctly without having to worry about the details of `Client`!


Unlike Ruby or Javascript, where testing frameworks are built by external parties that aim to give their languages a rich testing culture, Go's testing features are built in with the default libraries. What usually ends up happening with Go tests are suites that may need a little extra setup, but the benefit is that there's no additional testing DSL that programmers need to learn - it's all just Go code! Writing tests so closely to the executed code helps bring extra clarity to what you're working with.

# Benchmarking

Benchmarking, in Go and all other languages, is the process of evaluating the performance of code.

In Go, benchmark tests are constructed similarly to conventional tests. Benchmark tests follow some strict guidelines and look like so:

```go
func Fib(n int) int {
  if n < 2 {
    return n
  }
  return Fib(n-1) + Fib(n-2)
}
```

```go
import "testing"

func BenchmarkFib(b *testing.B) {
  for n := 0; n < b.N; n++ {
    Fib(20) // run the Fib function b.N times
  }
}
```

In benchmark tests, functions start with the prefix `Benchmark` and take a pointer to a `testing.B`, not `testing.T` like logic tests. The tests commonly involve a loop up to `b.N`, where the benchmark tests control what `N` is and how many times the tests are run. The tests can be run with the `-bench` flag, ie:

```
go test -bench=.
```

The `.` here is a regular expression which tells Go to bench everything! If you have a complex suite, you can run a slice of your tests and bench however you like! The results will look something like this:

```
pkg: farisj/mypackage/fib
BenchmarkFib-4       30000       49477 ns/op
PASS
ok    farisj/mypackage/fib 1.996s
```

This type of work is useful when refactoring, for example. Benchmarking two different algorithms can help programmers make engineering decisions which are backed not only by their own intuition, but by numbers too!

Something that I personally struggle with in regards to Benchmarking is getting used to the idea that the numbers presented are _relative_ to my machine and that there is no objective "speed" that code runs at. Not only are there so many different platforms/processors that run Go code, all of which perform differently, but there are usually a lot of other processes running which use up resources. If I had it my way, there would be some objective 'score' that the code could have, but that's just the way it is! I have to trust the relative performance of different functions and hope that my computer's processor can handle it.

![Fox Meditating]({{ site.baseurl }}/assets/images/go-tips-tricks/zen.gif)

(Action shot of me trying to accept the things I cannot change.)

# Use Delve To Debug

[Delve](https://github.com/derekparker/delve) is a Go package that starts a interactive session where the user can set breakpoints in functions, evaluate variables, and step through code in real time. It's similar to [binding.pry](https://pryrepl.org/) in Ruby, although there are a few differences. From what I've seen so far, `delve` is a little more involved than `binding.pry`, but hey, that's just Go vs Ruby for ya! It's a tradeoff for sure.

Given the following example program, we can use `delve` to inspect our variables:

```go
func (user User) String() string {
  return fmt.Sprintf("%s %s", user.First, user.Last)
}

func main() {
  names := []string{
    "Steven",
    "Connie",
    "Onion",
  }
  for index := 0; index < 3; index++ {
    name := names[index]
    message := Hello(name, index == 0)
    fmt.Printf(message)
  }
}

func Hello(name string, firstTime bool) string {
  if firstTime {
    return fmt.Sprintf("Hello %s. You're first!\n", name)
  }
  return fmt.Sprintf("Hello %s. Happy to see you!\n", name)
}
```

Once you've got it installed (check out the [docs](https://github.com/derekparker/delve/tree/master/Documentation/installation)), Delve is run with:

```
dlv debug ./main.go
```

where `./main.go` is the application entry point. From there, the session begins, where you can set breakpoints, evaluate variables, etc. Here are a few of the basics:

* Set a breakpoint with the `breakpoint` command and specify either the function name or line number, ie:

```
break main.go:25
```

or

```
break main.Hello
```

Once the breakpoints are set, run `continue` to start the program. From there, it will begin to execute the code normally and stop when a set breakpoint is reached. Supposing we set a breakpoint on `Hello()`, we would see the debugger wait for input there:

```
(dlv) continue
> main.Hello() ./main.go:25 (hits goroutine(1):1 total:1) (PC: 0x1088ffb)
    20:                 message := Hello(name, index == 0)
    21:                 fmt.Printf(message)
    22:         }
    23: }
    24:
=>  25: func Hello(name string, firstTime bool) string {
    26:         if firstTime {
    27:                 return fmt.Sprintf("Hello %s. You're first!\n", name)
    28:         }
    29:         return fmt.Sprintf("Hello %s. Happy to see you!\n", name)
    30: }
```

Then, one can type `firstTime` to see the value of that boolean! After hitting `continue` and returning to this breakpoint, the value will have changed from `true` to `false`.

Delve seems to be a very effective and helpful tool for debugging applications that provides more flexibility and control than simple `fmt.Println()` debugging. That being said, I've gotten by so far with `fmt.Println()` debugging, so I haven't found its usefulness apparent firsthand. I'm waiting for the day I'm working with a very complex application where `fmt.Println` just doesn't cut it!

# Summary

Golang is a powerful language with a lot of conventions, but also a lot of tools for developers built right in. With an expressive and highly opinionated syntax and structure, it's easy to jump into a project and start contributing without spending so much time untangling any complicated metaprogramming conventions, monkey patching, or stubbed methods. Switching back and forth between Rails and Go projects lately, I can see how Go's lower-level syntax and static types can appear challenging, but the truth is that those language choices mean that there are fewer places for bugs to hide!

Venturing into a new language can be pretty daunting, but Go's vibrant community and stellar documentation tools make it much easier to get started. I would definitely recommend anyone interested in backend programming to dip their toe in the waters of Go and see what it's all about!

Thanks to everyone taking the time to read my blog post highlighting a few things I've learned about Go. Hopefully I'll be back soon with another informative post!
