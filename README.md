# Ruby 1.8 for Emscripten

tl;dr Here you can get the whole Ruby 1.8 interpreter (without extensions) compatible with emscripten.

**Fixing this was no fun at all** but I hope you will have more fun in using it :).

## Compiling

Works as usual:

 1. Run `emconfigure configure` and `make` (don't use `make install`) until you reach an execvp miniruby error.
 2. Copy miniruby from a native build over
 3. Run `make`
 4. Run `make install`

## How it works

Ruby 1.8 is full of bad function pointer calls (signature mismatch) which I "simply" replaced with correct function pointers. This was mostly done by running ruby scripts through a ruby executable instrumented via clang `-fsanitize-cfi-icall-generalize-pointers -fsanitize=cfi-icall`. This simulates an emscripten environment pretty good because it will crash with an illegal instruction when the function pointers mismatch. Then you fix that one crash, recompile, rerun, and so on until it doesn't crash anymore...

Please report any emscripten asserts you encounter, I probably missed some because I can only fix crashes that I encountered in the scripts I used for testing.

## Emterpreter Whitelist

When your application uses ruby that calls into your custom C(++) code and you need to emscripten_sleep use this Emterpreter whitelist:

```
-s EMTERPRETIFY_WHITELIST=\"[
  '_call_cfunc','_rb_call0','_rb_call','_rb_eval',
  '_rb_eval_string','_eval_node','_eval','_rb_f_eval','_rb_funcall2','_rb_protect',
  '_rb_yield_0','_loop_i','_rb_rescue2','_rb_f_loop','_module_setup'
]\""
```

## Troubleshooting

In case you want to go bad pointer hunting yourself and get errors during linking that the library compiled with `-fsanitize=cli-icall` is corrupted simply replace the `-lruby-static` with `libruby-static.a` (don't ask my why but it works).
