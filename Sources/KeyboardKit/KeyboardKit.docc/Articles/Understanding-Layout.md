# Understanding Layout

This article describes the KeyboardKit layout engine.

A flexible keyboard layout is an important part of a software keyboard, where many factors like device, screen orientation, locale, etc. can affect the layout.

In KeyboardKit, ``InputSet``s and ``KeyboardLayout``s are important concepts when creating keyboard layouts, where an input set specifies input keys and the keyboard layout specifies the full set of keys and their layout.

[KeyboardKit Pro][Pro] unlocks and registers localized input sets and layout providers. Information about Pro features can be found at the end of this article.



## Input Sets

An ``InputSet`` set specifies the input keys of a keyboard.

KeyboardKit has some input sets, like ``InputSet/qwerty``, ``InputSet/standardNumeric(currency:)`` and ``InputSet/standardSymbolic(currencies:)``. You can also create your own custom input sets.



## Keyboard layouts

A ``KeyboardLayout`` specifies the full set of keys of a keyboard. 

Keyboard layouts can vary greatly for different device types, screen orientations, locales, etc. For instance, iOS keyboards often have 3 input rows, where the input keys are surrounded by actions on either side, as well as a bottom row with a space key and action buttons. 

This is however not true for all locales. For instance, Armenian has 4 input rows, Greek removes many side-buttons, Arabic removes the shift key, etc. These differences can be significat, so the layout engine has to be flexible. 



## Keyboard layout providers

In KeyboardKit, a ``KeyboardLayoutProvider`` can be used to create a dynamic layout based on many different factors, such as the current device type, orientation, locale, etc.

KeyboardKit registers a ``StandardKeyboardLayoutProvider`` with ``KeyboardInputViewController/services`` when a keyboard is loaded. It has a QWERTY layout by default, but you can inject localized providers into it or modify it, or replace it, at any time.

You can add and replace localized providers to the default ``StandardKeyboardLayoutProvider``, or replace the ``KeyboardInputViewController/services`` provider with a custom ``KeyboardLayoutProvider``.



## How to create a custom provider

You can create a custom ``KeyboardLayoutProvider`` by inheriting ``StandardKeyboardLayoutProvider`` and customize what want, or implement the ``KeyboardLayoutProvider`` protocol from scratch.

There are also other base classes, such as ``BaseKeyboardLayoutProvider``, ``InputSetBasedKeyboardLayoutProvider``, ``iPadKeyboardLayoutProvider`` and ``iPhoneKeyboardLayoutProvider``. 

For instance, here's a custom provider that inherits ``StandardKeyboardLayoutProvider`` and injects a tab key into the layout:

```swift
class CustomKeyboardLayoutProvider: StandardKeyboardLayoutProvider {
    
    override func keyboardLayout(for context: KeyboardContext) -> KeyboardLayout {
        let layout = super.keyboardLayout(for: context)
        var rows = layout.itemRows
        var row = layout.itemRows[0]
        let next = row[0]
        let size = KeyboardLayout.ItemSize(width: .available, height: next.size.height)
        let tab = KeyboardLayout.Item(action: .tab, size: size, insets: next.insets)
        row.insert(tab, at: 0)
        rows[0] = row
        layout.itemRows = rows
        return layout
    }
}
```

To use this provider instead of the standard one, just set the ``KeyboardInputViewController/services`` provider to this custom provider:

```swift
class KeyboardViewController: KeyboardInputViewController {

    override func viewDidLoad() {
services.keyboardLayoutProvider = CustomKeyboardLayoutProvider()
        super.viewDidLoad()
    }
}
```

This will make KeyboardKit use your custom implementation instead of the standard one.



## 👑 Pro features

[KeyboardKit Pro][Pro] unlocks a ``KeyboardLayoutProvider`` for every locale that your license unlocks, and automatically registers them with the ``StandardKeyboardLayoutProvider``.


### How to access your localized providers

You can access all localized providers in your license like this:

```swift
let providers = License.current.localizedKeyboardLayoutProviders
```

or any locale-specific provider like this:

```swift
let provider = try ProKeyboardLayoutProvider.Swedish()
```

> Note: A localized provider will throw a license error if the locale isn't unlocked.  


### How to customize a localized provider

You can inherit and customize any localized provider:

```swift
class CustomProvider: ProKeyboardLayoutProvider.Swedish {

    override func keyboardLayout(for context: KeyboardContext) -> KeyboardLayout {
        var layout = keyboardLayoutProvider(for: context)
            .keyboardLayout(for: context)
        layout.tryInsertFlag(for: context)
        return layout
    }
}

private extension KeyboardLayout {

    func tryInsertFlag() {
        guard let item = tryCreateBottomRowItem(for:  .character("🇸🇪")) else { return }
        itemRows.insert(item, after: .space, atRow: bottomRowIndex)
    }
}
```

You can register this provider *after* registering your license key:

```swift
class KeyboardController: KeyboardInputViewController {

    override func viewWillSetupKeyboard() {
        super.viewWillSetupKeyboard()

        setupPro(withLicenseKey: "...") { license in
            setupCustomProvider()
        } view: { controller in
            // Return your keyboard view here
        }
    }

    func setupCustomProvider() {
        do {
            let provider = try CustomProvider()
            let standard = services.calloutActionProvider as? StandardCalloutActionProvider
            standard?.registerLocalizedProvider(provider)
        } catch {
            print(error)
        }
    }
}
```

Note that the provider cast will fail if you replace the instance.


[Pro]: https://github.com/KeyboardKit/KeyboardKitPro
