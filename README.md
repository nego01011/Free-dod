--[[
    Babis | Dodge Or Die
    Adapted from original "Babis Hub v3.8" — UI only, functions preserved.
]]

local Players           = game:GetService("Players")
local RunService        = game:GetService("RunService")
local TweenService      = game:GetService("TweenService")
local UserInputService  = game:GetService("UserInputService")
local StarterGui        = game:GetService("StarterGui")
local Workspace         = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local HttpService       = game:GetService("HttpService")
local SoundService      = game:GetService("SoundService")
local PathfindingService= game:GetService("PathfindingService")
local Debris            = game:GetService("Debris")
local LocalPlayer       = Players.LocalPlayer
local PlayerGui         = LocalPlayer:WaitForChild("PlayerGui")

-- ============================================================
-- CLEANUP PREVIOUS
-- ============================================================
if getgenv().BabisHub           then pcall(function() getgenv().BabisHub:Destroy() end) end
if getgenv().BabisConnections   then for _, c in ipairs(getgenv().BabisConnections) do pcall(function() c:Disconnect() end) end end
if getgenv().BabisAutoWinPlatform then pcall(function() getgenv().BabisAutoWinPlatform:Destroy() end) end
if getgenv().BabisESPFolder     then pcall(function() getgenv().BabisESPFolder:Destroy() end) end
if getgenv().BabisESPConn       then pcall(function() getgenv().BabisESPConn:Disconnect() end) end
if getgenv().BabisAntiKickHook  then pcall(function() getgenv().BabisAntiKickHook() end) end
if getgenv().BabisWatermark     then pcall(function() getgenv().BabisWatermark:Destroy() end) end

getgenv().BabisConnections = {}
local function StoreConn(conn) table.insert(getgenv().BabisConnections, conn) return conn end

-- ============================================================
-- INTRO ASSET
-- ============================================================
local INTRO_IMAGE_ID = "rbxassetid://119251117614023"
local INTRO_DURATION = 5.5
local INTRO_IMG_W = 156
local INTRO_IMG_H = 139

-- ============================================================
-- CONFIG — BABIS BLUE THEME
-- ============================================================
local CONFIG = {
    appName  = "Babis | Dodge Or Die",
    version  = "v1.0.0",
    edition  = "Free Edition",

    discordInvite = "hunter.exe7133",

    colors = {
        windowBg      = Color3.fromRGB(8, 10, 14),
        headerTint    = Color3.fromRGB(8, 10, 14),
        divider       = Color3.fromRGB(65, 150, 245),
        accent        = Color3.fromRGB(90, 165, 255),
        accentSoft    = Color3.fromRGB(160, 210, 255),
        accentDark    = Color3.fromRGB(40, 90, 160),

        navBg         = Color3.fromRGB(18, 18, 19),
        navBorder     = Color3.fromRGB(90, 88, 92),
        navIcon       = Color3.fromRGB(52, 61, 69),
        navIconActive = Color3.fromRGB(160, 210, 255),
        navActiveBg   = Color3.fromRGB(18, 18, 19),
        navActiveBrd  = Color3.fromRGB(160, 210, 255),

        cardBg        = Color3.fromRGB(16, 16, 20),
        cardBorder    = Color3.fromRGB(56, 55, 62),

        toggleOff     = Color3.fromRGB(38, 38, 44),
        toggleOn      = Color3.fromRGB(90, 165, 255),

        textPrimary   = Color3.fromRGB(240, 240, 248),
        textDesc      = Color3.fromRGB(150, 148, 168),
        textMuted     = Color3.fromRGB(130, 130, 150),

        closeHover    = Color3.fromRGB(220, 70, 90),

        handIdle      = Color3.fromRGB(240, 245, 255),
        handHover     = Color3.fromRGB(255, 255, 255),

        cfgIcon       = Color3.fromRGB(170, 120, 255),
        cfgBtnBg      = Color3.fromRGB(24, 24, 28),
        cfgBtnBorder  = Color3.fromRGB(48, 48, 56),

        bellBg        = Color3.fromRGB(30, 38, 48),
        bellTint      = Color3.fromRGB(90, 165, 255),

        discordGradLeft  = Color3.fromRGB(88, 101, 242),
        discordGradRight = Color3.fromRGB(22, 24, 45),
        discordBorder    = Color3.fromRGB(88, 101, 242),

        premium       = Color3.fromRGB(255, 200, 60),
        lockedCard    = Color3.fromRGB(14, 14, 18),
        lockedBorder  = Color3.fromRGB(40, 38, 44),
        lockedText    = Color3.fromRGB(120, 118, 135),
    },

    sizes = {
        windowWidth  = 460,
        windowHeight = 700,
        windowRadius = 22,
        headerHeight = 82,

        navHeight    = 88,
        navBtnSize   = 70,
        navBtnRadius = 14,
        navGap       = 8,
        navPadX      = 12,

        contentPadX  = 14,
        scrollPadL   = 6,
        scrollPadR   = 8,

        iconSize     = 46,
        mainIconBoost= 6,
        playerIconBoost = 36,

        cardHeight   = 128,
        cardRadius   = 16,
        cardPadX     = 16,
        cardGap      = 14,

        toggleW      = 72,
        toggleH      = 38,
        toggleKnob   = 30,
        toggleRight  = 22,

        controlW     = 130,
        controlH     = 44,
        controlRight = 22,

        handIconSize = 82,
        handIconY    = 20,

        cfgHeight    = 516,
    },

    fonts = {
        appName = Enum.Font.GothamBold,
        version = Enum.Font.Gotham,
        intro   = Enum.Font.Code,
    },

    textSizes = {
        appName = 22,
        version = 15,
        intro   = 20,

        cardTitle = 24,
        cardDesc  = 15,
        cardBtn   = 18,

        cfgTitle  = 22,
        cfgLabel  = 17,
        cfgBtn    = 17,
        cfgHint   = 16,
        cfgInput  = 17,

        aboutTitle   = 30,
        aboutDesc    = 19,
        discordTitle = 28,
        discordSub   = 18,
    },

    anim = {
        fast   = 0.14,
        normal = 0.20,
        slow   = 0.32,
        easing = Enum.EasingStyle.Quint,
        dir    = Enum.EasingDirection.Out,
    },

    tabs = {
        { id = "main",     name = "Movement", iconId = 7733960981,     boost = true,  boostAmt = 6  },
        { id = "visual",   name = "Visuals",  iconId = 7733774602,     boost = false, boostAmt = 0  },
        { id = "player",   name = "Player",   iconId = 92187976272467, boost = true,  boostAmt = 36 },
        { id = "misc",     name = "Misc",     iconId = 10723387563,    boost = false, boostAmt = 0  },
        { id = "settings", name = "Settings", iconId = 7734053495,     boost = false, boostAmt = 0  },
    },
    defaultTab = "main",

    handIconId    = "rbxassetid://88060480140568",
    cfgIconId     = "rbxassetid://10709791036",
    infoIconId    = "rbxassetid://7733964719",
    discordIconId = "rbxassetid://100770414662869",
    lockIconId    = "rbxassetid://113617763566572",

    bellIconId          = "rbxassetid://7072706001",
    notifySoundId       = "rbxassetid://5153734608",
    notifySoundVol      = 0.5,

    minimizedDefaultPos = UDim2.new(0.5, 0, 0.5, 0),
    normalPos           = UDim2.new(0.5, 0, 0.5, 0),

    cfgFolder     = "Babis/configs",
    cfgExt        = ".json",
    autoloadFile  = "Babis/autoload.txt",
}

-- ============================================================
-- HELPERS
-- ============================================================
local function destroyPrevious()
    for _, n in ipairs({"EssenceUI", "BabisIntro", "BabisNotification", "BabisHub", "BabisWatermark"}) do
        local old = PlayerGui:FindFirstChild(n)
        if old then old:Destroy() end
    end
end

local function corner(parent, r)
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, r)
    c.Parent = parent
    return c
end

local function stroke(parent, color, thickness, transparency)
    local s = Instance.new("UIStroke")
    s.Color = color
    s.Thickness = thickness or 1
    s.Transparency = transparency or 0
    s.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    s.Parent = parent
    return s
end

local function tween(obj, props, dur, style, dir)
    local info = TweenInfo.new(dur or CONFIG.anim.normal, style or CONFIG.anim.easing, dir or CONFIG.anim.dir)
    local t = TweenService:Create(obj, info, props)
    t:Play()
    return t
end

local function popTween(obj, props, dur)
    local info = TweenInfo.new(dur or 0.25, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
    local t = TweenService:Create(obj, info, props)
    t:Play()
    return t
end

local function safeCall(fn, ...)
    if type(fn) ~= "function" then return false end
    local ok, res = pcall(fn, ...)
    if ok then return res end
    return nil
end

-- ============================================================
-- INTRO
-- ============================================================
local function showIntro(onComplete)
    local introGui = Instance.new("ScreenGui")
    introGui.Name           = "BabisIntro"
    introGui.ResetOnSpawn   = false
    introGui.IgnoreGuiInset = true
    introGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    introGui.DisplayOrder   = 200
    introGui.Parent         = PlayerGui

    local overlay = Instance.new("Frame")
    overlay.Size = UDim2.new(1, 0, 1, 0)
    overlay.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    overlay.BackgroundTransparency = 1
    overlay.BorderSizePixel = 0
    overlay.Parent = introGui

    local center = Instance.new("Frame")
    center.Size = UDim2.new(0, 260, 0, 260)
    center.Position = UDim2.new(0.5, 0, 0.5, -30)
    center.AnchorPoint = Vector2.new(0.5, 0.5)
    center.BackgroundTransparency = 1
    center.Parent = overlay

    local glow = Instance.new("Frame")
    glow.Size = UDim2.new(0, 220, 0, 220)
    glow.Position = UDim2.new(0.5, 0, 0.5, 0)
    glow.AnchorPoint = Vector2.new(0.5, 0.5)
    glow.BackgroundColor3 = CONFIG.colors.accent
    glow.BackgroundTransparency = 1
    glow.BorderSizePixel = 0
    glow.Parent = center
    corner(glow, 110)

    local glowRing = Instance.new("Frame")
    glowRing.Size = UDim2.new(0, 260, 0, 260)
    glowRing.Position = UDim2.new(0.5, 0, 0.5, 0)
    glowRing.AnchorPoint = Vector2.new(0.5, 0.5)
    glowRing.BackgroundTransparency = 1
    glowRing.Parent = center
    corner(glowRing, 130)
    local ringStroke = stroke(glowRing, CONFIG.colors.accent, 1, 1)

    local image = Instance.new("ImageLabel")
    image.Size = UDim2.new(0, INTRO_IMG_W, 0, INTRO_IMG_H)
    image.Position = UDim2.new(0.5, 0, 0.5, 0)
    image.AnchorPoint = Vector2.new(0.5, 0.5)
    image.BackgroundTransparency = 1
    image.Image = INTRO_IMAGE_ID
    image.ImageTransparency = 1
    image.ScaleType = Enum.ScaleType.Fit
    image.Parent = center

    local subtitle = Instance.new("TextLabel")
    subtitle.Size = UDim2.new(1, 0, 0, 30)
    subtitle.Position = UDim2.new(0.5, 0, 0.5, 120)
    subtitle.AnchorPoint = Vector2.new(0.5, 0)
    subtitle.BackgroundTransparency = 1
    subtitle.Text = "CONNECTING TO SERVER"
    subtitle.TextColor3 = Color3.fromRGB(225, 230, 245)
    subtitle.TextTransparency = 1
    subtitle.Font = CONFIG.fonts.intro
    subtitle.TextSize = CONFIG.textSizes.intro
    subtitle.TextXAlignment = Enum.TextXAlignment.Center
    subtitle.Parent = overlay

    local line = Instance.new("Frame")
    line.Size = UDim2.new(0, 0, 0, 1)
    line.Position = UDim2.new(0.5, 0, 0.5, 158)
    line.AnchorPoint = Vector2.new(0.5, 0)
    line.BackgroundColor3 = CONFIG.colors.accent
    line.BackgroundTransparency = 1
    line.BorderSizePixel = 0
    line.Parent = overlay

    tween(overlay, { BackgroundTransparency = 0.55 }, 0.5)
    tween(image, { ImageTransparency = 0 }, 0.5)
    tween(glow, { BackgroundTransparency = 0.88 }, 0.6)
    tween(ringStroke, { Transparency = 0.6 }, 0.6)
    tween(subtitle, { TextTransparency = 0 }, 0.5)
    tween(line, { BackgroundTransparency = 0.2 }, 0.5)
    tween(line, { Size = UDim2.new(0, 190, 0, 1) }, 0.7)

    local spinning = true

    task.spawn(function()
        while spinning do
            local spin = TweenService:Create(image,
                TweenInfo.new(1.6, Enum.EasingStyle.Quint, Enum.EasingDirection.Out),
                { Rotation = image.Rotation + 360 })
            spin:Play()
            spin.Completed:Wait()
            if not spinning then break end
            task.wait(0.45)
        end
    end)

    task.spawn(function()
        local base = "CONNECTING TO SERVER"
        local n = 0
        while spinning do
            n = (n + 1) % 4
            subtitle.Text = base .. string.rep(".", n)
            task.wait(0.35)
        end
    end)

    task.spawn(function()
        while spinning do
            tween(glow, { BackgroundTransparency = 0.95 }, 0.9)
            tween(ringStroke, { Transparency = 0.9 }, 0.9)
            task.wait(0.9)
            if not spinning then break end
            tween(glow, { BackgroundTransparency = 0.82 }, 0.9)
            tween(ringStroke, { Transparency = 0.4 }, 0.9)
            task.wait(0.9)
        end
    end)

    task.delay(INTRO_DURATION, function()
        spinning = false
        tween(overlay, { BackgroundTransparency = 1 }, 0.55)
        tween(image, { ImageTransparency = 1 }, 0.5)
        tween(glow, { BackgroundTransparency = 1 }, 0.5)
        tween(ringStroke, { Transparency = 1 }, 0.5)
        tween(subtitle, { TextTransparency = 1 }, 0.4)
        tween(line, { BackgroundTransparency = 1 }, 0.4)
        tween(line, { Size = UDim2.new(0, 0, 0, 1) }, 0.5)
        task.wait(0.6)
        introGui:Destroy()
        onComplete()
    end)
end

-- ============================================================
-- NOTIFICATIONS
-- ============================================================
local function createNotifier()
    local notifGui = Instance.new("ScreenGui")
    notifGui.Name           = "BabisNotification"
    notifGui.ResetOnSpawn   = false
    notifGui.IgnoreGuiInset = true
    notifGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    notifGui.DisplayOrder   = 300
    notifGui.Parent         = PlayerGui

    local notifySound = Instance.new("Sound")
    notifySound.SoundId = CONFIG.notifySoundId
    notifySound.Volume  = CONFIG.notifySoundVol
    notifySound.Parent  = SoundService

    local container = Instance.new("Frame")
    container.BackgroundTransparency = 1
    container.Size = UDim2.new(0, 320, 0, 0)
    container.AutomaticSize = Enum.AutomaticSize.Y
    container.AnchorPoint = Vector2.new(1, 0)
    container.Position = UDim2.new(1, -20, 0, 20)
    container.Parent = notifGui

    local layout = Instance.new("UIListLayout")
    layout.FillDirection = Enum.FillDirection.Vertical
    layout.Padding = UDim.new(0, 10)
    layout.SortOrder = Enum.SortOrder.LayoutOrder
    layout.VerticalAlignment = Enum.VerticalAlignment.Top
    layout.HorizontalAlignment = Enum.HorizontalAlignment.Right
    layout.Parent = container

    -- ALL BLUE by default; premium = gold (used only for confirmed PAID features)
    local kindColors = {
        info    = Color3.fromRGB(90, 165, 255),
        success = Color3.fromRGB(90, 165, 255),
        warn    = Color3.fromRGB(90, 165, 255),
        error   = Color3.fromRGB(90, 165, 255),
        premium = Color3.fromRGB(255, 200, 60),
    }

    local order = 0

    local function dismiss(row)
        if not row or not row.Parent then return end
        if row:GetAttribute("Dismissing") then return end
        row:SetAttribute("Dismissing", true)

        local card = row:FindFirstChild("Card")
        local startSize = row.Size

        if card then
            card.AnchorPoint = Vector2.new(0, 0)
            tween(card, {
                Position = UDim2.new(1, 420, 0, 0),
                BackgroundTransparency = 0.7,
            }, 0.34, Enum.EasingStyle.Quint, Enum.EasingDirection.In)
            for _, d in ipairs(card:GetDescendants()) do
                if d:IsA("TextLabel") then
                    tween(d, { TextTransparency = 1 }, 0.28, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
                elseif d:IsA("ImageLabel") then
                    tween(d, { ImageTransparency = 1 }, 0.28, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
                elseif d:IsA("Frame") then
                    tween(d, { BackgroundTransparency = 1 }, 0.28, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
                elseif d:IsA("UIStroke") then
                    tween(d, { Transparency = 1 }, 0.28, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
                end
            end
        end

        tween(row, {
            Size = UDim2.new(startSize.X.Scale, startSize.X.Offset, 0, 0)
        }, 0.34, Enum.EasingStyle.Quint, Enum.EasingDirection.In)

        task.delay(0.38, function()
            if row and row.Parent then row:Destroy() end
        end)
    end

    local function notify(title, message, kind, duration)
        title    = title    or "Notification"
        message  = message  or ""
        kind     = kind     or "info"
        duration = duration or 2.0

        if notifySound.IsPlaying then notifySound:Stop() end
        notifySound.TimePosition = 0
        notifySound:Play()

        local accentColor = kindColors[kind] or kindColors.info
        order = order + 1

        local row = Instance.new("Frame")
        row.Size = UDim2.new(1, 0, 0, 76)
        row.BackgroundTransparency = 1
        row.ClipsDescendants = false
        row.LayoutOrder = order
        row.Parent = container

        local card = Instance.new("Frame")
        card.Name = "Card"
        card.Size = UDim2.new(1, 0, 1, 0)
        card.Position = UDim2.new(1, 380, 0, 0)
        card.BackgroundColor3 = Color3.fromRGB(18, 20, 26)
        card.BorderSizePixel = 0
        card.Parent = row
        corner(card, 12)
        stroke(card, Color3.fromRGB(56, 55, 62), 1.5, 0)

        local bar = Instance.new("Frame")
        bar.Size = UDim2.new(0, 5, 1, -22)
        bar.Position = UDim2.new(0, 9, 0, 11)
        bar.BackgroundColor3 = accentColor
        bar.BorderSizePixel = 0
        bar.Parent = card
        corner(bar, 3)

        local bellBg = Instance.new("Frame")
        bellBg.Size = UDim2.new(0, 42, 0, 42)
        bellBg.Position = UDim2.new(0, 20, 0.5, -21)
        bellBg.BackgroundColor3 = CONFIG.colors.bellBg
        bellBg.BorderSizePixel = 0
        bellBg.Parent = card
        corner(bellBg, 21)
        stroke(bellBg, accentColor, 1.2, 0.4)

        local bell = Instance.new("ImageLabel")
        bell.Size = UDim2.new(0, 28, 0, 28)
        bell.Position = UDim2.new(0.5, -14, 0.5, -14)
        bell.BackgroundTransparency = 1
        bell.Image = CONFIG.bellIconId
        bell.ImageColor3 = accentColor
        bell.ScaleType = Enum.ScaleType.Fit
        bell.Parent = bellBg

        local titleLbl = Instance.new("TextLabel")
        titleLbl.BackgroundTransparency = 1
        titleLbl.Text = title
        titleLbl.TextColor3 = Color3.fromRGB(245, 248, 255)
        titleLbl.Font = Enum.Font.GothamBold
        titleLbl.TextSize = 16
        titleLbl.TextXAlignment = Enum.TextXAlignment.Left
        titleLbl.TextYAlignment = Enum.TextYAlignment.Top
        titleLbl.Size = UDim2.new(1, -90, 0, 22)
        titleLbl.Position = UDim2.new(0, 72, 0, 16)
        titleLbl.Parent = card

        local msgLbl = Instance.new("TextLabel")
        msgLbl.BackgroundTransparency = 1
        msgLbl.Text = message
        msgLbl.TextColor3 = Color3.fromRGB(160, 165, 185)
        msgLbl.Font = Enum.Font.Gotham
        msgLbl.TextSize = 14
        msgLbl.TextXAlignment = Enum.TextXAlignment.Left
        msgLbl.TextYAlignment = Enum.TextYAlignment.Top
        msgLbl.TextWrapped = true
        msgLbl.Size = UDim2.new(1, -90, 0, 34)
        msgLbl.Position = UDim2.new(0, 72, 0, 38)
        msgLbl.Parent = card

        tween(card, { Position = UDim2.new(0, 0, 0, 0) }, 0.35)

        task.spawn(function()
            task.wait(0.4)
            if not bell.Parent then return end
            bell.Rotation = -12
            tween(bell, { Rotation = 12 }, 0.10, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
            task.wait(0.11)
            if not bell.Parent then return end
            tween(bell, { Rotation = -8 }, 0.10, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
            task.wait(0.11)
            if not bell.Parent then return end
            tween(bell, { Rotation = 6 }, 0.10, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
            task.wait(0.11)
            if not bell.Parent then return end
            tween(bell, { Rotation = 0 }, 0.12, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
        end)

        task.spawn(function()
            task.wait(duration)
            dismiss(row)
        end)

        return row
    end

    return notify
end

-- ============================================================
-- AHOSP-STYLE CONFIG SYSTEM (kept from Babis)
-- ============================================================
local ConfigSys = {}
do
    function ConfigSys.ensure()
        if not safeCall(isfolder, "Babis") then safeCall(makefolder, "Babis") end
        if not safeCall(isfolder, CONFIG.cfgFolder) then safeCall(makefolder, CONFIG.cfgFolder) end
    end

    function ConfigSys.list()
        ConfigSys.ensure()
        local out = {}
        local files = safeCall(listfiles, CONFIG.cfgFolder)
        if type(files) ~= "table" then return out end
        for _, path in ipairs(files) do
            local name = path:match("([^/\\]+)$") or path
            name = name:gsub(CONFIG.cfgExt:gsub("%.", "%%."), "")
            table.insert(out, name)
        end
        return out
    end

    function ConfigSys.save(name, data)
        ConfigSys.ensure()
        if type(name) ~= "string" or name == "" then return false, "invalid name" end
        local path = CONFIG.cfgFolder .. "/" .. name .. CONFIG.cfgExt
        local okEnc, resEnc = pcall(function() return HttpService:JSONEncode(data) end)
        if not okEnc then return false, "encode failed" end
        local ok = safeCall(writefile, path, resEnc)
        if ok == nil then return false, "writefile failed" end
        return true
    end

    function ConfigSys.load(name)
        ConfigSys.ensure()
        local path = CONFIG.cfgFolder .. "/" .. name .. CONFIG.cfgExt
        local content = safeCall(readfile, path)
        if type(content) ~= "string" then return nil end
        local ok, data = pcall(function() return HttpService:JSONDecode(content) end)
        if not ok then return nil end
        return data
    end

    function ConfigSys.delete(name)
        local path = CONFIG.cfgFolder .. "/" .. name .. CONFIG.cfgExt
        return safeCall(delfile, path) ~= nil
    end

    function ConfigSys.getAutoload()
        local content = safeCall(readfile, CONFIG.autoloadFile)
        if type(content) == "string" and content ~= "" then return content end
        return nil
    end

    function ConfigSys.setAutoload(name)
        ConfigSys.ensure()
        if name == nil or name == "" then
            safeCall(delfile, CONFIG.autoloadFile)
        else
            safeCall(writefile, CONFIG.autoloadFile, name)
        end
    end
end

-- ============================================================
-- ═══════════════ ORIGINAL SCRIPT FUNCTIONS (PRESERVED) ═══════════
-- ============================================================
local States = {
    AutoWin = false, AutoDodge = false, AntiAbilities = false,
    ESPBall = false, AutoPlay = false,
}

-- ── Anti Kick (FREE, always-on by default) ──
local AntiKickActive = true
local OldNamecall
local hasHook = typeof(hookmetamethod) == "function"
if hasHook then
    OldNamecall = hookmetamethod(game, "__namecall", function(self, ...)
        if not AntiKickActive then return OldNamecall(self, ...) end
        local method = getnamecallmethod()
        if self == LocalPlayer and method == "Kick" then
            warn("[Babis-AntiKick] Blocked kick attempt")
            return nil
        end
        return OldNamecall(self, ...)
    end)
end

getgenv().BabisAntiKickHook = function()
    AntiKickActive = false
    getgenv().BabisAntiKickHook = nil
end

-- ── Auto Win (PAID) ──
local AutoWin_OriginalPos = nil
local AutoWin_ReturnLoop = nil
local PLATFORM_POS = Vector3.new(-330.09, -500, 27.99)

local function AutoWin_Toggle(on)
    local char = LocalPlayer.Character
    if not char then States.AutoWin = false; return false end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then States.AutoWin = false; return false end

    if on then
        AutoWin_OriginalPos = hrp.CFrame
        if Workspace:FindFirstChild("BabisAutoWinPlatform") then Workspace.BabisAutoWinPlatform:Destroy() end
        local p = Instance.new("Part")
        p.Name = "BabisAutoWinPlatform"
        p.Size = Vector3.new(30, 2, 30)
        p.Position = PLATFORM_POS
        p.Anchored = true
        p.CanCollide = true
        p.Material = Enum.Material.SmoothPlastic
        p.Color = Color3.fromRGB(200, 200, 200)
        p.Parent = Workspace
        getgenv().BabisAutoWinPlatform = p
        hrp.CFrame = CFrame.new(p.Position + Vector3.new(0, 5, 0))
        return true
    else
        if AutoWin_OriginalPos then
            local targetPos = AutoWin_OriginalPos
            AutoWin_OriginalPos = nil
            if AutoWin_ReturnLoop then pcall(function() AutoWin_ReturnLoop:Disconnect() end) end
            task.spawn(function()
                local startTime = tick()
                while tick() - startTime < 2 do
                    local c = LocalPlayer.Character
                    local h = c and c:FindFirstChild("HumanoidRootPart")
                    if h and h.Parent then
                        h.CFrame = targetPos
                        h.AssemblyLinearVelocity = Vector3.zero
                        h.AssemblyAngularVelocity = Vector3.zero
                    end
                    task.wait(0.03)
                end
            end)
        end
        if getgenv().BabisAutoWinPlatform then
            pcall(function() getgenv().BabisAutoWinPlatform:Destroy() end)
            getgenv().BabisAutoWinPlatform = nil
        end
        return true
    end
end

-- ── Auto Dodge (FREE) ──
local DodgeConfig = {
    ScanRadius = 200,
    BoxWidth = 10, BoxHeight = 10, BoxDepth = 10, BoxOffsetY = 0,
    MaxStep = 4, UrgentStep = 9,
    PredictTime = 0.12, LeadTime = 0.35, UrgentTime = 0.16,
    ExtraMargin = 0.35, ClearMargin = 0.6,
    MaxSubsteps = 20, SubstepStud = 2,
    Predictive = true, Vertical = true, KeepVelocity = true, NoDrag = true,
    CenterDeadzone = 0.6, SideCommitTime = 0.4,
    Smoothing = 0.55, UrgentSmoothing = 1,
    MinStep = 0.03, VerticalDamp = 0.35, SlowSpeed = 2,
    ClearanceProbe = 14, ClearanceMin = 5, WallPadding = 1.2, GroundDrop = 60,
    RefreshInterval = 0.2,
    SphereMinSpeed = 4, SphereMinSize = 3, MaxBalls = 24,
    BallsFolderName = "Balls",
    PracticePath = {"Lobby","PracticeArea","PracticeBalls"},
    BallKeywords = {"ball","practice"},
}

local v3new, v3zero = Vector3.new, Vector3.zero
local mHuge, mMin, mMax, mAbs, mCeil, mClamp = math.huge, math.min, math.max, math.abs, math.ceil, math.clamp
local tInsert, sLower, sFind = table.insert, string.lower, string.find

local BallCache, BallVelocity, BallSample, BlockedAxes = {}, {}, {}, {}
local BallsContainer, PracticeContainer, BallsFolder3
local lastRefresh, sideSign, sideChosenAt, lastEngaged = 0, 1, 0, 0
local DodgeChar, DodgeHum, DodgeRoot
local DodgeConn = nil
local dodgeRay = RaycastParams.new()
dodgeRay.FilterType = Enum.RaycastFilterType.Exclude
dodgeRay.IgnoreWater = true

local function DodgeUpdateFilter()
    local f = {}
    if DodgeChar then tInsert(f, DodgeChar) end
    if BallsContainer then tInsert(f, BallsContainer) end
    if PracticeContainer then tInsert(f, PracticeContainer) end
    if BallsFolder3 then tInsert(f, BallsFolder3) end
    dodgeRay.FilterDescendantsInstances = f
end

local function DodgeBindChar(char)
    DodgeChar = char
    DodgeHum = char:FindFirstChildOfClass("Humanoid") or char:WaitForChild("Humanoid", 3)
    DodgeRoot = char:FindFirstChild("HumanoidRootPart") or char:WaitForChild("HumanoidRootPart", 3)
    DodgeUpdateFilter()
end

if LocalPlayer.Character then DodgeBindChar(LocalPlayer.Character) end
LocalPlayer.CharacterAdded:Connect(DodgeBindChar)

local function resolvePath(root, path)
    local n = root
    for _, name in ipairs(path) do if not n then return nil end; n = n:FindFirstChild(name) end
    return n
end

local function matchesKeyword(name)
    local l = sLower(name)
    for _, k in ipairs(DodgeConfig.BallKeywords) do if sFind(l, k, 1, true) then return true end end
    return false
end

local function isPlayerModel(inst)
    local m = inst:FindFirstAncestorOfClass("Model")
    while m do if Players:GetPlayerFromCharacter(m) then return true end; m = m:FindFirstAncestorOfClass("Model") end
    return false
end

local function isSphereBall(part)
    if not part:IsA("Part") then return false end
    if part.Shape ~= Enum.PartType.Ball then return false end
    if part.Size.X < DodgeConfig.SphereMinSize then return false end
    if part.AssemblyLinearVelocity.Magnitude < DodgeConfig.SphereMinSpeed then return false end
    return not isPlayerModel(part)
end

local function collectFrom(container, parts, seen)
    if not container then return end
    for _, d in ipairs(container:GetDescendants()) do
        if d:IsA("BasePart") and not seen[d] then seen[d] = true; tInsert(parts, d) end
    end
end

local function refreshBalls(now)
    if now - lastRefresh < DodgeConfig.RefreshInterval then return end
    lastRefresh = now; local rescan = false
    if not BallsContainer or not BallsContainer.Parent then BallsContainer = Workspace:FindFirstChild(DodgeConfig.BallsFolderName); rescan = true end
    if not PracticeContainer or not PracticeContainer.Parent then PracticeContainer = resolvePath(Workspace, DodgeConfig.PracticePath); rescan = true end
    if not BallsFolder3 or not BallsFolder3.Parent then local c = Workspace:GetChildren(); if c[3] then BallsFolder3 = c[3]; rescan = true else BallsFolder3 = nil end end
    if rescan then DodgeUpdateFilter() end
    local parts, seen = {}, {}
    if BallsContainer then collectFrom(BallsContainer, parts, seen) end
    if PracticeContainer then collectFrom(PracticeContainer, parts, seen) end
    if BallsFolder3 then
        for _, d in ipairs(BallsFolder3:GetDescendants()) do
            if d:IsA("BasePart") and not seen[d] and not isPlayerModel(d) then seen[d] = true; tInsert(parts, d) end
        end
        if BallsFolder3:IsA("BasePart") and not seen[BallsFolder3] and not isPlayerModel(BallsFolder3) then seen[BallsFolder3] = true; tInsert(parts, BallsFolder3) end
    end
    for _, child in ipairs(Workspace:GetChildren()) do
        if child ~= BallsContainer and child ~= PracticeContainer and child ~= BallsFolder3 then
            if not Players:GetPlayerFromCharacter(child) then
                if child:IsA("BasePart") and not seen[child] and (matchesKeyword(child.Name) or isSphereBall(child)) then seen[child] = true; tInsert(parts, child)
                elseif child:IsA("Folder") or child:IsA("Model") then
                    if matchesKeyword(child.Name) then collectFrom(child, parts, seen)
                    else for _, sub in ipairs(child:GetChildren()) do if sub:IsA("BasePart") and not seen[sub] and (matchesKeyword(sub.Name) or isSphereBall(sub)) then seen[sub] = true; tInsert(parts, sub) end end end
                end
            end
        end
    end
    for i = #BallCache, 1, -1 do BallCache[i] = nil end
    for i, part in ipairs(parts) do if i > DodgeConfig.MaxBalls then break end; tInsert(BallCache, part) end
    for part in pairs(BallSample) do if not part.Parent then BallSample[part] = nil; BallVelocity[part] = nil end end
end

local function velocityOf(part, pos, now)
    local vel = part.AssemblyLinearVelocity
    if vel.Magnitude > 1 then BallVelocity[part] = vel; BallSample[part] = {pos, now}; return vel end
    local s = BallSample[part]
    if s then local dt = now - s[2]; if dt > 0.05 then local est = (pos - s[1]) / dt; BallVelocity[part] = est; BallSample[part] = {pos, now}; return est end; return BallVelocity[part] or vel end
    BallSample[part] = {pos, now}; return BallVelocity[part] or vel
end

local function boxGap(lp, r)
    local dx = mAbs(lp.X) - (DodgeConfig.BoxWidth * 0.5 + r)
    local dy = mAbs(lp.Y - DodgeConfig.BoxOffsetY) - (DodgeConfig.BoxHeight * 0.5 + r)
    local dz = mAbs(lp.Z) - (DodgeConfig.BoxDepth * 0.5 + r)
    return mMax(dx, mMax(dy, dz))
end

local function clearanceAlong(origin, dir)
    local probe = Workspace:Raycast(origin, dir * DodgeConfig.ClearanceProbe, dodgeRay)
    if not probe then return DodgeConfig.ClearanceProbe end
    return probe.Distance
end

local function chooseSide(rootCF, lateralWorld, offsetAlongLateral, now)
    local preferred
    if mAbs(offsetAlongLateral) > DodgeConfig.CenterDeadzone then preferred = offsetAlongLateral >= 0 and -1 or 1 else preferred = sideSign end
    if now - sideChosenAt < DodgeConfig.SideCommitTime then return sideSign end
    local origin = rootCF.Position
    local room = clearanceAlong(origin, lateralWorld * preferred)
    if room < DodgeConfig.ClearanceMin then
        local otherRoom = clearanceAlong(origin, lateralWorld * -preferred)
        if otherRoom > room then preferred = -preferred end
    end
    if preferred ~= sideSign then sideSign = preferred; sideChosenAt = now end
    return sideSign
end

local function boxExtentAlong(axis)
    return DodgeConfig.BoxWidth * 0.5 * mAbs(axis.X) + DodgeConfig.BoxHeight * 0.5 * mAbs(axis.Y) + DodgeConfig.BoxDepth * 0.5 * mAbs(axis.Z)
end

local function staticPushout(lp, r)
    local hx = DodgeConfig.BoxWidth * 0.5 + r + DodgeConfig.ExtraMargin
    local hy = DodgeConfig.BoxHeight * 0.5 + r + DodgeConfig.ExtraMargin
    local hz = DodgeConfig.BoxDepth * 0.5 + r + DodgeConfig.ExtraMargin
    local oy = lp.Y - DodgeConfig.BoxOffsetY
    local px, py, pz = hx - mAbs(lp.X), hy - mAbs(oy), hz - mAbs(lp.Z)
    if px <= 0 or py <= 0 or pz <= 0 then return nil end
    local ax, dp = "X", px
    if pz < dp then ax, dp = "Z", pz end
    if DodgeConfig.Vertical and py * (1 / DodgeConfig.VerticalDamp) < dp then ax, dp = "Y", py end
    if ax == "X" then return lp.X >= 0 and v3new(-1, 0, 0) or v3new(1, 0, 0), dp
    elseif ax == "Z" then return lp.Z >= 0 and v3new(0, 0, -1) or v3new(0, 0, 1), dp end
    return oy >= 0 and v3new(0, -1, 0) or v3new(0, 1, 0), dp
end

local function safeOffset(origin, dir, dist)
    local res = Workspace:Raycast(origin, dir * (dist + DodgeConfig.WallPadding), dodgeRay)
    if not res then return dist end
    local allowed = res.Distance - DodgeConfig.WallPadding
    if allowed <= 0.05 then return 0 end
    return mMin(dist, allowed)
end

local function stripDrag(push)
    if not DodgeConfig.NoDrag then return push end
    for i = 1, #BlockedAxes do local ax = BlockedAxes[i]; local al = push:Dot(ax); if al > 0 then push = push - ax * al end end
    return push
end

local function applyPush(push, urgent, now)
    push = stripDrag(push); local len = push.Magnitude
    if len < DodgeConfig.MinStep then return false end
    local dir = push.Unit
    if urgent then len = mMin(len * DodgeConfig.UrgentSmoothing, DodgeConfig.UrgentStep)
    else len = mMin(len * DodgeConfig.Smoothing, DodgeConfig.MaxStep) end
    if len < DodgeConfig.MinStep then return false end
    local origin = DodgeRoot.Position; local allowed = safeOffset(origin, dir, len)
    if allowed <= 0.02 then
        local side = v3new(-dir.Z, 0, dir.X)
        if side.Magnitude < 0.05 then return false end
        side = stripDrag(side.Unit)
        if side.Magnitude < 0.05 then return false end
        side = side.Unit; local sideAllowed = safeOffset(origin, side, len)
        if sideAllowed <= 0.02 then
            local flipped = stripDrag(-side)
            if flipped.Magnitude < 0.05 then return false end
            side = flipped.Unit; sideAllowed = safeOffset(origin, side, len)
        end
        if sideAllowed <= 0.02 then return false end
        sideSign = -sideSign; sideChosenAt = now; dir = side; allowed = sideAllowed
    end
    local offset = dir * allowed
    if mAbs(offset.Y) < 0.05 then
        local ground = Workspace:Raycast(DodgeRoot.Position + offset + v3new(0, 5, 0), v3new(0, -DodgeConfig.GroundDrop, 0), dodgeRay)
        if not ground then return false end
    end
    local vel = DodgeRoot.AssemblyLinearVelocity
    DodgeRoot.CFrame = DodgeRoot.CFrame + offset
    if DodgeConfig.KeepVelocity then DodgeRoot.AssemblyLinearVelocity = vel end
    return true
end

local function resolveOnce(now, dt)
    if not DodgeRoot or not DodgeRoot.Parent or not DodgeHum or DodgeHum.Health <= 0 then return false end
    local rootCF = DodgeRoot.CFrame; local origin = rootCF.Position
    local ownVel = DodgeRoot.AssemblyLinearVelocity
    local sweepHorizon = dt + (DodgeConfig.Predictive and DodgeConfig.PredictTime or 0)
    local leadHorizon = sweepHorizon + DodgeConfig.LeadTime
    local push, urgent, count = v3zero, false, 0
    for i = #BlockedAxes, 1, -1 do BlockedAxes[i] = nil end
    for _, part in ipairs(BallCache) do
        if part and part.Parent then
            local pos = part.Position
            if (pos - origin).Magnitude < DodgeConfig.ScanRadius then
                local r = part.Size.X * 0.5; local vel = velocityOf(part, pos, now)
                local rel = vel - ownVel; local speed = rel.Magnitude
                local travel = rel * leadHorizon; local steps = 1; local span = travel.Magnitude
                if span > DodgeConfig.SubstepStud then steps = mClamp(mCeil(span / DodgeConfig.SubstepStud), 1, DodgeConfig.MaxSubsteps) end
                local threatened, threatTime = false, mHuge
                for si = 0, steps do
                    local frac = si / steps; local sp = pos + travel * frac; local lp = rootCF:PointToObjectSpace(sp)
                    if boxGap(lp, r) < DodgeConfig.ExtraMargin then threatened = true; threatTime = frac * leadHorizon; break end
                end
                if threatened then
                    local escape = nil
                    if speed > DodgeConfig.SlowSpeed then
                        local approach = rel.Unit; tInsert(BlockedAxes, approach)
                        local lateral = v3new(approach.Z, 0, -approach.X)
                        if lateral.Magnitude > 0.05 then
                            lateral = lateral.Unit; local offVec = pos - origin; local sep = offVec:Dot(lateral)
                            local sign = chooseSide(rootCF, lateral, sep, now)
                            local localLat = rootCF:VectorToObjectSpace(lateral)
                            local corridor = boxExtentAlong(localLat) + r + DodgeConfig.ExtraMargin + DodgeConfig.ClearMargin
                            local needed = corridor - mAbs(sep)
                            if sep * sign > 0 then needed = corridor + mAbs(sep) end
                            if needed > 0 then escape = lateral * sign * needed end
                        end
                    end
                    if not escape then
                        local lp = rootCF:PointToObjectSpace(pos); local direction, depth = staticPushout(lp, r)
                        if direction then escape = rootCF:VectorToWorldSpace(direction) * depth end
                    end
                    if escape then
                        if threatTime <= DodgeConfig.UrgentTime then urgent = true end
                        push = push + escape; count = count + 1
                    end
                end
            end
        end
    end
    if count == 0 then return false end
    lastEngaged = now; push = v3new(push.X, push.Y * DodgeConfig.VerticalDamp, push.Z)
    return applyPush(push, urgent, now)
end

local function gojoStep(dt)
    if not States.AutoDodge then return end
    local now = tick(); refreshBalls(now)
    if not DodgeRoot or not DodgeRoot.Parent or not DodgeHum or DodgeHum.Health <= 0 then return end
    if now - lastEngaged > 1 then sideChosenAt = 0 end
    for _ = 1, 3 do local ok, res = pcall(resolveOnce, now, dt); if not ok or not res then break end end
end

local function AutoDodge_Toggle(on)
    if not on then
        if DodgeConn then pcall(function() DodgeConn:Disconnect() end); DodgeConn = nil end
        return true
    end
    if DodgeConn then pcall(function() DodgeConn:Disconnect() end) end
    DodgeConn = RunService.Heartbeat:Connect(function(dt) local ok, err = pcall(gojoStep, dt); if not ok then warn("Dodge err: "..tostring(err)) end end)
    StoreConn(DodgeConn)
    return true
end

-- ── Anti Abilities (PAID) ──
local AntiConns = {}
local LastWS, LastJP = 16, 50

local function RemoveRagdoll(char)
    if not char then return end
    for _, obj in ipairs(char:GetDescendants()) do
        if obj:IsA("BallSocketConstraint") and obj.Name == "RagdollBallSocket" then pcall(function() obj:Destroy() end) end
        if obj:IsA("NoCollisionConstraint") and obj.Name == "RagdollNoCollision" then pcall(function() obj:Destroy() end) end
        if obj:IsA("AnimationConstraint") and obj.Name ~= "Root" then pcall(function() obj:Destroy() end) end
        if obj:IsA("BodyVelocity") or obj:IsA("BodyGyro") or obj:IsA("BodyPosition") then pcall(function() obj:Destroy() end) end
    end
end

local function ReenableMotors(char)
    if not char then return end
    for _, obj in ipairs(char:GetDescendants()) do
        if obj:IsA("Motor6D") and obj.Name ~= "Root" and obj.Enabled == false then pcall(function() obj.Enabled = true end) end
    end
end

local function SetupAntiAbilities(char)
    if not char then return end
    for _, c in ipairs(AntiConns) do pcall(function() c:Disconnect() end) end
    AntiConns = {}
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not hum then return end
    LastWS = hum.WalkSpeed > 0 and hum.WalkSpeed or 16
    LastJP = hum.JumpPower > 0 and hum.JumpPower or 50

    local sc = hum.StateChanged:Connect(function(_, ns)
        if not States.AntiAbilities then return end
        if ns == Enum.HumanoidStateType.Ragdoll or ns == Enum.HumanoidStateType.FallingDown or ns == Enum.HumanoidStateType.Physics then
            hum:ChangeState(Enum.HumanoidStateType.Running); RemoveRagdoll(char); ReenableMotors(char)
        end
    end); table.insert(AntiConns, sc)

    local pc = hum:GetPropertyChangedSignal("PlatformStand"):Connect(function()
        if not States.AntiAbilities then return end
        if hum.PlatformStand then hum.PlatformStand = false; hum:ChangeState(Enum.HumanoidStateType.GettingUp); RemoveRagdoll(char); ReenableMotors(char) end
    end); table.insert(AntiConns, pc)

    local sic = hum:GetPropertyChangedSignal("Sit"):Connect(function()
        if not States.AntiAbilities then return end
        if hum.Sit then hum.Sit = false; hum:ChangeState(Enum.HumanoidStateType.Running) end
    end); table.insert(AntiConns, sic)

    local wc = hum:GetPropertyChangedSignal("WalkSpeed"):Connect(function()
        if not States.AntiAbilities then return end
        if hum.WalkSpeed < 1 and hum.WalkSpeed > 0 then hum.WalkSpeed = LastWS elseif hum.WalkSpeed > 0 then LastWS = hum.WalkSpeed end
    end); table.insert(AntiConns, wc)

    local dc = char.DescendantAdded:Connect(function(desc)
        if not States.AntiAbilities then return end
        if desc:IsA("BallSocketConstraint") and desc.Name == "RagdollBallSocket" then task.delay(0.05, function() pcall(function() desc:Destroy() end) end) end
        if desc:IsA("AnimationConstraint") and desc.Name ~= "Root" then task.delay(0.05, function() pcall(function() desc:Destroy() end) end) end
    end); table.insert(AntiConns, dc)
end

local function AntiAbilities_Toggle(on)
    if not on then
        for _, c in ipairs(AntiConns) do pcall(function() c:Disconnect() end) end
        AntiConns = {}
        return true
    end
    if LocalPlayer.Character then SetupAntiAbilities(LocalPlayer.Character) end
    return true
end

LocalPlayer.CharacterAdded:Connect(function(char)
    task.wait(0.5)
    if States.AntiAbilities then SetupAntiAbilities(char) end
end)

-- ── ESP Ball (PAID) ──
local ESP_Labels = {}
local ESP_Conn = nil
local ESP_LastScan = 0

local function ESP_Clear()
    for part, data in pairs(ESP_Labels) do
        if data.Billboard then pcall(function() data.Billboard:Destroy() end) end
    end
    ESP_Labels = {}
    if ESP_Conn then pcall(function() ESP_Conn:Disconnect() end); ESP_Conn = nil end
    if getgenv().BabisESPFolder then pcall(function() getgenv().BabisESPFolder:Destroy() end); getgenv().BabisESPFolder = nil end
end

local function ESP_MakeLabel(part)
    if ESP_Labels[part] then return end
    if not getgenv().BabisESPFolder then return end
    local bb = Instance.new("BillboardGui")
    bb.Name = "ESP"
    bb.Adornee = part
    bb.Size = UDim2.new(0, 80, 0, 28)
    bb.StudsOffset = Vector3.new(0, 2, 0)
    bb.AlwaysOnTop = true
    bb.Parent = getgenv().BabisESPFolder

    local nl = Instance.new("TextLabel", bb)
    nl.Size = UDim2.new(1, 0, 0.55, 0)
    nl.Position = UDim2.new(0, 0, 0, 0)
    nl.BackgroundTransparency = 1
    nl.Text = "BALL"
    nl.TextColor3 = Color3.fromRGB(0, 200, 255)
    nl.TextStrokeColor3 = Color3.new(0, 0, 0)
    nl.TextStrokeTransparency = 0
    nl.Font = Enum.Font.Code
    nl.TextSize = 11

    local sl = Instance.new("TextLabel", bb)
    sl.Size = UDim2.new(1, 0, 0.45, 0)
    sl.Position = UDim2.new(0, 0, 0.55, 0)
    sl.BackgroundTransparency = 1
    sl.Text = "0 km/h"
    sl.TextColor3 = Color3.fromRGB(255, 220, 50)
    sl.TextStrokeColor3 = Color3.new(0, 0, 0)
    sl.TextStrokeTransparency = 0
    sl.Font = Enum.Font.Code
    sl.TextSize = 10

    ESP_Labels[part] = {Billboard = bb, SpeedLabel = sl}
end

local function ESP_Scan()
    local now = tick()
    if now - ESP_LastScan < 0.1 then return end
    ESP_LastScan = now

    local folder = Workspace:GetChildren()[3]
    if not folder then
        for part, data in pairs(ESP_Labels) do
            if data.Billboard then pcall(function() data.Billboard:Destroy() end) end
            ESP_Labels[part] = nil
        end
        return
    end

    local found = {}
    local count = 0
    for _, child in ipairs(folder:GetChildren()) do
        if child:IsA("BasePart") then
            found[child] = true; ESP_MakeLabel(child); count = count + 1
            if count > 50 then break end
        end
    end
    if count <= 50 then
        for _, desc in ipairs(folder:GetDescendants()) do
            if desc:IsA("BasePart") then
                found[desc] = true; ESP_MakeLabel(desc); count = count + 1
                if count > 50 then break end
            end
        end
    end

    for part, data in pairs(ESP_Labels) do
        if found[part] and part.Parent then
            local vel = part.AssemblyLinearVelocity.Magnitude
            data.SpeedLabel.Text = tostring(math.floor(vel * 3.6)) .. " km/h"
        else
            if data.Billboard then pcall(function() data.Billboard:Destroy() end) end
            ESP_Labels[part] = nil
        end
    end
end

local function ESPBall_Toggle(on)
    if not on then ESP_Clear(); return true end
    ESP_Clear()
    local f = Instance.new("Folder"); f.Name = "BabisESP"; f.Parent = Workspace; getgenv().BabisESPFolder = f
    ESP_Conn = RunService.Heartbeat:Connect(function() pcall(ESP_Scan) end)
    StoreConn(ESP_Conn)
    return true
end

-- ── Auto Play (PAID) ──
local AutoPlayConn = nil
local AutoPlayThread = nil
local AutoPlayEmergencyDest = nil
local AutoPlayRunning = false
local AutoPlayMapCenter = Vector3.new(0, 10, 0)
local AutoPlaySafeRadius = 60
local AutoPlayStuckTimeout = 8
local AutoPlayArriveDist = 4

local function AutoPlay_GetNil(name, class)
    if not getnilinstances then return nil end
    for _, v in next, getnilinstances() do
        if v.ClassName == class and v.Name == name then return v end
    end
    return nil
end

local function AutoPlay_GetTargetBalls()
    local targets = {}
    local balls = Workspace:FindFirstChild("Balls")
    if balls then
        local names = {"Big Ball", "Speed Ball", "Death Ball", "Duo Ball", "Destruction Ball", "Invis Ball"}
        for _, name in ipairs(names) do
            local b = balls:FindFirstChild(name)
            if b and b:IsA("BasePart") then table.insert(targets, b) end
        end
        local children = balls:GetChildren()
        if children[2] and children[2]:IsA("BasePart") then table.insert(targets, children[2]) end
        if children[3] and children[3]:IsA("BasePart") then table.insert(targets, children[3]) end
        if children[4] and children[4]:IsA("BasePart") then table.insert(targets, children[4]) end
    end
    local nilNames = {"Virus Ball", "Invis Ball", "Destruction Ball", "Speed Ball"}
    for _, name in ipairs(nilNames) do
        local b = AutoPlay_GetNil(name, "Part")
        if b then table.insert(targets, b) end
    end
    return targets
end

local function AutoPlay_CheckClear(pos, dir, dist)
    local r = Workspace:Raycast(pos + Vector3.new(0, 2, 0), dir * dist)
    if r then return r.Distance end
    return dist
end

local function AutoPlay_GetEscape(threatPos, threatVel, playerPos)
    local toPlayer = playerPos - threatPos
    local threatDir = threatVel.Unit
    local proj = toPlayer:Dot(threatDir)
    if proj < 0 then return nil end
    local closest = threatPos + threatDir * proj
    local distToPath = (playerPos - closest).Magnitude
    local r = 8
    if distToPath > r then return nil end
    local timeToHit = proj / threatVel.Magnitude
    if timeToHit > 1.5 or timeToHit < 0 then return nil end

    local right = Vector3.new(-threatDir.Z, 0, threatDir.X).Unit
    local left = -right
    local back = -threatDir
    local forward = threatDir
    local dirs = {right, left, back, forward}

    local bestDir = nil
    local bestClear = 0
    for i, d in ipairs(dirs) do
        local clear = AutoPlay_CheckClear(playerPos, d, 25)
        if clear > bestClear then
            bestClear = clear
            bestDir = d
        end
    end
    if not bestDir then return nil end
    return playerPos + bestDir * 12
end

local function AutoPlay_IsGround(pos)
    local r = Workspace:Raycast(pos + Vector3.new(0, 5, 0), Vector3.new(0, -25, 0))
    return r ~= nil
end

local function AutoPlay_RandomPointNear(origin)
    for i = 1, 6 do
        local angle = math.random() * math.pi * 2
        local distance = math.random(10, AutoPlaySafeRadius)
        local offset = Vector3.new(math.cos(angle) * distance, 0, math.sin(angle) * distance)
        local p = origin + offset
        if AutoPlay_IsGround(p) then return p end
    end
    return AutoPlayMapCenter + Vector3.new(0, 10, 0)
end

local function AutoPlay_WalkTo(humanoid, rootPart, destination)
    local path = PathfindingService:CreatePath({
        AgentRadius = 2,
        AgentHeight = 5,
        AgentCanJump = true,
    })

    local success = pcall(function()
        path:ComputeAsync(rootPart.Position, destination)
    end)

    if not success or path.Status ~= Enum.PathStatus.Success then
        humanoid:MoveTo(destination)
        local moveFinished = false
        local conn
        conn = humanoid.MoveToFinished:Connect(function() moveFinished = true end)
        local startTime = tick()
        while not moveFinished and AutoPlayRunning do
            if tick() - startTime > AutoPlayStuckTimeout then break end
            if AutoPlayEmergencyDest then break end
            task.wait(0.2)
        end
        conn:Disconnect()
        return
    end

    local waypoints = path:GetWaypoints()
    for _, waypoint in ipairs(waypoints) do
        if not AutoPlayRunning then return end
        if AutoPlayEmergencyDest then return end

        if waypoint.Action == Enum.PathWaypointAction.Jump then
            humanoid.Jump = true
        end

        humanoid:MoveTo(waypoint.Position)
        local moveFinished = false
        local conn
        conn = humanoid.MoveToFinished:Connect(function() moveFinished = true end)
        local startTime = tick()
        while not moveFinished and AutoPlayRunning do
            if tick() - startTime > AutoPlayStuckTimeout then break end
            if AutoPlayEmergencyDest then break end
            if (rootPart.Position - waypoint.Position).Magnitude <= AutoPlayArriveDist then break end
            task.wait(0.1)
        end
        conn:Disconnect()
    end
end

local function AutoPlay_Heartbeat()
    if not AutoPlayRunning then return end
    local char = LocalPlayer.Character
    if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart")
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not root or not hum or hum.Health <= 0 then return end

    local rootPos = root.Position
    local fall = Workspace:Raycast(rootPos + Vector3.new(0, 3, 0), Vector3.new(0, -60, 0))
    if not fall or fall.Distance > 35 then
        root.CFrame = CFrame.new(AutoPlayMapCenter + Vector3.new(0, 10, 0))
        AutoPlayEmergencyDest = AutoPlay_RandomPointNear(root.Position)
        if hum then hum:MoveTo(AutoPlayEmergencyDest) end
        return
    end

    local targets = AutoPlay_GetTargetBalls()
    local bestEscape = nil
    local minDist = math.huge
    for _, part in ipairs(targets) do
        if part and (part.Parent or AutoPlay_GetNil(part.Name, "Part")) then
            local vel = part.AssemblyLinearVelocity
            if vel.Magnitude > 2 then
                local esc = AutoPlay_GetEscape(part.Position, vel, rootPos)
                if esc then
                    local d = (part.Position - rootPos).Magnitude
                    if d < minDist then
                        minDist = d
                        bestEscape = esc
                    end
                end
            end
        end
    end
    if bestEscape then
        AutoPlayEmergencyDest = bestEscape
        hum:MoveTo(AutoPlayEmergencyDest)
    end
end

local function AutoPlay_WanderLoop()
    while AutoPlayRunning do
        local char = LocalPlayer.Character
        if not char then
            task.wait(1)
            continue
        end
        local hum = char:FindFirstChildOfClass("Humanoid")
        local root = char:FindFirstChild("HumanoidRootPart")
        if not hum or not root or hum.Health <= 0 then
            task.wait(1)
            continue
        end

        if AutoPlayEmergencyDest then
            local dest = AutoPlayEmergencyDest
            AutoPlayEmergencyDest = nil
            hum:MoveTo(dest)
            local moveFinished = false
            local conn
            conn = hum.MoveToFinished:Connect(function() moveFinished = true end)
            local startTime = tick()
            while not moveFinished and AutoPlayRunning do
                if tick() - startTime > AutoPlayStuckTimeout then break end
                if AutoPlayEmergencyDest then break end
                task.wait(0.15)
            end
            conn:Disconnect()
        else
            local dest = AutoPlay_RandomPointNear(root.Position)
            pcall(function() AutoPlay_WalkTo(hum, root, dest) end)
        end
    end
end

local function AutoPlay_Toggle(on)
    if not on then
        AutoPlayRunning = false
        if AutoPlayConn then pcall(function() AutoPlayConn:Disconnect() end); AutoPlayConn = nil end
        if AutoPlayThread then AutoPlayThread = nil end
        local char = LocalPlayer.Character
        if char then
            local hum = char:FindFirstChildOfClass("Humanoid")
            if hum then hum:MoveTo(Vector3.zero) end
        end
        return true
    end
    local char = LocalPlayer.Character
    if not char then States.AutoPlay = false; return false end
    local hum = char:FindFirstChildOfClass("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")
    if not hum or not root then States.AutoPlay = false; return false end
    AutoPlayRunning = true
    AutoPlayConn = RunService.Heartbeat:Connect(function() pcall(AutoPlay_Heartbeat) end)
    StoreConn(AutoPlayConn)
    AutoPlayThread = task.spawn(AutoPlay_WanderLoop)
    return true
end

-- ── Redeem Codes (FREE) ──
local Codes = {"DODGE","250K","1KCCU"}
local IsRedeeming = false

local function RedeemAllCodes()
    if IsRedeeming then return false, "busy" end
    IsRedeeming = true
    local ok, safeNet = pcall(function() return require(ReplicatedStorage:WaitForChild("Packages",5):WaitForChild("safe-net",5)) end)
    if not ok or not safeNet then IsRedeeming = false; return false, "safe-net not found" end
    local remote = safeNet.get("RedeemCode")
    if not remote then IsRedeeming = false; return false, "remote not found" end
    for _, code in ipairs(Codes) do pcall(function() remote:InvokeServer(code) end); task.wait(1) end
    IsRedeeming = false
    return true
end

-- ============================================================
-- MAIN UI — Babis | Dodge Or Die
-- ============================================================
local function buildMainUI()
    local notify = createNotifier()

    local UI = {
        activeTab = CONFIG.defaultTab,
        connections = {},
        isMinimized = false,
        isOpen = false,
        destroyed = false,
        normalPos = CONFIG.normalPos,
        minimizedPosition = CONFIG.minimizedDefaultPos,
        options = {},
        openDropdowns = {},
        cfgSelected = nil,
        cfgList = {},
        rebuildTab = nil,
        infoOpen = false,
        listeningKey = false,
        toggleKeyConn = nil,
    }

    UI.options["toggle_key"]      = UI.options["toggle_key"] or "None"
    UI.options["opt_autododge"]   = false
    UI.options["opt_anti_kick"]   = true
    UI.options["opt_autowin"]     = false
    UI.options["opt_autoplay"]    = false
    UI.options["opt_antiable"]    = false
    UI.options["opt_espball"]     = false

    local function track(conn)
        table.insert(UI.connections, conn)
        return conn
    end

    local function cleanup()
        UI.destroyed = true
        for _, c in ipairs(UI.connections) do
            if c and c.Disconnect then c:Disconnect() end
        end
        UI.connections = {}
        if UI.toggleKeyConn then pcall(function() UI.toggleKeyConn:Disconnect() end) end
        UI.toggleKeyConn = nil
    end

    local screenGui = Instance.new("ScreenGui")
    screenGui.Name           = "EssenceUI"
    screenGui.ResetOnSpawn   = false
    screenGui.IgnoreGuiInset = true
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.DisplayOrder   = 100
    screenGui.Parent         = PlayerGui
    getgenv().BabisHub       = screenGui

    local scaleWrapper = Instance.new("Frame")
    scaleWrapper.BackgroundTransparency = 1
    scaleWrapper.Size        = UDim2.new(0, CONFIG.sizes.windowWidth, 0, CONFIG.sizes.windowHeight)
    scaleWrapper.Position    = UI.normalPos
    scaleWrapper.AnchorPoint = Vector2.new(0.5, 0.5)
    scaleWrapper.ClipsDescendants = true
    scaleWrapper.Parent      = screenGui

    local uiScale = Instance.new("UIScale")
    uiScale.Parent = scaleWrapper

    local mainWindow = Instance.new("Frame")
    mainWindow.Size = UDim2.new(1, 0, 1, 0)
    mainWindow.BackgroundColor3 = CONFIG.colors.windowBg
    mainWindow.BorderSizePixel = 0
    mainWindow.ClipsDescendants = true
    mainWindow.Parent = scaleWrapper
    corner(mainWindow, CONFIG.sizes.windowRadius)

    local function clampToViewport(px, py)
        local cam = workspace.CurrentCamera
        if not cam then return px, py end
        local vp = cam.ViewportSize
        local s = uiScale.Scale or 1
        local w = scaleWrapper.Size.X.Offset * s
        local h = scaleWrapper.Size.Y.Offset * s
        local halfW, halfH = w / 2, h / 2
        local minX, maxX = halfW, vp.X - halfW
        local minY, maxY = halfH, vp.Y - halfH
        if maxX < minX then px = vp.X / 2 else px = math.clamp(px, minX, maxX) end
        if maxY < minY then py = vp.Y / 2 else py = math.clamp(py, minY, maxY) end
        return px, py
    end

    local function posToPixels(pos)
        local cam = workspace.CurrentCamera
        local vp = cam and cam.ViewportSize or Vector2.new(1920, 1080)
        return pos.X.Scale * vp.X + pos.X.Offset, pos.Y.Scale * vp.Y + pos.Y.Offset
    end

    -- HEADER
    local header = Instance.new("Frame")
    header.Size = UDim2.new(1, 0, 0, CONFIG.sizes.headerHeight)
    header.BackgroundColor3 = CONFIG.colors.headerTint
    header.BorderSizePixel = 0
    header.Parent = mainWindow
    corner(header, CONFIG.sizes.windowRadius)

    local headerFlat = Instance.new("Frame")
    headerFlat.Size = UDim2.new(1, 0, 0, 16)
    headerFlat.Position = UDim2.new(0, 0, 1, -16)
    headerFlat.BackgroundColor3 = CONFIG.colors.headerTint
    headerFlat.BorderSizePixel = 0
    headerFlat.Parent = header

    local titleRow = Instance.new("Frame")
    titleRow.Size = UDim2.new(1, -260, 0, 34)
    titleRow.Position = UDim2.new(0, 22, 0, 8)
    titleRow.BackgroundTransparency = 1
    titleRow.Parent = header

    local appLabel = Instance.new("TextLabel")
    appLabel.BackgroundTransparency = 1
    appLabel.Text = CONFIG.appName
    appLabel.TextColor3 = CONFIG.colors.accent
    appLabel.Font = CONFIG.fonts.appName
    appLabel.TextSize = CONFIG.textSizes.appName
    appLabel.TextXAlignment = Enum.TextXAlignment.Left
    appLabel.TextYAlignment = Enum.TextYAlignment.Center
    appLabel.Size = UDim2.new(0, 380, 1, 0)
    appLabel.Parent = titleRow

    local subtitleLabel = Instance.new("TextLabel")
    subtitleLabel.BackgroundTransparency = 1
    subtitleLabel.Text = CONFIG.edition
    subtitleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    subtitleLabel.Font = Enum.Font.GothamBold
    subtitleLabel.TextSize = 14
    subtitleLabel.TextXAlignment = Enum.TextXAlignment.Left
    subtitleLabel.TextYAlignment = Enum.TextYAlignment.Center
    subtitleLabel.Size = UDim2.new(0, 220, 0, 18)
    subtitleLabel.Position = UDim2.new(0, 24, 0, 42)
    subtitleLabel.Parent = header

    local versionLabel = Instance.new("TextLabel")
    versionLabel.BackgroundTransparency = 1
    versionLabel.Text = CONFIG.version
    versionLabel.TextColor3 = CONFIG.colors.textMuted
    versionLabel.Font = CONFIG.fonts.version
    versionLabel.TextSize = 13
    versionLabel.TextXAlignment = Enum.TextXAlignment.Left
    versionLabel.TextYAlignment = Enum.TextYAlignment.Center
    versionLabel.Size = UDim2.new(0, 200, 0, 16)
    versionLabel.Position = UDim2.new(0, 24, 0, 60)
    versionLabel.Parent = header

    -- INFO
    local infoBtn = Instance.new("TextButton")
    infoBtn.Size = UDim2.new(0, 60, 0, 60)
    infoBtn.Position = UDim2.new(1, -182, 0, 12)
    infoBtn.BackgroundTransparency = 1
    infoBtn.Text = ""
    infoBtn.AutoButtonColor = false
    infoBtn.Parent = header

    local infoIcon = Instance.new("ImageLabel")
    infoIcon.Size = UDim2.new(0, 30, 0, 30)
    infoIcon.Position = UDim2.new(0.5, -15, 0.5, -15)
    infoIcon.BackgroundTransparency = 1
    infoIcon.Image = CONFIG.infoIconId
    infoIcon.ImageColor3 = CONFIG.colors.navIcon
    infoIcon.ScaleType = Enum.ScaleType.Fit
    infoIcon.Parent = infoBtn

    -- MINIMIZE
    local minBtn = Instance.new("TextButton")
    minBtn.Size = UDim2.new(0, 60, 0, 60)
    minBtn.Position = UDim2.new(1, -122, 0, 12)
    minBtn.BackgroundTransparency = 1
    minBtn.Text = ""
    minBtn.AutoButtonColor = false
    minBtn.Parent = header

    local minGlyph = Instance.new("Frame")
    minGlyph.Size = UDim2.new(0, 22, 0, 3)
    minGlyph.Position = UDim2.new(0.5, -11, 0.5, -1)
    minGlyph.BackgroundColor3 = CONFIG.colors.textPrimary
    minGlyph.BorderSizePixel = 0
    minGlyph.Parent = minBtn
    corner(minGlyph, 2)

    -- CLOSE
    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 60, 0, 60)
    closeBtn.Position = UDim2.new(1, -62, 0, 12)
    closeBtn.BackgroundTransparency = 1
    closeBtn.Text = ""
    closeBtn.AutoButtonColor = false
    closeBtn.Parent = header

    local xLen, xThk = 26, 3
    local xBar1 = Instance.new("Frame")
    xBar1.Size = UDim2.new(0, xLen, 0, xThk)
    xBar1.Position = UDim2.new(0.5, -xLen/2, 0.5, -xThk/2)
    xBar1.BackgroundColor3 = CONFIG.colors.textPrimary
    xBar1.BorderSizePixel = 0
    xBar1.Rotation = 45
    xBar1.Parent = closeBtn
    corner(xBar1, 2)

    local xBar2 = Instance.new("Frame")
    xBar2.Size = UDim2.new(0, xLen, 0, xThk)
    xBar2.Position = UDim2.new(0.5, -xLen/2, 0.5, -xThk/2)
    xBar2.BackgroundColor3 = CONFIG.colors.textPrimary
    xBar2.BorderSizePixel = 0
    xBar2.Rotation = -45
    xBar2.Parent = closeBtn
    corner(xBar2, 2)

    local divider = Instance.new("Frame")
    divider.Size = UDim2.new(1, 0, 0, 2)
    divider.Position = UDim2.new(0, 0, 0, CONFIG.sizes.headerHeight)
    divider.BackgroundColor3 = CONFIG.colors.divider
    divider.BorderSizePixel = 0
    divider.ZIndex = 2
    divider.Parent = mainWindow

    local divGrad = Instance.new("UIGradient")
    divGrad.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.85),
        NumberSequenceKeypoint.new(0.5, 0),
        NumberSequenceKeypoint.new(1, 0.85),
    })
    divGrad.Parent = divider

    -- NAV
    local navBar = Instance.new("Frame")
    navBar.Size = UDim2.new(1, -CONFIG.sizes.navPadX*2, 0, CONFIG.sizes.navHeight)
    navBar.Position = UDim2.new(0, CONFIG.sizes.navPadX, 0, CONFIG.sizes.headerHeight + 12)
    navBar.BackgroundTransparency = 1
    navBar.Parent = mainWindow

    local navLayout = Instance.new("UIListLayout")
    navLayout.FillDirection = Enum.FillDirection.Horizontal
    navLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    navLayout.VerticalAlignment = Enum.VerticalAlignment.Center
    navLayout.Padding = UDim.new(0, CONFIG.sizes.navGap)
    navLayout.SortOrder = Enum.SortOrder.LayoutOrder
    navLayout.Parent = navBar

    local tabButtons = {}

    local function createTabButton(tabData, order)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0, CONFIG.sizes.navBtnSize, 0, CONFIG.sizes.navBtnSize)
        btn.BackgroundColor3 = CONFIG.colors.navBg
        btn.Text = ""
        btn.AutoButtonColor = false
        btn.LayoutOrder = order
        btn.Parent = navBar
        corner(btn, CONFIG.sizes.navBtnRadius)
        local btnStroke = stroke(btn, CONFIG.colors.navBorder, 2, 0)

        local baseSize = CONFIG.sizes.iconSize + (tabData.boostAmt or 0)

        local iconGlow = Instance.new("ImageLabel")
        iconGlow.Size = UDim2.new(0, baseSize + 8, 0, baseSize + 8)
        iconGlow.Position = UDim2.new(0.5, -(baseSize+8)/2, 0.5, -(baseSize+8)/2)
        iconGlow.BackgroundTransparency = 1
        iconGlow.Image = "rbxassetid://" .. tabData.iconId
        iconGlow.ImageColor3 = CONFIG.colors.navIcon
        iconGlow.ImageTransparency = 0.85
        iconGlow.ScaleType = Enum.ScaleType.Fit
        iconGlow.Parent = btn

        local icon = Instance.new("ImageLabel")
        icon.Size = UDim2.new(0, baseSize, 0, baseSize)
        icon.Position = UDim2.new(0.5, -baseSize/2, 0.5, -baseSize/2)
        icon.BackgroundTransparency = 1
        icon.Image = "rbxassetid://" .. tabData.iconId
        icon.ImageColor3 = CONFIG.colors.navIcon
        icon.ScaleType = Enum.ScaleType.Fit
        icon.Parent = btn

        local hit = Instance.new("TextButton")
        hit.Size = UDim2.new(1, 0, 1, 1)
        hit.BackgroundTransparency = 1
        hit.Text = ""
        hit.AutoButtonColor = false
        hit.Parent = btn

        return { button = btn, stroke = btnStroke, icon = icon, iconGlow = iconGlow, hit = hit, id = tabData.id, baseSize = baseSize }
    end

    for i, tab in ipairs(CONFIG.tabs) do
        tabButtons[tab.id] = createTabButton(tab, i)
    end

    -- CONTENT
    local contentContainer = Instance.new("Frame")
    contentContainer.Size = UDim2.new(1, -CONFIG.sizes.contentPadX*2, 1, -(CONFIG.sizes.headerHeight + CONFIG.sizes.navHeight + 30))
    contentContainer.Position = UDim2.new(0, CONFIG.sizes.contentPadX, 0, CONFIG.sizes.headerHeight + CONFIG.sizes.navHeight + 20)
    contentContainer.BackgroundTransparency = 1
    contentContainer.Parent = mainWindow

    local scroll = Instance.new("ScrollingFrame")
    scroll.Size = UDim2.new(1, 0, 1, 0)
    scroll.BackgroundTransparency = 1
    scroll.BorderSizePixel = 0
    scroll.ScrollBarThickness = 4
    scroll.ScrollBarImageColor3 = CONFIG.colors.accent
    scroll.ScrollBarImageTransparency = 0.3
    scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
    scroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
    scroll.ClipsDescendants = true
    scroll.Parent = contentContainer

    local contentLayout = Instance.new("UIListLayout")
    contentLayout.FillDirection = Enum.FillDirection.Vertical
    contentLayout.Padding = UDim.new(0, CONFIG.sizes.cardGap)
    contentLayout.SortOrder = Enum.SortOrder.LayoutOrder
    contentLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    contentLayout.Parent = scroll

    local contentPad = Instance.new("UIPadding")
    contentPad.PaddingTop    = UDim.new(0, 6)
    contentPad.PaddingBottom = UDim.new(0, 16)
    contentPad.PaddingLeft   = UDim.new(0, CONFIG.sizes.scrollPadL)
    contentPad.PaddingRight  = UDim.new(0, CONFIG.sizes.scrollPadR)
    contentPad.Parent = scroll

    -- INFO FRAME
    local infoFrame = Instance.new("Frame")
    infoFrame.Name = "InfoFrame"
    infoFrame.Size = UDim2.new(1, -CONFIG.sizes.contentPadX*2, 1, -(CONFIG.sizes.headerHeight + 20))
    infoFrame.Position = UDim2.new(0, CONFIG.sizes.contentPadX, 0, CONFIG.sizes.headerHeight + 10)
    infoFrame.BackgroundColor3 = CONFIG.colors.cardBg
    infoFrame.BorderSizePixel = 0
    infoFrame.Visible = false
    infoFrame.BackgroundTransparency = 1
    infoFrame.Parent = mainWindow
    corner(infoFrame, CONFIG.sizes.cardRadius)
    local infoStroke = stroke(infoFrame, CONFIG.colors.cardBorder, 1.5, 1)

    local aboutTitle = Instance.new("TextLabel")
    aboutTitle.BackgroundTransparency = 1
    aboutTitle.Text = "About"
    aboutTitle.TextColor3 = CONFIG.colors.accent
    aboutTitle.Font = Enum.Font.GothamBold
    aboutTitle.TextSize = CONFIG.textSizes.aboutTitle
    aboutTitle.TextXAlignment = Enum.TextXAlignment.Left
    aboutTitle.TextYAlignment = Enum.TextYAlignment.Center
    aboutTitle.Size = UDim2.new(1, -32, 0, 40)
    aboutTitle.Position = UDim2.new(0, 18, 0, 20)
    aboutTitle.TextTransparency = 1
    aboutTitle.Parent = infoFrame

    local aboutDesc = Instance.new("TextLabel")
    aboutDesc.BackgroundTransparency = 1
    aboutDesc.Text = "Babis | Dodge Or Die\n\nAdapted build for Dodge Or Die. Free features are fully unlocked. Paid features (Auto Win, ESP Ball, Anti Abilities, Auto Play) are locked in this free edition.\n\nJoin the Discord below for support, updates and the premium build.\n\nThank you for being part of this."
    aboutDesc.TextColor3 = CONFIG.colors.textPrimary
    aboutDesc.Font = Enum.Font.Gotham
    aboutDesc.TextSize = CONFIG.textSizes.aboutDesc
    aboutDesc.TextXAlignment = Enum.TextXAlignment.Left
    aboutDesc.TextYAlignment = Enum.TextYAlignment.Top
    aboutDesc.TextWrapped = true
    aboutDesc.LineHeight = 1.2
    aboutDesc.Size = UDim2.new(1, -36, 1, -190)
    aboutDesc.Position = UDim2.new(0, 18, 0, 70)
    aboutDesc.TextTransparency = 1
    aboutDesc.Parent = infoFrame

    local discordBtn = Instance.new("TextButton")
    discordBtn.Size = UDim2.new(1, -36, 0, 108)
    discordBtn.Position = UDim2.new(0, 18, 1, -128)
    discordBtn.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    discordBtn.Text = ""
    discordBtn.AutoButtonColor = false
    discordBtn.BackgroundTransparency = 1
    discordBtn.Parent = infoFrame
    corner(discordBtn, 14)

    local dBtnGrad = Instance.new("Frame")
    dBtnGrad.Name = "GradientFill"
    dBtnGrad.Size = UDim2.new(1, 0, 1, 0)
    dBtnGrad.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    dBtnGrad.BorderSizePixel = 0
    dBtnGrad.ZIndex = 1
    dBtnGrad.Parent = discordBtn
    corner(dBtnGrad, 14)

    local dBtnGradFill = Instance.new("UIGradient")
    dBtnGradFill.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, CONFIG.colors.discordGradLeft),
        ColorSequenceKeypoint.new(1, CONFIG.colors.discordGradRight),
    })
    dBtnGradFill.Rotation = 0
    dBtnGradFill.Parent = dBtnGrad

    local dBtnStroke = stroke(discordBtn, CONFIG.colors.discordBorder, 1.5, 0.2)

    local dIcon = Instance.new("ImageLabel")
    dIcon.Size = UDim2.new(0, 76, 0, 76)
    dIcon.Position = UDim2.new(0, 16, 0.5, -38)
    dIcon.BackgroundTransparency = 1
    dIcon.Image = CONFIG.discordIconId
    dIcon.ImageColor3 = Color3.fromRGB(255, 255, 255)
    dIcon.ScaleType = Enum.ScaleType.Fit
    dIcon.ZIndex = 2
    dIcon.Parent = discordBtn

    local dTitle = Instance.new("TextLabel")
    dTitle.BackgroundTransparency = 1
    dTitle.Text = "Discord"
    dTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
    dTitle.Font = Enum.Font.GothamBold
    dTitle.TextSize = CONFIG.textSizes.discordTitle
    dTitle.TextXAlignment = Enum.TextXAlignment.Left
    dTitle.TextYAlignment = Enum.TextYAlignment.Center
    dTitle.Size = UDim2.new(1, -130, 0, 32)
    dTitle.Position = UDim2.new(0, 108, 0, 22)
    dTitle.ZIndex = 2
    dTitle.Parent = discordBtn

    local dSub = Instance.new("TextLabel")
    dSub.BackgroundTransparency = 1
    dSub.Text = "Tap to join the Discord Server"
    dSub.TextColor3 = Color3.fromRGB(220, 225, 245)
    dSub.Font = Enum.Font.Gotham
    dSub.TextSize = CONFIG.textSizes.discordSub
    dSub.TextXAlignment = Enum.TextXAlignment.Left
    dSub.TextYAlignment = Enum.TextYAlignment.Center
    dSub.Size = UDim2.new(1, -130, 0, 26)
    dSub.Position = UDim2.new(0, 108, 0, 58)
    dSub.ZIndex = 2
    dSub.Parent = discordBtn

    local hoveredGrad = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(105, 118, 255)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(30, 34, 60)),
    })
    local normalGrad = ColorSequence.new({
        ColorSequenceKeypoint.new(0, CONFIG.colors.discordGradLeft),
        ColorSequenceKeypoint.new(1, CONFIG.colors.discordGradRight),
    })

    discordBtn.MouseEnter:Connect(function()
        tween(dBtnGradFill, { Color = hoveredGrad }, CONFIG.anim.fast)
        tween(dBtnStroke, { Transparency = 0 }, CONFIG.anim.fast)
    end)
    discordBtn.MouseLeave:Connect(function()
        tween(dBtnGradFill, { Color = normalGrad }, CONFIG.anim.fast)
        tween(dBtnStroke, { Transparency = 0.2 }, CONFIG.anim.fast)
    end)

    discordBtn.Activated:Connect(function()
        pcall(function() setclipboard(CONFIG.discordInvite) end)
        notify("Discord", "Invite copied: " .. CONFIG.discordInvite, "success")
    end)

    -- INFO OPEN / CLOSE
    local function closeInfo()
        if not UI.infoOpen then return end
        UI.infoOpen = false

        tween(aboutTitle, { TextTransparency = 1 }, 0.18)
        tween(aboutDesc,  { TextTransparency = 1 }, 0.18)
        tween(dTitle,     { TextTransparency = 1 }, 0.18)
        tween(dSub,       { TextTransparency = 1 }, 0.18)
        tween(dIcon,      { ImageTransparency = 1 }, 0.18)
        tween(dBtnGrad,   { BackgroundTransparency = 1 }, 0.18)
        tween(discordBtn, { BackgroundTransparency = 1 }, 0.18)
        tween(dBtnStroke, { Transparency = 1 }, 0.18)
        tween(infoFrame,  { BackgroundTransparency = 1 }, 0.22)
        tween(infoStroke, { Transparency = 1 }, 0.22)

        task.delay(0.24, function()
            infoFrame.Visible = false
            if UI.isMinimized then
                navBar.Visible = false
                contentContainer.Visible = false
            else
                navBar.Visible = true
                contentContainer.Visible = true
            end
            tween(infoIcon, { ImageColor3 = CONFIG.colors.navIcon }, CONFIG.anim.fast)
        end)
    end

    local function openInfo()
        if UI.infoOpen then
            closeInfo()
            return
        end
        if UI.isMinimized then
            UI.isMinimized = false
            local px, py = posToPixels(UI.normalPos)
            local cx, cy = clampToViewport(px, py)
            tween(scaleWrapper, {
                Size     = UDim2.new(0, CONFIG.sizes.windowWidth, 0, CONFIG.sizes.windowHeight),
                Position = UDim2.new(0, cx, 0, cy),
            }, CONFIG.anim.slow)
        end

        UI.infoOpen = true
        infoFrame.Visible = true
        infoFrame.BackgroundTransparency = 1
        infoStroke.Transparency = 1
        aboutTitle.TextTransparency = 1
        aboutDesc.TextTransparency = 1
        dTitle.TextTransparency = 1
        dSub.TextTransparency = 1
        dIcon.ImageTransparency = 1
        dBtnGrad.BackgroundTransparency = 1
        discordBtn.BackgroundTransparency = 1
        dBtnStroke.Transparency = 1

        navBar.Visible = false
        contentContainer.Visible = false

        tween(infoFrame,  { BackgroundTransparency = 0 }, 0.28)
        tween(infoStroke, { Transparency = 0 }, 0.28)
        tween(aboutTitle, { TextTransparency = 0 }, 0.30)
        tween(aboutDesc,  { TextTransparency = 0 }, 0.36)
        task.delay(0.10, function()
            tween(dBtnGrad,   { BackgroundTransparency = 0 }, 0.28)
            tween(discordBtn, { BackgroundTransparency = 0 }, 0.28)
            tween(dBtnStroke, { Transparency = 0.2 }, 0.28)
            tween(dIcon,      { ImageTransparency = 0 }, 0.30)
            tween(dTitle,     { TextTransparency = 0 }, 0.30)
            tween(dSub,       { TextTransparency = 0 }, 0.32)
        end)

        tween(infoIcon, { ImageColor3 = CONFIG.colors.navIconActive }, CONFIG.anim.fast)
    end

    infoBtn.Activated:Connect(openInfo)

    infoBtn.MouseEnter:Connect(function()
        if not UI.infoOpen then
            tween(infoIcon, { ImageColor3 = CONFIG.colors.accentSoft }, CONFIG.anim.fast)
        end
    end)
    infoBtn.MouseLeave:Connect(function()
        if not UI.infoOpen then
            tween(infoIcon, { ImageColor3 = CONFIG.colors.navIcon }, CONFIG.anim.fast)
        end
    end)

    -- CARD BASE (with locked variant)
    local function createCard(order, height, locked)
        local card = Instance.new("Frame")
        card.Size = UDim2.new(1, 0, 0, height or CONFIG.sizes.cardHeight)
        card.BackgroundColor3 = locked and CONFIG.colors.lockedCard or CONFIG.colors.cardBg
        card.BorderSizePixel = 0
        card.LayoutOrder = order
        card.Parent = scroll
        corner(card, CONFIG.sizes.cardRadius)
        local cardStroke = stroke(card, locked and CONFIG.colors.lockedBorder or CONFIG.colors.cardBorder, 1.5, 0)

        local title = Instance.new("TextLabel")
        title.BackgroundTransparency = 1
        title.TextColor3 = locked and CONFIG.colors.lockedText or CONFIG.colors.textPrimary
        title.Font = Enum.Font.GothamBold
        title.TextSize = CONFIG.textSizes.cardTitle
        title.TextXAlignment = Enum.TextXAlignment.Left
        title.TextYAlignment = Enum.TextYAlignment.Top
        title.Size = UDim2.new(1, -160, 0, 30)
        title.Position = UDim2.new(0, CONFIG.sizes.cardPadX, 0, 24)
        title.Parent = card

        -- premium tag (only for locked)
        if locked then
            local tag = Instance.new("TextLabel")
            tag.BackgroundTransparency = 1
            tag.Text = "PREMIUM"
            tag.TextColor3 = CONFIG.colors.premium
            tag.Font = Enum.Font.GothamBold
            tag.TextSize = 11
            tag.TextXAlignment = Enum.TextXAlignment.Right
            tag.TextYAlignment = Enum.TextYAlignment.Top
            tag.Size = UDim2.new(0, 90, 0, 20)
            tag.Position = UDim2.new(1, -100, 0, 26)
            tag.Parent = card
        end

        local desc = Instance.new("TextLabel")
        desc.BackgroundTransparency = 1
        desc.TextColor3 = locked and CONFIG.colors.lockedText or CONFIG.colors.textDesc
        desc.Font = Enum.Font.Gotham
        desc.TextSize = CONFIG.textSizes.cardDesc
        desc.TextXAlignment = Enum.TextXAlignment.Left
        desc.TextYAlignment = Enum.TextYAlignment.Top
        desc.Size = UDim2.new(1, -160, 0, 22)
        desc.Position = UDim2.new(0, CONFIG.sizes.cardPadX, 0, 68)
        desc.Parent = card

        return card, title, desc, cardStroke
    end

    -- TOGGLE (supports locked premium)
    local function createToggleCard(order, titleText, descText, key, default, onChange, locked)
        local card, title, desc = createCard(order, nil, locked)
        title.Text = titleText
        desc.Text  = descText

        if UI.options[key] == nil then
            UI.options[key] = default and true or false
        end

        local tW = CONFIG.sizes.toggleW
        local tH = CONFIG.sizes.toggleH
        local knob = CONFIG.sizes.toggleKnob
        local pad  = (tH - knob) / 2
        local right = CONFIG.sizes.toggleRight

        local track2 = Instance.new("Frame")
        track2.Size = UDim2.new(0, tW, 0, tH)
        track2.Position = UDim2.new(1, -(tW + right), 0.5, -tH/2)
        track2.BackgroundColor3 = locked and Color3.fromRGB(28, 28, 32) or CONFIG.colors.toggleOff
        track2.BorderSizePixel = 0
        track2.Parent = card
        corner(track2, tH / 2)
        local trackStroke = stroke(track2, locked and CONFIG.colors.lockedBorder or CONFIG.colors.cardBorder, 1.5, 0)

        local knobFrame = Instance.new("Frame")
        knobFrame.Size = UDim2.new(0, knob, 0, knob)
        knobFrame.Position = UDim2.new(0, pad, 0.5, -knob/2)
        knobFrame.BackgroundColor3 = locked and Color3.fromRGB(70, 70, 78) or CONFIG.colors.textPrimary
        knobFrame.BorderSizePixel = 0
        knobFrame.Parent = track2
        corner(knobFrame, knob / 2)

        -- lock icon overlay if locked
        if locked then
            local lock = Instance.new("ImageLabel")
            lock.Size = UDim2.new(0, 20, 0, 20)
            lock.Position = UDim2.new(0.5, -10, 0.5, -10)
            lock.BackgroundTransparency = 1
            lock.Image = CONFIG.lockIconId
            lock.ImageColor3 = CONFIG.colors.premium
            lock.ScaleType = Enum.ScaleType.Fit
            lock.ZIndex = 5
            lock.Parent = track2
        end

        local hit = Instance.new("TextButton")
        hit.Size = UDim2.new(1, 0, 1, 0)
        hit.BackgroundTransparency = 1
        hit.Text = ""
        hit.AutoButtonColor = false
        hit.ZIndex = 10
        hit.Parent = card

        local offX, onX = pad, tW - knob - pad

        local function render(animate)
            if locked then return end
            local dur = animate and CONFIG.anim.normal or 0
            if UI.options[key] then
                tween(track2, { BackgroundColor3 = CONFIG.colors.toggleOn }, dur)
                tween(trackStroke, { Color = CONFIG.colors.toggleOn }, dur)
                tween(knobFrame, { Position = UDim2.new(0, onX, 0.5, -knob/2) }, dur)
            else
                tween(track2, { BackgroundColor3 = CONFIG.colors.toggleOff }, dur)
                tween(trackStroke, { Color = CONFIG.colors.cardBorder }, dur)
                tween(knobFrame, { Position = UDim2.new(0, offX, 0.5, -knob/2) }, dur)
            end
        end

        render(false)

        track(hit.Activated:Connect(function()
            if locked then
                -- premium feature: gold notification, no state change
                notify(titleText, "Premium only! Join Discord to unlock.", "premium", 3)
                return
            end
            UI.options[key] = not UI.options[key]
            render(true)
            if onChange then onChange(UI.options[key], titleText) end
            if UI.options[key] then
                notify(titleText, "Enabled", "success")
            else
                notify(titleText, "Disabled", "success")
            end
        end))

        return card
    end

    -- BUTTON
    local function createButtonCard(order, titleText, descText, onClick)
        local card, title, desc = createCard(order)
        title.Text = titleText
        desc.Text  = descText

        local iconSize = CONFIG.sizes.handIconSize
        local iconY    = CONFIG.sizes.handIconY
        local right    = CONFIG.sizes.controlRight

        local hand = Instance.new("ImageLabel")
        hand.Size = UDim2.new(0, iconSize, 0, iconSize)
        hand.Position = UDim2.new(1, -(iconSize + right), 0, iconY)
        hand.BackgroundTransparency = 1
        hand.Image = CONFIG.handIconId
        hand.ImageColor3 = CONFIG.colors.handIdle
        hand.ScaleType = Enum.ScaleType.Fit
        hand.Parent = card

        local hit = Instance.new("TextButton")
        hit.Size = UDim2.new(1, 0, 1, 0)
        hit.BackgroundTransparency = 1
        hit.Text = ""
        hit.AutoButtonColor = false
        hit.ZIndex = 5
        hit.Parent = card

        track(hit.MouseEnter:Connect(function()
            tween(card, { BackgroundColor3 = CONFIG.colors.navActiveBg }, CONFIG.anim.fast)
            tween(cardStroke, { Color = CONFIG.colors.accent }, CONFIG.anim.fast)
            tween(hand, { ImageColor3 = CONFIG.colors.handHover }, CONFIG.anim.fast)
        end))
        track(hit.MouseLeave:Connect(function()
            tween(card, { BackgroundColor3 = CONFIG.colors.cardBg }, CONFIG.anim.fast)
            tween(cardStroke, { Color = CONFIG.colors.cardBorder }, CONFIG.anim.fast)
            tween(hand, { ImageColor3 = CONFIG.colors.handIdle }, CONFIG.anim.fast)
        end))

        track(hit.Activated:Connect(function()
            local origSize, origPos = hand.Size, hand.Position
            local pop = iconSize - 4
            hand.Size = UDim2.new(0, pop, 0, pop)
            hand.Position = UDim2.new(1, -(pop + right) - 2, 0, iconY + 2)
            task.delay(0.08, function()
                hand.Size = origSize
                hand.Position = origPos
            end)
            if onClick then onClick() end
        end))

        return card
    end

    -- TEXTBOX
    local function createTextboxCard(order, titleText, descText, key, default, onConfirm)
        local card, title, desc = createCard(order)
        title.Text = titleText
        desc.Text  = descText
        if UI.options[key] == nil then UI.options[key] = default or "" end

        local boxW = CONFIG.sizes.controlW
        local boxH = CONFIG.sizes.controlH
        local right = CONFIG.sizes.controlRight

        local box = Instance.new("TextBox")
        box.Size = UDim2.new(0, boxW, 0, boxH)
        box.Position = UDim2.new(1, -(boxW + right), 0.5, -boxH/2)
        box.BackgroundColor3 = CONFIG.colors.navBg
        box.Text = UI.options[key]
        box.PlaceholderText = "0"
        box.PlaceholderColor3 = CONFIG.colors.textMuted
        box.TextColor3 = CONFIG.colors.textPrimary
        box.Font = Enum.Font.GothamBold
        box.TextSize = 20
        box.TextXAlignment = Enum.TextXAlignment.Center
        box.ClearTextOnFocus = false
        box.ZIndex = 10
        box.Parent = card
        corner(box, 10)
        local boxStroke = stroke(box, CONFIG.colors.cardBorder, 1.5, 0)

        track(box:GetPropertyChangedSignal("Text"):Connect(function()
            local filtered = box.Text:gsub("[^%d]", "")
            if filtered ~= box.Text then box.Text = filtered end
            UI.options[key] = filtered
        end))

        track(box.Focused:Connect(function()
            tween(box, { BackgroundColor3 = CONFIG.colors.navActiveBg }, CONFIG.anim.fast)
            tween(boxStroke, { Color = CONFIG.colors.accent }, CONFIG.anim.fast)
        end))
        track(box.FocusLost:Connect(function()
            tween(box, { BackgroundColor3 = CONFIG.colors.navBg }, CONFIG.anim.fast)
            tween(boxStroke, { Color = CONFIG.colors.cardBorder }, CONFIG.anim.fast)
            if onConfirm then onConfirm(box.Text, titleText) end
        end))

        return card, box
    end

    -- CONFIG UI HELPERS
    local function smallButton(parent, y, text, onClick)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(1, -28, 0, 34)
        btn.Position = UDim2.new(0, 14, 0, y)
        btn.BackgroundColor3 = CONFIG.colors.cfgBtnBg
        btn.Text = text
        btn.TextColor3 = CONFIG.colors.textPrimary
        btn.Font = Enum.Font.GothamBold
        btn.TextSize = CONFIG.textSizes.cfgBtn
        btn.AutoButtonColor = false
        btn.TextStrokeTransparency = 1
        btn.Parent = parent
        corner(btn, 8)
        local s = stroke(btn, CONFIG.colors.cfgBtnBorder, 1.2, 0)

        btn.MouseEnter:Connect(function()
            tween(btn, { BackgroundColor3 = CONFIG.colors.navActiveBg }, CONFIG.anim.fast)
            tween(s, { Color = CONFIG.colors.accent }, CONFIG.anim.fast)
        end)
        btn.MouseLeave:Connect(function()
            tween(btn, { BackgroundColor3 = CONFIG.colors.cfgBtnBg }, CONFIG.anim.fast)
            tween(s, { Color = CONFIG.colors.cfgBtnBorder }, CONFIG.anim.fast)
        end)
        btn.Activated:Connect(function()
            local osz, opos = btn.Size, btn.Position
            btn.Size = UDim2.new(1, -32, 0, 32)
            btn.Position = UDim2.new(0, 16, 0, y + 1)
            task.delay(0.07, function()
                btn.Size = osz
                btn.Position = opos
            end)
            if onClick then onClick() end
        end)
        return btn
    end

    local function smallLabel(parent, y, text)
        local lbl = Instance.new("TextLabel")
        lbl.Size = UDim2.new(1, -28, 0, 22)
        lbl.Position = UDim2.new(0, 14, 0, y)
        lbl.BackgroundTransparency = 1
        lbl.Text = text
        lbl.TextColor3 = CONFIG.colors.textPrimary
        lbl.Font = Enum.Font.GothamBold
        lbl.TextSize = CONFIG.textSizes.cfgLabel
        lbl.TextXAlignment = Enum.TextXAlignment.Left
        lbl.TextStrokeTransparency = 1
        lbl.Parent = parent
        return lbl
    end

    -- KEYBIND CARD
    local function buildKeybindCard()
        local card = Instance.new("Frame")
        card.Size = UDim2.new(1, 0, 0, 110)
        card.BackgroundColor3 = CONFIG.colors.cardBg
        card.BorderSizePixel = 0
        card.LayoutOrder = 0
        card.Parent = scroll
        corner(card, CONFIG.sizes.cardRadius)
        stroke(card, CONFIG.colors.cardBorder, 1.5, 0)

        local title = Instance.new("TextLabel")
        title.BackgroundTransparency = 1
        title.Text = "Toggle Keybind"
        title.TextColor3 = CONFIG.colors.textPrimary
        title.Font = Enum.Font.GothamBold
        title.TextSize = 24
        title.TextXAlignment = Enum.TextXAlignment.Left
        title.TextYAlignment = Enum.TextYAlignment.Top
        title.Size = UDim2.new(1, -160, 0, 28)
        title.Position = UDim2.new(0, 16, 0, 24)
        title.Parent = card

        local desc = Instance.new("TextLabel")
        desc.BackgroundTransparency = 1
        desc.Text = "Key to open/close the interface"
        desc.TextColor3 = CONFIG.colors.textDesc
        desc.Font = Enum.Font.Gotham
        desc.TextSize = 15
        desc.TextXAlignment = Enum.TextXAlignment.Left
        desc.TextYAlignment = Enum.TextYAlignment.Top
        desc.Size = UDim2.new(1, -160, 0, 22)
        desc.Position = UDim2.new(0, 16, 0, 60)
        desc.Parent = card

        local keyBtn = Instance.new("TextButton")
        keyBtn.Size = UDim2.new(0, 130, 0, 46)
        keyBtn.Position = UDim2.new(1, -146, 0.5, -23)
        keyBtn.BackgroundColor3 = CONFIG.colors.navBg
        keyBtn.Text = UI.options["toggle_key"]
        keyBtn.TextColor3 = CONFIG.colors.textPrimary
        keyBtn.Font = Enum.Font.GothamBold
        keyBtn.TextSize = 18
        keyBtn.AutoButtonColor = false
        keyBtn.Parent = card
        corner(keyBtn, 10)
        local keyStroke = stroke(keyBtn, CONFIG.colors.cardBorder, 1.5, 0)

        keyBtn.MouseEnter:Connect(function()
            if not UI.listeningKey then
                tween(keyBtn, { BackgroundColor3 = CONFIG.colors.navActiveBg }, CONFIG.anim.fast)
                tween(keyStroke, { Color = CONFIG.colors.accent }, CONFIG.anim.fast)
            end
        end)
        keyBtn.MouseLeave:Connect(function()
            if not UI.listeningKey then
                tween(keyBtn, { BackgroundColor3 = CONFIG.colors.navBg }, CONFIG.anim.fast)
                tween(keyStroke, { Color = CONFIG.colors.cardBorder }, CONFIG.anim.fast)
            end
        end)

        local listenConn = nil

        local function stopListening()
            UI.listeningKey = false
            if listenConn then pcall(function() listenConn:Disconnect() end) end
            listenConn = nil
            keyBtn.Text = UI.options["toggle_key"]
            tween(keyBtn, { BackgroundColor3 = CONFIG.colors.navBg }, CONFIG.anim.fast)
            tween(keyStroke, { Color = CONFIG.colors.cardBorder }, CONFIG.anim.fast)
        end

        keyBtn.Activated:Connect(function()
            if UI.listeningKey then return end
            UI.listeningKey = true
            keyBtn.Text = "Press a key..."
            tween(keyBtn, { BackgroundColor3 = CONFIG.colors.navActiveBg }, CONFIG.anim.fast)
            tween(keyStroke, { Color = CONFIG.colors.accent }, CONFIG.anim.fast)

            listenConn = UserInputService.InputBegan:Connect(function(input, gpe)
                if gpe then return end
                if input.UserInputType ~= Enum.UserInputType.Keyboard then return end
                local key = input.KeyCode
                if key == Enum.KeyCode.Unknown then return end

                UI.options["toggle_key"] = key.Name
                stopListening()
                notify("Keybind", "Set to " .. key.Name, "success")
            end)
        end)
    end

    local function buildConfigCard()
        local card = Instance.new("Frame")
        card.Size = UDim2.new(1, 0, 0, CONFIG.sizes.cfgHeight)
        card.BackgroundColor3 = CONFIG.colors.cardBg
        card.BorderSizePixel = 0
        card.LayoutOrder = 1
        card.Parent = scroll
        corner(card, CONFIG.sizes.cardRadius)
        stroke(card, CONFIG.colors.cardBorder, 1.5, 0)

        local icon = Instance.new("ImageLabel")
        icon.Size = UDim2.new(0, 26, 0, 26)
        icon.Position = UDim2.new(0, 14, 0, 14)
        icon.BackgroundTransparency = 1
        icon.Image = CONFIG.cfgIconId
        icon.ImageColor3 = CONFIG.colors.cfgIcon
        icon.ScaleType = Enum.ScaleType.Fit
        icon.Parent = card

        local header = Instance.new("TextLabel")
        header.Size = UDim2.new(1, -60, 0, 26)
        header.Position = UDim2.new(0, 48, 0, 16)
        header.BackgroundTransparency = 1
        header.Text = "Configuration"
        header.TextColor3 = CONFIG.colors.textPrimary
        header.Font = Enum.Font.GothamBold
        header.TextSize = CONFIG.textSizes.cfgTitle
        header.TextXAlignment = Enum.TextXAlignment.Left
        header.TextStrokeTransparency = 1
        header.Parent = card

        smallLabel(card, 54, "Config name")

        local nameBox = Instance.new("TextBox")
        nameBox.Size = UDim2.new(1, -28, 0, 38)
        nameBox.Position = UDim2.new(0, 14, 0, 78)
        nameBox.BackgroundColor3 = CONFIG.colors.cfgBtnBg
        nameBox.Text = ""
        nameBox.PlaceholderText = "type a name..."
        nameBox.PlaceholderColor3 = CONFIG.colors.textMuted
        nameBox.TextColor3 = CONFIG.colors.textPrimary
        nameBox.Font = Enum.Font.GothamBold
        nameBox.TextSize = CONFIG.textSizes.cfgInput
        nameBox.TextXAlignment = Enum.TextXAlignment.Left
        nameBox.ClearTextOnFocus = false
        nameBox.TextStrokeTransparency = 1
        nameBox.Parent = card
        corner(nameBox, 8)
        local nameStroke = stroke(nameBox, CONFIG.colors.cfgBtnBorder, 1.2, 0)
        local padLeft = Instance.new("UIPadding")
        padLeft.PaddingLeft = UDim.new(0, 12)
        padLeft.Parent = nameBox

        nameBox.Focused:Connect(function()
            tween(nameStroke, { Color = CONFIG.colors.accent }, CONFIG.anim.fast)
        end)
        nameBox.FocusLost:Connect(function()
            tween(nameStroke, { Color = CONFIG.colors.cfgBtnBorder }, CONFIG.anim.fast)
        end)

        smallButton(card, 126, "Create config", function()
            local name = nameBox.Text
            if name == "" then notify("Config", "Enter a name first", "warn") return end
            local ok, err = ConfigSys.save(name, UI.options)
            if ok then
                notify("Config", "\"" .. name .. "\" created", "success")
                nameBox.Text = ""
                UI.cfgList = ConfigSys.list()
                if UI.refreshCfgList then UI.refreshCfgList() end
            else
                notify("Config", "Failed: " .. tostring(err), "error")
            end
        end)

        smallLabel(card, 176, "Config list")

        local ddBtn = Instance.new("TextButton")
        ddBtn.Size = UDim2.new(1, -28, 0, 38)
        ddBtn.Position = UDim2.new(0, 14, 0, 200)
        ddBtn.BackgroundColor3 = CONFIG.colors.cfgBtnBg
        ddBtn.Text = "---"
        ddBtn.TextColor3 = CONFIG.colors.textPrimary
        ddBtn.Font = Enum.Font.GothamBold
        ddBtn.TextSize = CONFIG.textSizes.cfgInput
        ddBtn.TextXAlignment = Enum.TextXAlignment.Left
        ddBtn.AutoButtonColor = false
        ddBtn.TextStrokeTransparency = 1
        ddBtn.Parent = card
        corner(ddBtn, 8)
        local ddStroke = stroke(ddBtn, CONFIG.colors.cfgBtnBorder, 1.2, 0)
        local ddPad = Instance.new("UIPadding")
        ddPad.PaddingLeft = UDim.new(0, 12)
        ddPad.Parent = ddBtn

        local arrow = Instance.new("TextLabel")
        arrow.Size = UDim2.new(0, 24, 0, 38)
        arrow.Position = UDim2.new(1, -30, 0, 0)
        arrow.BackgroundTransparency = 1
        arrow.Text = "^"
        arrow.TextColor3 = CONFIG.colors.textDesc
        arrow.Font = Enum.Font.GothamBold
        arrow.TextSize = 16
        arrow.TextStrokeTransparency = 1
        arrow.Parent = ddBtn

        local ddListOpen = false
        local ddBackdrop, ddList = nil, nil

        local function closeDd()
            if not ddListOpen then return end
            ddListOpen = false
            if ddBackdrop and ddBackdrop.Parent then ddBackdrop:Destroy() end
            if ddList and ddList.Parent then ddList:Destroy() end
            ddBackdrop, ddList = nil, nil
            tween(ddStroke, { Color = CONFIG.colors.cfgBtnBorder }, CONFIG.anim.fast)
        end

        local function openDd()
            if ddListOpen then closeDd() return end
            ddListOpen = true

            ddBackdrop = Instance.new("TextButton")
            ddBackdrop.Size = UDim2.new(1, 0, 1, 0)
            ddBackdrop.BackgroundTransparency = 1
            ddBackdrop.Text = ""
            ddBackdrop.AutoButtonColor = false
            ddBackdrop.ZIndex = 60
            ddBackdrop.Parent = scaleWrapper
            ddBackdrop.Activated:Connect(closeDd)

            UI.cfgList = ConfigSys.list()
            local list = UI.cfgList
            if #list == 0 then
                tween(ddStroke, { Color = CONFIG.colors.accent }, CONFIG.anim.fast)
                notify("Config", "No configs found", "info")
                closeDd()
                return
            end

            local btnAbs = ddBtn.AbsolutePosition
            local wrapAbs = scaleWrapper.AbsolutePosition
            local s = uiScale.Scale
            local relX = (btnAbs.X - wrapAbs.X) / s
            local relW = ddBtn.AbsoluteSize.X / s
            local btnTopY = (btnAbs.Y - wrapAbs.Y) / s

            local itemH = 34
            local listH = #list * itemH + 8
            local relY = btnTopY - listH - 6

            ddList = Instance.new("Frame")
            ddList.Size = UDim2.new(0, relW, 0, listH)
            ddList.Position = UDim2.new(0, relX, 0, relY)
            ddList.BackgroundColor3 = CONFIG.colors.cardBg
            ddList.BorderSizePixel = 0
            ddList.ZIndex = 61
            ddList.Parent = scaleWrapper
            corner(ddList, 10)
            stroke(ddList, CONFIG.colors.accent, 1.5, 0)

            local pad = Instance.new("UIPadding")
            pad.PaddingTop = UDim.new(0, 4)
            pad.PaddingBottom = UDim.new(0, 4)
            pad.Parent = ddList

            local lay = Instance.new("UIListLayout")
            lay.FillDirection = Enum.FillDirection.Vertical
            lay.Parent = ddList

            for i, name in ipairs(list) do
                local item = Instance.new("TextButton")
                item.Size = UDim2.new(1, -8, 0, itemH)
                item.Position = UDim2.new(0, 4, 0, 0)
                item.BackgroundColor3 = CONFIG.colors.cardBg
                item.BackgroundTransparency = 1
                item.Text = name
                item.TextColor3 = (name == UI.cfgSelected) and CONFIG.colors.accent or CONFIG.colors.textPrimary
                item.Font = Enum.Font.GothamBold
                item.TextSize = 16
                item.TextXAlignment = Enum.TextXAlignment.Left
                item.AutoButtonColor = false
                item.TextStrokeTransparency = 1
                item.LayoutOrder = i
                item.ZIndex = 62
                item.Parent = ddList
                corner(item, 6)

                local ipad = Instance.new("UIPadding")
                ipad.PaddingLeft = UDim.new(0, 10)
                ipad.Parent = item

                item.MouseEnter:Connect(function()
                    tween(item, { BackgroundTransparency = 0, BackgroundColor3 = CONFIG.colors.navActiveBg }, CONFIG.anim.fast)
                end)
                item.MouseLeave:Connect(function()
                    tween(item, { BackgroundTransparency = 1 }, CONFIG.anim.fast)
                end)
                item.Activated:Connect(function()
                    UI.cfgSelected = name
                    ddBtn.Text = name
                    closeDd()
                end)
            end

            tween(ddStroke, { Color = CONFIG.colors.accent }, CONFIG.anim.fast)
        end

        UI.refreshCfgList = function()
            UI.cfgList = ConfigSys.list()
            ddBtn.Text = UI.cfgSelected or "---"
        end

        ddBtn.Activated:Connect(openDd)
        ddBtn.MouseEnter:Connect(function()
            tween(ddBtn, { BackgroundColor3 = CONFIG.colors.navActiveBg }, CONFIG.anim.fast)
        end)
        ddBtn.MouseLeave:Connect(function()
            tween(ddBtn, { BackgroundColor3 = CONFIG.colors.cfgBtnBg }, CONFIG.anim.fast)
        end)

        smallButton(card, 250, "Load config", function()
            local name = UI.cfgSelected
            if not name then notify("Config", "Select a config first", "warn") return end
            local data = ConfigSys.load(name)
            if not data then notify("Config", "Failed to load", "error") return end
            for k, v in pairs(data) do UI.options[k] = v end
            if UI.rebuildTab then UI.rebuildTab() end
            notify("Config", "\"" .. name .. "\" loaded", "success")
        end)

        smallButton(card, 288, "Overwrite config", function()
            local name = UI.cfgSelected
            if not name then notify("Config", "Select a config first", "warn") return end
            local ok, err = ConfigSys.save(name, UI.options)
            if ok then notify("Config", "\"" .. name .. "\" overwritten", "success")
            else notify("Config", "Failed: " .. tostring(err), "error") end
        end)

        smallButton(card, 326, "Delete config", function()
            local name = UI.cfgSelected
            if not name then notify("Config", "Select a config first", "warn") return end
            ConfigSys.delete(name)
            UI.cfgSelected = nil
            ddBtn.Text = "---"
            UI.cfgList = ConfigSys.list()
            notify("Config", "\"" .. name .. "\" deleted", "info")
        end)

        smallButton(card, 364, "Refresh list", function()
            UI.cfgList = ConfigSys.list()
            notify("Config", "List refreshed", "info")
        end)

        smallButton(card, 402, "Set as autoload", function()
            local name = UI.cfgSelected
            if not name then notify("Config", "Select a config first", "warn") return end
            ConfigSys.setAutoload(name)
            if UI.updateAutoloadText then UI.updateAutoloadText() end
            notify("Config", "\"" .. name .. "\" set as autoload", "success")
        end)

        smallButton(card, 440, "Reset autoload", function()
            ConfigSys.setAutoload(nil)
            if UI.updateAutoloadText then UI.updateAutoloadText() end
            notify("Config", "Autoload cleared", "info")
        end)

        local autoloadText = Instance.new("TextLabel")
        autoloadText.Size = UDim2.new(1, -28, 0, 22)
        autoloadText.Position = UDim2.new(0, 14, 0, 482)
        autoloadText.BackgroundTransparency = 1
        autoloadText.Text = "Current autoload config: " .. (ConfigSys.getAutoload() or "none")
        autoloadText.TextColor3 = CONFIG.colors.textDesc
        autoloadText.Font = Enum.Font.GothamBold
        autoloadText.TextSize = CONFIG.textSizes.cfgHint
        autoloadText.TextXAlignment = Enum.TextXAlignment.Left
        autoloadText.TextStrokeTransparency = 1
        autoloadText.Parent = card

        UI.updateAutoloadText = function()
            autoloadText.Text = "Current autoload config: " .. (ConfigSys.getAutoload() or "none")
        end
    end

    -- TAB BUILDERS
    local function clearContent()
        for _, c in ipairs(scroll:GetChildren()) do
            if not c:IsA("UIListLayout") and not c:IsA("UIPadding") then
                c:Destroy()
            end
        end
    end

    -- MOVEMENT tab
    local function buildMovementTab()
        clearContent()
        createToggleCard(1, "Auto Dodge", "Auto-dodges incoming balls", "opt_autododge", false,
            function(v) States.AutoDodge = v; AutoDodge_Toggle(v) end, false)
        createToggleCard(2, "Auto Win", "Teleports to a safe platform above the arena", "opt_autowin", false,
            function(v) States.AutoWin = v; AutoWin_Toggle(v) end, true)
        createToggleCard(3, "Auto Play", "Plays the round automatically — wanders & dodges", "opt_autoplay", false,
            function(v) States.AutoPlay = v; AutoPlay_Toggle(v) end, true)
    end

    -- VISUALS tab
    local function buildVisualsTab()
        clearContent()
        createToggleCard(1, "ESP Ball", "Shows ball labels + speed above every ball", "opt_espball", false,
            function(v) States.ESPBall = v; ESPBall_Toggle(v) end, true)
    end

    -- PLAYER tab
    local function buildPlayerTab()
        clearContent()
        createToggleCard(1, "Anti Kick", "Blocks kick attempts (always on by default)", "opt_anti_kick", true,
            function(v) AntiKickActive = v end, false)
        createToggleCard(2, "Anti Abilities", "Resists ragdoll, sit, walkspeed & platform-stand", "opt_antiable", false,
            function(v) States.AntiAbilities = v; AntiAbilities_Toggle(v) end, true)
    end

    -- MISC tab
    local function buildMiscTab()
        clearContent()
        createButtonCard(1, "Redeem Codes", "Redeems: DODGE, 250K, 1KCCU",
            function()
                local ok, err = RedeemAllCodes()
                if not ok then
                    notify("Redeem", "Failed: " .. tostring(err), "error")
                else
                    notify("Redeem", "Codes redeemed!", "success")
                end
            end)
    end

    local function buildSettingsTab()
        clearContent()
        buildKeybindCard()
        buildConfigCard()
    end

    local function buildTab(tabId)
        if tabId == "main" then buildMovementTab()
        elseif tabId == "visual" then buildVisualsTab()
        elseif tabId == "player" then buildPlayerTab()
        elseif tabId == "misc" then buildMiscTab()
        elseif tabId == "settings" then buildSettingsTab()
        else buildMovementTab() end
    end

    UI.rebuildTab = function() buildTab(UI.activeTab) end

    _G.BabisUI = UI
    _G.BabisUI.registerTab = function(tabId, builder)
        if type(tabId) ~= "string" or type(builder) ~= "function" then return end
        local orig = buildTab
        buildTab = function(id)
            if id == tabId then clearContent(); builder()
            else orig(id) end
        end
    end

    -- TAB ANIMATIONS
    local function animateTabActivate(entry, doPop)
        local baseSize = entry.baseSize
        local activeIconSize = UDim2.new(0, baseSize, 0, baseSize)
        local popSize = UDim2.new(0, baseSize * 1.18, 0, baseSize * 1.18)

        entry.icon.Size = activeIconSize
        tween(entry.button, { BackgroundColor3 = CONFIG.colors.navBg }, CONFIG.anim.fast)
        tween(entry.icon,   { ImageColor3 = CONFIG.colors.navIconActive }, CONFIG.anim.fast)
        tween(entry.stroke, { Color = CONFIG.colors.navActiveBrd, Transparency = 0, Thickness = 3.6 }, 0.18)
        task.delay(0.18, function()
            tween(entry.stroke, { Thickness = 2.4 }, 0.22)
        end)
        tween(entry.iconGlow, { ImageColor3 = CONFIG.colors.navIconActive, ImageTransparency = 0.35 }, CONFIG.anim.fast)

        if doPop then
            entry.icon.Size = UDim2.new(0, baseSize * 0.75, 0, baseSize * 0.75)
            entry.icon.Position = UDim2.new(0.5, -(baseSize * 0.75)/2, 0.5, -(baseSize * 0.75)/2)
            popTween(entry.icon, {
                Size = popSize,
                Position = UDim2.new(0.5, -(baseSize * 1.18)/2, 0.5, -(baseSize * 1.18)/2),
            }, 0.22)
            task.delay(0.22, function()
                tween(entry.icon, {
                    Size = activeIconSize,
                    Position = UDim2.new(0.5, -baseSize/2, 0.5, -baseSize/2),
                }, 0.16)
            end)
        end
    end

    local function animateTabDeactivate(entry)
        local baseSize = entry.baseSize
        tween(entry.button, { BackgroundColor3 = CONFIG.colors.navBg }, CONFIG.anim.fast)
        tween(entry.stroke, { Color = CONFIG.colors.navBorder, Transparency = 0, Thickness = 2 }, CONFIG.anim.fast)
        tween(entry.icon, { ImageColor3 = CONFIG.colors.navIcon }, CONFIG.anim.fast)
        tween(entry.iconGlow, { ImageColor3 = CONFIG.colors.navIcon, ImageTransparency = 0.85 }, CONFIG.anim.fast)
        entry.icon.Size = UDim2.new(0, baseSize, 0, baseSize)
        entry.icon.Position = UDim2.new(0.5, -baseSize/2, 0.5, -baseSize/2)
    end

    local function pressDownFeedback(entry)
        local s = CONFIG.sizes.navBtnSize
        tween(entry.button, { Size = UDim2.new(0, s * 0.92, 0, s * 0.92) }, 0.08, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    end

    local function pressUpFeedback(entry)
        local s = CONFIG.sizes.navBtnSize
        tween(entry.button, { Size = UDim2.new(0, s, 0, s) }, 0.14, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
    end

    local function selectTab(tabId, doPop)
        UI.activeTab = tabId
        for id, entry in pairs(tabButtons) do
            if id == tabId then animateTabActivate(entry, doPop)
            else animateTabDeactivate(entry) end
        end
        buildTab(tabId)
    end

    for id, entry in pairs(tabButtons) do
        track(entry.hit.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                pressDownFeedback(entry)
            end
        end))
        track(entry.hit.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                pressUpFeedback(entry)
            end
        end))
        track(entry.hit.Activated:Connect(function()
            if UI.activeTab == id then return end
            selectTab(id, true)
        end))
    end

    -- WINDOW CONTROLS
    local function minimizePanel()
        if UI.isMinimized then return end
        UI.isMinimized = true
        local h = CONFIG.sizes.headerHeight + 2
        navBar.Visible = false
        contentContainer.Visible = false
        if UI.infoOpen then infoFrame.Visible = false end

        local px, py = posToPixels(UI.minimizedPosition)
        local cx, cy = clampToViewport(px, py)
        tween(scaleWrapper, {
            Size     = UDim2.new(0, CONFIG.sizes.windowWidth, 0, h),
            Position = UDim2.new(0, cx, 0, cy),
        }, CONFIG.anim.slow)
    end

    local function restorePanel()
        if not UI.isMinimized then return end
        UI.isMinimized = false
        local px, py = posToPixels(UI.normalPos)
        local cx, cy = clampToViewport(px, py)
        tween(scaleWrapper, {
            Size     = UDim2.new(0, CONFIG.sizes.windowWidth, 0, CONFIG.sizes.windowHeight),
            Position = UDim2.new(0, cx, 0, cy),
        }, CONFIG.anim.slow)
        if UI.infoOpen then
            infoFrame.Visible = true
            navBar.Visible = false
            contentContainer.Visible = false
        else
            navBar.Visible = true
            contentContainer.Visible = true
        end
    end

    local function closePanel()
        if not UI.isOpen then return end
        UI.isOpen = false
        navBar.Visible = false
        contentContainer.Visible = false
        if UI.infoOpen then infoFrame.Visible = false end
        tween(scaleWrapper, { Size = UDim2.new(0, CONFIG.sizes.windowWidth, 0, 0) }, CONFIG.anim.normal)
    end

    local function openPanel()
        if UI.destroyed then return end
        if UI.isOpen then return end
        UI.isOpen = true
        local px, py = posToPixels(UI.normalPos)
        local cx, cy = clampToViewport(px, py)
        scaleWrapper.Position = UDim2.new(0, cx, 0, cy)
        tween(scaleWrapper, { Size = UDim2.new(0, CONFIG.sizes.windowWidth, 0, CONFIG.sizes.windowHeight) }, CONFIG.anim.slow)
        if UI.infoOpen then
            infoFrame.Visible = true
            navBar.Visible = false
            contentContainer.Visible = false
        else
            navBar.Visible = true
            contentContainer.Visible = true
        end
    end

    minBtn.Activated:Connect(function()
        if UI.isMinimized then restorePanel() else minimizePanel() end
    end)

    closeBtn.Activated:Connect(function()
        closePanel()
        task.delay(CONFIG.anim.normal + 0.05, function()
            -- turn off all active features on destroy
            pcall(function() if States.AutoDodge then AutoDodge_Toggle(false) end end)
            pcall(function() if States.AutoWin then AutoWin_Toggle(false) end end)
            pcall(function() if States.AntiAbilities then AntiAbilities_Toggle(false) end end)
            pcall(function() if States.ESPBall then ESPBall_Toggle(false) end end)
            pcall(function() if States.AutoPlay then AutoPlay_Toggle(false) end end)
            cleanup()
            if screenGui and screenGui.Parent then screenGui:Destroy() end
        end)
    end)

    minBtn.MouseEnter:Connect(function() tween(minGlyph, { BackgroundColor3 = CONFIG.colors.accent }, CONFIG.anim.fast) end)
    minBtn.MouseLeave:Connect(function() tween(minGlyph, { BackgroundColor3 = CONFIG.colors.textPrimary }, CONFIG.anim.fast) end)
    closeBtn.MouseEnter:Connect(function()
        tween(xBar1, { BackgroundColor3 = CONFIG.colors.closeHover }, CONFIG.anim.fast)
        tween(xBar2, { BackgroundColor3 = CONFIG.colors.closeHover }, CONFIG.anim.fast)
    end)
    closeBtn.MouseLeave:Connect(function()
        tween(xBar1, { BackgroundColor3 = CONFIG.colors.textPrimary }, CONFIG.anim.fast)
        tween(xBar2, { BackgroundColor3 = CONFIG.colors.textPrimary }, CONFIG.anim.fast)
    end)

    -- DRAG
    local dragging = false
    local dragStart, startPos

    header.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = scaleWrapper.Position
        end
    end)

    track(UserInputService.InputChanged:Connect(function(input)
        if not dragging then return end
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            local cam = workspace.CurrentCamera
            if not cam then return end
            local vp = cam.ViewportSize
            local delta = input.Position - dragStart
            local baseX = startPos.X.Scale * vp.X + startPos.X.Offset
            local baseY = startPos.Y.Scale * vp.Y + startPos.Y.Offset
            local px, py = clampToViewport(baseX + delta.X, baseY + delta.Y)
            local newPos = UDim2.new(0, px, 0, py)
            scaleWrapper.Position = newPos
            if UI.isMinimized then UI.minimizedPosition = newPos else UI.normalPos = newPos end
        end
    end))

    track(UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end))

    -- KEYBIND LISTENER
    UI.toggleKeyConn = UserInputService.InputBegan:Connect(function(input, gpe)
        if UI.destroyed then return end
        if gpe then return end
        if UI.listeningKey then return end
        if input.UserInputType ~= Enum.UserInputType.Keyboard then return end
        local savedKey = UI.options["toggle_key"]
        if not savedKey or savedKey == "None" or savedKey == "" then return end
        if input.KeyCode == Enum.KeyCode[savedKey] then
            if UI.isOpen then closePanel() else openPanel() end
        end
    end)
    track(UI.toggleKeyConn)

    -- SCALE + CLAMP
    local function updateScale()
        local cam = workspace.CurrentCamera
        if not cam then return end
        local vp = cam.ViewportSize
        local sx = (vp.X - 16) / CONFIG.sizes.windowWidth
        local sy = (vp.Y - 32) / CONFIG.sizes.windowHeight
        local s = math.min(1.25, math.min(sx, sy))
        s = math.max(s, 0.55)
        uiScale.Scale = s

        local cur = scaleWrapper.Position
        local px = cur.X.Scale * vp.X + cur.X.Offset
        local py = cur.Y.Scale * vp.Y + cur.Y.Offset
        local cx, cy = clampToViewport(px, py)
        scaleWrapper.Position = UDim2.new(0, cx, 0, cy)
        if UI.isMinimized then UI.minimizedPosition = scaleWrapper.Position
        else UI.normalPos = scaleWrapper.Position end
    end

    updateScale()
    local cam = workspace.CurrentCamera
    if cam then
        track(cam:GetPropertyChangedSignal("ViewportSize"):Connect(updateScale))
    end

    selectTab(CONFIG.defaultTab, false)
    openPanel()

    screenGui.Destroyed:Connect(cleanup)
end

-- ============================================================
-- UNLOAD
-- ============================================================
getgenv().BabisUnload = function()
    States.AutoWin = false; States.AutoDodge = false; States.AntiAbilities = false
    States.ESPBall = false; States.AutoPlay = false
    pcall(function() AutoWin_Toggle(false) end)
    pcall(function() AutoDodge_Toggle(false) end)
    pcall(function() AntiAbilities_Toggle(false) end)
    pcall(function() ESPBall_Toggle(false) end)
    pcall(function() AutoPlay_Toggle(false) end)
    if getgenv().BabisConnections then
        for _, c in ipairs(getgenv().BabisConnections) do pcall(function() c:Disconnect() end) end
    end
    if getgenv().BabisHub then pcall(function() getgenv().BabisHub:Destroy() end); getgenv().BabisHub = nil end
    if getgenv().BabisAutoWinPlatform then pcall(function() getgenv().BabisAutoWinPlatform:Destroy() end) end
    if getgenv().BabisAntiKickHook then pcall(function() getgenv().BabisAntiKickHook() end) end
    getgenv().BabisUnload = nil
    print("[Babis] Unloaded.")
end

-- ============================================================
-- BOOT
-- ============================================================
destroyPrevious()
showIntro(buildMainUI) 
