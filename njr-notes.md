# Can use this instead of nix.

sudo pacman -S base-devel python rust pkgconfig inetutils djvulibre openjpeg2 \
               clang sdl2 jbig2dec harfbuzz gumbo-parser xwayland-satellite \
               mupdf unzip


## Nix and devenv a docker container

```
# Install nix in single user mode, so that we don't need to start the
# nix-daemon service--We don't have systemd available.
curl -L https://nixos.org/nix/install | sh -s -- --no-daemon

# Enable experimental features needed for
mkdir -p ~/.config/nix && \
    echo "experimental-features = nix-command flakes" >> ~/.config/nix/nix.conf

. /home/njr/.nix-profile/etc/profile.d/nix.sh

nix-env --install --attr devenv -f https://github.com/NixOS/nixpkgs/tarball/nixpkgs-unstable

devenv shell

sudo pacman -S inetutils
```


export XDG_RUNTIME_DIR=/run/user/1000
export WAYLAND_DISPLAY=wayland-1

----

Super weird mutable reference to children!

Why does crates/core/src/view/icon.rs deal
with

                        Event::ToggleFrontlight
                        Event::ToggleNear(ViewId::MarginCropperMenu, self.rect));
                        Event::History(dir, false) => {


--------------------------------------------------------------------------------

 Compiling sqlx-sqlite v0.9.0
error: could not find native static library `sqlite3`, perhaps an -L flag is missing?

error: could not compile `libsqlite3-sys` (lib) due to 1 previous error
warning: build failed, waiting for other jobs to finish...
Error: `cargo` exited with status 101

Stack backtrace:
   0: anyhow::error::<impl anyhow::Error>::msg
             at /home/njr/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/anyhow-1.0.103/src/backtrace.rs:10:14
   1: anyhow::__private::format_err
             at /home/njr/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/anyhow-1.0.103/src/lib.rs:690:13
   2: xtask_lib::tasks::util::cmd::check_status
             at ./xtask/src/tasks/util/cmd.rs:103:23
   3: xtask_lib::tasks::util::cmd::run
             at ./xtask/src/tasks/util/cmd.rs:47:5
   4: xtask_lib::tasks::run_emulator::run
             at ./xtask/src/tasks/run_emulator.rs:63:5
   5: xtask_lib::run
             at ./xtask/src/lib.rs:107:39
   6: xtask::main
             at ./xtask/src/main.rs:9:5
   7: core::ops::function::FnOnce::call_once
             at /rustc/ac68faa20c58cbccd01ee7208bf3b6e93a7d7f96/library/core/src/ops/function.rs:250:5
   8: std::sys::backtrace::__rust_begin_short_backtrace
             at /rustc/ac68faa20c58cbccd01ee7208bf3b6e93a7d7f96/library/std/src/sys/backtrace.rs:166:18
   9: std::rt::lang_start::{{closure}}
             at /rustc/ac68faa20c58cbccd01ee7208bf3b6e93a7d7f96/library/std/src/rt.rs:206:18
  10: <&dyn core::ops::function::Fn<(), Output = i32> + core::marker::Sync + core::panic::unwind_safe::RefUnwindSafe as core::ops::function
::FnOnce<()>>::call_once
             at /rustc/ac68faa20c58cbccd01ee7208bf3b6e93a7d7f96/library/core/src/ops/function.rs:287:21
  11: std::panicking::catch_unwind::do_call::<&dyn core::ops::function::Fn<(), Output = i32> + core::marker::Sync + core::panic::unwind_saf
e::RefUnwindSafe, i32>
             at /rustc/ac68faa20c58cbccd01ee7208bf3b6e93a7d7f96/library/std/src/panicking.rs:581:40
  12: std::panicking::catch_unwind::<i32, &dyn core::ops::function::Fn<(), Output = i32> + core::marker::Sync + core::panic::unwind_safe::R
efUnwindSafe>
             at /rustc/ac68faa20c58cbccd01ee7208bf3b6e93a7d7f96/library/std/src/panicking.rs:544:19
  13: std::panic::catch_unwind::<&dyn core::ops::function::Fn<(), Output = i32> + core::marker::Sync + core::panic::unwind_safe::RefUnwindS
afe, i32>
             at /rustc/ac68faa20c58cbccd01ee7208bf3b6e93a7d7f96/library/std/src/panic.rs:359:14
  14: std::rt::lang_start_internal::{closure#0}
             at /rustc/ac68faa20c58cbccd01ee7208bf3b6e93a7d7f96/library/std/src/rt.rs:175:24
  15: std::panicking::catch_unwind::do_call::<std::rt::lang_start_internal::{closure#0}, isize>
             at /rustc/ac68faa20c58cbccd01ee7208bf3b6e93a7d7f96/library/std/src/panicking.rs:581:40
  16: std::panicking::catch_unwind::<isize, std::rt::lang_start_internal::{closure#0}>
             at /rustc/ac68faa20c58cbccd01ee7208bf3b6e93a7d7f96/library/std/src/panicking.rs:544:19
  17: std::panic::catch_unwind::<std::rt::lang_start_internal::{closure#0}, isize>
             at /rustc/ac68faa20c58cbccd01ee7208bf3b6e93a7d7f96/library/std/src/panic.rs:359:14
  18: std::rt::lang_start_internal
             at /rustc/ac68faa20c58cbccd01ee7208bf3b6e93a7d7f96/library/std/src/rt.rs:171:5
  19: std::rt::lang_start
             at /rustc/ac68faa20c58cbccd01ee7208bf3b6e93a7d7f96/library/std/src/rt.rs:205:5
  20: main
  21: __libc_start_call_main
  22: __libc_start_main_alias_2
  23: _start

----

@OGKevin suggested using locate<T: View>, but that is only reliable
when there is exactly one child of a particular type.
In the case if the Icons, there are multiple. So I feel that
acessors and a warning about re-numbering is safer. 

The bigger issue seems to be that the `View.children()`
and `View.children_mut()` API is broken.

    fn children(&self) -> &Vec<Box<dyn View>>;
    fn children_mut(&mut self) -> &mut Vec<Box<dyn View>>;It's signature, 

Their return type, a reference to a Vec, forces Views to store their
children in a Vec. This doesn't allow me to, use a much simpler
interface for SliderWithButtons:

    pub struct SliderWithButtons {
    	...
	increment: Icon,
	slider: Slider,
	decrement: Icon,
    }

The second method also allows us to mutate :e 

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# On branch njr
# Changes to be committed:
#	modified:   crates/core/src/view/slider.rs
#
# ------------------------ >8 ------------------------
# Do not modify or remove the line above.
# Everything below it will be ignored.
diff --git a/crates/core/src/view/slider.rs b/crates/core/src/view/slider.rs
index 6cde01e..31756c0 100644
--- a/crates/core/src/view/slider.rs
+++ b/crates/core/src/view/slider.rs
@@ -258,9 +258,20 @@ impl SliderWithButtons {
         SliderWithButtons {
             rect: rect,
             id: ID_FEEDER.next(),
+            // WARNING: Keep in-sync with *_child accessors below.
             children: vec![Box::new(decrement), Box::new(slider), Box::new(increment)],
         }
     }
+
+    pub fn decrement_child(&mut self) -> &mut Icon {
+        self.children_mut()[0].downcast_mut::<Icon>().unwrap()
+    }
+    pub fn slider_child(&mut self) -> &mut Slider {
+        self.children_mut()[1].downcast_mut::<Slider>().unwrap()
+    }
+    pub fn increment_child(&mut self) -> &mut Icon {
+        self.children_mut()[2].downcast_mut::<Icon>().unwrap()
+    }
 }
 
 impl View for SliderWithButtons {
@@ -278,7 +289,7 @@ impl View for SliderWithButtons {
             Event::SliderIncrement(amount) => {
                 let id = self.id;
                 let rect = self.rect;
-                let slider = self.children_mut()[1].downcast_mut::<Slider>().unwrap();
+                let slider = self.slider_child();
                 slider.increment(amount, rq);
                 rq.add(RenderData::new(id, rect, UpdateMode::Gui));
                 bus.push_back(Event::Slider(
@@ -346,14 +357,14 @@ mod tests {
         let mut rq = RenderQueue::new();
         let mut context = create_test_context();
 
-        let dec_bounds = slider.child(0).rect();
+        let dec_bounds = slider.decrement_child().rect();
         let point = Point::new(
             dec_bounds.min.x + dec_bounds.max.x / 2,
             dec_bounds.min.y + dec_bounds.max.y / 2,
         );
         let tap_event = Event::Gesture(GestureEvent::Tap(point));
         crate::view::handle_event(
-            slider.child_mut(0),
+            slider.decrement_child(),
             &tap_event,
             &hub,
             &mut bus,
