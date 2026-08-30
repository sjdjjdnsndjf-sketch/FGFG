local Players = game:GetService("Players")

local TweenService = game:GetService("TweenService")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")
local HS = game:GetService("HttpService")
local player = Players.LocalPlayer

-- Master table
local M = {}

-- ------------------------------------------------------------
-- EARLY CONFIG LOAD
-- ------------------------------------------------------------
M.introSoundEnabled = true
M.introSongChoice = 3
M.introGUIEnabled = true
M.hitSoundEnabled = false
M.hitSoundChoice = "Minecraft Critical Hit"
M.hitSoundCustomId = ""
M.hitSoundVolume = 1.0
M.hitSoundPitch = 1.0
if isfile and isfile("CherryConfig.json") then
    local ok, data = pcall(function() return HS:JSONDecode(readfile("CherryConfig.json")) end)
    if ok and type(data) == "table" then
        if data.introSoundEnabled ~= nil then M.introSoundEnabled = data.introSoundEnabled end
        if data.introSongChoice then M.introSongChoice = data.introSongChoice end
        if data.introGUIEnabled ~= nil then M.introGUIEnabled = data.introGUIEnabled end
        if data.autoBatMode == "BAT V2" then M.autoBatMode = "BAT V2" end
        if data.hitSoundEnabled ~= nil then M.hitSoundEnabled = data.hitSoundEnabled == true end
        if type(data.hitSoundChoice) == "string" then M.hitSoundChoice = data.hitSoundChoice end
        if type(data.hitSoundCustomId) == "string" then M.hitSoundCustomId = data.hitSoundCustomId end
        if type(data.hitSoundVolume) == "number" then M.hitSoundVolume = data.hitSoundVolume end
        if type(data.hitSoundPitch) == "number" then M.hitSoundPitch = data.hitSoundPitch end
        if type(data.Theme) == "string" then M._savedTheme = data.Theme end
        if type(data.colorScheme) == "string" then M._savedTheme = data.colorScheme end
    end
end

-- ============================================================
local introSoundInstance = nil
local ONI_INTRO_URL = "https://litter.catbox.moe/em9tgo7cqc5acj8u.mp3"
local ONI_INTRO_FILE = "oni_intro.mp3"

local function playOniIntroSound()
    if not M.introSoundEnabled then return end
    task.spawn(function()
        local ok, data = pcall(function() return game:HttpGet(ONI_INTRO_URL) end)
        if ok and type(data) == "string" and #data > 0 and type(writefile) == "function" then
            pcall(function() writefile(ONI_INTRO_FILE, data) end)
        end
        local getAsset = getcustomasset or getsynasset
        if type(getAsset) ~= "function" or type(isfile) ~= "function" or not isfile(ONI_INTRO_FILE) then return end
        introSoundInstance = Instance.new("Sound")
        pcall(function()
            introSoundInstance.SoundId = getAsset(ONI_INTRO_FILE)
            introSoundInstance.Volume = 3
            introSoundInstance.Looped = false
            introSoundInstance.Parent = game:GetService("CoreGui")
            introSoundInstance:Play()
        end)
    end)
end

local function playOniTerminalIntro()
    if not M.introGUIEnabled then return end
    local playerGui = player:WaitForChild("PlayerGui")
    local gui = Instance.new("ScreenGui")
    gui.Name = "ONITerminalIntro"
    gui.IgnoreGuiInset = true
    gui.ResetOnSpawn = false
    gui.DisplayOrder = 999999
    gui.Parent = playerGui

    local backdrop = Instance.new("Frame")
    backdrop.Size = UDim2.fromScale(1, 1)
    backdrop.BackgroundColor3 = Color3.fromRGB(8, 10, 15)
    backdrop.BackgroundTransparency = 0.28
    backdrop.BorderSizePixel = 0
    backdrop.Parent = gui

    local blur = Instance.new("BlurEffect")
    blur.Name = "ONIIntroBlur"
    blur.Size = 28
    blur.Parent = Lighting

    local panel = Instance.new("Frame")
    panel.AnchorPoint = Vector2.new(0.5, 0.5)
    panel.Position = UDim2.fromScale(0.5, 0.5)
    panel.Size = UDim2.fromOffset(760, 430)
    panel.BackgroundColor3 = Color3.fromRGB(17, 19, 25)
    panel.BackgroundTransparency = 0.06
    panel.BorderSizePixel = 0
    panel.Parent = backdrop

    local panelStroke = Instance.new("UIStroke", panel)
    panelStroke.Color = Color3.fromRGB(140, 150, 165)
    panelStroke.Thickness = 1
    panelStroke.Transparency = 0.18

    local panelGradient = Instance.new("UIGradient")
    panelGradient.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(30, 34, 43)),
        ColorSequenceKeypoint.new(0.55, Color3.fromRGB(17, 19, 25)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(10, 12, 17)),
    })
    panelGradient.Rotation = 90
    panelGradient.Parent = panel

    local topBar = Instance.new("Frame")
    topBar.Size = UDim2.new(1, -2, 0, 30)
    topBar.Position = UDim2.fromOffset(1, 1)
    topBar.BackgroundColor3 = Color3.fromRGB(10, 12, 17)
    topBar.BackgroundTransparency = 0.08
    topBar.BorderSizePixel = 0
    topBar.Parent = panel
    local topLine = Instance.new("Frame")
    topLine.Size = UDim2.new(1, -40, 0, 1)
    topLine.Position = UDim2.fromOffset(20, 30)
    topLine.BackgroundColor3 = Color3.fromRGB(80, 88, 102)
    topLine.BackgroundTransparency = 0.45
    topLine.BorderSizePixel = 0
    topLine.Parent = panel
    local windowText = Instance.new("TextLabel")
    windowText.BackgroundTransparency = 1
    windowText.Position = UDim2.fromOffset(20, 5)
    windowText.Size = UDim2.new(1, -40, 0, 20)
    windowText.Text = "ONI TERMINAL  /  SECURE BOOT"
    windowText.TextColor3 = Color3.fromRGB(155, 165, 180)
    windowText.Font = Enum.Font.Code
    windowText.TextSize = 11
    windowText.TextXAlignment = Enum.TextXAlignment.Left
    windowText.Parent = panel

    local function makeLabel(parent, text, pos, size, color, font, textSize)
        local obj = Instance.new("TextLabel")
        obj.BackgroundTransparency = 1
        obj.Position = pos
        obj.Size = size
        obj.Text = text
        obj.TextColor3 = color
        obj.Font = font or Enum.Font.Code
        obj.TextSize = textSize or 13
        obj.TextXAlignment = Enum.TextXAlignment.Left
        obj.TextYAlignment = Enum.TextYAlignment.Center
        obj.Parent = parent
        return obj
    end

    local function corner(x, y, rot)
        local c = Instance.new("Frame")
        c.BackgroundColor3 = Color3.fromRGB(235, 38, 62)
        c.BorderSizePixel = 0
        c.Position = UDim2.new(x, 0, y, 0)
        c.Size = UDim2.fromOffset(4, 32)
        c.Rotation = rot or 0
        c.Parent = panel
        local h = Instance.new("Frame")
        h.BackgroundColor3 = Color3.fromRGB(235, 38, 62)
        h.BorderSizePixel = 0
        h.Position = UDim2.new(x, 0, y, 0)
        h.Size = UDim2.fromOffset(32, 4)
        h.Parent = panel
    end
    corner(0, 0, 0)
    corner(1, 0, 0)
    corner(0, 1, 0)
    corner(1, 1, 0)

    makeLabel(panel, "ONI HUB", UDim2.fromOffset(28, 18), UDim2.fromOffset(370, 54), Color3.fromRGB(245, 42, 65), Enum.Font.Code, 38)
    makeLabel(panel, "●  secure connection", UDim2.new(1, -230, 0, 19), UDim2.fromOffset(195, 22), Color3.fromRGB(78, 238, 145), Enum.Font.Code, 11).TextXAlignment = Enum.TextXAlignment.Right
    makeLabel(panel, "[ operation secure shell ]", UDim2.fromOffset(28, 354), UDim2.new(1, -56, 0, 20), Color3.fromRGB(135, 145, 160), Enum.Font.Code, 11)

    local log = Instance.new("Frame")
    log.Position = UDim2.fromOffset(30, 83)
    log.Size = UDim2.new(1, -60, 0, 168)
    log.BackgroundTransparency = 1
    log.Parent = panel

    local messages = {
        "[~] oni#hub > connect secure.shell",
        "[+] Authenticating usercode...",
        "[+] Loading core modules...",
        "[+] Loading assets.................",
        "[+] Loading GUI....................",
        "[+] Using saved config.............",
        "[+] Checking for saved token.......",
        "[+] Scanning environment...........",
        "[+] Starting BAT systems...........",
    }
    local rows = {}
    for i, text in ipairs(messages) do
        local row = makeLabel(log, text, UDim2.fromOffset(0, (i - 1) * 18), UDim2.new(1, 0, 0, 16), Color3.fromRGB(205, 213, 225), Enum.Font.Code, 12)
        row.TextTransparency = 1
        rows[i] = row
    end

    local status = makeLabel(panel, "Loading decals", UDim2.fromOffset(30, 267), UDim2.fromOffset(220, 20), Color3.fromRGB(215, 220, 230), Enum.Font.Code, 12)
    local okText = makeLabel(panel, "[OK]", UDim2.fromOffset(240, 267), UDim2.fromOffset(55, 20), Color3.fromRGB(75, 240, 145), Enum.Font.Code, 12)
    local percent = makeLabel(panel, "0%", UDim2.new(1, -86, 0, 267), UDim2.fromOffset(56, 20), Color3.fromRGB(220, 225, 235), Enum.Font.Code, 12)
    percent.TextXAlignment = Enum.TextXAlignment.Right

    local barBack = Instance.new("Frame")
    barBack.Position = UDim2.fromOffset(300, 267)
    barBack.Size = UDim2.new(1, -390, 0, 20)
    barBack.BackgroundColor3 = Color3.fromRGB(28, 45, 42)
    barBack.BorderSizePixel = 0
    barBack.Parent = panel

    local bar = Instance.new("Frame")
    bar.Size = UDim2.new(0, 0, 1, 0)
    bar.BackgroundColor3 = Color3.fromRGB(62, 240, 137)
    bar.BorderSizePixel = 0
    bar.Parent = barBack

    local function setRowVisible(index)
        if rows[index] then rows[index].TextTransparency = 0 end
    end

    for i = 1, #messages do
        setRowVisible(i)
        local progress = i / #messages
        bar.Size = UDim2.new(progress * 0.72, 0, 1, 0)
        percent.Text = tostring(math.floor(progress * 72)) .. "%"
        status.Text = (i <= 3 and "Loading decals") or (i <= 6 and "Loading GUI") or "Scanning environment"
        task.wait(0.08)
    end

    for i = 73, 100, 3 do
        bar.Size = UDim2.new(i / 100, 0, 1, 0)
        percent.Text = tostring(i) .. "%"
        task.wait(0.025)
    end
    bar.Size = UDim2.new(1, 0, 1, 0)
    percent.Text = "100%"
    status.Text = "ONI HUB ready"
    okText.Text = "[OK]"
    makeLabel(panel, "[+] ONI HUB READY!", UDim2.fromOffset(30, 318), UDim2.new(1, -60, 0, 22), Color3.fromRGB(70, 240, 145), Enum.Font.Code, 13)
    task.wait(0.35)
    gui:Destroy()
    pcall(function() blur:Destroy() end)
end

playOniIntroSound()
playOniTerminalIntro()
repeat task.wait() until game:IsLoaded()

-- ============================================================
-- THEME SYSTEM & CONFIG SETUP
-- ============================================================
-- THEME SYSTEM & CONFIG SETUP (MONOCHROME BASE + ACCENT HIGHLIGHTS)
-- ============================================================
local CHERRY_CONFIG_NAME = "CherryConfig.json"
local CHERRY_THEMES = {
    Default  = { Accent=Color3.fromRGB(168,85,247),  AccentDim=Color3.fromRGB(130,60,200)  },
    Purple   = { Accent=Color3.fromRGB(207,159,255), AccentDim=Color3.fromRGB(160,120,210) },
    Blue     = { Accent=Color3.fromRGB(58,140,255),  AccentDim=Color3.fromRGB(40,90,180)   },
    Red      = { Accent=Color3.fromRGB(255,60,75),   AccentDim=Color3.fromRGB(180,40,50)   },
    Pink     = { Accent=Color3.fromRGB(255,105,180), AccentDim=Color3.fromRGB(200,80,140)  },
    Yellow   = { Accent=Color3.fromRGB(255,215,0),   AccentDim=Color3.fromRGB(200,170,0)   },
    Grey     = { Accent=Color3.fromRGB(190,190,200), AccentDim=Color3.fromRGB(120,120,125) },
    Forest   = { Accent=Color3.fromRGB(46,220,130),  AccentDim=Color3.fromRGB(30,140,80)   },
    Cyan     = { Accent=Color3.fromRGB(0,230,255),   AccentDim=Color3.fromRGB(0,160,190)   },
    Orange   = { Accent=Color3.fromRGB(255,145,45),  AccentDim=Color3.fromRGB(200,100,30)  },
    White    = { Accent=Color3.fromRGB(240,240,245), AccentDim=Color3.fromRGB(160,160,170) },
}

local CherryConfig = { Theme="Default" }
if M._savedTheme and CHERRY_THEMES[M._savedTheme] then
    CherryConfig.Theme = M._savedTheme
end
M.colorScheme = CherryConfig.Theme

local CHERRY_ACCENT   = Color3.fromRGB(168, 85, 247)
local UI_ACCENT       = Color3.fromRGB(168, 85, 247)
local UI_ACCENT_DIM   = Color3.fromRGB(180, 180, 190)
local UI_BG_DARK      = Color3.fromRGB(0, 0, 0)
local UI_ROW_BG       = Color3.fromRGB(0, 0, 0)
local UI_CARD_STROKE  = Color3.fromRGB(90, 90, 96)
local UI_TEXT_WHITE   = Color3.fromRGB(255, 255, 255)
local UI_TEXT_PRIMARY = Color3.fromRGB(255, 255, 255)
local UI_TEXT_DIM     = Color3.fromRGB(125, 125, 125)
local UI_TEXT_SECTION = Color3.fromRGB(255, 255, 255)
local UI_BTN_BG       = Color3.fromRGB(0, 0, 0)
local UI_TOGGLE_OFF   = Color3.fromRGB(0, 0, 0)
local UI_TOGGLE_KNOB  = Color3.fromRGB(128, 128, 128)
local UI_KNOB_ON      = Color3.fromRGB(255, 255, 255)
local UI_GRAD_TOP     = Color3.fromRGB(0, 0, 0)
local UI_GRAD_BOT     = Color3.fromRGB(0, 0, 0)

local function themeDarkFromAccent(accent, amount)
    amount = math.clamp(tonumber(amount) or 0.12, 0, 1)
    return Color3.new(
        math.clamp(accent.R * amount, 0, 1),
        math.clamp(accent.G * amount, 0, 1),
        math.clamp(accent.B * amount, 0, 1)
    )
end

local function isNearBlack(c, threshold)
    if typeof(c) ~= "Color3" then return false end
    threshold = threshold or 0.14
    return c.R <= threshold and c.G <= threshold and c.B <= threshold
end

local function applyAccentFromTheme()
    local name = CherryConfig.Theme or M.colorScheme or M._savedTheme or "Default"
    if not CHERRY_THEMES[name] then name = "Default" end
    CherryConfig.Theme = name
    M.colorScheme = name
    M._savedTheme = name

    local themeData = CHERRY_THEMES[name]
    local accent = themeData.Accent or Color3.fromRGB(255, 255, 255)
    local dim    = themeData.AccentDim or Color3.fromRGB(180, 180, 190)

    -- MONOCHROME DARK BASE: all cards and rows remain black/dark, only accents change
    local bg      = Color3.fromRGB(0, 0, 0)
    local row     = Color3.fromRGB(0, 0, 0)
    local btn     = Color3.fromRGB(0, 0, 0)
    local tog     = Color3.fromRGB(0, 0, 0)
    local cardStk = Color3.fromRGB(90, 90, 96)
    local gradTop = Color3.fromRGB(0, 0, 0)
    local gradBot = Color3.fromRGB(0, 0, 0)

    CHERRY_ACCENT   = accent
    UI_ACCENT       = accent
    UI_ACCENT_DIM   = dim
    UI_BG_DARK      = bg
    UI_ROW_BG       = row
    UI_BTN_BG       = btn
    UI_TOGGLE_OFF   = tog
    UI_TOGGLE_KNOB  = Color3.fromRGB(128, 128, 128)
    UI_KNOB_ON      = accent
    UI_TEXT_PRIMARY = Color3.fromRGB(255, 255, 255)
    UI_TEXT_WHITE   = Color3.fromRGB(255, 255, 255)
    UI_TEXT_DIM     = Color3.fromRGB(125, 125, 125)
    UI_TEXT_SECTION = Color3.fromRGB(255, 255, 255)
    UI_CARD_STROKE  = cardStk
    UI_GRAD_TOP     = gradTop
    UI_GRAD_BOT     = gradBot

    M.Theme = {
        Name = name,
        Accent = accent,
        AccentDim = dim,
        Bg = bg,
        Row = row,
        Btn = btn,
        Stroke = cardStk,
    }
end

function M.recolorBlacksToTheme(root)
    if not root then return end
    local accent = UI_ACCENT or Color3.fromRGB(255, 255, 255)
    local function recolor(obj)
        if obj:IsA("UIStroke") and obj.Name == "MainStroke" then
            obj.Color = accent
        end
    end
    recolor(root)
    for _, d in ipairs(root:GetDescendants()) do
        recolor(d)
    end
end

applyAccentFromTheme()

-- ============================================================
-- SKY THEME (PERSISTENT & BULLETPROOF)
-- ============================================================
M.CANDY_SKY_TAG = "MoveeSkyTheme"
M.currentSkyTheme = "Night"
M._skyEnforcerRunning = false
M._skyConns = {}

M.CANDY_SKY_PRESETS = {
    ["Off"]={kind="off"},
    ["Night"]={clock=22,brightness=2,ambient={110,100,130},outAmb={120,110,140},sky={stars=4000,moon=18,sun=0,moonTex=true},atm={dens=0.45,color={120,60,180},decay={60,20,100},glare=0.5,haze=1.2}},
    ["Aurora"]={clock=14,brightness=3,ambient={150,120,150},outAmb={160,130,150},atm={dens=0.55,color={255,80,200},decay={255,20,150},glare=2.5,haze=3},clouds={cover=0.7,dens=0.7,color={255,240,250}}},
    ["Sunset"]={clock=17.2,brightness=2.5,ambient={170,120,100},outAmb={180,130,110},sky={stars=0,sun=25,moon=0},atm={dens=0.5,color={255,130,60},decay={255,80,30},glare=2,haze=2.5},clouds={cover=0.55,dens=0.55,color={255,200,140}}},
    ["Galaxy"]={clock=0,brightness=1.5,ambient={70,60,100},outAmb={80,70,110},sky={stars=10000,moon=30,sun=0},atm={dens=0.15,color={40,20,80},decay={20,10,50},glare=0.3,haze=0.5}},
    ["Cyber"]={clock=21,brightness=2.2,ambient={90,130,170},outAmb={100,140,180},sky={stars=2000,moon=12},atm={dens=0.4,color={0,200,255},decay={150,0,255},glare=2,haze=2},clouds={cover=0.4,dens=0.6,color={100,200,255}}},
    ["Sakura"]={clock=11,brightness=3.5,ambient={170,150,160},outAmb={180,160,170},sky={sun=8},atm={dens=0.3,color={255,200,220},decay={255,170,200},glare=1,haze=1.5},clouds={cover=0.6,dens=0.4,color={255,250,252}}},
    ["Pink Night"]={clock=23,brightness=2.2,ambient={120,60,110},outAmb={140,70,120},sky={stars=5000,moon=22,sun=0,moonTex=true},atm={dens=0.5,color={255,80,180},decay={140,30,100},glare=0.7,haze=1.4},clouds={cover=0.3,dens=0.5,color={180,90,150}}},
    ["Blood Moon"]={clock=22.5,brightness=1.6,ambient={130,40,40},outAmb={150,50,50},sky={stars=1500,moon=28,sun=0,moonTex=true},atm={dens=0.6,color={220,30,30},decay={120,10,10},glare=1.4,haze=2},clouds={cover=0.5,dens=0.7,color={120,30,30}}},
    ["Emerald Dawn"]={clock=6.5,brightness=2.8,ambient={130,170,140},outAmb={140,180,150},sky={sun=18,moon=0,stars=0},atm={dens=0.4,color={80,200,140},decay={40,150,90},glare=1.8,haze=2.2},clouds={cover=0.5,dens=0.5,color={200,255,220}}},
    ["Volcanic"]={clock=19,brightness=2,ambient={180,80,40},outAmb={200,90,50},sky={stars=200,sun=12,moon=0},atm={dens=0.75,color={255,60,0},decay={180,20,0},glare=3,haze=3.5},clouds={cover=0.8,dens=0.9,color={120,40,20}}},
    ["Arctic"]={clock=9,brightness=3.2,ambient={200,220,235},outAmb={210,230,245},sky={sun=10,stars=0,moon=0},atm={dens=0.3,color={180,220,255},decay={140,200,240},glare=1.5,haze=1.8},clouds={cover=0.7,dens=0.6,color={250,253,255}}},
    ["Midnight Ocean"]={clock=1.5,brightness=1.7,ambient={60,90,130},outAmb={70,100,140},sky={stars=6000,moon=24,sun=0,moonTex=true},atm={dens=0.5,color={20,60,140},decay={10,30,90},glare=0.6,haze=1.5}},
    ["Vaporwave"]={clock=19.5,brightness=2.4,ambient={180,120,200},outAmb={190,130,210},sky={stars=1000,moon=14},atm={dens=0.45,color={255,100,220},decay={120,60,255},glare=2.2,haze=2.4},clouds={cover=0.55,dens=0.55,color={200,150,255}}},
    ["Toxic"]={clock=13,brightness=2.5,ambient={140,180,80},outAmb={150,190,90},atm={dens=0.55,color={100,220,40},decay={60,150,20},glare=1.8,haze=2.6},clouds={cover=0.65,dens=0.7,color={180,255,120}}},
    ["Solar Eclipse"]={clock=12,brightness=0.9,ambient={50,40,60},outAmb={60,50,70},sky={stars=3500,sun=22,moon=0},atm={dens=0.5,color={255,140,40},decay={30,20,40},glare=2.8,haze=1.8}},
    ["Hellscape"]={clock=18,brightness=1.8,ambient={200,60,30},outAmb={220,70,40},sky={stars=100,sun=30,moon=0},atm={dens=0.85,color={255,30,0},decay={120,0,0},glare=3.5,haze=4},clouds={cover=0.95,dens=0.95,color={80,20,10}}},
    ["Heaven"]={clock=12,brightness=4,ambient={240,235,210},outAmb={250,245,220},sky={sun=16,moon=0,stars=0},atm={dens=0.25,color={255,250,220},decay={255,240,200},glare=3,haze=1.5},clouds={cover=0.85,dens=0.5,color={255,255,255}}},
    ["Storm"]={clock=15,brightness=1.4,ambient={90,90,110},outAmb={100,100,120},sky={stars=0,sun=6,moon=0},atm={dens=0.65,color={80,90,120},decay={40,50,80},glare=0.5,haze=3},clouds={cover=0.95,dens=0.95,color={60,65,80}}},
    ["Sunrise"]={clock=6.2,brightness=2.8,ambient={220,180,130},outAmb={230,190,140},sky={sun=22,stars=0,moon=0},atm={dens=0.45,color={255,180,100},decay={255,140,80},glare=2.4,haze=2.2},clouds={cover=0.4,dens=0.4,color={255,220,180}}},
    ["Deep Space"]={clock=0,brightness=1,ambient={30,25,50},outAmb={40,35,60},sky={stars=15000,moon=0,sun=0},atm={dens=0.08,color={15,5,40},decay={5,0,20},glare=0.2,haze=0.3}},
    ["Lavender Dream"]={clock=18.5,brightness=2.6,ambient={180,160,220},outAmb={190,170,230},sky={stars=800,moon=16,sun=0},atm={dens=0.4,color={200,160,255},decay={160,120,220},glare=1.4,haze=1.8},clouds={cover=0.55,dens=0.5,color={220,200,255}}},
    ["Inferno"]={clock=17.5,brightness=2.2,ambient={220,100,40},outAmb={235,110,50},sky={sun=26,moon=0,stars=0},atm={dens=0.6,color={255,90,20},decay={200,40,0},glare=3,haze=3.2},clouds={cover=0.7,dens=0.7,color={200,80,40}}},
    ["Mint Sky"]={clock=10,brightness=3.2,ambient={180,230,210},outAmb={190,240,220},sky={sun=10},atm={dens=0.32,color={150,255,210},decay={100,220,180},glare=1.6,haze=1.6},clouds={cover=0.55,dens=0.45,color={240,255,250}}},
}
M.SkyOrder = {"Off","Night","Aurora","Sunset","Galaxy","Cyber","Sakura","Pink Night","Blood Moon","Emerald Dawn","Volcanic","Arctic","Midnight Ocean","Vaporwave","Toxic","Solar Eclipse","Hellscape","Heaven","Storm","Sunrise","Deep Space","Lavender Dream","Inferno","Mint Sky"}

local function candyColor(rgb)
    if not rgb then return Color3.fromRGB(127,127,127) end
    return Color3.fromRGB(rgb[1],rgb[2],rgb[3])
end

local function cleanGameSkies(keepCustom)
    for _, child in ipairs(Lighting:GetChildren()) do
        if child:IsA("Sky") or child:IsA("Atmosphere") then
            if not child:GetAttribute(M.CANDY_SKY_TAG) then
                pcall(function() child:Destroy() end)
            elseif not keepCustom then
                pcall(function() child:Destroy() end)
            end
        end
    end
    local terrain = workspace:FindFirstChildOfClass("Terrain")
    if terrain then
        for _, child in ipairs(terrain:GetChildren()) do
            if child:IsA("Clouds") then
                if not child:GetAttribute(M.CANDY_SKY_TAG) then
                    pcall(function() child:Destroy() end)
                elseif not keepCustom then
                    pcall(function() child:Destroy() end)
                end
            end
        end
    end
end

function M._applySkyOnce(mode)
    local preset = M.CANDY_SKY_PRESETS[mode]
    if not preset or preset.kind == "off" or mode == "Off" then
        cleanGameSkies(false)
        Lighting.ClockTime = 14
        Lighting.Brightness = 2
        Lighting.OutdoorAmbient = Color3.fromRGB(127, 127, 127)
        Lighting.Ambient = Color3.fromRGB(127, 127, 127)
        Lighting.FogStart = 0
        Lighting.FogEnd = 100000
        Lighting.FogColor = Color3.fromRGB(200, 200, 200)
        Lighting.ColorShift_Top = Color3.fromRGB(0, 0, 0)
        Lighting.ColorShift_Bottom = Color3.fromRGB(0, 0, 0)
        Lighting.GlobalShadows = true
        return
    end

    cleanGameSkies(true)
    local terrain = workspace:FindFirstChildOfClass("Terrain")

    Lighting.ClockTime = preset.clock or 14
    Lighting.Brightness = preset.brightness or 2
    if preset.outAmb then Lighting.OutdoorAmbient = candyColor(preset.outAmb) end
    if preset.ambient then Lighting.Ambient = candyColor(preset.ambient) end
    Lighting.FogStart = 0
    Lighting.FogEnd = 100000
    Lighting.FogColor = Color3.fromRGB(200, 200, 200)
    Lighting.ColorShift_Top = Color3.fromRGB(0, 0, 0)
    Lighting.ColorShift_Bottom = Color3.fromRGB(0, 0, 0)
    Lighting.GlobalShadows = true

    if preset.sky then
        local skyInst = Lighting:FindFirstChild("MoveeCustomSky")
        if not skyInst or not skyInst:IsA("Sky") or not skyInst:GetAttribute(M.CANDY_SKY_TAG) then
            if skyInst then pcall(function() skyInst:Destroy() end) end
            skyInst = Instance.new("Sky")
            skyInst.Name = "MoveeCustomSky"
            skyInst:SetAttribute(M.CANDY_SKY_TAG, true)
            if preset.sky.stars then skyInst.StarCount = preset.sky.stars end
            if preset.sky.moon then skyInst.MoonAngularSize = preset.sky.moon end
            if preset.sky.sun then skyInst.SunAngularSize = preset.sky.sun end
            if preset.sky.moonTex then skyInst.MoonTextureId = "rbxasset://sky/moon.jpg" end
            skyInst.Parent = Lighting
        end
    else
        for _, child in ipairs(Lighting:GetChildren()) do
            if child:IsA("Sky") and child:GetAttribute(M.CANDY_SKY_TAG) then
                pcall(function() child:Destroy() end)
            end
        end
    end

    if preset.atm then
        local atm = Lighting:FindFirstChild("MoveeCustomAtmosphere")
        if not atm or not atm:IsA("Atmosphere") or not atm:GetAttribute(M.CANDY_SKY_TAG) then
            if atm then pcall(function() atm:Destroy() end) end
            atm = Instance.new("Atmosphere")
            atm.Name = "MoveeCustomAtmosphere"
            atm:SetAttribute(M.CANDY_SKY_TAG, true)
            atm.Density = preset.atm.dens or 0.3
            atm.Color = candyColor(preset.atm.color)
            atm.Decay = candyColor(preset.atm.decay)
            atm.Glare = preset.atm.glare or 1
            atm.Haze = preset.atm.haze or 1
            atm.Parent = Lighting
        end
    else
        for _, child in ipairs(Lighting:GetChildren()) do
            if child:IsA("Atmosphere") and child:GetAttribute(M.CANDY_SKY_TAG) then
                pcall(function() child:Destroy() end)
            end
        end
    end

    if preset.clouds and terrain then
        local clouds = terrain:FindFirstChild("MoveeCustomClouds")
        if not clouds or not clouds:IsA("Clouds") or not clouds:GetAttribute(M.CANDY_SKY_TAG) then
            if clouds then pcall(function() clouds:Destroy() end) end
            clouds = Instance.new("Clouds")
            clouds.Name = "MoveeCustomClouds"
            clouds:SetAttribute(M.CANDY_SKY_TAG, true)
            clouds.Cover = preset.clouds.cover or 0.5
            clouds.Density = preset.clouds.dens or 0.5
            clouds.Color = candyColor(preset.clouds.color)
            clouds.Parent = terrain
        end
    else
        if terrain then
            for _, child in ipairs(terrain:GetChildren()) do
                if child:IsA("Clouds") and child:GetAttribute(M.CANDY_SKY_TAG) then
                    pcall(function() child:Destroy() end)
                end
            end
        end
    end
end

function M.CandyApplyCustomSky(mode)
    M.currentSkyTheme = mode or "Night"
    M._applySkyOnce(M.currentSkyTheme)

    for _, c in ipairs(M._skyConns) do
        pcall(function() c:Disconnect() end)
    end
    M._skyConns = {}

    if M.currentSkyTheme == "Off" then
        return
    end

    local c1 = Lighting.ChildAdded:Connect(function(child)
        if M.currentSkyTheme == "Off" then return end
        if (child:IsA("Sky") or child:IsA("Atmosphere")) and not child:GetAttribute(M.CANDY_SKY_TAG) then
            pcall(function() child:Destroy() end)
            M._applySkyOnce(M.currentSkyTheme)
        end
    end)
    table.insert(M._skyConns, c1)

    local terrain = workspace:FindFirstChildOfClass("Terrain")
    if terrain then
        local c2 = terrain.ChildAdded:Connect(function(child)
            if M.currentSkyTheme == "Off" then return end
            if child:IsA("Clouds") and not child:GetAttribute(M.CANDY_SKY_TAG) then
                pcall(function() child:Destroy() end)
                M._applySkyOnce(M.currentSkyTheme)
            end
        end)
        table.insert(M._skyConns, c2)
    end

    local c3 = Lighting:GetPropertyChangedSignal("ClockTime"):Connect(function()
        if M.currentSkyTheme == "Off" then return end
        local preset = M.CANDY_SKY_PRESETS[M.currentSkyTheme]
        if preset and preset.clock and math.abs(Lighting.ClockTime - preset.clock) > 0.05 then
            Lighting.ClockTime = preset.clock
        end
    end)
    table.insert(M._skyConns, c3)

    if not M._skyEnforcerRunning then
        M._skyEnforcerRunning = true
        task.spawn(function()
            while true do
                task.wait(0.5)
                if M.currentSkyTheme and M.currentSkyTheme ~= "Off" then
                    pcall(function()
                        M._applySkyOnce(M.currentSkyTheme)
                    end)
                end
            end
        end)
    end
end

-- ============================================================
-- ANIMATION PACKS (unchanged)
-- ============================================================
M.PACKS = {
    ["Adidas Sports"] = {
        WalkAnim = 18537392113,
        RunAnim  = 18537384940,
        JumpAnim = 18537380791,
        FallAnim = 18537367238,
        SwimIdle = 18537387180,
        Swim     = 18537389531,
        Animation1 = 18537376492,
        Animation2 = 18537371272,
        ClimbAnim = 18537363391,
    },
    ["Adidas Community"] = {
        WalkAnim = 122150855457006,
        RunAnim  = 82598234841035,
        JumpAnim = 75290611992385,
        FallAnim = 98600215928904,
        SwimIdle = 109346520324160,
        Swim     = 133308483266208,
        Animation1 = 122257458498464,
        Animation2 = 102357151005774,
        ClimbAnim = 88763136693023,
    },
    ["Adidas Aura"] = {
        WalkAnim = 83842218823011,
        RunAnim  = 118320322718866,
        JumpAnim = 109996626521204,
        FallAnim = 95603166884636,
        SwimIdle = 94922130551805,
        Swim     = 134530128383903,
        Animation1 = 110211186840347,
        Animation2 = 114191137265065,
        ClimbAnim = 97824616490448,
    },
    ["Wicked Popular"] = {
        WalkAnim = 92072849924640,
        RunAnim = 72301599441680,
        JumpAnim = 104325245285198,
        FallAnim = 121152442762481,
        Animation1 = 118832222982049,
        ClimbAnim = 131326830509784,
        SwimIdle = 113199415118199,
        Swim = 99384245425157,
        Animation2 = 76049494037641,
    },
    Elder = {
        WalkAnim = 10921111375,
        RunAnim  = 10921104374,
        JumpAnim = 10921107367,
        FallAnim = 10921105765,
        SwimIdle = 10921110146,
        Swim     = 10921108971,
        ClimbAnim = 10921100400,
        Animation1 = 10921101664,
        Animation2 = 10921102574,
    },
    Zombie = {
        WalkAnim = 10921355261,
        RunAnim  = 616163682,
        JumpAnim = 10921351278,
        FallAnim = 10921350320,
        SwimIdle = 10921353442,
        Swim     = 10921352344,
        Animation1 = 10921344533,
        Animation2 = 10921345304,
        ClimbAnim = 10921343576,
    },
    Mage = {
        WalkAnim = 10921152678,
        RunAnim  = 10921148209,
        JumpAnim = 10921149743,
        FallAnim = 10921148939,
        SwimIdle = 10921151661,
        Swim     = 10921150788,
        ClimbAnim = 10921143404,
        Animation1 = 10921144709,
        Animation2 = 10921145797,
    },
    ["Catwalk Glam"] = {
        WalkAnim = 109168724482748,
        RunAnim  = 81024476153754,
        JumpAnim = 116936326516985,
        FallAnim = 92294537340807,
        SwimIdle = 98854111361360,
        Swim     = 134591743181628,
        ClimbAnim = 119377220967554,
        Animation1 = 133806214992291,
        Animation2 = 94970088341563,
    },
    Astronaut = {
        WalkAnim = 10921046031,
        RunAnim  = 10921039308,
        JumpAnim = 10921042494,
        FallAnim = 10921040576,
        SwimIdle = 10921045006,
        Swim     = 10921044000,
        ClimbAnim = 10921032124,
        Animation1 = 10921034824,
        Animation2 = 10921036806,
    },
    ['Wicked "Dancing Through Life"'] = {
        WalkAnim = 73718308412641,
        RunAnim  = 135515454877967,
        JumpAnim = 78508480717326,
        FallAnim = 78147885297412,
        SwimIdle = 129183123083281,
        Swim     = 110657013921774,
        ClimbAnim = 129447497744818,
        Animation1 = 92849173543269,
        Animation2 = 132238900951109,
    },
    Werewolf = {
        WalkAnim = 10921342074,
        RunAnim  = 10921336997,
        JumpAnim = nil,
        FallAnim = 10921337907,
        SwimIdle = 10921341319,
        Swim     = 10921340419,
        ClimbAnim = 10921329322,
        Animation1 = 10921330408,
        Animation2 = 10921333667,
    },
    Superhero = {
        WalkAnim = 10921298616,
        RunAnim  = 10921291831,
        JumpAnim = 10921294559,
        FallAnim = 10921293373,
        SwimIdle = 10921297391,
        Swim     = 10921295495,
        ClimbAnim = 10921286911,
        Animation1 = 10921288909,
        Animation2 = 10921290167,
    },
    Toy = {
        WalkAnim = 10921312010,
        RunAnim  = 10921306285,
        JumpAnim = 10921308158,
        FallAnim = 10921307241,
        SwimIdle = 10921310341,
        Swim     = 10921309319,
        ClimbAnim = 10921300839,
        Animation1 = 10921301576,
        Animation2 = nil,
    },
    ["No Boundaries"] = {
        WalkAnim = 18747074203,
        RunAnim  = 18747070484,
        JumpAnim = 18747069148,
        FallAnim = 18747062535,
        SwimIdle = 18747071682,
        Swim     = 18747073181,
        ClimbAnim = 18747060903,
        Animation1 = 18747067405,
        Animation2 = 18747063918,
    },
    NFL = {
        WalkAnim = 110358958299415,
        RunAnim  = 117333533048078,
        JumpAnim = 119846112151352,
        FallAnim = 129773241321032,
        SwimIdle = 79090109939093,
        Swim     = 132697394189921,
        ClimbAnim = 134630013742019,
        Animation1 = 92080889861410,
        Animation2 = 74451233229259,
    },
    ["Amazon Unboxed"] = {
        WalkAnim = 90478085024465,
        RunAnim  = 134824450619865,
        JumpAnim = 121454505477205,
        FallAnim = 94788218468396,
        SwimIdle = 129126268464847,
        Swim     = 105962919001086,
        ClimbAnim = 121145883950231,
        Animation1 = 98281136301627,
        Animation2 = nil,
    },
    Vampire = {
        WalkAnim = 10921326949,
        RunAnim  = 10921320299,
        JumpAnim = 10921322186,
        FallAnim = 10921321317,
        SwimIdle = 10921325443,
        Swim     = 10921324408,
        ClimbAnim = 10921314188,
        Animation1 = 10921315373,
        Animation2 = nil,
    },
    Ninja = {
        Run=656118852, Walk=656121766, Jump=656117878, Fall=656115606,
        Swim=656119721, SwimIdle=656121397, Climb=656114359,
        Idle={656117400,656118341,886742569}
    },
    Robot = {
        Run=616091570, Walk=616095330, Jump=616090535, Fall=616087089,
        Swim=616092998, SwimIdle=616094091, Climb=616086039,
        Idle={616088211,616089559,885531463}
    },
    Levitation = {
        Run=616010382, Walk=616013216, Jump=616008936, Fall=616005863,
        Swim=616011509, SwimIdle=616012453, Climb=616003713,
        Idle={616006778,616008087,886862142}
    },
    Stylish = {
        Run=616140816, Walk=616146177, Jump=616139451, Fall=616134815,
        Swim=616143378, SwimIdle=616144772, Climb=616133594,
        Idle={616136790,616138447,886888594}
    },
    Bubbly = {
        Run=910025107, Walk=910034870, Jump=910016857, Fall=910001910,
        Swim=910028158, SwimIdle=910030921, Climb=909997997,
        Idle={910004836,910009958,1018536639}
    },
    Cartoon = {
        Run=742638842, Walk=742640026, Jump=742637942, Fall=742637151,
        Swim=742639220, SwimIdle=742639812, Climb=742636889,
        Idle={742637544,742638445,885477856}
    },
    ["Tryard"] = {
        WalkAnim = 707897309,
        RunAnim = 707861613,
        JumpAnim = 116936326516985,
        FallAnim = 116936326516985,
        ClimbAnim = 116936326516985,
        Swim = 116936326516985,
        SwimIdle = 116936326516985,
        Animation1 = 133806214992291,
        Animation2 = 94970088341563,
    },
}
M.animPack = "Adidas Sports"
M.animPackEnabled = true
M.savedAnimate = nil

-- Compatibilidade local importada do message(6): presets e aliases.
-- Links, polling web, speed e reset não são importados.
M.MESSAGE6_PRESET_FILE = "ONI_Message6_Presets.json"
function M.buildMessage6PresetSnapshot()
    return {
        version = 1,
        autoLeftEnabled = M.autoLeftEnabled,
        autoRightEnabled = M.autoRightEnabled,
        antiRagdollEnabled = M.antiRagdollEnabled,
        tryardAnimEnabled = M.animPack == "Tryard" and M.animPackEnabled or false,
        tpBatEnabled = M.bypassAimbotEnabled,
        stealMode = M.stealMode,
        autoStealEnabled = M.Steal and M.Steal.AutoStealEnabled or false,
    }
end
function M.saveMessage6Preset(name)
    if type(writefile) ~= "function" or type(HS) ~= "table" then return false end
    name = tostring(name or "Default")
    local presets = {}
    if type(isfile) == "function" and isfile(M.MESSAGE6_PRESET_FILE) then
        pcall(function() presets = HS:JSONDecode(readfile(M.MESSAGE6_PRESET_FILE)) or {} end)
    end
    presets[name] = M.buildMessage6PresetSnapshot()
    return pcall(function() writefile(M.MESSAGE6_PRESET_FILE, HS:JSONEncode(presets)) end)
end
function M.loadMessage6Preset(name)
    if type(isfile) ~= "function" or type(readfile) ~= "function" or type(HS) ~= "table" then return false end
    if not isfile(M.MESSAGE6_PRESET_FILE) then return false end
    local ok, presets = pcall(function() return HS:JSONDecode(readfile(M.MESSAGE6_PRESET_FILE)) end)
    local cfg = ok and presets and presets[tostring(name or "Default")] or nil
    if type(cfg) ~= "table" then return false end
    if cfg.antiRagdollEnabled ~= nil then
        M.antiRagdollEnabled = cfg.antiRagdollEnabled == true
        if M.antiRagdollEnabled then M.startAntiRagdoll() else M.stopAntiRagdoll() end
    end
    if cfg.tpBatEnabled ~= nil then
        M.bypassAimbotEnabled = cfg.tpBatEnabled == true
        if M.bypassAimbotEnabled then M.startBypassAimbot() else M.stopBypassAimbot() end
    end
    if cfg.stealMode then M.stealMode = cfg.stealMode end
    if M.Steal and cfg.autoStealEnabled ~= nil then
        M.Steal.AutoStealEnabled = cfg.autoStealEnabled == true
    end
    if cfg.tryardAnimEnabled then
        M.animPack = "Tryard"
        M.animPackEnabled = true
        M.applyAnimPack("Tryard")
    end
    return true
end
function M.toggleTPBat()
    M.bypassAimbotEnabled = not M.bypassAimbotEnabled
    if M.bypassAimbotEnabled then M.startBypassAimbot() else M.stopBypassAimbot() end
    if M.mobBtnRefs and M.mobBtnRefs.bypass then M.mobBtnRefs.bypass(M.bypassAimbotEnabled) end
    return M.bypassAimbotEnabled
end
function M.startTryardAnim()
    M.animPack = "Tryard"
    M.animPackEnabled = true
    return M.applyAnimPack("Tryard")
end
function M.stopTryardAnim()
    M.animPackEnabled = false
    return M.resetAnimations(player.Character)
end
-- ============================================================
-- CHARTER FEATURES (Headless & Korblox)
-- ============================================================
M.headlessEnabled = false
M.korbloxEnabled = false
M.wingsAuraEnabled = false
M._wingsAuraModel = nil
M._wingsAuraTemplate = nil
M._wingsAuraFollowConn = nil
M._wingsAuraApplying = false

local HEADLESS_MESH_ID = "rbxassetid://1095708"
local KORBLOX_MESH_ID = "rbxassetid://101851696"
local KORBLOX_TEXTURE_ID = "rbxassetid://101851254"
local DARK_GREY_COLOR = Color3.fromRGB(64, 64, 64)

local function removeFace(head)
    local face = head:FindFirstChild("face")
    if face then face:Destroy() end
end

function M.applyHeadlessToChar(char, enabled)
    if not char then return end
    local head = char:FindFirstChild("Head")
    if not head then return end

    if enabled then
        head.Transparency = 1
        head.CanCollide = false
        removeFace(head)

        for _, child in ipairs(head:GetChildren()) do
            if child:IsA("SpecialMesh") and child.MeshId == HEADLESS_MESH_ID then
                child:Destroy()
            end
        end

        local mesh = Instance.new("SpecialMesh")
        mesh.MeshType = Enum.MeshType.FileMesh
        mesh.MeshId = HEADLESS_MESH_ID
        mesh.Scale = Vector3.new(0.001, 0.001, 0.001)
        mesh.Name = "HeadlessMesh"
        mesh.Parent = head

        head:GetPropertyChangedSignal("Transparency"):Connect(function()
            if head.Transparency ~= 1 then
                head.Transparency = 1
            end
        end)
        head.ChildAdded:Connect(function(child)
            if child.Name == "face" and child:IsA("Decal") then
                child:Destroy()
            end
        end)
    else
        head.Transparency = 0
        head.CanCollide = true
        for _, child in ipairs(head:GetChildren()) do
            if child:IsA("SpecialMesh") and child.Name == "HeadlessMesh" then
                child:Destroy()
            end
        end
        removeFace(head)
    end
end

function M.applyKorbloxToChar(char, enabled)
    if not char then return end
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end

    if enabled then
        if humanoid.RigType == Enum.HumanoidRigType.R6 then
            local rightLeg = char:FindFirstChild("Right Leg")
            if rightLeg then
                for _, child in ipairs(rightLeg:GetChildren()) do
                    if child:IsA("SpecialMesh") or child:IsA("CharacterMesh") then
                        child:Destroy()
                    end
                end
                rightLeg.Color = DARK_GREY_COLOR
                rightLeg:GetPropertyChangedSignal("Color"):Connect(function()
                    if rightLeg.Color ~= DARK_GREY_COLOR then
                        rightLeg.Color = DARK_GREY_COLOR
                    end
                end)
                local mesh = Instance.new("SpecialMesh")
                mesh.MeshType = Enum.MeshType.FileMesh
                mesh.MeshId = KORBLOX_MESH_ID
                mesh.TextureId = KORBLOX_TEXTURE_ID
                mesh.Scale = Vector3.new(1, 1, 1)
                mesh.Name = "KorbloxMesh"
                mesh.Parent = rightLeg
            end
        elseif humanoid.RigType == Enum.HumanoidRigType.R15 then
            local rightUpperLeg = char:FindFirstChild("RightUpperLeg")
            if rightUpperLeg then
                rightUpperLeg.Transparency = 1
                local rightLowerLeg = char:FindFirstChild("RightLowerLeg")
                local rightFoot = char:FindFirstChild("RightFoot")
                if rightLowerLeg then rightLowerLeg.Transparency = 1 end
                if rightFoot then rightFoot.Transparency = 1 end

                local oldKorblox = char:FindFirstChild("KorbloxLeg")
                if oldKorblox then oldKorblox:Destroy() end

                local korbloxLeg = Instance.new("Part")
                korbloxLeg.Name = "KorbloxLeg"
                korbloxLeg.Size = Vector3.new(1, 2, 1)
                korbloxLeg.Anchored = false
                korbloxLeg.CanCollide = false
                korbloxLeg.Color = DARK_GREY_COLOR
                korbloxLeg.Parent = char

                local mesh = Instance.new("SpecialMesh")
                mesh.MeshType = Enum.MeshType.FileMesh
                mesh.MeshId = KORBLOX_MESH_ID
                mesh.TextureId = KORBLOX_TEXTURE_ID
                mesh.Scale = Vector3.new(1, 1, 1)
                mesh.Name = "KorbloxMesh"
                mesh.Parent = korbloxLeg

                local weld = Instance.new("Weld")
                weld.Part0 = rightUpperLeg
                weld.Part1 = korbloxLeg
                weld.C0 = CFrame.new(0, -0.8, 0)
                weld.Name = "KorbloxWeld"
                weld.Parent = korbloxLeg
            end
        end
    else
        if humanoid.RigType == Enum.HumanoidRigType.R6 then
            local rightLeg = char:FindFirstChild("Right Leg")
            if rightLeg then
                for _, child in ipairs(rightLeg:GetChildren()) do
                    if child:IsA("SpecialMesh") and child.Name == "KorbloxMesh" then
                        child:Destroy()
                    end
                end
                rightLeg.Color = Color3.fromRGB(255, 255, 255)
            end
        elseif humanoid.RigType == Enum.HumanoidRigType.R15 then
            local rightUpperLeg = char:FindFirstChild("RightUpperLeg")
            if rightUpperLeg then
                rightUpperLeg.Transparency = 0
                local rightLowerLeg = char:FindFirstChild("RightLowerLeg")
                local rightFoot = char:FindFirstChild("RightFoot")
                if rightLowerLeg then rightLowerLeg.Transparency = 0 end
                if rightFoot then rightFoot.Transparency = 0 end
                local korbloxLeg = char:FindFirstChild("KorbloxLeg")
                if korbloxLeg then korbloxLeg:Destroy() end
            end
        end
    end
end

function M.applyCharterToChar(char)
    if not char then return end
    M.applyHeadlessToChar(char, M.headlessEnabled)
    M.applyKorbloxToChar(char, M.korbloxEnabled)
end

player.CharacterAdded:Connect(function(char)
    task.wait(0.15)
    M.applyCharterToChar(char)
    if M.wingsAuraEnabled then
        M.applyWingsAura(char)
        -- Games often rebuild the character a moment later and strip extras.
        task.delay(0.8, function()
            if M.wingsAuraEnabled and player.Character == char then
                M.applyWingsAura(char)
            end
        end)
    end
end)

-- Only re-apply headless/korblox when one is actually enabled, and at most
-- a few times a second — NOT every frame (that alone tanked FPS).
task.spawn(function()
    while true do
        task.wait(0.5)
        if M.headlessEnabled or M.korbloxEnabled then
            local char = player.Character
            if char then M.applyCharterToChar(char) end
        end
    end
end)

-- ============================================================
-- WINGS AURA (Asset ID 90816193741219)
-- ============================================================
local WINGS_AURA_ASSET_ID = 90816193741219
local InsertService = game:GetService("InsertService")

-- This asset is a Dummy wearing particle "wings". Never parent the Dummy.
local WINGS_DUMMY_PARTS = {
    Head = true, Torso = true, HumanoidRootPart = true,
    ["Left Arm"] = true, ["Right Arm"] = true, ["Left Leg"] = true, ["Right Leg"] = true,
    UpperTorso = true, LowerTorso = true,
    LeftUpperArm = true, LeftLowerArm = true, LeftHand = true,
    RightUpperArm = true, RightLowerArm = true, RightHand = true,
    LeftUpperLeg = true, LeftLowerLeg = true, LeftFoot = true,
    RightUpperLeg = true, RightLowerLeg = true, RightFoot = true,
}

local function isWingsDummyBodyPart(inst)
    if not inst or not inst:IsA("BasePart") then return false end
    if WINGS_DUMMY_PARTS[inst.Name] then return true end
    local n = string.lower(inst.Name)
    return n == "dummy" or n == "dummyhead" or n == "nub"
end

local function isWingsCreditPart(inst)
    if not inst then return false end
    local n = string.lower(inst.Name)
    return n:find("credit", 1, true) or n:find("label", 1, true) or n:find("title", 1, true)
        or n:find("billboard", 1, true) or n:find("nametag", 1, true) or n:find("watermark", 1, true)
        or n:find("text", 1, true) or n:find("gui", 1, true) or n == "name" or n == "info"
end

local function isWingsFxInstance(inst)
    return inst:IsA("ParticleEmitter") or inst:IsA("Beam") or inst:IsA("Trail")
        or inst:IsA("PointLight") or inst:IsA("SpotLight") or inst:IsA("SurfaceLight")
        or inst:IsA("Fire") or inst:IsA("Smoke") or inst:IsA("Sparkles")
        or inst:IsA("Attachment")
end

local function stripWingsTextGuis(root)
    if not root then return end
    local kill = {}
    local list = {root}
    pcall(function()
        for _, d in ipairs(root:GetDescendants()) do
            table.insert(list, d)
        end
    end)
    for _, d in ipairs(list) do
        if d:IsA("BillboardGui") or d:IsA("SurfaceGui") or d:IsA("TextLabel")
            or d:IsA("TextButton") or d:IsA("TextBox") then
            table.insert(kill, d)
        end
    end
    for _, d in ipairs(kill) do
        pcall(function() d:Destroy() end)
    end
end

local function enableWingsFx(root)
    if not root then return end
    if root:IsA("ParticleEmitter") or root:IsA("Beam") or root:IsA("Trail")
        or root:IsA("Fire") or root:IsA("Smoke") or root:IsA("Sparkles") then
        root.Enabled = true
    elseif root:IsA("PointLight") or root:IsA("SpotLight") or root:IsA("SurfaceLight") then
        root.Enabled = true
    end
    for _, d in ipairs(root:GetDescendants()) do
        if d:IsA("ParticleEmitter") or d:IsA("Beam") or d:IsA("Trail")
            or d:IsA("Fire") or d:IsA("Smoke") or d:IsA("Sparkles") then
            d.Enabled = true
        elseif d:IsA("PointLight") or d:IsA("SpotLight") or d:IsA("SurfaceLight") then
            d.Enabled = true
        end
    end
end

local function neutralizeWingsAsset(asset)
    if not asset then return end
    pcall(function() asset.Parent = nil end)
    local toKill = {}
    for _, desc in ipairs(asset:GetDescendants()) do
        if desc:IsA("Humanoid") or desc:IsA("Animator") or desc:IsA("AnimationController")
            or desc:IsA("Script") or desc:IsA("LocalScript") or desc:IsA("ModuleScript")
            or desc:IsA("BodyColors") or desc:IsA("Shirt") or desc:IsA("Pants")
            or desc:IsA("ShirtGraphic") or desc:IsA("CharacterMesh")
            or desc:IsA("ForceField") or desc:IsA("HumanoidDescription")
            or desc:IsA("BillboardGui") or desc:IsA("SurfaceGui")
            or desc:IsA("TextLabel") or desc:IsA("TextButton") or desc:IsA("TextBox") then
            table.insert(toKill, desc)
        elseif desc:IsA("BasePart") then
            desc.Anchored = true
            desc.CanCollide = false
            desc.Massless = true
            pcall(function()
                desc.CanQuery = false
                desc.CanTouch = false
                desc.AssemblyLinearVelocity = Vector3.zero
                desc.AssemblyAngularVelocity = Vector3.zero
            end)
        end
    end
    for _, inst in ipairs(toKill) do
        pcall(function() inst:Destroy() end)
    end
end

local function stripStrayWingsDummies(char)
    if not char then return end
    for _, child in ipairs(char:GetChildren()) do
        if child ~= char and child:IsA("Model") then
            local extraHum = child:FindFirstChildOfClass("Humanoid")
            local looksDummy = extraHum
                or child.Name == "Dummy"
                or child:FindFirstChild("HumanoidRootPart") and child:FindFirstChild("Head") and child:FindFirstChild("Torso")
            if looksDummy and not child:FindFirstChild("WingsAuraTag") then
                pcall(function() child:Destroy() end)
            end
        elseif child:IsA("Humanoid") and child ~= char:FindFirstChildOfClass("Humanoid") then
            pcall(function() child:Destroy() end)
        end
    end
end

local function tagWingsInstance(inst)
    local tag = Instance.new("BoolValue")
    tag.Name = "WingsAuraTag"
    tag.Value = true
    tag.Parent = inst
    return inst
end

local function partHasWingsFx(part)
    if part:FindFirstChildOfClass("ParticleEmitter")
        or part:FindFirstChildOfClass("Beam")
        or part:FindFirstChildOfClass("Trail")
        or part:FindFirstChildOfClass("PointLight")
        or part:FindFirstChildOfClass("Fire")
        or part:FindFirstChildOfClass("Smoke")
        or part:FindFirstChildOfClass("Sparkles") then
        return true
    end
    for _, d in ipairs(part:GetDescendants()) do
        if d:IsA("ParticleEmitter") or d:IsA("Beam") or d:IsA("Trail")
            or d:IsA("PointLight") or d:IsA("SpotLight") or d:IsA("SurfaceLight")
            or d:IsA("Fire") or d:IsA("Smoke") or d:IsA("Sparkles") then
            return true
        end
    end
    return false
end

local function loadWingsAsset()
    local asset = nil

    if typeof(game.GetObjects) == "function" then
        local ok, objects = pcall(function()
            return game:GetObjects("rbxassetid://" .. WINGS_AURA_ASSET_ID)
        end)
        if ok and objects and #objects > 0 then
            asset = objects[1]
        end
    end

    if not asset then
        local ok2, model = pcall(function()
            return InsertService:LoadAsset(WINGS_AURA_ASSET_ID)
        end)
        if ok2 and model then
            asset = model
        end
    end

    neutralizeWingsAsset(asset)
    return asset
end

local function findWingsOriginCF(asset)
    local root = asset:FindFirstChild("HumanoidRootPart") or asset:FindFirstChild("Torso") or asset:FindFirstChild("UpperTorso")
    if not root then
        for _, d in ipairs(asset:GetDescendants()) do
            if d:IsA("BasePart") and (d.Name == "HumanoidRootPart" or d.Name == "Torso" or d.Name == "UpperTorso") then
                root = d
                break
            end
        end
    end
    if root and root:IsA("BasePart") then
        return root.CFrame
    end
    local ok, pivot = pcall(function() return asset:GetPivot() end)
    if ok and pivot then return pivot end
    return CFrame.new()
end

local function placeWingsProxy(proxy, originCF, sourceCF)
    proxy.Anchored = true
    proxy.CanCollide = false
    proxy.Massless = true
    pcall(function()
        proxy.CanQuery = false
        proxy.CanTouch = false
    end)
    proxy.CFrame = originCF:ToObjectSpace(sourceCF)
end

function M._isUnderWingsAura(obj)
    local p = obj
    local n = 0
    while p and n < 24 do
        if p == M._wingsAuraModel or p == M._wingsAuraTemplate then return true end
        if p.Name == "WingsAura" or p.Name == "WingsAuraRoot" then return true end
        if p:IsA("Model") and p:FindFirstChild("WingsAuraTag") then return true end
        p = p.Parent
        n += 1
    end
    return false
end

local function getWingsAuraParent()
    return workspace.CurrentCamera or workspace
end

local function stopWingsAuraFollow()
    if M._wingsAuraFollowConn then
        pcall(function() M._wingsAuraFollowConn:Disconnect() end)
        M._wingsAuraFollowConn = nil
    end
end

local function startWingsAuraFollow()
    stopWingsAuraFollow()
    M._wingsAuraFollowConn = RunService.Heartbeat:Connect(function()
        if not M.wingsAuraEnabled then return end
        local model = M._wingsAuraModel
        if not model or not model.Parent then return end
        local char = player.Character
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        if not hrp then return end
        pcall(function() model:PivotTo(hrp.CFrame) end)
    end)
end

local function wingsAuraIsAlive()
    local model = M._wingsAuraModel
    if not model or not model.Parent then return false end
    if not model:FindFirstChildWhichIsA("BasePart", true) then return false end
    return true
end

local function buildWingsAuraContainer(asset)
    local container = Instance.new("Model")
    container.Name = "WingsAura"
    tagWingsInstance(container)

    local root = Instance.new("Part")
    root.Name = "WingsAuraRoot"
    root.Size = Vector3.new(0.1, 0.1, 0.1)
    root.Transparency = 1
    root.Anchored = true
    root.CanCollide = false
    root.Massless = true
    root.CastShadow = false
    root.CFrame = CFrame.new()
    root.Parent = container
    container.PrimaryPart = root

    local originCF = findWingsOriginCF(asset)
    local equipped = false

    for _, part in ipairs(asset:GetDescendants()) do
        if not part:IsA("BasePart") then
            -- skip
        elseif isWingsCreditPart(part) then
            -- skip nametags
        elseif isWingsDummyBodyPart(part) then
            if partHasWingsFx(part) then
                local proxy = Instance.new("Part")
                proxy.Name = part.Name .. "_FX"
                proxy.Size = Vector3.new(0.2, 0.2, 0.2)
                proxy.Transparency = 1
                proxy.CastShadow = false
                for _, child in ipairs(part:GetChildren()) do
                    if isWingsFxInstance(child) then
                        local clone = child:Clone()
                        stripWingsTextGuis(clone)
                        clone.Parent = proxy
                    end
                end
                enableWingsFx(proxy)
                placeWingsProxy(proxy, originCF, part.CFrame)
                proxy.Parent = container
                equipped = true
            end
        else
            local p = part:Clone()
            stripWingsTextGuis(p)
            enableWingsFx(p)
            placeWingsProxy(p, originCF, part.CFrame)
            p.Parent = container
            equipped = true
        end
    end

    for _, child in ipairs(asset:GetChildren()) do
        if isWingsFxInstance(child) then
            local host = container:FindFirstChild("HumanoidRootPart_FX")
            if not host then
                host = Instance.new("Part")
                host.Name = "HumanoidRootPart_FX"
                host.Size = Vector3.new(0.2, 0.2, 0.2)
                host.Transparency = 1
                host.CastShadow = false
                placeWingsProxy(host, originCF, originCF)
                host.Parent = container
            end
            local clone = child:Clone()
            stripWingsTextGuis(clone)
            clone.Parent = host
            enableWingsFx(clone)
            equipped = true
        end
    end

    container.WorldPivot = CFrame.new()
    return container, equipped
end

local function ensureWingsAuraTemplate()
    local t = M._wingsAuraTemplate
    if t and t.Parent == nil and t:FindFirstChildWhichIsA("BasePart", true) then
        return t
    end
    local asset = loadWingsAsset()
    if asset then
        local container, equipped = buildWingsAuraContainer(asset)
        pcall(function() asset:Destroy() end)
        if equipped then
            container.Parent = nil
            M._wingsAuraTemplate = container
            return container
        end
        pcall(function() container:Destroy() end)
    end
    return nil
end

function M.applyWingsAura(char)
    if not M.wingsAuraEnabled then return end
    if M._wingsAuraApplying then return end
    M._wingsAuraApplying = true

    local okErr = pcall(function()
        M.removeWingsAura(false)
        if char then stripStrayWingsDummies(char) end

        local template = ensureWingsAuraTemplate()
        local model
        if template then
            model = template:Clone()
        else
            -- Asset failed — fallback particles (also cached as template).
            local fakeChar = char
            M._createFallbackWingsAura(fakeChar)
            model = M._wingsAuraModel
            if model then
                M._wingsAuraTemplate = model:Clone()
                M._wingsAuraTemplate.Parent = nil
            end
        end

        if not model then return end
        enableWingsFx(model)
        model.Parent = getWingsAuraParent()
        M._wingsAuraModel = model

        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        if hrp then
            pcall(function() model:PivotTo(hrp.CFrame) end)
        end
        startWingsAuraFollow()
    end)

    M._wingsAuraApplying = false
    if not okErr then
        -- ignore
    end
end

function M._createFallbackWingsAura(char)
    -- Beautiful particle-based wings aura as fallback
    local humanoidRP = char:FindFirstChild("HumanoidRootPart")
    if not humanoidRP then return end

    local wingsContainer = Instance.new("Part")
    wingsContainer.Name = "WingsAura"
    wingsContainer.Size = Vector3.new(1, 1, 1)
    wingsContainer.Transparency = 1
    wingsContainer.Anchored = false
    wingsContainer.CanCollide = false
    wingsContainer.Massless = true
    wingsContainer.Parent = char

    local weld = Instance.new("Weld")
    weld.Part0 = humanoidRP
    weld.Part1 = wingsContainer
    weld.C0 = CFrame.new(0, 0.5, 0.8)
    weld.Parent = wingsContainer

    -- Main wing glow particles
    local wingGlow = Instance.new("ParticleEmitter")
    wingGlow.Name = "WingGlow"
    wingGlow.Texture = "rbxassetid://6823291005"
    wingGlow.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(180, 140, 255)),
        ColorSequenceKeypoint.new(0.3, Color3.fromRGB(100, 200, 255)),
        ColorSequenceKeypoint.new(0.7, Color3.fromRGB(255, 180, 255)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(140, 100, 255)),
    })
    wingGlow.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.5),
        NumberSequenceKeypoint.new(0.3, 3.5),
        NumberSequenceKeypoint.new(0.7, 2.8),
        NumberSequenceKeypoint.new(1, 0),
    })
    wingGlow.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.6),
        NumberSequenceKeypoint.new(0.3, 0.2),
        NumberSequenceKeypoint.new(0.7, 0.4),
        NumberSequenceKeypoint.new(1, 1),
    })
    wingGlow.Lifetime = NumberRange.new(0.8, 1.5)
    wingGlow.Rate = 35
    wingGlow.Speed = NumberRange.new(1, 3)
    wingGlow.SpreadAngle = Vector2.new(45, 80)
    wingGlow.RotSpeed = NumberRange.new(-90, 90)
    wingGlow.Rotation = NumberRange.new(0, 360)
    wingGlow.LightEmission = 0.8
    wingGlow.LightInfluence = 0.2
    wingGlow.Parent = wingsContainer

    -- Sparkle trail particles
    local sparkles = Instance.new("ParticleEmitter")
    sparkles.Name = "WingSparks"
    sparkles.Texture = "rbxassetid://4727864990"
    sparkles.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 255, 255)),
        ColorSequenceKeypoint.new(0.5, Color3.fromRGB(200, 180, 255)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(150, 220, 255)),
    })
    sparkles.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.3),
        NumberSequenceKeypoint.new(0.5, 1.2),
        NumberSequenceKeypoint.new(1, 0),
    })
    sparkles.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.4),
        NumberSequenceKeypoint.new(0.5, 0.1),
        NumberSequenceKeypoint.new(1, 1),
    })
    sparkles.Lifetime = NumberRange.new(0.5, 1.2)
    sparkles.Rate = 20
    sparkles.Speed = NumberRange.new(2, 5)
    sparkles.SpreadAngle = Vector2.new(60, 90)
    sparkles.RotSpeed = NumberRange.new(-180, 180)
    sparkles.LightEmission = 1
    sparkles.LightInfluence = 0
    sparkles.Parent = wingsContainer

    -- Soft aura glow (point light)
    local light = Instance.new("PointLight")
    light.Name = "WingsLight"
    light.Color = Color3.fromRGB(160, 140, 255)
    light.Brightness = 2
    light.Range = 12
    light.Parent = wingsContainer

    M._wingsAuraModel = wingsContainer
end

function M.removeWingsAura()
    if M._wingsAuraModel then
        pcall(function() M._wingsAuraModel:Destroy() end)
        M._wingsAuraModel = nil
    end
    local char = player.Character
    if char then
        for _, child in ipairs(char:GetChildren()) do
            if child.Name == "WingsAura" or child:FindFirstChild("WingsAuraTag") then
                pcall(function() child:Destroy() end)
            elseif child:IsA("Model") and (child.Name == "Dummy" or (child:FindFirstChildOfClass("Humanoid") and child:FindFirstChild("Head"))) then
                -- Dummy that leaked into the character from the wings asset
                pcall(function() child:Destroy() end)
            end
        end
        stripStrayWingsDummies(char)
    end
end


-- ============================================================
-- STATE
-- ============================================================
M.NS = 60
M.CS = 30
M.LAGGER_SPEED = 15
M.LAGGER_CARRY_SPEED = 24.5
M.speedMethod = "VectorForce"
M.speedMethodList = {
    "Velocity", "AssemblyLinearVelocity", "Velocity Lerp", "AssemblyLinearVelocity Lerp",
    "CFrame", "CFrame Lerp", "Hyper CFrame", "Anchored CFrame", "PivotTo", "Model PivotTo", "Tween CFrame",
    "WalkSpeed", "Humanoid Move", "Humanoid MoveTo",
    "BodyVelocity", "BodyPosition", "BodyForce", "BodyThrust",
    "LinearVelocity", "VectorForce", "AlignPosition",
    "ApplyImpulse", "RocketPropulsion",
}
M.hyperMult = 4
M._lastSpeedMethod = nil
M._speedHRP = nil
M._anchoredBySpeed = nil
M._bodyVel = nil
M._bodyPosition = nil
M._bodyForce = nil
M._bodyThrust = nil
M._linearVel = nil
M._vectorForce = nil
M._alignPos = nil
M._rocket = nil
M._rocketTarget = nil
M._attLinVel = nil
M._attVecForce = nil
M._attAlign = nil
M._speedTween = nil
M.carrySpeedActive = false
M.laggerModeEnabled = false
M.laggerCarryActive = false
-- Estado salvo para a keybind temporária do Lagger.
M._laggerPreviousMode = nil

M.antiRagdollEnabled = false
M.antiRagdollMode = "Splatter"
M.infJumpEnabled = false
M.infJumpMode = "manual"
M.medusaCounterEnabled = false
M.batCounterEnabled = false
M.unwalkEnabled = false
M.medusaResetEnabled = false
M.medusaDebounce = false
M.medusaLastUsed = 0
M.dropActive = false
M.autoLeftEnabled = false
M.autoRightEnabled = false
M.autoPlayMode = "Semi"
M.autoBatEnabled = false
M.autoBatMode = "Normal"
M.autoSwingEnabled = true
M.autoMoveSwingEnabled = false
M.autoMoveSwingInterval = 0.3
M._alSwingDebounce = false
M._arSwingDebounce = false
    M.antiLagEnabled = false
    M.nukeOptimizerEnabled = false
    M.antiSummerBaseEnabled = false
M.antiSummerBaseConn = nil
M._antiSummerCleaned = {}

M.removeAccessoriesEnabled = false
M.antiLagDescConn = nil
M.stretchRezEnabled = false
M.stretchRezConn = nil
M.unwalkSavedAnimate = nil
M._anyKeyListening = false
M.autoTPEnabled = false
M.autoTPHeight = 20
M.autoTPConn = nil
M.cursedResetRemote = nil
M.CURSED_RESET_GUID = "f888ee6e-c86d-46e1-93d7-0639d6635d42"
M.guiTransparencyEnabled = false
M.mobileButtonsEnabled = true
M.mobileButtonsLocked = false
M.mobileButtonsSize = 56

M.circleButtonsEnabled = false
M.mobBtnRefs = {}
M.mobGuiRef = nil
M.fovValue = 80
M.fovOptions = {80,120,180}
M.fovIndex = 1
M.laggerModePillRef = nil
M.carryModePillRef = nil
M.autoSwitchSpeedEnabled = false
M.autoTurnOffSpeedEnabled = false
M.autoSwitchLaggerSpeedEnabled = false
M.AUTO_SWITCH_THRESHOLD = 25
M._autoSwitchSpeedConn = nil
M.customFontSelected = "None"
M._fontOrig = {}
M._fontConn = nil
M._fontMy = nil
M.FONT_NAMES = {"None", "Coding Font", "Summer", "Beachy", "Scary", "Bangers"}
M.mobBtnTransparencyEnabled = false
M.perButtonDragEnabled = true
M.antiKickEnabled = false
M.brainrotDetected = false
M.safeModeEnabled = false
M.mirrorTPDownEnabled = false
M.mirrorTPPreviousY = {}
M.mirrorTPLastTeleport = 0
M.MIRROR_TP_DROP_THRESHOLD = 3
M.MIRROR_TP_DOWN_Y = -7.00
M.activeBatBillboard = nil
M.activeMedusaBillboard = nil
M.ragdollGuiEnabled = true
M.persistentRagdollGui = nil
M.uiLocked = false
M.holdInfJumpConn = nil
M.DROP_ASCEND_DURATION = 0.2
M.DROP_ASCEND_SPEED = 150
M.autoResetOnDeath = false
M.bypassAimbotEnabled = false
M.bypassAimbotConn = nil
M._bypassGodConn = nil
M._bypassGodHealthConn = nil
M._bypassGodDiedConn = nil
M._bypassGodCharConn = nil
M.bypassPrevAutoRotate = nil
M.bypassHitCD = false
M.bypassSwingCD = 0.35
M.bypassHitDist = 8
M._bypassTarget = nil

M.stealMode = "V1"
M.stealBarSize = 300
M.stealBarScale = 0.3

function M.setAutoGrabGuiScale(scale)
    M.stealBarScale = math.clamp(tonumber(scale) or 0.3, 0.1, 2)
    if M.stealBarScaleRef then
        M.stealBarScaleRef.Scale = M.stealBarScale
    end
end

function M.adjustAutoGrabGuiScale(delta)
    M.setAutoGrabGuiScale((M.stealBarScale or 0.3) + (tonumber(delta) or 0.1))
end
M.Steal = {
    AutoStealEnabled = false,
    StealRadius = 60,
    StealDuration = 1.4,
    StopTime = 0.35,
}
M.V3 = {
    enabled = false,
    conn = nil,
    progress = 0,
    lastInRange = 0,
    currentUid = nil,
    holding = false,
    holdPrompt = nil,
    cooldownUntil = 0,
}
M.autoRadiusEnabled = false
M.V2 = M.V2 or {}
M.V2.radius = tonumber(M.V2.radius) or 60
function M.getAutoRadius()
    local radius = math.clamp((tonumber(M.NS) or 60) + 1, 1, 500)
    return math.floor(radius * 10 + 0.5) / 10
end
function M.getActiveStealRadius()
    if M.stealMode == "V2" then
        return math.clamp(tonumber(M.V2.radius) or 60, 1, 300)
    elseif M.stealMode == "Semi" or M.stealMode == "V4" then
        return math.min(tonumber(M.Semi.radius) or 10, 10)
    end
    return M.autoRadiusEnabled and M.getAutoRadius() or M.Steal.StealRadius
end
M.Semi = {
    enabled = false,
    holdMin = 1.3,
    holdMax = 2.6,
    entryDelay = 0.3,
    cooldown = 0.05,
    primeRange = 80,
    radius = 10, -- STEAL_RANGE from auto-grabber
    conn = nil,
    scanThread = nil,
    plotSync = {caches = {}, connections = {}},
    animals = {},
    promptCache = {},
    internalCache = {},
    state = {active = false, startTime = 0, phase = "idle", label = "", lastResult = "", lastResultTime = 0},
    plots = nil,
    syncReady = false,
}
M.isStealing = false
M.stealStartTime = 0
M.stealConn = nil
M.progressConn = nil
M.animalCache = {}
M.promptCache = {}
M.stealCache = {}
M.pingPopupActive = false
M.pingPopupGui = nil
M.pingCycleTimer = nil
M.Conns = {autoSteal=nil, antiRag=nil, batCounter=nil, anchor={}}
M._persistentConns = {}
M.alConn = nil
M.arConn = nil
M.alPhase = 1
M.arPhase = 1
M.aimbotConn = nil
M.lastMoveDir = Vector3.new(0,0,0)
M.batCounterDebounce = false
M.speedLabel = nil

-- Keybinds
M.KB = {
    DropBrainrot={kb=nil,gp=nil},
    AutoLeft={kb=nil,gp=nil},
    AutoRight={kb=nil,gp=nil},
    AutoBat={kb=nil,gp=nil},
    TPFloor={kb=nil,gp=nil},
    InstaReset={kb=nil,gp=nil},
    GuiHide={kb=nil,gp=nil},
    SpeedToggle={kb=nil,gp=nil},
    LaggerToggle={kb=Enum.KeyCode.R,gp=nil},
    BypassAimbot={kb=nil,gp=nil},
}

M.AP_L1 = Vector3.new(-476.47,-6.28,92.73)
M.AP_L2 = Vector3.new(-483.12,-4.95,94.81)
M.AP_R1 = Vector3.new(-476.16,-6.52,25.62)
M.AP_R2 = Vector3.new(-483.06,-5.03,25.48)
M._fullAutoLeftWaypoints = {
    Vector3.new(-475.69, -5.85, 92.99),
    Vector3.new(-487.27, -3.48, 93.83),
    Vector3.new(-475.69, -5.85, 92.99),
    Vector3.new(-477.26, -4.88, 25.90),
    Vector3.new(-486.03, -4.71, 17.52),
}
M._fullAutoRightWaypoints = {
    Vector3.new(-475.87, -5.60, 27.50),
    Vector3.new(-486.71, -3.68, 28.24),
    Vector3.new(-475.87, -5.60, 27.50),
    Vector3.new(-476.83, -5.17, 93.00),
    Vector3.new(-485.60, -3.53, 95.87),
}
M.MEDUSA_COOLDOWN = 25
M.BAT_COUNTER_SLAP_LIST = {"Bat","Slap","Iron Slap","Gold Slap","Diamond Slap","Emerald Slap","Ruby Slap","Dark Matter Slap","Flame Slap","Nuclear Slap","Galaxy Slap","Glitched Slap"}
M.fovConn = nil
M.defLightBrightness = nil
M.defLightClock = nil
M.defLightAmbient = nil
M.mainFrame = nil
M.normalBox = nil
M.carryBox = nil
M.laggerBox = nil
M.radInput = nil
M.autoTPHeightBox = nil
M.durationBox = nil
M.modeValLbl = nil
M.setInstaGrab = nil
M.setInfJumpVisual = nil
M.setAntiRagVisual = nil
M.setMedusaVisual = nil
M.setUnwalkVisual = nil
M.setAntiLagVisual = nil
M.setAutoSwingVisual = nil
M.setTranspVisual = nil
M.setLockVisual = nil
M.setMobVisual = nil
M.setCircleBtnsVisual = nil
M.setMedusaResetVisual = nil
M.antiKickSetVisual = nil
M.autoLeftSetVisual = nil
M.autoRightSetVisual = nil
M.autoBatSetVisual = nil
M.setAutoTPVisual = nil
M.setStretchRezVisual = nil
M.setAutoResetOnDeath = nil
M.setBypassVisual = nil
M._autoSwitchWasSteal = false

M.MOB_POS_FILE = "moveeduels_btnpos.json"
M.MOVE_KEYS = {
    [Enum.KeyCode.W]=true,
    [Enum.KeyCode.A]=true,
    [Enum.KeyCode.S]=true,
    [Enum.KeyCode.D]=true,
    [Enum.KeyCode.Up]=true,
    [Enum.KeyCode.Left]=true,
    [Enum.KeyCode.Down]=true,
    [Enum.KeyCode.Right]=true
}

M.removeAccEnabled = false
M.removeAccConn = nil
M.removedAccessories = {}
M.uiScale = 0.5
if UIS.TouchEnabled and not UIS.KeyboardEnabled then
    M.uiScale = 0.5
end
M.uiScaleSliderRef = nil
M.uiScaleLabelRef = nil
M.uiScaleBoxRef = nil
M.menuOpen = true

M.statusGui = nil
M.statusFill = nil
M.statusPctLbl = nil
M.statusRadiusLbl = nil
M.statusDot = nil
M.statusMain = nil
M.statusFpsLbl = nil

-- ============================================================
-- UTILITY FUNCTIONS
-- ============================================================
function M.addShimmerToLabel(lbl,color1,color2)
    local gr=Instance.new("UIGradient",lbl)
    gr.Color=ColorSequence.new({ColorSequenceKeypoint.new(0,color1 or Color3.fromRGB(100,100,100)),ColorSequenceKeypoint.new(0.5,color2 or Color3.fromRGB(255,255,255)),ColorSequenceKeypoint.new(1,color1 or Color3.fromRGB(100,100,100))})
    gr.Transparency=NumberSequence.new({NumberSequenceKeypoint.new(0,0.3,0),NumberSequenceKeypoint.new(0.5,0,0),NumberSequenceKeypoint.new(1,0.3,0)})
    return gr
end

function M.applyFOV()
    if M.fovConn then M.fovConn:Disconnect() end
    M.fovConn=RunService.RenderStepped:Connect(function() local cam=workspace.CurrentCamera;if cam then cam.FieldOfView=M.fovValue end end)
end

-- ============================================================
-- RAGDOLL TIMER
-- ============================================================
M.ragdollTimerThread = nil
M.ragdollTimerRemaining = 0
M.isRagdollActive = false

function M.updateRagdollTimer(duration)
    if M.ragdollTimerThread then
        task.cancel(M.ragdollTimerThread)
        M.ragdollTimerThread = nil
    end
    if duration <= 0 then
        M.isRagdollActive = false
        if M.headIndicator and M.headIndicator.ragdollTimer then
            M.headIndicator.ragdollTimer.Text = ""
        end
        return
    end
    M.isRagdollActive = true
    local startTime = tick()
    M.ragdollTimerRemaining = duration
    M.ragdollTimerThread = task.spawn(function()
        while M.isRagdollActive and M.ragdollTimerRemaining > 0 do
            local elapsed = tick() - startTime
            local remaining = math.max(0, duration - elapsed)
            M.ragdollTimerRemaining = remaining
            if M.headIndicator and M.headIndicator.ragdollTimer then
                M.headIndicator.ragdollTimer.Text = string.format("%.1fs", remaining)
            end
            if remaining <= 0 then
                M.isRagdollActive = false
                if M.headIndicator and M.headIndicator.ragdollTimer then
                    M.headIndicator.ragdollTimer.Text = ""
                end
                break
            end
            task.wait(0.05)
        end
        M.ragdollTimerThread = nil
    end)
end

function M.onHumanoidStateChanged(old,new)
    local char=player.Character;if not char then return end
    local hum=char:FindFirstChildOfClass("Humanoid");if not hum then return end
    local isRag=(new==Enum.HumanoidStateType.Physics or new==Enum.HumanoidStateType.Ragdoll or new==Enum.HumanoidStateType.FallingDown)
    if isRag and not hum.PlatformStand then
        M.updateRagdollTimer(2.6)
    end
end

function M.onMedusaStateChanged()
    local char=player.Character;if not char then return end
    local hum=char:FindFirstChildOfClass("Humanoid")
    if hum and hum.PlatformStand then
        M.updateRagdollTimer(4.5)
    end
end

function M.setupRagdollTriggers()
    local char=player.Character;if not char then return end
    local hum=char:FindFirstChildOfClass("Humanoid")
    if M._ragdollStateConn then M._ragdollStateConn:Disconnect(); M._ragdollStateConn=nil end
    if M._ragdollPlatformConn then M._ragdollPlatformConn:Disconnect(); M._ragdollPlatformConn=nil end
    if hum then
        M._ragdollStateConn = hum.StateChanged:Connect(M.onHumanoidStateChanged)
        M._ragdollPlatformConn = hum:GetPropertyChangedSignal("PlatformStand"):Connect(M.onMedusaStateChanged)
    end
end

-- ============================================================
-- ANIMATION FUNCTIONS (unchanged)
-- ============================================================
function M.waitForAnimate(char)
    for _ = 1, 40 do
        local a = char:FindFirstChild("Animate")
        if a and a:FindFirstChild("idle") and a:FindFirstChild("run") and a:FindFirstChild("walk") then
            return a
        end
        task.wait(0.1)
    end
    return nil
end

function M.setAnim(animObj, id)
    if animObj and id then
        animObj.AnimationId = "rbxassetid://" .. tostring(id)
    end
end

function M.stopAllTracks(hum)
    if not hum then return end
    for _, t in ipairs(hum:GetPlayingAnimationTracks()) do
        pcall(function() t:Stop(0) end)
    end
end

function M.ensureAnim(folder, name)
    if not folder then return nil end
    local a = folder:FindFirstChild(name)
    if not a then
        a = Instance.new("Animation")
        a.Name = name
        a.Parent = folder
    end
    return a
end

function M.ensureIdleSlots(idleFolder, n)
    if not idleFolder then return end
    n = n or 2
    for i=1,n do
        M.ensureAnim(idleFolder, "Animation" .. i)
    end
end

function M.pick(pack, ...)
    for i = 1, select("#", ...) do
        local k = select(i, ...)
        local v = pack[k]
        if v ~= nil then return v end
    end
    return nil
end

function M.saveOriginalAnimate(char)
    if not char then return end
    if M.savedAnimate then return end
    local animate = char:FindFirstChild("Animate")
    if animate then
        M.savedAnimate = animate:Clone()
    end
end

function M.restoreOriginalAnimate(char)
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if hum then
        M.stopAllTracks(hum)
    end
    local currentAnimate = char:FindFirstChild("Animate")
    if currentAnimate then
        currentAnimate:Destroy()
    end
    if M.savedAnimate then
        local newAnimate = M.savedAnimate:Clone()
        newAnimate.Parent = char
        newAnimate.Disabled = true
        task.wait(0.06)
        newAnimate.Disabled = false
    end
end

function M.resetAnimations(char)
    if not char then return end
    M.restoreOriginalAnimate(char)
end

local applyingAnim = false
function M.applyAnimPack(packName)
    if not M.animPackEnabled then
        local char = player.Character
        if char then
            M.resetAnimations(char)
        end
        return false
    end
    if applyingAnim then return false end
    applyingAnim = true

    local pack = M.PACKS[packName]
    if not pack then
        applyingAnim = false
        return false
    end

    local char = player.Character or player.CharacterAdded:Wait()
    M.saveOriginalAnimate(char)

    local animate = M.waitForAnimate(char)
    if not animate then
        applyingAnim = false
        return false
    end

    local hum = char:FindFirstChildOfClass("Humanoid")
    M.stopAllTracks(hum)

    local runObj   = M.ensureAnim(animate:FindFirstChild("run"),   "RunAnim")
    local walkObj  = M.ensureAnim(animate:FindFirstChild("walk"),  "WalkAnim")
    local jumpObj  = M.ensureAnim(animate:FindFirstChild("jump"),  "JumpAnim")
    local fallObj  = M.ensureAnim(animate:FindFirstChild("fall"),  "FallAnim")
    local climbObj = M.ensureAnim(animate:FindFirstChild("climb"), "ClimbAnim")
    local swimObj  = M.ensureAnim(animate:FindFirstChild("swim"),     "Swim")
    local swimIdleObj = M.ensureAnim(animate:FindFirstChild("swimidle"), "SwimIdle")
    local idleFolder = animate:FindFirstChild("idle")

    M.setAnim(walkObj,  M.pick(pack, "WalkAnim", "Walk"))
    M.setAnim(runObj,   M.pick(pack, "RunAnim", "Run"))
    M.setAnim(jumpObj,  M.pick(pack, "JumpAnim", "Jump"))
    M.setAnim(fallObj,  M.pick(pack, "FallAnim", "Fall"))
    M.setAnim(climbObj, M.pick(pack, "ClimbAnim", "Climb"))
    M.setAnim(swimObj,      M.pick(pack, "Swim"))
    M.setAnim(swimIdleObj,  M.pick(pack, "SwimIdle") or M.pick(pack, "Swim"))

    if idleFolder then
        local a1 = M.pick(pack, "Animation1")
        local a2 = M.pick(pack, "Animation2")
        if a1 or a2 then
            M.ensureIdleSlots(idleFolder, 2)
            local id1 = a1 or a2
            local id2 = a2 or a1 or id1
            M.setAnim(idleFolder:FindFirstChild("Animation1"), id1)
            M.setAnim(idleFolder:FindFirstChild("Animation2"), id2)
        elseif pack.Idle and #pack.Idle > 0 then
            M.ensureIdleSlots(idleFolder, math.max(2, #pack.Idle))
            M.setAnim(idleFolder:FindFirstChild("Animation1"), pack.Idle[1])
            M.setAnim(idleFolder:FindFirstChild("Animation2"), pack.Idle[2] or pack.Idle[1])
            for i = 3, #pack.Idle do
                local a = idleFolder:FindFirstChild("Animation" .. i)
                if a then M.setAnim(a, pack.Idle[i]) end
            end
        end
    end

    animate.Disabled = true
    task.wait(0.06)
    animate.Disabled = false

    if hum then
        pcall(function()
            hum:ChangeState(Enum.HumanoidStateType.Landed)
            task.wait(0.03)
            hum:ChangeState(Enum.HumanoidStateType.Running)
        end)
    end

    M.animPack = packName
    applyingAnim = false
    return true
end

-- ============================================================
-- OVER-HEAD INDICATOR
-- ============================================================
M.headIndicator = nil

function M.setupHeadIndicator(char)
    local head=char:WaitForChild("Head",5);if not head then return end
    if head:FindFirstChild("MoveeHeadIndicator") then head.MoveeHeadIndicator:Destroy() end
    local bb=Instance.new("BillboardGui",head)
    bb.Name="MoveeHeadIndicator"
    bb.Size=UDim2.new(0,250,0,58)
    bb.StudsOffset=Vector3.new(0,3.5,0)
    bb.AlwaysOnTop=true
    bb.Parent=head

    local accent = CHERRY_ACCENT or Color3.fromRGB(255,255,255)

    local ragdollLbl=Instance.new("TextLabel",bb)
    ragdollLbl.Name="RagdollTimer"
    ragdollLbl.Size=UDim2.new(1,0,0.5,0)
    ragdollLbl.Position=UDim2.new(0,0,0,0)
    ragdollLbl.BackgroundTransparency=1
    ragdollLbl.Text=""
    ragdollLbl.TextColor3=accent
    ragdollLbl.Font=Enum.Font.GothamBold
    ragdollLbl.TextScaled=true
    ragdollLbl.TextStrokeTransparency=0

    local discordLbl=Instance.new("TextLabel",bb)
    discordLbl.Name="Discord"
    discordLbl.Size=UDim2.new(1,0,0.5,0)
    discordLbl.Position=UDim2.new(0,0,0.5,0)
    discordLbl.BackgroundTransparency=1
    discordLbl.Text="ONI HUB"
    discordLbl.TextColor3=accent
    discordLbl.Font=Enum.Font.GothamBold
    discordLbl.TextScaled=true
    discordLbl.TextStrokeTransparency=0

    M.headIndicator = {bb=bb, discord=discordLbl, ragdollTimer=ragdollLbl}
    M.updateHeadTheme()
end

function M.updateHeadTheme()
    if not M.headIndicator then return end
    local accent = UI_ACCENT or CHERRY_ACCENT or Color3.fromRGB(255,255,255)
    if M.headIndicator.discord then
        M.headIndicator.discord.TextColor3 = accent
    end
    if M.headIndicator.ragdollTimer then
        M.headIndicator.ragdollTimer.TextColor3 = accent
    end
end

-- STEAL STATUS UI (Steal Bar)
-- ============================================================
function M.buildStatusUI()
    if M.statusGui then
        pcall(function() M.statusGui:Destroy() end)
        M.statusGui = nil
    end

    local gui = Instance.new("ScreenGui")
    gui.Name = "K7_StatusUI"
    gui.ResetOnSpawn = false
    gui.IgnoreGuiInset = true
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

    do
        local ok = false
        if gethui then
            ok = pcall(function() gui.Parent = gethui() end)
        elseif syn and syn.protect_gui then
            ok = pcall(function()
                syn.protect_gui(gui)
                gui.Parent = game:GetService("CoreGui")
            end)
        end
        if not ok then
            gui.Parent = player:WaitForChild("PlayerGui")
        end
    end

    for _, v in ipairs(gui.Parent:GetChildren()) do
        if v ~= gui and v:IsA("ScreenGui") and v.Name == gui.Name then
            v:Destroy()
        end
    end

    local C = {
        bg    = UI_BG_DARK or Color3.fromRGB(10, 10, 10),
        stroke = UI_ACCENT or Color3.fromRGB(255, 0, 0),
        white = UI_TEXT_WHITE or Color3.fromRGB(255, 255, 255),
        grey  = UI_TEXT_DIM or Color3.fromRGB(170, 170, 170),
        track = UI_BTN_BG or Color3.fromRGB(42, 42, 42),
        dotBg = UI_BTN_BG or Color3.fromRGB(85, 85, 85),
    }
    local ITALIC = Font.new("rbxasset://fonts/families/GothamSSm.json", Enum.FontWeight.Bold, Enum.FontStyle.Italic)

    local ACCENT = UI_ACCENT or Color3.fromRGB(255, 0, 0)
    local BAR_H = 37
    -- fixed internal layout width; the whole bar is scaled by M.stealBarScale
    local width = 420
    local main = Instance.new("Frame")
    main.Size = UDim2.fromOffset(width, BAR_H)
    -- restore saved position if present
    local savedBar = (M.loadBtnPositions and M.loadBtnPositions()) or {}
    if savedBar["_stealbar"] then
        local sp = savedBar["_stealbar"]
        main.AnchorPoint = Vector2.new(0, 0)
        main.Position = UDim2.new(sp.xs or 0, sp.xo or 0, sp.ys or 0, sp.yo or 0)
    else
        main.AnchorPoint = Vector2.new(0.5, 1)
        main.Position = UDim2.new(0.5, 0, 1, -80)
    end
    main.BackgroundColor3 = UI_BG_DARK or Color3.fromRGB(16, 16, 20)
    main.BorderSizePixel = 0
    main.Active = true
    main.Parent = gui
    Instance.new("UICorner", main).CornerRadius = UDim.new(1, 0)
    local mStroke = Instance.new("UIStroke", main)
    mStroke.Color = ACCENT
    mStroke.Thickness = 2.5
    mStroke.Transparency = 0
    -- scales the ENTIRE steal bar (text, pills, dot, height) together
    local barScale = Instance.new("UIScale")
    barScale.Scale = M.stealBarScale or 1
    barScale.Parent = main
    M.stealBarScaleRef = barScale

    -- layout cursor (left -> right), with fixed gaps so nothing overlaps
    local PAD = 5
    local GAP = 8
    local x = PAD

    -- fixed-size right-side elements; STEAL pill takes the rest so it stays long
    -- (% now lives INSIDE the steal pill, so it needs no separate slot)
    local FPS_W, PING_W = 78, 90
    local DOT_W  = 28
    -- width consumed by everything except the STEAL pill (with gaps)
    local rightUsed = FPS_W + GAP + PING_W + GAP + DOT_W + PAD
    local STEAL_W = width - PAD - GAP - rightUsed
    if STEAL_W < 110 then
        STEAL_W = 110
        width = PAD + STEAL_W + GAP + rightUsed
        main.Size = UDim2.fromOffset(width, BAR_H)
    end

    -- ===== STEAL pill (left): STEAL text + %, with progress fill =====
    local PILL_RADIUS = math.floor((BAR_H - 8) / 2)
    local stealPill = Instance.new("Frame")
    stealPill.Size = UDim2.new(0, STEAL_W, 1, -8)
    stealPill.Position = UDim2.fromOffset(x, 4)
    stealPill.BackgroundColor3 = UI_ROW_BG or Color3.fromRGB(24, 24, 28)
    stealPill.BorderSizePixel = 0
    stealPill.ClipsDescendants = true
    stealPill.Parent = main
    Instance.new("UICorner", stealPill).CornerRadius = UDim.new(0, PILL_RADIUS)

    -- solid fill with a fixed-radius rounded cap so the leading edge is round
    -- (moves like a capsule, not a square). Fixed pixel radius stays visible
    -- even at tiny widths, unlike a scale-based corner.
    local fill = Instance.new("Frame")
    fill.Size = UDim2.new(0, 0, 1, 0)
    fill.Position = UDim2.fromOffset(0, 0)
    fill.BackgroundColor3 = ACCENT
    fill.BorderSizePixel = 0
    fill.ZIndex = 2
    fill.Parent = stealPill
    Instance.new("UICorner", fill).CornerRadius = UDim.new(0, PILL_RADIUS)
    M.statusFill = fill
    M._stealPillW = STEAL_W

    local stealTxt = Instance.new("TextLabel")
    stealTxt.BackgroundTransparency = 1
    stealTxt.Position = UDim2.fromOffset(16, 0)
    stealTxt.Size = UDim2.new(0, 90, 1, 0)
    stealTxt.Font = Enum.Font.GothamBlack
    stealTxt.TextSize = 15
    stealTxt.TextColor3 = Color3.fromRGB(255,255,255)
    stealTxt.TextXAlignment = Enum.TextXAlignment.Left
    stealTxt.Text = "STEAL"
    stealTxt.ZIndex = 4
    stealTxt.Parent = stealPill

    -- percentage INSIDE the steal pill, aligned to its right side
    local pctLbl = Instance.new("TextLabel")
    pctLbl.BackgroundTransparency = 1
    pctLbl.AnchorPoint = Vector2.new(1, 0)
    pctLbl.Position = UDim2.new(1, -16, 0, 0)
    pctLbl.Size = UDim2.fromOffset(60, BAR_H - 8)
    pctLbl.Font = Enum.Font.GothamBlack
    pctLbl.TextSize = 15
    pctLbl.TextColor3 = Color3.fromRGB(255,255,255)
    pctLbl.TextXAlignment = Enum.TextXAlignment.Right
    pctLbl.Text = "0%"
    pctLbl.ZIndex = 4
    pctLbl.Parent = stealPill
    M.statusPctLbl = pctLbl

    x = x + STEAL_W + GAP

    -- ===== helper: outlined pill placed at absolute x =====
    local function makeInfoPill(px, w, labelText, valueText)
        local pill = Instance.new("Frame")
        pill.Size = UDim2.new(0, w, 1, -14)
        pill.Position = UDim2.new(0, px, 0.5, 0)
        pill.AnchorPoint = Vector2.new(0, 0.5)
        pill.BackgroundColor3 = UI_ROW_BG or Color3.fromRGB(24, 24, 28)
        pill.BackgroundTransparency = 0.15
        pill.BorderSizePixel = 0
        pill.Parent = main
        Instance.new("UICorner", pill).CornerRadius = UDim.new(1, 0)
        local ps = Instance.new("UIStroke", pill)
        ps.Color = ACCENT
        ps.Thickness = 2
        ps.Transparency = 0
        local lbl = Instance.new("TextLabel", pill)
        lbl.BackgroundTransparency = 1
        lbl.Size = UDim2.new(1, -12, 1, 0)
        lbl.Position = UDim2.fromOffset(8, 0)
        lbl.Font = Enum.Font.GothamBold
        lbl.TextSize = 12
        lbl.TextXAlignment = Enum.TextXAlignment.Center
        lbl.RichText = true
        lbl.Text = ("<font color=\"#ff6b6b\">%s</font> <b>%s</b>"):format(labelText, valueText)
        lbl.TextColor3 = Color3.fromRGB(255,255,255)
        return lbl
    end

    -- FPS pill then PING pill, laid out sequentially
    local fpsPill  = makeInfoPill(x, FPS_W, "FPS", "0")
    x = x + FPS_W + GAP
    local pingPill = makeInfoPill(x, PING_W, "PING", "0 ms")
    x = x + PING_W + GAP

    local infoLbl = { fps = fpsPill, ping = pingPill }
    M.statusFpsLbl = infoLbl

    -- ===== right-end dot =====
    local dotBg = Instance.new("Frame")
    dotBg.Size = UDim2.fromOffset(28, 28)
    dotBg.Position = UDim2.new(1, -PAD, 0.5, 0)
    dotBg.AnchorPoint = Vector2.new(1, 0.5)
    dotBg.BackgroundColor3 = UI_ROW_BG or Color3.fromRGB(24, 24, 28)
    dotBg.BorderSizePixel = 0
    dotBg.Parent = main
    Instance.new("UICorner", dotBg).CornerRadius = UDim.new(1, 0)
    local dbStroke = Instance.new("UIStroke", dotBg)
    dbStroke.Color = ACCENT
    dbStroke.Thickness = 2
    dbStroke.Transparency = 0
    local dot = Instance.new("Frame")
    dot.Size = UDim2.fromOffset(15, 15)
    dot.AnchorPoint = Vector2.new(0.5, 0.5)
    dot.Position = UDim2.fromScale(0.5, 0.5)
    dot.BackgroundColor3 = ACCENT
    dot.BorderSizePixel = 0
    dot.Parent = dotBg
    Instance.new("UICorner", dot).CornerRadius = UDim.new(1, 0)
    M.statusDot = dot

    

    -- FPS / PING updater
    local frameCount = 0
    local timeAccum = 0
    local fps = 0
    if M._statusFpsConn then pcall(function() M._statusFpsConn:Disconnect() end) M._statusFpsConn=nil end
    M._statusFpsConn = RunService.RenderStepped:Connect(function(delta)
        frameCount = frameCount + 1
        timeAccum = timeAccum + delta
        if timeAccum >= 0.5 then
            fps = math.floor(frameCount / timeAccum + 0.5)
            frameCount = 0
            timeAccum = 0
            local ping = 0
            pcall(function() ping = math.floor(player:GetNetworkPing() * 1000 + 0.5) end)
            if fpsPill and fpsPill.Parent then
                fpsPill.Text = ("<font color=\"#ff6b6b\">FPS</font> <b>%d</b>"):format(fps)
            end
            if pingPill and pingPill.Parent then
                pingPill.Text = ("<font color=\"#ff6b6b\">PING</font> <b>%d ms</b>"):format(ping)
            end
        end
    end)

    -- drag + save position
    local function saveStealBarPos()
        if type(writefile) ~= "function" then return end
        local cur = (M.loadBtnPositions and M.loadBtnPositions()) or {}
        cur["_stealbar"] = {xs=main.Position.X.Scale, xo=main.Position.X.Offset,
                             ys=main.Position.Y.Scale, yo=main.Position.Y.Offset}
        pcall(function() writefile(M.MOB_POS_FILE, HS:JSONEncode(cur)) end)
    end
    do
        local dragging, dragStart, startPos
        main.InputBegan:Connect(function(input)
            if M.uiLocked then return end
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                -- normalize anchor to top-left so dragging math is consistent
                if main.AnchorPoint ~= Vector2.new(0,0) then
                    local ap = main.AbsolutePosition
                    main.AnchorPoint = Vector2.new(0,0)
                    main.Position = UDim2.fromOffset(ap.X, ap.Y)
                end
                dragging = true
                dragStart = input.Position
                startPos = main.Position
                input.Changed:Connect(function()
                    if input.UserInputState == Enum.UserInputState.End then
                        dragging = false
                        saveStealBarPos()
                    end
                end)
            end
        end)
        UIS.InputChanged:Connect(function(input)
            if dragging and M.uiLocked then
                dragging = false
                return
            end
            if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
                local d = input.Position - dragStart
                main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
            end
        end)
    end

    M.statusGui = gui
    M.statusMain = main
    M.statusHolder = main
    M.statusBarPctLbl = pctLbl
    M.statusStealLbl = stealTxt
    M.statusRadiusLbl = nil
end

function M.updateStealProgress(progress, label)
    progress = math.clamp(progress or 0, 0, 1)
    local pct = math.floor(progress * 100 + 0.5)
    if M.statusFill then
        M.statusFill.Size = UDim2.new(progress, 0, 1, 0)
        -- Use the active menu accent for the entire Auto Grab progress bar.
        local col
        if progress < 0.5 then
            col = UI_ACCENT or Color3.fromRGB(255, 255, 255)
        elseif progress < 0.75 then
            col = UI_ACCENT or Color3.fromRGB(255, 230, 40)
        else
            col = UI_ACCENT or Color3.fromRGB(255, 0, 0)
        end
        M.statusFill.BackgroundColor3 = col
        local grad = M.statusFill:FindFirstChildOfClass("UIGradient")
        if grad then
            local themeAccent = UI_ACCENT or Color3.fromRGB(255, 255, 255)
            grad.Color = ColorSequence.new({
                ColorSequenceKeypoint.new(0, themeAccent),
                ColorSequenceKeypoint.new(1, themeAccent),
            })
        end
    end
    if M.statusKnob then
        M.statusKnob.Visible = progress > 0.02
    end
    local text
    if type(label) == "string" and label ~= "" and not tostring(label):match("^%d") then
        text = string.format("%d%%", pct)
    else
        text = string.format("%d%%", pct)
    end
    if M.statusBarPctLbl then M.statusBarPctLbl.Text = text end
    if M.statusPctLbl and M.statusPctLbl ~= M.statusBarPctLbl then
        M.statusPctLbl.Text = text
    end
    if M.statusStealLbl and M.statusStealLbl ~= M.statusBarPctLbl then
        M.statusStealLbl.Text = text
    end
    if progress > 0.02 and progress < 1 then
        if not M._stealShineActive and M.statusShine then
            M._stealShineActive = true
            M.statusShine.BackgroundTransparency = 0.55
            local function loopShine()
                if not M._stealShineActive or not M.statusShine or not M.statusShine.Parent then
                    M._stealShineActive = false
                    return
                end
                M.statusShine.Position = UDim2.new(-0.45, 0, 0, 0)
                local tw = TweenService:Create(M.statusShine, TweenInfo.new(0.85, Enum.EasingStyle.Linear), {
                    Position = UDim2.new(1.1, 0, 0, 0)
                })
                M._stealShineTween = tw
                tw:Play()
                tw.Completed:Connect(function()
                    if M._stealShineActive then
                        task.delay(0.15, loopShine)
                    end
                end)
            end
            loopShine()
        end
    else
        M._stealShineActive = false
        if M._stealShineTween then pcall(function() M._stealShineTween:Cancel() end) end
        if M.statusShine then
            M.statusShine.BackgroundTransparency = 1
            M.statusShine.Position = UDim2.new(-0.4, 0, 0, 0)
        end
        if progress <= 0 and M.statusKnob then
            M.statusKnob.Visible = false
        end
    end
end


function M.updateStatusRadius()
    if M.statusRadiusLbl then
        M.statusRadiusLbl.Text = "Radius: " .. tostring(M.getActiveStealRadius())
    end
    if M.headerRadiusLbl then
        M.headerRadiusLbl.Text = tostring(M.getActiveStealRadius())
    end
    if M.updateRadiusMarker then
        M.updateRadiusMarker()
    end
end

-- ============================================================

-- AUTO STEAL (unchanged)
-- ============================================================
if not fireproximityprompt then
    fireproximityprompt = (getgenv and getgenv().fireproximityprompt)
        or (genv and genv().fireproximityprompt)
        or function(prompt)
            pcall(function()
                prompt:InputHoldBegin()
                task.wait(0.05)
                prompt:InputHoldEnd()
            end)
        end
end

local function isMyPlot(plotName)
    local plots = workspace:FindFirstChild("Plots")
    if not plots then return false end
    local plot = plots:FindFirstChild(plotName)
    if not plot then return false end
    local sign = plot:FindFirstChild("PlotSign")
    if sign then
        local yb = sign:FindFirstChild("YourBase")
        if yb and yb:IsA("BillboardGui") then return yb.Enabled == true end
    end
    return false
end

local function scanPlotNormal(plot)
    if not plot or not plot:IsA("Model") then return end
    if isMyPlot(plot.Name) then return end
    local podiums = plot:FindFirstChild("AnimalPodiums")
    if not podiums then return end
    for _, pod in ipairs(podiums:GetChildren()) do
        if pod:IsA("Model") and pod:FindFirstChild("Base") then
            local uid = plot.Name .. "_" .. pod.Name
            for _, ex in ipairs(M.animalCache) do if ex.uid == uid then return end end
            table.insert(M.animalCache, {
                name = pod.Name,
                plot = plot.Name,
                slot = pod.Name,
                worldPosition = pod:GetPivot().Position,
                uid = uid,
            })
        end
    end
end

local function findPromptNormal(ad)
    if not ad then return nil end
    local cp = M.promptCache[ad.uid]
    if cp and cp.Parent then return cp end
    local plots = workspace:FindFirstChild("Plots")
    if not plots then return nil end
    local plot = plots:FindFirstChild(ad.plot)
    if not plot then return nil end
    local pods = plot:FindFirstChild("AnimalPodiums")
    if not pods then return nil end
    local pod = pods:FindFirstChild(ad.slot)
    if not pod then return nil end
    local base = pod:FindFirstChild("Base")
    if not base then return nil end
    local spawn = base:FindFirstChild("Spawn")
    if not spawn then return nil end
    local att = spawn:FindFirstChild("PromptAttachment")
    local prompt = nil
    if att then
        for _, p in ipairs(att:GetChildren()) do
            if p:IsA("ProximityPrompt") then prompt = p; break end
        end
    end
    if not prompt then
        for _, obj in ipairs(spawn:GetDescendants()) do
            if obj:IsA("ProximityPrompt") then prompt = obj; break end
        end
    end
    if prompt then M.promptCache[ad.uid] = prompt end
    return prompt
end

local function nearestAnimalNormal()
    local char = player.Character
    if not char then return nil end
    local hrp = char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("UpperTorso")
    if not hrp then return nil end
    local best, bestD = nil, math.huge
    for _, ad in ipairs(M.animalCache) do
        if not isMyPlot(ad.plot) and ad.worldPosition then
            local d = (hrp.Position - ad.worldPosition).Magnitude
            if d < bestD then bestD = d; best = ad end
        end
    end
    return best, bestD
end

local function buildCallbacks(prompt)
    if M.stealCache[prompt] then return end
    local data = { holdCallbacks = {}, triggerCallbacks = {}, ready = true }
    local ok1, c1 = pcall(getconnections, prompt.PromptButtonHoldBegan)
    if ok1 and type(c1) == "table" then
        for _, conn in ipairs(c1) do
            if type(conn.Function) == "function" then
                table.insert(data.holdCallbacks, conn.Function)
            end
        end
    end
    local ok2, c2 = pcall(getconnections, prompt.Triggered)
    if ok2 and type(c2) == "table" then
        for _, conn in ipairs(c2) do
            if type(conn.Function) == "function" then
                table.insert(data.triggerCallbacks, conn.Function)
            end
        end
    end
    if #data.holdCallbacks > 0 or #data.triggerCallbacks > 0 then
        M.stealCache[prompt] = data
    end
end

local function execStealNormal(prompt, animalName)
    local data = M.stealCache[prompt]
    if not data or not data.ready then return false end
    data.ready = false
    M.isStealing = true
    M.stealStartTime = tick()
    M.updateStealProgress(0.1)

    if M.progressConn then M.progressConn:Disconnect() end
    M.progressConn = RunService.Heartbeat:Connect(function()
        if not M.isStealing then
            M.progressConn:Disconnect()
            M.progressConn = nil
            return
        end
        local prog = math.clamp((tick() - M.stealStartTime) / M.Steal.StealDuration, 0, 1)
        M.updateStealProgress(prog)
    end)

    task.spawn(function()
        for _, fn in ipairs(data.holdCallbacks) do task.spawn(fn) end
        local elapsed = 0
        while elapsed < M.Steal.StealDuration do elapsed = elapsed + task.wait() end
        for _, fn in ipairs(data.triggerCallbacks) do task.spawn(fn) end
        task.wait(0.01)
        if M.progressConn then M.progressConn:Disconnect(); M.progressConn = nil end
        M.isStealing = false
        M.updateStealProgress(0)
        data.ready = true
    end)
    return true
end

function M.startNormalSteal()
    if M.stealConn then return end
    M.stealConn = RunService.Heartbeat:Connect(function()
        local now = tick()
        if now - (M._normalStealTick or 0) < 0.08 then return end
        M._normalStealTick = now
        if not M.Steal.AutoStealEnabled or (M.stealMode ~= "Normal" and M.stealMode ~= "V1") or M.isStealing then return end
        local target, dist = nearestAnimalNormal()
        if not target then return end
        if dist > M.getActiveStealRadius() then return end
        local prompt = M.promptCache[target.uid]
        if not prompt or not prompt.Parent then
            prompt = findPromptNormal(target)
        end
        if prompt then
            buildCallbacks(prompt)
            execStealNormal(prompt, target.name)
        end
    end)
end

function M.stopNormalSteal()
    if M.stealConn then
        M.stealConn:Disconnect()
        M.stealConn = nil
    end
    M.isStealing = false
    if M.progressConn then M.progressConn:Disconnect(); M.progressConn = nil end
    M.updateStealProgress(0)
end

-- ============================================================
-- V2 AUTO-STEAL (from ONI — pause at 75%, finish when close)
-- ============================================================
do
    local V2 = M.V2 or {}
    M.V2 = V2
    V2.enabled = false
    V2.isStealing = false
    V2.data = V2.data or {}
    V2.conn = nil
    V2.progressConn = nil
    V2.stealStartTime = 0
    V2.paused = false
    V2.pauseStarted = nil
    V2.pausedDuration = 0
    local function barSet(p, label)
        local progress = math.clamp(tonumber(p) or 0, 0, 1)
        local pct = math.floor(progress * 100 + 0.5)
        local text = nil
        if type(label) == "string" and label ~= "" then
            text = string.upper(label)
            if progress > 0 then text = text .. "  " .. tostring(pct) .. "%" end
        end
        M.updateStealProgress(progress, text)
    end
    local function barReset()
        M.updateStealProgress(0)
    end
    local function execStealV2(prompt)
        if V2.isStealing then return end
        if not prompt then return end
        buildCallbacks(prompt)
        local data = M.stealCache[prompt]
        if not data or not data.ready then return end
        data.ready = false
        V2.isStealing = true
        M.isStealing = true
        V2.stealStartTime = tick()
        V2.paused = false
        V2.pauseStarted = nil
        V2.pausedDuration = 0
        local duration = math.max(tonumber(M.Steal.StealDuration) or 1.4, 0.05)
        local pauseFraction = 0.75 -- always pause / hold bar at 75%
        local finishFraction = 1 - pauseFraction
        local targetPart = prompt:FindFirstAncestorWhichIsA("BasePart")
        local restarting = false
        local function modeStillActive()
            return V2.enabled and M.stealMode == "V2" and V2.isStealing
        end
        local function isTargetInCurrentRadius()
            local char = player.Character
            local root = char and (char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("UpperTorso"))
            local radius = M.getActiveStealRadius()
            return root and targetPart and targetPart.Parent and (root.Position - targetPart.Position).Magnitude <= radius
        end
        local function isCloseEnoughToGrab()
            -- While auto left/right pathing, don't require tight range — steal must continue
            if M.autoLeftEnabled or M.autoRightEnabled then
                return isTargetInCurrentRadius()
            end
            local char = player.Character
            local root = char and (char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("UpperTorso"))
            local closeRange = math.min(M.getActiveStealRadius(), 12)
            return root and targetPart and targetPart.Parent and (root.Position - targetPart.Position).Magnitude <= closeRange
        end
        local function canStillGrab()
            if not prompt or not prompt.Parent or not targetPart or not targetPart.Parent then return false end
            -- Keep grab alive during auto left/right even if prompt briefly disables
            if M.autoLeftEnabled or M.autoRightEnabled then
                return isTargetInCurrentRadius() or true
            end
            if not prompt.Enabled then return false end
            if not tostring(prompt.ActionText):lower():find("steal", 1, true) then return false end
            return isTargetInCurrentRadius()
        end
        local function restartFromZero()
            if restarting then return end
            restarting = true
            V2.paused = false
            V2.pauseStarted = nil
            V2.pausedDuration = 0
            if V2.progressConn then V2.progressConn:Disconnect(); V2.progressConn = nil end
            barReset()
            data.ready = true
            V2.isStealing = false
            M.isStealing = false
        end
        if V2.progressConn then V2.progressConn:Disconnect() end
        V2.progressConn = RunService.Heartbeat:Connect(function()
            if not V2.isStealing then
                if V2.progressConn then V2.progressConn:Disconnect(); V2.progressConn = nil end
                return
            end
            if not modeStillActive() or not canStillGrab() then
                restartFromZero()
                return
            end
            local elapsed = tick() - V2.stealStartTime - (V2.pausedDuration or 0)
            if V2.paused and V2.pauseStarted then
                -- freeze bar exactly at 75% while waiting to finish
                barSet(pauseFraction, "BRAINROT")
            else
                -- never show past 75% until finish phase resumes
                local p = math.clamp(elapsed / duration, 0, 1)
                if p > pauseFraction then p = pauseFraction end
                barSet(p)
            end
        end)
        task.spawn(function()
            for _, fn in ipairs(data.holdCallbacks) do task.spawn(fn) end
            if #data.holdCallbacks == 0 and #data.triggerCallbacks == 0 and fireproximityprompt then
                pcall(function()
                    fireproximityprompt(prompt, duration)
                end)
            end
            local holdStart = tick()
            while tick() - holdStart < duration * pauseFraction do
                if not modeStillActive() or not canStillGrab() then
                    restartFromZero()
                    return
                end
                task.wait()
            end
            V2.paused = true
            V2.pauseStarted = tick()
            barSet(pauseFraction, "BRAINROT") -- hard stop display at 75%
            local wasAbleToGrab = false
            local waitStart = tick()
            while modeStillActive() do
                if not canStillGrab() then
                    restartFromZero()
                    return
                end
                if isCloseEnoughToGrab() then
                    wasAbleToGrab = true
                    break
                end
                if tick() - waitStart >= 0.95 then
                    wasAbleToGrab = true
                    break
                end
                task.wait()
            end
            if not modeStillActive() or not canStillGrab() then
                restartFromZero()
                return
            end
            if wasAbleToGrab then
                V2.pausedDuration = (V2.pausedDuration or 0) + (tick() - V2.pauseStarted)
                V2.paused = false
                V2.pauseStarted = nil
                local finishStart = tick()
                while tick() - finishStart < duration * finishFraction do
                    if not modeStillActive() or not canStillGrab() then
                        restartFromZero()
                        return
                    end
                    task.wait()
                end
                for _, fn in ipairs(data.triggerCallbacks) do task.spawn(fn) end
                pcall(function() if _G.AutoCarrySpeed and _G.AutoCarrySpeed.WatchPickup then _G.AutoCarrySpeed.WatchPickup(1.25) end end)
                if V2.progressConn then V2.progressConn:Disconnect(); V2.progressConn = nil end
                barSet(1, "SUCCESS")
                task.wait(0.05)
                barReset()
                data.ready = true
                V2.isStealing = false
                M.isStealing = false
            else
                restartFromZero()
            end
        end)
    end
    function M.startV2Steal()
        V2.enabled = true
        V2.isStealing = false
        if V2.conn then V2.conn:Disconnect(); V2.conn = nil end
        V2.conn = RunService.Heartbeat:Connect(function()
            local now = tick()
            if now - (V2._lastTick or 0) < 0.08 then return end
            V2._lastTick = now
            if not V2.enabled then return end
            if not M.Steal.AutoStealEnabled then return end
            if M.stealMode ~= "V2" then M.stopV2Steal(); return end
            if V2.isStealing then return end
            local target, dist = nearestAnimalNormal()
            if not target then return end
            if dist > M.getActiveStealRadius() then return end
            local prompt = M.promptCache[target.uid]
            if not prompt or not prompt.Parent then
                prompt = findPromptNormal(target)
            end
            if prompt then
                execStealV2(prompt)
            end
        end)
    end
    function M.stopV2Steal()
        V2.enabled = false
        V2.isStealing = false
        V2.paused = false
        V2.pauseStarted = nil
        V2.pausedDuration = 0
        if V2.conn then V2.conn:Disconnect(); V2.conn = nil end
        if V2.progressConn then V2.progressConn:Disconnect(); V2.progressConn = nil end
        M.isStealing = false
        barReset()
    end
end

-- ============================================================
-- SEMI AUTO-STEAL (unchanged)
-- ============================================================
do
    local A = M.Semi
    if A.conn then pcall(function() A.conn:Disconnect() end); A.conn = nil end
    A.enabled = false
    A.holdMin = tonumber(A.holdMin) or 1.3
    A.holdMax = tonumber(A.holdMax) or 2.6
    A.entryDelay = tonumber(A.entryDelay) or 0.3
    A.cooldown = tonumber(A.cooldown) or 0.05
    A.primeRange = tonumber(A.primeRange) or 80
    A.radius = math.min(tonumber(A.radius) or 10, 10)
    A.plotSync = A.plotSync or {caches = {}, connections = {}}
    A.animals = A.animals or {}
    A.promptCache = A.promptCache or {}
    A.internalCache = A.internalCache or {}
    A.state = A.state or {active = false, startTime = 0, phase = "idle", label = "", lastResult = "", lastResultTime = 0}

    local function barSet(p, label)
        local progress = math.clamp(tonumber(p) or 0, 0, 1)
        local pct = math.floor(progress * 100 + 0.5)
        local text = nil
        if type(label) == "string" and label ~= "" then
            text = string.upper(label)
            if progress > 0 then
                text = text .. "  " .. tostring(pct) .. "%"
            end
        end
        M.updateStealProgress(progress, text)
    end
    local function barReset()
        M.updateStealProgress(0)
    end
    local function rootPart()
        local char = player.Character
        return char and (char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("UpperTorso")) or nil
    end
    local function splitPath(path)
        if typeof(path) == "table" then return path end
        local out = {}
        for part in string.gmatch(tostring(path), "[^%.]+") do
            table.insert(out, tonumber(part) or part)
        end
        return out
    end
    local function resolvePath(path, root)
        local current, parent, key = root, nil, nil
        for _, part in ipairs(splitPath(path)) do
            parent = current
            key = part
            current = current and current[part] or nil
        end
        return current, parent, key
    end
    local function applySyncDiff(channelName, packet)
        local cache = A.plotSync.caches[channelName]
        if typeof(cache) ~= "table" then return end
        local path, action, a, b = packet[1], packet[2], packet[3], packet[4]
        local current, parent, key = resolvePath(path, cache)
        if action == "Changed" then
            if parent ~= nil then parent[key] = a end
        elseif action == "ArrayInsert" then
            if current ~= nil then table.insert(current, b, a) end
        elseif action == "ArrayRemoved" then
            if current ~= nil then table.remove(current, b) end
        elseif action == "DictionaryInsert" then
            if current ~= nil then current[b] = a end
        elseif action == "DictionaryRemoved" then
            if current ~= nil then current[b] = nil end
        end
    end
    local function attachPlotChannel(remote, plots, requestData)
        if A.plotSync.connections[remote] then return end
        local channelName = tostring(remote.Name)
        if not plots:FindFirstChild(channelName) then return end
        if requestData and A.plotSync.caches[channelName] == nil then
            local ok, data = pcall(function() return requestData:InvokeServer(channelName) end)
            A.plotSync.caches[channelName] = (ok and typeof(data) == "table") and data or {}
        elseif A.plotSync.caches[channelName] == nil then
            A.plotSync.caches[channelName] = {}
        end
        A.plotSync.connections[remote] = remote.OnClientEvent:Connect(function(queue)
            for _, packet in ipairs(queue) do applySyncDiff(channelName, packet) end
        end)
    end

    function M.initSemiSync()
        if A.syncReady then return true end
        local ok = pcall(function()
            local rs = game:GetService("ReplicatedStorage")
            A.packages = rs:WaitForChild("Packages", 10)
            A.datas = rs:WaitForChild("Datas", 10)
            A.plots = workspace:WaitForChild("Plots", 10)
            if not (A.packages and A.datas and A.plots) then return end
            A.animalsData = require(A.datas:WaitForChild("Animals", 10))
            local sync = A.packages:WaitForChild("Synchronizer", 10)
            A.channelFolder = sync:WaitForChild("Channel", 10)
            A.routeRemote = sync:WaitForChild("CommunicationRoute", 10)
            A.requestData = sync:FindFirstChild("RequestData")
            for _, child in ipairs(A.channelFolder:GetChildren()) do
                if child:IsA("RemoteEvent") then attachPlotChannel(child, A.plots, A.requestData) end
            end
            A.channelFolder.ChildAdded:Connect(function(child)
                if child:IsA("RemoteEvent") then attachPlotChannel(child, A.plots, A.requestData) end
            end)
            A.routeRemote.OnClientEvent:Connect(function(actions)
                for _, action in ipairs(actions) do
                    local kind, channelName = action[1], tostring(action[2])
                    if A.plots and A.plots:FindFirstChild(channelName) then
                        if kind == "ListenerAdded" then
                            local remote = A.channelFolder and A.channelFolder:FindFirstChild(channelName)
                            if remote and remote:IsA("RemoteEvent") then attachPlotChannel(remote, A.plots, A.requestData) end
                        elseif kind == "ListenerRemoved" then
                            for remote, conn in pairs(A.plotSync.connections) do
                                if tostring(remote.Name) == channelName then
                                    pcall(function() conn:Disconnect() end)
                                    A.plotSync.connections[remote] = nil
                                    A.plotSync.caches[channelName] = nil
                                    break
                                end
                            end
                        end
                    end
                end
            end)
            A.syncReady = true
        end)
        return ok and A.syncReady == true
    end

    local function getPlotOwner(plot)
        local sign = plot and plot:FindFirstChild("PlotSign")
        local frame = sign and sign:FindFirstChild("SurfaceGui") and sign.SurfaceGui:FindFirstChild("Frame")
        local label = frame and frame:FindFirstChild("TextLabel")
        if not label or label.Text == "Empty Base" then return nil end
        return label.Text:gsub("'s [Bb]ase$", ""):gsub("%s+$", "")
    end
    local function isMyBaseAnimal(animalData)
        if not animalData or not animalData.plot or not A.plots then return false end
        local plot = A.plots:FindFirstChild(animalData.plot)
        if not plot then return false end
        local owner = getPlotOwner(plot)
        return owner == player.DisplayName or owner == player.Name
    end
    local function podiumFor(animalData)
        local plot = A.plots and A.plots:FindFirstChild(animalData.plot)
        local podiums = plot and plot:FindFirstChild("AnimalPodiums")
        return podiums and podiums:FindFirstChild(animalData.slot) or nil
    end
    local function animalPos(animalData)
        local podium = podiumFor(animalData)
        return podium and podium:GetPivot().Position or nil
    end
    local function distToAnimal(animalData)
        local root = rootPart()
        local pos = animalPos(animalData)
        return root and pos and (root.Position - pos).Magnitude or math.huge
    end
    local function findPromptForAnimal(animalData)
        if not animalData then return nil end
        local cached = A.promptCache[animalData.uid]
        if cached and cached.Parent then return cached end
        local podium = podiumFor(animalData)
        local base = podium and podium:FindFirstChild("Base")
        local spawn = base and base:FindFirstChild("Spawn")
        local attach = spawn and spawn:FindFirstChild("PromptAttachment")
        if not attach then return nil end
        for _, prompt in ipairs(attach:GetChildren()) do
            if prompt:IsA("ProximityPrompt") then
                A.promptCache[animalData.uid] = prompt
                return prompt
            end
        end
        return nil
    end

    function M.scanAllPlotsSemi()
        if not M.initSemiSync() then return 0 end
        local newCache = {}
        for _, plot in ipairs(A.plots:GetChildren()) do
            local cache = A.plotSync.caches[plot.Name]
            local animalList = cache and cache.AnimalList
            if typeof(animalList) == "table" then
                for slot, animalData in pairs(animalList) do
                    if type(animalData) == "table" then
                        local animalName = animalData.Index
                        local info = A.animalsData and A.animalsData[animalName]
                        if info then
                            table.insert(newCache, {
                                name = info.DisplayName or animalName,
                                plot = plot.Name,
                                slot = tostring(slot),
                                uid = plot.Name .. "_" .. tostring(slot),
                            })
                        end
                    end
                end
            end
        end
        A.animals = newCache
        return #newCache
    end

    local function pickClosest()
        local root = rootPart()
        if not root then return nil end
        local best, bestDist = nil, math.huge
        for _, animalData in ipairs(A.animals) do
            if not isMyBaseAnimal(animalData) then
                local pos = animalPos(animalData)
                local dist = pos and (root.Position - pos).Magnitude or math.huge
                if dist <= (A.primeRange or 80) and dist < bestDist then
                    best, bestDist = animalData, dist
                end
            end
        end
        return best
    end
    local function buildCallbacks(prompt)
        if A.internalCache[prompt] then return end
        local data = {holdCallbacks = {}, triggerCallbacks = {}, ready = true}
        local okHold, holds = pcall(getconnections, prompt.PromptButtonHoldBegan)
        if okHold and type(holds) == "table" then
            for _, conn in ipairs(holds) do
                if type(conn.Function) == "function" then table.insert(data.holdCallbacks, conn.Function) end
            end
        end
        local okTrigger, triggers = pcall(getconnections, prompt.Triggered)
        if okTrigger and type(triggers) == "table" then
            for _, conn in ipairs(triggers) do
                if type(conn.Function) == "function" then table.insert(data.triggerCallbacks, conn.Function) end
            end
        end
        if #data.holdCallbacks > 0 or #data.triggerCallbacks > 0 then A.internalCache[prompt] = data end
    end
    local function executeSemi(prompt, animalData)
        if not prompt or not prompt.Parent or not animalData then return false end
        buildCallbacks(prompt)
        local data = A.internalCache[prompt]
        if not data or not data.ready then return false end
        data.ready = false
        A.state.active = true
        A.state.startTime = tick()
        A.state.phase = "holding"
        A.state.label = animalData.name or "Animal"
        M.isStealing = true
        M.stealStartTime = A.state.startTime
        task.spawn(function()
            local startTime = A.state.startTime
            for _, fn in ipairs(data.holdCallbacks) do task.spawn(function() pcall(fn) end) end
            while A.enabled and (M.stealMode == "Semi" or M.stealMode == "V4") and tick() - startTime < (A.holdMin or 1.3) do
                local elapsed = tick() - startTime
                A.state.phase = "holding"
                barSet(elapsed / (A.holdMax or 2.6), "HOLDING " .. tostring(A.state.label))
                task.wait()
            end
            A.state.phase = "waitingRange"
            local alreadyInRange = distToAnimal(animalData) <= (tonumber(A.radius) or 10)
            local fired = false
            while A.enabled and (M.stealMode == "Semi" or M.stealMode == "V4") and prompt.Parent do
                local elapsed = tick() - startTime
                if elapsed > (A.holdMax or 2.6) then break end
                barSet(elapsed / (A.holdMax or 2.6), "MOVE CLOSER  " .. tostring(A.state.label))
                if distToAnimal(animalData) <= (tonumber(A.radius) or 10) then
                    if not alreadyInRange then task.wait(A.entryDelay or 0.3) end
                    if A.enabled and (M.stealMode == "Semi" or M.stealMode == "V4") then
                        for _, fn in ipairs(data.triggerCallbacks) do task.spawn(function() pcall(fn) end) end
                        pcall(function() if _G.AutoCarrySpeed and _G.AutoCarrySpeed.WatchPickup then _G.AutoCarrySpeed.WatchPickup(1.25) end end)
                        fired = true
                    end
                    break
                end
                task.wait()
            end
            A.state.lastResult = fired and ("Stole " .. tostring(A.state.label)) or ("Missed window: " .. tostring(A.state.label))
            A.state.active = false
            A.state.phase = "idle"
            A.state.lastResultTime = tick()
            if fired then
                barSet(1, "STOLE " .. tostring(A.state.label))
            else
                barSet(0, A.state.lastResult)
            end
            task.wait(A.cooldown or 0.05)
            data.ready = true
            M.isStealing = false
            barReset()
        end)
        return true
    end

    function M.stopSemiSteal()
        A.enabled = false
        if A.conn then A.conn:Disconnect(); A.conn = nil end
        A.state.active = false
        A.state.phase = "idle"
        M.isStealing = false
        barReset()
    end

    function M.startSemiSteal()
        A.radius = math.min(tonumber(A.radius) or 10, 10)
        A.enabled = true
        M.initSemiSync()
        pcall(M.scanAllPlotsSemi)
        if A.conn then A.conn:Disconnect(); A.conn = nil end
        A.conn = RunService.Heartbeat:Connect(function()
            local now = tick()
            if now - (A._lastTick or 0) < 0.08 then return end
            A._lastTick = now
            if not A.enabled then return end
            if not M.Steal.AutoStealEnabled then return end
            if M.stealMode ~= "Semi" and M.stealMode ~= "V4" then M.stopSemiSteal(); return end
            if A.state.active then return end
            local target = pickClosest()
            if not target then return end
            local prompt = findPromptForAnimal(target)
            if prompt then executeSemi(prompt, target) end
        end)
    end
end

local function v3ReleasePrompt(prompt)
    if not prompt then return end
    pcall(function()
        if prompt.InputHoldEnd then prompt:InputHoldEnd() end
    end)
end

local function v3HoldPrompt(prompt)
    if not prompt or not prompt.Parent then return false end
    -- Native hold (works without getconnections)
    local ok = pcall(function()
        if prompt.InputHoldBegin then
            prompt:InputHoldBegin()
        end
    end)
    if not ok then
        pcall(function()
            if fireproximityprompt then
                fireproximityprompt(prompt)
            end
        end)
    end
    -- Also fire hooked hold callbacks if available
    buildCallbacks(prompt)
    local data = M.stealCache[prompt]
    if data then
        for _, fn in ipairs(data.holdCallbacks) do
            task.spawn(function() pcall(fn) end)
        end
    end
    return true
end

local function v3TriggerPrompt(prompt)
    if not prompt then return end
    buildCallbacks(prompt)
    local data = M.stealCache[prompt]
    if data then
        for _, fn in ipairs(data.triggerCallbacks) do
            task.spawn(function() pcall(fn) end)
        end
    end
    pcall(function()
        if prompt.InputHoldEnd then prompt:InputHoldEnd() end
    end)
    pcall(function()
        if fireproximityprompt then
            fireproximityprompt(prompt)
        end
    end)
end

local function v3LiveDist(ad, hrp)
    if not ad or not hrp then return math.huge end
    -- Prefer live podium position so cache doesn't go stale
    local plots = workspace:FindFirstChild("Plots")
    local plot = plots and plots:FindFirstChild(ad.plot)
    local pods = plot and plot:FindFirstChild("AnimalPodiums")
    local pod = pods and pods:FindFirstChild(ad.slot)
    if pod then
        local ok, pos = pcall(function() return pod:GetPivot().Position end)
        if ok and pos then
            ad.worldPosition = pos
            return (hrp.Position - pos).Magnitude
        end
    end
    if ad.worldPosition then
        return (hrp.Position - ad.worldPosition).Magnitude
    end
    return math.huge
end

function M.startV3Steal()
    if M.V3.conn then return end
    M.V3.enabled = true
    M.V3.progress = 0
    M.V3.currentUid = nil
    M.V3.lastInRange = 0
    M.V3.holding = false
    M.V3.holdPrompt = nil
    M.V3.cooldownUntil = 0
    M.V3.lastHoldPulse = 0

    M.V3.conn = RunService.Heartbeat:Connect(function(dt)
        local now = tick()
        if now - (M.V3._lastTick or 0) < 0.08 then return end
        M.V3._lastTick = now
        if not M.Steal.AutoStealEnabled or M.stealMode ~= "V3" or not M.V3.enabled then
            if M.V3.holdPrompt then v3ReleasePrompt(M.V3.holdPrompt) end
            if M.V3.progress > 0 or M.V3.holding or M.isStealing then
                M.V3.progress = 0
                M.V3.currentUid = nil
                M.V3.holding = false
                M.V3.holdPrompt = nil
                M.isStealing = false
                M.updateStealProgress(0)
            end
            return
        end

        local stopT = math.max(tonumber(M.Steal.StopTime) or 0.35, 0.05)
        local holdT = math.max(tonumber(M.Steal.StealDuration) or 1.4, 0.05)

        if tick() < (M.V3.cooldownUntil or 0) then
            M.updateStealProgress(0)
            return
        end

        local char = player.Character
        local hrp = char and (char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("UpperTorso"))
        if not hrp then return end

        local target = nearestAnimalNormal()
        local dist = target and v3LiveDist(target, hrp) or math.huge
        local radius = M.getActiveStealRadius()
        local inRange = target ~= nil and dist <= radius

        if inRange then
            M.V3.lastInRange = tick()

            if M.V3.currentUid ~= target.uid then
                -- switched pet: release old hold, keep some progress only if same fill preferred restart
                if M.V3.holdPrompt then v3ReleasePrompt(M.V3.holdPrompt) end
                M.V3.currentUid = target.uid
                M.V3.progress = 0
                M.V3.holding = false
                M.V3.holdPrompt = nil
            end

            local prompt = M.promptCache[target.uid]
            if not prompt or not prompt.Parent then
                prompt = findPromptNormal(target)
            end
            if not prompt then
                -- still show proximity progress so bar matches video feel
                M.V3.progress = math.clamp(M.V3.progress + (dt / holdT), 0, 1)
                M.updateStealProgress(M.V3.progress)
                M.isStealing = M.V3.progress > 0
                return
            end

            -- Keep hold alive: pulse InputHoldBegin ~10x/sec while in range
            M.V3.holdPrompt = prompt
            M.isStealing = true
            local now = tick()
            if (not M.V3.holding) or (now - (M.V3.lastHoldPulse or 0) > 0.1) then
                M.V3.holding = true
                M.V3.lastHoldPulse = now
                v3HoldPrompt(prompt)
            end

            M.V3.progress = math.clamp(M.V3.progress + (dt / holdT), 0, 1)
            M.updateStealProgress(M.V3.progress)

            if M.V3.progress >= 1 then
                v3TriggerPrompt(prompt)
                M.V3.progress = 0
                M.V3.currentUid = nil
                M.V3.holding = false
                M.V3.holdPrompt = nil
                M.isStealing = false
                M.updateStealProgress(0)
                M.V3.cooldownUntil = tick() + math.max(stopT, 0.25)
            end
        else
            -- Out of range: release hold, decay progress over Stop Time (video-style drop)
            if M.V3.holding or M.V3.holdPrompt then
                v3ReleasePrompt(M.V3.holdPrompt)
                M.V3.holding = false
                M.V3.holdPrompt = nil
            end

            if M.V3.progress > 0 then
                local decay = dt / stopT
                M.V3.progress = math.max(0, M.V3.progress - decay)
                M.updateStealProgress(M.V3.progress)
                if M.V3.progress <= 0 then
                    M.V3.currentUid = nil
                    M.isStealing = false
                    M.updateStealProgress(0)
                else
                    M.isStealing = true
                end
            else
                M.isStealing = false
            end
        end
    end)
end

function M.stopV3Steal()
    M.V3.enabled = false
    if M.V3.holdPrompt then
        v3ReleasePrompt(M.V3.holdPrompt)
    end
    if M.V3.conn then
        pcall(function() M.V3.conn:Disconnect() end)
        M.V3.conn = nil
    end
    M.V3.progress = 0
    M.V3.currentUid = nil
    M.V3.holding = false
    M.V3.holdPrompt = nil
    M.V3.cooldownUntil = 0
    M.V3.lastInRange = 0
    M.V3.lastHoldPulse = 0
    M.isStealing = false
    M.updateStealProgress(0)
end

function M.startAutoSteal()
    if M.statusGui then M.statusGui.Enabled = true end
    if M.stealMode == "Semi" or M.stealMode == "V4" then
        M.startSemiSteal()
    elseif M.stealMode == "V3" then
        M.startV3Steal()
    elseif M.stealMode == "V1" then
        M.startNormalSteal()
    else
        M.stealMode = "V2"
        M.startV2Steal()
    end
end

function M.stopAutoSteal()
    if M.statusGui then M.statusGui.Enabled = true end
    M.stopNormalSteal()
    M.stopSemiSteal()
    if M.stopV2Steal then M.stopV2Steal() end
    if M.stopV3Steal then M.stopV3Steal() end
    M.isStealing = false
    M.updateStealProgress(0)
end

function M.setStealRadius(radius)
    M.Steal.StealRadius = radius
    M.updateStatusRadius()
end

-- ============================================================

-- OTHER CORE FUNCTIONS (unchanged - abbreviate per spazio)
-- ============================================================
function M.findBat()
    local char=player.Character;if not char then return nil end
    for _,tool in ipairs(char:GetChildren()) do if tool:IsA("Tool") and (tool.Name:lower():find("bat") or tool.Name:lower():find("slap")) then return tool end end
    local bp=player:FindFirstChild("Backpack");if bp then for _,tool in ipairs(bp:GetChildren()) do if tool:IsA("Tool") and (tool.Name:lower():find("bat") or tool.Name:lower():find("slap")) then return tool end end end
    return nil
end

function M.findMedusa()
    local c=player.Character;if not c then return nil end
    for _,t in ipairs(c:GetChildren()) do if t:IsA("Tool") then local n=t.Name:lower();if n:find("medusa") or n:find("head") or n:find("stone") then return t end end end
    local bp=player:FindFirstChild("Backpack");if bp then for _,t in ipairs(bp:GetChildren()) do if t:IsA("Tool") then local n=t.Name:lower();if n:find("medusa") or n:find("head") or n:find("stone") then return t end end end end
    return nil
end

function M.useMedusaCounter()
    if M.medusaDebounce then return end;if M.MEDUSA_COOLDOWN>(tick()-M.medusaLastUsed) then return end
    local c=player.Character;if not c then return end;M.medusaDebounce=true
    local med=M.findMedusa();if not med then M.medusaDebounce=false;return end
    if med.Parent~=c then local hum2=c:FindFirstChildOfClass("Humanoid");if hum2 then hum2:EquipTool(med) end end
    pcall(function() med:Activate() end);M.medusaLastUsed=tick();M.medusaDebounce=false
end

function M.onAnchorChanged(part)
    return part:GetPropertyChangedSignal("Anchored"):Connect(function()
        if part.Anchored and part.Transparency==1 then
            if M.medusaResetEnabled then M.cursedInstaReset()
            elseif M.medusaCounterEnabled then M.useMedusaCounter() end
        end
    end)
end

function M.setupMedusa(char)
    for _,c in pairs(M.Conns.anchor) do pcall(function() c:Disconnect() end) end;M.Conns.anchor={}
    if not char then return end
    for _,part in ipairs(char:GetDescendants()) do if part:IsA("BasePart") then table.insert(M.Conns.anchor,M.onAnchorChanged(part)) end end
    table.insert(M.Conns.anchor,char.DescendantAdded:Connect(function(part) if part:IsA("BasePart") then table.insert(M.Conns.anchor,M.onAnchorChanged(part)) end end))
end

function M.stopMedusaCounter() for _,c in pairs(M.Conns.anchor) do pcall(function() c:Disconnect() end) end;M.Conns.anchor={} end

function M.findBatForCounter()
    local c=player.Character;if not c then return nil end;local bp=player:FindFirstChildOfClass("Backpack")
    for _,name in ipairs(M.BAT_COUNTER_SLAP_LIST) do local t=c:FindFirstChild(name) or (bp and bp:FindFirstChild(name));if t then return t end end
    for _,ch in ipairs(c:GetChildren()) do if ch:IsA("Tool") and ch.Name:lower():find("bat") then return ch end end
    if bp then for _,ch in ipairs(bp:GetChildren()) do if ch:IsA("Tool") and ch.Name:lower():find("bat") then return ch end end end
    return nil
end

function M.swingBatForCounter(bat,char)
    local hum2=char:FindFirstChildOfClass("Humanoid")
    if bat.Parent~=char then if hum2 then pcall(function() hum2:EquipTool(bat) end) end;task.wait(0.05) end
    local remote=bat:FindFirstChildOfClass("RemoteEvent") or bat:FindFirstChildOfClass("RemoteFunction")
    if remote and remote:IsA("RemoteEvent") then pcall(function() remote:FireServer() end);task.wait(0.15);pcall(function() remote:FireServer() end)
    else pcall(function() bat:Activate() end);task.wait(0.15);pcall(function() bat:Activate() end) end
end

function M.startBatCounter()
    if M.Conns.batCounter then return end
    M.Conns.batCounter=RunService.Heartbeat:Connect(function()
        if not M.batCounterEnabled or M.batCounterDebounce then return end
        local char=player.Character;if not char then return end;local hum2=char:FindFirstChildOfClass("Humanoid");if not hum2 then return end
        local st=hum2:GetState()
        if st==Enum.HumanoidStateType.Physics or st==Enum.HumanoidStateType.Ragdoll or st==Enum.HumanoidStateType.FallingDown then
            M.batCounterDebounce=true;task.spawn(function() local bat=M.findBatForCounter();if bat then M.swingBatForCounter(bat,char) end;task.wait(0.5);M.batCounterDebounce=false end)
        end
    end)
end

function M.stopBatCounter() if M.Conns.batCounter then M.Conns.batCounter:Disconnect();M.Conns.batCounter=nil end;M.batCounterDebounce=false end

-- ============================================================
-- NORMAL AIMBOT (ONI logic)
-- ============================================================
M.aimbotSpeed = M.aimbotSpeed or 58
M.laggerAimbotSpeed = M.laggerAimbotSpeed or 40
M._aimbotSwingCooldown = false

function M.findBatForAimbot()
    local char = player.Character
    if not char then return nil end
    local bp = player:FindFirstChild("Backpack") or player:FindFirstChildOfClass("Backpack")

    for _, name in ipairs(M.BAT_COUNTER_SLAP_LIST or {}) do
        local t = char:FindFirstChild(name) or (bp and bp:FindFirstChild(name))
        if t and t:IsA("Tool") then return t end
    end
    for _, tool in ipairs(char:GetChildren()) do
        if tool:IsA("Tool") then
            local n = tool.Name:lower()
            if n:find("bat") or n:find("slap") or n:find("sword") or n:find("stick") or n:find("hit") or n:find("hammer") then
                return tool
            end
        end
    end
    if bp then
        for _, tool in ipairs(bp:GetChildren()) do
            if tool:IsA("Tool") then
                local n = tool.Name:lower()
                if n:find("bat") or n:find("slap") or n:find("sword") or n:find("stick") or n:find("hit") or n:find("hammer") then
                    return tool
                end
            end
        end
        -- fallback to first tool in backpack if nothing matched
        for _, tool in ipairs(bp:GetChildren()) do
            if tool:IsA("Tool") then return tool end
        end
    end
    return nil
end

function M.getClosestTargetAimbot()
    local root = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if not root then return nil end
    local closest, minDist = nil, math.huge
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= player and plr.Character then
            local tRoot = plr.Character:FindFirstChild("HumanoidRootPart")
            local hum = plr.Character:FindFirstChildOfClass("Humanoid")
            if tRoot and hum and hum.Health > 0 then
                local dist = (tRoot.Position - root.Position).Magnitude
                if dist < minDist then
                    minDist = dist
                    closest = tRoot
                end
            end
        end
    end
    return closest
end

function M.getNormalAimbotSpeed()
    if M.laggerModeEnabled or M.laggerCarryActive then
        return tonumber(M.laggerAimbotSpeed) or 40
    end
    return tonumber(M.aimbotSpeed) or 58
end

function M.startBatAimbot()
    if not M.safeModeTryStart() then return end
    if M.aimbotConn then
        pcall(function() M.aimbotConn:Disconnect() end)
        M.aimbotConn = nil
    end

    if M.autoLeftEnabled then
        M.autoLeftEnabled = false
        if M.autoLeftSetVisual then M.autoLeftSetVisual(false) end
        M.stopAutoLeft()
    end
    if M.autoRightEnabled then
        M.autoRightEnabled = false
        if M.autoRightSetVisual then M.autoRightSetVisual(false) end
        M.stopAutoRight()
    end

    M._autoTPWasEnabledForBat = false
    if M.autoTPEnabled then
        M._autoTPWasEnabledForBat = true
        M.stopAutoTP()
        if M.setAutoTPVisual then M.setAutoTPVisual(false) end
    end

    M.autoBatEnabled = true
    M._aimbotTarget = nil
    M._aimbotLastScan = 0
    M._aimbotSwingCooldown = false
    M.autoBatEquippedThisRun = false

    -- ============================================================
    -- SCYTHE DUELS normal aimbot logic (exact)
    -- ============================================================
    M.aimbotConn = RunService.Heartbeat:Connect(function()
        if not M.autoBatEnabled then return end
        local char = player.Character
        if not char then return end
        local root = char:FindFirstChild("HumanoidRootPart")
        if not root then return end
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not hum then return end

        local bat = char:FindFirstChildOfClass("Tool") or M.findBatForAimbot()
        if bat and bat.Parent ~= char then
            pcall(function() hum:EquipTool(bat) end)
        end

        -- target scan (0.1s cache like Scythe)
        local now = tick()
        local target = M._aimbotTarget
        if now - (M._aimbotLastScan or 0) > 0.1 or not target or not target.Parent then
            M._aimbotLastScan = now
            target = nil
            local closest, minDist = nil, math.huge
            for _, plr in ipairs(Players:GetPlayers()) do
                if plr ~= player and plr.Character then
                    local tRoot = plr.Character:FindFirstChild("HumanoidRootPart")
                    local th = plr.Character:FindFirstChildOfClass("Humanoid")
                    if tRoot and th and th.Health > 0 then
                        local dist = (tRoot.Position - root.Position).Magnitude
                        if dist < minDist then
                            minDist = dist
                            closest = tRoot
                        end
                    end
                end
            end
            target = closest
            M._aimbotTarget = target
        else
            local th = target.Parent and target.Parent:FindFirstChildOfClass("Humanoid")
            if not th or th.Health <= 0 then
                M._aimbotTarget = nil
                target = nil
            end
        end

        if not target then
            hum.AutoRotate = true
            root.AssemblyAngularVelocity = Vector3.zero
            return
        end

        hum.AutoRotate = false
        local targetVel = target.AssemblyLinearVelocity
        local myPos = root.Position
        local targetPos = target.Position
        local predictPos = targetPos + targetVel * 0.14
        predictPos = predictPos + target.CFrame.LookVector * 0.3
        local direction = predictPos - myPos
        local flatDir = Vector3.new(direction.X, 0, direction.Z)
        if flatDir.Magnitude > 0.01 then
            flatDir = flatDir.Unit
        else
            flatDir = Vector3.new(0, 0, 1)
        end

        local chaseSpeed = 58
        local desiredHeight = targetPos.Y + 3.7
        local yVel = (desiredHeight - myPos.Y) * 19.5 + targetVel.Y * 0.8
        if hum.FloorMaterial ~= Enum.Material.Air then
            yVel = math.max(yVel, 13)
        end
        yVel = math.clamp(yVel, -70, 110)

        local desiredVel = Vector3.new(flatDir.X * chaseSpeed, yVel, flatDir.Z * chaseSpeed)
        root.AssemblyLinearVelocity = root.AssemblyLinearVelocity:Lerp(desiredVel, 0.8)

        local speed3 = targetVel.Magnitude
        local predictTime = math.clamp(speed3 / 150, 0.05, 0.2)
        local predictedPos = targetPos + targetVel * predictTime
        local toPredict = predictedPos - myPos
        if toPredict.Magnitude > 0.1 then
            local goalCF = CFrame.lookAt(myPos, predictedPos)
            local diffCF = root.CFrame:Inverse() * goalCF
            local rx, ry, rz = diffCF:ToEulerAnglesXYZ()
            rx = math.clamp(rx, -2.5, 2.5)
            ry = math.clamp(ry, -2.5, 2.5)
            rz = math.clamp(rz, -2.5, 2.5)
            root.AssemblyAngularVelocity = root.CFrame:VectorToWorldSpace(Vector3.new(rx * 42, ry * 42, rz * 42))
        end

        local currentDist = (targetPos - myPos).Magnitude
        if bat and (M.autoSwingEnabled or currentDist <= 16) then
            pcall(function()
                bat:Activate()
                local remote = bat:FindFirstChildOfClass("RemoteEvent")
                if remote then remote:FireServer() end
            end)
            if M.notifyPlayerSwung then M.notifyPlayerSwung() end
        end
    end)

    if M.autoBatSetVisual then M.autoBatSetVisual(true) end
    if M.mobBtnRefs.autoBat then M.mobBtnRefs.autoBat(true) end
end

function M.stopBatAimbot()
    if M.aimbotConn then
        pcall(function() M.aimbotConn:Disconnect() end)
        M.aimbotConn = nil
    end
    M._aimbotTarget = nil
    M._aimbotSwingCooldown = false
    M.autoBatEnabled = false
    M.autoBatEquippedThisRun = false

    local char = player.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if root then
        -- Scythe-style soft reset
        root.AssemblyLinearVelocity = root.AssemblyLinearVelocity * 0.3
        root.AssemblyAngularVelocity = Vector3.zero
    end
    local hum2 = char and char:FindFirstChildOfClass("Humanoid")
    if hum2 then hum2.AutoRotate = true end

    if M._autoTPWasEnabledForBat then
        M._autoTPWasEnabledForBat = false
        M.autoTPEnabled = true
        if M.setAutoTPVisual then M.setAutoTPVisual(true) end
        M.startAutoTP()
    end

    if M.autoBatSetVisual then M.autoBatSetVisual(false) end
    if M.mobBtnRefs.autoBat then M.mobBtnRefs.autoBat(false) end
end

-- BAT V2 AIMBOT (ONI STYLE, SEM PREVISÃO)
-- ============================================================
M.batV2Speed = M.batV2Speed or 58
M.batV2SwingEnabled = M.batV2SwingEnabled ~= false
M.batV2SwingDistance = M.batV2SwingDistance or 16
M.batV2ScanInterval = M.batV2ScanInterval or 0.1
M.batV2AngularGain = M.batV2AngularGain or 42
M.batV2AngularClamp = M.batV2AngularClamp or 2.5
M.batV2YVelocityGain = M.batV2YVelocityGain or 19.5
M.batV2HeightOffset = M.batV2HeightOffset or 3.7
M.batV2MaxYVelocity = M.batV2MaxYVelocity or 110
M.batV2MinYVelocity = M.batV2MinYVelocity or -70
M.batV2FloorBoost = M.batV2FloorBoost or 13
M.batV2Target = nil
M.batV2LastScan = 0
M.batV2PreviousAutoRotate = nil

function M.findAnyToolBatV2()
    local char = player.Character
    if not char then return nil end
    for _, name in ipairs(M.BAT_COUNTER_SLAP_LIST or {}) do
        local tool = char:FindFirstChild(name)
        if tool and tool:IsA("Tool") then return tool end
    end
    local backpack = player:FindFirstChildOfClass("Backpack")
    if backpack then
        for _, name in ipairs(M.BAT_COUNTER_SLAP_LIST or {}) do
            local tool = backpack:FindFirstChild(name)
            if tool and tool:IsA("Tool") then
                local hum = char:FindFirstChildOfClass("Humanoid")
                if hum then pcall(function() hum:EquipTool(tool) end) end
                return tool
            end
        end
    end
    for _, tool in ipairs(char:GetChildren()) do
        if tool:IsA("Tool") then
            local name = tool.Name:lower()
            if name:find("bat") or name:find("slap") then return tool end
        end
    end
    return nil
end

function M.getClosestTargetBatV2()
    local char = player.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not root then return nil end
    local closest, bestDist = nil, math.huge
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= player and plr.Character then
            local targetRoot = plr.Character:FindFirstChild("HumanoidRootPart")
            local hum = plr.Character:FindFirstChildOfClass("Humanoid")
            if targetRoot and hum and hum.Health > 0 then
                local dist = (targetRoot.Position - root.Position).Magnitude
                if dist < bestDist then closest, bestDist = targetRoot, dist end
            end
        end
    end
    return closest
end

function M.tryHitBatV2(tool)
    if M.batV2HittingCooldown then return end
    M.batV2HittingCooldown = true
    if tool then
        pcall(function()
            tool:Activate()
            local remote = tool:FindFirstChildOfClass("RemoteEvent")
            if remote then remote:FireServer() end
        end)
    end
    task.delay(M.batV2SwingCooldown or 0.35, function()
        M.batV2HittingCooldown = false
    end)
end

function M.startBatV2Aimbot()
    if not M.safeModeTryStart() then return end
    if M.aimbotConn then pcall(function() M.aimbotConn:Disconnect() end); M.aimbotConn = nil end
    if M.batV2Conn then pcall(function() M.batV2Conn:Disconnect() end); M.batV2Conn = nil end

    local char = player.Character
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if not char or not hum then return end

    -- Mesmo comportamento de exclusividade do Bat Normal.
    if M.autoLeftEnabled then M.stopAutoLeft() end
    if M.autoRightEnabled then M.stopAutoRight() end
    M._autoTPWasEnabledForBatV2 = false
    if M.autoTPEnabled then
        M._autoTPWasEnabledForBatV2 = true
        M.stopAutoTP()
        if M.setAutoTPVisual then M.setAutoTPVisual(false) end
    end

    if M.batV2PreviousAutoRotate == nil then M.batV2PreviousAutoRotate = hum.AutoRotate end
    hum.AutoRotate = false
    M.autoBatEnabled = true
    M.batV2Enabled = true
    M.batV2Target = nil
    M.batV2LastScan = 0
    M.batV2HittingCooldown = false

    M.batV2Conn = RunService.Heartbeat:Connect(function()
        if not M.autoBatEnabled or not M.batV2Enabled then return end
        local currentChar = player.Character
        local root = currentChar and currentChar:FindFirstChild("HumanoidRootPart")
        local currentHum = currentChar and currentChar:FindFirstChildOfClass("Humanoid")
        if not root or not currentHum then return end

        local bat = M.findAnyToolBatV2()
        if bat and bat.Parent ~= currentChar then
            pcall(function() currentHum:EquipTool(bat) end)
        end

        local now = tick()
        if now - M.batV2LastScan > M.batV2ScanInterval or not M.batV2Target or not M.batV2Target.Parent then
            M.batV2LastScan = now
            M.batV2Target = M.getClosestTargetBatV2()
        else
            local targetHum = M.batV2Target.Parent and M.batV2Target.Parent:FindFirstChildOfClass("Humanoid")
            if not targetHum or targetHum.Health <= 0 then M.batV2Target = nil end
        end

        local target = M.batV2Target
        if not target then
            currentHum.AutoRotate = true
            root.AssemblyAngularVelocity = Vector3.zero
            return
        end

        currentHum.AutoRotate = false
        local targetPos = target.Position
        local myPos = root.Position
        local direction = targetPos - myPos
        local flatDir = Vector3.new(direction.X, 0, direction.Z)
        if flatDir.Magnitude > 0.01 then flatDir = flatDir.Unit else flatDir = Vector3.new(0, 0, 1) end

        local desiredHeight = targetPos.Y + M.batV2HeightOffset
        local yVel = (desiredHeight - myPos.Y) * M.batV2YVelocityGain
        if currentHum.FloorMaterial ~= Enum.Material.Air then yVel = math.max(yVel, M.batV2FloorBoost) end
        yVel = math.clamp(yVel, M.batV2MinYVelocity, M.batV2MaxYVelocity)

        -- Usa a mesma velocidade base do Bat Normal, sem previsão.
        local speed = tonumber(M.aimbotSpeed) or tonumber(M.batV2Speed) or 58
        local desiredVel = Vector3.new(flatDir.X * speed, yVel, flatDir.Z * speed)
        root.AssemblyLinearVelocity = root.AssemblyLinearVelocity:Lerp(desiredVel, 0.8)

        local toTarget = targetPos - myPos
        if toTarget.Magnitude > 0.1 then
            local goalCF = CFrame.lookAt(myPos, targetPos)
            local diffCF = root.CFrame:Inverse() * goalCF
            local rx, ry, rz = diffCF:ToEulerAnglesXYZ()
            rx = math.clamp(rx, -M.batV2AngularClamp, M.batV2AngularClamp)
            ry = math.clamp(ry, -M.batV2AngularClamp, M.batV2AngularClamp)
            rz = math.clamp(rz, -M.batV2AngularClamp, M.batV2AngularClamp)
            root.AssemblyAngularVelocity = root.CFrame:VectorToWorldSpace(Vector3.new(
                rx * M.batV2AngularGain, ry * M.batV2AngularGain, rz * M.batV2AngularGain
            ))
        else
            root.AssemblyAngularVelocity = root.AssemblyAngularVelocity * 0.9
        end

        local distance = (targetPos - myPos).Magnitude
        if bat and (M.batV2SwingEnabled or distance <= M.batV2SwingDistance) then
            M.tryHitBatV2(bat)
        end
    end)
    if M.autoBatSetVisual then M.autoBatSetVisual(true) end
end

function M.stopBatV2Aimbot()
    if M.batV2Conn then pcall(function() M.batV2Conn:Disconnect() end); M.batV2Conn = nil end
    M.batV2Enabled = false
    M.batV2Target = nil
    M.batV2HittingCooldown = false
    local char = player.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if root then
        root.AssemblyLinearVelocity = Vector3.zero
        root.AssemblyAngularVelocity = Vector3.zero
    end
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if hum then
        hum.AutoRotate = (M.batV2PreviousAutoRotate == nil) and true or M.batV2PreviousAutoRotate
    end
    M.batV2PreviousAutoRotate = nil

    if M._autoTPWasEnabledForBatV2 then
        M._autoTPWasEnabledForBatV2 = false
        M.autoTPEnabled = true
        if M.setAutoTPVisual then M.setAutoTPVisual(true) end
        M.startAutoTP()
    end
end


function M.queueAutoBatStart()
    if not M.safeModeTryStart() then return end
    if M.antiKickEnabled and M.brainrotDetected then return end
    if M.autoLeftEnabled then M.autoLeftEnabled=false; if M.autoLeftSetVisual then M.autoLeftSetVisual(false) end; M.stopAutoLeft() end
    if M.autoRightEnabled then M.autoRightEnabled=false; if M.autoRightSetVisual then M.autoRightSetVisual(false) end; M.stopAutoRight() end
    if M.autoBatMode == "BAT V2" then M.startBatV2Aimbot() else M.startBatAimbot() end
end

function M.swingCurrentBatAimbot(char)
    if not M.autoSwingEnabled then return end
    local bat = M.findBatForAimbot()
    if bat and bat.Parent == char then
        pcall(function() bat:Activate() end)
    end
end

-- ============================================================
-- BAT TP (Grape logic integrated; large-script UI/API preserved)
-- ============================================================
M._bypassTarget = nil
M._bypassHRP = nil
M._bypassHum = nil
M.tpBatRange = M.tpBatRange or 1e9 -- unlimited: always nearest enemy
M.tpBatClose = M.tpBatClose or 5 -- Green Duels threshold
M.tpBatOffset = M.tpBatOffset or 0
M.tpBatHitMode = M.tpBatHitMode or "Sure" -- "Sure" | "Normal"
M.tpBatSureHitEnabled = true
M._tpBatLastSwing = 0
M._bypassSwingCooldown = false
M._sureHitCD = false
M._normalHitCD = false
M._bypassRenderConn = nil -- Green-style RenderStepped twin

-- Find / equip any bat tool (exact or name contains "bat")
function M._bypassFindBat()
    local char = player.Character
    if not char then return nil end
    local function isBat(t)
        if not t or not t:IsA("Tool") then return false end
        local n = string.lower(t.Name)
        return n == "bat" or n:find("bat", 1, true) ~= nil
    end
    for _, t in ipairs(char:GetChildren()) do
        if isBat(t) then return t end
    end
    local bp = player:FindFirstChild("Backpack")
    if bp then
        for _, t in ipairs(bp:GetChildren()) do
            if isBat(t) then
                pcall(function() t.Parent = char end)
                return t
            end
        end
    end
    return nil
end

-- Sure Hit — auto-swing multi-fire
function M._bypassTryHitBat()
    if M.tpBatAutoSwing == false and M.autoSwingEnabled == false then return end
    if M._sureHitCD then return end
    M._sureHitCD = true
    pcall(function()
        local bat = M._bypassFindBat()
        if bat then
            for _ = 1, 3 do
                pcall(function() bat:Activate() end)
                local ev = bat:FindFirstChildWhichIsA("RemoteEvent")
                if ev then pcall(function() ev:FireServer() end) end
            end
            local rf = bat:FindFirstChildWhichIsA("RemoteFunction")
            if rf then pcall(function() rf:InvokeServer() end) end
        end
    end)
    task.delay(0.045, function() M._sureHitCD = false end)
end

-- Normal Hit — comportamento igual ao código enviado pelo usuário.
function M._bypassTryHitBatNormal()
    if M.tpBatAutoSwing == false and M.autoSwingEnabled == false then return end
    if M._normalHitCD then return end
    M._normalHitCD = true
    pcall(function()
        local bat = M._bypassFindBat()
        if bat then
            pcall(function() bat:Activate() end)
            local ev = bat:FindFirstChildWhichIsA("RemoteEvent")
            if ev then pcall(function() ev:FireServer() end) end
        end
    end)
    task.delay(0.08, function() M._normalHitCD = false end)
end

-- Prefer enemy in camera view / where you aim (shiftlock look), then nearest
function M._bypassGetClosest()
    local root = M._bypassHRP or (player.Character and player.Character:FindFirstChild("HumanoidRootPart"))
    if not root then return nil, math.huge end
    local cam = workspace.CurrentCamera
    local closest, bestScore = nil, math.huge
    local bestDist = math.huge
    local camPos = cam and cam.CFrame.Position or root.Position
    local look = cam and cam.CFrame.LookVector or root.CFrame.LookVector
    local vp = cam and cam.ViewportSize or Vector2.new(1920, 1080)
    local cx, cy = vp.X * 0.5, vp.Y * 0.5
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= player and plr.Character then
            local tRoot = plr.Character:FindFirstChild("HumanoidRootPart")
            local tHum = plr.Character:FindFirstChildOfClass("Humanoid")
            if tRoot and tHum and tHum.Health > 0 then
                local dist = (root.Position - tRoot.Position).Magnitude
                local score = dist
                if cam then
                    local screen, onScreen = cam:WorldToViewportPoint(tRoot.Position)
                    if onScreen and screen.Z > 0 then
                        local sd = (Vector2.new(screen.X, screen.Y) - Vector2.new(cx, cy)).Magnitude
                        -- Prefer who you are looking at (center of screen)
                        score = sd * 0.85 + dist * 0.25
                    else
                        local toT = (tRoot.Position - camPos)
                        if toT.Magnitude > 0.1 then
                            local dot = look:Dot(toT.Unit)
                            if dot > 0.15 then
                                score = dist * (1.6 - math.clamp(dot, 0, 1))
                            else
                                score = dist * 3.5 -- behind camera = low priority
                            end
                        end
                    end
                end
                if score < bestScore then
                    bestScore = score
                    bestDist = dist
                    closest = plr
                end
            end
        end
    end
    return closest, bestDist
end

-- Flat look from camera (shiftlock-aware) for placement
function M._bypassCamLookFlat()
    local cam = workspace.CurrentCamera
    local root = M._bypassHRP
    local look
    if cam then
        look = cam.CFrame.LookVector
    elseif root then
        look = root.CFrame.LookVector
    else
        look = Vector3.new(0, 0, -1)
    end
    look = Vector3.new(look.X, 0, look.Z)
    if look.Magnitude < 0.05 then
        look = Vector3.new(0, 0, -1)
    else
        look = look.Unit
    end
    return look
end

-- Smooth body face + place near target along camera direction (does NOT lock camera)
function M._bypassSmoothToTarget(tr)
    local hrp = M._bypassHRP
    if not hrp or not tr then return end
    local look = M._bypassCamLookFlat()
    local tVel = tr.AssemblyLinearVelocity or Vector3.zero
    -- light prediction
    local pred = tr.Position + Vector3.new(tVel.X, 0, tVel.Z) * 0.08
    -- stand slightly in front of target relative to where YOU are looking
    local standDist = 2.0
    local standPos = pred - look * standDist + Vector3.new(0, 0.55, 0)
    local faceAt = Vector3.new(pred.X, standPos.Y, pred.Z)
    local goal = CFrame.lookAt(standPos, faceAt)
    -- smooth lerp (more success, less jitter)
    local alpha = 0.62
    hrp.CFrame = hrp.CFrame:Lerp(goal, alpha)
    -- kill sideways drift
    local v = hrp.AssemblyLinearVelocity
    hrp.AssemblyLinearVelocity = Vector3.new(v.X * 0.35, v.Y, v.Z * 0.35)
end


-- ============================================================
-- ANTI-BYPASS GODMODE (immune while bypass aimbot is on)
-- ============================================================
function M._bypassClearGodConns()
    for _, key in ipairs({"_bypassGodConn", "_bypassGodHealthConn", "_bypassGodDiedConn", "_bypassGodCharConn", "_bypassGodStateConn"}) do
        local c = M[key]
        if c then pcall(function() c:Disconnect() end); M[key] = nil end
    end
end

function M._bypassProtectCharacter(char)
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not hum then return end
    pcall(function()
        hum.MaxHealth = math.huge
        hum.Health = math.huge
        hum.BreakJointsOnDeath = false
        hum.RequiresNeck = false
        hum:SetStateEnabled(Enum.HumanoidStateType.Dead, false)
        hum.PlatformStand = false
        hum.Sit = false
    end)
    if M._bypassGodHealthConn then pcall(function() M._bypassGodHealthConn:Disconnect() end) end
    M._bypassGodHealthConn = hum:GetPropertyChangedSignal("Health"):Connect(function()
        if not M.bypassAimbotEnabled then return end
        pcall(function()
            if hum.Health < (hum.MaxHealth or 100) or hum.Health <= 0 then
                hum.Health = hum.MaxHealth or 100
                hum.PlatformStand = false
                hum.Sit = false
            end
        end)
    end)
    if M._bypassGodDiedConn then pcall(function() M._bypassGodDiedConn:Disconnect() end) end
    M._bypassGodDiedConn = hum.Died:Connect(function()
        if not M.bypassAimbotEnabled then return end
        -- NO insta reset — only try to stay alive client-side
        pcall(function()
            hum.BreakJointsOnDeath = false
            hum.Health = hum.MaxHealth or 100
            hum.PlatformStand = false
            hum.Sit = false
            hum:ChangeState(Enum.HumanoidStateType.Running)
            task.defer(function()
                if not M.bypassAimbotEnabled then return end
                pcall(function()
                    hum.Health = hum.MaxHealth or 100
                    hum:ChangeState(Enum.HumanoidStateType.Running)
                end)
            end)
        end)
    end)
    pcall(function()
        if M._bypassGodStateConn then pcall(function() M._bypassGodStateConn:Disconnect() end) end
        M._bypassGodStateConn = hum.StateChanged:Connect(function(_, new)
            if not M.bypassAimbotEnabled then return end
            if new == Enum.HumanoidStateType.Dead
                or new == Enum.HumanoidStateType.Ragdoll
                or new == Enum.HumanoidStateType.FallingDown
                or new == Enum.HumanoidStateType.Physics then
                pcall(function()
                    hum.Health = hum.MaxHealth or 100
                    hum.PlatformStand = false
                    hum.Sit = false
                    hum:ChangeState(Enum.HumanoidStateType.Running)
                end)
            end
        end)
    end)
end

-- ============================================================
-- ANTI FLING / GLITCH DIE (global — not only aimbot)
-- Caps insane physics that often cause random death
-- ============================================================
M.antiFlingEnabled = true -- always default ON (stops fling deaths)
M.antiFlingMaxSpeed = M.antiFlingMaxSpeed or 120
M.antiFlingMaxY = M.antiFlingMaxY or 70
M.antiFlingMaxAng = M.antiFlingMaxAng or 25
M._antiFlingConn = nil

function M.startAntiFling()
    if M._antiFlingConn then
        pcall(function() M._antiFlingConn:Disconnect() end)
        M._antiFlingConn = nil
    end
    M.antiFlingEnabled = true
    M._antiFlingConn = RunService.Heartbeat:Connect(function()
        if not M.antiFlingEnabled then return end
        if M.bypassAimbotEnabled or M.dropActive then return end
        if M.dropActive then return end -- let drop fling work
        local char = player.Character
        if not char then return end
        local root = char:FindFirstChild("HumanoidRootPart")
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not root then return end

        local cfgMax = math.max(tonumber(M.NS) or 60, tonumber(M.CS) or 30, tonumber(M.LAGGER_CARRY_SPEED) or 25)
        local maxSpd = math.max(tonumber(M.antiFlingMaxSpeed) or 140, cfgMax * 2.2, 200)
        local maxY = math.max(tonumber(M.antiFlingMaxY) or 90, 120)
        local maxAng = tonumber(M.antiFlingMaxAng) or 40

        pcall(function()
            local v = root.AssemblyLinearVelocity
            local bad = false
            if v.Magnitude > maxSpd or math.abs(v.Y) > maxY then
                root.AssemblyLinearVelocity = Vector3.new(
                    math.clamp(v.X, -maxSpd, maxSpd),
                    math.clamp(v.Y, -maxY, maxY),
                    math.clamp(v.Z, -maxSpd, maxSpd)
                )
                bad = true
            end
            local ang = root.AssemblyAngularVelocity
            if ang.Magnitude > maxAng then
                root.AssemblyAngularVelocity = Vector3.zero
                bad = true
            end
            -- Nan / inf guards (rare but lethal)
            if v ~= v or ang ~= ang then
                root.AssemblyLinearVelocity = Vector3.zero
                root.AssemblyAngularVelocity = Vector3.zero
                bad = true
            end
            if hum then
                if bad then
                    hum.PlatformStand = false
                    hum.Sit = false
                    pcall(function() hum:ChangeState(Enum.HumanoidStateType.Running) end)
                end
                local st = hum:GetState()
                if st == Enum.HumanoidStateType.Flying
                    or st == Enum.HumanoidStateType.Ragdoll
                    or st == Enum.HumanoidStateType.FallingDown
                    or st == Enum.HumanoidStateType.Physics then
                    if bad or v.Magnitude > maxSpd * 0.7 then
                        pcall(function()
                            hum:ChangeState(Enum.HumanoidStateType.Running)
                            hum.PlatformStand = false
                            hum.Sit = false
                        end)
                    end
                end
            end
        end)
    end)
end

function M.stopAntiFling()
    M.antiFlingEnabled = false
    if M._antiFlingConn then
        pcall(function() M._antiFlingConn:Disconnect() end)
        M._antiFlingConn = nil
    end
end



function M.syncCombatAntiDie() end
function M.enableBypassGodmode()
    M._bypassClearGodConns()
    local char = player.Character
    if char then M._bypassProtectCharacter(char) end
    M._bypassGodCharConn = player.CharacterAdded:Connect(function(c)
        if not M.bypassAimbotEnabled then return end
        task.wait(0.05)
        M._bypassProtectCharacter(c)
    end)
    -- ANTI-DIE while TP lock ON: every Heartbeat (no throttle) so you cannot die
    M._bypassGodConn = RunService.Heartbeat:Connect(function()
        if not M.bypassAimbotEnabled then return end
        local char = player.Character
        if not char then return end
        local hum = char:FindFirstChildOfClass("Humanoid")
        local root = char:FindFirstChild("HumanoidRootPart")
        if not hum then return end
        pcall(function()
            hum.BreakJointsOnDeath = false
            hum:SetStateEnabled(Enum.HumanoidStateType.Dead, false)
            hum.MaxHealth = math.huge
            if hum.Health < hum.MaxHealth or hum.Health <= 0 then
                hum.Health = math.huge
            end
            hum.PlatformStand = false
            hum.Sit = false
            local st = hum:GetState()
            if st == Enum.HumanoidStateType.Dead
                or st == Enum.HumanoidStateType.Ragdoll
                or st == Enum.HumanoidStateType.FallingDown
                or st == Enum.HumanoidStateType.Physics then
                hum.Health = hum.MaxHealth
                hum:ChangeState(Enum.HumanoidStateType.Running)
            end
            -- void / fling safety while locked
            if root then
                local p = root.Position
                if p.Y < -40 or p ~= p or math.abs(p.X) > 1e5 or math.abs(p.Z) > 1e5 then
                    root.CFrame = CFrame.new(p.X, 30, p.Z)
                    root.AssemblyLinearVelocity = Vector3.zero
                    root.AssemblyAngularVelocity = Vector3.zero
                    hum.Health = hum.MaxHealth
                    hum:ChangeState(Enum.HumanoidStateType.Running)
                end
            end
        end)
    end)
end

function M.disableBypassGodmode()
    M._bypassClearGodConns()
end

function M.startBypassAimbot()
    if not M.safeModeTryStart() then return end
    if M.bypassAimbotConn then
        pcall(function() M.bypassAimbotConn:Disconnect() end)
        M.bypassAimbotConn = nil
    end
    if M._bypassRenderConn then
        pcall(function() M._bypassRenderConn:Disconnect() end)
        M._bypassRenderConn = nil
    end

    -- Mutual exclusive: TP Bat / Bypass OFFS ping lagger
    pcall(function()
        if M.pingActive and M.setPingActive then M.setPingActive(false, true) end
        M.pingActive = false
        M.pingLoopRunning = false
    end)
    -- Force auto-swing ON for TP Bat
    M.autoSwingEnabled = true
    M.tpBatAutoSwing = true

    -- Stop left/right & pause auto TP
    if M.autoLeftEnabled then
        M.autoLeftEnabled = false
        if M.autoLeftSetVisual then M.autoLeftSetVisual(false) end
        M.stopAutoLeft()
    end
    if M.autoRightEnabled then
        M.autoRightEnabled = false
        if M.autoRightSetVisual then M.autoRightSetVisual(false) end
        M.stopAutoRight()
    end

    M._autoTPWasEnabledForBypass = false
    if M.autoTPEnabled then
        M._autoTPWasEnabledForBypass = true
        M.stopAutoTP()
        if M.setAutoTPVisual then M.setAutoTPVisual(false) end
    end

    M.bypassAimbotEnabled = true
    M.enableBypassGodmode()
    M._bypassTarget = nil
    M._bypassSwingCooldown = false
    M._tpBatLastSwing = 0
    M._sureHitCD = false
    M._normalHitCD = false

    local char0 = player.Character
    if char0 then
        M._bypassHRP = char0:FindFirstChild("HumanoidRootPart")
        M._bypassHum = char0:FindFirstChildOfClass("Humanoid")
        if M._bypassHum then
            M.bypassPrevAutoRotate = M._bypassHum.AutoRotate
            M._bypassHum.AutoRotate = false
        end
    end

    local hitMode = (M.tpBatHitMode == "Normal") and "Normal" or "Sure"

    if hitMode == "Sure" then
        -- SURE HIT — camera/shiftlock aware (does NOT hard-lock camera)
        M.bypassAimbotConn = RunService.Heartbeat:Connect(function()
            if not M.bypassAimbotEnabled then return end
            local char = player.Character
            if not char then return end
            M._bypassHum = char:FindFirstChildOfClass("Humanoid")
            M._bypassHRP = char:FindFirstChild("HumanoidRootPart")
            if not M._bypassHum or not M._bypassHRP then return end

            local target = select(1, M._bypassGetClosest())
            if not (target and target.Character) then return end
            local tr = target.Character:FindFirstChild("HumanoidRootPart")
            if not tr then return end
            M._bypassTarget = tr

            if sethiddenproperty then
                pcall(function()
                    sethiddenproperty(M._bypassHRP, "PhysicsRepRootPart", tr)
                end)
            end

            -- Smooth place relative to YOUR look direction (shiftlock rotate works)
            M._bypassSmoothToTarget(tr)
            M._bypassTryHitBat()
        end)
        -- RenderStepped twin for smoother body follow + hit rate
        M._bypassRenderConn = RunService.RenderStepped:Connect(function()
            if not M.bypassAimbotEnabled then return end
            if not M._bypassHRP then return end
            local target = select(1, M._bypassGetClosest())
            if not (target and target.Character) then return end
            local tr = target.Character:FindFirstChild("HumanoidRootPart")
            if not tr then return end
            M._bypassSmoothToTarget(tr)
            M._bypassTryHitBat()
        end)
    else
        -- NORMAL HIT — same camera-aware placement
        M.bypassAimbotConn = RunService.Heartbeat:Connect(function()
            if not M.bypassAimbotEnabled then return end
            local char = player.Character
            if not char then return end
            M._bypassHum = char:FindFirstChildOfClass("Humanoid")
            M._bypassHRP = char:FindFirstChild("HumanoidRootPart")
            if not M._bypassHum or not M._bypassHRP then return end

            local target = select(1, M._bypassGetClosest())
            if not (target and target.Character) then return end
            local tr = target.Character:FindFirstChild("HumanoidRootPart")
            if not tr then return end
            M._bypassTarget = tr

            if sethiddenproperty then
                pcall(function()
                    sethiddenproperty(M._bypassHRP, "PhysicsRepRootPart", tr)
                end)
            end

            -- TP BAT NORMAL: teleporte direto para a posição atual do alvo.
            -- Não usa previsão, suavização ou velocidade antecipada.
            local targetPos = tr.Position + Vector3.new(0, 0.9, 0)
            if (M._bypassHRP.Position - targetPos).Magnitude > 8 then
                M._bypassHRP.CFrame = CFrame.new(targetPos)
            end
            local cam = workspace.CurrentCamera
            if cam then
                cam.CFrame = CFrame.new(cam.CFrame.Position, tr.Position)
            end
            M._bypassTryHitBatNormal()
        end)

    end

    if M.setBypassVisual then M.setBypassVisual(true) end
    if M.mobBtnRefs.bypass then M.mobBtnRefs.bypass(true) end
end

function M.stopBypassAimbot()
    if M.bypassAimbotConn then
        pcall(function() M.bypassAimbotConn:Disconnect() end)
        M.bypassAimbotConn = nil
    end
    if M._bypassRenderConn then
        pcall(function() M._bypassRenderConn:Disconnect() end)
        M._bypassRenderConn = nil
    end
    pcall(function() RunService:UnbindFromRenderStep("VynxBypassRotCam") end)
    M._bypassLookPos = nil

    M.bypassAimbotEnabled = false
    M.disableBypassGodmode()
    M._bypassTarget = nil
    M._bypassSwingCooldown = false
    M.bypassHitCD = false
    M._sureHitCD = false
    M._normalHitCD = false
    M._bypassHRP = nil
    M._bypassHum = nil

    local char = player.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if root then
        root.AssemblyLinearVelocity = Vector3.zero
        root.AssemblyAngularVelocity = Vector3.zero
    end

    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if hum then
        hum.AutoRotate = (M.bypassPrevAutoRotate == nil) and true or M.bypassPrevAutoRotate
    end

    if M._autoTPWasEnabledForBypass then
        M._autoTPWasEnabledForBypass = false
        M.autoTPEnabled = true
        if M.setAutoTPVisual then M.setAutoTPVisual(true) end
        M.startAutoTP()
    end

    if M.setBypassVisual then M.setBypassVisual(false) end
    if M.mobBtnRefs.bypass then M.mobBtnRefs.bypass(false) end
end

-- ============================================================
-- GREEN TP BAT OVERRIDE
-- Mantém normal bat/BAT V2 separados; substitui apenas TP Bat.
-- ============================================================
do
    local greenTpBatCooldown = false
    local function greenFindBat()
        local char = player.Character
        if not char then return nil end
        local tool = char:FindFirstChild("Bat")
        if tool and tool:IsA("Tool") then return tool end
        local bp = player:FindFirstChildOfClass("Backpack")
        if bp then
            tool = bp:FindFirstChild("Bat")
            if tool and tool:IsA("Tool") then
                local hum = char:FindFirstChildOfClass("Humanoid")
                if hum then pcall(function() hum:EquipTool(tool) end) end
                return tool
            end
        end
        for _, container in ipairs({char, bp}) do
            if container then
                for _, candidate in ipairs(container:GetChildren()) do
                    if candidate:IsA("Tool") and candidate.Name:lower():find("bat", 1, true) then
                        return candidate
                    end
                end
            end
        end
        return nil
    end

    local function greenTryHitBat()
        if greenTpBatCooldown then return end
        greenTpBatCooldown = true
        pcall(function()
            local bat = greenFindBat()
            if bat then
                bat:Activate()
                local ev = bat:FindFirstChildWhichIsA("RemoteEvent")
                if ev then ev:FireServer() end
                local rf = bat:FindFirstChildWhichIsA("RemoteFunction")
                if rf then pcall(function() rf:InvokeServer() end) end
                if M.notifyPlayerSwung then M.notifyPlayerSwung() end
            end
        end)
        task.delay(0.08, function() greenTpBatCooldown = false end)
    end

    local function greenClosestTarget(root)
        local closest, closestDist = nil, math.huge
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= player and plr.Character then
                local targetRoot = plr.Character:FindFirstChild("HumanoidRootPart")
                local targetHum = plr.Character:FindFirstChildOfClass("Humanoid")
                if targetRoot and targetHum and targetHum.Health > 0 then
                    local dist = (root.Position - targetRoot.Position).Magnitude
                    if dist < closestDist then
                        closest, closestDist = plr, dist
                    end
                end
            end
        end
        return closest
    end

    function M.startBypassAimbot()
        if M.bypassAimbotConn then M.bypassAimbotConn:Disconnect(); M.bypassAimbotConn=nil end
        if M._bypassRenderConn then M._bypassRenderConn:Disconnect(); M._bypassRenderConn=nil end
        M.bypassAimbotEnabled = true
        M._bypassTarget = nil
        M._bypassHRP = nil
        M._bypassHum = nil
        M._greenTpLastTarget = nil

        local char = player.Character
        if char then
            M._bypassHRP = char:FindFirstChild("HumanoidRootPart")
            M._bypassHum = char:FindFirstChildOfClass("Humanoid")
        end

        M.bypassAimbotConn = RunService.Heartbeat:Connect(function()
            if not M.bypassAimbotEnabled then return end
            local currentChar = player.Character
            local root = currentChar and currentChar:FindFirstChild("HumanoidRootPart")
            local hum = currentChar and currentChar:FindFirstChildOfClass("Humanoid")
            if not root or not hum or hum.Health <= 0 then return end
            M._bypassHRP, M._bypassHum = root, hum

            local target = greenClosestTarget(root)
            if not target or not target.Character then return end
            local targetRoot = target.Character:FindFirstChild("HumanoidRootPart")
            if not targetRoot then return end
            M._bypassTarget = targetRoot

            if sethiddenproperty then
                pcall(function() sethiddenproperty(root, "PhysicsRepRootPart", targetRoot) end)
            end

            local targetPos = targetRoot.Position + Vector3.new(0, 0.9, 0)
            if (root.Position - targetPos).Magnitude > 5 then
                root.CFrame = CFrame.new(targetPos)
            end
            local cam = workspace.CurrentCamera
            if cam then cam.CFrame = CFrame.new(cam.CFrame.Position, targetRoot.Position) end
            greenTryHitBat()
        end)

        if M.setBypassVisual then M.setBypassVisual(true) end
        if M.mobBtnRefs and M.mobBtnRefs.bypass then M.mobBtnRefs.bypass(true) end
    end

    function M.stopBypassAimbot()
        if M.bypassAimbotConn then M.bypassAimbotConn:Disconnect(); M.bypassAimbotConn=nil end
        if M._bypassRenderConn then M._bypassRenderConn:Disconnect(); M._bypassRenderConn=nil end
        M.bypassAimbotEnabled = false
        M._bypassTarget = nil
        M._bypassHRP = nil
        M._bypassHum = nil
        greenTpBatCooldown = false
        local char = player.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        if root then
            root.AssemblyLinearVelocity = Vector3.zero
            root.AssemblyAngularVelocity = Vector3.zero
        end
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        if hum then hum.AutoRotate = true end
        if M.setBypassVisual then M.setBypassVisual(false) end
        if M.mobBtnRefs and M.mobBtnRefs.bypass then M.mobBtnRefs.bypass(false) end
    end
end

function M.toggleBypassAimbot()
    M.bypassAimbotEnabled = not M.bypassAimbotEnabled
    if M.bypassAimbotEnabled then
        M.startBypassAimbot()
    else
        M.stopBypassAimbot()
    end
    if M.setBypassVisual then
        M.setBypassVisual(M.bypassAimbotEnabled)
    end
    if M.mobBtnRefs.bypass then
        M.mobBtnRefs.bypass(M.bypassAimbotEnabled)
    end
    saveCherryConfig()
    return M.bypassAimbotEnabled
end

-- ============================================================

-- REST OF CORE FUNCTIONS
-- ============================================================
function M.doAutoTPDown(force)
    local char=player.Character;if not char then return end;local hrp=char:FindFirstChild("HumanoidRootPart");if not hrp then return end
    local hum2=char:FindFirstChildOfClass("Humanoid");if not hum2 then return end
    if not force then if hum2.FloorMaterial~=Enum.Material.Air then return end;if not(hrp.Position.Y>=M.autoTPHeight) then return end end
    hrp.CFrame=CFrame.new(hrp.Position.X,-7.00,hrp.Position.Z)*CFrame.Angles(0,select(2,hrp.CFrame:ToEulerAnglesYXZ()),0);hrp.Velocity=Vector3.zero
end

function M.startAutoTP()
    if M.autoTPConn then task.cancel(M.autoTPConn);M.autoTPConn=nil end
    M.autoTPConn=task.spawn(function() while M.autoTPEnabled do task.wait(0.1);pcall(function() M.doAutoTPDown(false) end) end end)
end

function M.stopAutoTP() M.autoTPEnabled=false;if M.autoTPConn then task.cancel(M.autoTPConn);M.autoTPConn=nil end end

function M.runTPFloor() pcall(function() M.doAutoTPDown(true) end) end

function M.enableStretchRez()
    M.stretchRezEnabled=true;if M.stretchRezConn then M.stretchRezConn:Disconnect() end
    pcall(function() RunService:UnbindFromRenderStep("Movee_Stretch") end)
    pcall(function() RunService:BindToRenderStep("Movee_Stretch",Enum.RenderPriority.Last.Value-1,function() local cam=workspace.CurrentCamera;if cam then cam.CFrame=cam.CFrame*CFrame.new(0,0,0,1,0,0,0,0.8,0,0,0,1) end end) end)
end

function M.disableStretchRez() M.stretchRezEnabled=false;pcall(function() RunService:UnbindFromRenderStep("Movee_Stretch") end) end

--------------------------------------------------------------------------------
-- ANTI SUMMER BASE (ONLY remove blocking Anchor parts — never wipe bases)
--------------------------------------------------------------------------------
function M.isSummerBaseName(name)
    if not name then return false end
    local n = tostring(name):lower()
    -- strict: only explicit summer base names (not beach/palm/prop — those kill enemy bases)
    return n == "summerbase"
        or n == "summer_base"
        or n:find("summerbase", 1, true) ~= nil
        or n:find("summer_base", 1, true) ~= nil
end

function M.isAnchorName(name)
    if not name then return false end
    local n = tostring(name):lower()
    return n == "anchor" or n == "anchors"
end

function M.stripBlockingAnchor(obj)
    if not obj or not obj.Parent then return end
    local key = tostring(obj:GetFullName())
    if M._antiSummerCleaned[key] then return end
    M._antiSummerCleaned[key] = true
    pcall(function()
        if obj:IsA("BasePart") or obj:IsA("MeshPart") then
            obj.CanCollide = false
            obj.CanQuery = false
            obj.CanTouch = false
            obj.Transparency = 1
        end
        obj:Destroy()
    end)
end

function M.cleanSummerBaseAnchors()
    if not M.antiSummerBaseEnabled then return end
    local plots = workspace:FindFirstChild("Plots")
    if not plots then return end

    -- Only scan Plots (not whole workspace — was causing lag + wiping bases)
    for _, plot in ipairs(plots:GetChildren()) do
        local isSummer = M.isSummerBaseName(plot.Name)
        if not isSummer then
            for _, d in ipairs(plot:GetDescendants()) do
                if M.isSummerBaseName(d.Name) then
                    isSummer = true
                    break
                end
            end
        end
        if not isSummer then continue end
        -- ONLY strip objects literally named Anchor / Anchors
        for _, d in ipairs(plot:GetDescendants()) do
            if M.isAnchorName(d.Name) then
                M.stripBlockingAnchor(d)
            end
        end
    end
end

function M.enableAntiSummerBase()
    M.antiSummerBaseEnabled = true
    M._antiSummerCleaned = {}
    M.cleanSummerBaseAnchors()
    if M.antiSummerBaseConn then
        pcall(function() M.antiSummerBaseConn:Disconnect() end)
        M.antiSummerBaseConn = nil
    end
    M.antiSummerBaseConn = workspace.DescendantAdded:Connect(function(obj)
        if not M.antiSummerBaseEnabled then return end
        if not M.isAnchorName(obj.Name) then return end
        task.defer(function()
            if not M.antiSummerBaseEnabled or not obj.Parent then return end
            -- only if under Plots and near a summer-named container
            local p = obj
            local underPlots, nearSummer = false, false
            while p and p ~= workspace do
                if p.Name == "Plots" or (p.Parent and p.Parent.Name == "Plots") then underPlots = true end
                if M.isSummerBaseName(p.Name) then nearSummer = true end
                p = p.Parent
            end
            if underPlots and nearSummer then
                M.stripBlockingAnchor(obj)
            end
        end)
    end)
    task.spawn(function()
        while M.antiSummerBaseEnabled do
            M.cleanSummerBaseAnchors()
            task.wait(5) -- slower scan = less lag
        end
    end)
end

function M.disableAntiSummerBase()
    M.antiSummerBaseEnabled = false
    if M.antiSummerBaseConn then
        pcall(function() M.antiSummerBaseConn:Disconnect() end)
        M.antiSummerBaseConn = nil
    end
end

function M._isUnderPlots(obj)
    local p = obj
    while p and p ~= workspace do
        if p.Name == "Plots" then return true end
        p = p.Parent
    end
    return false
end

function M.applyAntiLagDerender(obj)
    if not obj or not obj.Parent then return end
    -- Preserve gameplay-critical bases and the ONI aura while optimizing everything else.
    if M._isUnderPlots(obj) then return end
    if M._isUnderWingsAura and M._isUnderWingsAura(obj) then return end
    pcall(function()
        if obj:IsA("Accessory") or obj:IsA("Hat") then
            local char = obj:FindFirstAncestorOfClass("Model")
            if char and Players:GetPlayerFromCharacter(char) then obj:Destroy() end
        elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Beam")
            or obj:IsA("Fire") or obj:IsA("Smoke") or obj:IsA("Sparkles") then
            obj.Enabled = false
        elseif obj:IsA("PointLight") or obj:IsA("SpotLight") or obj:IsA("SurfaceLight") then
            obj.Enabled = false
        elseif obj:IsA("Decal") or obj:IsA("Texture") then
            obj.Transparency = 1
        elseif obj:IsA("BasePart") or obj:IsA("MeshPart") then
            obj.CastShadow = false
            obj.Material = Enum.Material.Plastic
            obj.Reflectance = 0
            pcall(function() obj.MaterialVariant = "" end)
        end
    end)
end

function M.enableAntiLag()
    if M.antiLagEnabled and M.antiLagDescConn then return end
    M.removeAccessoriesEnabled = true
    M.antiLagEnabled = true
    M.defLightBrightness = M.defLightBrightness or Lighting.Brightness
    M.defLightClock = M.defLightClock or Lighting.ClockTime
    M.defLightAmbient = M.defLightAmbient or Lighting.OutdoorAmbient
    M.defLightGlobalShadows = M.defLightGlobalShadows == nil and Lighting.GlobalShadows or M.defLightGlobalShadows
    M.defLightFogEnd = M.defLightFogEnd or Lighting.FogEnd
    M.defLightDiffuse = M.defLightDiffuse or Lighting.EnvironmentDiffuseScale
    M.defLightSpecular = M.defLightSpecular or Lighting.EnvironmentSpecularScale
    Lighting.GlobalShadows = false
    Lighting.FogEnd = 1e10
    Lighting.Brightness = 1
    Lighting.EnvironmentDiffuseScale = 0
    Lighting.EnvironmentSpecularScale = 0
    -- Apply the FPS cap once; avoid a permanent Heartbeat loop.
    pcall(function()
        local gameSettings = UserSettings():GetService("UserGameSettings")
        if gameSettings then gameSettings:SetFpsCap(1000) end
    end)
    if M._skyConns then
        for _, c in ipairs(M._skyConns) do pcall(function() c:Disconnect() end) end
        M._skyConns = {}
    end
    M.currentSkyTheme = "Off"
    for _, e in ipairs(Lighting:GetChildren()) do
        pcall(function()
            if e:IsA("BlurEffect") or e:IsA("SunRaysEffect") or e:IsA("ColorCorrectionEffect")
                or e:IsA("BloomEffect") or e:IsA("DepthOfFieldEffect") then e.Enabled = false end
        end)
    end
    local terrain = workspace:FindFirstChildOfClass("Terrain")
    if terrain then
        M.defTerrainDecoration = M.defTerrainDecoration == nil and terrain.Decoration or M.defTerrainDecoration
        M.defWaterWaveSize = M.defWaterWaveSize or terrain.WaterWaveSize
        M.defWaterWaveSpeed = M.defWaterWaveSpeed or terrain.WaterWaveSpeed
        M.defWaterReflectance = M.defWaterReflectance or terrain.WaterReflectance
        M.defWaterTransparency = M.defWaterTransparency or terrain.WaterTransparency
        pcall(function()
            terrain.Decoration = false
            terrain.WaterWaveSize = 0
            terrain.WaterWaveSpeed = 0
            terrain.WaterReflectance = 0
            terrain.WaterTransparency = 1
        end)
    end
    if M.antiLagThread then pcall(function() task.cancel(M.antiLagThread) end) end
    M.antiLagThread = task.spawn(function()
        local snapshot = workspace:GetDescendants()
        for i, obj in ipairs(snapshot) do
            if not M.antiLagEnabled then return end
            M.applyAntiLagDerender(obj)
            if i % 250 == 0 then task.wait() end
        end
    end)
    if M.antiLagDescConn then M.antiLagDescConn:Disconnect() end
    M.antiLagDescConn = workspace.DescendantAdded:Connect(function(obj)
        if M.antiLagEnabled then task.defer(function() M.applyAntiLagDerender(obj) end) end
    end)
end

function M.disableAntiLag()
    M.removeAccessoriesEnabled = false
    M.antiLagEnabled = false
    if M.antiLagDescConn then M.antiLagDescConn:Disconnect(); M.antiLagDescConn = nil end
    if M.antiLagThread then pcall(function() task.cancel(M.antiLagThread) end); M.antiLagThread = nil end
    pcall(function()
        if M.defLightBrightness then Lighting.Brightness = M.defLightBrightness end
        if M.defLightClock then Lighting.ClockTime = M.defLightClock end
        if M.defLightAmbient then Lighting.OutdoorAmbient = M.defLightAmbient end
        if M.defLightFogEnd then Lighting.FogEnd = M.defLightFogEnd end
        if M.defLightDiffuse then Lighting.EnvironmentDiffuseScale = M.defLightDiffuse end
        if M.defLightSpecular then Lighting.EnvironmentSpecularScale = M.defLightSpecular end
        if M.defLightGlobalShadows ~= nil then Lighting.GlobalShadows = M.defLightGlobalShadows end
        local terrain = workspace:FindFirstChildOfClass("Terrain")
        if terrain then
            if M.defTerrainDecoration ~= nil then terrain.Decoration = M.defTerrainDecoration end
            if M.defWaterWaveSize then terrain.WaterWaveSize = M.defWaterWaveSize end
            if M.defWaterWaveSpeed then terrain.WaterWaveSpeed = M.defWaterWaveSpeed end
            if M.defWaterReflectance then terrain.WaterReflectance = M.defWaterReflectance end
            if M.defWaterTransparency then terrain.WaterTransparency = M.defWaterTransparency end
        end
        Lighting.ExposureCompensation = 0
    end)
end

-- ============================================================
-- NUKE OPTIMISER (Ace Mobile adaptation)
-- ============================================================
M.nukeOptimizerEnabled = M.nukeOptimizerEnabled or false
M._nukeOptimizerOn = false
M._nukeOptimizerConns = {}
M._nukeOptimizerThread = nil
function M._nukeSafeDestroy(obj)
    if not obj or not obj.Parent or obj.Name == "Overhead" then return end
    pcall(function() obj:Destroy() end)
end
function M._nukeIsCharacterPart(obj)
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr.Character and obj:IsDescendantOf(plr.Character) then return true end
    end
    return false
end
function M._nukeProcess(obj)
    if not M._nukeOptimizerOn or not obj or not obj.Parent or M._nukeIsCharacterPart(obj) then return end
    pcall(function()
        local name = string.lower(obj.Name or "")
        local isBase = name == "baseplate" or name == "spawnlocation" or name == "spawn location"
        if obj:IsA("Accessory") or obj:IsA("Hat") or obj:IsA("Shirt") or obj:IsA("Pants") or obj:IsA("ShirtGraphic") then
            M._nukeSafeDestroy(obj)
        elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Beam") or obj:IsA("Smoke") or obj:IsA("Fire") or obj:IsA("Sparkles") or obj:IsA("SurfaceAppearance") or obj:IsA("Decal") or obj:IsA("Texture") then
            M._nukeSafeDestroy(obj)
        elseif obj:IsA("PointLight") or obj:IsA("SpotLight") or obj:IsA("SurfaceLight") or obj:IsA("Explosion") then
            M._nukeSafeDestroy(obj)
        elseif obj:IsA("BasePart") then
            obj.CastShadow = false
            obj.Material = Enum.Material.Plastic
            obj.MaterialVariant = ""
            obj.Reflectance = 0
            if isBase then obj.Transparency = 1 end
        end
    end)
end
function M.enableNukeOptimizer()
    if M._nukeOptimizerOn then return end
    M._nukeOptimizerOn = true
    M.nukeOptimizerEnabled = true
    pcall(function()
        settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
        settings().Rendering.MeshPartDetailLevel = Enum.MeshPartDetailLevel.Level01
    end)
    pcall(function()
        Lighting.GlobalShadows = false
        Lighting.FogEnd = 1e10
        Lighting.FogStart = 1e10
        Lighting.EnvironmentDiffuseScale = 0
        Lighting.EnvironmentSpecularScale = 0
        for _, e in ipairs(Lighting:GetChildren()) do
            if e:IsA("BlurEffect") or e:IsA("SunRaysEffect") or e:IsA("ColorCorrectionEffect") or e:IsA("BloomEffect") or e:IsA("DepthOfFieldEffect") then e.Enabled = false end
        end
        local terrain = workspace:FindFirstChildOfClass("Terrain")
        if terrain then terrain.Decoration = false; terrain.WaterWaveSize = 0; terrain.WaterWaveSpeed = 0; terrain.WaterReflectance = 0; terrain.WaterTransparency = 1 end
    end)
    M._nukeOptimizerThread = task.spawn(function()
        if not game:IsLoaded() then game.Loaded:Wait() end
        local snapshot = workspace:GetDescendants()
        for i, obj in ipairs(snapshot) do
            if not M._nukeOptimizerOn then return end
            M._nukeProcess(obj)
            if i % 250 == 0 then task.wait() end
        end
    end)
    table.insert(M._nukeOptimizerConns, workspace.DescendantAdded:Connect(function(obj)
        if M._nukeOptimizerOn then task.defer(function() M._nukeProcess(obj) end) end
    end))
end
function M.disableNukeOptimizer()
    M._nukeOptimizerOn = false
    M.nukeOptimizerEnabled = false
    for _, c in ipairs(M._nukeOptimizerConns) do pcall(function() c:Disconnect() end) end
    M._nukeOptimizerConns = {}
    if M._nukeOptimizerThread then pcall(function() task.cancel(M._nukeOptimizerThread) end); M._nukeOptimizerThread = nil end
end
-- ============================================================
-- ANTI-RAGDOLL
-- ============================================================
M.antiRagdollNoSplatterCooldown = 0

function M.forceNoSplatterReset()
    local char = player.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")
    if not hum or not root or hum.Health <= 0 then return end

    pcall(function()
        hum:ChangeState(Enum.HumanoidStateType.GettingUp)
        root.Velocity = Vector3.zero
        root.RotVelocity = Vector3.zero
        root.AssemblyLinearVelocity = Vector3.zero
        root.AssemblyAngularVelocity = Vector3.zero

        for _, obj in ipairs(char:GetDescendants()) do
            if obj:IsA("Motor6D") then obj.Enabled = true end
            if obj:IsA("Constraint") then obj.Enabled = true end
        end

        workspace.CurrentCamera.CameraSubject = hum

        local PM = player.PlayerScripts:FindFirstChild("PlayerModule")
        if PM then
            local CM = require(PM:FindFirstChild("ControlModule"))
            if CM then CM:Enable() end
        end

        hum.AutoRotate = true
        hum.PlatformStand = false
        hum.Sit = false
    end)
end

function M.startAntiRagdoll()
    if M.Conns.antiRag then return end
    M.Conns.antiRag = RunService.Heartbeat:Connect(function()
        if not M.antiRagdollEnabled then return end
        local char = player.Character
        if not char then return end
        local hum = char:FindFirstChildOfClass("Humanoid")
        local root = char:FindFirstChild("HumanoidRootPart")
        if not hum or hum.Health <= 0 then return end

        local state = hum:GetState()
        local ragdolled = (state == Enum.HumanoidStateType.Physics or
                          state == Enum.HumanoidStateType.Ragdoll or
                          state == Enum.HumanoidStateType.FallingDown)

        if M.antiRagdollMode == "No Splatter" then
            if ragdolled then
                local now = tick()
                if now - (M.antiRagdollNoSplatterCooldown or 0) > 0.15 then
                    M.antiRagdollNoSplatterCooldown = now
                    M.forceNoSplatterReset()
                end
            end
            return
        end

        if not root then return end
        local endTime = player:GetAttribute("RagdollEndTime")
        if endTime and (endTime - workspace:GetServerTimeNow()) > 0 then
            ragdolled = true
        end
        if ragdolled then
            pcall(function()
                player:SetAttribute("RagdollEndTime", workspace:GetServerTimeNow())
            end)
            for _, d in ipairs(char:GetDescendants()) do
                if d:IsA("BallSocketConstraint") or
                   (d:IsA("Attachment") and d.Name:find("RagdollAttachment")) then
                    d:Destroy()
                end
            end
            for _, obj in ipairs(char:GetDescendants()) do
                if obj:IsA("Motor6D") and obj.Enabled == false then
                    obj.Enabled = true
                end
            end
            if hum.Health > 0 then
                hum:ChangeState(Enum.HumanoidStateType.Running)
            end
            workspace.CurrentCamera.CameraSubject = hum
            root.Anchored = false
            root.AssemblyLinearVelocity = Vector3.zero
            root.AssemblyAngularVelocity = Vector3.zero
        end
    end)
end

function M.stopAntiRagdoll()
    if M.Conns.antiRag then
        M.Conns.antiRag:Disconnect()
        M.Conns.antiRag = nil
    end
end

-- ============================================================
-- INFINITE JUMP (BodyVelocity, anti-TPBack / anti-kick safe)
-- ============================================================
M.jumpHeld = false
M.infJumpThread = nil
M._infJumpBoosting = false
M._infJumpLastBoost = 0
M.INF_JUMP_BOOST_FORCE = 25
M.INF_JUMP_BOOST_FRAMES = 2
M.INF_JUMP_BOOST_COOLDOWN = 0.12

local function M_applyInfJumpBoost(root)
    if not root or M._infJumpBoosting then return end
    local now = tick()
    if now - M._infJumpLastBoost < M.INF_JUMP_BOOST_COOLDOWN then return end
    M._infJumpLastBoost = now
    M._infJumpBoosting = true

    local bv = Instance.new("BodyVelocity")
    bv.MaxForce = Vector3.new(0, math.huge, 0)
    bv.P = 1250
    bv.Velocity = Vector3.new(root.Velocity.X, M.INF_JUMP_BOOST_FORCE, root.Velocity.Z)
    bv.Parent = root

    local frameCount = 0
    local conn
    conn = RunService.Heartbeat:Connect(function()
        if frameCount < M.INF_JUMP_BOOST_FRAMES then
            frameCount = frameCount + 1
            if bv and bv.Parent then
                bv.Velocity = bv.Velocity + Vector3.new(0, 0.01, 0)
            end
        else
            if bv then pcall(function() bv:Destroy() end) end
            if conn then conn:Disconnect() end
            M._infJumpBoosting = false
        end
    end)
end

task.spawn(function()
    local pg = player:WaitForChild("PlayerGui", 10)
    if pg then
        local function hookJumpButton(btn)
            if btn:IsA("GuiButton") and btn.Name == "JumpButton" and not btn:GetAttribute("InfJumpHooked") then
                btn:SetAttribute("InfJumpHooked", true)
                btn.MouseButton1Down:Connect(function()
                    if M.infJumpEnabled then
                        M.jumpHeld = true
                    end
                end)
                btn.MouseButton1Up:Connect(function() M.jumpHeld = false end)
                btn.MouseLeave:Connect(function() M.jumpHeld = false end)
            end
        end
        for _, d in ipairs(pg:GetDescendants()) do hookJumpButton(d) end
        pg.DescendantAdded:Connect(hookJumpButton)
    end
end)

UIS.JumpRequest:Connect(function()
    if M.infJumpEnabled and M.infJumpMode == "manual" then
        M.jumpHeld = true
        task.delay(0.08, function() M.jumpHeld = false end)
    end
end)

UIS.InputBegan:Connect(function(inp, gpe)
    if gpe then return end
    if M.infJumpEnabled
        and inp.UserInputType == Enum.UserInputType.Keyboard
        and inp.KeyCode == Enum.KeyCode.Space then
        M.jumpHeld = true
    end
end)

UIS.InputEnded:Connect(function(inp)
    if inp.UserInputType == Enum.UserInputType.Keyboard and inp.KeyCode == Enum.KeyCode.Space then
        M.jumpHeld = false
    end
end)

function M.startManualInfJumpLoop()
    if M.infJumpThread then M.infJumpThread:Disconnect() end
    M.infJumpThread = RunService.Heartbeat:Connect(function()
        if not M.infJumpEnabled or M.infJumpMode ~= "manual" then return end
        if not M.jumpHeld then return end
        local char = player.Character
        if not char then return end
        local hum = char:FindFirstChildOfClass("Humanoid")
        local root = char:FindFirstChild("HumanoidRootPart")
        if not hum or not root or hum.Health <= 0 then return end
        M_applyInfJumpBoost(root)
    end)
end

function M.stopManualInfJumpLoop()
    if M.infJumpThread then
        M.infJumpThread:Disconnect()
        M.infJumpThread = nil
    end
    M.jumpHeld = false
    M._infJumpBoosting = false
end

function M.startHoldInfJump()
    if M.holdInfJumpConn then M.holdInfJumpConn:Disconnect() end
    M.holdInfJumpConn = RunService.Heartbeat:Connect(function()
        if not M.infJumpEnabled or M.infJumpMode ~= "hold" then return end
        local char = player.Character
        if not char then return end
        local root = char:FindFirstChild("HumanoidRootPart")
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not root or not hum then return end
        -- Hold logic from original ONI: continuous Velocity boost while Space/Jump held
        local isJumpHeld = UIS:IsKeyDown(Enum.KeyCode.Space) or M.jumpHeld or (hum.Jump == true)
        local vel = root.AssemblyLinearVelocity
        if isJumpHeld and vel.Y < 35 then
            root.AssemblyLinearVelocity = Vector3.new(vel.X, 55, vel.Z)
        end
        -- Cap fall speed
        vel = root.AssemblyLinearVelocity
        if vel.Y < -120 then
            root.AssemblyLinearVelocity = Vector3.new(vel.X, -120, vel.Z)
        end
    end)
end

function M.stopHoldInfJump()
    if M.holdInfJumpConn then
        M.holdInfJumpConn:Disconnect()
        M.holdInfJumpConn = nil
    end
end

-- ============================================================
function M.startUnwalk()
    local c=player.Character;if not c then return end;local hum=c:FindFirstChildOfClass("Humanoid")
    if hum then for _,t in ipairs(hum:GetPlayingAnimationTracks()) do t:Stop() end end
    local anim=c:FindFirstChild("Animate");if anim then M.unwalkSavedAnimate=anim:Clone();anim:Destroy() end
end

function M.stopUnwalk() local c=player.Character;if c and M.unwalkSavedAnimate then M.unwalkSavedAnimate:Clone().Parent=c;M.unwalkSavedAnimate=nil end end

function M.cursedInstaReset()
    if M._instaResetBusy then return end

    local character = player.Character
    local humanoid = character and character:FindFirstChildOfClass("Humanoid")
    local hrp = character and (character:FindFirstChild("HumanoidRootPart") or character:FindFirstChild("UpperTorso"))
    if not character or not hrp or not hrp:IsDescendantOf(character) then return end
    if humanoid and humanoid.Health <= 0 then return end

    M._instaResetBusy = true
    M._aceResetToken = (M._aceResetToken or 0) + 1
    local resetToken = M._aceResetToken

    local camera = workspace.CurrentCamera
    if camera then
        camera.CameraType = Enum.CameraType.Scriptable
        camera.CFrame = CFrame.new(
            -337.938599, -0.585044861, 106.739204,
            0.133411571, -0.379638135, 0.915465117,
            0, 0.923722506, 0.383062422,
            -0.991060734, -0.0511049591, 0.123235278
        )
        camera.Focus = CFrame.new(
            -349.381927, -5.37332535, 105.198761,
            1, 0, 0,
            0, 1, 0,
            0, 0, 1
        )
    end

    if humanoid then
        humanoid.BreakJointsOnDeath = true
        humanoid.PlatformStand = true
        humanoid:ChangeState(Enum.HumanoidStateType.Physics)
    end

    hrp.AssemblyAngularVelocity = Vector3.zero
    hrp.AssemblyLinearVelocity = Vector3.new(0, 1000000, 0)

    task.delay(0.1, function()
        if resetToken ~= M._aceResetToken then return end
        if camera and camera.Parent then
            camera.CameraType = Enum.CameraType.Custom
            if humanoid and humanoid.Parent then camera.CameraSubject = humanoid end
        end
    end)

    task.delay(0.5, function()
        if resetToken == M._aceResetToken then
            M._instaResetBusy = false
        end
    end)
end

function M.hasBrainrotInHand()
    local char = player.Character
    if not char then return false end
    for _, item in ipairs(char:GetChildren()) do
        if item:IsA("Tool") then
            local name = item.Name:lower()
            if name:find("brainrot", 1, true) or name:find("skibidi", 1, true) or name:find("toilet", 1, true) then
                return true
            end
        end
    end
    return false
end

function M.forceLaggerCarryWhileHolding()
    if not M.hasBrainrotInHand() then return false end
    M.carrySpeedActive = false
    M.laggerModeEnabled = false
    M.laggerCarryActive = true
    return true
end

function M.toggleCarryMode()
    if M.forceLaggerCarryWhileHolding() then
        M.refreshSpeedModeLabel()
        if M.mobBtnRefs.carrySpeed then M.mobBtnRefs.carrySpeed(false) end
        if M.mobBtnRefs.lagger then M.mobBtnRefs.lagger(false) end
        if M.mobBtnRefs.laggerCarry then M.mobBtnRefs.laggerCarry(true) end
        if M.carryModeBtn then M.carryModeBtn.Text = "Carry Off" end
        if M.laggerModeBtn then M.laggerModeBtn.Text = "Lag Off" end
        if M.laggerCarryBtn then M.laggerCarryBtn.Text = "L.Carry On" end
        saveCherryConfig()
        return
    end
    -- Lagger Speed has priority over Carry.
    -- If it is active, keep Carry disabled instead of allowing both modes at once.
    if M.laggerModeEnabled then
        M.carrySpeedActive = false
        M.refreshSpeedModeLabel()
        if M.mobBtnRefs.carrySpeed then M.mobBtnRefs.carrySpeed(false) end
        if M.carryModeBtn then M.carryModeBtn.Text = "Carry Off" end
        saveCherryConfig()
        return
    end

    M.carrySpeedActive = not M.carrySpeedActive
    if M.carrySpeedActive then M.laggerCarryActive = false end
    M.refreshSpeedModeLabel()
    if M.mobBtnRefs.carrySpeed then M.mobBtnRefs.carrySpeed(M.carrySpeedActive) end
    if M.mobBtnRefs.laggerCarry then M.mobBtnRefs.laggerCarry(M.laggerCarryActive) end
    if M.carryModeBtn then
        M.carryModeBtn.Text = M.carrySpeedActive and "Carry On" or "Carry Off"
    end
    if M.laggerCarryBtn then
        M.laggerCarryBtn.Text = M.laggerCarryActive and "L.Carry On" or "L.Carry Off"
    end
    saveCherryConfig()
end

function M.toggleLaggerMode()
    if M.forceLaggerCarryWhileHolding() then
        M.refreshSpeedModeLabel()
        if M.mobBtnRefs.lagger then M.mobBtnRefs.lagger(false) end
        if M.mobBtnRefs.laggerCarry then M.mobBtnRefs.laggerCarry(true) end
        if M.laggerModeBtn then M.laggerModeBtn.Text = "Lag Off" end
        if M.laggerCarryBtn then M.laggerCarryBtn.Text = "L.Carry On" end
        saveCherryConfig()
        return
    end
    M.laggerModeEnabled = not M.laggerModeEnabled
    if M.laggerModeEnabled then
        -- Lagger Speed takes priority and turns Carry off.
        M.laggerCarryActive = false
        M.carrySpeedActive = false
    end
    M.refreshSpeedModeLabel()
    if M.mobBtnRefs.lagger then M.mobBtnRefs.lagger(M.laggerModeEnabled) end
    if M.mobBtnRefs.laggerCarry then M.mobBtnRefs.laggerCarry(M.laggerCarryActive) end
    if M.laggerModeBtn then
        M.laggerModeBtn.Text = M.laggerModeEnabled and "Lag On" or "Lag Off"
    end
    if M.laggerCarryBtn then
        M.laggerCarryBtn.Text = M.laggerCarryActive and "L.Carry On" or "L.Carry Off"
    end
    saveCherryConfig()
end

function M.cycleLaggerModeBind()
    -- A keybind funciona como um override temporário:
    -- ativa Lagger por cima do modo atual e restaura esse modo ao desligar.
    if not M.laggerModeEnabled then
        M._laggerPreviousMode = {
            carrySpeedActive = M.carrySpeedActive == true,
            laggerCarryActive = M.laggerCarryActive == true,
        }
        M.carrySpeedActive = false
        M.laggerCarryActive = false
        M.laggerModeEnabled = true
    else
        local previous = M._laggerPreviousMode or {
            carrySpeedActive = false,
            laggerCarryActive = false,
        }
        M.laggerModeEnabled = false
        M.laggerCarryActive = previous.laggerCarryActive == true
        M.carrySpeedActive = previous.carrySpeedActive == true
        M._laggerPreviousMode = nil
    end

    M.refreshSpeedModeLabel()
    if M.mobBtnRefs.carrySpeed then M.mobBtnRefs.carrySpeed(M.carrySpeedActive) end
    if M.mobBtnRefs.lagger then M.mobBtnRefs.lagger(M.laggerModeEnabled) end
    if M.mobBtnRefs.laggerCarry then M.mobBtnRefs.laggerCarry(M.laggerCarryActive) end
    if M.carryModeBtn then M.carryModeBtn.Text = M.carrySpeedActive and "Carry On" or "Carry Off" end
    if M.laggerModeBtn then M.laggerModeBtn.Text = M.laggerModeEnabled and "Lag On" or "Lag Off" end
    if M.laggerCarryBtn then M.laggerCarryBtn.Text = M.laggerCarryActive and "L.Carry On" or "L.Carry Off" end

    -- O estado temporário não é salvo durante o Lagger ativo.
    if not M.laggerModeEnabled then saveCherryConfig() end
end

function M.toggleLaggerCarry()
    M.laggerCarryActive = not M.laggerCarryActive
    if M.laggerCarryActive then
        M.laggerModeEnabled = false
        M.carrySpeedActive = false
    end
    M.refreshSpeedModeLabel()
    if M.mobBtnRefs.carrySpeed then M.mobBtnRefs.carrySpeed(M.carrySpeedActive) end
    if M.mobBtnRefs.lagger then M.mobBtnRefs.lagger(M.laggerModeEnabled) end
    if M.mobBtnRefs.laggerCarry then M.mobBtnRefs.laggerCarry(M.laggerCarryActive) end
    if M.laggerModeBtn then
        M.laggerModeBtn.Text = M.laggerModeEnabled and "Lag On" or "Lag Off"
    end
    if M.carryModeBtn then
        M.carryModeBtn.Text = M.carrySpeedActive and "Carry On" or "Carry Off"
    end
    if M.laggerCarryBtn then
        M.laggerCarryBtn.Text = M.laggerCarryActive and "L.Carry On" or "L.Carry Off"
    end
    saveCherryConfig()
end

function M.stopAutoLeft()
    M.autoLeftEnabled = false
    if M._autoPlayStealingConn then pcall(function() M._autoPlayStealingConn:Disconnect() end); M._autoPlayStealingConn = nil end
    M._autoPlayStealingReady = false
    M.destroyAutoPlayProxy()
    if M.alConn then M.alConn:Disconnect(); M.alConn = nil end
    M.alPhase = 1
    local char = player.Character
    if char then
        local h = char:FindFirstChildOfClass("Humanoid")
        if h then h:Move(Vector3.zero, false) end
    end
    if M.autoLeftSetVisual then M.autoLeftSetVisual(false) end
    if M.mobBtnRefs.autoLeft then M.mobBtnRefs.autoLeft(false) end
end

function M.stopAutoRight()
    M.autoRightEnabled = false
    if M._autoPlayStealingConn then pcall(function() M._autoPlayStealingConn:Disconnect() end); M._autoPlayStealingConn = nil end
    M._autoPlayStealingReady = false
    M.destroyAutoPlayProxy()
    if M.arConn then M.arConn:Disconnect(); M.arConn = nil end
    M.arPhase = 1
    local char = player.Character
    if char then
        local h = char:FindFirstChildOfClass("Humanoid")
        if h then h:Move(Vector3.zero, false) end
    end
    if M.autoRightSetVisual then M.autoRightSetVisual(false) end
    if M.mobBtnRefs.autoRight then M.mobBtnRefs.autoRight(false) end
end

-- Auto Play proxy movement: keeps the original ONI speed values while
-- applying direct horizontal velocity like the supplied Auto Play reference.
M._autoPlayProxy = nil

function M.ensureAutoPlayProxy(char, hrp)
    if not char or not hrp then return nil end
    if M._autoPlayProxy and M._autoPlayProxy.Parent == char then return M._autoPlayProxy end
    if M._autoPlayProxy then pcall(function() M._autoPlayProxy:Destroy() end) end
    local proxy = Instance.new("Part")
    proxy.Name = "ONI_AutoPlayProxy"
    proxy.Size = Vector3.new(1, 1, 1)
    proxy.Transparency = 1
    proxy.CanCollide = false
    proxy.CanTouch = false
    proxy.CanQuery = false
    proxy.Massless = true
    proxy.Parent = char
    local weld = Instance.new("Weld")
    weld.Part0 = hrp
    weld.Part1 = proxy
    weld.C0 = CFrame.new()
    weld.Parent = proxy
    M._autoPlayProxy = proxy
    return proxy
end

function M.destroyAutoPlayProxy()
    if M._autoPlayProxy then
        pcall(function() M._autoPlayProxy:Destroy() end)
        M._autoPlayProxy = nil
    end
end

function M.moveAutoPlayProxy(target, speed)
    local char = player.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if not hrp or not hum or not target then return false end

    local dir = target - hrp.Position
    local flat = Vector3.new(dir.X, 0, dir.Z)
    local spd = math.clamp(tonumber(speed) or tonumber(M.getAutoPathSpeed and M.getAutoPathSpeed()) or tonumber(M.NS) or 60, 1, 500)

    if flat.Magnitude < 0.01 then
        hum:Move(Vector3.zero, false)
        local current = hrp.AssemblyLinearVelocity
        hrp.AssemblyLinearVelocity = Vector3.new(0, current.Y, 0)
        return true
    end

    local moveDir = flat.Unit
    hum:Move(moveDir, false)
    if M.applyVelocitySpeed then
        M.applyVelocitySpeed(spd, moveDir)
    end
    return false
end

-- Auto Play gate: after L2/R2, continue only when Stealing becomes true.
M._autoPlayStealingReady = false
M._autoPlayStealingConn = nil
function M.beginAutoPlayStealingGate()
    if M._autoPlayStealingConn then
        pcall(function() M._autoPlayStealingConn:Disconnect() end)
        M._autoPlayStealingConn = nil
    end
    M._autoPlayStealingReady = player:GetAttribute("Stealing") == true
    if not M._autoPlayStealingReady then
        M._autoPlayStealingConn = player:GetAttributeChangedSignal("Stealing"):Connect(function()
            if player:GetAttribute("Stealing") == true then
                M._autoPlayStealingReady = true
                if M._autoPlayStealingConn then
                    pcall(function() M._autoPlayStealingConn:Disconnect() end)
                    M._autoPlayStealingConn = nil
                end
            end
        end)
    end
end

-- Original fixed-path Auto Left / Right (as before)
function M.startAutoLeftFull()
    if M.alConn then M.alConn:Disconnect() end
    if M.arConn then M.arConn:Disconnect(); M.arConn=nil end
    M.destroyAutoPlayProxy()
    M.autoLeftEnabled=true
    M.alPhase=1
    M._fullALIndex=1
    M._fullALPauseUntil=nil
    M._fullALWaitingForSteal=false
    M.alConn=RunService.Stepped:Connect(function()
        if not M.autoLeftEnabled then return end
        local char=player.Character
        local hrp=char and char:FindFirstChild("HumanoidRootPart")
        if not hrp then return end
        local target=M._fullAutoLeftWaypoints[M._fullALIndex]
        if not target then return end
        if M._fullALWaitingForSteal then
            if not M._autoPlayStealingReady then
                M.moveAutoPlayProxy(hrp.Position, 0)
                return
            end
            M._fullALWaitingForSteal=false
        end
        local speed=M.getAutoPathSpeed and M.getAutoPathSpeed(M._fullALIndex >= 3) or M.NS or 60
        if Vector3.new(target.X - hrp.Position.X, 0, target.Z - hrp.Position.Z).Magnitude < 2.5 then
            M._fullALIndex=M._fullALIndex+1
            if M._fullALIndex > #M._fullAutoLeftWaypoints then M.stopAutoLeft(); return end
            if M._fullALIndex == 3 then
                M.beginAutoPlayStealingGate()
                M._fullALWaitingForSteal=true
            end
        else
            M.moveAutoPlayProxy(target,speed)
        end
    end)
end

function M.startAutoRightFull()
    if M.arConn then M.arConn:Disconnect() end
    if M.alConn then M.alConn:Disconnect(); M.alConn=nil end
    M.destroyAutoPlayProxy()
    M.autoRightEnabled=true
    M.arPhase=1
    M._fullARIndex=1
    M._fullARPauseUntil=nil
    M._fullARWaitingForSteal=false
    M.arConn=RunService.Stepped:Connect(function()
        if not M.autoRightEnabled then return end
        local char=player.Character
        local hrp=char and char:FindFirstChild("HumanoidRootPart")
        if not hrp then return end
        local target=M._fullAutoRightWaypoints[M._fullARIndex]
        if not target then return end
        if M._fullARWaitingForSteal then
            if not M._autoPlayStealingReady then
                M.moveAutoPlayProxy(hrp.Position, 0)
                return
            end
            M._fullARWaitingForSteal=false
        end
        local speed=M.getAutoPathSpeed and M.getAutoPathSpeed(M._fullARIndex >= 3) or M.NS or 60
        if Vector3.new(target.X - hrp.Position.X, 0, target.Z - hrp.Position.Z).Magnitude < 2.5 then
            M._fullARIndex=M._fullARIndex+1
            if M._fullARIndex > #M._fullAutoRightWaypoints then M.stopAutoRight(); return end
            if M._fullARIndex == 3 then
                M.beginAutoPlayStealingGate()
                M._fullARWaitingForSteal=true
            end
        else
            M.moveAutoPlayProxy(target,speed)
        end
    end)
end

function M.startAutoLeftSemi()
    if M.alConn then M.alConn:Disconnect() end
    if M.arConn then M.arConn:Disconnect(); M.arConn = nil end
    pcall(M.destroySpeedObjects)
    M.alPhase = 1
    M.autoLeftEnabled = true
    M.alConn = RunService.Heartbeat:Connect(function()
        if not M.autoLeftEnabled then return end
        local char = player.Character
        if not char then return end
        local hrp = char:FindFirstChild("HumanoidRootPart")
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not hrp or not hum then return end
        local spd = (M.getAutoPathSpeed and M.getAutoPathSpeed()) or (M.NS or 60)
        if M.alPhase == 1 then
            local tgt = Vector3.new(M.AP_L1.X, hrp.Position.Y, M.AP_L1.Z)
            if (Vector3.new(tgt.X - hrp.Position.X, 0, tgt.Z - hrp.Position.Z)).Magnitude < 1 then
                M.alPhase = 2
                local d = M.AP_L2 - hrp.Position
                local mv = Vector3.new(d.X, 0, d.Z)
                if mv.Magnitude > 0.01 then mv = mv.Unit end
                hum:Move(mv, false)
                if M.applyVelocitySpeed then M.applyVelocitySpeed(spd, mv) end
                return
            end
            local d = M.AP_L1 - hrp.Position
            local mv = Vector3.new(d.X, 0, d.Z)
            if mv.Magnitude > 0.01 then mv = mv.Unit end
            hum:Move(mv, false)
            if M.applyVelocitySpeed then M.applyVelocitySpeed(spd, mv) end
        elseif M.alPhase == 2 then
            local tgt = Vector3.new(M.AP_L2.X, hrp.Position.Y, M.AP_L2.Z)
            if (Vector3.new(tgt.X - hrp.Position.X, 0, tgt.Z - hrp.Position.Z)).Magnitude < 1 then
                hum:Move(Vector3.zero, false)
                hrp.AssemblyLinearVelocity = Vector3.zero
                M.autoLeftEnabled = false
                if M.alConn then M.alConn:Disconnect(); M.alConn = nil end
                M.alPhase = 1
                if M.autoLeftSetVisual then M.autoLeftSetVisual(false) end
                if M.mobBtnRefs.autoLeft then M.mobBtnRefs.autoLeft(false) end
                return
            end
            local d = M.AP_L2 - hrp.Position
            local mv = Vector3.new(d.X, 0, d.Z)
            if mv.Magnitude > 0.01 then mv = mv.Unit end
            hum:Move(mv, false)
            if M.applyVelocitySpeed then M.applyVelocitySpeed(spd, mv) end
        end
        if M.autoMoveSwingEnabled and not M._alSwingDebounce then
            M._alSwingDebounce = true
            local bat = M.findBat and M.findBat() or (M.findBatForAimbot and M.findBatForAimbot())
            if bat then
                if bat.Parent ~= char then pcall(function() hum:EquipTool(bat) end) end
                pcall(function() bat:Activate() end)
            end
            task.delay(M.autoMoveSwingInterval or 0.3, function() M._alSwingDebounce = false end)
        end
    end)
end

function M.startAutoRightSemi()
    if M.arConn then M.arConn:Disconnect() end
    if M.alConn then M.alConn:Disconnect(); M.alConn = nil end
    pcall(M.destroySpeedObjects)
    M.arPhase = 1
    M.autoRightEnabled = true
    M.arConn = RunService.Heartbeat:Connect(function()
        if not M.autoRightEnabled then return end
        local char = player.Character
        if not char then return end
        local hrp = char:FindFirstChild("HumanoidRootPart")
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not hrp or not hum then return end
        local spd = (M.getAutoPathSpeed and M.getAutoPathSpeed()) or (M.NS or 60)
        if M.arPhase == 1 then
            local tgt = Vector3.new(M.AP_R1.X, hrp.Position.Y, M.AP_R1.Z)
            if (Vector3.new(tgt.X - hrp.Position.X, 0, tgt.Z - hrp.Position.Z)).Magnitude < 1 then
                M.arPhase = 2
                local d = M.AP_R2 - hrp.Position
                local mv = Vector3.new(d.X, 0, d.Z)
                if mv.Magnitude > 0.01 then mv = mv.Unit end
                hum:Move(mv, false)
                if M.applyVelocitySpeed then M.applyVelocitySpeed(spd, mv) end
                return
            end
            local d = M.AP_R1 - hrp.Position
            local mv = Vector3.new(d.X, 0, d.Z)
            if mv.Magnitude > 0.01 then mv = mv.Unit end
            hum:Move(mv, false)
            if M.applyVelocitySpeed then M.applyVelocitySpeed(spd, mv) end
        elseif M.arPhase == 2 then
            local tgt = Vector3.new(M.AP_R2.X, hrp.Position.Y, M.AP_R2.Z)
            if (Vector3.new(tgt.X - hrp.Position.X, 0, tgt.Z - hrp.Position.Z)).Magnitude < 1 then
                hum:Move(Vector3.zero, false)
                hrp.AssemblyLinearVelocity = Vector3.zero
                M.autoRightEnabled = false
                if M.arConn then M.arConn:Disconnect(); M.arConn = nil end
                M.arPhase = 1
                if M.autoRightSetVisual then M.autoRightSetVisual(false) end
                if M.mobBtnRefs.autoRight then M.mobBtnRefs.autoRight(false) end
                return
            end
            local d = M.AP_R2 - hrp.Position
            local mv = Vector3.new(d.X, 0, d.Z)
            if mv.Magnitude > 0.01 then mv = mv.Unit end
            hum:Move(mv, false)
            if M.applyVelocitySpeed then M.applyVelocitySpeed(spd, mv) end
        end
        if M.autoMoveSwingEnabled and not M._arSwingDebounce then
            M._arSwingDebounce = true
            local bat = M.findBat and M.findBat() or (M.findBatForAimbot and M.findBatForAimbot())
            if bat then
                if bat.Parent ~= char then pcall(function() hum:EquipTool(bat) end) end
                pcall(function() bat:Activate() end)
            end
            task.delay(M.autoMoveSwingInterval or 0.3, function() M._arSwingDebounce = false end)
        end
    end)
end

function M.startAutoLeft()
    if M.autoPlayMode == "Full" then M.startAutoLeftFull() else M.startAutoLeftSemi() end
end

function M.startAutoRight()
    if M.autoPlayMode == "Full" then M.startAutoRightFull() else M.startAutoRightSemi() end
end

function M.enableAntiKick()
    M.antiKickEnabled = true
    task.spawn(function()
        while M.antiKickEnabled do
            task.wait(0.5)
            local char = player.Character
            if char then
                local found = false
                for _, tool in ipairs(char:GetChildren()) do
                    if tool:IsA("Tool") then
                        local n = tool.Name:lower()
                        if n:find("brainrot") or n:find("skibidi") or n:find("toilet") then
                            found = true
                            break
                        end
                    end
                end
                M.brainrotDetected = found
                if found then
                    if M.autoBatEnabled then M.stopBatAimbot() end
                    if M.autoLeftEnabled then M.autoLeftEnabled=false; if M.autoLeftSetVisual then M.autoLeftSetVisual(false) end; M.stopAutoLeft() end
                    if M.autoRightEnabled then M.autoRightEnabled=false; if M.autoRightSetVisual then M.autoRightSetVisual(false) end; M.stopAutoRight() end
                end
            end
        end
    end)
end

function M.disableAntiKick()
    M.antiKickEnabled = false
    M.brainrotDetected = false
end

--------------------------------------------------------------------------------
-- SAFE MODE (locks combat during duel countdown / while holding brainrot)
--------------------------------------------------------------------------------
function M.safeModeGetCountdownLabel()
    local ok, label = pcall(function()
        local pg = player:FindFirstChild("PlayerGui")
        if not pg then return nil end
        local top = pg:FindFirstChild("DuelsMachineTopFrame")
        if not top then return nil end
        local inner = top:FindFirstChild("DuelsMachineTopFrame")
        if not inner then return nil end
        local timer = inner:FindFirstChild("Timer")
        if not timer then return nil end
        return timer:FindFirstChild("Label")
    end)
    return (ok and label) or nil
end

function M.safeModeCountdownNumber(text)
    local t = tostring(text or ""):upper():gsub("^%s+", ""):gsub("%s+$", "")
    if t == "GO" or t == "START" or t == "READY" then return true end
    local n = tonumber(t)
    return n ~= nil and n >= 0 and n <= 10
end

function M.safeModeInDuelCountdown()
    local label = M.safeModeGetCountdownLabel()
    return label and M.safeModeCountdownNumber(label.Text) or false
end

M.SAFE_MODE_BLOCKED_TOOLS = {
    bat=true, slap=true, sword=true, gun=true, pistol=true, rifle=true,
    medusa=true, hammer=true, axe=true, knife=true, katana=true, blade=true, fist=true,
}

function M.safeModeIsCarryableTool(tool)
    if not tool or not tool:IsA("Tool") then return false end
    local name = tool.Name:lower()
    for word in pairs(M.SAFE_MODE_BLOCKED_TOOLS) do
        if name:find(word, 1, true) then return false end
    end
    return true
end

function M.safeModeHoldingBrainrot()
    local ok, val = pcall(function() return player:GetAttribute("Stealing") end)
    if ok and val == true then return true end
    local ok2, val2 = pcall(function() return player:GetAttribute("AntiKick") end)
    if ok2 and val2 == true then return true end
    local char = player.Character
    if not char then return false end
    local ok3, val3 = pcall(function() return char:GetAttribute("Stealing") end)
    if ok3 and val3 == true then return true end
    if M.brainrotDetected then return true end
    if M.hasBrainrotInHand and M.hasBrainrotInHand() then return true end
    for _, name in ipairs({"Carrying", "IsCarrying", "Grabbed", "Holding", "StealHold", "HasGrab"}) do
        local v = char:FindFirstChild(name, true)
        if v then
            if v:IsA("BoolValue") and v.Value then return true end
            if v:IsA("ObjectValue") and v.Value then return true end
            if v:IsA("StringValue") and v.Value ~= "" then return true end
        end
    end
    for _, child in ipairs(char:GetChildren()) do
        if child:IsA("Model") and child:FindFirstChildWhichIsA("BasePart", true) then
            local n = child.Name:lower()
            if n:find("brainrot") or n:find("animal") or n:find("carry") or n:find("grab") or n:find("steal") or n:find("hold") then
                return true
            end
        end
    end
    return false
end

function M.safeModeIsLocked()
    if not M.safeModeEnabled then return false end
    return M.safeModeInDuelCountdown() or M.safeModeHoldingBrainrot()
end

function M.safeModeForceStop(reason)
    local stopped = false
    if M.autoBatEnabled then
        M.stopBatAimbot()
        stopped = true
    end
    if M.bypassAimbotEnabled then
        M.stopBypassAimbot()
        stopped = true
    end
    if M.autoLeftEnabled then
        M.autoLeftEnabled = false
        if M.autoLeftSetVisual then M.autoLeftSetVisual(false) end
        M.stopAutoLeft()
        stopped = true
    end
    if M.autoRightEnabled then
        M.autoRightEnabled = false
        if M.autoRightSetVisual then M.autoRightSetVisual(false) end
        M.stopAutoRight()
        stopped = true
    end
    if stopped then
        -- optional toast; silent if no notifier
        pcall(function()
            if type(showActionNotification) == "function" then
                showActionNotification(reason or "SAFE MODE LOCK")
            end
        end)
    end
end

function M.safeModeTryStart()
    if M.safeModeIsLocked() then
        M.safeModeForceStop("SAFE MODE LOCK")
        return false
    end
    return true
end

function M.enableSafeMode()
    M.safeModeEnabled = true
end

function M.disableSafeMode()
    M.safeModeEnabled = false
end

if not M._safeModeMonitorStarted then
    M._safeModeMonitorStarted = true
    RunService.Heartbeat:Connect(function()
        if M.safeModeEnabled and M.safeModeIsLocked() then
            M.safeModeForceStop("SAFE MODE LOCK")
        end
    end)
end

--------------------------------------------------------------------------------
-- MIRROR TP DOWN (teleport down when opponent drops while aimbot is on)
--------------------------------------------------------------------------------
function M.mirrorTPAimbotActive()
    return M.autoBatEnabled == true or M.bypassAimbotEnabled == true
end

function M.mirrorTPTeleportDown()
    local char = player.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    local humanoid = char and char:FindFirstChildOfClass("Humanoid")
    if not root or not humanoid or humanoid.Health <= 0 then return end
    local now = tick()
    if now - (M.mirrorTPLastTeleport or 0) < 0.08 then return end
    M.mirrorTPLastTeleport = now
    local _, yaw = root.CFrame:ToEulerAnglesYXZ()
    local y = (M.MIRROR_TP_DOWN_Y or -7) + (math.random() * 0.6 - 0.3)
    root.CFrame = CFrame.new(root.Position.X, y, root.Position.Z) * CFrame.Angles(0, yaw, 0)
    root.AssemblyLinearVelocity = Vector3.new((math.random()-0.5)*0.4, 0, (math.random()-0.5)*0.4)
end

if not M._mirrorTPStarted then
    M._mirrorTPStarted = true
    RunService.Heartbeat:Connect(function()
        if not M.mirrorTPDownEnabled or not M.mirrorTPAimbotActive() then
            if next(M.mirrorTPPreviousY) then
                table.clear(M.mirrorTPPreviousY)
            end
            return
        end
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= player and plr.Character then
                local root = plr.Character:FindFirstChild("HumanoidRootPart")
                if root then
                    local currentY = root.Position.Y
                    local previousY = M.mirrorTPPreviousY[plr.UserId]
                    if previousY and previousY - currentY >= (M.MIRROR_TP_DROP_THRESHOLD or 3) then
                        pcall(M.mirrorTPTeleportDown)
                        table.clear(M.mirrorTPPreviousY)
                        return
                    end
                    M.mirrorTPPreviousY[plr.UserId] = currentY
                end
            end
        end
    end)
end

function M.setMirrorTPDown(enabled)
    M.mirrorTPDownEnabled = enabled == true
    if not M.mirrorTPDownEnabled then
        table.clear(M.mirrorTPPreviousY)
    end
    if M.setMirrorTPVisual then M.setMirrorTPVisual(M.mirrorTPDownEnabled) end
end


function M.isStealState()
    -- Match auto-switch carry script: WalkSpeed drops while carrying / stealing
    local char = player.Character
    if not char then return false end
    if M.hasBrainrotInHand() then return true end
    local h = char:FindFirstChildOfClass("Humanoid")
    if h and h.WalkSpeed < 25 then return true end
    local ok, val = pcall(function() return player:GetAttribute("Stealing") end)
    if ok and val == true then return true end
    local ok2, val2 = pcall(function() return char:GetAttribute("Stealing") end)
    if ok2 and val2 == true then return true end
    return false
end

function M.getActiveMoveSpeed()
    -- Auto Carry Speed: pick speed from steal state without forcing mode flags every frame
    if M.autoSwitchSpeedEnabled then
        local isSteal = M.isStealState()
        local inLagger = M.laggerModeEnabled or M.laggerCarryActive
        if inLagger then
            return isSteal and M.LAGGER_CARRY_SPEED or M.LAGGER_SPEED
        end
        return isSteal and M.CS or M.NS
    end

    -- Manual modes
    if M.hasBrainrotInHand() then
        return M.LAGGER_CARRY_SPEED
    end
    if M.laggerCarryActive then return M.LAGGER_CARRY_SPEED
    elseif M.laggerModeEnabled then return M.LAGGER_SPEED
    elseif M.carrySpeedActive then return M.CS
    else return M.NS end
end

function M.getAutoPathSpeed(stealPhase)
    -- Auto Play Full informa explicitamente quando entrou na etapa de roubo.
    -- Fora dessa etapa, preserva a seleção normal do ONI.
    if stealPhase then
        if M.laggerModeEnabled or M.laggerCarryActive or M.hasBrainrotInHand() then
            return math.clamp(tonumber(M.LAGGER_CARRY_SPEED) or 24.5, 1, 500)
        end
        return math.clamp(tonumber(M.CS) or 30, 1, 500)
    end
    if M.laggerModeEnabled or M.laggerCarryActive then
        return math.clamp(tonumber(M.LAGGER_SPEED) or 15, 1, 500)
    end
    return math.clamp(tonumber(M.NS) or 60, 1, 500)
end

function M.setModeNormalFlags()
    M.carrySpeedActive = false
    M.laggerModeEnabled = false
    M.laggerCarryActive = false
    if M.mobBtnRefs.carrySpeed then M.mobBtnRefs.carrySpeed(false) end
    if M.mobBtnRefs.lagger then M.mobBtnRefs.lagger(false) end
    if M.mobBtnRefs.laggerCarry then M.mobBtnRefs.laggerCarry(false) end
    if M.carryModeBtn then M.carryModeBtn.Text = "Carry Off" end
    if M.laggerModeBtn then M.laggerModeBtn.Text = "Lag Off" end
    if M.laggerCarryBtn then M.laggerCarryBtn.Text = "L.Carry Off" end
    if M.refreshSpeedModeLabel then M.refreshSpeedModeLabel() end
end

function M.setModeCarryFlags()
    M.carrySpeedActive = true
    M.laggerModeEnabled = false
    M.laggerCarryActive = false
    if M.mobBtnRefs.carrySpeed then M.mobBtnRefs.carrySpeed(true) end
    if M.mobBtnRefs.lagger then M.mobBtnRefs.lagger(false) end
    if M.mobBtnRefs.laggerCarry then M.mobBtnRefs.laggerCarry(false) end
    if M.carryModeBtn then M.carryModeBtn.Text = "Carry On" end
    if M.laggerModeBtn then M.laggerModeBtn.Text = "Lag Off" end
    if M.laggerCarryBtn then M.laggerCarryBtn.Text = "L.Carry Off" end
    if M.refreshSpeedModeLabel then M.refreshSpeedModeLabel() end
end

function M.setModeLaggerCarryFlags()
    M.carrySpeedActive = false
    M.laggerModeEnabled = false
    M.laggerCarryActive = true
    if M.mobBtnRefs.carrySpeed then M.mobBtnRefs.carrySpeed(false) end
    if M.mobBtnRefs.lagger then M.mobBtnRefs.lagger(false) end
    if M.mobBtnRefs.laggerCarry then M.mobBtnRefs.laggerCarry(true) end
    if M.carryModeBtn then M.carryModeBtn.Text = "Carry Off" end
    if M.laggerModeBtn then M.laggerModeBtn.Text = "Lag Off" end
    if M.laggerCarryBtn then M.laggerCarryBtn.Text = "L.Carry On" end
    if M.refreshSpeedModeLabel then M.refreshSpeedModeLabel() end
end

function M.stopWalkSpeedAutoSwitch()
    if M._autoSwitchSpeedConn then
        pcall(function() M._autoSwitchSpeedConn:Disconnect() end)
        M._autoSwitchSpeedConn = nil
    end
end

function M.startWalkSpeedAutoSwitch()
    if M._autoSwitchSpeedConn then return end
    M._autoSwitchSpeedConn = RunService.Heartbeat:Connect(function()
        if not M.autoSwitchSpeedEnabled and not M.autoTurnOffSpeedEnabled and not M.autoSwitchLaggerSpeedEnabled then
            M.stopWalkSpeedAutoSwitch()
            return
        end
        local char = player.Character
        if not char then return end
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not hum then return end
        local ws = hum.WalkSpeed or 16
        local thr = tonumber(M.AUTO_SWITCH_THRESHOLD) or 25

        -- Auto Switch Speed: game lowered WalkSpeed -> turn on carry
        if M.autoSwitchSpeedEnabled and ws <= thr and not M.carrySpeedActive and not M.laggerCarryActive then
            M.setModeCarryFlags()
        -- Auto Turn Off Speed: WalkSpeed back above threshold -> normal
        elseif M.autoTurnOffSpeedEnabled and ws > thr and M.carrySpeedActive then
            M.setModeNormalFlags()
        end

        -- Auto Switch Lagger: low WalkSpeed -> lagger carry; high -> normal
        if M.autoSwitchLaggerSpeedEnabled and ws <= thr and not M.laggerCarryActive and not M.laggerModeEnabled then
            M.setModeLaggerCarryFlags()
        elseif M.autoSwitchLaggerSpeedEnabled and ws > thr and (M.laggerCarryActive or M.laggerModeEnabled) then
            M.setModeNormalFlags()
        end
    end)
end

function M.refreshWalkSpeedAutoSwitch()
    if M.autoSwitchSpeedEnabled or M.autoTurnOffSpeedEnabled or M.autoSwitchLaggerSpeedEnabled then
        M.startWalkSpeedAutoSwitch()
    else
        M.stopWalkSpeedAutoSwitch()
    end
end

function M.updateAutoSwitchSpeed()
    -- Steal-based auto carry (existing)
    if M.autoSwitchSpeedEnabled then
        local isSteal = M.isStealState()
        if isSteal ~= M._autoSwitchWasSteal then
            M._autoSwitchWasSteal = isSteal
            local inLagger = M.laggerModeEnabled or M.laggerCarryActive
            if isSteal then
                if inLagger then
                    if M.mobBtnRefs.laggerCarry then M.mobBtnRefs.laggerCarry(true) end
                    if M.mobBtnRefs.carrySpeed then M.mobBtnRefs.carrySpeed(false) end
                    if M.laggerCarryBtn then M.laggerCarryBtn.Text = "L.Carry On" end
                    if M.carryModeBtn then M.carryModeBtn.Text = "Carry Off" end
                else
                    if M.mobBtnRefs.carrySpeed then M.mobBtnRefs.carrySpeed(true) end
                    if M.mobBtnRefs.laggerCarry then M.mobBtnRefs.laggerCarry(false) end
                    if M.carryModeBtn then M.carryModeBtn.Text = "Carry On" end
                    if M.laggerCarryBtn then M.laggerCarryBtn.Text = "L.Carry Off" end
                end
            else
                if inLagger then
                    if M.mobBtnRefs.laggerCarry then M.mobBtnRefs.laggerCarry(M.laggerCarryActive) end
                    if M.mobBtnRefs.lagger then M.mobBtnRefs.lagger(M.laggerModeEnabled) end
                    if M.laggerCarryBtn then M.laggerCarryBtn.Text = M.laggerCarryActive and "L.Carry On" or "L.Carry Off" end
                    if M.laggerModeBtn then M.laggerModeBtn.Text = M.laggerModeEnabled and "Lag On" or "Lag Off" end
                    if M.carryModeBtn then M.carryModeBtn.Text = "Carry Off" end
                else
                    if M.mobBtnRefs.carrySpeed then M.mobBtnRefs.carrySpeed(false) end
                    if M.carryModeBtn then M.carryModeBtn.Text = M.carrySpeedActive and "Carry On" or "Carry Off" end
                end
            end
            if M.refreshSpeedModeLabel then M.refreshSpeedModeLabel() end
        end
    end
end

function M.isRagdollState(hum)
    if not hum then return true end;local st=hum:GetState()
    return hum.PlatformStand or st==Enum.HumanoidStateType.Physics or st==Enum.HumanoidStateType.Ragdoll or st==Enum.HumanoidStateType.FallingDown
end

function M.runDrop()
    if M.dropActive then return end
    M.stopAutoTPForAction()
    local char = player.Character
    if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return end
    M.dropActive = true
    local startTime = tick()
    local dropConn
    dropConn = RunService.Heartbeat:Connect(function()
        local currentChar = player.Character
        local currentRoot = currentChar and currentChar:FindFirstChild("HumanoidRootPart")
        if not currentChar or not currentRoot then
            if dropConn then dropConn:Disconnect() end
            M.dropActive = false
            return
        end
        if tick() - startTime >= M.DROP_ASCEND_DURATION then
            if dropConn then dropConn:Disconnect() end
            local rayParams = RaycastParams.new()
            rayParams.FilterDescendantsInstances = {currentChar}
            rayParams.FilterType = Enum.RaycastFilterType.Exclude
            local rayResult = workspace:Raycast(currentRoot.Position, Vector3.new(0, -2000, 0), rayParams)
            if rayResult then
                local hum = currentChar:FindFirstChildOfClass("Humanoid")
                local offset = (hum and hum.HipHeight or 2) + (currentRoot.Size.Y / 2)
                currentRoot.CFrame = CFrame.new(currentRoot.Position.X, rayResult.Position.Y + offset, currentRoot.Position.Z)
                currentRoot.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
                currentRoot.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
            end
            M.dropActive = false
            return
        end
        currentRoot.Velocity = Vector3.new(currentRoot.Velocity.X, M.DROP_ASCEND_SPEED, currentRoot.Velocity.Z)
    end)
end

function M.stopAutoTPForAction()
    if M.autoTPEnabled then
        M.stopAutoTP()
        pcall(function() if M.setAutoTPVisual then M.setAutoTPVisual(false) end end)
        pcall(function() if M.saveConfig then M.saveConfig() end end)
    end
end


local function setupDeathReset()
    if M.autoResetOnDeath then
        local char = player.Character
        if char then
            local hum = char:FindFirstChildOfClass("Humanoid")
            if hum then
                if M._deathResetConn then M._deathResetConn:Disconnect() end
                M._deathResetConn = hum.Died:Connect(function()
                    if M.autoResetOnDeath then
                        M.cursedInstaReset()
                    end
                end)
            end
        end
        if not M._deathResetCharAdded then
            M._deathResetCharAdded = player.CharacterAdded:Connect(function(char)
                task.wait(0.5)
                setupDeathReset()
            end)
        end
    else
        if M._deathResetConn then M._deathResetConn:Disconnect(); M._deathResetConn = nil end
        if M._deathResetCharAdded then M._deathResetCharAdded:Disconnect(); M._deathResetCharAdded = nil end
    end
end

function M.startRemoveAcc()
    if M.removeAccEnabled then return end
    M.removeAccEnabled = true
    local function removeAccDo()
        if not M.removeAccEnabled then return end
        local char = player.Character
        if not char then return end
        for _,obj in ipairs(char:GetDescendants()) do
            if obj:IsA("Accessory") or obj:IsA("Hat") then
                if not M.removedAccessories[obj] then
                    M.removedAccessories[obj] = true
                    pcall(function() obj:Destroy() end)
                end
            end
        end
    end
    removeAccDo()
    M.removeAccConn = player.CharacterAdded:Connect(function()
        task.wait(0.5)
        if M.removeAccEnabled then removeAccDo() end
    end)
end

function M.stopRemoveAcc()
    M.removeAccEnabled = false
    if M.removeAccConn then
        M.removeAccConn:Disconnect()
        M.removeAccConn = nil
    end
    M.removedAccessories = {}
end

-- ============================================================
-- MOBILE BUTTONS (trascinabili singolarmente)
-- ============================================================




function M.destroyMobileButtons()
    if M.mobGuiRef then
        pcall(function() M.mobGuiRef:Destroy() end)
        M.mobGuiRef = nil
    end
    for _,n in ipairs({"MoveeMobileButtons"}) do
        local old = game:GetService("CoreGui"):FindFirstChild(n); if old then old:Destroy() end
        local pgui = player:FindFirstChild("PlayerGui"); if pgui then local o = pgui:FindFirstChild(n); if o then o:Destroy() end end
    end
    M.mobBtnRefs = {}
end

function M.loadBtnPositions()
    if not(isfile and isfile(M.MOB_POS_FILE)) then return {} end
    local ok, data = pcall(function() return HS:JSONDecode(readfile(M.MOB_POS_FILE)) end)
    if ok and type(data)=="table" then return data end
    return {}
end

function M.saveBtnPositions()
    if not writefile then return end
    if not M.mobGuiRef then return end
    local out = {}
    for _,child in ipairs(M.mobGuiRef:GetDescendants()) do
        if child:IsA("TextButton") and child:GetAttribute("BtnKey") then
            local key = child:GetAttribute("BtnKey")
            out[key] = {x=child.Position.X.Offset, y=child.Position.Y.Offset}
        end
    end
    pcall(function() writefile(M.MOB_POS_FILE, HS:JSONEncode(out)) end)
end

function M.resetMobilePositions()
    -- Clear saved positions (os.remove often missing in executors)
    pcall(function()
        if type(delfile) == "function" then
            delfile(M.MOB_POS_FILE)
        elseif type(writefile) == "function" then
            writefile(M.MOB_POS_FILE, "{}")
        end
    end)
    pcall(function()
        if isfile and isfile(M.MOB_POS_FILE) and type(writefile) == "function" then
            writefile(M.MOB_POS_FILE, "{}")
        end
    end)
    M._forceDefaultMobPos = true
    M.buildMobileButtons()
    M._forceDefaultMobPos = false
    -- Snap any leftover to defaults again after build
    pcall(function()
        if not M.mobGuiRef then return end
        local out = {}
        for _, child in ipairs(M.mobGuiRef:GetDescendants()) do
            if child:IsA("TextButton") and child:GetAttribute("BtnKey") then
                local key = child:GetAttribute("BtnKey")
                local dx = child:GetAttribute("DefaultX")
                local dy = child:GetAttribute("DefaultY")
                if typeof(dx) == "number" and typeof(dy) == "number" then
                    child.Position = UDim2.new(0, dx, 0, dy)
                    out[key] = {x = dx, y = dy}
                end
            end
        end
        if writefile then
            writefile(M.MOB_POS_FILE, HS:JSONEncode(out))
        end
    end)
end

function M.buildMobileButtons()
    M.destroyMobileButtons()
    if not M.mobileButtonsEnabled then return end

    local savedPositions = M._forceDefaultMobPos and {} or M.loadBtnPositions()
    local vp = workspace.CurrentCamera and workspace.CurrentCamera.ViewportSize or Vector2.new(800,600)

    -- Layout compacto em duas colunas, com botões quadrados como na referência.
    local BTN_H    = math.max(24, math.floor(M.mobileButtonsSize))
    local BTN_W    = BTN_H
    local CORNER_R = 18
    if M.circleButtonsEnabled then
        CORNER_R = math.floor(BTN_H / 2)
    end

    local mobGui = Instance.new("ScreenGui")
    mobGui.Name = "MoveeMobileButtons"
    mobGui.ResetOnSpawn = false
    mobGui.DisplayOrder = 15
    mobGui.IgnoreGuiInset = true
    pcall(function() if syn and syn.protect_gui then syn.protect_gui(mobGui) end end)
    if not pcall(function() mobGui.Parent = game:GetService("CoreGui") end) then
        mobGui.Parent = player:WaitForChild("PlayerGui")
    end
    M.mobGuiRef = mobGui

    local accent = UI_ACCENT or CHERRY_ACCENT or Color3.fromRGB(255, 255, 255)
    local BTN_OFF   = Color3.fromRGB(0, 0, 0)
    local BTN_ON    = Color3.fromRGB(214, 174, 26)
    local TXT_OFF   = Color3.fromRGB(255, 255, 255)
    local TXT_ON    = Color3.fromRGB(255, 255, 255)
    -- If accent is very dark, keep on-text readable
    -- O estado ativo usa amarelo/dourado, igual ao botão CARRY SPEED da referência.
    TXT_ON = Color3.fromRGB(255, 255, 255)

    local btnDefs = {
        {"drop", "DROP", false},
        {"autoLeft", "AUTO\nLEFT", true},
        {"autoBat", "AIMBOT", true},
        {"autoRight", "AUTO\nRIGHT", true},
        {"tpDown", "TP\nDOWN", false},
        {"carrySpeed", "CARRY\nSPEED", true},
        {"laggerCarry", "LAGGER\nCARRY", true},
        {"lagger", "LAGGER\nSPEED", true},
        {"bypass", "TP\nDESYNC", true},
        {"instaReset", "INSTA\nRESET", false},

    }

    local cols = 2
    local gap = 10
    local padding = 8
    local startX = vp.X - (cols * BTN_W + (cols - 1) * gap) - padding
    local startY = 48

    for i, def in ipairs(btnDefs) do
        local key = def[1]
        local label = def[2]
        local isToggle = def[3]

        local row = math.floor((i-1) / cols)
        local col = (i-1) % cols
        local defaultX = startX + col * (BTN_W + gap)
        local defaultY = startY + row * (BTN_H + gap)

        local saved = (not M._forceDefaultMobPos) and savedPositions[key] or nil
        local posX = (saved and type(saved.x) == "number") and saved.x or defaultX
        local posY = (saved and type(saved.y) == "number") and saved.y or defaultY

        local btn = Instance.new("TextButton")
        btn.Name = "Btn_" .. key
        local buttonW = BTN_W
        local buttonH = BTN_H
        btn.Size = UDim2.new(0, buttonW, 0, buttonH)
        btn.Position = UDim2.new(0, posX, 0, posY)
        btn:SetAttribute("DefaultX", defaultX)
        btn:SetAttribute("DefaultY", defaultY)
        btn.BackgroundColor3 = BTN_OFF
        btn.BackgroundTransparency = 0.05
        btn.Text = label
        btn.TextColor3 = TXT_OFF
        btn.TextSize = 11
        btn.Font = Enum.Font.GothamBlack
        btn.TextWrapped = true
        btn.BorderSizePixel = 0
        btn.ZIndex = 101
        btn.AutoButtonColor = false
        btn:SetAttribute("BtnKey", key)
        btn.Parent = mobGui

        local corner = Instance.new("UICorner", btn)
        if M.circleButtonsEnabled then
            corner.CornerRadius = UDim.new(1, 0)
        else
            corner.CornerRadius = UDim.new(0, CORNER_R)
        end
        btn.TextStrokeTransparency = 1
        do
            local st0 = Instance.new("UIStroke")
            st0.Name = "BtnStroke"
            st0.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
            st0.Color = Color3.fromRGB(25, 25, 25)
            st0.Thickness = 1
            st0.Transparency = 0.15
            st0.Parent = btn
        end
        local mobImgId = tonumber(M.mobBtnBgId) or 0
        if mobImgId > 0 then
            btn.BackgroundTransparency = 1
            local bgImg = Instance.new("ImageLabel")
            bgImg.Name = "BtnBgImage"
            bgImg.BackgroundTransparency = 1
            bgImg.Image = "rbxassetid://" .. tostring(mobImgId)
            bgImg.ScaleType = Enum.ScaleType.Crop
            bgImg.Size = UDim2.fromScale(1, 1)
            bgImg.ZIndex = btn.ZIndex
            bgImg.Parent = btn
            local bgc = Instance.new("UICorner", bgImg)
            bgc.CornerRadius = M.circleButtonsEnabled and UDim.new(1, 0) or UDim.new(0, CORNER_R)
            -- keep text above image
            btn.ZIndex = btn.ZIndex + 1
        end

        local isOn = false
        local function setOn(v)
            isOn = v
            local stroke = btn:FindFirstChild("BtnStroke")
            if not stroke then
                stroke = Instance.new("UIStroke")
                stroke.Name = "BtnStroke"
                stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
                stroke.Parent = btn
            end
            if v then
                TweenService:Create(btn, TweenInfo.new(0.12), {
                    BackgroundColor3 = BTN_ON,
                    TextColor3 = TXT_ON,
                }):Play()
                btn.TextStrokeColor3 = BTN_ON
                btn.TextStrokeTransparency = 0.35
                stroke.Color = BTN_ON
                stroke.Thickness = 2
                stroke.Transparency = 0.05
            else
                TweenService:Create(btn, TweenInfo.new(0.12), {
                    BackgroundColor3 = BTN_OFF,
                    TextColor3 = TXT_OFF,
                }):Play()
                btn.TextStrokeTransparency = 1
                stroke.Color = Color3.fromRGB(25, 25, 25)
                stroke.Thickness = 1
                stroke.Transparency = 0.15
            end
        end

        M.mobBtnRefs[key] = setOn

        btn.MouseButton1Down:Connect(function()
            TweenService:Create(btn, TweenInfo.new(0.05), {
                BackgroundColor3 = BTN_ON,
                TextColor3 = TXT_ON
            }):Play()
        end)
        btn.MouseButton1Up:Connect(function()
            if not isOn then
                TweenService:Create(btn, TweenInfo.new(0.1), {
                    BackgroundColor3 = BTN_OFF,
                    TextColor3 = TXT_OFF
                }):Play()
            end
        end)

        -- Drag individuale
        local dragging = false
        local dragStart = nil
        local startPos = nil
        btn.InputBegan:Connect(function(input)
            if M.uiLocked then return end
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                dragging = true
                dragStart = input.Position
                startPos = btn.Position
                input.Changed:Connect(function()
                    if input.UserInputState == Enum.UserInputState.End then
                        dragging = false
                        M.saveBtnPositions()
                    end
                end)
            end
        end)
        btn.InputChanged:Connect(function(input)
            if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
                local delta = input.Position - dragStart
                btn.Position = UDim2.new(0, startPos.X.Offset + delta.X, 0, startPos.Y.Offset + delta.Y)
            end
        end)
        UIS.InputChanged:Connect(function(input)
            if dragging and M.uiLocked then
                dragging = false
            end
        end)

        btn.Activated:Connect(function()
            if key == "drop" then
                M.runDrop()

            elseif key == "tpDown" then
                M.runTPFloor()
            elseif key == "instaReset" then
                M.cursedInstaReset()
            elseif key == "autoLeft" then
                if M.autoBatEnabled then
                    M.stopBatAimbot()
                    if M.autoBatSetVisual then M.autoBatSetVisual(false) end
                    if M.mobBtnRefs.autoBat then M.mobBtnRefs.autoBat(false) end
                end
                if M.autoRightEnabled then
                    M.autoRightEnabled = false
                    M.stopAutoRight()
                    if M.autoRightSetVisual then M.autoRightSetVisual(false) end
                    if M.mobBtnRefs.autoRight then M.mobBtnRefs.autoRight(false) end
                end
                M.autoLeftEnabled = not M.autoLeftEnabled
                if M.autoLeftEnabled then M.startAutoLeft() else M.stopAutoLeft() end
                setOn(M.autoLeftEnabled)
                if M.autoLeftSetVisual then M.autoLeftSetVisual(M.autoLeftEnabled) end
                saveCherryConfig()
            elseif key == "autoRight" then
                if M.autoBatEnabled then
                    M.stopBatAimbot()
                    if M.autoBatSetVisual then M.autoBatSetVisual(false) end
                    if M.mobBtnRefs.autoBat then M.mobBtnRefs.autoBat(false) end
                end
                if M.autoLeftEnabled then
                    M.autoLeftEnabled = false
                    M.stopAutoLeft()
                    if M.autoLeftSetVisual then M.autoLeftSetVisual(false) end
                    if M.mobBtnRefs.autoLeft then M.mobBtnRefs.autoLeft(false) end
                end
                M.autoRightEnabled = not M.autoRightEnabled
                if M.autoRightEnabled then M.startAutoRight() else M.stopAutoRight() end
                setOn(M.autoRightEnabled)
                if M.autoRightSetVisual then M.autoRightSetVisual(M.autoRightEnabled) end
                saveCherryConfig()
            elseif key == "autoBat" then
                if M.autoLeftEnabled then
                    M.autoLeftEnabled = false
                    M.stopAutoLeft()
                    if M.autoLeftSetVisual then M.autoLeftSetVisual(false) end
                    if M.mobBtnRefs.autoLeft then M.mobBtnRefs.autoLeft(false) end
                end
                if M.autoRightEnabled then
                    M.autoRightEnabled = false
                    M.stopAutoRight()
                    if M.autoRightSetVisual then M.autoRightSetVisual(false) end
                    if M.mobBtnRefs.autoRight then M.mobBtnRefs.autoRight(false) end
                end
                if not M.autoBatEnabled then
                    M.queueAutoBatStart()
                else
                    M.stopBatAimbot()
                end
                setOn(M.autoBatEnabled)
                if M.autoBatSetVisual then M.autoBatSetVisual(M.autoBatEnabled) end
                saveCherryConfig()
            elseif key == "lagger" then
                M.toggleLaggerMode()
                setOn(M.laggerModeEnabled)
                if M.mobBtnRefs.carrySpeed then M.mobBtnRefs.carrySpeed(M.carrySpeedActive) end
                if M.laggerModeBtn then
                    M.laggerModeBtn.Text = M.laggerModeEnabled and "Lag On" or "Lag Off"
                end
                saveCherryConfig()
            elseif key == "carrySpeed" then
                M.toggleCarryMode()
                setOn(M.carrySpeedActive)
                if M.mobBtnRefs.lagger then M.mobBtnRefs.lagger(M.laggerModeEnabled) end
                if M.carryModeBtn then
                    M.carryModeBtn.Text = M.carrySpeedActive and "Carry On" or "Carry Off"
                end
                saveCherryConfig()
            elseif key == "bypass" then
                M.toggleBypassAimbot()
                setOn(M.bypassAimbotEnabled)
                if M.setBypassVisual then M.setBypassVisual(M.bypassAimbotEnabled) end
                saveCherryConfig()
            elseif key == "laggerCarry" then
                M.toggleLaggerCarry()
                setOn(M.laggerCarryActive)
                saveCherryConfig()
            end
        end)
    end

    if M.mobBtnRefs.autoLeft then M.mobBtnRefs.autoLeft(M.autoLeftEnabled) end
    if M.mobBtnRefs.autoRight then M.mobBtnRefs.autoRight(M.autoRightEnabled) end
    if M.mobBtnRefs.autoBat then M.mobBtnRefs.autoBat(M.autoBatEnabled) end

    if M.mobBtnRefs.lagger then M.mobBtnRefs.lagger(M.laggerModeEnabled) end
    if M.mobBtnRefs.carrySpeed then M.mobBtnRefs.carrySpeed(M.carrySpeedActive) end
    if M.mobBtnRefs.bypass then M.mobBtnRefs.bypass(M.bypassAimbotEnabled) end
    if M.mobBtnRefs.laggerCarry then M.mobBtnRefs.laggerCarry(M.laggerCarryActive) end
end

-- ============================================================
-- CONFIG SAVE/LOAD
-- ============================================================
M.customBgId = 0
M.customBgOpacity = 0.35
M.mobBtnBgId = 0
M.BG_IMAGE_IDS = {
    79737099962715,
    71211662493854,
    15556272558,
    1471587689,
    14349182390,
    108236541541009,
}
M.MOB_BTN_IMAGE_IDS = {
    15101684346,
    39396,
    109592813321691,
    83661129801187,
    94353803110527,
    109100201685955,
}


local function loadCherryConfig()
    if type(readfile)~="function" or type(isfile)~="function" then return end
    local ok,d = pcall(function()
        if not isfile(CHERRY_CONFIG_NAME) then return nil end
        return HS:JSONDecode(readfile(CHERRY_CONFIG_NAME))
    end)
    if ok and type(d)=="table" then
        local themeName = nil
        if type(d.Theme)=="string" and CHERRY_THEMES[d.Theme] then themeName = d.Theme end
        if type(d.colorScheme)=="string" and CHERRY_THEMES[d.colorScheme] then themeName = d.colorScheme end
        if themeName then
            CherryConfig.Theme = themeName
            M.colorScheme = themeName
            M._savedTheme = themeName
        end
        if type(d.normalSpeed)=="number" then M.NS=d.normalSpeed end
        if type(d.carrySpeed)=="number" then M.CS=d.carrySpeed end
        if type(d.laggerSpeed)=="number" then M.LAGGER_SPEED=d.laggerSpeed end
        if type(d.laggerCarrySpeed)=="number" then M.LAGGER_CARRY_SPEED=d.laggerCarrySpeed end
        if type(d.speedMethod)=="string" then
            for _,sm in ipairs(M.speedMethodList) do if sm==d.speedMethod then M.speedMethod=sm; break end end
        end
        if type(d.grabRadius)=="number" then M.Steal.StealRadius=d.grabRadius end
        if type(d.stealDuration)=="number" then M.Steal.StealDuration=d.stealDuration end
        if type(d.stealStopTime)=="number" then M.Steal.StopTime=d.stealStopTime end
        if type(d.stealMode)=="string" then
            if d.stealMode == "Semi" or d.stealMode == "Normal" or d.stealMode == "V1" or d.stealMode == "V2" or d.stealMode == "V3" or d.stealMode == "V4" then
                M.stealMode=d.stealMode
            end
        end
        if type(d.autoTPHeight)=="number" then M.autoTPHeight=d.autoTPHeight end
        if type(d.fovValue)=="number" then M.fovValue=d.fovValue end
        if type(d.uiScale)=="number" then M.uiScale=d.uiScale end
        if type(d.infJumpMode)=="string" then M.infJumpMode=d.infJumpMode end
        if type(d.mobileButtonsSize)=="number" then M.mobileButtonsSize=d.mobileButtonsSize end

        if type(d.skyTheme)=="string" then M.currentSkyTheme=d.skyTheme end
        if type(d.stealBarSize)=="number" then M.stealBarSize=d.stealBarSize end
        if type(d.stealBarScale)=="number" then M.stealBarScale=math.clamp(d.stealBarScale, 0.1, 2) end
        if d.carrySpeedActive~=nil then M.carrySpeedActive=d.carrySpeedActive end
        if d.laggerModeEnabled~=nil then M.laggerModeEnabled=d.laggerModeEnabled end
        if d.autoSwing~=nil then M.autoSwingEnabled=d.autoSwing==true end
        if d.introSoundEnabled~=nil then M.introSoundEnabled=d.introSoundEnabled==true end
        if d.introSongChoice then M.introSongChoice=d.introSongChoice end
        if d.introGUIEnabled~=nil then M.introGUIEnabled=d.introGUIEnabled==true end
        if d.autoBatMode == "BAT V2" then M.autoBatMode="BAT V2" else M.autoBatMode="Normal" end
        if d.autoPlayMode == "Full" then M.autoPlayMode="Full" else M.autoPlayMode="Semi" end
        if d.ragdollGui~=nil then M.ragdollGuiEnabled=d.ragdollGui==true end
        if d.circleButtonsEnabled~=nil then M.circleButtonsEnabled=d.circleButtonsEnabled==true end
        if d.perButtonDrag~=nil then M.perButtonDragEnabled=d.perButtonDrag==true end
        if d.mobileButtonsEnabled~=nil then M.mobileButtonsEnabled=d.mobileButtonsEnabled end
        if d.medusaReset~=nil then M.medusaResetEnabled=d.medusaReset==true end
        if d.autoMoveSwing~=nil then M.autoMoveSwingEnabled=d.autoMoveSwing==true end
        if d.autoSwitchSpeed~=nil then M.autoSwitchSpeedEnabled=d.autoSwitchSpeed==true end
        if d.autoTurnOffSpeed~=nil then M.autoTurnOffSpeedEnabled=d.autoTurnOffSpeed==true end
        if d.autoSwitchLaggerSpeed~=nil then M.autoSwitchLaggerSpeedEnabled=d.autoSwitchLaggerSpeed==true end
        if type(d.customFont)=="string" then M.customFontSelected=d.customFont end
        if d.removeAcc~=nil then M.removeAccEnabled=d.removeAcc end
        if d.antiRagdoll~=nil then M.antiRagdollEnabled=d.antiRagdoll end
        if type(d.antiRagdollMode)=="string" and (d.antiRagdollMode=="Splatter" or d.antiRagdollMode=="No Splatter") then M.antiRagdollMode=d.antiRagdollMode end
        if d.autoStealEnabled~=nil then M.Steal.AutoStealEnabled=d.autoStealEnabled end
        if d.autoRadiusEnabled~=nil then M.autoRadiusEnabled=d.autoRadiusEnabled==true end
        if d.hitSoundEnabled~=nil then M.hitSoundEnabled=d.hitSoundEnabled==true end
        if type(d.hitSoundChoice)=="string" then M.hitSoundChoice=d.hitSoundChoice end
        if type(d.hitSoundCustomId)=="string" then M.hitSoundCustomId=d.hitSoundCustomId end
        if type(d.hitSoundVolume)=="number" then M.hitSoundVolume=d.hitSoundVolume end
        if type(d.hitSoundPitch)=="number" then M.hitSoundPitch=d.hitSoundPitch end

        if d.infiniteJump~=nil then M.infJumpEnabled=d.infiniteJump end
        if d.medusaCounter~=nil then M.medusaCounterEnabled=d.medusaCounter end
        if d.batCounter~=nil then M.batCounterEnabled=d.batCounter end
        if d.unwalkEnabled~=nil then M.unwalkEnabled=d.unwalkEnabled end
        if d.antiLag~=nil then M.antiLagEnabled=d.antiLag end
        if d.antiSummerBase~=nil then M.antiSummerBaseEnabled=d.antiSummerBase end
        if d.uiLocked~=nil then M.uiLocked=d.uiLocked==true end
        if d.stretchRez~=nil then M.stretchRezEnabled=d.stretchRez end
        if d.autoTPEnabled~=nil then M.autoTPEnabled=d.autoTPEnabled end
        if d.antiKick~=nil then M.antiKickEnabled=d.antiKick end
        if d.safeMode~=nil then M.safeModeEnabled=d.safeMode end
        if d.mirrorTPDown~=nil then M.mirrorTPDownEnabled=d.mirrorTPDown end
        if type(d.customBgId)=="number" then M.customBgId=d.customBgId end
        if type(d.customBgOpacity)=="number" then M.customBgOpacity=math.clamp(d.customBgOpacity,0,1) end
        if type(d.mobBtnBgId)=="number" then M.mobBtnBgId=d.mobBtnBgId end
        if d.autoBat~=nil then M.autoBatEnabled=d.autoBat end
        if d.semiHoldMin then M.Semi.holdMin=d.semiHoldMin end
        if d.semiHoldMax then M.Semi.holdMax=d.semiHoldMax end
        if d.semiEntryDelay then M.Semi.entryDelay=d.semiEntryDelay end
        if d.semiPrimeRange then M.Semi.primeRange=d.semiPrimeRange end
        if type(d.semiRadius)=="number" then M.Semi.radius=math.min(d.semiRadius, 10) end
        if d.menuOpen~=nil then M.menuOpen=d.menuOpen~=false end
        -- theme already applied above; keep M._savedTheme in sync
        if type(d.Theme)=="string" and CHERRY_THEMES[d.Theme] then M._savedTheme=d.Theme; M.colorScheme=d.Theme end
        if type(d.colorScheme)=="string" and CHERRY_THEMES[d.colorScheme] then M._savedTheme=d.colorScheme; M.colorScheme=d.colorScheme end
        if d.autoResetOnDeath~=nil then M.autoResetOnDeath=d.autoResetOnDeath end
        if type(d.animPack)=="string" then M.animPack=d.animPack end
        if d.headlessEnabled~=nil then M.headlessEnabled=d.headlessEnabled end
        if d.korbloxEnabled~=nil then M.korbloxEnabled=d.korbloxEnabled end
        if d.bypassAimbotEnabled~=nil then M.bypassAimbotEnabled=d.bypassAimbotEnabled end
        if d.animPackEnabled~=nil then M.animPackEnabled=d.animPackEnabled end
        if d.wingsAuraEnabled~=nil then M.wingsAuraEnabled=d.wingsAuraEnabled end
        local function lk(e,d2)
            if type(d2)~="table" then return end
            if d2.kb and Enum.KeyCode[d2.kb] then e.kb=Enum.KeyCode[d2.kb] else e.kb=nil end
            if d2.gp and Enum.KeyCode[d2.gp] then e.gp=Enum.KeyCode[d2.gp] else e.gp=nil end
        end
        if d.dodgeTPHeight~=nil then M.dodgeTPHeight=d.dodgeTPHeight end
        if d.dropBrainrotKey then lk(M.KB.DropBrainrot,d.dropBrainrotKey) end
        if d.autoLeftKey then lk(M.KB.AutoLeft,d.autoLeftKey) end
        if d.autoRightKey then lk(M.KB.AutoRight,d.autoRightKey) end
        if d.autoBatKey then lk(M.KB.AutoBat,d.autoBatKey) end
        if d.laggerToggleKey then lk(M.KB.LaggerToggle,d.laggerToggleKey) end
        if d.tpFloorKey then lk(M.KB.TPFloor,d.tpFloorKey) end
        if d.instaResetKey then lk(M.KB.InstaReset,d.instaResetKey) end
        if d.guiHideKey then lk(M.KB.GuiHide,d.guiHideKey) end
        if d.speedToggleKey then lk(M.KB.SpeedToggle,d.speedToggleKey) end
        if d.bypassAimbotKey then lk(M.KB.BypassAimbot,d.bypassAimbotKey) end
    end
end

local function saveCherryConfig()
    if type(writefile)~="function" then return end
    local function ks(e)
        if type(e) ~= "table" then return {kb=nil,gp=nil} end
        return {
            kb = (e.kb and e.kb.Name) or nil,
            gp = (e.gp and e.gp.Name) or nil,
        }
    end
    local cfg = {
        Theme=CherryConfig.Theme, colorScheme=M.colorScheme or CherryConfig.Theme, menuOpen=M.menuOpen~=false,
        normalSpeed=M.NS, carrySpeed=M.CS, laggerSpeed=M.LAGGER_SPEED,
        laggerCarrySpeed=M.LAGGER_CARRY_SPEED, speedMethod=M.speedMethod, grabRadius=M.Steal.StealRadius,
        stealDuration=M.Steal.StealDuration, stealStopTime=M.Steal.StopTime, stealMode=M.stealMode,
        autoTPHeight=M.autoTPHeight, fovValue=M.fovValue, uiScale=M.uiScale,
        infJumpMode=M.infJumpMode,
        mobileButtonsSize=M.mobileButtonsSize, skyTheme=M.currentSkyTheme,
        customBgId=tonumber(M.customBgId) or 0, customBgOpacity=tonumber(M.customBgOpacity) or 0.35,
        mobBtnBgId=tonumber(M.mobBtnBgId) or 0,
        stealBarSize=M.stealBarSize, stealBarScale=M.stealBarScale,
        carrySpeedActive=M.carrySpeedActive, laggerModeEnabled=M.laggerModeEnabled,
        autoSwing=M.autoSwingEnabled, introSoundEnabled=M.introSoundEnabled,
        introSongChoice=M.introSongChoice,
        introGUIEnabled=M.introGUIEnabled,
        ragdollGui=M.ragdollGuiEnabled, circleButtonsEnabled=M.circleButtonsEnabled,
        perButtonDrag=M.perButtonDragEnabled, mobileButtonsEnabled=M.mobileButtonsEnabled,
        medusaReset=M.medusaResetEnabled, autoMoveSwing=M.autoMoveSwingEnabled,
        autoSwitchSpeed=M.autoSwitchSpeedEnabled, autoTurnOffSpeed=M.autoTurnOffSpeedEnabled, autoSwitchLaggerSpeed=M.autoSwitchLaggerSpeedEnabled, customFont=M.customFontSelected,
        removeAcc=M.removeAccEnabled,
        autoStealEnabled=M.Steal.AutoStealEnabled,
        autoRadiusEnabled=M.autoRadiusEnabled,
        antiRagdoll=M.antiRagdollEnabled, antiRagdollMode=M.antiRagdollMode, infiniteJump=M.infJumpEnabled,
        medusaCounter=M.medusaCounterEnabled, batCounter=M.batCounterEnabled,
        unwalkEnabled=M.unwalkEnabled, antiLag=M.antiLagEnabled, antiSummerBase=M.antiSummerBaseEnabled, uiLocked=M.uiLocked,
        stretchRez=M.stretchRezEnabled, autoTPEnabled=M.autoTPEnabled,
        antiKick=M.antiKickEnabled, safeMode=M.safeModeEnabled, mirrorTPDown=M.mirrorTPDownEnabled, autoBat=M.autoBatEnabled, autoBatMode=M.autoBatMode,
        autoPlayMode=M.autoPlayMode,
        semiHoldMin=M.Semi.holdMin, semiHoldMax=M.Semi.holdMax,
        semiEntryDelay=M.Semi.entryDelay,
        semiPrimeRange=M.Semi.primeRange,
        semiRadius=math.min(M.Semi.radius, 10),
        autoResetOnDeath=M.autoResetOnDeath,
        animPack=M.animPack,
        headlessEnabled=M.headlessEnabled,
        korbloxEnabled=M.korbloxEnabled,
        bypassAimbotEnabled=M.bypassAimbotEnabled,
        animPackEnabled=M.animPackEnabled,
        wingsAuraEnabled=M.wingsAuraEnabled,
        hitSoundEnabled=M.hitSoundEnabled==true,
        hitSoundChoice=M.hitSoundChoice or "Minecraft Critical Hit",
        hitSoundCustomId=tostring(M.hitSoundCustomId or ""),
        hitSoundVolume=tonumber(M.hitSoundVolume) or 1.0,
        hitSoundPitch=tonumber(M.hitSoundPitch) or 1.0,
        dropBrainrotKey=ks(M.KB.DropBrainrot), autoLeftKey=ks(M.KB.AutoLeft),
        autoRightKey=ks(M.KB.AutoRight), autoBatKey=ks(M.KB.AutoBat),
        laggerToggleKey=ks(M.KB.LaggerToggle), tpFloorKey=ks(M.KB.TPFloor),
        instaResetKey=ks(M.KB.InstaReset), guiHideKey=ks(M.KB.GuiHide),
        speedToggleKey=ks(M.KB.SpeedToggle), bypassAimbotKey=ks(M.KB.BypassAimbot),
    }
    pcall(function() writefile(CHERRY_CONFIG_NAME, HS:JSONEncode(cfg)) end)
end

M.saveConfig = saveCherryConfig

-- ============================================================
function M.trackConn(conn) table.insert(M._persistentConns,conn); return conn end
function M.clearPersistentConns()
    for _,c in ipairs(M._persistentConns) do pcall(function() c:Disconnect() end) end
    M._persistentConns={}
end

function M.makeNumberCallback(tbl,key,min,max)
    return function(v)
        if min and v<min then return end
        if max and v>max then return end
        tbl[key]=v
        if key=="mobileButtonsSize" and M.mobileButtonsEnabled then M.buildMobileButtons() end
        if key=="stealBarSize" then M.buildStatusUI() end
        saveCherryConfig()
    end
end

-- ============================================================
-- HIT SOUNDS SYSTEM
-- ============================================================
M.hitSoundEnabled = (M.hitSoundEnabled == true)
M.hitSoundChoice = M.hitSoundChoice or "Minecraft Critical Hit"
M.hitSoundCustomId = M.hitSoundCustomId or ""
M.hitSoundVolume = tonumber(M.hitSoundVolume) or 1.0
M.hitSoundPitch = tonumber(M.hitSoundPitch) or 1.0
M._lastHitSoundTick = 0

local HIT_SOUND_PRESETS = {
    ["Minecraft Critical Hit"] = "122699784909910",
    ["Minecraft Hit"]          = "8816864160",
    ["COD Hitmarker"]          = "3744371091",
    ["Neverlose Bell"]         = "6607204501",
    ["Rust Headshot"]          = "1255040462",
    ["Skeet Hit"]              = "4817809188",
    ["Bonk Hit"]               = "3945417387",
    ["Custom"]                 = "",
}

M.HIT_SOUND_NAMES = {
    "Minecraft Critical Hit",
    "Minecraft Hit",
    "COD Hitmarker",
    "Neverlose Bell",
    "Rust Headshot",
    "Skeet Hit",
    "Bonk Hit",
    "Custom",
}

function M.getHitSoundAssetId()
    local choice = M.hitSoundChoice or "Minecraft Critical Hit"
    local presetId = HIT_SOUND_PRESETS[choice]
    if choice == "Custom" or not presetId or presetId == "" then
        local raw = tostring(M.hitSoundCustomId or ""):gsub("%D+", "")
        if #raw > 0 then return raw end
        return "122699784909910"
    end
    return presetId
end

function M.playHitSound()
    local now = tick()
    if now - (M._lastHitSoundTick or 0) < 0.04 then return end
    M._lastHitSoundTick = now

    local assetId = M.getHitSoundAssetId()
    if not assetId or assetId == "" then return end

    task.spawn(function()
        local sound = Instance.new("Sound")
        sound.Name = "WaveHubHitSound"
        sound.SoundId = "rbxassetid://" .. tostring(assetId)
        sound.Volume = math.clamp(tonumber(M.hitSoundVolume) or 1.0, 0, 10)
        sound.PlaybackSpeed = math.clamp(tonumber(M.hitSoundPitch) or 1.0, 0.1, 5)
        sound.Parent = game:GetService("SoundService")
        sound:Play()
        sound.Ended:Connect(function()
            pcall(function() sound:Destroy() end)
        end)
        task.delay(4, function()
            pcall(function() if sound and sound.Parent then sound:Destroy() end end)
        end)
    end)
end

local _lastPlayerSwingTick = 0
function M.notifyPlayerSwung()
    _lastPlayerSwingTick = tick()
end

local _enemyHealthCache = {}
local _hitSoundScanAcc = 0
RunService.Heartbeat:Connect(function(dt)
    if not M.hitSoundEnabled then return end
    _hitSoundScanAcc = _hitSoundScanAcc + (tonumber(dt) or 0)
    if _hitSoundScanAcc < 0.12 then return end
    _hitSoundScanAcc = 0
    local char = player.Character
    local myRoot = char and char:FindFirstChild("HumanoidRootPart")
    if not myRoot then return end

    local now = tick()
    local recentlySwung = (now - _lastPlayerSwingTick) < 0.65 or M.autoBatEnabled or M.bypassAimbotEnabled

    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= player and p.Character then
            local eChar = p.Character
            local eHum = eChar:FindFirstChildOfClass("Humanoid")
            local eRoot = eChar:FindFirstChild("HumanoidRootPart")
            if eHum and eRoot and eHum.Health > 0 then
                local prevHealth = _enemyHealthCache[p]
                local curHealth = eHum.Health
                if prevHealth and curHealth < prevHealth then
                    local dist = (eRoot.Position - myRoot.Position).Magnitude
                    if dist <= 32 or recentlySwung then
                        M.playHitSound()
                    end
                end
                _enemyHealthCache[p] = curHealth
            else
                _enemyHealthCache[p] = nil
            end
        else
            _enemyHealthCache[p] = nil
        end
    end
end)

local function hookToolActivated(tool)
    if not tool:IsA("Tool") then return end
    pcall(function()
        tool.Activated:Connect(function()
            M.notifyPlayerSwung()
        end)
    end)
end

local function hookCharTools(char)
    if not char then return end
    if M._toolChildConn then M._toolChildConn:Disconnect(); M._toolChildConn=nil end
    for _, child in ipairs(char:GetChildren()) do
        hookToolActivated(child)
    end
    M._toolChildConn = char.ChildAdded:Connect(hookToolActivated)
end

if player.Character then hookCharTools(player.Character) end
player.CharacterAdded:Connect(hookCharTools)

-- ============================================================
-- ONIX DUELS UI (LUXURY SMOOTH ROUNDED MODERN DESIGN)
-- ============================================================

pcall(applyAccentFromTheme)

local UI_TWEEN_FAST = TweenInfo.new(0.25, Enum.EasingStyle.Quart, Enum.EasingDirection.Out)
local UI_TWEEN_MED  = TweenInfo.new(0.35, Enum.EasingStyle.Quart, Enum.EasingDirection.Out)

-- UI STYLE HELPERS (LUXURY ROUNDED DESIGN)
local function uiCardStyle(f)
    f.BackgroundColor3 = UI_ROW_BG
    f.BackgroundTransparency = 0.3
    f.BorderSizePixel = 0
    local c = f:FindFirstChildOfClass("UICorner")
    if not c then
        c = Instance.new("UICorner")
        c.Parent = f
    end
    c.CornerRadius = UDim.new(0, 10)
    
    local s = f:FindFirstChildOfClass("UIStroke")
    if not s then
        s = Instance.new("UIStroke")
        s.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
        s.Parent = f
    end
    s.Thickness = 1
    s.Color = UI_CARD_STROKE or Color3.fromRGB(50, 52, 65)
    s.Transparency = 0.4
end

local function uiSmallBtn(p)
    local b = Instance.new("TextButton")
    b.Position = p.Pos or UDim2.new(0,0,0,0)
    b.Size = p.Size or UDim2.new(0,40,0,24)
    b.BackgroundColor3 = p.Bg or UI_BTN_BG
    b.BackgroundTransparency = 0.25
    b.BorderSizePixel = 0
    b.Text = p.Text or ""
    b.TextColor3 = p.Col or UI_TEXT_DIM
    b.TextSize = p.TS or 11
    b.Font = Enum.Font.GothamBold
    b.AutoButtonColor = false
    b.ZIndex = p.Z or 1
    b.Parent = p.Parent
    
    local c = Instance.new("UICorner", b)
    c.CornerRadius = UDim.new(0, p.CR or 8)
    
    local s = Instance.new("UIStroke", b)
    s.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    s.Color = p.SC or Color3.fromRGB(70, 72, 85)
    s.Thickness = 1
    s.Transparency = p.STr or 0.35

    b.MouseEnter:Connect(function()
        TweenService:Create(b, UI_TWEEN_FAST, {BackgroundTransparency = 0.05, TextColor3 = UI_TEXT_WHITE}):Play()
        s.Color = UI_ACCENT or Color3.fromRGB(255, 255, 255)
        s.Transparency = 0.1
    end)
    b.MouseLeave:Connect(function()
        TweenService:Create(b, UI_TWEEN_FAST, {BackgroundTransparency = 0.25, TextColor3 = p.Col or UI_TEXT_DIM}):Play()
        s.Color = p.SC or Color3.fromRGB(70, 72, 85)
        s.Transparency = p.STr or 0.35
    end)
    return b
end

local function uiAccentBar(parent, on)
    local b = Instance.new("Frame")
    b.Position = UDim2.new(0, 3, 0.2, 0)
    b.Size = UDim2.new(0, 3, 0.6, 0)
    b.BackgroundColor3 = UI_ACCENT or Color3.fromRGB(255,255,255)
    b.BackgroundTransparency = on and 0 or 1
    b.BorderSizePixel = 0
    b.Parent = parent
    local c = Instance.new("UICorner", b)
    c.CornerRadius = UDim.new(1, 0)
    return b
end

local function uiAutoCanvas(scroll)
    local lay = scroll:FindFirstChildOfClass("UIListLayout")
    if not lay then return end
    local pad = scroll:FindFirstChildOfClass("UIPadding")
    local function upd()
        local padBottom = (pad and pad.PaddingBottom.Offset or 0)
        local padTop = (pad and pad.PaddingTop.Offset or 0)
        local h = lay.AbsoluteContentSize.Y + padBottom + padTop + 24
        scroll.CanvasSize = UDim2.new(0, 0, 0, math.max(h, 1))
    end
    lay:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(upd)
    task.defer(upd)
    task.delay(0.15, upd)
    task.delay(0.5, upd)
end

local function uiSectionHeader(parent, text)
    local r = Instance.new("Frame")
    r.Size = UDim2.new(1,0,0,28)
    r.BackgroundTransparency = 1
    r.Parent = parent
    
    local dot = Instance.new("Frame")
    dot.Size = UDim2.new(0, 6, 0, 6)
    dot.Position = UDim2.new(0, 4, 0.5, -3)
    dot.BackgroundColor3 = UI_ACCENT or Color3.fromRGB(255, 255, 255)
    dot.BorderSizePixel = 0
    dot.Parent = r
    Instance.new("UICorner", dot).CornerRadius = UDim.new(1, 0)

    local l = Instance.new("TextLabel")
    l.Position = UDim2.new(0, 16, 0, 0)
    l.Size = UDim2.new(1, -18, 0, 18)
    l.BackgroundTransparency = 1
    l.Text = string.upper(text)
    l.TextColor3 = UI_TEXT_SECTION
    l.TextSize = 11
    l.Font = Enum.Font.GothamBlack
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.Parent = r
    
    local line = Instance.new("Frame")
    line.Size = UDim2.new(1, -16, 0, 1)
    line.Position = UDim2.new(0, 16, 0, 22)
    line.BackgroundColor3 = UI_ACCENT or Color3.fromRGB(255,255,255)
    line.BackgroundTransparency = 0.55
    line.BorderSizePixel = 0
    line.Parent = r
    do
        local g = Instance.new("UIGradient", line)
        g.Transparency = NumberSequence.new({
            NumberSequenceKeypoint.new(0, 0.2),
            NumberSequenceKeypoint.new(0.7, 0.8),
            NumberSequenceKeypoint.new(1, 1),
        })
    end
    return r
end

local function uiInputRow(parent, label, def, hidden)
    local r = Instance.new("Frame")
    r.ClipsDescendants = true
    r.Size = UDim2.new(1,0,0,44)
    r.BackgroundColor3 = UI_ROW_BG
    r.BackgroundTransparency = 0.3
    r.BorderSizePixel = 0
    if hidden then r.Visible = false end
    r.Parent = parent
    uiCardStyle(r)
    
    local l = Instance.new("TextLabel")
    l.Position = UDim2.new(0,14,0,0)
    l.Size = UDim2.new(1,-90,1,0)
    l.BackgroundTransparency = 1
    l.Text = label
    l.TextColor3 = UI_TEXT_PRIMARY
    l.TextSize = 13
    l.Font = Enum.Font.GothamMedium
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.Parent = r
    
    local bx = Instance.new("TextBox")
    bx.Position = UDim2.new(1,-76,0.5,-12)
    bx.Size = UDim2.new(0,64,0,24)
    bx.BackgroundColor3 = UI_BTN_BG
    bx.BackgroundTransparency = 0.15
    bx.BorderSizePixel = 0
    bx.Text = tostring(def)
    bx.TextColor3 = UI_ACCENT or Color3.fromRGB(255, 255, 255)
    bx.TextSize = 12
    bx.Font = Enum.Font.GothamBold
    bx.Parent = r
    Instance.new("UICorner", bx).CornerRadius = UDim.new(0, 8)
    
    local st = Instance.new("UIStroke", bx)
    st.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    st.Color = Color3.fromRGB(65, 68, 80)
    st.Thickness = 1
    st.Transparency = 0.35
    return r, bx
end

local function uiToggleRow(parent, label, on, callback)
    local r = Instance.new("Frame")
    r.ClipsDescendants = true
    r.Size = UDim2.new(1,0,0,44)
    r.BackgroundColor3 = UI_ROW_BG
    r.BackgroundTransparency = 0.3
    r.BorderSizePixel = 0
    r.Parent = parent
    uiCardStyle(r)
    
    local bar = uiAccentBar(r, on)
    local l = Instance.new("TextLabel")
    l.Position = UDim2.new(0,16,0,0)
    l.Size = UDim2.new(1,-74,1,0)
    l.BackgroundTransparency = 1
    l.Text = label
    l.TextColor3 = UI_TEXT_PRIMARY
    l.TextSize = 13
    l.Font = Enum.Font.GothamMedium
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.Parent = r

    -- Modern iOS-style Pill Switch
    local switchTrack = Instance.new("TextButton")
    switchTrack.Position = UDim2.new(1,-48,0.5,-11)
    switchTrack.Size = UDim2.new(0,36,0,22)
    switchTrack.BackgroundColor3 = on and (UI_ACCENT or Color3.fromRGB(255,255,255)) or Color3.fromRGB(30, 32, 40)
    switchTrack.BackgroundTransparency = on and 0.1 or 0.4
    switchTrack.BorderSizePixel = 0
    switchTrack.Text = ""
    switchTrack.AutoButtonColor = false
    switchTrack.Parent = r
    Instance.new("UICorner", switchTrack).CornerRadius = UDim.new(1, 0)
    
    local trackStroke = Instance.new("UIStroke", switchTrack)
    trackStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    trackStroke.Color = on and (UI_ACCENT or Color3.fromRGB(255,255,255)) or Color3.fromRGB(70,72,85)
    trackStroke.Thickness = on and 1.5 or 1
    trackStroke.Transparency = on and 0.1 or 0.4
    
    local switchKnob = Instance.new("Frame")
    switchKnob.Size = UDim2.new(0, 16, 0, 16)
    switchKnob.Position = on and UDim2.new(1, -19, 0.5, -8) or UDim2.new(0, 3, 0.5, -8)
    switchKnob.BackgroundColor3 = on and Color3.fromRGB(0, 0, 0) or Color3.fromRGB(200, 200, 210)
    switchKnob.BorderSizePixel = 0
    switchKnob.Parent = switchTrack
    Instance.new("UICorner", switchKnob).CornerRadius = UDim.new(1, 0)

    local state = on
    local function set(v)
        state = v
        TweenService:Create(switchTrack, UI_TWEEN_FAST, {
            BackgroundColor3 = v and (UI_ACCENT or Color3.fromRGB(255,255,255)) or Color3.fromRGB(30, 32, 40),
            BackgroundTransparency = v and 0.1 or 0.4
        }):Play()
        TweenService:Create(trackStroke, UI_TWEEN_FAST, {
            Color = v and (UI_ACCENT or Color3.fromRGB(255,255,255)) or Color3.fromRGB(70,72,85),
            Thickness = v and 1.5 or 1
        }):Play()
        TweenService:Create(switchKnob, UI_TWEEN_FAST, {
            Position = v and UDim2.new(1, -19, 0.5, -8) or UDim2.new(0, 3, 0.5, -8),
            BackgroundColor3 = v and Color3.fromRGB(0, 0, 0) or Color3.fromRGB(200, 200, 210)
        }):Play()
        TweenService:Create(bar, UI_TWEEN_FAST, {BackgroundTransparency = v and 0 or 1}):Play()
    end
    switchTrack.MouseButton1Click:Connect(function()
        set(not state)
        if callback then callback(state) end
        saveCherryConfig()
    end)
    return r, set
end

local function uiActionRow(parent, label, callback)
    local r = Instance.new("Frame")
    r.ClipsDescendants = true
    r.Size = UDim2.new(1,0,0,40)
    r.BackgroundColor3 = UI_ROW_BG
    r.BackgroundTransparency = 0.3
    r.BorderSizePixel = 0
    r.Parent = parent
    uiCardStyle(r)
    
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1,0,1,0)
    btn.BackgroundTransparency = 1
    btn.Text = label
    btn.TextColor3 = UI_TEXT_PRIMARY
    btn.TextSize = 13
    btn.Font = Enum.Font.GothamBold
    btn.Parent = r
    local bar = uiAccentBar(r, false)

    btn.MouseButton1Click:Connect(function()
        bar.BackgroundColor3 = UI_ACCENT
        TweenService:Create(bar, UI_TWEEN_FAST, {BackgroundTransparency = 0}):Play()
        task.delay(0.3, function() TweenService:Create(bar, UI_TWEEN_FAST, {BackgroundTransparency = 1}):Play() end)
        if callback then callback() end
    end)
    return r, btn
end

local function uiNumberRow(parent, label, value, minV, maxV, callback)
    local r, bx = uiInputRow(parent, label, value)
    bx.FocusLost:Connect(function()
        local n = tonumber(bx.Text)
        if n and n >= minV and n <= maxV then
            if callback then callback(n) end
            saveCherryConfig()
        else
            bx.Text = tostring(value)
        end
    end)
    return r, bx
end

local function uiChoiceRow(parent, label, options, defaultIndex, callback)
    local r = Instance.new("Frame")
    r.ClipsDescendants = true
    r.Size = UDim2.new(1,0,0,44)
    r.BackgroundColor3 = UI_ROW_BG
    r.BackgroundTransparency = 0.3
    r.BorderSizePixel = 0
    r.Parent = parent
    uiCardStyle(r)
    
    local l = Instance.new("TextLabel")
    l.Position = UDim2.new(0,14,0,0)
    l.Size = UDim2.new(0.38,0,0,44)
    l.BackgroundTransparency = 1
    l.Text = label
    l.TextColor3 = UI_TEXT_PRIMARY
    l.TextSize = 13
    l.Font = Enum.Font.GothamMedium
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.Parent = r
    
    local la = uiSmallBtn({Parent=r, Pos=UDim2.new(1,-186,0,9), Size=UDim2.new(0,26,0,26), Text="<", Col=UI_TEXT_PRIMARY, TS=12, CR=13})
    local vl = Instance.new("TextLabel")
    vl.Position = UDim2.new(1,-156,0,9)
    vl.Size = UDim2.new(0,124,0,26)
    vl.BackgroundColor3 = UI_BTN_BG
    vl.BackgroundTransparency = 0.25
    vl.BorderSizePixel = 0
    vl.Text = options[defaultIndex or 1]
    vl.TextColor3 = UI_TEXT_PRIMARY
    vl.TextSize = 10
    vl.Font = Enum.Font.GothamBold
    vl.Parent = r
    Instance.new("UICorner", vl).CornerRadius = UDim.new(0, 8)
    
    local vst = Instance.new("UIStroke", vl)
    vst.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    vst.Color = Color3.fromRGB(65, 68, 80)
    vst.Thickness = 1
    vst.Transparency = 0.35
    
    local ra = uiSmallBtn({Parent=r, Pos=UDim2.new(1,-28,0,9), Size=UDim2.new(0,26,0,26), Text=">", Col=UI_TEXT_PRIMARY, TS=12, CR=13})
    local idx = defaultIndex or 1
    local function upd()
        vl.Text = options[idx]
        if callback then callback(options[idx]) end
        saveCherryConfig()
    end
    la.MouseButton1Click:Connect(function() idx=idx-1; if idx<1 then idx=#options end; upd() end)
    ra.MouseButton1Click:Connect(function() idx=idx+1; if idx>#options then idx=1 end; upd() end)
    local function setVal(v)
        for i,o in ipairs(options) do if o==v then idx=i; vl.Text=o; break end end
    end
    return r, setVal
end

local function styleArrowButton(arrow)
    local border = Instance.new("UIStroke")
    border.Name = "AnimatedArrowBorder"
    border.Color = UI_ACCENT or Color3.fromRGB(255, 255, 255)
    border.Thickness = 1
    border.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    border.Transparency = 0.3
    border.Parent = arrow
end

local function styleOptionChip(btn, active)
    btn.TextColor3 = active and Color3.fromRGB(0, 0, 0) or Color3.fromRGB(235, 235, 245)
    btn.BackgroundColor3 = active and (UI_ACCENT or Color3.fromRGB(255, 255, 255)) or Color3.fromRGB(24, 25, 32)
    btn.BackgroundTransparency = active and 0.1 or 0.35
    local stroke = btn:FindFirstChildOfClass("UIStroke")
    if not stroke then
        stroke = Instance.new("UIStroke")
        stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
        stroke.Parent = btn
    end
    stroke.Color = active and (UI_ACCENT or Color3.fromRGB(255, 255, 255)) or Color3.fromRGB(65, 68, 80)
    stroke.Thickness = active and 1.5 or 1
    stroke.Transparency = active and 0.1 or 0.35
end

local function uiExpandToggleRow(parent, label, on, options, defaultIndex, onToggle, onOption)
    local container = Instance.new("Frame")
    container.BackgroundTransparency = 1
    container.Size = UDim2.new(1, 0, 0, 44)
    container.AutomaticSize = Enum.AutomaticSize.Y
    container.ClipsDescendants = false
    container.Parent = parent

    local col = Instance.new("UIListLayout")
    col.FillDirection = Enum.FillDirection.Vertical
    col.SortOrder = Enum.SortOrder.LayoutOrder
    col.Padding = UDim.new(0, 5)
    col.Parent = container

    -- MAIN ROW (toggle + arrow)
    local r = Instance.new("Frame")
    r.LayoutOrder = 1
    r.ClipsDescendants = true
    r.Size = UDim2.new(1, 0, 0, 44)
    r.BackgroundColor3 = UI_ROW_BG
    r.BackgroundTransparency = 0.3
    r.BorderSizePixel = 0
    r.Parent = container
    uiCardStyle(r)

    local bar = uiAccentBar(r, on)
    local l = Instance.new("TextLabel")
    l.Position = UDim2.new(0, 16, 0, 0)
    l.Size = UDim2.new(1, -110, 1, 0)
    l.BackgroundTransparency = 1
    l.Text = label
    l.TextColor3 = UI_TEXT_PRIMARY
    l.TextSize = 13
    l.Font = Enum.Font.GothamMedium
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.Parent = r

    local expanded = false
    local arrow = Instance.new("TextButton")
    arrow.Name = "ArrowButton"
    arrow.Position = UDim2.new(1, -86, 0.5, -11)
    arrow.Size = UDim2.new(0, 30, 0, 22)
    arrow.BackgroundColor3 = Color3.fromRGB(24, 25, 32)
    arrow.BackgroundTransparency = 0.3
    arrow.BorderSizePixel = 0
    arrow.Text = "▼"
    arrow.TextColor3 = Color3.fromRGB(255, 255, 255)
    arrow.TextSize = 11
    arrow.Font = Enum.Font.GothamBlack
    arrow.AutoButtonColor = false
    arrow.Parent = r
    Instance.new("UICorner", arrow).CornerRadius = UDim.new(1, 0)
    styleArrowButton(arrow)

    -- Pill Toggle Switch
    local switchTrack = Instance.new("TextButton")
    switchTrack.Position = UDim2.new(1, -48, 0.5, -11)
    switchTrack.Size = UDim2.new(0, 36, 0, 22)
    switchTrack.BackgroundColor3 = on and (UI_ACCENT or Color3.fromRGB(255,255,255)) or Color3.fromRGB(30, 32, 40)
    switchTrack.BackgroundTransparency = on and 0.1 or 0.4
    switchTrack.BorderSizePixel = 0
    switchTrack.Text = ""
    switchTrack.AutoButtonColor = false
    switchTrack.Parent = r
    Instance.new("UICorner", switchTrack).CornerRadius = UDim.new(1, 0)
    
    local ts2 = Instance.new("UIStroke", switchTrack)
    ts2.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    ts2.Color = on and (UI_ACCENT or Color3.fromRGB(255,255,255)) or Color3.fromRGB(70,72,85)
    ts2.Thickness = on and 1.5 or 1
    ts2.Transparency = on and 0.1 or 0.4
    
    local switchKnob = Instance.new("Frame")
    switchKnob.Size = UDim2.new(0, 16, 0, 16)
    switchKnob.Position = on and UDim2.new(1, -19, 0.5, -8) or UDim2.new(0, 3, 0.5, -8)
    switchKnob.BackgroundColor3 = on and Color3.fromRGB(0, 0, 0) or Color3.fromRGB(200, 200, 210)
    switchKnob.BorderSizePixel = 0
    switchKnob.Parent = switchTrack
    Instance.new("UICorner", switchKnob).CornerRadius = UDim.new(1, 0)

    local useScroll = #options > 4
    local optFrame = Instance.new("Frame")
    optFrame.LayoutOrder = 2
    optFrame.Size = UDim2.new(1, 0, 0, useScroll and 130 or 38)
    optFrame.BackgroundColor3 = UI_ROW_BG
    optFrame.BackgroundTransparency = 0.35
    optFrame.BorderSizePixel = 0
    optFrame.Visible = false
    optFrame.ClipsDescendants = true
    optFrame.Parent = container
    uiCardStyle(optFrame)

    local optPad = Instance.new("UIPadding")
    optPad.PaddingLeft = UDim.new(0, 6)
    optPad.PaddingRight = UDim.new(0, 6)
    optPad.PaddingTop = UDim.new(0, 5)
    optPad.PaddingBottom = UDim.new(0, 5)
    optPad.Parent = optFrame

    local optParent = optFrame
    if useScroll then
        local scroll = Instance.new("ScrollingFrame")
        scroll.Name = "OptionsScroll"
        scroll.Size = UDim2.new(1, -2, 1, -2)
        scroll.Position = UDim2.new(0, 1, 0, 1)
        scroll.BackgroundTransparency = 1
        scroll.BorderSizePixel = 0
        scroll.ScrollBarThickness = 4
        scroll.ScrollBarImageColor3 = UI_ACCENT
        scroll.ScrollingDirection = Enum.ScrollingDirection.Y
        scroll.ElasticBehavior = Enum.ElasticBehavior.Always
        scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
        scroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
        scroll.Parent = optFrame
        local sPad = Instance.new("UIPadding")
        sPad.PaddingLeft = UDim.new(0, 4)
        sPad.PaddingRight = UDim.new(0, 6)
        sPad.PaddingTop = UDim.new(0, 4)
        sPad.PaddingBottom = UDim.new(0, 6)
        sPad.Parent = scroll
        local sLay = Instance.new("UIListLayout")
        sLay.FillDirection = Enum.FillDirection.Vertical
        sLay.Padding = UDim.new(0, 4)
        sLay.SortOrder = Enum.SortOrder.LayoutOrder
        sLay.Parent = scroll
        optParent = scroll
    else
        local optLayout = Instance.new("UIListLayout")
        optLayout.FillDirection = Enum.FillDirection.Horizontal
        optLayout.Padding = UDim.new(0, 6)
        optLayout.VerticalAlignment = Enum.VerticalAlignment.Center
        optLayout.SortOrder = Enum.SortOrder.LayoutOrder
        optLayout.Parent = optFrame
    end

    local settingsHost = Instance.new("Frame")
    settingsHost.Name = "ModeSettings"
    settingsHost.LayoutOrder = 3
    settingsHost.Size = UDim2.new(1, 0, 0, 0)
    settingsHost.AutomaticSize = Enum.AutomaticSize.Y
    settingsHost.BackgroundTransparency = 1
    settingsHost.Visible = false
    settingsHost.Parent = container

    local settingsLayout = Instance.new("UIListLayout")
    settingsLayout.Padding = UDim.new(0, 5)
    settingsLayout.SortOrder = Enum.SortOrder.LayoutOrder
    settingsLayout.Parent = settingsHost

    local idx = defaultIndex or 1
    local optionBtns = {}
    local state = on
    local modeSettings = {}

    local function refreshOptionVisuals()
        for i, b in ipairs(optionBtns) do
            styleOptionChip(b, i == idx)
        end
    end

    local function refreshModeSettings()
        local any = false
        for lab, fr in pairs(modeSettings) do
            local show = expanded and (lab == options[idx])
            fr.Visible = show
            if show then any = true end
        end
        settingsHost.Visible = any
    end

    for i, opt in ipairs(options) do
        local b = Instance.new("TextButton")
        b.LayoutOrder = i
        if useScroll then
            b.Size = UDim2.new(1, -2, 0, 26)
        else
            b.Size = UDim2.new(0, math.max(52, #tostring(opt) * 8 + 16), 0, 26)
        end
        b.BorderSizePixel = 0
        b.Text = tostring(opt)
        b.TextSize = 11
        b.Font = Enum.Font.GothamBold
        b.TextXAlignment = useScroll and Enum.TextXAlignment.Left or Enum.TextXAlignment.Center
        b.AutoButtonColor = false
        b.Parent = optParent
        Instance.new("UICorner", b).CornerRadius = UDim.new(0, 8)
        if useScroll then
            local p = Instance.new("UIPadding")
            p.PaddingLeft = UDim.new(0, 8)
            p.Parent = b
        end
        styleOptionChip(b, i == idx)
        b.MouseButton1Click:Connect(function()
            idx = i
            refreshOptionVisuals()
            refreshModeSettings()
            if onOption then onOption(options[idx]) end
            saveCherryConfig()
        end)
        optionBtns[i] = b
    end

    local function setExpanded(v)
        expanded = v
        optFrame.Visible = v
        arrow.Text = v and "▲" or "▼"
        refreshModeSettings()
    end

    arrow.MouseButton1Click:Connect(function()
        setExpanded(not expanded)
    end)

    local function set(v)
        state = v
        TweenService:Create(switchTrack, UI_TWEEN_FAST, {
            BackgroundColor3 = v and (UI_ACCENT or Color3.fromRGB(255,255,255)) or Color3.fromRGB(30, 32, 40),
            BackgroundTransparency = v and 0.1 or 0.4
        }):Play()
        TweenService:Create(ts2, UI_TWEEN_FAST, {
            Color = v and (UI_ACCENT or Color3.fromRGB(255,255,255)) or Color3.fromRGB(70,72,85),
            Thickness = v and 1.5 or 1
        }):Play()
        TweenService:Create(switchKnob, UI_TWEEN_FAST, {
            Position = v and UDim2.new(1, -19, 0.5, -8) or UDim2.new(0, 3, 0.5, -8),
            BackgroundColor3 = v and Color3.fromRGB(0, 0, 0) or Color3.fromRGB(200, 200, 210)
        }):Play()
        TweenService:Create(bar, UI_TWEEN_FAST, {BackgroundTransparency = v and 0 or 1}):Play()
    end

    switchTrack.MouseButton1Click:Connect(function()
        set(not state)
        if onToggle then onToggle(state) end
        saveCherryConfig()
    end)

    local function setOption(v)
        for i, o in ipairs(options) do
            if o == v then
                idx = i
                refreshOptionVisuals()
                refreshModeSettings()
                break
            end
        end
    end

    local function registerModeSettings(optionLabel, frame)
        frame.Parent = settingsHost
        frame.Visible = false
        frame.Size = UDim2.new(1, 0, 0, 0)
        frame.AutomaticSize = Enum.AutomaticSize.Y
        modeSettings[optionLabel] = frame
        task.defer(refreshModeSettings)
    end

    return container, set, setOption, registerModeSettings, function() return options[idx] end
end

local function uiKeybindRow(parent, label, entry)
    local r = Instance.new("Frame")
    r.ClipsDescendants = true
    r.Size = UDim2.new(1, 0, 0, 44)
    r.BackgroundColor3 = UI_ROW_BG
    r.BackgroundTransparency = 0.3
    r.BorderSizePixel = 0
    r.Parent = parent
    uiCardStyle(r)

    local l = Instance.new("TextLabel")
    l.Position = UDim2.new(0, 14, 0, 0)
    l.Size = UDim2.new(1, -90, 1, 0)
    l.BackgroundTransparency = 1
    l.Text = label
    l.TextColor3 = UI_TEXT_PRIMARY
    l.TextSize = 13
    l.Font = Enum.Font.GothamMedium
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.Parent = r

    local btn = uiSmallBtn({
        Parent = r,
        Pos = UDim2.new(1, -76, 0.5, -12),
        Size = UDim2.new(0, 64, 0, 24),
        Text = (entry and entry.kb and entry.kb.Name) or "[ NONE ]",
        Col = UI_ACCENT,
        TS = 11,
        CR = 8,
    })

    local listening = false
    btn.MouseButton1Click:Connect(function()
        listening = true
        btn.Text = "..."
        btn.TextColor3 = Color3.fromRGB(255, 255, 100)
    end)

    UIS.InputBegan:Connect(function(input, gp)
        if not listening then return end
        if input.UserInputType == Enum.UserInputType.Keyboard then
            if input.KeyCode == Enum.KeyCode.Escape or input.KeyCode == Enum.KeyCode.Backspace then
                entry.kb = nil
                btn.Text = "[ NONE ]"
            else
                entry.kb = input.KeyCode
                btn.Text = input.KeyCode.Name
            end
            btn.TextColor3 = UI_ACCENT
            listening = false
            saveCherryConfig()
        end
    end)
    return r, btn
end

local function uiMakePage(parent, name, order, vis)
    local p = Instance.new("ScrollingFrame")
    p.Name = name
    p.Visible = vis ~= false
    p.LayoutOrder = order
    p.Size = UDim2.new(1,0,1,0)
    p.BackgroundTransparency = 1
    p.BorderSizePixel = 0
    p.ScrollBarThickness = 4
    p.ScrollBarImageColor3 = UI_ACCENT or Color3.fromRGB(255, 255, 255)
    p.ScrollBarImageTransparency = 0.3
    p.ScrollingEnabled = true
    p.ScrollingDirection = Enum.ScrollingDirection.Y
    p.ElasticBehavior = Enum.ElasticBehavior.Always
    p.AutomaticCanvasSize = Enum.AutomaticSize.Y
    p.CanvasSize = UDim2.new(0,0,0,0)
    p.Parent = parent
    
    local l = Instance.new("UIListLayout")
    l.Padding = UDim.new(0,6)
    l.SortOrder = Enum.SortOrder.LayoutOrder
    l.Parent = p
    
    local pd = Instance.new("UIPadding")
    pd.PaddingTop = UDim.new(0,4)
    pd.PaddingBottom = UDim.new(0,36)
    pd.PaddingRight = UDim.new(0,4)
    pd.PaddingLeft = UDim.new(0,2)
    pd.Parent = p
    uiAutoCanvas(p)
    return p
end

local function uiMakeTab(parent, name, text, a, b)
    local active = false
    local labelText = tostring(text or "")
    if type(a) == "boolean" then
        active = a
    elseif type(b) == "boolean" then
        active = b
        if type(a) == "string" and #a > 0 then
            labelText = a .. "  " .. labelText
        end
    end

    local btn = Instance.new("TextButton")
    btn.Name = name
    btn.ZIndex = 9
    btn.Size = UDim2.new(1, -12, 0, 36)
    btn.Position = UDim2.new(0, 6, 0, 0)
    btn.BackgroundColor3 = active and (UI_ACCENT or Color3.fromRGB(168, 85, 247)) or Color3.fromRGB(22, 23, 30)
    btn.BackgroundTransparency = active and 0.12 or 0.7
    btn.BorderSizePixel = 0
    btn.Text = ""
    btn.AutoButtonColor = false
    btn.Parent = parent
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)

    local stroke = Instance.new("UIStroke", btn)
    stroke.Name = "TabStroke"
    stroke.Color = active and (UI_ACCENT or Color3.fromRGB(168, 85, 247)) or Color3.fromRGB(55, 58, 72)
    stroke.Thickness = 1
    stroke.Transparency = active and 0.2 or 0.8
    stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border

    local indicator = Instance.new("Frame", btn)
    indicator.Name = "Indicator"
    indicator.Position = UDim2.new(0, 3, 0.5, -8)
    indicator.Size = UDim2.new(0, 3, 0, 16)
    indicator.BackgroundColor3 = UI_ACCENT or Color3.fromRGB(168, 85, 247)
    indicator.BackgroundTransparency = active and 0 or 1
    indicator.BorderSizePixel = 0
    Instance.new("UICorner", indicator).CornerRadius = UDim.new(1, 0)

    local lbl = Instance.new("TextLabel", btn)
    lbl.Name = "TabLabel"
    lbl.Position = UDim2.new(0, 14, 0, 0)
    lbl.Size = UDim2.new(1, -18, 1, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text = labelText
    lbl.TextColor3 = active and Color3.fromRGB(0, 0, 0) or Color3.fromRGB(220, 220, 230)
    lbl.TextSize = 12
    lbl.Font = Enum.Font.GothamBold
    lbl.TextXAlignment = Enum.TextXAlignment.Left

    btn:SetAttribute("IsActiveTab", active and true or false)

    btn.MouseEnter:Connect(function()
        if not btn:GetAttribute("IsActiveTab") then
            TweenService:Create(btn, UI_TWEEN_FAST, {
                BackgroundColor3 = Color3.fromRGB(32, 34, 44),
                BackgroundTransparency = 0.35,
            }):Play()
            TweenService:Create(lbl, UI_TWEEN_FAST, {TextColor3 = Color3.fromRGB(255, 255, 255)}):Play()
            stroke.Transparency = 0.4
        end
    end)
    btn.MouseLeave:Connect(function()
        if not btn:GetAttribute("IsActiveTab") then
            TweenService:Create(btn, UI_TWEEN_FAST, {
                BackgroundColor3 = Color3.fromRGB(22, 23, 30),
                BackgroundTransparency = 0.7,
            }):Play()
            TweenService:Create(lbl, UI_TWEEN_FAST, {TextColor3 = Color3.fromRGB(220, 220, 230)}):Play()
            stroke.Transparency = 0.8
        end
    end)
    return btn
end

-- MAIN BUILD
function M.applyCustomBackground(frame)
    if not frame then return end
    local existing = frame:FindFirstChild("CustomBgImage")
    if existing then existing:Destroy() end
    local id = tonumber(M.customBgId) or 0
    if id <= 0 then return end
    local img = Instance.new("ImageLabel")
    img.Name = "CustomBgImage"
    img.BackgroundTransparency = 1
    img.Image = "rbxassetid://" .. tostring(id)
    img.ScaleType = Enum.ScaleType.Crop
    img.Size = UDim2.fromScale(1, 1)
    img.Position = UDim2.fromScale(0, 0)
    img.ZIndex = 0
    img.ImageTransparency = math.clamp(tonumber(M.customBgOpacity) or 0.35, 0, 1)
    img.Parent = frame
    Instance.new("UICorner", img).CornerRadius = UDim.new(0, 14)
end

function M.openImagePicker(kind)
    -- kind = "bg" | "mob"
    local isBg = kind == "bg"
    local ids = isBg and M.BG_IMAGE_IDS or M.MOB_BTN_IMAGE_IDS
    local title = isBg and "CUSTOM BG" or "BUTTON BG"
    local currentId = isBg and (tonumber(M.customBgId) or 0) or (tonumber(M.mobBtnBgId) or 0)
    local opacity = math.clamp(tonumber(M.customBgOpacity) or 0.35, 0, 1)

    local old = player.PlayerGui:FindFirstChild("VynxImagePicker")
    if old then old:Destroy() end
    local cg = game:GetService("CoreGui"):FindFirstChild("VynxImagePicker")
    if cg then cg:Destroy() end

    local gui = Instance.new("ScreenGui")
    gui.Name = "VynxImagePicker"
    gui.ResetOnSpawn = false
    gui.IgnoreGuiInset = true
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    gui.DisplayOrder = 120
    pcall(function() gui.Parent = game:GetService("CoreGui") end)
    if not gui.Parent then gui.Parent = player:WaitForChild("PlayerGui") end

    -- dim backdrop (tap to close)
    local dim = Instance.new("TextButton")
    dim.Size = UDim2.fromScale(1, 1)
    dim.BackgroundColor3 = Color3.new(0, 0, 0)
    dim.BackgroundTransparency = 0.45
    dim.Text = ""
    dim.AutoButtonColor = false
    dim.ZIndex = 1
    dim.Parent = gui
    dim.MouseButton1Click:Connect(function() gui:Destroy() end)

    local panel = Instance.new("Frame")
    panel.AnchorPoint = Vector2.new(0.5, 0.5)
    panel.Position = UDim2.new(0.5, 0, 0.5, 0)
    panel.Size = UDim2.new(0, 220, 0, isBg and 280 or 230)
    panel.BackgroundColor3 = Color3.fromRGB(12, 12, 16)
    panel.BorderSizePixel = 0
    panel.ZIndex = 2
    panel.ClipsDescendants = true
    panel.Parent = gui
    Instance.new("UICorner", panel).CornerRadius = UDim.new(0, 14)
    local pst = Instance.new("UIStroke", panel)
    pst.Color = Color3.fromRGB(50, 50, 60)
    pst.Thickness = 1

    local hdr = Instance.new("TextLabel")
    hdr.Size = UDim2.new(1, -40, 0, 28)
    hdr.Position = UDim2.new(0, 12, 0, 6)
    hdr.BackgroundTransparency = 1
    hdr.Text = title
    hdr.TextColor3 = Color3.fromRGB(230, 230, 235)
    hdr.Font = Enum.Font.GothamBold
    hdr.TextSize = 12
    hdr.TextXAlignment = Enum.TextXAlignment.Left
    hdr.ZIndex = 3
    hdr.Parent = panel

    local close = Instance.new("TextButton")
    close.Size = UDim2.new(0, 24, 0, 24)
    close.Position = UDim2.new(1, -30, 0, 6)
    close.BackgroundTransparency = 1
    close.Text = "×"
    close.TextColor3 = Color3.fromRGB(180, 180, 190)
    close.Font = Enum.Font.GothamBold
    close.TextSize = 18
    close.ZIndex = 3
    close.Parent = panel
    close.MouseButton1Click:Connect(function() gui:Destroy() end)

    local preview = Instance.new("ImageLabel")
    preview.Name = "Preview"
    preview.Size = UDim2.new(1, -24, 0, 100)
    preview.Position = UDim2.new(0, 12, 0, 34)
    preview.BackgroundColor3 = Color3.fromRGB(20, 20, 24)
    preview.BorderSizePixel = 0
    preview.ScaleType = Enum.ScaleType.Crop
    preview.Image = currentId > 0 and ("rbxassetid://" .. currentId) or ""
    preview.ImageTransparency = isBg and opacity or 0
    preview.ZIndex = 3
    preview.Parent = panel
    Instance.new("UICorner", preview).CornerRadius = UDim.new(0, 10)

    local scroll = Instance.new("ScrollingFrame")
    scroll.Size = UDim2.new(1, -24, 0, 56)
    scroll.Position = UDim2.new(0, 12, 0, 142)
    scroll.BackgroundTransparency = 1
    scroll.BorderSizePixel = 0
    scroll.ScrollBarThickness = 4
    scroll.ScrollBarImageColor3 = UI_ACCENT or Color3.fromRGB(200, 200, 200)
    scroll.ScrollingDirection = Enum.ScrollingDirection.X
    scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
    scroll.AutomaticCanvasSize = Enum.AutomaticSize.X
    scroll.ZIndex = 3
    scroll.Parent = panel

    local lay = Instance.new("UIListLayout")
    lay.FillDirection = Enum.FillDirection.Horizontal
    lay.Padding = UDim.new(0, 8)
    lay.VerticalAlignment = Enum.VerticalAlignment.Center
    lay.Parent = scroll

    local selectedId = currentId

    local function selectId(id)
        selectedId = id
        preview.Image = id > 0 and ("rbxassetid://" .. id) or ""
        if isBg then
            M.customBgId = id
            if M.mainFrame then M.applyCustomBackground(M.mainFrame) end
        else
            M.mobBtnBgId = id
            if M.mobileButtonsEnabled then M.buildMobileButtons() end
        end
        saveCherryConfig()
    end

    -- None option
    local none = Instance.new("TextButton")
    none.Size = UDim2.new(0, 48, 0, 48)
    none.BackgroundColor3 = Color3.fromRGB(28, 28, 34)
    none.Text = "OFF"
    none.TextColor3 = Color3.fromRGB(200, 200, 210)
    none.Font = Enum.Font.GothamBold
    none.TextSize = 10
    none.ZIndex = 4
    none.Parent = scroll
    Instance.new("UICorner", none).CornerRadius = UDim.new(0, 8)
    none.MouseButton1Click:Connect(function() selectId(0) end)

    for _, id in ipairs(ids) do
        local thumb = Instance.new("ImageButton")
        thumb.Size = UDim2.new(0, 48, 0, 48)
        thumb.BackgroundColor3 = Color3.fromRGB(28, 28, 34)
        thumb.Image = "rbxassetid://" .. tostring(id)
        thumb.ScaleType = Enum.ScaleType.Crop
        thumb.ZIndex = 4
        thumb.Parent = scroll
        Instance.new("UICorner", thumb).CornerRadius = UDim.new(0, 8)
        local st = Instance.new("UIStroke", thumb)
        st.Color = Color3.fromRGB(255, 255, 255)
        st.Transparency = (id == currentId) and 0.2 or 0.7
        st.Thickness = 1
        thumb.MouseButton1Click:Connect(function()
            selectId(id)
            for _, ch in ipairs(scroll:GetChildren()) do
                if ch:IsA("ImageButton") then
                    local s = ch:FindFirstChildOfClass("UIStroke")
                    if s then s.Transparency = 0.7 end
                end
            end
            st.Transparency = 0.2
        end)
    end

    if isBg then
        local opLbl = Instance.new("TextLabel")
        opLbl.Size = UDim2.new(0.5, -12, 0, 18)
        opLbl.Position = UDim2.new(0, 12, 0, 208)
        opLbl.BackgroundTransparency = 1
        opLbl.Text = "OPACITY"
        opLbl.TextColor3 = Color3.fromRGB(160, 160, 170)
        opLbl.Font = Enum.Font.GothamBold
        opLbl.TextSize = 10
        opLbl.TextXAlignment = Enum.TextXAlignment.Left
        opLbl.ZIndex = 3
        opLbl.Parent = panel

        local opVal = Instance.new("TextLabel")
        opVal.Size = UDim2.new(0.5, -12, 0, 18)
        opVal.Position = UDim2.new(0.5, 0, 0, 208)
        opVal.BackgroundTransparency = 1
        opVal.Text = tostring(math.floor((1 - opacity) * 100)) .. "%"
        opVal.TextColor3 = Color3.fromRGB(200, 200, 210)
        opVal.Font = Enum.Font.GothamBold
        opVal.TextSize = 10
        opVal.TextXAlignment = Enum.TextXAlignment.Right
        opVal.ZIndex = 3
        opVal.Parent = panel

        -- simple slider track
        local track = Instance.new("Frame")
        track.Size = UDim2.new(1, -24, 0, 8)
        track.Position = UDim2.new(0, 12, 0, 232)
        track.BackgroundColor3 = Color3.fromRGB(40, 40, 48)
        track.BorderSizePixel = 0
        track.ZIndex = 3
        track.Parent = panel
        Instance.new("UICorner", track).CornerRadius = UDim.new(1, 0)

        local fill = Instance.new("Frame")
        fill.Size = UDim2.new(1 - opacity, 0, 1, 0)
        fill.BackgroundColor3 = Color3.fromRGB(220, 220, 230)
        fill.BorderSizePixel = 0
        fill.ZIndex = 4
        fill.Parent = track
        Instance.new("UICorner", fill).CornerRadius = UDim.new(1, 0)

        local knob = Instance.new("Frame")
        knob.Size = UDim2.new(0, 16, 0, 16)
        knob.AnchorPoint = Vector2.new(0.5, 0.5)
        knob.Position = UDim2.new(1 - opacity, 0, 0.5, 0)
        knob.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
        knob.BorderSizePixel = 0
        knob.ZIndex = 5
        knob.Parent = track
        Instance.new("UICorner", knob).CornerRadius = UDim.new(1, 0)

        local dragging = false
        local function setFromX(x)
            local rel = math.clamp((x - track.AbsolutePosition.X) / math.max(track.AbsoluteSize.X, 1), 0, 1)
            -- rel 1 = fully opaque image (low ImageTransparency)
            local imageTransparency = 1 - rel
            opacity = imageTransparency
            M.customBgOpacity = opacity
            fill.Size = UDim2.new(rel, 0, 1, 0)
            knob.Position = UDim2.new(rel, 0, 0.5, 0)
            preview.ImageTransparency = opacity
            opVal.Text = tostring(math.floor(rel * 100)) .. "%"
            if M.mainFrame then M.applyCustomBackground(M.mainFrame) end
            saveCherryConfig()
        end
        track.InputBegan:Connect(function(i)
            if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
                dragging = true
                setFromX(i.Position.X)
            end
        end)
        UIS.InputChanged:Connect(function(i)
            if dragging and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then
                setFromX(i.Position.X)
            end
        end)
        UIS.InputEnded:Connect(function(i)
            if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
                dragging = false
            end
        end)
    end
end


-- ============================================================
-- CUSTOM FONTS (from EXE)
-- ============================================================
function M._fontShouldTouch(obj)
    if not (obj:IsA("TextLabel") or obj:IsA("TextButton") or obj:IsA("TextBox")) then return false end
    if obj.TextStrokeTransparency ~= 1 then return false end
    return true
end

function M._fontApplyOne(txt)
    if not M._fontShouldTouch(txt) then return end
    if not M._fontOrig[txt] then M._fontOrig[txt] = txt.FontFace end
    if M._fontMy then
        pcall(function() txt.FontFace = M._fontMy end)
    end
end

function M._fontSetupCoding()
    if M._fontMy and M.customFontSelected == "Coding Font" then return true end
    local ok = pcall(function()
        if isfile and writefile and getcustomasset then
            if not isfile("vynx_starborn.ttf") then
                writefile("vynx_starborn.ttf", game:HttpGet("https://granny.anondrop.net/uploads/6c2505542959f371/Starborn.ttf"))
            end
            writefile("vynx_starborn.json", HS:JSONEncode({
                name = "Starborn",
                faces = {{name = "Regular", weight = 400, style = "normal", assetId = getcustomasset("vynx_starborn.ttf")}}
            }))
            M._fontMy = Font.new(getcustomasset("vynx_starborn.json"))
        end
    end)
    return ok and M._fontMy ~= nil
end

function M.getFontForName(name)
    if not name or name == "None" then return nil end
    if name == "Coding Font" then
        if M._fontSetupCoding() then return M._fontMy end
        return nil
    elseif name == "Summer" then
        return Font.new("rbxasset://fonts/families/PermanentMarker.json")
    elseif name == "Beachy" then
        return Font.new("rbxasset://fonts/families/DenkOne.json")
    elseif name == "Scary" then
        return Font.new("rbxasset://fonts/families/Creepster.json")
    elseif name == "Bangers" then
        return Font.new("rbxasset://fonts/families/Bangers.json")
    end
    return nil
end

function M.applyCustomFont(name)
    if M._fontConn then pcall(function() M._fontConn:Disconnect() end); M._fontConn = nil end
    for obj, orig in pairs(M._fontOrig) do
        pcall(function() if obj and obj.Parent then obj.FontFace = orig end end)
    end
    M._fontOrig = {}
    M.customFontSelected = name or "None"
    if name and name ~= "None" then
        local font = M.getFontForName(name)
        if font then
            M._fontMy = font
            for _, v in ipairs(game:GetDescendants()) do
                M._fontApplyOne(v)
            end
            M._fontConn = game.DescendantAdded:Connect(function(obj)
                if M.customFontSelected ~= "None" then M._fontApplyOne(obj) end
            end)
        end
    else
        M._fontMy = nil
    end
    -- always persist selection
    pcall(function()
        if type(saveCherryConfig) == "function" then saveCherryConfig() end
    end)
end

function M.buildGui()
    applyAccentFromTheme()
    M.clearPersistentConns()

    for _,n in ipairs({"MoveeDuels","Cherry_Menu","K7HubGUI","VantaHubUI","VynxxHubUI","VynxHubUI","OniHubUI","AceDuelsAdaptReconstruct"}) do
        local cg=game:GetService("CoreGui")
        local old=cg:FindFirstChild(n); if old then old:Destroy() end
        local pg=player:FindFirstChild("PlayerGui")
        if pg then local o=pg:FindFirstChild(n); if o then o:Destroy() end end
    end

    M.buildStatusUI()

    local gui = Instance.new("ScreenGui")
    gui.Name = "OniHubUI"
    gui.ResetOnSpawn = false
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    gui.Parent = player:WaitForChild("PlayerGui")

    local Frame = Instance.new("Frame")
    Frame.Name = "Frame"
    Frame.ClipsDescendants = false
    Frame.AnchorPoint = Vector2.new(0.5, 0.5)
    Frame.Position = UDim2.new(0.5, 0, 0.5, 0)
    Frame.Size = UDim2.new(0, 560, 0, 420)
    Frame.BackgroundColor3 = Color3.fromRGB(13, 14, 18)
    Frame.BorderSizePixel = 0
    Frame.Active = true
    Frame.Parent = gui
    M.mainFrame = Frame
    M.applyCustomBackground(Frame)

    local UIScale = Instance.new("UIScale")
    UIScale.Name = "BDUIScale"
    UIScale.Scale = M.uiScale or 0.8
    UIScale.Parent = Frame
    M.uiScaleRef = UIScale

    local frameCorner = Instance.new("UICorner")
    frameCorner.Name = "MainCorner"
    frameCorner.CornerRadius = UDim.new(0, 14)
    frameCorner.Parent = Frame
    do
        local g = Instance.new("UIGradient")
        g.Name = "MainGradient"
        local accent = UI_ACCENT or Color3.fromRGB(168,85,247)
        local bg = Color3.fromRGB(13,14,18)
        g.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, bg:Lerp(accent, 0.05)),
            ColorSequenceKeypoint.new(0.5, bg),
            ColorSequenceKeypoint.new(1, bg:Lerp(accent, 0.02)),
        })
        g.Rotation = 135
        g.Parent = Frame
        local stroke = Instance.new("UIStroke")
        stroke.Name = "MainStroke"
        stroke.Color = Color3.fromRGB(50, 52, 65)
        stroke.Thickness = 1
        stroke.Transparency = 0.35
        stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
        stroke.Parent = Frame
    end

    -- LEFT SIDEBAR (Width: 155px)
    local Sidebar = Instance.new("Frame")
    Sidebar.Name = "Sidebar"
    Sidebar.Size = UDim2.new(0, 155, 1, 0)
    Sidebar.BackgroundColor3 = Color3.fromRGB(16, 17, 23)
    Sidebar.BackgroundTransparency = 0.25
    Sidebar.BorderSizePixel = 0
    Sidebar.Active = true
    Sidebar.Parent = Frame
    Instance.new("UICorner", Sidebar).CornerRadius = UDim.new(0, 14)

    -- Sidebar Right Border Separator
    local sideBorder = Instance.new("Frame")
    sideBorder.Size = UDim2.new(0, 1, 1, 0)
    sideBorder.Position = UDim2.new(1, -1, 0, 0)
    sideBorder.BackgroundColor3 = Color3.fromRGB(48, 50, 65)
    sideBorder.BackgroundTransparency = 0.5
    sideBorder.BorderSizePixel = 0
    sideBorder.Parent = Sidebar

    -- Branding Top in Sidebar
    local BrandBox = Instance.new("Frame")
    BrandBox.Size = UDim2.new(1, 0, 0, 56)
    BrandBox.BackgroundTransparency = 1
    BrandBox.Active = true
    BrandBox.Parent = Sidebar

    do
        local dot = Instance.new("Frame")
        dot.Size = UDim2.new(0, 8, 0, 8)
        dot.Position = UDim2.new(0, 14, 0, 18)
        dot.BackgroundColor3 = UI_ACCENT or Color3.fromRGB(168, 85, 247)
        dot.BorderSizePixel = 0
        dot.Parent = BrandBox
        Instance.new("UICorner", dot).CornerRadius = UDim.new(1, 0)

        local t = Instance.new("TextLabel")
        t.Position = UDim2.new(0, 28, 0, 10)
        t.Size = UDim2.new(1, -34, 0, 22)
        t.BackgroundTransparency = 1
        t.Text = 'ONI <font color="#8A8A94">HUB</font>'
        t.TextColor3 = UI_TEXT_WHITE
        t.TextSize = 16
        t.Font = Enum.Font.GothamBlack
        t.TextXAlignment = Enum.TextXAlignment.Left
        t.RichText = true
        t.Parent = BrandBox

        local badge = Instance.new("TextLabel")
        badge.Position = UDim2.new(0, 28, 0, 32)
        badge.Size = UDim2.new(0, 74, 0, 14)
        badge.BackgroundColor3 = Color3.fromRGB(28, 30, 40)
        badge.BackgroundTransparency = 0.3
        badge.BorderSizePixel = 0
        badge.Text = "PAID VERSION"
        badge.TextColor3 = UI_ACCENT or Color3.fromRGB(168, 85, 247)
        badge.TextSize = 7.5
        badge.Font = Enum.Font.GothamBold
        badge.Parent = BrandBox
        Instance.new("UICorner", badge).CornerRadius = UDim.new(0, 4)
    end

    -- Tab Container in Sidebar
    local TabContainer = Instance.new("Frame")
    TabContainer.Name = "TabContainer"
    TabContainer.Position = UDim2.new(0, 0, 0, 60)
    TabContainer.Size = UDim2.new(1, 0, 1, -112)
    TabContainer.BackgroundTransparency = 1
    TabContainer.Parent = Sidebar

    local TabsLayout = Instance.new("UIListLayout")
    TabsLayout.FillDirection = Enum.FillDirection.Vertical
    TabsLayout.Padding = UDim.new(0, 3)
    TabsLayout.SortOrder = Enum.SortOrder.LayoutOrder
    TabsLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    TabsLayout.Parent = TabContainer

    local TSpeed  = uiMakeTab(TabContainer, "Tab_SPEED",  "Speed",     true);  TSpeed.LayoutOrder=1
    local TMech   = uiMakeTab(TabContainer, "Tab_MECH",   "Combat",    false); TMech.LayoutOrder=2
    local TVis    = uiMakeTab(TabContainer, "Tab_VIS",    "Visuals",   false); TVis.LayoutOrder=3
    local TSounds = uiMakeTab(TabContainer, "Tab_SOUNDS", "Sounds",    false); TSounds.LayoutOrder=4
    local TUtil   = uiMakeTab(TabContainer, "Tab_UTIL",   "Utility",   false); TUtil.LayoutOrder=5
    local TKB     = uiMakeTab(TabContainer, "Tab_KB",     "Keybinds",  false); TKB.LayoutOrder=6

    -- Sidebar User Profile Card
    local UserCard = Instance.new("Frame")
    UserCard.Name = "UserCard"
    UserCard.Position = UDim2.new(0, 8, 1, -46)
    UserCard.Size = UDim2.new(1, -16, 0, 38)
    UserCard.BackgroundColor3 = Color3.fromRGB(22, 23, 30)
    UserCard.BackgroundTransparency = 0.4
    UserCard.BorderSizePixel = 0
    UserCard.Parent = Sidebar
    Instance.new("UICorner", UserCard).CornerRadius = UDim.new(0, 8)
    do
        local ust = Instance.new("UIStroke", UserCard)
        ust.Color = Color3.fromRGB(48, 50, 62)
        ust.Thickness = 1
        ust.Transparency = 0.5

        local av = Instance.new("ImageLabel")
        av.Size = UDim2.new(0, 24, 0, 24)
        av.Position = UDim2.new(0, 7, 0.5, -12)
        av.BackgroundColor3 = Color3.fromRGB(30, 32, 42)
        av.BorderSizePixel = 0
        av.Image = "rbxthumb://type=AvatarHeadShot&id=" .. tostring(player.UserId) .. "&w=48&h=48"
        av.Parent = UserCard
        Instance.new("UICorner", av).CornerRadius = UDim.new(1, 0)

        local un = Instance.new("TextLabel")
        un.Size = UDim2.new(1, -40, 0, 14)
        un.Position = UDim2.new(0, 36, 0, 5)
        un.BackgroundTransparency = 1
        un.Text = player.DisplayName or player.Name
        un.TextColor3 = Color3.fromRGB(220, 220, 230)
        un.TextSize = 10
        un.Font = Enum.Font.GothamBold
        un.TextXAlignment = Enum.TextXAlignment.Left
        un.TextTruncate = Enum.TextTruncate.AtEnd
        un.Parent = UserCard

        local stt = Instance.new("TextLabel")
        stt.Size = UDim2.new(1, -40, 0, 12)
        stt.Position = UDim2.new(0, 36, 0, 19)
        stt.BackgroundTransparency = 1
        stt.Text = "● Ready"
        stt.TextColor3 = UI_ACCENT or Color3.fromRGB(80, 220, 120)
        stt.TextSize = 8
        stt.Font = Enum.Font.GothamMedium
        stt.TextXAlignment = Enum.TextXAlignment.Left
        stt.Parent = UserCard
    end

    -- RIGHT MAIN AREA (Width: 405px)
    local MainArea = Instance.new("Frame")
    MainArea.Name = "MainArea"
    MainArea.Position = UDim2.new(0, 155, 0, 0)
    MainArea.Size = UDim2.new(1, -155, 1, 0)
    MainArea.BackgroundTransparency = 1
    MainArea.Parent = Frame

    -- Right Top Header Bar
    local TopBar = Instance.new("Frame")
    TopBar.Name = "TopBar"
    TopBar.Size = UDim2.new(1, 0, 0, 48)
    TopBar.BackgroundTransparency = 1
    TopBar.Active = true
    TopBar.Parent = MainArea

    local headerCategory = Instance.new("TextLabel")
    headerCategory.Position = UDim2.new(0, 14, 0, 10)
    headerCategory.Size = UDim2.new(0, 220, 0, 18)
    headerCategory.BackgroundTransparency = 1
    headerCategory.Text = "SPEED CONFIGURATION"
    headerCategory.TextColor3 = UI_TEXT_WHITE
    headerCategory.TextSize = 13
    headerCategory.Font = Enum.Font.GothamBlack
    headerCategory.TextXAlignment = Enum.TextXAlignment.Left
    headerCategory.Parent = TopBar

    local headerSub = Instance.new("TextLabel")
    headerSub.Position = UDim2.new(0, 14, 0, 28)
    headerSub.Size = UDim2.new(0, 220, 0, 12)
    headerSub.BackgroundTransparency = 1
    headerSub.Text = "Movement & Velocity Tweaks"
    headerSub.TextColor3 = UI_TEXT_DIM
    headerSub.TextSize = 9
    headerSub.Font = Enum.Font.GothamMedium
    headerSub.TextXAlignment = Enum.TextXAlignment.Left
    headerSub.Parent = TopBar

    local MinBtn = Instance.new("TextButton")
    MinBtn.ZIndex = 3
    MinBtn.Position = UDim2.new(1, -34, 0, 12)
    MinBtn.Size = UDim2.new(0, 24, 0, 24)
    MinBtn.BackgroundColor3 = UI_BTN_BG
    MinBtn.BorderSizePixel = 0
    MinBtn.Text = "—"
    MinBtn.TextColor3 = UI_TEXT_PRIMARY
    MinBtn.TextSize = 13
    MinBtn.Font = Enum.Font.GothamBold
    MinBtn.AutoButtonColor = false
    MinBtn.Parent = TopBar
    Instance.new("UICorner", MinBtn).CornerRadius = UDim.new(1, 0)
    local minSt = Instance.new("UIStroke", MinBtn)
    minSt.Color = Color3.fromRGB(65, 68, 80); minSt.Thickness = 1; minSt.Transparency = 0.35
    MinBtn.MouseButton1Down:Connect(function() TweenService:Create(MinBtn, UI_TWEEN_FAST, {BackgroundColor3 = UI_ACCENT, TextColor3 = Color3.fromRGB(0,0,0)}):Play() end)
    MinBtn.MouseButton1Up:Connect(function() TweenService:Create(MinBtn, UI_TWEEN_FAST, {BackgroundColor3 = UI_BTN_BG, TextColor3 = UI_TEXT_PRIMARY}):Play() end)

    local lockButton = Instance.new("TextButton")
    lockButton.Size = UDim2.new(0, 60, 0, 22)
    lockButton.Position = UDim2.new(1, -102, 0, 13)
    lockButton.BackgroundColor3 = UI_BTN_BG
    lockButton.BorderSizePixel = 0
    lockButton.Text = "UNLOCK"
    lockButton.TextColor3 = UI_TEXT_DIM
    lockButton.Font = Enum.Font.GothamBold
    lockButton.TextSize = 9
    lockButton.AutoButtonColor = false
    lockButton.ZIndex = 3
    lockButton.Parent = TopBar
    Instance.new("UICorner", lockButton).CornerRadius = UDim.new(0, 6)
    local lockSt = Instance.new("UIStroke", lockButton)
    lockSt.Color = Color3.fromRGB(65, 68, 80); lockSt.Thickness = 1; lockSt.Transparency = 0.35

    local locked = M.uiLocked == true
    lockButton.Text = locked and "LOCKED" or "UNLOCK"
    lockButton.TextColor3 = locked and UI_ACCENT or UI_TEXT_DIM
    lockButton.Activated:Connect(function()
        locked = not locked
        M.uiLocked = locked
        lockButton.Text = locked and "LOCKED" or "UNLOCK"
        lockButton.TextColor3 = locked and UI_ACCENT or UI_TEXT_DIM
        saveCherryConfig()
    end)

    local TopDiv = Instance.new("Frame")
    TopDiv.Position = UDim2.new(0, 10, 0, 46)
    TopDiv.Size = UDim2.new(1, -20, 0, 1)
    TopDiv.BackgroundColor3 = Color3.fromRGB(48, 50, 65)
    TopDiv.BackgroundTransparency = 0.5
    TopDiv.BorderSizePixel = 0
    TopDiv.Parent = MainArea

    -- PAGED CONTENT
    local PagedContent = Instance.new("Frame")
    PagedContent.Name = "PagedContent"
    PagedContent.Position = UDim2.new(0, 8, 0, 52)
    PagedContent.Size = UDim2.new(1, -16, 1, -58)
    PagedContent.BackgroundTransparency = 1
    PagedContent.Parent = MainArea

    -- PAGES
    local PM      = uiMakePage(PagedContent, "Page_SPEED",     1, true)
    local PMech   = uiMakePage(PagedContent, "Page_MECHANICS", 2, false)
    local PVis    = uiMakePage(PagedContent, "Page_VISUALS",   3, false)
    local PSounds = uiMakePage(PagedContent, "Page_SOUNDS",    4, false)
    local PUtil   = uiMakePage(PagedContent, "Page_UTILITY",   5, false)
    local PKB     = uiMakePage(PagedContent, "Page_KEYBINDS",  6, false)

    local Pages = {SPEED=PM, MECHANICS=PMech, VISUALS=PVis, SOUNDS=PSounds, UTILITY=PUtil, KEYBINDS=PKB}
    local Tabs  = {SPEED=TSpeed, MECHANICS=TMech, VISUALS=TVis, SOUNDS=TSounds, UTILITY=TUtil, KEYBINDS=TKB}
    local TabTitles = {
        SPEED     = {"SPEED CONFIGURATION", "Movement & Velocity Tweaks"},
        MECHANICS = {"COMBAT & MECHANICS", "Auto Bat, Anti-Die & Steal Engines"},
        VISUALS   = {"VISUALS & DISPLAY", "FOV & Theme Customization"},
        SOUNDS    = {"SOUNDS & AUDIO", "Hit Sounds, Music & Audio Triggers"},
        UTILITY   = {"UTILITY & SYSTEM", "Mobile Layout, Font & Configurations"},
        KEYBINDS  = {"HOTKEYS & BINDS", "Keyboard Shortcuts & Actions"},
    }
    local curTab = "SPEED"

    local function switchTab(name)
        if curTab == name then return end
        curTab = name
        for k, p in pairs(Pages) do p.Visible = (k == name) end
        for k, b in pairs(Tabs) do
            local act = (k == name)
            b:SetAttribute("IsActiveTab", act)
            local ind = b:FindFirstChild("Indicator")
            local lbl = b:FindFirstChild("TabLabel")
            local st  = b:FindFirstChild("TabStroke")
            
            TweenService:Create(b, UI_TWEEN_FAST, {
                BackgroundColor3 = act and (UI_ACCENT or Color3.fromRGB(255, 255, 255)) or Color3.fromRGB(22, 23, 30),
                BackgroundTransparency = act and 0.12 or 0.7
            }):Play()
            if ind then
                TweenService:Create(ind, UI_TWEEN_FAST, {BackgroundTransparency = act and 0 or 1}):Play()
            end
            if lbl then
                TweenService:Create(lbl, UI_TWEEN_FAST, {TextColor3 = act and Color3.fromRGB(0, 0, 0) or Color3.fromRGB(220, 220, 230)}):Play()
            end
            if st then
                st.Color = act and (UI_ACCENT or Color3.fromRGB(255, 255, 255)) or Color3.fromRGB(55, 58, 72)
                st.Transparency = act and 0.2 or 0.8
            end
        end
        if TabTitles[name] then
            headerCategory.Text = TabTitles[name][1]
            headerSub.Text = TabTitles[name][2]
        end
    end
    M.selectTab = switchTab

    TSpeed.MouseButton1Click:Connect(function() switchTab("SPEED") end)
    TMech.MouseButton1Click:Connect(function() switchTab("MECHANICS") end)
    TVis.MouseButton1Click:Connect(function() switchTab("VISUALS") end)
    TSounds.MouseButton1Click:Connect(function() switchTab("SOUNDS") end)
    TUtil.MouseButton1Click:Connect(function() switchTab("UTILITY") end)
    TKB.MouseButton1Click:Connect(function() switchTab("KEYBINDS") end)

    -- close / minimize
    local function closeUI()
        local tween = TweenService:Create(Frame, TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.In), {
            Size = UDim2.new(0, 560, 0, 0),
            BackgroundTransparency = 1
        })
        tween:Play()
        tween.Completed:Connect(function()
            Frame.Visible = false
            Frame.Size = UDim2.new(0, 560, 0, 420)
            Frame.AnchorPoint = Vector2.new(0.5, 0.5)
            Frame.Position = UDim2.new(0.5, 0, 0.5, 0)
            Frame.BackgroundTransparency = 0
        end)
    end
    local MinPill = Instance.new("Frame")
    MinPill.Visible = false
    MinPill.Active = true
    MinPill.ZIndex = 40
    MinPill.AnchorPoint = Vector2.new(0.5, 0)
    MinPill.Position = UDim2.new(0.5, 0, 0, 10)
    MinPill.Size = UDim2.new(0, 140, 0, 34)
    MinPill.BackgroundColor3 = Color3.fromRGB(12, 13, 18)
    MinPill.BackgroundTransparency = 0.08
    MinPill.BorderSizePixel = 0
    MinPill.Parent = gui
    Instance.new("UICorner", MinPill).CornerRadius = UDim.new(1, 0)
    do
        local pst = Instance.new("UIStroke")
        pst.Color = UI_ACCENT or Color3.fromRGB(255, 255, 255)
        pst.Thickness = 1.2
        pst.Transparency = 0.35
        pst.Parent = MinPill
    end
    do
        local l = Instance.new("TextLabel")
        l.Size = UDim2.new(1, 0, 1, 0)
        l.BackgroundTransparency = 1
        l.Text = "ONI HUB"
        l.TextColor3 = UI_ACCENT
        l.TextSize = 12
        l.Font = Enum.Font.GothamBlack
        l.Parent = MinPill

        local b = Instance.new("TextButton")
        b.ZIndex = 41
        b.Size = UDim2.new(1, 0, 1, 0)
        b.BackgroundTransparency = 1
        b.Text = ""
        b.AutoButtonColor = false
        b.Parent = MinPill
        b.MouseButton1Click:Connect(function()
            MinPill.Visible = false
            Frame.Visible = true
            M.menuOpen = true
            pcall(saveCherryConfig)
        end)
    end

    local function minimize()
        Frame.Visible = false
        MinPill.Visible = true
        M.menuOpen = false
        pcall(saveCherryConfig)
    end
    MinBtn.MouseButton1Click:Connect(minimize)

    -- DRAGGING
    do
        local function makeDrag(obj, target)
            local drag,dStart,sPos
            obj.InputBegan:Connect(function(i)
                if M.uiLocked then return end
                if i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch then
                    drag=true; dStart=i.Position; sPos=target.Position
                    i.Changed:Connect(function() if i.UserInputState==Enum.UserInputState.End then drag=false end end)
                end
            end)
            obj.InputChanged:Connect(function(i)
                if i.UserInputType==Enum.UserInputType.MouseMovement or i.UserInputType==Enum.UserInputType.Touch then
                    if drag then
                        local d=i.Position-dStart
                        target.Position=UDim2.new(sPos.X.Scale,sPos.X.Offset+d.X,sPos.Y.Scale,sPos.Y.Offset+d.Y)
                    end
                end
            end)
            UIS.InputChanged:Connect(function(i)
                if drag and M.uiLocked then drag=false end
            end)
        end
        makeDrag(BrandBox, Frame)
        makeDrag(TopBar, Frame)
        makeDrag(MinPill, MinPill)
    end

    -- KEYBIND CAPTURE
    M._anyKeyListening = false
    local activeKBBtn = nil
    M.keybindButtons = M.keybindButtons or {}
    local listeningTimeout = nil

    local function resetKeybindCapture()
        if activeKBBtn then
            for e,b in pairs(M.keybindButtons) do
                if b == activeKBBtn then
                    local parts = {}
                    if e.kb then table.insert(parts, e.kb.Name) end
                    if e.gp then table.insert(parts, e.gp.Name) end
                    b.Text = (#parts > 0) and table.concat(parts, " / ") or "..."
                    b.TextColor3 = UI_TEXT_DIM
                    break
                end
            end
            activeKBBtn = nil
            M._anyKeyListening = false
            if listeningTimeout then task.cancel(listeningTimeout); listeningTimeout = nil end
        end
    end

    local function formatKeybindText(entry)
        if not entry then return "..." end
        local parts = {}
        if entry.kb then table.insert(parts, entry.kb.Name) end
        if entry.gp then table.insert(parts, entry.gp.Name) end
        if #parts == 0 then return "..." end
        return table.concat(parts, " / ")
    end

    local function isGamepadInputType(uit)
        return uit == Enum.UserInputType.Gamepad1
            or uit == Enum.UserInputType.Gamepad2
            or uit == Enum.UserInputType.Gamepad3
            or uit == Enum.UserInputType.Gamepad4
            or uit == Enum.UserInputType.Gamepad5
            or uit == Enum.UserInputType.Gamepad6
            or uit == Enum.UserInputType.Gamepad7
            or uit == Enum.UserInputType.Gamepad8
    end

    local function uiKeybindRow(parent, label, kbEntry)
        local r = Instance.new("Frame"); r.ClipsDescendants = true; r.Size = UDim2.new(1,0,0,44)
        r.BackgroundColor3 = UI_ROW_BG; r.BackgroundTransparency = 0.1; r.BorderSizePixel = 0; r.Parent = parent; uiCardStyle(r)
        local l = Instance.new("TextLabel"); l.Position=UDim2.new(0,13,0,0); l.Size=UDim2.new(0.42,0,0,44); l.BackgroundTransparency=1
        l.Text=label; l.TextColor3=UI_TEXT_PRIMARY; l.TextSize=13; l.Font=Enum.Font.GothamMedium; l.TextXAlignment=Enum.TextXAlignment.Left; l.Parent=r
        local btn = uiSmallBtn({Parent=r, Pos=UDim2.new(1,-150,0.5,-12), Size=UDim2.new(0,140,0,25),
            Text=formatKeybindText(kbEntry),
            Col=UI_TEXT_DIM, TS=10, CR=6})
        M.keybindButtons[kbEntry] = btn

        btn.MouseButton1Click:Connect(function()
            if activeKBBtn and activeKBBtn ~= btn then resetKeybindCapture() end
            activeKBBtn = btn
            btn.Text = "Press key / button..."
            btn.TextColor3 = Color3.fromRGB(150,150,150)
            M._anyKeyListening = true
            if listeningTimeout then task.cancel(listeningTimeout) end
            listeningTimeout = task.delay(8, resetKeybindCapture)
        end)
        return r
    end

    local function kbMatch(entry, keycode)
        if not entry or not keycode or keycode == Enum.KeyCode.Unknown then return false end
        if entry.kb and entry.kb == keycode then return true end
        if entry.gp and entry.gp == keycode then return true end
        return false
    end

    M._keybindCaptureConn = UIS.InputBegan:Connect(function(input, gameProcessed)
        -- Always capture while rebinding (even if gameProcessed)
        if M._anyKeyListening then
            if activeKBBtn then
                local kc = input.KeyCode
                if kc == Enum.KeyCode.Escape then
                    resetKeybindCapture(); pcall(saveCherryConfig); return
                end
                local uit = input.UserInputType
                if uit == Enum.UserInputType.Keyboard and kc ~= Enum.KeyCode.Unknown then
                    for e,b in pairs(M.keybindButtons) do
                        if b == activeKBBtn then
                            e.kb = kc
                            -- keep existing gamepad bind so both PC + controller work
                            b.Text = formatKeybindText(e)
                            b.TextColor3 = UI_TEXT_DIM
                            activeKBBtn = nil; M._anyKeyListening = false
                            if listeningTimeout then task.cancel(listeningTimeout); listeningTimeout = nil end
                            pcall(saveCherryConfig)
                            break
                        end
                    end
                elseif isGamepadInputType(uit) and kc ~= Enum.KeyCode.Unknown then
                    for e,b in pairs(M.keybindButtons) do
                        if b == activeKBBtn then
                            e.gp = kc
                            -- keep existing keyboard bind
                            b.Text = formatKeybindText(e)
                            b.TextColor3 = UI_TEXT_DIM
                            activeKBBtn = nil; M._anyKeyListening = false
                            if listeningTimeout then task.cancel(listeningTimeout); listeningTimeout = nil end
                            pcall(saveCherryConfig)
                            break
                        end
                    end
                end
            end
            return
        end

        -- Process configured keybinds even when a GUI marks the input as processed.
        -- This keeps menu/Auto Grab UI focus from swallowing the shortcuts.
        if input.UserInputType ~= Enum.UserInputType.Keyboard and not isGamepadInputType(input.UserInputType) then return end
        local kc = input.KeyCode
        if kc == Enum.KeyCode.Unknown then return end

        if kbMatch(M.KB.LaggerToggle, kc) then
            local now = tick()
            if not M._lastLaggerBindPress or now - M._lastLaggerBindPress > 0.15 then
                M._lastLaggerBindPress = now
                M.cycleLaggerModeBind()
            end
            return
        end
        if kbMatch(M.KB.SpeedToggle, kc) then M.toggleCarryMode(); saveCherryConfig() end
        if kbMatch(M.KB.DropBrainrot, kc) then M.runDrop() end
        if kbMatch(M.KB.TPFloor, kc) then M.runTPFloor() end
        if kbMatch(M.KB.InstaReset, kc) then M.cursedInstaReset() end
        if kbMatch(M.KB.AutoLeft, kc) then
            M.autoLeftEnabled = not M.autoLeftEnabled
            if M.autoLeftEnabled then
                if M.autoRightEnabled then M.autoRightEnabled = false; M.stopAutoRight() end
                if M.autoBatEnabled then M.stopBatAimbot() end
                M.startAutoLeft()
            else M.stopAutoLeft() end
            if M.autoLeftSetVisual then M.autoLeftSetVisual(M.autoLeftEnabled) end
            if M.mobBtnRefs.autoLeft then M.mobBtnRefs.autoLeft(M.autoLeftEnabled) end
            saveCherryConfig()
        end
        if kbMatch(M.KB.AutoRight, kc) then
            M.autoRightEnabled = not M.autoRightEnabled
            if M.autoRightEnabled then
                if M.autoLeftEnabled then M.autoLeftEnabled = false; M.stopAutoLeft() end
                if M.autoBatEnabled then M.stopBatAimbot() end
                M.startAutoRight()
            else M.stopAutoRight() end
            if M.autoRightSetVisual then M.autoRightSetVisual(M.autoRightEnabled) end
            if M.mobBtnRefs.autoRight then M.mobBtnRefs.autoRight(M.autoRightEnabled) end
            saveCherryConfig()
        end
        if kbMatch(M.KB.AutoBat, kc) then
            if not M.autoBatEnabled then
                if M.autoLeftEnabled then M.autoLeftEnabled = false; M.stopAutoLeft() end
                if M.autoRightEnabled then M.autoRightEnabled = false; M.stopAutoRight() end
                M.queueAutoBatStart()
            else M.stopBatAimbot() end
            if M.autoBatSetVisual then M.autoBatSetVisual(M.autoBatEnabled) end
            if M.mobBtnRefs.autoBat then M.mobBtnRefs.autoBat(M.autoBatEnabled) end
            saveCherryConfig()
        end
        if kbMatch(M.KB.BypassAimbot, kc) then
            M.toggleBypassAimbot()
            if M.setBypassVisual then M.setBypassVisual(M.bypassAimbotEnabled) end
            if M.mobBtnRefs.bypass then M.mobBtnRefs.bypass(M.bypassAimbotEnabled) end
            saveCherryConfig()
        end
        if kbMatch(M.KB.GuiHide, kc) then
            if Frame then
                Frame.Visible = not Frame.Visible
                MinPill.Visible = not Frame.Visible
                M.menuOpen = Frame.Visible == true
                pcall(saveCherryConfig)
            end
        end
    end)

    -- ============================================================
    -- PAGINE
    -- ============================================================

    -- PAGE: SPEED
    uiSectionHeader(PM, "SPEEDS")
    local _, nsBox = uiNumberRow(PM, "Normal Speed", M.NS, 1, 500, function(v) M.NS = v end)
    local _, csBox = uiNumberRow(PM, "Carry Speed", M.CS, 1, 500, function(v) M.CS = v end)
    M.normalBox = nsBox; M.carryBox = csBox

    do
        local r = Instance.new("Frame"); r.ClipsDescendants=true; r.Size=UDim2.new(1,0,0,46)
        r.BackgroundColor3=UI_ROW_BG; r.BackgroundTransparency=0.03; r.BorderSizePixel=0; r.Parent=PM; uiCardStyle(r)
        local l = Instance.new("TextLabel"); l.Position=UDim2.new(0,14,0,0); l.Size=UDim2.new(1,-74,1,0)
        l.BackgroundTransparency=1; l.Text="Carry Mode"; l.TextColor3=UI_TEXT_PRIMARY; l.TextSize=14; l.Font=Enum.Font.GothamMedium; l.TextXAlignment=Enum.TextXAlignment.Left; l.Parent=r
        local carryBtn = uiSmallBtn({Parent=r, Pos=UDim2.new(1,-100,0.5,-13), Size=UDim2.new(0,88,0,26),
            Text=M.carrySpeedActive and "Carry On" or "Carry Off", Col=UI_TEXT_PRIMARY, TS=12, CR=6, SC=UI_ACCENT, STr=0.3})
        carryBtn.MouseButton1Click:Connect(function()
            M.carrySpeedActive = not M.carrySpeedActive
            carryBtn.Text = M.carrySpeedActive and "Carry On" or "Carry Off"
            M.refreshSpeedModeLabel()
            if M.mobBtnRefs.carrySpeed then M.mobBtnRefs.carrySpeed(M.carrySpeedActive) end
            if M.mobBtnRefs.laggerCarry then M.mobBtnRefs.laggerCarry(M.laggerCarryActive) end
            if M.carryModeBtn then M.carryModeBtn.Text = M.carrySpeedActive and "Carry On" or "Carry Off" end
            if M.laggerCarryBtn then M.laggerCarryBtn.Text = M.laggerCarryActive and "L.Carry On" or "L.Carry Off" end
            saveCherryConfig()
        end)
        M.carryModeBtn = carryBtn
    end

    local _, setAutoCarry = uiToggleRow(PM, "Auto Carry Speed", M.autoSwitchSpeedEnabled, function(on)
        M.autoSwitchSpeedEnabled = on
        M._autoSwitchWasSteal = nil
        if not on then
            if M.carryModeBtn then
                M.carryModeBtn.Text = M.carrySpeedActive and "Carry On" or "Carry Off"
            end
            if M.mobBtnRefs.carrySpeed then M.mobBtnRefs.carrySpeed(M.carrySpeedActive) end
        end
        M.refreshWalkSpeedAutoSwitch()
        saveCherryConfig()
    end)
    M.setAutoCarryVisual = setAutoCarry

    local _, setAutoTurnOff = uiToggleRow(PM, "Auto Turn Off Speed", M.autoTurnOffSpeedEnabled, function(on)
        M.autoTurnOffSpeedEnabled = on
        M.refreshWalkSpeedAutoSwitch()
        saveCherryConfig()
    end)
    M.setAutoTurnOffVisual = setAutoTurnOff

    local _, setAutoLagSwitch = uiToggleRow(PM, "Auto Switch Lagger Speed", M.autoSwitchLaggerSpeedEnabled, function(on)
        M.autoSwitchLaggerSpeedEnabled = on
        M.refreshWalkSpeedAutoSwitch()
        saveCherryConfig()
    end)
    M.setAutoSwitchLaggerVisual = setAutoLagSwitch

    uiSectionHeader(PM, "LAGGER")
    local _, lsBox = uiNumberRow(PM, "Lagger Normal", M.LAGGER_SPEED, 1, 500, function(v) M.LAGGER_SPEED = v end)
    local _, lcBox = uiNumberRow(PM, "Lagger Carry", math.min(M.LAGGER_CARRY_SPEED,23), 1, 23, function(v) M.LAGGER_CARRY_SPEED = math.min(v,23) end)

    do
        local r = Instance.new("Frame"); r.ClipsDescendants=true; r.Size=UDim2.new(1,0,0,46)
        r.BackgroundColor3=UI_ROW_BG; r.BackgroundTransparency=0.03; r.BorderSizePixel=0; r.Parent=PM; uiCardStyle(r)
        local l = Instance.new("TextLabel"); l.Position=UDim2.new(0,14,0,0); l.Size=UDim2.new(1,-74,1,0)
        l.BackgroundTransparency=1; l.Text="Lagger Mode"; l.TextColor3=UI_TEXT_PRIMARY; l.TextSize=14; l.Font=Enum.Font.GothamMedium; l.TextXAlignment=Enum.TextXAlignment.Left; l.Parent=r
        local modeBtn = uiSmallBtn({Parent=r, Pos=UDim2.new(1,-100,0.5,-13), Size=UDim2.new(0,88,0,26),
            Text=M.laggerModeEnabled and "Lag On" or "Lag Off", Col=UI_TEXT_PRIMARY, TS=12, CR=6, SC=UI_ACCENT, STr=0.3})
        modeBtn.MouseButton1Click:Connect(function()
            M.toggleLaggerMode()
            modeBtn.Text = M.laggerModeEnabled and "Lag On" or "Lag Off"
        end)
        M.laggerModeBtn = modeBtn
    end

    do
        local r = Instance.new("Frame"); r.ClipsDescendants=true; r.Size=UDim2.new(1,0,0,46)
        r.BackgroundColor3=UI_ROW_BG; r.BackgroundTransparency=0.03; r.BorderSizePixel=0; r.Parent=PM; uiCardStyle(r)
        local l = Instance.new("TextLabel"); l.Position=UDim2.new(0,14,0,0); l.Size=UDim2.new(1,-74,1,0)
        l.BackgroundTransparency=1; l.Text="Lagger Carry Mode"; l.TextColor3=UI_TEXT_PRIMARY; l.TextSize=14; l.Font=Enum.Font.GothamMedium; l.TextXAlignment=Enum.TextXAlignment.Left; l.Parent=r
        local modeBtn = uiSmallBtn({Parent=r, Pos=UDim2.new(1,-100,0.5,-13), Size=UDim2.new(0,88,0,26),
            Text=M.laggerCarryActive and "L.Carry On" or "L.Carry Off", Col=UI_TEXT_PRIMARY, TS=12, CR=6, SC=UI_ACCENT, STr=0.3})
        modeBtn.MouseButton1Click:Connect(function()
            M.toggleLaggerCarry()
            modeBtn.Text = M.laggerCarryActive and "L.Carry On" or "L.Carry Off"
        end)
        M.laggerCarryBtn = modeBtn
    end

    -- PAGE: MECHANICS (contenuto completo con scroll funzionante)
    uiSectionHeader(PMech, "COMBAT")
    local batModeIdx = (M.autoBatMode == "BAT V2") and 2 or 1
    local _, setBatAimbot, setBatModeUI = uiExpandToggleRow(
        PMech, "Bat Aimbot", M.autoBatEnabled, {"Normal", "BAT V2"}, batModeIdx,
        function(on)
            if on then M.queueAutoBatStart() else M.stopBatAimbot(); if M.stopBatV2Aimbot then M.stopBatV2Aimbot() end end
        end,
        function(newMode)
            M.autoBatMode = (newMode == "BAT V2") and "BAT V2" or "Normal"
            if M.autoBatEnabled then
                M.stopBatAimbot()
                if M.stopBatV2Aimbot then M.stopBatV2Aimbot() end
                M.queueAutoBatStart()
            end
            saveCherryConfig()
        end
    )
    M.autoBatSetVisual = setBatAimbot
    M.setAutoBatModeUI = setBatModeUI

    local _, setBatCounter = uiToggleRow(PMech, "Bat Counter", M.batCounterEnabled, function(on)
        M.batCounterEnabled = on
        if on then M.startBatCounter() else M.stopBatCounter() end
    end)
    M.setBatCounterVisual = setBatCounter

    local tpBatModeIdx = (M.tpBatHitMode == "Normal") and 2 or 1
    local _, setBypassVis, setTpBatModeUI = uiExpandToggleRow(
        PMech, "TP Bat", M.bypassAimbotEnabled, {"Sure Hit", "Normal Hit"}, tpBatModeIdx,
        function(on)
            M.bypassAimbotEnabled=on
            if on then M.startBypassAimbot() else M.stopBypassAimbot() end
        end,
        function(newMode)
            local wasOn=M.bypassAimbotEnabled
            M.tpBatHitMode=(newMode=="Normal Hit" or newMode=="Normal") and "Normal" or "Sure"
            if wasOn then M.stopBypassAimbot(); M.startBypassAimbot() end
        end
    )
    M.setBypassVisual=setBypassVis
    M.setTpBatModeUI=setTpBatModeUI
    uiSectionHeader(PMech, "STEAL")
    local _, setMedusaCount = uiToggleRow(PMech, "Medusa Count", M.medusaCounterEnabled, function(on)
        M.medusaCounterEnabled = on
        if on then M.setupMedusa(player.Character) else M.stopMedusaCounter() end
    end)
    M.setMedusaVisual = setMedusaCount

    local stealModeLabels = {"V1", "V2", "V3", "V4"}
    local function stealLabelToMode(lab)
        if lab == "V4" then return "V4" end
        if lab == "V3" then return "V3" end
        if lab == "V2" then return "V2" end
        return "V1"
    end
    local function stealModeToLabel(mode)
        if mode == "V4" or mode == "Semi" then return "V4" end
        if mode == "V3" then return "V3" end
        if mode == "V2" then return "V2" end
        return "V1"
    end
    local stealDefaultIdx = 1
    do
        local lab = stealModeToLabel(M.stealMode)
        for i, v in ipairs(stealModeLabels) do if v == lab then stealDefaultIdx = i break end end
    end

    local _, setAutoSteal, setStealModeUI, regStealSettings = uiExpandToggleRow(
        PMech,
        "Auto Steal",
        M.Steal.AutoStealEnabled,
        stealModeLabels,
        stealDefaultIdx,
        function(on)
            M.Steal.AutoStealEnabled = on
            if on then M.startAutoSteal() else M.stopAutoSteal() end
        end,
        function(newLabel)
            local oldMode = M.stealMode
            M.stealMode = stealLabelToMode(newLabel)
            if M.updateStatusModeBadge then pcall(M.updateStatusModeBadge) end
            if oldMode ~= M.stealMode and M.Steal.AutoStealEnabled then
                M.stopAutoSteal(); M.startAutoSteal()
            end
            M.updateStatusRadius()
        end
    )
    M.setInstaGrab = setAutoSteal
    M.setAutoGrabModeUI = setStealModeUI

    -- V2 settings
    local v2Box = Instance.new("Frame"); v2Box.BackgroundTransparency=1; v2Box.Size=UDim2.new(1,0,0,0); v2Box.AutomaticSize=Enum.AutomaticSize.Y
    local v2Lay = Instance.new("UIListLayout"); v2Lay.Padding=UDim.new(0,6); v2Lay.Parent=v2Box
    local _, srBox = uiNumberRow(v2Box, "Grab Radius", M.Steal.StealRadius, 0.5, 300, function(v)
        M.Steal.StealRadius = v; M.setStealRadius(v); M.updateStatusRadius()
    end)
    M.radInput = srBox
    local _, sdBox = uiNumberRow(v2Box, "Hold Duration", M.Steal.StealDuration, 0.1, 10, function(v)
        M.Steal.StealDuration = v
    end)
    M.durationBox = sdBox
    local _, setAutoRadius = uiToggleRow(v2Box, "Auto Radius", M.autoRadiusEnabled, function(on)
        M.autoRadiusEnabled = on; M.updateStatusRadius()
    end)
    M.setAutoRadiusVisual = setAutoRadius
    regStealSettings("V2", v2Box)

    -- Semi settings
    local semiBox = Instance.new("Frame"); semiBox.BackgroundTransparency=1; semiBox.Size=UDim2.new(1,0,0,0); semiBox.AutomaticSize=Enum.AutomaticSize.Y
    local semiLay = Instance.new("UIListLayout"); semiLay.Padding=UDim.new(0,6); semiLay.Parent=semiBox
    local _, semiRadBox = uiNumberRow(semiBox, "Semi Radius (max 10)", math.min(M.Semi.radius,10), 0.5, 10, function(v)
        M.Semi.radius = math.min(v,10)
        if semiRadBox then semiRadBox.Text = tostring(M.Semi.radius) end
    end)
    M.semiRadInput = semiRadBox
    local _, semiHoldMin = uiNumberRow(semiBox, "Hold Min", M.Semi.holdMin or 1.3, 0.1, 5, function(v) M.Semi.holdMin = v end)
    local _, semiHoldMax = uiNumberRow(semiBox, "Hold Max", M.Semi.holdMax or 2.6, 0.1, 8, function(v) M.Semi.holdMax = v end)
    regStealSettings("Semi", semiBox)


    local _, sbBox = uiNumberRow(PMech, "Steal Bar Size", M.stealBarSize, 100, 800, function(v)
        M.stealBarSize = v; M.buildStatusUI()
    end)
    uiActionRow(PMech, "AUTO GRAB GUI -", function()
        M.adjustAutoGrabGuiScale(-0.1)
    end)
    uiActionRow(PMech, "AUTO GRAB GUI +", function()
        M.adjustAutoGrabGuiScale(0.1)
    end)


    uiSectionHeader(PMech, "MOTION")
    local jumpDefaultIdx = (M.infJumpMode == "hold") and 2 or 1
    local _, setInfJump, setJumpModeUI = uiExpandToggleRow(
        PMech,
        "Infinite Jump",
        M.infJumpEnabled,
        {"Manual", "Hold"},
        jumpDefaultIdx,
        function(on)
            M.infJumpEnabled = on
            if on and M.infJumpMode == "manual" then M.startManualInfJumpLoop()
            elseif on and M.infJumpMode == "hold" then M.startHoldInfJump()
            else M.stopManualInfJumpLoop(); M.stopHoldInfJump() end
        end,
        function(newMode)
            local wasOn = M.infJumpEnabled
            M.infJumpMode = (newMode == "Hold") and "hold" or "manual"
            if wasOn then
                M.stopManualInfJumpLoop(); M.stopHoldInfJump()
                if M.infJumpMode == "manual" then M.startManualInfJumpLoop()
                else M.startHoldInfJump() end
            end
        end
    )
    M.setInfJumpVisual = setInfJump
    M.setJumpModeUI = setJumpModeUI


    local _, setMirrorTP = uiToggleRow(PMech, "Mirror TP Down", M.mirrorTPDownEnabled, function(on)
        M.setMirrorTPDown(on)
        saveCherryConfig()
    end)
    M.setMirrorTPVisual = setMirrorTP

    local _, setAntiRag = uiToggleRow(PMech, "Anti Ragdoll", M.antiRagdollEnabled, function(on)
        M.antiRagdollEnabled = on
        if on then M.startAntiRagdoll() else M.stopAntiRagdoll() end
        saveCherryConfig()
    end)
    M.setAntiRagVisual = setAntiRag

    local antiRagModeIdx = (M.antiRagdollMode == "No Splatter") and 2 or 1
    local _, setAntiRagModeUI = uiChoiceRow(PMech, "Anti Ragdoll Mode", {"Splatter", "No Splatter"}, antiRagModeIdx, function(v)
        M.antiRagdollMode = (v == "No Splatter") and "No Splatter" or "Splatter"
        saveCherryConfig()
    end)
    M.setAntiRagModeUI = setAntiRagModeUI

    local autoPlayModeIdx = (M.autoPlayMode == "Full") and 2 or 1
    local _, setAutoPlayModeUI = uiChoiceRow(PMech, "Auto Play Mode", {"Semi", "Full"}, autoPlayModeIdx, function(v)
        M.autoPlayMode = (v == "Full") and "Full" or "Semi"
        if M.autoLeftEnabled then M.stopAutoLeft(); M.startAutoLeft() end
        if M.autoRightEnabled then M.stopAutoRight(); M.startAutoRight() end
        saveCherryConfig()
    end)
    M.setAutoPlayModeUI = setAutoPlayModeUI

    local _, setAL = uiToggleRow(PMech, "Auto Left", M.autoLeftEnabled, function(on)
        if on then
            if M.autoRightEnabled then M.autoRightEnabled=false; M.stopAutoRight(); if M.autoRightSetVisual then M.autoRightSetVisual(false) end end
            if M.autoBatEnabled then M.stopBatAimbot(); if M.autoBatSetVisual then M.autoBatSetVisual(false) end end
            M.autoLeftEnabled=true; M.startAutoLeft()
        else M.autoLeftEnabled=false; M.stopAutoLeft() end
        if M.mobBtnRefs.autoLeft then M.mobBtnRefs.autoLeft(on) end
    end)
    M.autoLeftSetVisual = setAL

    local _, setAR = uiToggleRow(PMech, "Auto Right", M.autoRightEnabled, function(on)
        if on then
            if M.autoLeftEnabled then M.autoLeftEnabled=false; M.stopAutoLeft(); if M.autoLeftSetVisual then M.autoLeftSetVisual(false) end end
            if M.autoBatEnabled then M.stopBatAimbot(); if M.autoBatSetVisual then M.autoBatSetVisual(false) end end
            M.autoRightEnabled=true; M.startAutoRight()
        else M.autoRightEnabled=false; M.stopAutoRight() end
        if M.mobBtnRefs.autoRight then M.mobBtnRefs.autoRight(on) end
    end)
    M.autoRightSetVisual = setAR

    local _, setATP = uiToggleRow(PMech, "Auto TP Down", M.autoTPEnabled, function(on)
        M.autoTPEnabled = on
        if on then M.startAutoTP() else M.stopAutoTP() end
    end)
    M.setAutoTPVisual = setATP

    local _, tpHBox = uiNumberRow(PMech, "TP Height", M.autoTPHeight, 1, 100, function(v) M.autoTPHeight = v end)
    M.autoTPHeightBox = tpHBox

    -- PAGE: VISUALS
    uiSectionHeader(PVis, "SKY & VISION")
    do
        local r = Instance.new("Frame"); r.ClipsDescendants=true; r.Size=UDim2.new(1,0,0,46)
        r.BackgroundColor3=UI_ROW_BG; r.BackgroundTransparency=0.03; r.BorderSizePixel=0; r.Parent=PVis; uiCardStyle(r)
        local l = Instance.new("TextLabel"); l.Position=UDim2.new(0,13,0,0); l.Size=UDim2.new(0.55,0,1,0)
        l.BackgroundTransparency=1; l.Text="Sky Theme"; l.TextColor3=UI_TEXT_PRIMARY; l.TextSize=13; l.Font=Enum.Font.GothamMedium; l.TextXAlignment=Enum.TextXAlignment.Left; l.Parent=r
        local skyLbl = Instance.new("TextLabel"); skyLbl.Position=UDim2.new(0.55,0,0,0); skyLbl.Size=UDim2.new(0.45,-10,1,0)
        skyLbl.BackgroundTransparency=1; skyLbl.Text=M.currentSkyTheme; skyLbl.TextColor3=UI_ACCENT; skyLbl.Font=Enum.Font.GothamBold; skyLbl.TextSize=12; skyLbl.TextXAlignment=Enum.TextXAlignment.Right; skyLbl.Parent=r
        local skyIdx = 1
        for i,t in ipairs(M.SkyOrder) do if t == M.currentSkyTheme then skyIdx = i; break end end
        local btn = Instance.new("TextButton",r); btn.Size=UDim2.new(1,0,1,0); btn.BackgroundTransparency=1; btn.Text=""
        btn.Activated:Connect(function()
            skyIdx = skyIdx % #M.SkyOrder + 1
            local t = M.SkyOrder[skyIdx]
            skyLbl.Text = t; M.currentSkyTheme = t; M.CandyApplyCustomSky(t); saveCherryConfig()
        end)
    end
    do
        local r = Instance.new("Frame"); r.ClipsDescendants=true; r.Size=UDim2.new(1,0,0,46)
        r.BackgroundColor3=UI_ROW_BG; r.BackgroundTransparency=0.03; r.BorderSizePixel=0; r.Parent=PVis; uiCardStyle(r)
        local l = Instance.new("TextLabel"); l.Position=UDim2.new(0,13,0,0); l.Size=UDim2.new(0.55,0,1,0)
        l.BackgroundTransparency=1; l.Text="FOV"; l.TextColor3=UI_TEXT_PRIMARY; l.TextSize=13; l.Font=Enum.Font.GothamMedium; l.TextXAlignment=Enum.TextXAlignment.Left; l.Parent=r
        local fovLbl = Instance.new("TextLabel"); fovLbl.Position=UDim2.new(0.55,0,0,0); fovLbl.Size=UDim2.new(0.45,-10,1,0)
        fovLbl.BackgroundTransparency=1; fovLbl.Text=tostring(M.fovValue); fovLbl.TextColor3=UI_ACCENT; fovLbl.Font=Enum.Font.GothamBold; fovLbl.TextSize=12; fovLbl.TextXAlignment=Enum.TextXAlignment.Right; fovLbl.Parent=r
        local fovIdx = 1
        local btn = Instance.new("TextButton",r); btn.Size=UDim2.new(1,0,1,0); btn.BackgroundTransparency=1; btn.Text=""
        btn.Activated:Connect(function()
            fovIdx = fovIdx % #M.fovOptions + 1
            M.fovValue = M.fovOptions[fovIdx]; fovLbl.Text = tostring(M.fovValue); M.applyFOV(); saveCherryConfig()
        end)
    end

    uiSectionHeader(PVis, "COLOUR SCHEME")
    do
        local themeNames = {}
        for name in pairs(CHERRY_THEMES) do table.insert(themeNames, name) end
        table.sort(themeNames)
        local cur = CherryConfig.Theme or "Default"
        local idx = 1
        for i,n in ipairs(themeNames) do if n == cur then idx = i break end end

        local r = Instance.new("Frame"); r.ClipsDescendants=true; r.Size=UDim2.new(1,0,0,46)
        r.BackgroundColor3=UI_ROW_BG; r.BackgroundTransparency=0.03; r.BorderSizePixel=0; r.Parent=PVis; uiCardStyle(r)
        local l = Instance.new("TextLabel"); l.Position=UDim2.new(0,13,0,0); l.Size=UDim2.new(0.4,0,1,0)
        l.BackgroundTransparency=1; l.Text="Theme"; l.TextColor3=UI_TEXT_PRIMARY; l.TextSize=13
        l.Font=Enum.Font.GothamMedium; l.TextXAlignment=Enum.TextXAlignment.Left; l.Parent=r
        local themeLbl = Instance.new("TextLabel"); themeLbl.Position=UDim2.new(0.4,0,0,0); themeLbl.Size=UDim2.new(0.6,-10,1,0)
        themeLbl.BackgroundTransparency=1; themeLbl.Text=cur; themeLbl.TextColor3=UI_ACCENT
        themeLbl.Font=Enum.Font.GothamBold; themeLbl.TextSize=12; themeLbl.TextXAlignment=Enum.TextXAlignment.Right; themeLbl.Parent=r

        -- color swatches
        local sw = Instance.new("Frame"); sw.Size=UDim2.new(1,0,0,36); sw.BackgroundTransparency=1; sw.Parent=PVis
        local swLay = Instance.new("UIListLayout"); swLay.FillDirection=Enum.FillDirection.Horizontal
        swLay.Padding=UDim.new(0,6); swLay.VerticalAlignment=Enum.VerticalAlignment.Center; swLay.Parent=sw
        local function applyTheme(name)
            local t = CHERRY_THEMES[name]; if not t then return end
            CherryConfig.Theme = name
            M.colorScheme = name
            M._savedTheme = name
            applyAccentFromTheme()
            themeLbl.Text = name
            themeLbl.TextColor3 = t.Accent
            if M.mainFrame then
                M.mainFrame.BackgroundColor3 = UI_BG_DARK
                local st = M.mainFrame:FindFirstChild("MainStroke")
                if st then st.Color = t.Accent end
                local gr = M.mainFrame:FindFirstChild("MainGradient")
                if gr then
                    gr.Color = ColorSequence.new({
                        ColorSequenceKeypoint.new(0, UI_GRAD_TOP),
                        ColorSequenceKeypoint.new(0.45, UI_BG_DARK),
                        ColorSequenceKeypoint.new(1, UI_GRAD_BOT),
                    })
                end
            end
            -- Recolor any remaining pure-black parts immediately
            pcall(function()
                if M.mainFrame then M.recolorBlacksToTheme(M.mainFrame) end
                if M.mobGuiRef then M.recolorBlacksToTheme(M.mobGuiRef) end
                if M.statusGui then M.recolorBlacksToTheme(M.statusGui) end
            end)
            M.applyStealBarTheme(t.Accent)
            M.updateHeadTheme()
            saveCherryConfig()
            task.defer(function()
                local wasOpen = M.menuOpen ~= false
                applyAccentFromTheme()
                M.menuOpen = wasOpen
                M.buildGui()
                -- buildGui restores menuOpen from M.menuOpen
                pcall(function()
                    if M.mainFrame then M.recolorBlacksToTheme(M.mainFrame) end
                    if M.mobGuiRef then M.recolorBlacksToTheme(M.mobGuiRef) end
                    if M.statusGui then M.recolorBlacksToTheme(M.statusGui) end
                end)
                pcall(function() M.applyStealBarTheme(UI_ACCENT) end)
                pcall(function() M.updateHeadTheme() end)
                if M.mobileButtonsEnabled then
                    pcall(function() M.buildMobileButtons() end)
                end
                saveCherryConfig()
            end)
        end
        for _, name in ipairs(themeNames) do
            local t = CHERRY_THEMES[name]
            local b = Instance.new("TextButton")
            b.Size = UDim2.new(0, 28, 0, 16)
            b.BackgroundColor3 = t.Accent
            b.Text = ""
            b.AutoButtonColor = false
            b.Parent = sw
            Instance.new("UICorner", b).CornerRadius = UDim.new(0, 4)
            local st = Instance.new("UIStroke"); st.Color = Color3.fromRGB(255,255,255); st.Transparency = 0.4; st.Parent = b
            b.MouseButton1Click:Connect(function() applyTheme(name) end)
        end
        local btn = Instance.new("TextButton", r); btn.Size=UDim2.new(1,0,1,0); btn.BackgroundTransparency=1; btn.Text=""
        btn.Activated:Connect(function()
            idx = idx % #themeNames + 1
            applyTheme(themeNames[idx])
        end)
    end

    uiSectionHeader(PVis, "BACKGROUND")
    uiActionRow(PVis, "Custom Background", function()
        M.openImagePicker("bg")
    end)
    uiActionRow(PVis, "Mobile Button Images", function()
        M.openImagePicker("mob")
    end)

    uiSectionHeader(PVis, "✦ AURA ✦")
    local _, setWingsAura = uiToggleRow(PVis, "✦ Wings Aura ✦", M.wingsAuraEnabled, function(on)
        M.wingsAuraEnabled = on
        if on then
            local char = player.Character
            if char then M.applyWingsAura(char) end
        else
            M.removeWingsAura(true)
        end
        saveCherryConfig()
    end)
    M.setWingsAuraVisual = setWingsAura

    -- PAGE: SOUNDS
    uiSectionHeader(PSounds, "HIT SOUNDS")
    local _, setHitSound = uiToggleRow(PSounds, "Hit Sound", M.hitSoundEnabled, function(on)
        M.hitSoundEnabled = on
        saveCherryConfig()
    end)
    M.setHitSoundVisual = setHitSound

    local hitSoundIdx = 1
    for i, n in ipairs(M.HIT_SOUND_NAMES) do
        if n == (M.hitSoundChoice or "Minecraft Critical Hit") then hitSoundIdx = i break end
    end
    local _, setHitSoundChoice = uiChoiceRow(PSounds, "Hit Sound Type", M.HIT_SOUND_NAMES, hitSoundIdx, function(v)
        M.hitSoundChoice = v
        saveCherryConfig()
    end)
    M.setHitSoundChoiceUI = setHitSoundChoice

    local _, customSoundBox = uiInputRow(PSounds, "Custom Sound ID", M.hitSoundCustomId or "")
    customSoundBox.PlaceholderText = "Asset ID..."
    customSoundBox.FocusLost:Connect(function()
        M.hitSoundCustomId = customSoundBox.Text
        saveCherryConfig()
    end)
    M.hitSoundCustomBox = customSoundBox

    local _, hitVolBox = uiNumberRow(PSounds, "Hit Volume", M.hitSoundVolume or 1.0, 0.1, 10, function(v)
        M.hitSoundVolume = v
    end)
    M.hitSoundVolBox = hitVolBox

    local _, hitPitchBox = uiNumberRow(PSounds, "Hit Pitch", M.hitSoundPitch or 1.0, 0.1, 5, function(v)
        M.hitSoundPitch = v
    end)
    M.hitSoundPitchBox = hitPitchBox

    uiActionRow(PSounds, "▶ Test Hit Sound", function()
        local prev = M.hitSoundEnabled
        M.hitSoundEnabled = true
        M.playHitSound()
        M.hitSoundEnabled = prev
    end)

    uiSectionHeader(PSounds, "INTRO MUSIC")
    local _, setIntro = uiToggleRow(PSounds, "Intro Song", M.introSoundEnabled, function(on)
        M.introSoundEnabled = on
        if not on and introSoundInstance and introSoundInstance.IsPlaying then
            pcall(function() introSoundInstance:Stop() end)
        end
        saveCherryConfig()
    end)

    local _, setIntroSongUI = uiChoiceRow(PSounds, "Intro Song Choice", {"Song 1","Song 2","Song 3"},
        M.introSongChoice or 3,
        function(v)
            local map = {["Song 1"]=1, ["Song 2"]=2, ["Song 3"]=3}
            M.introSongChoice = map[v] or 3
            saveCherryConfig()
        end
    )

    local _, setIntroGUI = uiToggleRow(PSounds, "Intro GUI", M.introGUIEnabled, function(on)
        M.introGUIEnabled = on
        saveCherryConfig()
    end)

    -- PAGE: UTILITY
    uiSectionHeader(PUtil, "MISC")
    local _, setUnwalk = uiToggleRow(PUtil, "Unwalk", M.unwalkEnabled, function(on)
        M.unwalkEnabled = on
        if on then M.startUnwalk() else M.stopUnwalk() end
    end)
    M.setUnwalkVisual = setUnwalk

    local _, setAntiLag = uiToggleRow(PUtil, "Anti-Lag", M.antiLagEnabled, function(on)
        if on then M.enableAntiLag() else M.disableAntiLag() end
        saveCherryConfig()
    end)
    M.setAntiLagVisual = setAntiLag

    local _, setNukeOptimizer = uiToggleRow(PUtil, "Nuke Optimiser", M.nukeOptimizerEnabled, function(on)
        M.nukeOptimizerEnabled = on
        if on then M.enableNukeOptimizer() else M.disableNukeOptimizer() end
        saveCherryConfig()
    end)
    M.setNukeOptimizerVisual = setNukeOptimizer

    local _, setAntiSummer = uiToggleRow(PUtil, "Anti Summer Base", M.antiSummerBaseEnabled, function(on)
        M.antiSummerBaseEnabled = on
        if on then M.enableAntiSummerBase() else M.disableAntiSummerBase() end
        saveCherryConfig()
    end)
    M.setAntiSummerVisual = setAntiSummer

    local _, setStretch = uiToggleRow(PUtil, "Stretch Rez", M.stretchRezEnabled, function(on)
        M.stretchRezEnabled = on
        if on then M.enableStretchRez() else M.disableStretchRez() end
    end)
    M.setStretchRezVisual = setStretch

    local _, setRemoveAcc = uiToggleRow(PUtil, "Remove Accessories", M.removeAccEnabled, function(on)
        M.removeAccEnabled = on
        if on then M.startRemoveAcc() else M.stopRemoveAcc() end
    end)

    local _, setAntiKick = uiToggleRow(PUtil, "Anti-Kick", M.antiKickEnabled, function(on)
        M.antiKickEnabled = on
        if on then M.enableAntiKick() else M.disableAntiKick() end
        saveCherryConfig()
    end)
    M.antiKickSetVisual = setAntiKick

    local _, setSafeMode = uiToggleRow(PUtil, "Safe Mode", M.safeModeEnabled, function(on)
        M.safeModeEnabled = on
        if on then M.enableSafeMode() else M.disableSafeMode() end
        saveCherryConfig()
    end)
    M.setSafeModeVisual = setSafeMode

    do
        local fontIdx = 1
        for i, n in ipairs(M.FONT_NAMES) do
            if n == (M.customFontSelected or "None") then fontIdx = i break end
        end
        local _, setFontUI = uiChoiceRow(PUtil, "Custom Font", M.FONT_NAMES, fontIdx, function(v)
            M.applyCustomFont(v)
            saveCherryConfig()
        end)
    end

    local _, setMobBtns = uiToggleRow(PUtil, "Mobile Buttons", M.mobileButtonsEnabled, function(on)
        M.mobileButtonsEnabled = on
        if on then M.buildMobileButtons() else M.destroyMobileButtons() end
        saveCherryConfig()
    end)

    local _, setCircleBtns = uiToggleRow(PUtil, "Circle Buttons", M.circleButtonsEnabled, function(on)
        M.circleButtonsEnabled = on
        if M.mobileButtonsEnabled then M.buildMobileButtons() end
        saveCherryConfig()
    end)
    M.setCircleBtnsVisual = setCircleBtns

    local _, btnSzBox = uiNumberRow(PUtil, "Button Size", M.mobileButtonsSize, 40, 150, function(v)
        M.mobileButtonsSize = v
        if M.mobileButtonsEnabled then M.buildMobileButtons() end
    end)

    local _, menuScaleBox = uiNumberRow(PUtil, "Menu Scale", M.uiScale, 0.5, 2.0, function(v)
        M.uiScale = v
        if M.uiScaleRef then M.uiScaleRef.Scale = v end
        saveCherryConfig()
    end)

    uiActionRow(PUtil, "Reset Mobile Positions", function() M.resetMobilePositions() end)

    uiSectionHeader(PUtil, "CHARTER")
    local packNames = {}
    for name in pairs(M.PACKS) do table.insert(packNames, name) end
    table.sort(packNames)

    local packDefaultIdx = 1
    for i,v in ipairs(packNames) do if v == M.animPack then packDefaultIdx = i break end end
    local _, setAnimPackToggle, setPackUI = uiExpandToggleRow(
        PUtil,
        "Animation Pack",
        M.animPackEnabled,
        packNames,
        packDefaultIdx,
        function(on)
            M.animPackEnabled = on
            if on then M.applyAnimPack(M.animPack)
            else local char=player.Character; if char then M.resetAnimations(char) end end
            saveCherryConfig()
        end,
        function(v)
            M.animPack = v
            if M.animPackEnabled then M.applyAnimPack(v) end
            saveCherryConfig()
        end
    )
    M.setPackModeUI = setPackUI

    uiActionRow(PUtil, "Apply Animation Pack", function()
        if M.animPackEnabled then M.applyAnimPack(M.animPack) end
        saveCherryConfig()
    end)

    local _, setHeadless = uiToggleRow(PUtil, "Headless", M.headlessEnabled, function(on)
        M.headlessEnabled = on
        M.applyHeadlessToChar(player.Character, on)
        saveCherryConfig()
    end)
    local _, setKorblox = uiToggleRow(PUtil, "Korblox", M.korbloxEnabled, function(on)
        M.korbloxEnabled = on
        M.applyKorbloxToChar(player.Character, on)
        saveCherryConfig()
    end)

    uiSectionHeader(PUtil, "PANELS")
    do
        local r = Instance.new("Frame"); r.ClipsDescendants=true; r.Size=UDim2.new(1,0,0,46)
        r.BackgroundColor3=UI_ROW_BG; r.BackgroundTransparency=0.03; r.BorderSizePixel=0; r.Parent=PUtil; uiCardStyle(r)
        local l = Instance.new("TextLabel"); l.Position=UDim2.new(0,14,0,0); l.Size=UDim2.new(1,-74,1,0)
        l.BackgroundTransparency=1; l.Text="Save Config"; l.TextColor3=UI_TEXT_PRIMARY; l.TextSize=14; l.Font=Enum.Font.GothamMedium; l.TextXAlignment=Enum.TextXAlignment.Left; l.Parent=r
        local sBtn = uiSmallBtn({Parent=r, Pos=UDim2.new(1,-80,0.5,-13), Size=UDim2.new(0,68,0,26),
            Text="SAVE", Col=Color3.fromRGB(200,200,200), TS=12, CR=6, SC=Color3.fromRGB(40,40,40), STr=0.2})
        sBtn.Activated:Connect(function()
            saveCherryConfig()
            sBtn.Text = "OK"
            task.delay(0.8, function() if sBtn and sBtn.Parent then sBtn.Text = "SAVE" end end)
        end)
    end
    do
        local r = Instance.new("Frame"); r.ClipsDescendants=true; r.Size=UDim2.new(1,0,0,46)
        r.BackgroundColor3=UI_ROW_BG; r.BackgroundTransparency=0.03; r.BorderSizePixel=0; r.Parent=PUtil; uiCardStyle(r)
        local l = Instance.new("TextLabel"); l.Position=UDim2.new(0,14,0,0); l.Size=UDim2.new(1,-74,1,0)
        l.BackgroundTransparency=1; l.Text="Reset All Settings"; l.TextColor3=UI_TEXT_PRIMARY; l.TextSize=14; l.Font=Enum.Font.GothamMedium; l.TextXAlignment=Enum.TextXAlignment.Left; l.Parent=r
        local rBtn = uiSmallBtn({Parent=r, Pos=UDim2.new(1,-80,0.5,-13), Size=UDim2.new(0,68,0,26),
            Text="RESET", Col=Color3.fromRGB(200,200,200), TS=12, CR=6, SC=Color3.fromRGB(40,40,40), STr=0.2})
        rBtn.Activated:Connect(function() M.resetAllSettings() end)
    end

    -- PAGE: KEYBINDS
    uiSectionHeader(PKB, "KEYBINDS")
    uiKeybindRow(PKB, "Hide GUI",       M.KB.GuiHide)
    uiKeybindRow(PKB, "Carry Mode",     M.KB.SpeedToggle)
    uiKeybindRow(PKB, "Lagger Mode",    M.KB.LaggerToggle)
    uiKeybindRow(PKB, "Bat Aimbot",     M.KB.AutoBat)
    uiKeybindRow(PKB, "Bat TP",         M.KB.BypassAimbot)
    uiKeybindRow(PKB, "Auto Left",      M.KB.AutoLeft)
    uiKeybindRow(PKB, "Auto Right",     M.KB.AutoRight)
    uiKeybindRow(PKB, "Drop Brainrot",  M.KB.DropBrainrot)
    uiKeybindRow(PKB, "TP Down",        M.KB.TPFloor)
    uiKeybindRow(PKB, "Insta Reset",    M.KB.InstaReset)


    -- Restore menu open/closed from config
    do
        local open = M.menuOpen ~= false
        Frame.Visible = open
        MinPill.Visible = not open
        M.menuOpen = open
    end

    -- APPLY INITIAL STATES
    M.applyStealBarTheme(CHERRY_ACCENT)
    M.updateHeadTheme()
    M.applyFOV()

    M.autoTPHeightBox = tpHBox
    M.radInput = srBox
    M.durationBox = sdBox
    M.btnSzBox = btnSzBox
    M.sbBox = sbBox

    if M.setHitSoundVisual then M.setHitSoundVisual(M.hitSoundEnabled) end
    if M.setHitSoundChoiceUI then M.setHitSoundChoiceUI(M.hitSoundChoice or "Minecraft Critical Hit") end
    if M.setAntiRagVisual then M.setAntiRagVisual(M.antiRagdollEnabled) end
    if M.setSafeModeVisual then M.setSafeModeVisual(M.safeModeEnabled) end
    if M.setAutoCarryVisual then M.setAutoCarryVisual(M.autoSwitchSpeedEnabled) end
    if M.setAutoBatModeUI then M.setAutoBatModeUI(M.autoBatMode or "Normal") end
    if M.setAutoPlayModeUI then M.setAutoPlayModeUI(M.autoPlayMode or "Semi") end
    if M.setCircleBtnsVisual then M.setCircleBtnsVisual(M.circleButtonsEnabled) end
    -- re-apply saved font after GUI rebuild
    if M.customFontSelected and M.customFontSelected ~= "None" then
        task.defer(function() pcall(function() M.applyCustomFont(M.customFontSelected) end) end)
    end
    if M.setMirrorTPVisual then M.setMirrorTPVisual(M.mirrorTPDownEnabled) end
    if M.safeModeEnabled then M.enableSafeMode() end
    if M.antiKickEnabled then M.enableAntiKick() end

    if M.setAntiRagModeUI then M.setAntiRagModeUI(M.antiRagdollMode == "No Splatter" and "No Splatter" or "Splatter") end
    if M.setInfJumpVisual then M.setInfJumpVisual(M.infJumpEnabled) end
    if M.setMedusaVisual then M.setMedusaVisual(M.medusaCounterEnabled) end
    if M.setMedusaResetVisual then M.setMedusaResetVisual(M.medusaResetEnabled) end
    if M.setBatCounterVisual then M.setBatCounterVisual(M.batCounterEnabled) end
    if M.setUnwalkVisual then M.setUnwalkVisual(M.unwalkEnabled) end
    if M.setAntiLagVisual then M.setAntiLagVisual(M.antiLagEnabled) end
    if M.setNukeOptimizerVisual then M.setNukeOptimizerVisual(M.nukeOptimizerEnabled) end
    if M.setAntiSummerVisual then M.setAntiSummerVisual(M.antiSummerBaseEnabled) end
    if M.setStretchRezVisual then M.setStretchRezVisual(M.stretchRezEnabled) end
    if M.setAutoTPVisual then M.setAutoTPVisual(M.autoTPEnabled) end
    if M.antiKickSetVisual then M.antiKickSetVisual(M.antiKickEnabled) end
    if M.setInstaGrab then M.setInstaGrab(M.Steal.AutoStealEnabled) end
    if M.setAutoGrabModeUI then pcall(M.setAutoGrabModeUI, (M.stealMode == "Semi" and "V4") or M.stealMode or "V1") end
    if M.setAutoRadiusVisual then M.setAutoRadiusVisual(M.autoRadiusEnabled) end
    if M.autoBatSetVisual then M.autoBatSetVisual(M.autoBatEnabled) end
    if M.autoLeftSetVisual then M.autoLeftSetVisual(M.autoLeftEnabled) end
    if M.autoRightSetVisual then M.autoRightSetVisual(M.autoRightEnabled) end
    if M.setAutoSwingVisual then M.setAutoSwingVisual(M.autoSwingEnabled) end
    if M.setBypassVisual then M.setBypassVisual(M.bypassAimbotEnabled) end
    if M.mobBtnRefs.autoBat then M.mobBtnRefs.autoBat(M.autoBatEnabled) end
    if M.mobBtnRefs.autoLeft then M.mobBtnRefs.autoLeft(M.autoLeftEnabled) end
    if M.mobBtnRefs.autoRight then M.mobBtnRefs.autoRight(M.autoRightEnabled) end
    if M.mobBtnRefs.carrySpeed then M.mobBtnRefs.carrySpeed(M.carrySpeedActive) end
    if M.mobBtnRefs.lagger then M.mobBtnRefs.lagger(M.laggerModeEnabled) end
    if M.mobBtnRefs.bypass then M.mobBtnRefs.bypass(M.bypassAimbotEnabled) end
    if M.setAutoResetOnDeath then M.setAutoResetOnDeath(M.autoResetOnDeath) end
    if M.headlessEnabled then M.applyHeadlessToChar(player.Character, true) end
    if M.korbloxEnabled then M.applyKorbloxToChar(player.Character, true) end
    if M.wingsAuraEnabled and player.Character then M.applyWingsAura(player.Character) end
    if M.setStealModeUI then
        local lab = "V1"
        if M.stealMode == "V4" or M.stealMode == "Semi" then lab = "V4"
        elseif M.stealMode == "V3" then lab = "V3"
        elseif M.stealMode == "V2" then lab = "V2" end
        M.setStealModeUI(lab)
    end
    if M.setJumpModeUI then M.setJumpModeUI(M.infJumpMode == "hold" and "Hold" or "Manual") end
    if M.setPackModeUI and M.animPack then M.setPackModeUI(M.animPack) end
    if M.animPackEnabled then
        task.wait(0.5); M.applyAnimPack(M.animPack)
    else
        local char = player.Character; if char then M.resetAnimations(char) end
    end


    M.updateStatusRadius()
end

function M.applyStealBarTheme(accentColor)
    local col = accentColor or UI_ACCENT or Color3.fromRGB(255, 0, 0)
    if M.statusFill then
        M.statusFill.BackgroundColor3 = col
    end
    if M.statusDot then
        M.statusDot.BackgroundColor3 = col
    end
    if M.statusMain then
        local st = M.statusMain:FindFirstChildOfClass("UIStroke")
        if st then st.Color = col end
    end
end

-- ============================================================
-- RESET ALL SETTINGS
-- ============================================================
function M.resetAllSettings()
    M.NS = 60
    M.CS = 30
    M.LAGGER_SPEED = 15
    M.LAGGER_CARRY_SPEED = 24.5
    M.speedMethod = "Velocity"
    M.hyperMult = 4
    M._lastSpeedMethod = nil
    M._anchoredBySpeed = nil
    M.carrySpeedActive = false
    M.laggerModeEnabled = false
    M.laggerCarryActive = false
    M.antiRagdollEnabled = false
    M.antiRagdollMode = "Splatter"
    M.infJumpEnabled = false
    M.infJumpMode = "manual"
    M.medusaCounterEnabled = false
    M.batCounterEnabled = false
    M.unwalkEnabled = false
    M.medusaResetEnabled = false
    M.medusaDebounce = false
    M.medusaLastUsed = 0
    M.autoLeftEnabled = false
    M.autoRightEnabled = false
    M.autoBatEnabled = false
    M.autoSwingEnabled = true
    M.autoMoveSwingEnabled = false
    M._laggerPreviousMode = nil
    M.antiLagEnabled = false
    M.removeAccessoriesEnabled = false
    M.stretchRezEnabled = false
    M.autoTPEnabled = false
    M.autoTPHeight = 20
    M.guiTransparencyEnabled = false
    M.mobileButtonsEnabled = true
    M.mobileButtonsSize = 56

    M.circleButtonsEnabled = false
    M.fovValue = 80
    M.fovIndex = 1
    M.autoSwitchSpeedEnabled = false
    M.antiKickEnabled = false
    M.brainrotDetected = false
    M.ragdollGuiEnabled = true
    M.introSoundEnabled = true
    M.introSongChoice = 3
    M.introGUIEnabled = true
    M.Steal.AutoStealEnabled = false
    M.autoRadiusEnabled = false
    M.Steal.StealRadius = 60
    M.Steal.StealDuration = 1.4
    M.Steal.StopTime = 0.35
    M.stealMode = "V1"
    M.Semi.holdMin = 1.3
    M.Semi.holdMax = 2.6
    M.Semi.entryDelay = 0.3
    M.Semi.radius = 10
    M.Semi.primeRange = 80
    M.removeAccEnabled = false
            M.uiScale = 0.5
    M.perButtonDragEnabled = true
    M.stealBarSize = 300
    M.stealBarScale = 0.3
            M.autoResetOnDeath = false
    M.animPack = "Adidas Sports"
    M.headlessEnabled = false
    M.korbloxEnabled = false
    M.bypassAimbotEnabled = false
    M.animPackEnabled = true

    M.stopAutoSteal()
    M.stopBatAimbot()
    M.stopAutoLeft()
    M.stopAutoRight()
    M.stopAntiRagdoll()
    M.stopHoldInfJump()
    M.stopManualInfJumpLoop()
    M.stopMedusaCounter()
    M.stopBatCounter()
    M.stopUnwalk()
    M.disableAntiLag()
    M.disableNukeOptimizer()
    M.disableStretchRez()
    M.stopAutoTP()
    M.disableAntiKick()
    M.stopBypassAimbot()
    M.stopRemoveAcc()
    M.autoResetOnDeath = false
    setupDeathReset()

    saveCherryConfig()
    M.buildGui()
end

-- ============================================================
-- INITIALIZATION
-- ============================================================
repeat task.wait() until game:IsLoaded()
loadCherryConfig()
if M._savedTheme and CHERRY_THEMES[M._savedTheme] then
    CherryConfig.Theme = M._savedTheme
    M.colorScheme = M._savedTheme
elseif M.colorScheme and CHERRY_THEMES[M.colorScheme] then
    CherryConfig.Theme = M.colorScheme
    M._savedTheme = M.colorScheme
end
applyAccentFromTheme()
pcall(saveCherryConfig)
M.buildGui() -- applies M.menuOpen (closed stays closed)
pcall(function()
    if M.applyStealBarTheme then M.applyStealBarTheme(UI_ACCENT) end
    if M.updateHeadTheme then M.updateHeadTheme() end
    if M.mainFrame then M.recolorBlacksToTheme(M.mainFrame) end
    if M.statusGui then M.recolorBlacksToTheme(M.statusGui) end
end)
task.defer(function()
    if M.mobileButtonsEnabled then M.buildMobileButtons() end
    if M.antiRagdollEnabled then M.startAntiRagdoll() end
    if M.infJumpEnabled then
        if M.infJumpMode=="manual" then M.startManualInfJumpLoop()
        elseif M.infJumpMode=="hold" then M.startHoldInfJump() end
    end
    if M.medusaCounterEnabled then M.setupMedusa(player.Character) end
    if M.batCounterEnabled then M.startBatCounter() end
    if M.unwalkEnabled then M.startUnwalk() end
    if M.autoTPEnabled then M.startAutoTP() end
    if M.autoBatEnabled then M.queueAutoBatStart() end
    if M.autoLeftEnabled then M.startAutoLeft() end
    if M.autoRightEnabled then M.startAutoRight() end
    if M.Steal.AutoStealEnabled then M.startAutoSteal() end
    if M.bypassAimbotEnabled then M.startBypassAimbot() end
    if M.antiKickEnabled then M.enableAntiKick() end
    if M.antiLagEnabled then M.enableAntiLag() end
    if M.nukeOptimizerEnabled then M.enableNukeOptimizer() end
    if M.antiSummerBaseEnabled then M.enableAntiSummerBase() end
    if M.stretchRezEnabled then M.enableStretchRez() end
    if M.removeAccEnabled then M.startRemoveAcc() end
    if M.autoResetOnDeath then setupDeathReset() end
end)

if M.animPackEnabled and M.animPack and M.PACKS[M.animPack] then
    task.wait(0.5)
    M.applyAnimPack(M.animPack)
else
    local char = player.Character
    if char then
        M.resetAnimations(char)
    end
end

if M.headlessEnabled or M.korbloxEnabled then
    task.wait(0.3)
    M.applyCharterToChar(player.Character)
end

M.CandyApplyCustomSky(M.currentSkyTheme)

M.updateStatusRadius()

if player.Character then
    M.setupHeadIndicator(player.Character)
    M.setupRagdollTriggers()
end
player.CharacterAdded:Connect(function(char)
    -- Limpa conexões do personagem anterior antes de reinstalar recursos.
    if M._ragdollStateConn then M._ragdollStateConn:Disconnect(); M._ragdollStateConn=nil end
    if M._ragdollPlatformConn then M._ragdollPlatformConn:Disconnect(); M._ragdollPlatformConn=nil end
    if M._toolChildConn then M._toolChildConn:Disconnect(); M._toolChildConn=nil end
    -- Limpa estado transitório do personagem anterior antes de reinstalar recursos.
    if M.destroySpeedObjects then pcall(M.destroySpeedObjects) end
    M._speedLastRoot = nil
    M._speedWasMoving = false
    M._speedObjectsCleaned = false
    M._speedSwitchAcc = 0
    spoofedVelocity = Vector3.zero
    task.wait(0.5)
    if M.currentSkyTheme and M.currentSkyTheme ~= "Off" then
        pcall(function() M.CandyApplyCustomSky(M.currentSkyTheme) end)
    end
    M.setupHeadIndicator(char)
    M.setupRagdollTriggers()
    if M.medusaCounterEnabled then M.setupMedusa(char) end
    if M.batCounterEnabled then M.startBatCounter() end
    if M.unwalkEnabled then task.wait(0.5); M.startUnwalk() end
    if M.autoResetOnDeath then setupDeathReset() end
    if M.animPackEnabled and M.animPack and M.PACKS[M.animPack] then
        task.wait(0.2)
        M.applyAnimPack(M.animPack)
    else
        M.resetAnimations(char)
    end
    if M.headlessEnabled or M.korbloxEnabled then
        task.wait(0.2)
        M.applyCharterToChar(char)
    end
    if M.bypassAimbotEnabled then
        task.wait(0.2)
        M.startBypassAimbot()
    end
end)

-- Lightweight no-collide (no GetDescendants every frame — avoids lag/ping spikes)
do
    local _ncAcc = 0
    RunService.Heartbeat:Connect(function(dt)
        _ncAcc = _ncAcc + dt
        if _ncAcc < 0.35 then return end
        _ncAcc = 0
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= player and p.Character then
                local hrp = p.Character:FindFirstChild("HumanoidRootPart")
                if hrp then hrp.CanCollide = false end
                local head = p.Character:FindFirstChild("Head")
                if head then head.CanCollide = false end
            end
        end
    end)
end

local function destroySpeedObjects()
    if M._anchoredBySpeed then pcall(function() M._anchoredBySpeed.Anchored = false end); M._anchoredBySpeed = nil end
    if M._bodyVel then pcall(function() M._bodyVel:Destroy() end); M._bodyVel = nil end
    if M._bodyPosition then pcall(function() M._bodyPosition:Destroy() end); M._bodyPosition = nil end
    if M._bodyForce then pcall(function() M._bodyForce:Destroy() end); M._bodyForce = nil end
    if M._bodyThrust then pcall(function() M._bodyThrust:Destroy() end); M._bodyThrust = nil end
    if M._linearVel then pcall(function() M._linearVel:Destroy() end); M._linearVel = nil end
    if M._vectorForce then pcall(function() M._vectorForce:Destroy() end); M._vectorForce = nil end
    if M._alignPos then pcall(function() M._alignPos:Destroy() end); M._alignPos = nil end
    if M._rocket then pcall(function() M._rocket:Destroy() end); M._rocket = nil end
    if M._rocketTarget then pcall(function() M._rocketTarget:Destroy() end); M._rocketTarget = nil end
    if M._attLinVel then pcall(function() M._attLinVel:Destroy() end); M._attLinVel = nil end
    if M._attVecForce then pcall(function() M._attVecForce:Destroy() end); M._attVecForce = nil end
    if M._attAlign then pcall(function() M._attAlign:Destroy() end); M._attAlign = nil end
    if M._speedTween then pcall(function() M._speedTween:Cancel() end); M._speedTween = nil end
end
M.destroySpeedObjects = destroySpeedObjects

-- Nova lógica de speed do ONI: aplica velocidade diretamente no HumanoidRootPart
-- e mantém a velocidade exposta reduzida para as leituras externas.
local spoofedVelocity = Vector3.zero
local speedValue = 59

local oldIndex
oldIndex = hookmetamethod(game, "__index", newcclosure(function(self, key)
    if not checkcaller() and (key == "AssemblyLinearVelocity" or key == "Velocity") then
        local character = player.Character
        if character and typeof(self) == "Instance" and self:IsA("BasePart")
            and self.Name == "HumanoidRootPart" and self:IsDescendantOf(character) then
            return spoofedVelocity
        end
    end
    return oldIndex(self, key)
end))

local oldNewIndex
oldNewIndex = hookmetamethod(game, "__newindex", newcclosure(function(self, key, value)
    if not checkcaller() and (key == "AssemblyLinearVelocity" or key == "Velocity") then
        local character = player.Character
        if character and typeof(self) == "Instance" and self:IsA("BasePart")
            and self.Name == "HumanoidRootPart" and self:IsDescendantOf(character) then
            spoofedVelocity = value
            return
        end
    end
    return oldNewIndex(self, key, value)
end))

local function applyVelocitySpeed(speed, forcedDirection)
    local char = player.Character
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not char or not hum or not root or hum.Health <= 0 then return end

    local dir = forcedDirection or hum.MoveDirection
    local requestedSpeed = math.clamp(tonumber(speed) or speedValue, 1, 500)
    if dir.Magnitude > 0.05 then
        -- SetNetworkOwner apenas uma vez por personagem; repetir em todo frame é caro.
        if M._speedLastRoot ~= root then
            M._speedLastRoot = root
            pcall(function()
                if root.SetNetworkOwner then root:SetNetworkOwner(player) end
            end)
        end
        M._speedWasMoving = true
        M._speedObjectsCleaned = false

        local unit = dir.Unit
        local y = root.AssemblyLinearVelocity.Y
        spoofedVelocity = Vector3.new(unit.X * 16, y, unit.Z * 16)
        root.AssemblyLinearVelocity = Vector3.new(unit.X * requestedSpeed, y, unit.Z * requestedSpeed)
    else
        -- Só escreve velocidade quando havia movimento; evita uma escrita física por frame parado.
        if M._speedWasMoving then
            local y = root.AssemblyLinearVelocity.Y
            spoofedVelocity = Vector3.new(0, y, 0)
            root.AssemblyLinearVelocity = Vector3.new(0, y, 0)
            M._speedWasMoving = false
        end
    end
end
M.applyVelocitySpeed = applyVelocitySpeed

RunService.PreSimulation:Connect(function(dt)
    -- Os modos BAT V2 e normal bat continuam fora desta lógica.
    dt = tonumber(dt) or 0
    M._speedSwitchAcc = (M._speedSwitchAcc or 0) + dt
    if M.autoBatEnabled or M.autoLeftEnabled or M.autoRightEnabled then return end

    local char = player.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")
    if not hum or not root then return end
    if M.isRagdollState(hum) then
        M.lastMoveDir = Vector3.zero
        if not M._speedObjectsCleaned then
            destroySpeedObjects()
            M._speedObjectsCleaned = true
        end
        M._speedWasMoving = false
        return
    end

    if hum.Health <= 0 then
        if not M._speedObjectsCleaned then
            destroySpeedObjects()
            M._speedObjectsCleaned = true
        end
        return
    end

    -- A troca automática não precisa ser recalculada em todos os frames.
    if M._speedSwitchAcc >= 0.1 then
        M._speedSwitchAcc = 0
        M.updateAutoSwitchSpeed()
    end
    local md = hum.MoveDirection
    local dir = Vector3.zero
    if md.Magnitude > 0 then
        M.lastMoveDir = md
        dir = md
    elseif M.antiRagdollEnabled and M.lastMoveDir.Magnitude > 0 then
        local anyHeld = false
        for key in pairs(M.MOVE_KEYS) do
            if UIS:IsKeyDown(key) then anyHeld = true; break end
        end
        if anyHeld then dir = M.lastMoveDir end
    end

    speedValue = tonumber(M.getActiveMoveSpeed and M.getActiveMoveSpeed()) or speedValue
    if dir.Magnitude > 0 then
        applyVelocitySpeed(speedValue, dir)
    else
        local y = root.AssemblyLinearVelocity.Y
        if M._speedWasMoving then
            spoofedVelocity = Vector3.new(0, y, 0)
            root.AssemblyLinearVelocity = Vector3.new(0, y, 0)
            M._speedWasMoving = false
        end
        if not M._speedObjectsCleaned then
            destroySpeedObjects()
            M._speedObjectsCleaned = true
        end
    end
end)

task.spawn(function()
    task.wait(2); if M.cursedResetRemote then return end
    for _,desc in ipairs(game:GetDescendants()) do
        if desc:IsA("RemoteEvent") and desc.Name:sub(1,3)=="RE/" then M.cursedResetRemote=desc; break end
    end
end)

task.spawn(function()
    while task.wait(5) do saveCherryConfig() end
end)

M.applyFOV()
task.spawn(function()
    while true do
        task.wait(3)
        pcall(M.saveBtnPositions)
    end
end)

task.spawn(function()
    local plots = workspace:FindFirstChild("Plots")
    if not plots then plots = workspace:WaitForChild("Plots",10) end
    if plots then
        for _,plot in ipairs(plots:GetChildren()) do
            if plot:IsA("Model") then scanPlotNormal(plot) end
        end
        plots.ChildAdded:Connect(function(plot)
            if plot:IsA("Model") then task.wait(0.5); scanPlotNormal(plot) end
        end)
        while true do
            task.wait(5)
            M.animalCache={}; M.promptCache={}; M.stealCache={}
            for _,plot in ipairs(plots:GetChildren()) do
                if plot:IsA("Model") then scanPlotNormal(plot) end
            end
        end
    end
end)

task.spawn(function()
    M.initSemiSync()
    while true do
        task.wait(5)
        if M.Semi.enabled or M.stealMode == "Semi" or M.stealMode == "V4" then
            pcall(M.scanAllPlotsSemi)
        end
    end
end)

function M.refreshSpeedModeLabel()
    -- not used
end

pcall(function()
    M.refreshWalkSpeedAutoSwitch()
    if M.customFontSelected and M.customFontSelected ~= "None" then
        task.spawn(function()
            task.wait(0.4)
            pcall(function() M.applyCustomFont(M.customFontSelected) end)
        end)
    end
end)

-- ============================================================
print("ONI HUB loaded successfully!")
return M
-- ONI HUB -- leaked by discord.gg/printedvs -- Polsec De-obfuscation 90% correct
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local HttpService = game:GetService("HttpService")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local ConfigFile = "RubyHubConfig.json"
local NIVELES = {
    Low = { poder = 32 },
    Mid = { poder = 52 },
    High = { poder = 70 }
}
local keybind = Enum.KeyCode.M
local listeningForInput = false
local laggerActive = false
local lagThread = nil
local nivelActual = "Low"
local ventanaBloqueada = false
local UI_CONFIG = {
    MainBg = Color3.fromRGB(0, 0, 0),
    TitleColor = Color3.fromRGB(255, 255, 255),
    TextColor = Color3.fromRGB(255, 255, 255),
    ButtonInact = Color3.fromRGB(40, 40, 40),
    ButtonAct = Color3.fromRGB(0, 0, 0),
    ToggleOff = Color3.fromRGB(180, 180, 180),
    ToggleOn = Color3.fromRGB(0, 0, 0),
    LockColor = Color3.fromRGB(255, 255, 255),
    UnlockColor = Color3.fromRGB(255, 255, 255),
    Font = Enum.Font.GothamBold,
    BorderColor = Color3.fromRGB(220, 220, 220),
    GlowColor = Color3.fromRGB(255, 0, 0),
    RainColor = Color3.fromRGB(100, 110, 120),
    SelectorBg = Color3.fromRGB(240, 240, 240),
    SelectorAct = Color3.fromRGB(0, 0, 0),
}
local function SaveConfig()
    local data = {
        Keybind = keybind.Name,
        Nivel = nivelActual,
        Bloqueado = ventanaBloqueada
    }
    pcall(function()
        writefile(ConfigFile, HttpService:JSONEncode(data))
    end)
end
local function LoadConfig()
    if pcall(isfile, ConfigFile) and isfile(ConfigFile) then
        pcall(function()
            local data = HttpService:JSONDecode(readfile(ConfigFile))
            keybind = Enum.KeyCode[data.Keybind] or Enum.KeyCode.M
            nivelActual = data.Nivel or "Low"
            ventanaBloqueada = data.Bloqueado or false
        end)
    end
end
LoadConfig()
local function bomb(poder)
    local main, spam = {}, {{}}
    local z = spam[1]
    for i = 1, 25 do
        local t = {}
        table.insert(z, t)
        z = t
    end
    local max = math.min(12000, poder * 50)
    for i = 1, max do
        table.insert(main, spam)
    end
    pcall(function()
        game:GetService("RobloxReplicatedStorage").SetPlayerBlockList:FireServer(main)
    end)
end
local toggleBall, toggleContainer, btnLow, btnMid, btnHigh, lockButton
local titleLabel, subtitleLabel, keybindLabel, keybindButton, toggleClick
local function actualizarBotonesNivel()
    if nivelActual == "Low" then
        btnLow.BackgroundColor3 = UI_CONFIG.ButtonAct
        btnLow.TextColor3 = Color3.fromRGB(255,255,255)
        btnLow.BorderSizePixel = 0
    else
        btnLow.BackgroundColor3 = UI_CONFIG.ButtonInact
        btnLow.TextColor3 = Color3.fromRGB(255,255,255)
        btnLow.BorderSizePixel = 1
        btnLow.BorderColor3 = UI_CONFIG.BorderColor
    end
    if nivelActual == "Mid" then
        btnMid.BackgroundColor3 = UI_CONFIG.ButtonAct
        btnMid.TextColor3 = Color3.fromRGB(255,255,255)
        btnMid.BorderSizePixel = 0
    else
        btnMid.BackgroundColor3 = UI_CONFIG.ButtonInact
        btnMid.TextColor3 = Color3.fromRGB(255,255,255)
        btnMid.BorderSizePixel = 1
        btnMid.BorderColor3 = UI_CONFIG.BorderColor
    end
    if nivelActual == "High" then
        btnHigh.BackgroundColor3 = UI_CONFIG.ButtonAct
        btnHigh.TextColor3 = Color3.fromRGB(255,255,255)
        btnHigh.BorderSizePixel = 0
    else
        btnHigh.BackgroundColor3 = UI_CONFIG.ButtonInact
        btnHigh.TextColor3 = Color3.fromRGB(255,255,255)
        btnHigh.BorderSizePixel = 1
        btnHigh.BorderColor3 = UI_CONFIG.BorderColor
    end
end
local function actualizarSwitch()
    if toggleContainer then
        toggleContainer.BackgroundColor3 = laggerActive and UI_CONFIG.ToggleOn or UI_CONFIG.ToggleOff
    end
    if toggleBall then
        toggleBall.BackgroundColor3 = laggerActive and UI_CONFIG.ToggleOn or UI_CONFIG.ToggleOff
        if laggerActive then
            toggleBall.Position = UDim2.new(1, -18, 0.5, -9)
        else
            toggleBall.Position = UDim2.new(0, 3, 0.5, -9)
        end
    end
    if toggleClick then
        toggleClick.Text = laggerActive and "ON" or "OFF"
        if laggerActive then
            toggleClick.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
            toggleClick.TextColor3 = Color3.fromRGB(255, 255, 255)
        else
            toggleClick.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
            toggleClick.TextColor3 = Color3.fromRGB(0, 0, 0)
        end
    end
end
local function actualizarCandado()
    lockButton.Text = ventanaBloqueada and "Lock" or "Unlock"
    lockButton.TextColor3 = UI_CONFIG.TextColor
end
local function actualizarKeybindButton()
    if keybindButton then
        local display = keybind.Name
        if display:match("Button") then
            display = display:gsub("Button", "")
        end
        keybindButton.Text = display
    end
end
local function toggleLagger()
    laggerActive = not laggerActive
    local targetPos = laggerActive and UDim2.new(1, -18, 0.5, -9) or UDim2.new(0, 3, 0.5, -9)
    local targetColor = laggerActive and UI_CONFIG.ToggleOn or UI_CONFIG.ToggleOff
    TweenService:Create(toggleBall, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
        Position = targetPos,
        BackgroundColor3 = targetColor
    }):Play()
    TweenService:Create(toggleContainer, TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
        BackgroundColor3 = targetColor
    }):Play()
    toggleClick.Text = laggerActive and "ON" or "OFF"
    if laggerActive then
        toggleClick.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        toggleClick.TextColor3 = Color3.fromRGB(255, 255, 255)
    else
        toggleClick.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
        toggleClick.TextColor3 = Color3.fromRGB(0, 0, 0)
    end
    if laggerActive then
        if lagThread then
            task.cancel(lagThread)
        end
        lagThread = task.spawn(function()
            while laggerActive do
                pcall(function()
                    game:GetService("NetworkClient"):SetOutgoingKBPSLimit(80000)
                end)
                bomb(NIVELES[nivelActual].poder)
                task.wait(0.18)
            end
        end)
    else
        if lagThread then
            task.cancel(lagThread); lagThread = nil
        end
    end
end
if CoreGui:FindFirstChild("RubyHub_UI") then
    CoreGui.RubyHub_UI:Destroy()
end
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "RubyHub_UI"
screenGui.Parent = CoreGui
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.ResetOnSpawn = false
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.BackgroundColor3 = UI_CONFIG.MainBg
mainFrame.BorderSizePixel = 2
mainFrame.BorderColor3 = UI_CONFIG.GlowColor
mainFrame.Size = UDim2.new(0, 200, 0, 120)
mainFrame.Position = UDim2.new(0.15, 0, 0.5, -60)
mainFrame.Parent = screenGui
mainFrame.ClipsDescendants = true
Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 8)
local rainParticles = {}
local rainCanvas = Instance.new("Frame")
rainCanvas.Name = "RainCanvas"
rainCanvas.BackgroundTransparency = 1
rainCanvas.Size = UDim2.new(1, 0, 1, 0)
rainCanvas.Parent = mainFrame
rainCanvas.ZIndex = 0
for i = 1, 50 do
    local drop = Instance.new("Frame")
    drop.Name = "RainDrop_" .. i
    drop.BackgroundColor3 = UI_CONFIG.RainColor
    drop.BackgroundTransparency = 0.1 + math.random() * 0.3
    drop.Size = UDim2.new(0, 1 + math.random() * 1.5, 0, 4 + math.random() * 6)
    drop.Position = UDim2.new(math.random(), 0, math.random(), 0)
    drop.BorderSizePixel = 0
    drop.Parent = rainCanvas
    drop.ZIndex = 0
    local speed = 0.5 + math.random() * 0.7
    local drift = (math.random() - 0.5) * 0.1
    table.insert(rainParticles, {
        frame = drop,
        speed = speed,
        drift = drift
    })
end
RunService.Heartbeat:Connect(function(dt)
    for _, p in ipairs(rainParticles) do
        if p.frame and p.frame.Parent then
            local newY = p.frame.Position.Y.Scale + p.speed * dt * 1.5
            if newY > 1 then
                newY = -0.1
                p.frame.Position = UDim2.new(math.random(), 0, newY, 0)
                p.frame.Size = UDim2.new(0, 1 + math.random() * 1.5, 0, 4 + math.random() * 6)
                p.frame.BackgroundTransparency = 0.1 + math.random() * 0.3
            else
                p.frame.Position = UDim2.new(
                    p.frame.Position.X.Scale + p.drift * dt * 0.05, 0,
                    newY, 0
                )
            end
        end
    end
end)
titleLabel = Instance.new("TextLabel", mainFrame)
titleLabel.BackgroundTransparency = 1
titleLabel.Position = UDim2.new(0, 10, 0, 0)
titleLabel.Size = UDim2.new(1, -45, 0, 28)
titleLabel.Font = Enum.Font.GothamBlack
titleLabel.Text = "ONI HUB"
titleLabel.TextColor3 = UI_CONFIG.TitleColor
titleLabel.TextSize = 14
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.TextYAlignment = Enum.TextYAlignment.Center
titleLabel.ZIndex = 1
subtitleLabel = Instance.new("TextLabel", mainFrame)
subtitleLabel.BackgroundTransparency = 1
subtitleLabel.Position = UDim2.new(0, 10, 0, 28)
subtitleLabel.Size = UDim2.new(0, 180, 0, 12)
subtitleLabel.Font = Enum.Font.Gotham
subtitleLabel.Text = "oni hub made by Sami"
subtitleLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
subtitleLabel.TextSize = 8
subtitleLabel.TextXAlignment = Enum.TextXAlignment.Left
subtitleLabel.TextYAlignment = Enum.TextYAlignment.Top
subtitleLabel.ZIndex = 1
lockButton = Instance.new("TextButton", mainFrame)
lockButton.BackgroundTransparency = 1
lockButton.Position = UDim2.new(1, -50, 0, 3)
lockButton.Size = UDim2.new(0, 45, 0, 18)
lockButton.Font = UI_CONFIG.Font
lockButton.TextSize = 10
lockButton.TextColor3 = UI_CONFIG.TextColor
lockButton.AutoButtonColor = false
lockButton.ZIndex = 1
lockButton.MouseButton1Click:Connect(function()
    ventanaBloqueada = not ventanaBloqueada
    actualizarCandado()
    SaveConfig()
end)
actualizarCandado()
local keybindY = 44
keybindLabel = Instance.new("TextLabel", mainFrame)
keybindLabel.BackgroundTransparency = 1
keybindLabel.Position = UDim2.new(0, 10, 0, keybindY)
keybindLabel.Size = UDim2.new(0, 75, 0, 16)
keybindLabel.Font = UI_CONFIG.Font
keybindLabel.Text = "KEYBIND FOR LAGGER •"
keybindLabel.TextColor3 = UI_CONFIG.TextColor
keybindLabel.TextSize = 9
keybindLabel.TextXAlignment = Enum.TextXAlignment.Left
keybindLabel.ZIndex = 1
keybindButton = Instance.new("TextButton", mainFrame)
keybindButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
keybindButton.Position = UDim2.new(0, 115, 0, keybindY)
keybindButton.Size = UDim2.new(0, 20, 0, 16)
keybindButton.Font = UI_CONFIG.Font
keybindButton.Text = "M"
keybindButton.TextColor3 = Color3.fromRGB(255,255,255)
keybindButton.TextSize = 9
keybindButton.AutoButtonColor = false
keybindButton.ZIndex = 1
Instance.new("UICorner", keybindButton).CornerRadius = UDim.new(0, 3)
actualizarKeybindButton()
toggleContainer = Instance.new("Frame", mainFrame)
toggleContainer.BackgroundColor3 = UI_CONFIG.ToggleOff
toggleContainer.Position = UDim2.new(1, -55, 0, keybindY)
toggleContainer.Size = UDim2.new(0, 42, 0, 20)
toggleContainer.ZIndex = 1
Instance.new("UICorner", toggleContainer).CornerRadius = UDim.new(1,0)
toggleBall = Instance.new("Frame", toggleContainer)
toggleBall.BackgroundColor3 = UI_CONFIG.ToggleOff
toggleBall.Size = UDim2.new(0, 18, 0, 18)
toggleBall.Position = UDim2.new(0, 2, 0.5, -9)
toggleBall.ZIndex = 1
Instance.new("UICorner", toggleBall).CornerRadius = UDim.new(1,0)
toggleClick = Instance.new("TextButton", toggleContainer)
toggleClick.BackgroundTransparency = 0
toggleClick.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
toggleClick.Size = UDim2.new(1,0,1,0)
toggleClick.ZIndex = 2
toggleClick.Font = UI_CONFIG.Font
toggleClick.Text = "OFF"
toggleClick.TextSize = 9
toggleClick.TextColor3 = Color3.fromRGB(0, 0, 0)
toggleClick.TextXAlignment = Enum.TextXAlignment.Center
toggleClick.TextYAlignment = Enum.TextYAlignment.Center
toggleClick.MouseButton1Click:Connect(toggleLagger)
toggleClick.AutoButtonColor = false
local corner = Instance.new("UICorner", toggleClick)
corner.CornerRadius = UDim.new(1,0)
keybindButton.MouseButton1Click:Connect(function()
    if listeningForInput then return end
    listeningForInput = true
    keybindButton.Text = "..."
    keybindButton.BackgroundColor3 = UI_CONFIG.GlowColor
    keybindButton.TextColor3 = Color3.fromRGB(255,255,255)
end)
local inputConnection
inputConnection = UserInputService.InputBegan:Connect(function(input, gp)
    if not listeningForInput then return end
    if gp then return end
    local newKey = nil
    if input.KeyCode ~= Enum.KeyCode.Unknown then
        newKey = input.KeyCode
    elseif input.UserInputType == Enum.UserInputType.Gamepad1 and input.KeyCode ~= Enum.KeyCode.Unknown then
        newKey = input.KeyCode
    end
    if newKey then
        keybind = newKey
        actualizarKeybindButton()
        SaveConfig()
        listeningForInput = false
        keybindButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        keybindButton.TextColor3 = Color3.fromRGB(255,255,255)
    end
end)
local btnY = 76
local btnW = 60
local btnH = 24
local espaciado = 5
local margenIzq = 5
local function aplicarEfectoHover(btn)
    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.2), {
            BackgroundColor3 = UI_CONFIG.GlowColor,
            TextColor3 = Color3.fromRGB(255,255,255)
        }):Play()
    end)
    btn.MouseLeave:Connect(function()
        actualizarBotonesNivel()
    end)
end
btnLow = Instance.new("TextButton", mainFrame)
btnLow.Size = UDim2.new(0, btnW, 0, btnH)
btnLow.Position = UDim2.new(0, margenIzq, 0, btnY)
btnLow.Font = UI_CONFIG.Font
btnLow.Text = "LOW"
btnLow.TextColor3 = Color3.fromRGB(255,255,255)
btnLow.TextSize = 10
btnLow.AutoButtonColor = false
btnLow.BackgroundColor3 = UI_CONFIG.ButtonInact
btnLow.BorderSizePixel = 1
btnLow.BorderColor3 = UI_CONFIG.BorderColor
btnLow.ZIndex = 1
Instance.new("UICorner", btnLow).CornerRadius = UDim.new(0, 5)
btnLow.MouseButton1Click:Connect(function()
    nivelActual = "Low"
    actualizarBotonesNivel()
    SaveConfig()
end)
aplicarEfectoHover(btnLow)
btnMid = Instance.new("TextButton", mainFrame)
btnMid.Size = UDim2.new(0, btnW, 0, btnH)
btnMid.Position = UDim2.new(0, margenIzq + btnW + espaciado, 0, btnY)
btnMid.Font = UI_CONFIG.Font
btnMid.Text = "MID"
btnMid.TextColor3 = Color3.fromRGB(255,255,255)
btnMid.TextSize = 10
btnMid.AutoButtonColor = false
btnMid.BackgroundColor3 = UI_CONFIG.ButtonInact
btnMid.BorderSizePixel = 1
btnMid.BorderColor3 = UI_CONFIG.BorderColor
btnMid.ZIndex = 1
Instance.new("UICorner", btnMid).CornerRadius = UDim.new(0, 5)
btnMid.MouseButton1Click:Connect(function()
    nivelActual = "Mid"
    actualizarBotonesNivel()
    SaveConfig()
end)
aplicarEfectoHover(btnMid)
btnHigh = Instance.new("TextButton", mainFrame)
btnHigh.Size = UDim2.new(0, btnW, 0, btnH)
btnHigh.Position = UDim2.new(0, margenIzq + (btnW + espaciado) * 2, 0, btnY)
btnHigh.Font = UI_CONFIG.Font
btnHigh.Text = "HIGH"
btnHigh.TextColor3 = Color3.fromRGB(255,255,255)
btnHigh.TextSize = 10
btnHigh.AutoButtonColor = false
btnHigh.BackgroundColor3 = UI_CONFIG.ButtonInact
btnHigh.BorderSizePixel = 1
btnHigh.BorderColor3 = UI_CONFIG.BorderColor
btnHigh.ZIndex = 1
Instance.new("UICorner", btnHigh).CornerRadius = UDim.new(0, 5)
btnHigh.MouseButton1Click:Connect(function()
    nivelActual = "High"
    actualizarBotonesNivel()
    SaveConfig()
end)
aplicarEfectoHover(btnHigh)
actualizarBotonesNivel()
actualizarSwitch()
local isDragging, dragStart, startPos = false, nil, nil
mainFrame.InputBegan:Connect(function(input)
    if ventanaBloqueada then return end
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        isDragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if not isDragging or ventanaBloqueada then return end
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
mainFrame.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        isDragging = false
    end
end)
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == keybind or (input.UserInputType == Enum.UserInputType.Gamepad1 and input.KeyCode == keybind) then
        toggleLagger()
    end
end)
