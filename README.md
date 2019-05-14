# Key Modifiers

Key Modifiers is a simple script that lets me use some keys to modify the behavior of other keys. For example, I use <kbd>SHIFT</kbd> to modify lots of my regular keys, such as my mouse buttons and my arrow keys.

In theory, any key can be set-up to modify other keys, but it's probably best to designate a few "modifier" keys, such as <kbd>SHIFT</kbd>, <kbd>ALT</kbd>, and/or <kbd>CTRL</kbd>.

## How to use it

### Example

It's easier to explain how to use this by walking through an example. I have another script, the [Hightower Practice Script](https://github.com/rufio-tf2/hightower-practice), which uses the numpad keys to teleport me around Hightower.

One user doesn't have a numpad, though. He wants to set up an <kbd>ALT</kbd> modifier to modify his regular number keys, so that when he holds <kbd>ALT</kbd> and presses a number it will perform its equivalent Hightower Practice Script numpad action.

He wants his regular number keys to have two states. Each one will have an **unmodified state** and a **alt-modified state**. The unmodified state is the same thing as its default state. The alt-modified state, in this case, will perform its equivalent numpad action. (If he also wanted to shift-modify these keys, they would have three potential states.)

**For any key that I want to modify, I'll create a folder.** In this example, I have folders for `0`-`9`, as well as the <kbd>-</kbd> (minus) key which will perform the numpad period action <kbd>. (DEL)</kbd> (`KP_DEL`). I also have a `mouse1` for a [later example](#a-slightly-harder-example).

Each key's folder gets two base files, and then a file for each modifier key that I want to use on it:

- `bind.cfg` -- Bind the key to its state variable
- `default.cfg` -- The key's unmodified state
- `alt.cfg` -- The key's alt-modified state
- `shift.cfg` -- The key's shift-modified state

Scripting in TF2 is a little janky, and the system doesn't seem to handle it very well when we use some keys to re-bind other keys.

Rather than using modifier keys (like <kbd>ALT</kbd>) to re-bind what other keys do, I use a system where I set up keys to only be bound to a designated variable, and then use modifier keys to mutate the definition of those variables.

In the <kbd>1</kbd> key's [`bind.cfg`](key-modifiers/1/bind.cfg) file, I bind the <kbd>1</kbd> key to a variable that will contain the key's state:

```go
// 1/bind.cfg
bind 1 oneState
```

Then, in its [`default.cfg`](key-modifiers/minus/default.cfg) file, I set that state variable to its default, unmodified state:

```go
// 1/default.cfg
alias oneState slot1
```

**Note:** The `slot1` action is the <kbd>1</kbd> key's default action, which switches to my first weapon slot. (Here's the [full list of default keys](https://wiki.teamfortress.com/wiki/List_of_default_keys).)

Now we can create any modified states that we want, in this example we want an alt-modified state.

In the [`alt.cfg`](key-modifiers/1/alt.cfg) file, I set the state variable to the alt-modified state:

```go
// 1/alt.cfg
alias oneState htp_kp1
```

**Note:** That `htp_kp1` action comes from the Hightower Practice Script, and refers to the numpad <kbd>1 (END)</kbd> (`KP_END`).

Once I finish creating these files, they need to be wired into the controller files that actually change these states. These script files are [`altModify.cfg`](key-modifiers/altModify.cfg), [`shiftModify.cfg`](key-modifiers/shiftModify.cfg), and [`resetKeys.cfg`](key-modifiers/resetKeys.cfg). I'm going to ignore the shift-modifier stuff, but working with it would be equivalent to working with the alt-modifier.

In these files, I need to execute the keys' respective files. I've already done this step for these keys so that there's an example to follow:

```go
// resetKeys.cfg
exec key-modifiers/1/default
```

```go
// altModify.cfg
exec key-modifiers/1/alt
```

In the root [`init.cfg`](key-modifiers/init.cfg) file, I have the <kbd>ALT</kbd> key set to execute `altModify.cfg` when I press down, and then to execute `resetKeys.cfg` when I let up, thus switching the keys between the two states. (I also have the <kbd>SHIFT</kbd> key set up to execute `shiftModify.cfg`).

### A (slightly) harder example

In the example above, the <kbd>1</kbd> key's actions are:

- Unmodified: `slot1`
- Alt-modified: `htp_kp1`

Neither of these actions are "plus/minus" (`+/-`) actions. Plus/minus actions are special Source Engine commands. They tend to be actions that keep happening while a key is pressed, and then stop when the key is released. For example, using a weapon's primary attack with <kbd>MOUSE1</kbd>.

The default action for <kbd>MOUSE1</kbd> is `+attack`. When I press down <kbd>MOUSE1</kbd> it executes `+attack`, and when I release the button it stops attacking by executing `-attack`.

To handle these plus/minus commands, I need to create (and maintain) both a plus and a minus version of the state variable.

In the <kbd>MOUSE1</kbd> [`default.cfg`][./mouse1/default.cfg] file, I create a plus and minus version of the state variable:

```go
// mouse1/default.cfg
alias +mouse1State +attack
alias -mouse1State -attack
```

Then, I bind MOUSE1 to the **plus** state:

```go
// mouse1/bind.cfg
bind MOUSE1 +mouse1State
```

As an example for an alt-modified state, let's pretend that I want <kbd>ALT</kbd>+<kbd>MOUSE1</kbd> to make me jump. In its [`alt.cfg`](key-modifiers/mouse1/alt.cfg) file, I **redefine both the plus state and the minus state**:

```go
// mouse1/alt.cfg
alias +mouse1State +jump
alias -mouse1State -jump
```

In this example, `+jump` is a plus/minus command like `+attack`, so they align with the `+mouse1State` and `-mouse1State`.

#### One last example

Imagine that, instead of jumping, I want <kbd>ALT</kbd>+<kbd>MOUSE1</kbd> to say something to the server (something clever and witty about random crits, for example). The `say` command isn't a plus/minus command, so it doesn't obviously align with `+mouse1State` and `-mouse1State`.

1. I'll create an alias to contain my wit.
1. `bind` the **plus** state to release that wit (even though this alias doesn't have a plus-version)
1. `bind` the **minus** state to "noop" (pronounced no-op, as in "no operation"), because there is no minus-version of the `say` command. (In TF2 scripting, I define `noop` as just empty quotes.)

Here's what the file would look like:

```go
// mouse1/alt.cfg
alias spewMyWit "say Think I'm Oscar Wilde, but really just Dorian Gray"
alias +mouse1State spewMyWit
alias -mouse1State "" // noop
```

So MOUSE1's unmodified state attacks as usual, and its alt-modified state releases my wit in the server's face.

Finally lastly, I need to wire-up MOUSE1's modifiers to the controller files:

```go
// resetKeys.cfg
exec key-modifiers/mouse1/default
```

```go
// altModify.cfg
exec key-modifiers/mouse1/alt
```

### Create your own

If you don't care about modifying the keys that I've included in this example, just rename my folders (and update the scripts) or delete them. To create your own:

1. Make a folder for each key that you want to modify
1. Add a `default.cfg`, `bind.cfg`, and any modifier files you want (`alt.cfg`, `shift.cfg`, etc.)
1. Define its unmodified state in its `default.cfg` file
1. Define its modified state(s) in its modifier files
1. Bind the key to its state variable in its `bind.cfg` file
1. **Remember to bind it to the plus-version of the state if necessary**

## Install

1. [Download](https://github.com/rufio-tf2/key-modifiers/archive/master.zip) this, extract it, and move the `key-modifier` subfolder into your `custom/YOUR_CONFIG/cfg` folder (replace `YOUR_CONFIG` with the name of your config)
1. In your `autoexec.cfg`, wire it up by adding this line at the top:

   ```go
   // autoexec.cfg
   exec key-modifiers/init
   ```
