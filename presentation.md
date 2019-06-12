## Rust: Borrow checker odtajniony
#### Bartłomiej Kuras

----

### Bartłomiej Kuras
* Ponad 6 lat programista C++
* Ostatni rok programista Rust w Anixe
* Od roku członek, od niedawna współorganizator Rust Wrocław
* Entuzjasta Rusta od ponad 2 lat

----

### Rust Wrocław
* Non-profit <!-- .element: class="fragment" -->
* Rust Wrocław Meetup - pierwsze czwartki miesiąca <!-- .element: class="fragment" -->
    * 10 spotkań <!-- .element: class="fragment" -->
    * 1 warsztat <!-- .element: class="fragment" -->
    * Przerwa wakacyjna (ale planujemy mniej formalne spotkania) <!-- .element: class="fragment" -->

====

### Rust Wrocław
* Slack: [rust-wroclaw](https://join.slack.com/t/rust-wroclaw/shared_invite/enQtNTQ2NjEwOTA3OTIwLTQwMjRmM2VlMWU1OGZhZjcwYjA3ZWNiNTU2MDg3MjEzYmFjNGQ5NzNjNmYwN2EwZTAyZDY4MDczNDVhMDkxNTI) 
* Facebook: https://www.facebook.com/rustwroclaw/
* WWW: https://rust-wroclaw.github.io/

----

## Po co nowy język?
### Pamięć <!-- .element: class="fragment" -->
* Use after free <!-- .element: class="fragment" -->
* Double free <!-- .element: class="fragment" -->
* Null pointer deref <!-- .element: class="fragment" -->
* Data race <!-- .element: class="fragment" -->

[Rust: A language for the Next 40 Years - Carol Nichols](https://www.youtube.com/watch?v=A3AdN7U24iU) <!-- .element: class="fragment" -->

====

> Około 70% wszystkich błędów w produktach Microsoftu adresowanych
> w aktualizacjach bezpieczeństwa każdego roku, to problemy związane
> z bezpieczeństwem pamięci.

Catalin Cimpanu, MS security engineer, ZDNet, 2019-02-11

====

> Borrow checker pozwala mi wreszcie nie bać się o to co dzieje
> się z danymi pomiędzy przerwaniami.

Michał Chodzikiewicz, Programista Embeded, Rust Wrocław Meetup 2019-06-06

----

### Podstawy

```rust
fn main() {
    let _v = vec![1, 2, 3];
}
```

====

### Podstawy

```rust
fn main() {
    let _v = vec![1, 2, 3];
    Drop::drop(_v);
}
```

====

### Podstawy

```cpp
int main() {
    std::vector<int> v;
}
```

----

### RAII

```rust
fn main() {
    let a = Box::new(10);
}
```

====

### RAII

```cpp
int main() {
    auto a = std::make_unique<int>(10);
}
```

====

### RAII

```cpp
int main() {
    auto a = std::make_unique<int>(10);
    auto b = std::move(a);
    *a = 20; // < ?! UB
}
```

====

### RAII

```rust
fn main() {
    let mut a = Box::new(10);
    let _b = a; // Move is default behaviour
    *a = 20;
}
```

====

### RAII

```rust
fn main() {
    let mut a = Box::new(10);
    let _b = a; // Move is default behaviour
    *a = 20; // < Doesn't compile
}
```

```
error[E0382]: use of moved value: `a`
 --> src/main.rs:4:5
  |
2 |     let mut a = Box::new(10);
  |         ----- move occurs because `a`
  |               has type `std::boxed::Box<i32>`,
  |               which does not implement the `Copy` trait
3 |     let _b = a; // Move is default behaviour
  |              - value moved here
4 |     *a = 20;
  |     ^^^^^^^ value used here after move
```

----

### Borrowing

```rust
fn main() {
    let mut a = 10;
    let b = &a;
    println!("{}", b);
    a = 20;
}
```

====

### Borrowing

```cpp
int main() {
    int a = 10;
    const int &b = a;
    std::cout << b << '\n';
    a = 20;
}
```

====

### Borrowing

```cpp
const int & foo(int x) { return x; }

int main() {
    int a = 10;
    auto b = foo(a);
}
```

https://www.onlinegdb.com/online_c++_compiler: Bez ostrzeżeń, SEGFAULT w RT

https://repl.it/languages/cpp: Ostrzeżenie, żadnych objawów w RT

====

#### Borrowing

```rust
fn foo(x: i32) -> &i32 {
    x
}

fn main() {
    let a = 10;
    let b = foo(a);
}
```

====

#### Borrowing

```rust
fn foo(x: i32) -> &i32 { &x }

fn main() {
    let a = 10;
    let b = foo(a);
}
```

```
error[E0106]: missing lifetime specifier
 --> src/main.rs:1:19
  |
1 | fn foo(x: i32) -> &i32 {
  |                   ^ help: consider giving it an explicit
  |                     bounded or 'static lifetime: `&'static`
  |
  = help: this function's return type contains a
          borrowed value with an elided lifetime,
          but the lifetime cannot be derived from the arguments
```
====

#### Borrowing

```rust
fn foo(x: i32) -> &'static i32 { x }

fn main() {
    let a = 10;
    let _b = foo(a);
}
```

```
error[E0515]: cannot return reference to function parameter `x`
 --> src/main.rs:2:5
  |
2 |     &x
  |     ^^ returns a reference to data owned
  |        by the current function
```

----

#### Mutable borrowing

```rust
fn main() {
    let mut a = 10;
    let b = &mut a;
    *b = 20;
    println!("{}", a);
}
```

====

#### Mutable borrowing

```cpp
int main() {
    int a = 10;
    int & b = a;
    b = 20;
    std::cout << a << '\n';
}
```

====

#### Mutable borrowing

```cpp
int main() {
    int a = 10;
    int & b = a;
    std::cout << a << '\n';
    b = 20;
}
```

====

#### Mutable borrowing

```rust
fn main() {
    let mut a= 10;
    let b = &mut a;
    println!("{}", a);
    *b = 20;
}
```

====

#### Mutable borrowing

```rust
fn main() {
    let mut a= 10;
    let b = &mut a;
    println!("{}", a);
    *b = 20;
}
```

```
error[E0502]: cannot borrow `a` as immutable because
              it is also borrowed as mutable
 --> src/main.rs:4:20
  |
3 |     let b = &mut a;
  |             ------ mutable borrow occurs here
4 |     println!("{}", a);
  |                    ^ immutable borrow occurs here
5 |     *b = 20;
  |     ------- mutable borrow later used here
```

----

### Zasady pożyczania

W każdym miejscu może istnieć:

* 1 modyfikowalna referencja *XOR*
* dowolna ilość stałych referencji

do każdego dostępnego zasobu

----

#### Inwalidacja iteratora

```cpp
int main() {
    std::vector<int> a { 1, 2, 3, 4, 5 };
    for(const auto & b: a) {
        if b % 2 == 1 { a.push_back(b*2); }
    }
}
```

====

#### Inwalidacja iteratora

```rust
fn main() {
    let a = vec![1, 2, 3, 4, 5];
    for b in a.iter() {
        if b % 2 == 1 { a.push(b*2) }
    }
}
```

====

#### Inwalidacja iteratora

```rust
fn main() {
    let mut a = vec![1, 2, 3, 4, 5];
    for b in a.iter() {
        if b % 2 == 1 { a.push(b*2) }
    }
}
```

```
error[E0502]: cannot borrow `a` as mutable because it
              is also borrowed as immutable
 --> src/main.rs:4:25
  |
3 |     for b in a.iter() {
  |              --------
  |              |
  |              immutable borrow occurs here
  |              immutable borrow later used here
4 |         if b % 2 == 1 { a.push(b*2) }
  |                         ^^^^^^^^^^^ mutable borrow
  |                                     occurs here
```

====

#### Inwalidacja iteratora

```rust
fn main() {
    let mut a = vec![1, 2, 3, 4, 5];
    for i in 0..a.len() {
        if a[i] % 2 == 1 { a.push(a[i] * 2) }
    }
}
```

====

#### Inwalidacja iteratora

```rust
fn main() {
    let mut a = vec![1, 2, 3, 4, 5];
    let new_items: Vec<_> =
        a.iter()
         .filter(|b| *b % 2 == 1)
         .map(|b| *b * 2)
         .collect();
    a.extend(new_items);
}
```

----

#### Lifetimes

```rust
fn foo<'a>(x: &'a i32) -> &'a i32 { x }

fn main() {
    let arg = 40;
    let res = foo(&a);
}
```

====

#### Lifetimes

```rust
fn foo<'a>(x: &'a i32) -> &'a i32 { x }

fn main() {
    'scope: {
        let arg: i32 + 'scope = 40;
        let res: &'scope i32 = foo::<'scope>(&a);
    }
}
```

====

#### Lifetimes

```rust
fn foo(x: &i32) -> &i32 { x }

fn main() {
    let a = 40;
    let b = foo(&a);
}
```

----

#### Borrows a struktury

```rust
struct Foo {
    bar: &i32,
}

fn main() {
    let x = 15;
    let a = Foo { bar: &x };
}
```
====

#### Borrows a struktury

```rust
struct Foo {
    bar: &i32,
}

fn main() {
    let x = 15;
    let foo = Foo { bar: &x };
}
```

```
error[E0106]: missing lifetime specifier
 --> src/main.rs:2:8
  |
2 |     bar: &i32,
  |          ^ expected lifetime parameter

```

====

#### Borrows a struktury

```rust
struct Foo<'a> {
    bar: &'a i32,
}

fn main() {
    let x = 15;
    let foo = Foo { bar: &x };
}
```

====

#### Borrows a struktury

```rust
struct Foo<'a> {
    bar: &'a i32,
}

fn take_foo<'a>(x: Foo<'a>) -> &'a i32 { x.bar }

fn main() {
    let x = 15;
    let foo = Foo { bar: &x };
    let b = take_foo(foo);
}
```

----

#### Struktury CD

```rust
struct Foo { bar: i32 }

impl Foo {
    fn get_bar(&self) -> &i32 { &self.bar }
    fn get_mut_bar(&mut self) -> &mut i32 { &mut self.bar }
}

fn main() {
    let foo = Foo { bar: 40 };
    let bar_ref = foo.get_bar();
    let bar_mut = foo.get_mut_bar();
    println!("{}", bar_ref); // < Compile error
    *bar_mut = 20;
}
```

====

#### Struktury CD

```rust
struct Foo { xxx: i32, yyy: i32 }

impl Foo {
    fn get_xxx(&self) -> &i32 { &self.xxx }
    fn get_mut_yyy(&mut self) -> &mut i32 { &mut self.yyy }
}

fn main() {
    let mut foo = Foo { xxx: 10, yyy: 20 };
    let x = foo.get_xxx();
    if x > 10 {
        *foo.get_yyy() = 30;
    }
    pritnln!("{}", x);
}
```

====

#### Struktury CD

```rust
struct Foo { xxx: i32, yyy: i32 }

impl Foo {
    fn split(&mut self) -> (&i32, &mut i32) {
        (&self.xxx, &mut self.yyy)
    }
}

fn main() {
    let mut foo = Foo { xxx: 10, yyy: 20 };
    let (x, y) = foo.split();
    if *x > 10 { *y = 30; }
    println!("{}", x);
}
```

----

#### Borrows a traity

```rust
trait T<'a> {
    fn get(&self) -> &'a i32;
}

struct Foo<'b> { bar: &'b i32 }

impl<'x> T<'x> for Foo<'x> {
    fn get(&self) -> &'x i32 { self.bar }
}

fn foo<'y>(arg: impl T<'y>) -> &'y i32 { x.get() }
```

====

```rust
trait T {
    fn get<'a>(&'a self) -> &'a i32;
}

struct Foo<'b> { bar: &'b i32 }

impl<'x> T for Foo<'x> {
    fn get<'c>(&'c self) -> &'c i32 { self.bar }
}

fn take_foo<'y>(arg: &'y i32) -> impl T {
    Foo { bar: arg }
}
```

====


```
error: cannot infer an appropriate lifetime
  --> src/lib.rs:12:5
   |
11 | fn take_foo<'y>(arg: &'y i32) -> impl T {
   |                                  ------ this return type evaluates
   |                                         to the `'static` lifetime...
12 |     Foo { bar: arg }
   |     ^          - ...but this borrow...
   |
note: ...can't outlive the lifetime 'y as defined on the function body at 11:8
  --> src/lib.rs:11:8
   |
11 | fn take_foo<'y>(x: &'y i32) -> impl T {
   |             ^^
help: you can add a constraint to the return type to make it last less
   |  than `'static` and match the lifetime 'y as defined on the
   |  function body at 11:8
   |
11 | fn take_foo<'y>(x: &'y i32) -> impl T + 'y {
   |                                ^^^^^^^^^^^
```

====

```rust
trait T {
    fn get<'a>(&'a self) -> &'a i32;
}

struct Foo<'b> { bar: &'b i32 }

impl<'x> T for Foo<'x> {
    fn get<'c>(&'c self) -> &'c i32 { self.bar }
}

fn take_foo<'y>(x: &'y i32) -> impl T + 'y {
    Foo { bar: arg }
}
```

----

# Dziękuję

## Pytania?

https://github.com/hashedone/borrow_checker_demyth_pl
