# Key Modifiers

Key Modifiers is a simple script that lets me use some keys to modify the behavior of other keys. For example, I use <kbd>SHIFT</kbd> to modify lots of my regular keys, such as my mouse buttons and my arrow keys.

In theory, any key can be set-up to modify other keys, but it's probably best to designate a few "modifier" keys, such as <kbd>SHIFT</kbd>, <kbd>ALT</kbd>, and/or <kbd>CTRL</kbd>.

## How to use it

I've got this set up to show the following examples of how to use it. The example code is commented out. Feel free to delete any of the stuff you aren't using:

- Clean up key files from the [`keys/`](./key-modifiers/keys) folder
- Remove unused from [`resetKeys.cfg`](./key-modifiers/resetKeys.cfg) file
- Remove unused from [`altModify.cfg`](./key-modifiers/altModify.cfg) file

### Example

It's easier to explain how to use this by walking through an example. I have another script, the [Hightower Practice Script](https://github.com/rufio-tf2/hightower-practice), which uses the numpad keys to teleport me around Hightower.

One user doesn't have a numpad, though. He wants to set up an <kbd>ALT</kbd> modifier to modify his regular number keys, so that when he holds <kbd>ALT</kbd> and presses a number it will perform its equivalent Hightower Practice Script numpad action.

He wants his regular number keys to have two states. Each one will have an **unmodified state** and an **alt-modified state**. The unmodified state is the same thing as its default state. The alt-modified state, in this case, will perform its equivalent numpad action. (If he also wanted to shift-modify these keys, they would have three potential states.)

**For any key that I want to modify, I'll create a file for it inside of the [`keys/`](./key-modifiers/keys/) folder.** In this example, I have files for the <kbd>0</kbd>-<kbd>9</kbd> keys, as well as for the <kbd>-</kbd> (minus) key which I'll use to perform the numpad period action <kbd>. (DEL)</kbd> (`KP_DEL`). I also have a `mouse1.cfg` for a [later example](#a-slightly-harder-example).

This is what the file for the <kbd>1</kbd> key does ([`1.cfg`](./key-modifiers/keys/1.cfg)):

- Defines a default state (`alias 1Default slot1`)
- Starts the key's state variable (`1State`) at its unmodified state (`alias 1State 1Default`)
- Binds the key to its state variable (`bind 1 1State`)

Then, since I want to alt-modify the key, I'll also add that state's definition (commented out).

```go
// 1.cfg
alias 1Default slot1
alias 1Alt htp_kp1 // Create an alt-modified state

alias 1State 1Default
bind 1 1State
```

Scripting in TF2 is a little janky, and (apparently) the system doesn't seem to handle it very well when we use some keys to re-bind other keys.

Rather than using modifier keys (like <kbd>ALT</kbd>) to re-bind what other keys do, this system sets keys to only be bound to a designated variable, and then uses modifier keys to mutate the definition of those variables without having to re-bind the key.

**Note:** The `slot1` action is the <kbd>1</kbd> key's default action, which switches to my first weapon slot. (Here's the [full list of default keys](https://wiki.teamfortress.com/wiki/List_of_default_keys).) The `htp_kp1` action comes from the Hightower Practice Script, and refers to the numpad <kbd>1 (END)</kbd> (`KP_END`).

Now, I need to have the controller files actually change these states. These "controller" files are:

- [`resetKeys.cfg`](key-modifiers/resetKeys.cfg)
- [`altModify.cfg`](key-modifiers/altModify.cfg),
- [`shiftModify.cfg`](key-modifiers/shiftModify.cfg)

I'm going to ignore the shift-modifier stuff, but working with it would be equivalent to working with the alt-modifier.

In these files, I need to mutate the key's state variable. I've already done this step for these keys so that there's an example to follow.

In the `resetKeys.cfg`, I'll set the `1State` to its default, unmodified state:

```go
// resetKeys.cfg
alias 1State 1Default
```

In the `altModify.cfg`, I'll set the `1State` to its alt-modified state (see the commented lines):

```go
// altModify.cfg
alias 1State 1Alt
```

In the root [`init.cfg`](key-modifiers/init.cfg) file, I have the <kbd>ALT</kbd> key set to execute `altModify.cfg` when I press down, and then to execute `resetKeys.cfg` when I let up, thus switching the keys between these two states. (I also have the <kbd>SHIFT</kbd> key set up to execute `shiftModify.cfg`).

### A (slightly) harder example

In the example above, the <kbd>1</kbd> key's actions are:

- Unmodified: `slot1`
- Alt-modified: `htp_kp1`

Neither of these actions are "plus/minus" (`+/-`) actions. Plus/minus actions are special Source Engine commands. They tend to be actions that keep happening while a key is pressed, and then stop when the key is released. For example, using a weapon's primary attack with <kbd>MOUSE1</kbd>.

The default action for <kbd>MOUSE1</kbd> is `+attack`. When I press down <kbd>MOUSE1</kbd> it executes `+attack`, and when I release the button it stops attacking by executing `-attack`.

In the Hightower Practice Script, however, the default state for <kbd>MOUSE1</kbd> is `+CUSTOM_attack`. I've found that using an intermediate variable like that allows for more flexibility. (Note, the `default.cfg` file in the Hightower Practice Script would need additional work, but that's out of scope for this.)

I'll put this at the top of my file:

```
alias +CUSTOM_attack +attack; alias -CUSTOM_attack -attack;
```

In this example, let's pretend that I want the alt-modified state to make my character jump, which is `+jump` and `-jump`.

To set this up, I do the same things that I did in the first example, but now I create these plus/minus versions of everything. Finally, I bind `MOUSE1` to the plus-version of its state variable.

The [`mouse1.cfg`](./key-modifiers/keys/mouse1.cfg) file looks like:

```go
// keys/mouse1.cfg
alias +CUSTOM_attack +attack; alias -CUSTOM_attack -attack;

alias +mouse1Default +CUSTOM_attack
alias -mouse1Default -CUSTOM_attack

alias +mouse1Alt +jump
alias -mouse1Alt -jump

alias +mouse1State +mouse1Default
alias -mouse1State -mouse1Default

bind MOUSE1 +mouse1State
```

Then, to wire this up in the controller files, I need to change both the plus and minus versions of the <kbd>MOUSE1</kbd> button's state variable:

```go
// altModify.cfg
alias +mouse1State +mouse1Default; alias -mouse1State -mouse1Default;
```

```go
// altModify.cfg
alias +mouse1State +mouse1Alt; alias -mouse1State -mouse1Alt;
```

#### One last example

Imagine that, instead of jumping, I want <kbd>ALT</kbd>+<kbd>MOUSE1</kbd> to say something to the server (something clever and witty about random crits, for example).

```
alias spewMyWit "say I think I'm Oscar Wilde, but I'm really just Dorian Gray"
```

The `say` command isn't a plus/minus command, so it doesn't obviously align with `+mouse1State` and `-mouse1State`.

To handle this, I'll set the plus command to our witty action, and I'll set the minus command to perform nothing:

```go
alias +mouse1Alt spewMyWit
alias -mouse1Alt ""
```

This would be the full file:

```go
// mouse1/alt.cfg
alias spewMyWit "say I think I'm Oscar Wilde, but I'm really just Dorian Gray"
alias +CUSTOM_attack +attack; alias -CUSTOM_attack -attack;

alias +mouse1Default +CUSTOM_attack
alias -mouse1Default -CUSTOM_attack

alias +mouse1Alt spewMyWit
alias -mouse1Alt ""

alias +mouse1State +mouse1Default
alias -mouse1State -mouse1Default

bind MOUSE1 +mouse1State
```

So MOUSE1's unmodified state attacks as usual, and its alt-modified state releases my wit in the server's face.

### Create your own

Create your own following the examples above. Take note of:

- Default action
- Modified actions

If any of those are plus/minus commands, you'll need to account for that. It's annoying, I know. But, we're scripting.

## Install

1. [Download](https://github.com/rufio-tf2/key-modifiers/archive/master.zip) this, extract it, and move the `key-modifier` subfolder into your `custom/YOUR_CONFIG/cfg` folder (replace `YOUR_CONFIG` with the name of your config)
1. In your `autoexec.cfg`, wire it up by adding this line at the top:

   ```go
   // autoexec.cfg
   exec key-modifiers/init
   ```
