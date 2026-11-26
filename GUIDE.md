# utk_handhold

Players hold each other while they stand, walk and run: hand in hand in rows and rings of any size, and both hands
face to face.

Nothing is attached and nobody is frozen. Every hold is drawn with arm IK on each client that sees it, so both
players keep walking on their own, the hands follow them, and a hold that is pulled too far lets go and comes back by
itself.

Version 1.0.0. Standalone: no framework is required.

## Contents

1. [Requirements](#1-requirements)
2. [Installation](#2-installation)
3. [How it plays](#3-how-it-plays)
4. [Configuration](#4-configuration)
5. [Interaction (target) systems](#5-interaction-target-systems)
6. [Key guide](#6-key-guide)
7. [Frameworks](#7-frameworks)
8. [Using it from your own resources](#8-using-it-from-your-own-resources)
9. [Troubleshooting](#9-troubleshooting)
10. [Licence and support](#10-licence-and-support)

---

## 1. Requirements

- A FiveM server with **OneSync** on (`set onesync on`).
- Nothing else. There is no database, no item and no dependency on another resource.

Target systems, text UIs and frameworks are optional. Without any of them the resource uses its own menu, hand
marker, countdown rings and key guide, and every hold is available.

## 2. Installation

1. Put the folder `utk_handhold` into your `resources`. **Do not rename it**: the bundled animation and the exports
   use that name.
2. Add it to `server.cfg`, after your framework and target system if you have them:

   ```cfg
   ensure ox_target        # only if you use one
   ensure utk_handhold
   ```

3. Restart the server. To add it to a running server instead, type these in the server console (the resource streams
   an animation file, so `refresh` has to come first):

   ```
   refresh
   ensure utk_handhold
   ```

4. Join with two players, stand side by side and press **G** on both. Their hands should join. Let go, and **hold G**
   for the menu with the other holds.

### Which files you edit

| File | What for | Escrowed |
|---|---|---|
| `config/config.lua` | Settings: language, systems, keys, which holds exist, times | no |
| `config/poses.lua` | How the hands look on your server | no |
| `locales/*.lua` | Every word a player reads | no |
| `bridge/client.lua` | Target systems, key guide, "is this player busy" on the client | no |
| `bridge/server.lua` | Who may not hold anyone | no |
| `client`, `server`, `shared` | The script itself | yes |

**Updating:** keep your own copies of `config/`, `locales/` and `bridge/`, replace the folder, and carry your changes
over.

## 3. How it plays

Every player can rebind the keys in Settings > Key Bindings > FiveM.

| Key | Default | What it does |
|---|---|---|
| Hand | G | **Tap:** take a hand held out next to you; or take your own held-out hand back; or, while holding, let go of everyone; or hold a hand out. **Hold for half a second:** next to a player, the built-in menu on them (both hands, take hand, let go); or, while holding someone with a hand still free, hold that hand out for the next player. |
| Firm grip | H | While two hold one hand each: the first leads the second by the hand. The hold bears more, the leading arm may trail far behind, and the second walks along with the first for as long as they press nothing themselves. If it comes apart on a sharp turn, the second is drawn back and it joins again. |
| Front | J | While two hold one hand each: both hands face to face, and back to side by side. |
| Menu | O | Built-in target only: the same menu, also while holding someone. Wheel or arrows move, E, Enter or a click chooses, Esc, Backspace or a right click closes. |

The menu on another player offers **Take hand**, **Hold both hands** and **Let go**, each only while it can be done.

Good to know:

- **Two hands per player.** Each hand can hold a different player, so players form rows, and rings of three or more.
- **Holds break and recover.** Too far, turned too far, a wall between, a fall, a jump, aiming: the hands let go. For
  a few seconds the hold can come back, and it joins again by itself once the two stand right.
- **Some things end a hold at once:** letting go, a vehicle, death, being blocked (cuffed and so on), changing routing
  bucket, or being far away.
- **No consent prompt.** A held-out hand is simply taken. The other player ends the hold with the hand key, with Let
  go in the menu, or by walking off.
- **Shoes of any height.** Heels lift a ped. In a hand in hand or front hold the two hands are brought to one height.
- **One pace.** While a player holds someone, their ped walks in the game's default walk style, so that two players
  with different walk styles keep together. Their own style comes back when they let go (`Config.DefaultWalk`).
- **Players' own settings.** `/handhold` opens a small panel where each player sets the countdown ring's size,
  opacity and colour. It is kept on their machine. Switch it off with `Config.PlayerSettings`.

## 4. Configuration

### `config/config.lua`

| Setting | Default | Meaning |
|---|---|---|
| `Config.Locale` | `'en'` | A file name in `locales/` without `.lua`. English and Turkish are included. |
| `Config.Accent` | `'#ffc526'` | The colour of the built-in menu, marker, rings and key guide, as `'#rrggbb'`. |
| `Config.PlayerSettings` | on, `handhold` | The `/handhold` panel for each player's own ring. `enabled = false`: no command, everyone sees the server's look. |
| `Config.Target` | `'auto'` | Where the menu on another player lives. See [section 5](#5-interaction-target-systems). |
| `Config.HandMarker` | `true` | A marker on a held-out hand, with the key to take it. `false`: nothing is drawn and nothing runs for it. |
| `Config.KeyGuide` | `'auto'` | How the firm-grip and front keys are shown. See [section 6](#6-key-guide). |
| `Config.Keys` | G, H, J, O | Default keys. A default only reaches players who never had the binding. `''` leaves a key unbound. |
| `Config.DefaultWalk` | `true` | The game's default walk style while holding, so that two players keep one pace; their own style comes back after. |
| `Config.Holds` | all `true` | Switch `hand`, `front` or `firm` off for everyone. |
| `Config.OfferTime` | `45` | Seconds a hand stays held out. |
| `Config.RecoverTime` | `8` | Seconds a broken hold can come back. |
| `Config.JoinTime` | `14` | Seconds two players have to step together when a hold begins. |
| `Config.BlockingStates` | see file | Player state bag keys that mean "cannot hold anyone". See [section 7](#7-frameworks). |

Example: no firm grip, hand key on B, Turkish:

```lua
Config.Locale = 'tr'

Config.Keys = { hand = 'B', firm = 'H', front = 'J', menu = 'O' }

Config.Holds = { hand = true, front = true, firm = false }
```

### `locales/`

To add a language, copy `locales/en.lua` to, for example, `locales/de.lua`, change `Locales['en']` to
`Locales['de']`, translate the right-hand sides, and set `Config.Locale = 'de'`. Missing lines fall back to English.

### `config/poses.lua`

Your own poses, replacing the built-in ones for every player. The built-in ones fit the game's default male and female
peds; the file lists the values worth changing. For example:

```lua
Config.Poses = {
    frontGap = 0.65, -- both hands in front: the two stand 65 cm apart
    level = 0,       -- do not bring the hands to one height (shoes that lift a ped)
}
```

## 5. Interaction (target) systems

`Config.Target` decides where the menu on another player lives:

| Value | What is used |
|---|---|
| `'auto'` | The first of ox_target, qb-target, qtarget that is started. If none is, the built-in menu. |
| `'builtin'` | The resource's own menu, marker, rings and key guide, whatever else the server runs. |
| `'ox_target'`, `'qb-target'`, `'qtarget'` | That system. |
| `'custom'` | Your own code in `bridge/client.lua`. See below. |
| `'none'` | No menu. Hand in hand and the front hold still work with keys. |

The ox_target, qb-target and qtarget versions are ready-made but have not been run on every release of those
resources. If one changed its exports, adjust its entry in `bridge/client.lua`.

### Writing your own

Set `Config.Target = 'custom'` and replace two places in `bridge/client.lua`: the function `adders['custom']`
(register the options) and the entry `['custom']` in the `removers` table (take them away when this resource stops).

`options` is a list. Each entry:

```lua
{
    id = 'utk_handhold:front',            -- unique name
    label = 'Hold both hands',            -- already in the server's language
    icon = 'fa-solid fa-hands',           -- a Font Awesome class
    canInteract = function(ped) end,      -- true while it can be done on that player's ped; cheap, no side effects
    onSelect = function(ped) end,         -- the player chose it on that ped
}
```

`ped` is always the **other** player's ped. Two things to remember:

- Ask `canInteract` before showing an option. Options that cannot be done have to stay hidden.
- Your system may start after this resource, or be restarted. `onClientResourceStart` covers both.

The examples use `started` and `RANGE` (2.5 m), which are defined at the top of the same file. They were written
against each resource's public source and have not been run on a live server: check them against your version.

#### Example: sleepless_interact

```lua
adders['custom'] = function(options)
    local function register()
        local list = {}
        for i, option in ipairs(options) do
            list[i] = {
                name = option.id,
                label = option.label,
                icon = option.icon,
                distance = RANGE,
                canInteract = function(entity) return option.canInteract(entity) == true end,
                onSelect = function(data) option.onSelect(data.entity) end,
            }
        end
        exports.sleepless_interact:addGlobalPlayer(list)
    end

    if started('sleepless_interact') then register() end
    AddEventHandler('onClientResourceStart', function(resource)
        if resource == 'sleepless_interact' then register() end
    end)
end
```

In the `removers` table:

```lua
['custom'] = function(options)
    local names = {}
    for i, option in ipairs(options) do names[i] = option.id end
    exports.sleepless_interact:removeGlobalPlayer(names)
end,
```

#### Example: interact (darktrovx)

```lua
adders['custom'] = function(options)
    local function register()
        local list = {}
        for i, option in ipairs(options) do
            list[i] = {
                label = option.label,
                canInteract = function(entity) return option.canInteract(entity) == true end,
                action = function(entity) option.onSelect(entity) end,
            }
        end
        exports.interact:addGlobalPlayerInteraction({
            id = 'utk_handhold',
            distance = 4.0,
            interactDst = RANGE,
            offset = vec3(0.0, 0.0, 0.0),
            options = list,
        })
    end

    if started('interact') then register() end
    AddEventHandler('onClientResourceStart', function(resource)
        if resource == 'interact' then register() end
    end)
end
```

In the `removers` table:

```lua
['custom'] = function()
    exports.interact:RemoveGlobalPlayerInteraction('utk_handhold')
end,
```

#### Example: a key that opens an ox_lib context menu

For servers with ox_lib and no target system. The same shape works for any menu resource. Nothing has to be taken
away, so the `removers` entry stays empty.

```lua
local function nearestPlayerPed()
    local me, here = PlayerPedId(), GetEntityCoords(PlayerPedId())
    local best, bestDistance = nil, RANGE
    for _, player in ipairs(GetActivePlayers()) do
        local ped = GetPlayerPed(player)
        if ped ~= 0 and ped ~= me and DoesEntityExist(ped) then
            local distance = #(GetEntityCoords(ped) - here)
            if distance < bestDistance then best, bestDistance = ped, distance end
        end
    end
    return best
end

adders['custom'] = function(options)
    RegisterCommand('holdmenu', function()
        if IsNuiFocused() or IsPauseMenuActive() or not started('ox_lib') then return end
        local ped = nearestPlayerPed()
        if not ped then return end

        local rows = {}
        for _, option in ipairs(options) do
            if option.canInteract(ped) then
                rows[#rows + 1] = {
                    title = option.label,
                    icon = 'hand',
                    onSelect = function() option.onSelect(ped) end,
                }
            end
        end
        if #rows == 0 then return end

        exports.ox_lib:registerContext({ id = 'utk_handhold', title = 'Hold', options = rows })
        exports.ox_lib:showContext('utk_handhold')
    end, false)
    RegisterKeyMapping('holdmenu', 'Hold menu on the nearest player', 'keyboard', 'O')
end
```

#### Example: an interaction system of your own

`my_interact` and its exports stand for yours.

```lua
adders['custom'] = function(options)
    local function register()
        for _, option in ipairs(options) do
            exports.my_interact:addPlayerOption({
                id = option.id,
                label = option.label,
                show = function(ped) return option.canInteract(ped) == true end,
                action = function(ped) option.onSelect(ped) end,
            })
        end
    end

    if started('my_interact') then register() end
    AddEventHandler('onClientResourceStart', function(resource)
        if resource == 'my_interact' then register() end
    end)
end
```

In the `removers` table:

```lua
['custom'] = function(options)
    for _, option in ipairs(options) do exports.my_interact:removePlayerOption(option.id) end
end,
```

If your system hands over a server id instead of a ped, turn it into one:
`GetPlayerPed(GetPlayerFromServerId(serverId))`.

### Walk styles the script cannot put back

With `Config.DefaultWalk` the script puts a player's walk style back after a hold. It knows the game's own styles by
name. A style it does not know (an addon clip set) is handed to `Bridge.restoreWalk(ped, hash)` in
`bridge/client.lua`: put it back there, for example through your emote resource, and return `true`.

### A marker or ring of your own

The resource draws a marker on a held-out hand and a countdown ring round two hands that reach for each other. If
your own system can draw on a point of a ped, `Bridge.markHand` / `Bridge.unmarkHand` and `Bridge.timerAdd` /
`Bridge.timerMove` / `Bridge.timerRemove` in `bridge/client.lua` let it. Return `true` and the built-in one stays
away. Their comments list the parameters.

## 6. Key guide

While two players hold one hand each, the firm-grip and front keys are shown. `Config.KeyGuide`:

| Value | What is used |
|---|---|
| `'auto'` | The built-in guide with the built-in target; else ox_lib's text UI if it is started; else the built-in guide. |
| `'builtin'` | The resource's own: the keys at the right edge of the screen. |
| `'ox_lib'` | ox_lib's text UI. |
| `'native'` | The game's help box. |
| `'custom'` | Your own code in `bridge/client.lua`. |
| `'none'` | Nothing is shown. The keys still work. |

For `'custom'`, replace the `['custom']` entry of the `guides` table. `rows` is a list of
`{ command = '+utk_handhold_firm', label = 'Firm grip' }`; `keyOf(row.command)` gives the key the player has bound.

#### Example: QBCore DrawText

```lua
['custom'] = {
    show = function(rows)
        local lines = {}
        for i, row in ipairs(rows) do lines[i] = ('[%s] %s'):format(keyOf(row.command), row.label) end
        exports['qb-core']:DrawText(table.concat(lines, '<br>'), 'left')
    end,
    hide = function() exports['qb-core']:HideText() end,
},
```

#### Example: esx_textui

```lua
['custom'] = {
    show = function(rows)
        local lines = {}
        for i, row in ipairs(rows) do lines[i] = ('[%s] %s'):format(keyOf(row.command), row.label) end
        exports.esx_textui:TextUI(table.concat(lines, '<br>'), 'info')
    end,
    hide = function() exports.esx_textui:HideUI() end,
},
```

## 7. Frameworks

The script needs no framework. The one thing it cannot know by itself is which players **your** resources consider
unable to hold anyone. It already refuses players who are dead by the game's health or in a vehicle. There are three
ways to add the rest; use the first that fits.

### Way 1: state bags (no code)

If your resource publishes a player state bag, add its key to `Config.BlockingStates`. Any value but `nil`, `false`
and `0` blocks, on the server and the client.

```lua
Config.BlockingStates = {
    'dead', 'isDead', 'cuffed', 'isCuffed', 'isHandcuffed', 'tied', 'inStretcher',  -- shipped
    'inJail', 'isCarried',                                                          -- yours
}
```

### Way 2: `Bridge.isBlocked` in `bridge/server.lua`

Return `true` to refuse. It is asked when a hold starts or joins, and twice a second for each player in a hold, so
keep it cheap: no database queries, no waiting.

### Way 3: `setBlocked` from your own resource

No file of this resource is touched. See [section 8](#8-using-it-from-your-own-resources).

### Framework by framework

| Framework | What to do |
|---|---|
| None | Nothing. |
| QBCore | Nothing: dead, last stand and handcuffed are read from the player's metadata. |
| Qbox | Nothing: the same three. |
| ESX | Handcuffs of esx_policejob are covered by the state bag `isHandcuffed`. For esx_death add the example below. |
| ox_core | Death is covered by the state bag `isDead`. Add the keys of your cuff script. |
| Your own | Way 1, 2 or 3. |

The resource itself uses no framework. It has run with ESX Legacy; the QBCore and Qbox checks and the examples below
were written against each framework's public source.

The examples go into `Bridge.isBlocked(source)`, above its last `return false`.

#### Example: ESX with esx_death

```lua
if started('esx_death') then
    local ok, dead = pcall(function() return exports.esx_death:IsDead(source) end)
    if ok and dead then return true end
end
```

#### Example: ESX, an unemployed player may not (any rule on xPlayer)

```lua
if started('es_extended') then
    local ok, blocked = pcall(function()
        local xPlayer = exports.es_extended:getSharedObject().GetPlayerFromId(source)
        return xPlayer ~= nil and xPlayer.getJob().name == 'unemployed'
    end)
    if ok and blocked then return true end
end
```

#### Example: QBCore or Qbox, more metadata

Both are checked already. To add something of your own, extend the line inside their blocks:

```lua
return meta ~= nil and (meta.isdead == true or meta.inlaststand == true or meta.ishandcuffed == true
    or (tonumber(meta.injail) or 0) > 0)
```

#### Example: a framework of your own

```lua
if started('my_core') then
    local ok, blocked = pcall(function()
        local player = exports.my_core:getPlayer(source)
        return player ~= nil and (player.dead or player.cuffed or player.jailed)
    end)
    if ok and blocked then return true end
end
```

Always check that the other resource is started and wrap the call in `pcall`, as above. An error that leaves
`Bridge.isBlocked` counts as "not blocked".

### The client side

`Bridge.isBusy()` in `bridge/client.lua` is the same question for the player's own keys and menu. It only keeps the
menu honest: the server decides. Fill it in for things only your client scripts know.

```lua
function Bridge.isBusy()
    if started('my_carry') and exports.my_carry:isCarrying() then return true end
    if started('my_minigame') and exports.my_minigame:isPlaying() then return true end
    return false
end
```

## 8. Using it from your own resources

### Server exports

```lua
exports.utk_handhold:getHolds(source)      -- { { id, first, second, kind, held, firm }, ... }
exports.utk_handhold:getHold(id)           -- one hold, or nil
exports.utk_handhold:isHolding(source)     -- true while a hold of theirs is joined
exports.utk_handhold:getPartners(source)   -- server ids of everyone they are in a hold with
exports.utk_handhold:canHold(source)       -- true, or false and why: 'gone', 'dead', 'vehicle', 'busy'

exports.utk_handhold:startHold(first, second, kind)  -- the hold's id, or false and why
exports.utk_handhold:endHold(id, reason)             -- true if there was one
exports.utk_handhold:endHolds(source, reason)        -- ends every hold of a player; returns how many
exports.utk_handhold:setBlocked(source, true)        -- cannot hold anyone, and their holds end...
exports.utk_handhold:setBlocked(source, false)       -- ...until this
```

- `kind` is `'hand'` (default) or `'front'`.
- `first` is the player in front in a hand hold, and the one the other is brought before in a front hold.
- `held` is false while a hold is apart and may still come back.
- `startHold` follows the same rules as the keys. It refuses with `'same'`, `'off'` (that hold is switched off),
  `'gone'`, `'busy'`, `'far'` (more than 3 m apart) or `'hands'` (no hand free).

### Server events

Listen with `AddEventHandler`, never `RegisterNetEvent`, so that no client can fake one.

```lua
AddEventHandler('utk_handhold:holdStarted', function(id, first, second, kind) end)          -- began, not joined yet
AddEventHandler('utk_handhold:holdJoined',  function(id, first, second, kind) end)          -- joined, first time or again
AddEventHandler('utk_handhold:holdParted',  function(id, first, second, kind, reason) end)  -- apart, may come back
AddEventHandler('utk_handhold:holdEnded',   function(id, first, second, kind, reason) end)  -- over
```

Reasons include `letgo`, `lost` (did not come back in time), `distance`, `angle`, `strain`, `wall`, `ragdoll`,
`vehicle`, `dead`, `busy` and `gone` (left the server), or the one you pass to `endHold` and `endHolds`.

### Client exports

```lua
exports.utk_handhold:isHoldingHands()   -- true while a hold of this player is joined
exports.utk_handhold:getPartner()       -- the partner's server id while in exactly one hold, else nil
exports.utk_handhold:getPartners()      -- server ids of everyone this player is in a hold with
```

### State bags

Read-only for other resources, replicated to every client:

- `Player(id).state.hhOffer`: `'L'` or `'R'` while the player holds that hand out.
- `Player(id).state.hhHolds`: a list of `{ s = holdId, p = partnerId, k = kind, h = held, f = firstId, arm = 'L'|'R' }`.

### Examples

Names starting with `my_` stand for your own resources and events.

**Handcuffs** (server): block while cuffed, release after.

```lua
AddEventHandler('my_police:cuffed', function(target, on)
    if GetResourceState('utk_handhold') == 'started' then
        exports.utk_handhold:setBlocked(target, on == true)
    end
end)
```

**Teleport or jail** (server): end the holds first.

```lua
local function sendToJail(target)
    if GetResourceState('utk_handhold') == 'started' then
        exports.utk_handhold:endHolds(target, 'jail')
    end
    -- ... your teleport
end
```

**An action of your own** (server): start a hand hold from your own menu or item. `startHold` checks that both are
near and able; whether this player may do it, and how often, is for you to check.

```lua
RegisterNetEvent('my_script:holdHands', function(target)
    local src = source
    target = tonumber(target)
    if not target or GetResourceState('utk_handhold') ~= 'started' then return end
    local id, why = exports.utk_handhold:startHold(src, target, 'hand')
    if not id then print(('no hold: %s'):format(why)) end
end)
```

**Less stress for two who hold hands** (server). `holdJoined` fires again each time a broken hold comes back, so a
reward needs a cooldown.

```lua
local last = {}
AddEventHandler('utk_handhold:holdJoined', function(id, first, second, kind)
    for _, player in ipairs({ first, second }) do
        if GetGameTimer() - (last[player] or -60000) >= 60000 then
            last[player] = GetGameTimer()
            TriggerEvent('my_stress:relieve', player, 5)
        end
    end
end)
AddEventHandler('playerDropped', function() last[source] = nil end)
```

**No emotes while holding** (client), at the top of your emote function:

```lua
if GetResourceState('utk_handhold') == 'started' and exports.utk_handhold:isHoldingHands() then return end
```

## 9. Troubleshooting

| Problem | What to check |
|---|---|
| Fingers stay open, or an animation error in F8 | Run `refresh` before `ensure` (or restart the server), and do not rename the folder. |
| No options in the target menu | `Config.Target`; with `'auto'` the target has to be started. Without one, hold the hand key (default G) next to a player. |
| My `'custom'` target shows nothing | Is it registered after your system started? See `onClientResourceStart` in section 5. An error while registering prints `[utk_handhold] target custom: ...` in F8. |
| The built-in menu closes at once, or does not open | Another resource uses the same key. Hold the hand key (G) instead, or set another `menu` key in `Config.Keys`. |
| A key does nothing | The player may have it on another key, or an older binding: Settings > Key Bindings > FiveM. |
| A player cannot hold anyone | They are in a vehicle, dead or blocked. `exports.utk_handhold:canHold(source)` says which. `'busy'` means `Config.BlockingStates`, `Bridge.isBlocked` or `setBlocked`. |
| The hands join and let go at once | The two stand too far apart or turned away. Step closer. |
| Others do not see a hold | OneSync has to be on, and nothing may block state bags. |
| The key guide does not show | Try `Config.KeyGuide = 'builtin'`. |
| A player's walk style does not come back after a hold | It is not one of the game's own styles: fill in `Bridge.restoreWalk` (section 5), or set `Config.DefaultWalk = false`. |


## 10. Licence and support

See `LICENSE`. In short: the product is for your own servers under the Cfx.re account it was bought for. You may edit
the files in `config`, `locales` and `bridge`. You may not resell, share or redistribute any part of it, and you may
not change or reverse engineer the protected files.

When asking for support, include: the version, your server artifact, your framework and target system with versions,
the start order, and the server and F8 console errors.
