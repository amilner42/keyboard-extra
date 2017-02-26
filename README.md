# Keyboard Extra

Forked from: https://github.com/ohanhi/keyboard-extra.

### Description

Intended to keep track of the current keys being pressed, but to avoid bugs
where `onKeyUp` does not always get triggered for all the keys pressed down,
this only keeps track of the last 3 keys. This way if a key gets added for
`onKeyDown` and doesn't get removed for `onKeyUp`, it gets cleared pretty
quickly anyway in just few key-presses.

[Stack Overflow Post Talking about missing `onKeyUp` event](http://stackoverflow.com/questions/27380018/when-cmd-key-is-kept-pressed-keyup-is-not-triggered-for-any-other-key)

This library is not designed for games, it's designed for apps where you want
to add hotkeys to trigger certain events.

If you have hotkeys that are longer than 3-keys, then this library will not
work for you, if you have that use-case make an issue and I can _possibly_
change the code up a bit.

### Usage

Add it to your Model:

```elm
type alias Model =
    { keysDown : Keyboard.Extra.Model
    -- ...
    }
```

Add it to your init:

```elm
init =
    { keysDown = Keyboard.Extra.init
    -- ...
    }

-- If your init also expects a Cmd then pair it with Cmd.none.
```

Add it to your messages:

```elm
type Msg =
    KeyboardExtraMsg Keyboard.Extra.Msg
    -- ...
```

Add it to your update.

```elm
case msg of
    KeyboardExtraMsg keyMsg ->
        let
            newKeysDown =
                Keyboard.Extra.update keyMsg model.keysDown
        in
            -- If you want to react to key-presses, call a function here instead
            -- of just updating the model (you should still update the model).
            ({ model | keysDown = newKeysDown }, Cmd.none)
    -- ...
```

And lastly, hook up your subscriptions:

```elm
subscriptions model =
    Sub.batch
       [ Sub.map KeyboardExtraMsg Keyboard.Extra.subscriptions
       -- ...
       ]
```

### Common Pitfalls

Let's say you add a hotkey for a double-key, like shift-tab, and you want the
user to not only be able to click shift-tab multiple times, but click shift-tab
and hold on to shift again but keep clicking tab. This is relatively standard
behaviour and someone is _likely_ to do this over letting go of the shift every
time. Let's take a look at what happens:

Model:

```
[]
-> [Shift]
-> [Tab, Shift]
-> [Nothing, Shift]
-> [Tab, Nothing, Shift]
-> [Nothing, Nothing, Shift]
-> [Tab, Nothing, Nothing]
```

And as the `Shift` walks off the edge the user will still be holding it but
will get the behaviour of a `Tab`! Not good...

To avoid this you can do a basic check for any of your double-keys that you
want to permit this type of behaviour for and simply reset it so:

`[Tab, Nothing, Shift] -> [Tab, Shift]`

Example code to do this:

```elm
newKeysDown =
    KK.update msg model.shared.keysDown
   
newKeysDownAllowingShiftTab =
    case newKeysDown of
        [ Just key1, Nothing, Just key2 ] ->
            if
                ((KK.fromCode key1) == KK.Tab)
                    && ((KK.fromCode key2) == KK.Shift)
            then
               [ Just key1, Just key2 ]
            else
                newKeysDown
```

That way the user can hold shift while tabbing as much as they want.

I'd like to be able to handle this logic inside the library itself, but
I haven't though of a way to generalize it to _any keys_. If I always
watch for the pattern `[Just a, Nothing, Just b]` and update that to
`[Just a, Just b]` then the entire point of this library removing
"ghost keys" gets ruined because keys that get stuck in there could
get stuck for a long time.

