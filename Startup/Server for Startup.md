In these notes, I will briefly talk about the how, what and why of the decisions made to create this server.

I decided to build it in Rust because its just the best language right now. It has similar strength to Gods own language, the mighty C, but also has modern updates and usability. Using TypeScript is also an option but the more I read about `npm` and its problems with dependencies and malicious injection of code, I decided that a fully self contained server would be a better idea.

This allows me manage the server wherever I am, not having to worry about dependencies not being up to date or incompatible.

## Logging

During the building and test phase, I obviously made some mistakes and needed to check logs for those errors. Therefore I've built a `logger` to save the logs to be referred back to

```
panicked at src/main.rs:29:59:
called `Result::unwrap()` on an `Err` value: Os { code: 2, kind: NotFound, message: "No such file or directory" }

Backtrace:
   0: server::logger::crash_log::{{closure}}
             at ./logger.rs:7:18
   1: <alloc::boxed::Box<F,A> as core::ops::function::Fn<Args>>::call
             at /rustc/254b59607d4417e9dffbc307138ae5c86280fe4c/library/alloc/src/boxed.rs:2220:9
   2: std::panicking::panic_with_hook
             at /rustc/254b59607d4417e9dffbc307138ae5c86280fe4c/library/std/src/panicking.rs:833:13
   3: std::panicking::panic_handler::{{closure}}
             at /rustc/254b59607d4417e9dffbc307138ae5c86280fe4c/library/std/src/panicking.rs:698:13
   4: std::sys::backtrace::__rust_end_short_backtrace
             at /rustc/254b59607d4417e9dffbc307138ae5c86280fe4c/library/std/src/sys/backtrace.rs:176:18
   5: __rustc::rust_begin_unwind
             at /rustc/254b59607d4417e9dffbc307138ae5c86280fe4c/library/std/src/panicking.rs:689:5
   6: core::panicking::panic_fmt
             at /rustc/254b59607d4417e9dffbc307138ae5c86280fe4c/library/core/src/panicking.rs:80:14
   7: core::result::unwrap_failed
             at /rustc/254b59607d4417e9dffbc307138ae5c86280fe4c/library/core/src/result.rs:1867:5
   8: core::result::Result<T,E>::unwrap
             at /home/schmitzi/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/core/src/result.rs:1233:23
   9: server::handle_connection
             at ./main.rs:29:59
  10: server::main
             at ./main.rs:48:9
  11: core::ops::function::FnOnce::call_once
             at /home/schmitzi/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/core/src/ops/function.rs:250:5
  12: std::sys::backtrace::__rust_begin_short_backtrace
             at /home/schmitzi/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/std/src/sys/backtrace.rs:160:18
  13: std::rt::lang_start::{{closure}}
             at /home/schmitzi/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/std/src/rt.rs:206:18
  14: core::ops::function::impls::<impl core::ops::function::FnOnce<A> for &F>::call_once
             at /rustc/254b59607d4417e9dffbc307138ae5c86280fe4c/library/core/src/ops/function.rs:287:21
  15: std::panicking::catch_unwind::do_call
             at /rustc/254b59607d4417e9dffbc307138ae5c86280fe4c/library/std/src/panicking.rs:581:40
  16: std::panicking::catch_unwind
             at /rustc/254b59607d4417e9dffbc307138ae5c86280fe4c/library/std/src/panicking.rs:544:19
  17: std::panic::catch_unwind
             at /rustc/254b59607d4417e9dffbc307138ae5c86280fe4c/library/std/src/panic.rs:359:14
  18: std::rt::lang_start_internal::{{closure}}
             at /rustc/254b59607d4417e9dffbc307138ae5c86280fe4c/library/std/src/rt.rs:175:24
  19: std::panicking::catch_unwind::do_call
             at /rustc/254b59607d4417e9dffbc307138ae5c86280fe4c/library/std/src/panicking.rs:581:40
  20: std::panicking::catch_unwind
             at /rustc/254b59607d4417e9dffbc307138ae5c86280fe4c/library/std/src/panicking.rs:544:19
  21: std::panic::catch_unwind
             at /rustc/254b59607d4417e9dffbc307138ae5c86280fe4c/library/std/src/panic.rs:359:14
  22: std::rt::lang_start_internal
             at /rustc/254b59607d4417e9dffbc307138ae5c86280fe4c/library/std/src/rt.rs:171:5
  23: std::rt::lang_start
             at /home/schmitzi/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/std/src/rt.rs:205:5
  24: main
  25: <unknown>
  26: __libc_start_main
  27: _start
```
