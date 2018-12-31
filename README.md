# Keyboard Extra

Forked from: https://github.com/ohanhi/keyboard-extra.

NOTE: I haven't upgraded this to Elm 0.19, I don't intend on doing this either, I'm not sure I want to keep this
library alive.

### Description

A very lightweight library to create hotkeys for apps. Not intended for games.

###### Warning

[Javascript has issues with key events](http://stackoverflow.com/questions/27380018/when-cmd-key-is-kept-pressed-keyup-is-not-triggered-for-any-other-key)

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
