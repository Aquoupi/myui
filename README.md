# KNCRYPT HUB — UI Library Documentation

> **Author:** 4lpaca | **Version:** 0.1 | **Runtime:** Luau (Roblox)

---

## Table of Contents

1. [Overview](#overview)
2. [Global Properties](#global-properties)
3. [Creating a Window](#creating-a-window)
4. [Window Methods](#window-methods)
5. [Adding Tabs](#adding-tabs)
6. [Tab Types](#tab-types)
7. [Element Methods](#element-methods)
8. [Notifier](#notifier)
9. [Dropdown](#dropdown)
10. [TextEffect Utilities](#texteffect-utilities)
11. [Icon Reference](#icon-reference)

---

## Overview

KNCRYPT HUB is a Roblox UI library for building in-game menus. It provides a draggable, resizable, animated window with a vertical icon tab bar, and a set of interactive elements (toggles, sliders, buttons, keybinds, textboxes, dropdowns, paragraphs, sections, and a chat view).

```lua
local UI = loadstring(game:HttpGet("..."))()

local Window = UI.new({
    Title   = "My Script",
    Logo    = "rbxassetid://...",
    Keybind = Enum.KeyCode.RightShift,
    Size    = UDim2.new(0, 580, 0, 365),
})
```

---

## Global Properties

These live on the module table and can be read or overridden before calling `UI.new`.

| Property | Type | Default | Description |
|---|---|---|---|
| `Version` | `string` | `"0.1"` | Library version |
| `Hightlight` | `Color3` | `rgb(0,134,105)` | Accent colour used by toggles, sliders, and the tab highlight |
| `Font` | `Font` | GothamSSm Medium | Default font object |
| `LuaVersion` | `string` | `"Luau"` | Runtime identifier |
| `ProcessId` | `number` | random 100–1000 | Unique ID per load |
| `CoreGui` | `Instance` | `CoreGui` or `PlayerGui` | Parent for all ScreenGuis |

---

## Creating a Window

```lua
local Window = UI.new(args)
```

### args

| Field | Type | Default | Description |
|---|---|---|---|
| `Title` | `string` | `"Kncrypt Hub"` | Title shown in the header bar |
| `Logo` | `string` | (default asset) | Roblox asset ID for the header logo |
| `Keybind` | `Enum.KeyCode` | `LeftControl` | Key that toggles the window visibility |
| `Size` | `UDim2` | `580 × 365` | Initial window size. Min enforced at resize: 460 × 310 |

### Returns

A `KNC_Interface` table with the methods below.

---

## Window Methods

### `Window:AddTab(args)` → `KNC_Tab`

Adds a tab button to the left icon sidebar. Returns the tab object.

```lua
local Tab = Window:AddTab({
    Title = "Combat",
    Icon  = "sword",          -- icon name or asset ID string
    Type  = "Multiple",       -- "Single" | "Multiple" | "Chat"
})
```

| Field | Type | Default | Description |
|---|---|---|---|
| `Title` | `string` | `"Tab"` | Label shown when the sidebar is expanded |
| `Icon` | `string` | `"home"` | Icon name from the built-in icon table, or a raw asset ID string |
| `Type` | `string` | `"Multiple"` | Layout mode — see [Tab Types](#tab-types) |

---

### `Window:Authorize(args)`

Shows a full-screen key system over the window, then yields until authentication succeeds or is cancelled.

```lua
Window:Authorize({
    Title        = "Key System",
    Placeholder  = "Enter your key...",
    ClickLabel   = "Get Key",
    LoginLabel   = "Login",

    OnStart  = function(connector)
        -- fires immediately; use connector:Loading(), connector:Default(), connector:Cancel()
    end,

    OnClick  = function()
        -- fires when the user presses the ClickLabel button (e.g. open a link)
    end,

    OnLogin  = function(text)
        -- fires when the user presses Login; receives the textbox value
        -- return true to accept (dismiss the overlay), anything else to reject
        return validateKey(text)
    end,
})
-- execution continues here after auth is complete
```

#### `connector` object (passed to `OnStart`)

| Method | Description |
|---|---|
| `connector:Loading()` | Plays the loading animation |
| `connector:Default()` | Resets to the default auth state |
| `connector:Cancel()` | Dismisses the overlay immediately |

---

### `Window:SetHide()`

Slides the sidebar and header off-screen and marks the window as not ready for interaction. Used internally during Authorize.

---

### `Window:SetDefault()`

Slides the sidebar and header back in and marks the window as ready.

---

## Adding Tabs

The first `AddTab` call is automatically selected on open. Subsequent tabs are selected by clicking their icon.

Programmatically select a tab:

```lua
local Tab = Window:AddTab({ Title = "Settings", Icon = "settings" })
Tab:Select()
```

---

## Tab Types

### `"Multiple"` (default)

Two-column layout. Elements added to `Tab.Left` appear in the left column; elements added to `Tab.Right` appear in the right column.

```lua
local Tab = Window:AddTab({ Title = "Combat", Type = "Multiple" })

Tab.Left:AddToggle({ Title = "Aimbot", Default = false, Callback = function(v) end })
Tab.Right:AddSlider({ Title = "FOV", Min = 1, Max = 360, Default = 90, Callback = function(v) end })
```

---

### `"Single"`

Full-width single scrolling column. Elements are added directly to the tab:

```lua
local Tab = Window:AddTab({ Title = "Info", Type = "Single" })
Tab:AddParagraph({ Title = "Welcome", Description = "Hello world" })
```

---

### `"Chat"`

A chat-style tab with a message feed, a text input, and optional header buttons.

```lua
local Tab = Window:AddTab({
    Title       = "Chat",
    Type        = "Chat",
    ChatName    = "@General",
    Placeholder = "Type a message...",
})

-- Add header button
Tab:AddButton({
    Title    = "Clear",
    Callback = function() Tab:ClearMessage() end,
})

-- Display a message in the feed
Tab:SendMessage({
    Profile  = "rbxassetid://...",  -- profile icon asset ID
    Username = "4lpaca",
    Content  = "Hello world!",
})

-- Listen for user input
Tab:OnMessage(function(text)
    Tab:SendMessage({ Username = "Me", Content = text })
end)

-- Clear all messages
Tab:ClearMessage()
```

---

## Element Methods

All element methods below are available on the object returned by `Tab`, `Tab.Left`, or `Tab.Right` (all are `CreateInstances` objects).

---

### `:AddSection(args)` → nested `CreateInstances`

A collapsible grouping container. Returns another element container so you can nest elements inside it.

```lua
local Section = Tab.Left:AddSection({ Title = "Movement" })
Section:AddToggle({ Title = "Speed", Default = false, Callback = function(v) end })
```

| Field | Type | Default |
|---|---|---|
| `Title` | `string` | `"Section"` |

---

### `:AddParagraph(args)` → `args`

A read-only text block with a title and optional description.

```lua
Tab:AddParagraph({
    Title       = "Notice",
    Description = "This script is for educational purposes only.",
})
```

| Field | Type | Default |
|---|---|---|
| `Title` | `string` | `"Content"` |
| `Description` | `string?` | `nil` |

#### Instance methods

```lua
local p = Tab:AddParagraph({ Title = "Hello" })
p:Title("New Title")
p:Visible(false)
```

---

### `:AddButton(args)` → `args`

A clickable row button.

```lua
Tab.Left:AddButton({
    Title    = "Teleport",
    Callback = function(frame) print("clicked") end,
})
```

| Field | Type | Default |
|---|---|---|
| `Title` | `string` | `"Button"` |
| `Callback` | `function(frame)` | no-op |

#### Instance methods

```lua
local b = Tab:AddButton({ Title = "Go" })
b:Title("Stop")
b:Visible(false)
```

---

### `:AddToggle(args)` → `args`

An on/off switch row.

```lua
Tab.Left:AddToggle({
    Title    = "Fly",
    Default  = false,
    Callback = function(value)
        print("Fly:", value)
    end,
})
```

| Field | Type | Default |
|---|---|---|
| `Title` | `string` | `"Button"` |
| `Default` | `boolean` | `false` |
| `Callback` | `function(bool)` | no-op |

#### Instance methods

```lua
local t = Tab.Left:AddToggle({ Title = "Fly", Default = false, Callback = cb })
t:Title("Anti-Fly")
t:Visible(false)
t:Value(true)   -- sets state and fires Callback
```

---

### `:AddSlider(args)` → `args`

A draggable value slider.

```lua
Tab.Right:AddSlider({
    Title    = "Walk Speed",
    Min      = 16,
    Max      = 100,
    Default  = 16,
    Rounding = 0,       -- decimal places
    Type     = "",      -- suffix shown in the value label, e.g. "%" or " studs"
    Callback = function(value)
        game.Players.LocalPlayer.Character.Humanoid.WalkSpeed = value
    end,
})
```

| Field | Type | Default |
|---|---|---|
| `Title` | `string` | `"Slider"` |
| `Min` | `number` | `0` |
| `Max` | `number` | `100` |
| `Default` | `number` | `50` |
| `Rounding` | `number` | `0` (integer) |
| `Type` | `string` | `"%"` |
| `Callback` | `function(number)` | no-op |

#### Instance methods

```lua
local s = Tab.Right:AddSlider({ Title = "Speed", Min = 0, Max = 100, Default = 50, Callback = cb })
s:Title("Jump Power")
s:Visible(false)
s:Value(75)   -- sets value and fires Callback
```

---

### `:AddKeybind(args)` → `args`

A clickable keybind picker. Click to start listening, press any key to bind.

```lua
Tab.Left:AddKeybind({
    Title    = "Toggle Fly",
    Default  = Enum.KeyCode.F,
    Callback = function(keyCode)
        print("New keybind:", keyCode.Name)
    end,
})
```

| Field | Type | Default |
|---|---|---|
| `Title` | `string` | `"Keybind"` |
| `Default` | `Enum.KeyCode` or `string` | `"NONE"` |
| `Callback` | `function(Enum.KeyCode)` | no-op |

#### Instance methods

```lua
local k = Tab.Left:AddKeybind({ Title = "Fly Key", Default = Enum.KeyCode.F, Callback = cb })
k:Title("Sprint Key")
k:Visible(false)
k:Value(Enum.KeyCode.G)  -- sets keybind and fires Callback
```

---

### `:AddTextbox(args)` → `args`

A text input field. Fires on every keystroke, or only on focus-lost if `Finished = true`.

```lua
Tab.Right:AddTextbox({
    Title       = "Player Name",
    Placeholder = "Enter name...",
    Default     = "",
    Finished    = true,   -- if true, callback fires only when focus is lost
    Callback    = function(text)
        print("Input:", text)
    end,
})
```

| Field | Type | Default |
|---|---|---|
| `Title` | `string` | `"Textbox"` |
| `Placeholder` | `string` | `""` |
| `Default` | `string` | `""` |
| `Finished` | `boolean` | `false` |
| `Callback` | `function(string)` | no-op |

#### Instance methods

```lua
local tb = Tab.Right:AddTextbox({ Title = "Name", Placeholder = "...", Default = "", Callback = cb })
tb:Title("Username")
tb:Visible(false)
tb:Value("Roblox")   -- sets text and fires Callback
```

---

## Notifier

Creates a notification stack anchored to the bottom-right of the screen. Each call to `UI:Notifier()` returns a notifier object; typically you create one and reuse it.

```lua
local Notifier = UI:Notifier()

local n = Notifier.new({
    Title       = "Success",
    Description = "Operation complete.",  -- optional
    Duration    = 5,                       -- seconds before auto-dismiss
})

-- Manually dismiss before duration expires:
n:Destroy()
```

### `Notifier.new(args)` → `{ Destroy }`

| Field | Type | Default |
|---|---|---|
| `Title` | `string` | `"Notification"` |
| `Description` | `string?` | `nil` |
| `Duration` | `number` | `5` |

---

## Dropdown

A shared floating dropdown is automatically created and accessible via `UI:SetDropdown`. It is used internally; you can also drive it directly.

```lua
UI:SetDropdown(
    { "Option A", "Option B", "Option C" },  -- items
    "Option A",                               -- default (string for single, table for multi)
    false,                                    -- IsMulti
    function(selected)
        print("Selected:", selected)
    end
)
```

For multi-select, pass `IsMulti = true` and a table as default:

```lua
UI:SetDropdown(
    { "Red", "Green", "Blue" },
    { "Red", "Blue" },   -- pre-selected values
    true,
    function(selectionTable)
        -- selectionTable is { ["Red"] = true, ["Blue"] = true, ... }
    end
)
```

Show / hide the dropdown manually:

```lua
UI.Dropdown:Show()
UI.Dropdown:Hide()
```

---

## TextEffect Utilities

Helper functions that return rich-text strings for use in any RichText-enabled label.

```lua
local UI = require(...)

-- Colour a span of text
local coloured = UI.TextEffect:AddColor("Hello", Color3.fromRGB(255, 100, 0))
-- → '<font color="rgb(255,100,0)">Hello</font>'

-- Make text semi-transparent
local faded = UI.TextEffect:Transparency("World", 0.5)
-- → '<font transparency="0.5">World</font>'
```

### `TextEffect:AddColor(text, color)` → `string`

| Param | Type |
|---|---|
| `text` | `string` |
| `color` | `Color3` |

### `TextEffect:Transparency(text, value)` → `string`

| Param | Type | Range |
|---|---|---|
| `text` | `string` | — |
| `value` | `number` | 0 (opaque) – 1 (invisible) |

---

## Icon Reference

Pass any of the following names as the `Icon` field in `AddTab`. The library also accepts a raw `rbxassetid://...` string directly.

Icons are drawn from two sets: the original compact set and the full `lucide-*` set.

**Compact set (sample):**

`aperture`, `bug`, `settings`, `home`, `image`, `star`, `sun`, `moon`, `globe`, `gamepad`, `gamepad-2`, `crown`, `coins`, `battery`, `battery-full`, `battery-medium`, `camera`, `camera-off`, `terminal`, `code`, `code-2`, `server`, `server-crash`, `shield-check`, `shield-alert`, `shield-close`, `wifi` (via lucide set), `bluetooth`, `bluetooth-off`, `bluetooth-searching`, `book`, `book-open`, `bookmark`, `check`, `check-circle-2`, `check-square`, `trash`, `delete`, `edit-2`, `edit-3`, `pencil`, `pen-tool`, `save`, `copy`, `scissors`, `link`, `upload`, `download`, `download-cloud`, `file`, `file-text`, `file-check`, `file-check-2`, `file-plus`, `file-minus`, `file-x-2`, `file-search`, `folder-plus`, `folder-minus`, `tag`, `hash`, `info`, `help-circle`, `alert-triangle`, `eye`, `layers`, `grid` (lucide), `layout` (lucide), `lock` (lucide), `unlock`, `key` (lucide), `mail` (lucide), `inbox`, `reply`, `reply-all`, `share`, `share-2`, `external-link`, `link`, `user` (lucide), `users`, `user-check`, `user-plus`, `graduation-cap`, `briefcase` (lucide), `map` (lucide), `navigation`, `compass` (lucide), `target`, `crosshair` (lucide), `search` (lucide), `filter` (lucide), `sidebar`, `columns`, `align-justify`, `wifi` (lucide), `signal-low`, `network`, `database`, `cpu`, `monitor`, `smartphone`, `tablet`, `phone`, `phone-off`, `phone-incoming`, `phone-outgoing`, `phone-forwarded`, `headphones`, `mic` (lucide), `mic-off` (lucide), `volume` (lucide), `video` (lucide), `video-off`, `tv`, `tv-2`, `radio`, `podcast`, `music` (lucide), `image`, `images` (lucide), `film` (lucide), `camera`, `gift`, `gift-card`, `ticket`, `star`, `gem`, `crown`, `coins`, `currency`, `dollar-sign`, `pound-sterling`, `megaphone`, `bell` (lucide), `bell-off`, `bell-plus`, `timer`, `timer-off`, `timer-reset`, `clock-1` … `clock-12`, `sunrise`, `sunset`, `sun`, `moon`, `cloud-fog`, `cloud-off`, `cloud-sun`, `cloud-moon`, `cloud-drizzle`, `cloud-hail`, `wind`, `thermometer`, `thermometer-sun`, `thermometer-snowflake`, `umbrella`, `tornado`, `mountain-snow`, `tent`, `bike`, `truck`, `navigation`, `airplay`, `cast`, `screen-share`, `wifi`, `bluetooth`, `battery`, `flashlight`, `flashlight-off`, `lightbulb-off`, `infinity`, `sigma`, `hash`, `percent` (lucide), `regex`, `command`, `option`, `indent`, `outdent`, `wrap-text`, `subscript`, `superscript`, `italic`, `underline`, `bold` (lucide), `quote`, `type`, `languages`, `book`, `library`, `feather`, `palette`, `brush`, `highlighter`, `wand`, `axe`, `shovel`, `scissors`, `pin`, `paperclip`, `archive`, `package`, `package-search`, `package-plus`, `codesandbox`, `figma`, `framer`, `dribbble`, `github`, `gitlab`, `trello`, `bot`, `terminal`, `function-square`, `server`, `database`, `cpu`, `network`, `fingerprint` (lucide), `shield`, `lock`, `key`, `bug`, `inspect`, `gauge`, `expand`, `maximize`, `minimize-2`, `sidebar`, `columns`, `grid`, `layout`, `table`, `sort-asc`, `bar-chart`, `trending-up`, `trending-down`, `activity` (lucide), `pie-chart` (lucide), `zap` (lucide), `award` (lucide), `flag` (lucide), `map-pin` (lucide), `locate`, `locate-fixed`, `building`, `home`, `rocking-chair`, `coffee`, `pocket`, `watch`, `glasses`, `shirt`, `shovel`, `clover`, `egg`, `beaker`, `microscope` (lucide), `dna` (lucide), `atom` (lucide), `leaf` (lucide), `palmtree` (lucide), `mountain`, `tent`, `bike`, `car` (lucide), `truck`, `plane` (lucide), `rocket` (lucide), `sailboat` (lucide), `anchor` (lucide), `webcam`, `qr-code`, `barcode` (lucide), `scan` (lucide), `printer` (lucide), `hard-hat`, `person-standing`, `hand`, `move-diagonal`, `move-vertical`, `corner-up-right`, `corner-right-down`, `corner-right-up`, `corner-down-left`, `arrow-left`, `arrow-right` (lucide), `arrow-up` (lucide), `arrow-down`, `arrow-left-right`, `arrow-up-right`, `arrow-down-right`, `arrow-big-right`, `arrow-big-left`, `arrow-big-down`, `chevron-right`, `chevrons-right`, `chevrons-down-up`, `chevrons-up`, `chevron-first`, `chevron-last`, `rotate-ccw`, `refresh-cw`, `rewind`, `skip-back`, `skip-forward`, `play` (lucide), `play-circle`, `pause` (lucide), `reply`, `repeat`, `shuffle`, `switch-camera`, `more-vertical`, `more-horizontal` (lucide), `menu` (lucide), `list-x`, `list-checks`, `list-ordered`, `clipboard-check`, `clipboard-list`, `clipboard-copy`, `clipboard-x`, `check-circle-2`, `x-circle`, `x-square`, `plus-square`, `minus-circle`, `divide-circle`, `equal`, `equal-not`, `copyleft`, `copyright` (lucide), `asterisk`, `cross`, `octagon`, `triangle`, `circle` (lucide), `square` (lucide), `hexagon` (lucide), `pentagon` (lucide), `haze`, `lock` (lucide), `on-charge`, `electricity-off`, `scan-line`, `align-*` (many alignment variants), `stretch-horizontal`, `stretch-vertical`, `grip-horizontal`, `grip-vertical`, `move-diagonal`, `loader-2`, `album`, `box` (lucide), `boxes` (lucide), `package-open` (lucide)

**Lucide set prefix:** Any of the above can also be referenced with the `lucide-` prefix (e.g. `"lucide-home"`, `"lucide-settings"`).

---

## Full Example

```lua
local UI = loadstring(game:HttpGet("..."))()

local Window = UI.new({
    Title   = "My Hub",
    Logo    = "rbxassetid://125085241530623",
    Keybind = Enum.KeyCode.RightShift,
    Size    = UDim2.new(0, 580, 0, 365),
})

-- Optional key system
Window:Authorize({
    Title      = "Key System",
    Placeholder = "Enter key...",
    OnLogin    = function(text)
        return text == "mykey123"
    end,
    OnClick    = function()
        setclipboard("https://discord.gg/example")
    end,
})

-- Notifier
local Notifier = UI:Notifier()
Notifier.new({ Title = "Loaded", Description = "Hub ready.", Duration = 3 })

-- Tabs
local CombatTab = Window:AddTab({ Title = "Combat", Icon = "sword", Type = "Multiple" })
local SettingsTab = Window:AddTab({ Title = "Settings", Icon = "settings", Type = "Single" })

-- Elements
CombatTab.Left:AddToggle({
    Title    = "Aimbot",
    Default  = false,
    Callback = function(v) print("Aimbot:", v) end,
})

CombatTab.Right:AddSlider({
    Title    = "FOV",
    Min      = 1,
    Max      = 360,
    Default  = 90,
    Callback = function(v) print("FOV:", v) end,
})

CombatTab.Left:AddKeybind({
    Title    = "Toggle",
    Default  = Enum.KeyCode.T,
    Callback = function(k) print("Bound to:", k.Name) end,
})

CombatTab.Right:AddTextbox({
    Title       = "Target",
    Placeholder = "Player name...",
    Default     = "",
    Finished    = true,
    Callback    = function(t) print("Target:", t) end,
})

local section = SettingsTab:AddSection({ Title = "Display" })
section:AddButton({
    Title    = "Reset Position",
    Callback = function() end,
})

section:AddParagraph({
    Title       = "About",
    Description = "v1.0 — by 4lpaca",
})
```
