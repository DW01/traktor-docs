---
layout: default
permalink: /editor/localization-editor/
title: Localization Editor
parent: Editor
nav_order: 15
---

# Localization Editor (I18N)

![TODO: Screenshot of the Localization Editor showing a table with string IDs in the left column and translations in multiple language columns (English, German, Japanese, etc.) with full Unicode text visible]

The **Localization Editor**, often called the **I18N editor** (internationalization), is a specialized tool for managing translated text across multiple languages. Instead of hardcoding strings in your game scripts and UI, you reference them by ID, and the localization system provides the appropriate text for the player's language.

This approach makes adding new languages straightforward. Translate the strings, add a column to the localization table, and your game automatically supports the new language. No code changes required.

## What is Localization?

**Localization** (often abbreviated L10N or I18N) is the process of adapting your game for different languages and regions. This includes:
- Translating UI text, dialog, tutorials, and menus
- Handling text direction (left-to-right vs. right-to-left for Arabic/Hebrew)
- Adjusting formatting for dates, numbers, and currency
- Supporting Unicode for languages with non-Latin scripts (Chinese, Japanese, Korean, Russian, etc.)

Traktor's localization system focuses on **string tables**: structured data mapping string IDs to translations in each supported language.

## Opening the Localization Editor

1. Create a new localization asset in the Database: Right-click → **New instance** → Select `traktor.i18n.Dictionary`
2. Double-click the dictionary asset to open the Localization Editor

## Editor Layout

The Localization Editor displays a **table** with:

**String ID column (left)** - Unique identifiers for each string. These IDs are referenced in your scripts and UI. For example: `UI_START_GAME`, `DIALOG_WELCOME`, `ITEM_SWORD_NAME`.

**Language columns (right)** - One column per supported language. Each cell contains the translation for that string in that language. Add as many language columns as needed.

The editor has **full Unicode support**, correctly displaying all languages including Chinese, Japanese, Korean, Cyrillic, Arabic, and emoji.

## Adding Languages

**To add a new language column:**

1. Right-click the table header
2. Select **Add language** (or similar option)
3. Enter the language code (e.g., `en`, `de`, `ja`, `zh-CN`)
4. Click OK

A new column appears. You can now add translations for that language.

## Adding Strings

**To add a new translatable string:**

1. Right-click in the table
2. Select **Add string** (or click the **Add** button in the toolbar)
3. Enter a unique **String ID** (use consistent naming: `CATEGORY_CONTEXT_NAME`)
4. Enter translations for each language column
5. Save the dictionary

## Using Localization in Scripts

Reference localized strings by their ID in your game scripts:

```lua
-- Lua example
local text = traktor.i18n.get("UI_START_GAME")
print(text)  -- Displays "Start Game" (English) or "ゲームを始める" (Japanese)
```

The localization system automatically selects the appropriate language based on the player's settings or system locale.

## Best Practices

**Use descriptive IDs:** `DIALOG_MERCHANT_GREETING` is clearer than `STRING_042`. IDs should describe context and purpose.

**Organize by category:** Prefix IDs with category names. `UI_`, `DIALOG_`, `ITEM_`, `ERROR_`. This makes the table easier to navigate as it grows.

**Plan for text expansion:** Translated text can be significantly longer or shorter than the source language. German tends to be longer, Chinese shorter. Design UI with flexible text areas.

**Include context comments:** Add a notes column (if supported) or document IDs separately. Translators need context. "OK" could mean "okay" or "confirm" or "acknowledge" depending on usage.

**Test with real translations early:** Placeholder "Lorem ipsum" or machine translations reveal layout issues before professional translation.

**Version control the dictionary:** Localization assets are part of your game data. Track changes, review translations, and maintain history.

## See Also

- [UI](../engine/ui/) - Using localized text in user interfaces
- [Scripting](../engine/scripting/) - Accessing localization from scripts
- [Database](database/) - Managing localization assets
