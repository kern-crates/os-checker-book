# rap

当前支持的示例

## memory leak

```rust
let buf = Box::new("buffer");
let _ptr = Box::into_raw(buf);
```

```rust
struct Proxy<T> { _p: *mut T }

let buf = Box::new("buffer");
let ptr = Box::into_raw(buf);
let _proxy = Proxy { _p:ptr };
```

## use after free

```rust
fn create_vec() -> *mut Vec<i32> {
    let mut v = Vec::new();
    v.push(1);
    &mut v as *mut Vec<i32>
}
```

```rust
let mut s = String::from("a tmp string");
let ptr = s.as_mut_ptr();
let _v = unsafe { Vec::from_raw_parts(ptr, s.len(), s.len()) };
```

```rust
let mut slot = ManuallyDrop::<Box<u8>>::new(Box::new(1));
unsafe { ManuallyDrop::drop(&mut slot); }
println!("{:?}", slot);
```

```rust
struct MyRef<'a> { a: &'a str, }

impl<'a> MyRef<'a> {
    fn print(&self) { println!("{}", self.a); }
}

unsafe fn f<'a>(myref: MyRef<'a>) -> MyRef<'static> {
    unsafe { std::mem::transmute(myref) }
}

fn main() {
    let string = "Hello World!".to_string();
    unsafe {
        let my_ref = f(MyRef { a: &string });
        drop(string);
        my_ref.print();
    }
}
```
