# Key Modifiers

Key Modifiers is a simple script that lets you use some keys to modify the behavior of other keys. For example, I use <kbd>SHIFT</kbd> to modify lots of my regular keys, such as my mouse buttons and my arrow keys.

In theory, any key can be set-up to modify other keys, but it's probably best to designate a few "modifier" keys, such as <kbd>SHIFT</kbd>, <kbd>ALT</kbd>, and/or <kbd>CTRL</kbd>.

## Example

It's easier to explain using an example. I have another script, the [Hightower Practice Script](https://github.com/rufio-tf2/hightower-practice), which uses the numpad keys to teleport you around Hightower.

One user doesn't have a numpad though. He wants to set up an ALT modifier, to modify his regular number keys to use the Hightower Practice Script numpad actions.

This means that he wants his regular number keys to have two states. Each one will have an **unmodified state** and a **alt-modified state**. The unmodified state is the same thing as its **default state**. The alt-modified state, in this case, will perform the Hightower Practice Script numpad actions. (If he also wanted to shift-modify these keys, they would have three potential states.)

**For any key that I want to modify, I'll create a folder.** In this example, you'll see that I have folders for 0-9, as well as the <kbd>-</kbd> (minus) key which will perform the numpad period action <kbd>. (DEL)</kbd> (`KP_DEL`).

Each key's folder gets two base files:

- `default.cfg` -- The key's unmodified state
- `bind.cfg` -- Bind the key to its state variable

Scripting in TF2 is a little janky, and the system doesn't seem to handle it very well when we use some keys to re-bind other keys.

Rather than using modifier keys (like Shift or Alt) to re-bind what some keys do, I use a system where I set up keys to only be bound to a designated variable, and then use modifier keys to change the definition of those variables.

In the <kbd>1</kbd> key's [`bind.cfg`](key-modifiers/1/bind.cfg) file, you'll see this line:

```go
// 1/bind.cfg
bind 1 oneState
```

Then, in its [`default.cfg`](key-modifiers/minus/default.cfg) file, I establish its default, unmodified state:

```go
// 1/default.cfg
alias oneState slot1
```

The `slot1` action is the <kbd>1</kbd> key's default action, which switches to your first weapon slot. (Here's the [full list of default keys](https://wiki.teamfortress.com/wiki/List_of_default_keys).)

Okay, now we can create modified states. I've included an `alt.cfg` and `shift.cfg` file for all of these keys, but in this example we'll just mess with the `alt.cfg` files.

In the [`alt.cfg`](key-modifiers/1/alt.cfg) file, you can see its alt-modified state

```go
// 1/alt.cfg
alias oneState htp_kp1
```

That `htp_kp1` action comes from the Hightower Practice Script, and refers to the numpad <kbd>1 (END)</kbd> (`KP_END`).

Finally, the keys' states need to be wired in to the script files that actually change the states. These script files are [`altModify.cfg`](key-modifiers/altModify.cfg), [`shiftModify.cfg`](key-modifiers/shiftModify.cfg), and [`resetKeys.cfg`](key-modifiers/resetKeys.cfg). Again, in this example we'll ignore the shift-modifier stuff, but working with it would be equivalent to working with the alt-modifier.

In these files, you need to execute your keys' respective files. I've already done this step for these keys so that you can see what I mean:

```go
// resetKeys.cfg
exec key-modifiers/1/default
```

```go
// altModify.cfg
exec key-modifiers/1/alt
```

In the root [`init.cfg`](key-modifiers/init.cfg) file, I have the <kbd>ALT</kbd> key set to execute `altModify.cfg` when you press down, and then to execute `resetKeys.cfg` when you let up, thus switching the keys between the two states. (I also have the <kbd>SHIFT</kbd> key set up to execute `shiftModify.cfg`).

## A (slightly) harder example

In the example above, the <kbd>1</kbd> key's actions are:

- Unmodified: `slot1`
- Alt-modified: `htp_kp1`

Neither of these actions are "plus/minus" (`+/-`) actions. Plus/minus actions are special Source Engine commands. They tend to be actions that keep happening while a key is pressed, and then stop when the key is released. For example, using a weapon's primary attack with <kbd>MOUSE1</kbd>.

The default action for <kbd>MOUSE1</kbd> is `+attack`. When you press down <kbd>MOUSE1</kbd> it executes `+attack`, and when you release the button it executes `-attack`.

This means that I need to create (and maintain) both a plus and a minus version of its state variable.

The <kbd>MOUSE1</kbd> [`default.cfg`][./mouse1/default.cfg] file looks like this:

```go
// mouse1/default.cfg
alias +mouse1State +attack
alias -mouse1State -attack
```

This means that I have to bind MOUSE1 to its plus state:

```go
// mouse1/bind.cfg
bind MOUSE1 +mouse1State
```

As an example, let's pretend that I want <kbd>ALT</kbd>+<kbd>MOUSE1</kbd> to make me jump. In its [`alt.cfg`](key-modifiers/mouse1/alt.cfg) file, I **redefine both the plus state and the minus state**:

```go
// mouse1/alt.cfg
alias +mouse1State +jump
alias -mouse1State -jump
```

In this example, `+jump` is a plus/minus command like `+attack`, so they align with the `+mouse1State` and `-mouse1State`.

One last example. Imagine that I want MOUSE1's alt-modified state to say something funny, unique, clever, and witty about random crits to the server so win at the social game as well as the video game and they'll be impressed and think that I'm a hilarious genius, then I'd use the `say` command.

The `say` command isn't a plus/minus command, so it doesn't obviously align with `+mouse1State` and `-mouse1State`. This is what I'd do:

First, create an alias to contain my wit. Then, bind the **plus** state to release that wit. Finally, bind the **minus** state to "noop" (pronounced no-op, as in "no operation"). This is because there is no minus-version of the `say` command. (In TF2 scripting, I define `noop` as just empty quotes.)

```go
// mouse1/alt.cfg
alias spewMyWit "say Think I'm Oscar Wilde, but really just Dorian Gray"
alias +mouse1State spewMyWit
alias -mouse1State ""
```

So MOUSE1's unmodified state attacks as usual, and its alt-modified state releases my wit in the server's face.

Finally lastly, you need to wire-up MOUSE1's modifiers to the controller files:

```go
// resetKeys.cfg
exec key-modifiers/mouse1/default
```

```go
// altModify.cfg
exec key-modifiers/mouse1/alt
```

## Install

1. [Download](https://github.com/rufio-tf2/key-modifiers/archive/master.zip) this, extract it, and move the `key-modifier` subfolder into your `custom/YOUR_CONFIG/cfg` folder (replace `YOUR_CONFIG` with the name of your config)
1. In your `autoexec.cfg`, wire it up by adding this line at the top:

   ```go
   // autoexec.cfg
   exec key-modifiers/init
   ```
