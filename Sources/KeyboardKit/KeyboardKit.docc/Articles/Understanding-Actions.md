# Understanding Actions

This article describes the KeyboardKit action engine.

In KeyboardKit, the ``KeyboardAction`` enum defines keyboard-specific actions that can be bound to buttons and handled with a ``KeyboardActionHandler``, which can be used to handle actions, trigger feedback, etc.

[KeyboardKit Pro][Pro] unlocks a pro action handler that automatically registers the most recently used emojis. Information about Pro features can be found at the end of this article.



## Keyboard actions

The ``KeyboardAction`` enum contains a bunch of actions, for instance:

* ``KeyboardAction/character(_:)`` - inserts a text character.
* ``KeyboardAction/backspace`` - deletes backwards.
* ``KeyboardAction/dismissKeyboard`` - dismisses the keyboard.
* ``KeyboardAction/keyboardType(_:)`` - changes the keyboard type.
* ``KeyboardAction/nextKeyboard`` - triggers the system keyboard switcher.
* ``KeyboardAction/nextLocale`` - triggers the locale switcher action.

See the ``KeyboardAction`` documentation for a list of all available action types.



## How to handle keyboard actions

Keyboard actions can be handled with a ``KeyboardActionHandler``, which is a protocol that can be implemented by any class that can handle keyboard actions.

KeyboardKit registers a ``StandardKeyboardActionHandler`` with ``KeyboardInputViewController/services`` when a keyboard is loaded. You can modify or replace this instance at any time.

KeyboardKit will automatically handle actions whever the user interacts with the keyboard, or when certain system events happen. You can intercept these actions by creating a custom action handler.

You can trigger actions programmatically by calling `handle(_:)` or `handle(_:on:)` to trigger an action with an optional gesture:

```swift
func doStuff(with actionHandler: KeyboardActionHandler) {
    actionHandler.handle(.backspace)
    actionHandler.handle(.press, on: .character("a")
    actionHandler.handle(.release, on: .dismissKeyboard)
}
```

Keyboard actions can also be triggered by a ``KeyboardButton/Button`` or by applying `.keyboardButton(...)` to any view:

```swift
Text("Button")
    .keyboardButton(...)
```

``SystemKeyboard`` will automatically apply this modifier to all its buttons, which makes them support gestures for press, release, long press, repeat press, drag, etc.



## How to handle autocomplete suggestions

A ``KeyboardActionHandler`` can also handle ``Autocomplete/Suggestion`` items, since it must be able to do so when handling various actions and gestures.

You can handle suggestions programmatically by calling `handle(_:)`:

```swift
func doStuff(with actionHandler: KeyboardActionHandler) {
    let suggestion = Autocomlete.Suggestion(text: "Hello")
    actionHandler.handle(suggestion)
}
```

This will by default insert the provided suggestion into the text document proxy and reset the autocomplete state. You can customize this behavior by creating a custom action handler.



## How to create a custom action handler

You must create a custom action handler to handle actions that don't have a default behavior, like ``KeyboardAction/image``, or customize the standard behavior of certain actions or gestures.

You can either inherit ``StandardKeyboardActionHandler`` and customize what you want, or implement the ``KeyboardActionHandler`` protocol from scratch. 

For instance, here's a custom handler that inherits ``StandardKeyboardActionHandler`` and extends it with the image capabilities:

```swift
class CustomActionHandler: StandardKeyboardActionHandler {
    
    init(inputViewController: KeyboardInputViewController) {
        super.init(inputViewController: inputViewController)
    }
    
    override func action(
        for gesture: KeyboardGesture, 
        on action: KeyboardAction
    ) -> KeyboardAction.GestureAction? {
        let standard = super.action(for: gesture, on: action)
        switch gesture {
        case .longPress: return longPressAction(for: action) ?? standard
        case .release: return releaseAction(for: action) ?? standard
        default: return standard
        }
    }
    
    func longPressAction(for action: KeyboardAction) -> KeyboardAction.GestureAction? {
        switch action {
        case .image(_, _, let imageName): return { [weak self] _ in self?.saveImage(named: imageName) }
        default: return nil
        }
    }
    
    func releaseAction(for action: KeyboardAction) -> KeyboardAction.GestureAction? {
        switch action {
        case .image(_, _, let imageName): return { [weak self] _ in self?.copyImage(named: imageName) }
        default: return nil
        }
    }
}
```

To use this handler instead of the standard one, just set the ``KeyboardInputViewController/services`` action handler to this custom handler:

```swift
class KeyboardViewController: KeyboardInputViewController {

    override func viewDidLoad() {
        services.actionHandler = CustomActionHandler(inputViewController: self)
        super.viewDidLoad()
    }
}
```

This will make KeyboardKit use your custom implementation instead of the standard one.



## 👑 Pro features

[KeyboardKit Pro][Pro] injects a ``StandardKeyboardActionHandler/emojiRegistrationAction`` into the ``StandardKeyboardActionHandler``, that automatically registers emojis as you use them. This will automatically update the "most recent" emojis category in the emoji keyboard.



[Pro]: https://github.com/KeyboardKit/KeyboardKitPro
