--[[
    MM2 Halloween Hub — FULL
    Executor: Delta
    UI: Rayfield (sirius.menu)
    Game: Murder Mystery 2 (142823291)
    Theme: Halloween
    Includes: role ESP, auto pickup, CFrame-glitch fling, anti-cheat bypass layers
    Save as: mm2_halloween_hub.lua
]]

if game.PlaceId ~= 142823291 then
    return warn("[MM2 Hub] Wrong place. Join Murder Mystery 2.")
end

local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Players     = game:GetService("Players")
local RunService  = game:GetService("RunService")
local Workspace   = game:GetService("Workspace")
local LocalPlayer = Players.LocalPlayer

-- ============================================================
-- HALLOWEEN THEME
-- ============================================================
local HalloweenTheme = {
    TextColor = Color3.fromRGB(240, 235, 225),
    Background = Color3.fromRGB(12, 10, 18),
    Topbar = Color3.fromRGB(18, 14, 26),
    Shadow = Color3.fromRGB(8, 6, 12),
    NotificationBackground = Color3.fromRGB(18, 14, 26),
    NotificationActionsBackground = Color3.fromRGB(230, 140, 30),
    TabBackground = Color3.fromRGB(30, 22, 40),
    TabStroke = Color3.fromRGB(60, 40, 80),
    TabBackgroundSelected = Color3.fromRGB(230, 120, 20),
    TabTextColor = Color3.fromRGB(220, 210, 200),
    SelectedTabTextColor = Color3.fromRGB(12, 10, 18),
    ElementBackground = Color3.fromRGB(22, 18, 30),
    ElementBackgroundHover = Color3.fromRGB(32, 26, 42),
    SecondaryElementBackground = Color3.fromRGB(16, 12, 22),
    ElementStroke = Color3.fromRGB(60, 40, 80),
    SecondaryElementStroke = Color3.fromRGB(45, 30, 60),
    SliderBackground = Color3.fromRGB(230, 120, 20),
    SliderProgress = Color3.fromRGB(230, 120, 20),
    SliderStroke = Color3.fromRGB(255, 160, 50),
    ToggleBackground = Color3.fromRGB(25, 20, 35),
    ToggleEnabled = Color3.fromRGB(230, 120, 20),
    ToggleDisabled = Color3.fromRGB(90, 70, 110),
    ToggleEnabledStroke = Color3.fromRGB(255, 160, 50),
    ToggleDisabledStroke = Color3.fromRGB(100, 80, 120),
    ToggleEnabledOuterStroke = Color3.fromRGB(100, 70, 40),
    ToggleDisabledOuterStroke = Color3.fromRGB(60, 45, 75),
    DropdownSelected = Color3.fromRGB(32, 26, 42),
    DropdownUnselected = Color3.fromRGB(22, 18, 30),
    InputBackground = Color3.fromRGB(22, 18, 30),
    InputStroke = Color3.fromRGB(60, 40, 80),
    PlaceholderColor = Color3.fromRGB(160, 140, 180),
}

local Window = Rayfield:CreateWindow({
    Name = "MM2 Halloween Hub",
    LoadingTitle = "MM2 Halloween Hub",
    LoadingSubtitle = "Delta / Rayfield",
    Theme = HalloweenTheme,
    ConfigurationSaving = {
        Enabled = true,
        FolderName = "MM2_Halloween_Hub",
        FileName = "MM2_Halloween_Config",
    },
    KeySystem = false,
})

-- ============================================================
-- ROLE CACHE
-- ============================================================
local RoleCache = {}
local RoleCacheTime = 0.6

local function getRole(player)
    if not player or not player.Character then return "Innocent" end
    local char = player.Character
    local backpack = player:FindFirstChild("Backpack")
    if char:FindFirstChild("Knife") or (backpack and backpack:FindFirstChild("Knife")) then
        return "Murderer"
    end
    if char:FindFirstChild("Gun") or (backpack and backpack:FindFirstChild("Gun")) then
        return "Sheriff"
    end
    return "Innocent"
end

task.spawn(function()
    while task.wait(RoleCacheTime) do
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer then
                RoleCache[plr] = getRole(plr)
            end
        end
    end
end)

-- ============================================================
-- FLING ENGINE (CFrame glitch — método que funciona)
-- ============================================================
local function flingPlayerRaw(target)
    if not target or not target.Character then return end
    local char = target.Character
    local hrp  = char:FindFirstChild("HumanoidRootPart")
    local hum  = char:FindFirstChildOfClass("Humanoid")
    if not hrp or not hum then return end

    pcall(function()
        for _, p in ipairs(char:GetDescendants()) do
            if p:IsA("BasePart") then p.Anchored = false end
        end
    end)

    pcall(function()
        hum.PlatformStand = true
        hum:ChangeState(Enum.HumanoidStateType.Physics)
    end)

    pcall(function()
        for _ = 1, 6 do
            hrp.CFrame = hrp.CFrame * CFrame.Angles(math.rad(1e6), math.rad(1e6), math.rad(1e6))
            hrp.CFrame = CFrame.new(math.huge, math.huge, math.huge)
            task.wait()
        end
    end)

    pcall(function()
        hrp.AssemblyLinearVelocity  = Vector3.new(1e7, 1e7, 1e7)
        hrp.AssemblyAngularVelocity = Vector3.new(1e7, 1e7, 1e7)
        hrp.Velocity    = Vector3.new(1e7, 1e7, 1e7)
        hrp.RotVelocity = Vector3.new(1e7, 1e7, 1e7)
    end)

    pcall(function()
        local bav = Instance.new("BodyAngularVelocity")
        bav.AngularVelocity = Vector3.new(1e7, 1e7, 1e7)
        bav.MaxTorque = Vector3.new(1e7, 1e7, 1e7)
        bav.P = 1e7
        bav.Parent = hrp
        task.delay(0.4, function()
            if bav and bav.Parent then bav:Destroy() end
        end)
    end)

    task.delay(1.2, function()
        pcall(function()
            if hum and hum.Parent then hum.PlatformStand = false end
        end)
    end)
end

-- ============================================================
-- BYPASS CONFIG + HELPERS
-- ============================================================
local Bypass = {
    Enabled       = false,
    SafeFling     = false,
    SafeTeleport  = false,
    SpeedClamp    = false,
    Jitter        = true,
    MaxSafeSpeed  = 45,
    FlingCooldown = 1.5,
    lastFling     = {},
    origWalkSpeed = {},
}

local function jitterVec(base, amt)
    if not Bypass.Jitter then return base end
    return base + Vector3.new(
        (math.random() - 0.5) * amt,
        (math.random() - 0.5) * amt,
        (math.random() - 0.5) * amt
    )
end

local function cooldownOk(player)
    if not (Bypass.Enabled and Bypass.SafeFling) then return true end
    local last = Bypass.lastFling[player] or 0
    if os.clock() - last < Bypass.FlingCooldown then return false end
    Bypass.lastFling[player] = os.clock()
    return true
end

local function safeFling(target)
    local char = target.Character
    local hrp  = char:FindFirstChild("HumanoidRootPart")
    local hum  = char:FindFirstChildOfClass("Humanoid")
    if not hrp or not hum then return false end

    pcall(function()
        for _, p in ipairs(char:GetDescendants()) do
            if p:IsA("BasePart") then p.Anchored = false end
        end
    end)

    pcall(function()
        hum.PlatformStand = true
        hum:ChangeState(Enum.HumanoidStateType.Physics)
    end)

    pcall(function()
        local v = jitterVec(Vector3.new(
            Bypass.MaxSafeSpeed * 6,
            Bypass.MaxSafeSpeed * 20,
            Bypass.MaxSafeSpeed * 6
        ), 40)
        hrp.AssemblyLinearVelocity  = v
        hrp.AssemblyAngularVelocity = jitterVec(Vector3.new(120, 120, 120), 60)
    end)

    task.delay(0.3, function()
        pcall(function()
            if hum and hum.Parent then hum.PlatformStand = false end
        end)
    end)
    return true
end

local function safeTeleport(targetPos)
    local char = LocalPlayer.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if not hrp then return false end
    local start = hrp.CFrame.Position
    local dist = (targetPos - start).Magnitude
    local steps = math.clamp(math.ceil(dist / 8), 4, 40)
    for i = 1, steps do
        local alpha = i / steps
        hrp.CFrame = CFrame.new(start:Lerp(targetPos, alpha) + jitterVec(Vector3.zero, 1.5))
        task.wait()
    end
    return true
end

-- Router: substitui flingPlayer por versão que respeita bypass
flingPlayer = function(target)
    if not target or not target.Character then return end
    if not cooldownOk(target) then return end
    if Bypass.Enabled and Bypass.SafeFling then
        safeFling(target)
    else
        flingPlayerRaw(target)
    end
end

local function flingRole(roleName)
    local count = 0
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then
            local cached = RoleCache[plr] or getRole(plr)
            if cached == roleName then
                flingPlayer(plr)
                count += 1
            end
        end
    end
    return count
end

-- ============================================================
-- SPEED CLAMP
-- ============================================================
local function installSpeedClamp()
    local char = LocalPlayer.Character
    local hum  = char and char:FindFirstChildOfClass("Humanoid")
    if not hum then return end
    Bypass.origWalkSpeed[LocalPlayer] = hum.WalkSpeed
    if Bypass._speedConn then pcall(function() Bypass._speedConn:Disconnect() end) end
    Bypass._speedConn = hum:GetPropertyChangedSignal("WalkSpeed"):Connect(function()
        if not (Bypass.Enabled and Bypass.SpeedClamp) then return end
        if hum.WalkSpeed > Bypass.MaxSafeSpeed then
            hum.WalkSpeed = Bypass.MaxSafeSpeed
        end
    end)
end

local function removeSpeedClamp()
    if Bypass._speedConn then
        pcall(function() Bypass._speedConn:Disconnect() end)
        Bypass._speedConn = nil
    end
    local char = LocalPlayer.Character
    local hum  = char and char:FindFirstChildOfClass("Humanoid")
    if hum and Bypass.origWalkSpeed[LocalPlayer] then
        hum.WalkSpeed = Bypass.origWalkSpeed[LocalPlayer]
    end
end

-- ============================================================
-- AUTO PICKUP
-- ============================================================
local AutoPickupEnabled = false
local PickupDistance = 12

local function findDroppedGun()
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("Tool") and (obj.Name == "Gun" or obj.Name == "Revolver") then
            return obj
        end
    end
    return nil
end

local function pickupWeapon()
    local char = LocalPlayer.Character
    if not char then return false end
    local hum  = char:FindFirstChildOfClass("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")
    if not hum or not root then return false end
    local gun = findDroppedGun()
    if not gun then return false end
    local dist = (gun:GetPivot().Position - root.Position).Magnitude
    if dist <= PickupDistance then
        hum:EquipTool(gun)
        return true
    end
    hum:MoveTo(gun:GetPivot().Position)
    task.wait(0.25)
    if gun.Parent and (gun:GetPivot().Position - root.Position).Magnitude <= PickupDistance then
        hum:EquipTool(gun)
        return true
    end
    return false
end

task.spawn(function()
    while task.wait(0.4) do
        if AutoPickupEnabled then pcall(pickupWeapon) end
    end
end)

-- ============================================================
-- ROLE ESP
-- ============================================================
local ESPFolder = Instance.new("Folder")
ESPFolder.Name = "MM2_Halloween_ESP"
ESPFolder.Parent = Workspace

local ESPObjects = {}

local function clearESP()
    for _, obj in pairs(ESPObjects) do
        pcall(function() obj:Destroy() end)
    end
    ESPObjects = {}
end

local function createESP(player, color)
    if not player.Character then return end
    local head = player.Character:FindFirstChild("Head")
    if not head then return end

    local billboard = Instance.new("BillboardGui")
    billboard.Name = "MM2_ESP_" .. player.Name
    billboard.Adornee = head
    billboard.Size = UDim2.new(0, 120, 0, 24)
    billboard.StudsOffset = Vector3.new(0, 2.2, 0)
    billboard.AlwaysOnTop = true
    billboard.Parent = ESPFolder

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.TextColor3 = color
    label.TextStrokeTransparency = 0
    label.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    label.TextScaled = true
    label.Font = Enum.Font.GothamBold
    label.Text = player.Name .. " [" .. (RoleCache[player] or "?") .. "]"
    label.Parent = billboard

    ESPObjects[player] = billboard
end

local ESPMurderer = false
local ESPSheriff  = false

local function refreshESP()
    clearESP()
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then
            local role = RoleCache[plr] or getRole(plr)
            if role == "Murderer" and ESPMurderer then
                createESP(plr, Color3.fromRGB(255, 40, 40))
            elseif role == "Sheriff" and ESPSheriff then
                createESP(plr, Color3.fromRGB(40, 120, 255))
            end
        end
    end
end

RunService.Heartbeat:Connect(function()
    for plr, billboard in pairs(ESPObjects) do
        if not plr.Character or not plr.Character:FindFirstChild("Head") or not billboard.Adornee then
            pcall(function() billboard:Destroy() end)
            ESPObjects[plr] = nil
        end
    end
    if ESPMurderer or ESPSheriff then
        for plr, billboard in pairs(ESPObjects) do
            local label = billboard:FindFirstChildOfClass("TextLabel")
            if label then
                label.Text = plr.Name .. " [" .. (RoleCache[plr] or getRole(plr)) .. "]"
            end
        end
    end
end)

-- ============================================================
-- TABS
-- ============================================================
local MainTab     = Window:CreateTab("Main", "home")
local FlingTab    = Window:CreateTab("Fling", "zap")
local ESPTab      = Window:CreateTab("ESP", "eye")
local BypassTab   = Window:CreateTab("Bypass", "shield")
local SettingsTab = Window:CreateTab("Settings", "settings")

-- ============================================================
-- MAIN TAB
-- ============================================================
MainTab:CreateSection("Role Awareness")

MainTab:CreateToggle({
    Name = "See Murderer",
    CurrentValue = false,
    Flag = "SeeMurderer",
    Callback = function(v) ESPMurderer = v; refreshESP() end,
})

MainTab:CreateToggle({
    Name = "See Sheriff",
    CurrentValue = false,
    Flag = "SeeSheriff",
    Callback = function(v) ESPSheriff = v; refreshESP() end,
})

MainTab:CreateSection("Weapon")

MainTab:CreateToggle({
    Name = "Auto Pickup Weapon",
    CurrentValue = false,
    Flag = "AutoPickup",
    Callback = function(v) AutoPickupEnabled = v end,
})

MainTab:CreateSlider({
    Name = "Pickup Range",
    Range = {4, 30},
    Increment = 1,
    Suffix = " studs",
    CurrentValue = 12,
    Flag = "PickupRange",
    Callback = function(v) PickupDistance = v end,
})

MainTab:CreateButton({
    Name = "Pickup Weapon Now",
    Callback = function()
        local ok = pickupWeapon()
        Rayfield:Notify({
            Title = "MM2 Hub",
            Content = ok and "Weapon picked up." or "No dropped weapon in range.",
            Duration = 3,
        })
    end,
})

-- ============================================================
-- FLING TAB
-- ============================================================
FlingTab:CreateSection("Fling by Role")

FlingTab:CreateButton({
    Name = "Fling All",
    Callback = function()
        local n = 0
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer then
                flingPlayer(plr)
                n += 1
            end
        end
        Rayfield:Notify({
            Title = "MM2 Hub",
            Content = "Flung " .. n .. " players.",
            Duration = 3,
        })
    end,
})

FlingTab:CreateButton({
    Name = "Fling Murderer",
    Callback = function()
        local n = flingRole("Murderer")
        Rayfield:Notify({
            Title = "MM2 Hub",
            Content = n > 0 and ("Flung " .. n .. " murderer(s).") or "No murderer found.",
            Duration = 3,
        })
    end,
})

FlingTab:CreateButton({
    Name = "Fling Sheriff",
    Callback = function()
        local n = flingRole("Sheriff")
        Rayfield:Notify({
            Title = "MM2 Hub",
            Content = n > 0 and ("Flung " .. n .. " sheriff(s).") or "No sheriff found.",
            Duration = 3,
        })
    end,
})

FlingTab:CreateButton({
    Name = "Fling Murderer + Sheriff",
    Callback = function()
        local m = flingRole("Murderer")
        local s = flingRole("Sheriff")
        Rayfield:Notify({
            Title = "MM2 Hub",
            Content = "Murderer: " .. m .. " | Sheriff: " .. s,
            Duration = 3,
        })
    end,
})

FlingTab:CreateButton({
    Name = "Fling Innocents",
    Callback = function()
        local n = flingRole("Innocent")
        Rayfield:Notify({
            Title = "MM2 Hub",
            Content = n > 0 and ("Flung " .. n .. " innocent(s).") or "No innocents found.",
            Duration = 3,
        })
    end,
})

FlingTab:CreateSection("Fling + Weapon")

FlingTab:CreateButton({
    Name = "Pickup Weapon Then Fling Murderer + Sheriff",
    Callback = function()
        pcall(pickupWeapon)
        task.wait(0.25)
        local m = flingRole("Murderer")
        local s = flingRole("Sheriff")
        Rayfield:Notify({
            Title = "MM2 Hub",
            Content = "Pickup attempted | Murderer: " .. m .. " | Sheriff: " .. s,
            Duration = 3,
        })
    end,
})

FlingTab:CreateButton({
    Name = "Pickup Weapon Then Fling Innocents",
    Callback = function()
        pcall(pickupWeapon)
        task.wait(0.25)
        local n = flingRole("Innocent")
        Rayfield:Notify({
            Title = "MM2 Hub",
            Content = "Pickup attempted | Innocents flung: " .. n,
            Duration = 3,
        })
    end,
})

-- ============================================================
-- ESP TAB
-- ============================================================
ESPTab:CreateSection("Role ESP")

ESPTab:CreateToggle({
    Name = "Murderer ESP",
    CurrentValue = false,
    Flag = "ESPMurderer",
    Callback = function(v) ESPMurderer = v; refreshESP() end,
})

ESPTab:CreateToggle({
    Name = "Sheriff ESP",
    CurrentValue = false,
    Flag = "ESPSheriff",
    Callback = function(v) ESPSheriff = v; refreshESP() end,
})

ESPTab:CreateButton({
    Name = "Refresh ESP",
    Callback = function()
        refreshESP()
        Rayfield:Notify({ Title = "MM2 Hub", Content = "ESP refreshed.", Duration = 2 })
    end,
})

ESPTab:CreateButton({
    Name = "Clear ESP",
    Callback = function()
        clearESP()
        Rayfield:Notify({ Title = "MM2 Hub", Content = "ESP cleared.", Duration = 2 })
    end,
})

-- ============================================================
-- BYPASS TAB
-- ============================================================
BypassTab:CreateSection("Master")

local statusLabel

BypassTab:CreateToggle({
    Name = "Enable Anti-Cheat Bypass",
    CurrentValue = false,
    Flag = "BypassMaster",
    Callback = function(v)
        Bypass.Enabled = v
        if v then installSpeedClamp() else removeSpeedClamp() end
        if statusLabel then
            statusLabel:Set(v and "Bypass: ON — camadas ativas" or "Bypass: OFF — comportamento original")
        end
        Rayfield:Notify({
            Title = "MM2 Bypass",
            Content = v and "Camadas de segurança ativadas." or "Bypass desligado.",
            Duration = 3,
        })
    end,
})

statusLabel = BypassTab:CreateLabel("Bypass: OFF — comportamento original")

BypassTab:CreateSection("Layers")

BypassTab:CreateToggle({
    Name = "Safe Fling (clamp + jitter + cooldown)",
    CurrentValue = false,
    Flag = "BypassSafeFling",
    Callback = function(v) Bypass.SafeFling = v end,
})

BypassTab:CreateToggle({
    Name = "Safe Teleport (stepped)",
    CurrentValue = false,
    Flag = "BypassSafeTP",
    Callback = function(v) Bypass.SafeTeleport = v end,
})

BypassTab:CreateToggle({
    Name = "Speed 
