In Go there are various ways to return a struct value or slice thereof. For individual ones I've seen:
```
type MyStruct struct {
    Val int
}

func myfunc() MyStruct {
    return MyStruct{Val: 1}
}

func myfunc() *MyStruct {
    return &MyStruct{}
}

func myfunc(s *MyStruct) {
    s.Val = 1
}
```
I understand the differences between these. The first returns a copy of the struct, the second a pointer to the struct value created within the function, the third expects an existing struct to be passed in and overrides the value.

I've seen all of these patterns be used in various contexts, I'm wondering what the best practices are regarding these. When would you use which? For instance, the first one could be ok for small structs (because the overhead is minimal), the second for bigger ones. And the third if you want to be extremely memory efficient, because you can easily reuse a single struct instance between calls. Are there any best practices for when to use which?

Similarly, the same question regarding slices:
```
func myfunc() []MyStruct {
    return []MyStruct{ MyStruct{Val: 1} }
}

func myfunc() []*MyStruct {
    return []MyStruct{ &MyStruct{Val: 1} }
}

func myfunc(s *[]MyStruct) {
    *s = []MyStruct{ MyStruct{Val: 1} }
}

func myfunc(s *[]*MyStruct) {
    *s = []MyStruct{ &MyStruct{Val: 1} }
}
```
Again: what are best practices here. I know slices are always pointers, so returning a pointer to a slice isn't useful. However, should I return a slice of struct values, a slice of pointers to structs, should I pass in a pointer to a slice as argument (a pattern used in the Go App Engine API)?



Methods using receiver pointers are common; the rule of thumb for receivers is, "If in doubt, use a pointer."
Slices, maps, channels, strings, function values, and interface values are implemented with pointers internally, and a pointer to them is often redundant.
Elsewhere, use pointers for big structs or structs you'll have to change, and otherwise pass values, because getting things changed by surprise via a pointer is confusing.
Some specific cases where you don't have to use pointers:

Receivers are pointers more often than other arguments. It's not unusual for methods to modify the thing they're called on, or for named types to be large structs, so the guidance is to default to pointers except in rare cases.
Jeff Hodges' copyfighter tool automatically searches for non-tiny receivers passed by value.
Code review guidelines suggest passing small structs like type Point struct { latitude, longitude float64 }, and maybe even things a bit bigger, as values, unless the function you're calling needs to be able to modify them in place.
Value semantics avoid aliasing situations where an assignment over here changes a value over there by surprise.
It's not Go-y to sacrifice clean semantics for a little speed, and sometimes passing small structs by value is actually more efficient, because it avoids cache misses or heap allocations.
So, Go Wiki's code review comments page suggests passing by value when structs are small and likely to stay that way.
If the "large" cutoff seems vague, it is; arguably many structs are in a range where either a pointer or a value is OK. As a lower bound, the code review comments suggest slices (three machine words) are reasonable to use as value receivers. As something nearer an upper bound, bytes.Replace takes 10 words' worth of args (three slices and an int).
For slices, you don't need to pass a pointer to change elements of the array. io.Reader.Read(p []byte) changes the bytes of p, for instance. It's arguably a special case of "treat little structs like values," since internally you're passing around a little structure called a slice header (see Russ Cox's explanation at http://research.swtch.com/godata). Similarly, you don't need a pointer to modify a map or communicate on a channel.
For slices you'll reslice (change the start/length/capacity of), built-in functions like appendaccept a slice value and return a new one. I'd imitate that; it avoids aliasing, returning a new slice helps call attention to the fact that a new array might be allocated, and it's familiar to callers.
It's not always practical follow that pattern. Some tools like database interfaces orserializers need to append to a slice whose type isn't known at compile time. They sometimes accept a pointer to a slice in an interface{} parameter.
Maps, channels, strings, and function and interface values, like slices, are internally references or structures that contain references already, so if you're just trying to avoid getting the underlying data copied, you don't need to pass pointers to them. (rsc wrote a separate post on how interface values are stored).
You still may need to pass pointers in the rarer case that you want to modify the caller's struct: flag.StringVar takes a *string for that reason, for example.
Where you use pointers:

Consider whether your function should be a method on whichever struct you need a pointer to. People expect a lot of methods on x to modify x, so making the modified struct the receiver may help to minimize surprise. There are guidelines on when receivers should be pointers.
Functions that have effects on their non-receiver params should make that clear in the godoc, or better yet, the godoc and the name (like reader.WriteTo(writer)).
You mention accepting a pointer to avoid allocations by allowing reuse; changing APIs for the sake of memory reuse is an optimization I'd delay until it's clear the allocations have a nontrivial cost, and then I'd look for a way that doesn't force the trickier API on all users:
Consider a Reset() method to put an object back in a blank state, like some stdlib types offer. Users who don't care or can't save an allocation don't have to call it.
Consider writing modify-in-place methods and create-from-scratch functions as matching pairs, for convenience: existingUser.LoadFromJSON(json []byte) error could be wrapped by NewUserFromJSON(json []byte) (*User, error). Again, it pushes the choice between laziness and pinching allocations to the individual caller.
Callers seeking to save memory can outsource some of the work of recycling to sync.Pool. Do it! (CloudFlare published a useful (pre-sync.Pool) blog post about recycling.)
For avoiding allocations, Go's escape analysis is your friend. You can sometimes help it avoid heap allocations by making types that can be initialized with a trivial constructor, a plain literal, or a useful zero value like bytes.Buffer.
Curiously, for complicated constructors, new(Foo).Reset() can sometimes avoid an allocation when NewFoo() would not. Not idiomatic; careful trying that one at home.
Finally, on slices of objects versus slices of pointers: slices of objects can be useful, especially for small value types (a few numbers/strings/etc.), but things like reallocation when growing a slice can present a problem. For example, you need a slice of pointers if:

You'll naturally allocate your objects from/in different places (e.g., you have to initialize with NewFoo() *Foo).
You'll hold onto pointers to individual structs across an append. Those pointers will point to the old slice if append reallocates. :( Using indices into the slice instead of pointers may be a workaround here depending on the context.
You'll hold onto pointers to individual structs after the whole slice is garbage. Those pointers to individual objects keep the slice from being garbage-collected.
You'll do a lot of swapping (e.g. sorting) and full objects are huge and expensive to swap.
You'll copy a lot (e.g. insert/delete in the middle) and full objects are huge and expensive to copy.
Your object contains something like a Mutex that can't be copied.
If it is natural to use a slice of objects rather than a slice of pointers, though, you should go for it, because it keeps them in one contiguous block of memory (locality good!) and only performs a single allocation, which reduces memory-management overhead. Especially if your objects have a useful zero value (so plain make([]JobStatus, 123) makes 123 useful JobStatuses) and none of the blockers above apply, you might as well use []JobStatus rather than[]*JobStatus.