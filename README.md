-- ========== CONFIG ==========

-- Müzik ID’leri
local musicIds = {
    135905702909529,129956211201589,
    126265599978689,125858109122379,
    72928510467705,128214703292900,
    108920664907736,99409598156364,
    129932630593014,133606719802850,
    115262512648819,72953360415621,
    102549127891820,76424067353965,
    92979002969874,114608169341947,
    94301557485291,137179635212128,
    81057982701796,103456533543835,
    137875928536202,122791619159604,
    133674068037842,113386216095936,
    77308119523078,16190784875,
    4857572997,91299590639144,
    16190760005,74406273981175,
    3172774016,97113848276426,
    15689445424,109784877184952,
    89907278904871,126665348638798,
    116352402771776,110519906029322,
    117016788667676,93722084656579,
    132436320685732,81967674437872,
    5672354144,104364160220905,
    18160236567,104042367284948,
    99128910197696,9099130679,
    107813436145992,8624529051,
    128065862392247,97173189032263,
    86585300300844,16190782181,
    140555145345310
}

-- Arka plan resmi (Backrooms tarlasında evler):  
local BACKGROUND_IMAGE_ID = "135346431020693"

-- =================================

local Players = game:GetService("Players")
local plr     = Players.LocalPlayer
local char    = plr.Character or plr.CharacterAdded:Wait()
local backpack= plr:WaitForChild("Backpack")

-- Boombox tool’unu bulma
local function getBoomboxTool()
    for _, t in ipairs(char:GetChildren()) do
        if t:IsA("Tool") and t.Name:lower():find("boombox") then
            return t
        end
    end
    for _, t in ipairs(backpack:GetChildren()) do
        if t:IsA("Tool") and t.Name:lower():find("boombox") then
            return t
        end
    end
    return nil
end

-- Uyarı bildirimi
local function notify(text)
    pcall(function()
        game.StarterGui:SetCore("SendNotification", {
            Title = "Boombox Music";
            Text = text;
            Duration = 2;
        })
    end)
end

-- Müzik çalma fonksiyonu
local function playPublicMusic(id)
    -- 1) Boombox’u bul
    local boombox = getBoomboxTool()
    if not boombox then
        return notify("You don't have a Boombox!")
    end

    -- 2) Elde değilse karaktere taşı
    if boombox.Parent ~= char then
        boombox.Parent = char
        wait(0.2)
    end

    -- 3) Handle’ı al
    local handle = boombox:FindFirstChild("Handle")
    if not handle then
        return notify("Boombox has no Handle part!")
    end

    -- 4) Eski sesleri temizle
    for _, c in ipairs(handle:GetChildren()) do
        if c:IsA("Sound") then
            c:Destroy()
        end
    end

    -- 5) Yeni Sound oluştur ve oynat
    local sound = Instance.new("Sound")
    sound.Name      = "BoomboxSound"
    sound.SoundId   = "rbxassetid://" .. tostring(id)
    sound.Volume    = 10
    sound.Looped    = false
    sound.Parent    = handle
    wait(0.1)

    -- 6) Boombox tool scriptini tetiklemek için Activate() (bazı modellerde gerekli)
    pcall(function() boombox:Activate() end)
    wait(0.2)

    sound:Play()

    -- 7) Boombox elden çıkarsa sesi durdur
    sound:GetPropertyChangedSignal("Parent"):Connect(function()
        if not sound.Parent then
            sound:Stop()
        end
    end)
    boombox.AncestryChanged:Connect(function(_, newParent)
        if not newParent then
            sound:Stop()
        end
    end)
end

-- ========== GUI OLUŞTURMA ==========

local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "BoomboxPublicGUI"

-- Arka plan resmi
local bg = Instance.new("ImageLabel", gui)
bg.Size               = UDim2.new(1,0,1,0)
bg.Position           = UDim2.new(0,0,0,0)
bg.Image              = "rbxassetid://"..BACKGROUND_IMAGE_ID
bg.BackgroundTransparency = 1
bg.ImageTransparency  = 0.25

-- Ana pencere
local frame = Instance.new("Frame", gui)
frame.Size               = UDim2.new(0,200,0,280)
frame.Position           = UDim2.new(0.05,0,0.25,0)
frame.BackgroundColor3   = Color3.fromRGB(30,30,30)
frame.BackgroundTransparency = 0.4
frame.BorderSizePixel    = 0
frame.Active             = true
frame.Draggable          = true

-- Başlık
local title = Instance.new("TextLabel", frame)
title.Size               = UDim2.new(1,0,0,26)
title.Position           = UDim2.new(0,0,0,0)
title.BackgroundColor3   = Color3.fromRGB(50,50,50)
title.BackgroundTransparency = 0.2
title.Text               = "Boombox Songs"
title.TextColor3         = Color3.new(1,1,1)
title.TextSize           = 15

-- Kaydırılabilir alan
local scroll = Instance.new("ScrollingFrame", frame)
scroll.Size               = UDim2.new(1,0,1,-26)
scroll.Position           = UDim2.new(0,0,0,26)
scroll.CanvasSize         = UDim2.new(0,0,0,#musicIds*36)
scroll.BackgroundTransparency = 1
scroll.ScrollBarThickness = 6

-- Şarkı butonları
for i, id in ipairs(musicIds) do
    local btn = Instance.new("TextButton", scroll)
    btn.Size               = UDim2.new(1,-8,0,32)
    btn.Position           = UDim2.new(0,4,0,(i-1)*36)
    btn.BackgroundColor3   = Color3.fromRGB(20,20,20)
    btn.BackgroundTransparency = 0.3
    btn.BorderSizePixel    = 0
    btn.TextColor3         = Color3.new(1,1,1)
    btn.TextSize           = 13
    btn.Text               = "Play Music "..i
    btn.MouseButton1Click:Connect(function()
        playPublicMusic(id)
    end)
end
