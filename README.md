# ZeninLib

A simple UI library for Roblox. You build a menu in 4 steps:

1. **Load** the library
2. Make a **window**
3. Add **tabs** and **sections** to it
4. Put **elements** (toggles, sliders, buttons...) into sections

Full working example: [`Example.lua`](Example.lua).

## 1. Load the library

```lua
local Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/whereesd0-star/Zenin/main/ZeninLib.lua"))()
```

## 2. Make a window

```lua
local Window = Library:CreateWindow({
    Title = "My Hub",                        -- name in the top bar
    Logo = "M",                              -- a letter, an icon ("zap") or an image ("rbxassetid://123")
    ToggleKey = Enum.KeyCode.RightShift,     -- key that opens / closes the menu
    Accent = Color3.fromRGB(255, 89, 89),    -- main color
    Configs = {Folder = "MyHub"},            -- save settings to files (or just: Configs = true)
    Watermark = {Title = "myhub", Site = "myhub.cc", Detail = "roblox"},
})
```

Only `Title` really matters — everything else has a good default.

| Option | What it does |
|---|---|
| `Title` | name in the header |
| `Logo` | letter, icon name or image id |
| `ToggleKey` | key that opens the menu |
| `Theme` | `"Dark"` or `"Light"` |
| `Accent` | main color |
| `Configs` | `true` or `{Folder = "MyHub"}` — turns on the configs page |
| `ScrollbarSize` | scrollbar thickness, 3–24 |
| `Watermark` | watermark settings, `false` to hide it |

Tip: clicking the watermark also opens / closes the menu.
The gear in the header always has settings for theme, color, size, scrollbar and watermark.

## 3. Tabs and sections

Think of it like a book: **tab** = chapter (bottom bar), **sub-tab** = page, **section** = box on the page.

```lua
local Combat  = Window:AddTab({Name = "Combat", Icon = "crosshair"})
local Aimbot  = Combat:AddSubTab({Name = "Aimbot", Icon = "crosshair"})   -- optional
local Section = Aimbot:AddSection({Name = "Target", Side = "Left"})       -- Side = "Left" or "Right"
```

No sub-tabs? Add sections straight to the tab: `Combat:AddSection({...})`.

Your own box in the settings page: `Window.Settings:AddSection({Name = "Info"})`.

## 4. Elements

Every element can have:

- `Name` — the text you see
- `Default` — the starting value
- `Flag` — a unique id, so the value is saved in configs and you can read it with `Library.Flags["MyFlag"]`
- `Callback` — a function that runs every time the value changes

```lua
-- on / off switch
Section:AddToggle({Name = "Aimbot", Default = false, Flag = "Aimbot", Callback = function(on)
    print("Aimbot is", on)
end})

-- number from Min to Max
Section:AddSlider({Name = "FOV", Min = 0, Max = 100, Default = 50, Step = 1, Suffix = "%", Flag = "FOV"})

-- pick one
Section:AddDropdown({Name = "Bone", Options = {"Head", "Body"}, Default = "Head", Flag = "Bone"})

-- pick many
Section:AddMultiDropdown({Name = "Hitboxes", Options = {"Head", "Chest", "Legs"}, Default = {"Head"}, Flag = "Hitboxes"})

-- color (or a gradient: give up to 4 colors)
Section:AddColorPicker({Name = "Color", Default = Color3.new(1, 1, 1), Flag = "Color"})
Section:AddColorPicker({Name = "Gradient", Default = {Color3.new(1, 0, 0), Color3.new(0, 0, 1)}, Flag = "Gradient"})

-- simple button
Section:AddButton({Name = "Reset", Callback = function() end})

-- button with an icon (great for links)
Section:AddCardButton({Name = "Telegram", Icon = "send", ButtonText = "Open", Callback = function()
    setclipboard("https://t.me/...")
    return "Copied"   -- text you return shows on the button for a moment
end})
```

**Show an element only when a toggle is on:** add `DependsOn = myToggle`.

```lua
local esp = Section:AddToggle({Name = "ESP", Flag = "ESP"})
Section:AddSlider({Name = "ESP Distance", Min = 0, Max = 1000, Flag = "EspDistance", DependsOn = esp})
```

**Things you can do with any element:**

| Method | What it does |
|---|---|
| `Set(value)` / `Get()` | change / read the value |
| `SetEnabled(bool)` | lock / unlock it |
| `SetVisible(bool)` | show / hide it |
| `OnChanged(fn)` | run a function when the value changes |
| `Destroy()` | remove it |
| `SetOptions(list)` | dropdowns only: new list of options |

### Little gear next to an element

```lua
local font = Section:AddDropdown({Name = "Font", Options = {"Small", "Big"}})
local more = font:AddSettings({Title = "Font Settings"})
more:AddSlider({Name = "Size", Min = 6, Max = 20, Default = 10})
```

Works on Toggle, Slider, Dropdown, MultiDropdown and ColorPicker.

## Window methods

| Method | What it does |
|---|---|
| `Show()`, `Hide()`, `Toggle()` | show / hide the menu |
| `SetTheme(name)`, `SetAccent(color)` | theme and color |
| `SetScale(n)`, `Center()` | size and position |
| `SetLogo(value)`, `SetTitle(text)` | logo and name |
| `Notify({Title, Content, Duration})` | pop-up message |
| `Destroy()` | close and remove the menu |

## Configs (saving settings)

Turn them on with `Configs = true` or `Configs = {Folder = "MyHub"}`.
Files are plain JSON (`MyHub/<name>.cfg`), so you can open and edit them in any text editor.

The configs page (the file button next to the gear) has:

- **Name** box + **Create New** — saves your current settings as a new config
- **Refresh List** — re-reads the folder
- for every config: **Load** (arrow), **Save** (disk) and **Delete** (trash)

Everything with a `Flag` is saved, plus theme color.

From code:

| Method | What it does |
|---|---|
| `Window.Configs:Save(name)` | save |
| `Window.Configs:Load(name)` | load |
| `Window.Configs:Delete(name)` | delete |
| `Window.Configs:List()` | list of config names |
| `Window.Configs:Reset()` | put every element back to its default |
| `Window.Configs:SetAutoLoad(name)` | load this config automatically next time (`nil` to turn off) |
| `Window.Configs:GetActive()` | name of the loaded config |
| `Rename(old, new)`, `Duplicate(name)`, `Exists(name)` | manage files |
| `Ignore(flag, ...)` | don't save these flags |
| `Register(key, get, set)` | save your own data too |
| `OnSave(fn)`, `OnLoad(fn)` | run code on save / load |

```lua
-- save your own table inside configs
Window.Configs:Register("Waypoints", function() return myWaypoints end, function(v) myWaypoints = v end)
```

## Watermark

```lua
Window.Watermark:SetTitle("myhub")
Window.Watermark:SetSite("myhub.cc")
Window.Watermark:SetDetail("roblox")
Window.Watermark:SetModes({"Time", "FPS", "Ping"})   -- also "Product Type"
Window.Watermark:SetVisible(false)
```

## Your own theme

```lua
Library:AddTheme("Ocean", {WindowBg = Color3.fromRGB(10, 16, 22)})
Window:SetTheme("Ocean")
```
