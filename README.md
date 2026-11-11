-- Services & Safety Check
local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local Stats = game:GetService("Stats")
local Lighting = game:GetService("Lighting")
local SoundService = game:GetService("SoundService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local InsertService = game:GetService("InsertService")
local TextChatService = game:GetService("TextChatService")
local TeleportService = game:GetService("TeleportService")

local localPlayer = Players.LocalPlayer
local camera = workspace.CurrentCamera
local mouse = localPlayer:GetMouse()

-- OTOMATİK OYUN KONTROLÜ (You vs Hulk ID: 101527854251487)
local TARGET_PLACE_ID = 101527854251487
task.delay(1.5, function()
    if game.PlaceId ~= TARGET_PLACE_ID then
        pcall(function()
            TeleportService:Teleport(TARGET_PLACE_ID, localPlayer)
        end)
    end
end)

-- Dil Sistemi (Varsayılan Türkçe)
local currentLang = "TR"

-- ScreenGui (CoreGui Öncelikli Güvenli Yükleme)
local targetParent = pcall(function() return CoreGui end) and CoreGui or localPlayer:WaitForChild("PlayerGui")
local existingGui = targetParent:FindFirstChild("YouVsHulkUltimateMod")
if existingGui then existingGui:Destroy() end

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "YouVsHulkUltimateMod"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.DisplayOrder = 999999
screenGui.Parent = targetParent

local isMenuLocked = false
local soundEffectsEnabled = true

-- Rahatlatıcı Yumuşak Ses Efekti
local function playClickSound()
    if not soundEffectsEnabled then return end
    pcall(function()
        local sound = Instance.new("Sound")
        sound.SoundId = "rbxassetid://6895311235"
        sound.Volume = 0.35
        sound.Parent = SoundService
        sound:Play()
        sound.Ended:Connect(function() sound:Destroy() end)
    end)
end

-- 1. TOGGLE BUTTON
local toggleBtn = Instance.new("TextButton")
toggleBtn.Name = "ToggleMenuBtn"
toggleBtn.Size = UDim2.new(0, 42, 0, 42)
toggleBtn.Position = UDim2.new(0, 20, 0.5, -21)
toggleBtn.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
toggleBtn.BorderColor3 = Color3.fromRGB(0, 255, 128)
toggleBtn.BorderSizePixel = 2
toggleBtn.TextColor3 = Color3.fromRGB(0, 255, 128)
toggleBtn.TextSize = 12
toggleBtn.Font = Enum.Font.Code
toggleBtn.Text = "MOD"
toggleBtn.Parent = screenGui

local cornerToggle = Instance.new("UICorner")
cornerToggle.CornerRadius = UDim.new(0, 4)
cornerToggle.Parent = toggleBtn

-- 2. MAIN MENU
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 600, 0, 420)
mainFrame.Position = UDim2.new(0.5, -300, 0.5, -210)
mainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
mainFrame.BorderColor3 = Color3.fromRGB(50, 50, 50)
mainFrame.BorderSizePixel = 2
mainFrame.Visible = false
mainFrame.Parent = screenGui

local cornerMain = Instance.new("UICorner")
cornerMain.CornerRadius = UDim.new(0, 6)
cornerMain.Parent = mainFrame

-- General Drag Function
local function makeDraggable(frame, handle)
    handle = handle or frame
    local dragging, dragInput, dragStart, startPos
    
    handle.InputBegan:Connect(function(input)
        if isMenuLocked then return end
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = frame.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then dragging = false end
            end)
        end
    end)

    handle.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if not isMenuLocked and dragging and dragInput and (input == dragInput) then
            local delta = input.Position - dragStart
            frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
end

makeDraggable(toggleBtn)
makeDraggable(mainFrame, mainFrame)

toggleBtn.MouseButton1Click:Connect(function()
    playClickSound()
    mainFrame.Visible = not mainFrame.Visible
end)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.Insert then
        playClickSound()
        mainFrame.Visible = not mainFrame.Visible
    end
end)

-- Title & Stats
local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, 0, 0, 35)
titleLabel.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
titleLabel.BorderSizePixel = 0
titleLabel.Text = "  YOU VS HULK | ULTIMATE MOD MENU (v9.49)"
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.TextSize = 13
titleLabel.Font = Enum.Font.Code
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = mainFrame

local statsLabel = Instance.new("TextLabel")
statsLabel.Size = UDim2.new(0, 180, 1, 0)
statsLabel.Position = UDim2.new(1, -185, 0, 0)
statsLabel.BackgroundTransparency = 1
statsLabel.TextColor3 = Color3.fromRGB(0, 255, 128)
statsLabel.TextSize = 12
statsLabel.Font = Enum.Font.Code
statsLabel.TextXAlignment = Enum.TextXAlignment.Right
statsLabel.Text = "FPS: 60 | Ping: 45ms"
statsLabel.Parent = titleLabel

RunService.RenderStepped:Connect(function()
    local success, fpsVal = pcall(function() return math.floor(1 / RunService.RenderStepped:Wait()) end)
    local ping = 50
    pcall(function() ping = math.floor(Stats.Network.ServerStatsItem["Data Ping"]:GetValue()) end)
    if success and fpsVal then
        statsLabel.Text = string.format("FPS: %d | Ping: %dms", fpsVal, ping)
    end
end)

-- 3. LEFT TABS & CONTENT
local tabListFrame = Instance.new("ScrollingFrame")
tabListFrame.Size = UDim2.new(0, 140, 1, -35)
tabListFrame.Position = UDim2.new(0, 0, 0, 35)
tabListFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
tabListFrame.BorderSizePixel = 0
tabListFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
tabListFrame.Parent = mainFrame

local tabLayout = Instance.new("UIListLayout")
tabLayout.SortOrder = Enum.SortOrder.LayoutOrder
tabLayout.Parent = tabListFrame

local contentFrame = Instance.new("Frame")
contentFrame.Size = UDim2.new(1, -140, 1, -35)
contentFrame.Position = UDim2.new(0, 140, 0, 35)
contentFrame.BackgroundTransparency = 1
contentFrame.Parent = mainFrame

local pages = {}
local tabButtons = {}

local function createTab(nameTR, nameEN, nameRU)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 45)
    btn.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    btn.BorderSizePixel = 0
    btn.TextColor3 = Color3.fromRGB(180, 180, 180)
    btn.TextSize = 13
    btn.Font = Enum.Font.Code
    btn.Text = nameTR
    btn.Parent = tabListFrame

    btn:SetAttribute("NameTR", nameTR)
    btn:SetAttribute("NameEN", nameEN)
    btn:SetAttribute("NameRU", nameRU)
    table.insert(tabButtons, btn)

    local page = Instance.new("ScrollingFrame")
    page.Size = UDim2.new(1, 0, 1, 0)
    page.BackgroundTransparency = 1
    page.Visible = false
    page.CanvasSize = UDim2.new(0, 0, 0, 1300)
    page.ScrollBarThickness = 4
    page.Parent = contentFrame

    local pLayout = Instance.new("UIListLayout")
    pLayout.SortOrder = Enum.SortOrder.LayoutOrder
    pLayout.Padding = UDim.new(0, 10)
    pLayout.Parent = page

    local pPadding = Instance.new("UIPadding")
    pPadding.PaddingTop = UDim.new(0, 15)
    pPadding.PaddingLeft = UDim.new(0, 15)
    pPadding.Parent = page

    btn.MouseButton1Click:Connect(function()
        playClickSound()
        for _, p in pairs(pages) do p.Visible = false end
        page.Visible = true
    end)

    if #pages == 0 then page.Visible = true end
    table.insert(pages, page)
    return page
end

local pageMain = createTab("Ana Sayfa & Savaş", "Main & Combat", "Главная & Бой")
local pageVisuals = createTab("ESP (Görseller)", "ESP (Visuals)", "ESP (Визуальные)")
local pageMovement = createTab("Hareket", "Movement", "Движение")
local pageWorld = createTab("Işınlanma & Üretici", "TP & Spawner", "ТП & Спавнер")
local pageSettings = createTab("Ayarlar", "Settings", "Настройки")
local pageInfo = createTab("Bilgi", "Info", "Информация")

-- WATERMARK
local watermarkFrame = Instance.new("Frame")
watermarkFrame.Size = UDim2.new(0, 160, 0, 30)
watermarkFrame.Position = UDim2.new(1, -170, 0, 15)
watermarkFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
watermarkFrame.BorderColor3 = Color3.fromRGB(0, 255, 128)
watermarkFrame.BorderSizePixel = 1
watermarkFrame.BackgroundTransparency = 0.3
watermarkFrame.Parent = screenGui

local wmCorner = Instance.new("UICorner") wmCorner.CornerRadius = UDim.new(0, 4) wmCorner.Parent = watermarkFrame
local watermarkText = Instance.new("TextLabel")
watermarkText.Size = UDim2.new(1, 0, 1, 0)
watermarkText.BackgroundTransparency = 1
watermarkText.TextColor3 = Color3.fromRGB(0, 255, 128)
watermarkText.TextSize = 11
watermarkText.Font = Enum.Font.Code
watermarkText.Text = "black | v9.49 Ultimate"
watermarkText.Parent = watermarkFrame

-- TRANSLATABLE BUTTON REGISTRY
local translatableElements = {}

local function addToggleButton(parent, textTR, textEN, textRU, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 410, 0, 35)
    btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    btn.BorderSizePixel = 0
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.TextSize = 13
    btn.Font = Enum.Font.Code
    btn.Text = textTR .. " [ KAPALI ]"
    btn.Parent = parent

    btn:SetAttribute("TextTR", textTR)
    btn:SetAttribute("TextEN", textEN)
    btn:SetAttribute("TextRU", textRU)
    table.insert(translatableElements, btn)

    local active = false
    btn.MouseButton1Click:Connect(function()
        playClickSound()
        active = not active
        local currentBase = (currentLang == "TR") and textTR or ((currentLang == "EN") and textEN or textRU)
        local offText = (currentLang == "TR") and " [ KAPALI ]" or ((currentLang == "EN") and " [ OFF ]" or " [ ВЫКЛ ]")
        local onText = (currentLang == "TR") and " [ AÇIK ]" or ((currentLang == "EN") and " [ ON ]" or " [ ВКЛ ]")

        if active then
            btn.BackgroundColor3 = Color3.fromRGB(0, 120, 60)
            btn.Text = currentBase .. onText
        else
            btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
            btn.Text = currentBase .. offText
        end
        callback(active)
    end)
    return btn
end

-- 1. MAIN & COMBAT (HULK FLING & WALLBANG & AIMBOT)
local function isCharacterHulk(char)
    if not char then return false end
    for _, obj in pairs(char:GetDescendants()) do
        if obj.Name:lower():find("hulk") then return true end
    end
    local root = char:FindFirstChild("HumanoidRootPart")
    return root and root.Size.Y > 3.2
end

local aimbotActive = false
addToggleButton(pageMain, "Hulk Kafa Aimbot", "Hulk Head Aimbot", "Аимбот на голову Халка", function(state) aimbotActive = state end)

local triggerbotActive = false
addToggleButton(pageMain, "Oto Tetikçi (Auto Ateş)", "Auto Triggerbot (Auto Shoot)", "Автострельба (Триггербот)", function(state) triggerbotActive = state end)

local fovLockActive = false
addToggleButton(pageMain, "FOV Hedef Kilidi", "FOV Target Lock", "Блокировка цели по FOV", function(state) fovLockActive = state end)

local wallbangActive = false
addToggleButton(pageMain, "Duvar Arkası Vuruş (Wallbang)", "Wallbang (Shoot Through Walls)", "Стрельба сквозь стены (Wallbang)", function(state) wallbangActive = state end)

-- HULK FLING (SADECE HULK OLDUĞUNDA ETRAFI UÇURMA / HARİTA DIŞINA FIRLATMA)
local hulkFlingActive = false
addToggleButton(pageMain, "Hulk Fling (Hulk Oyuncu Fırlatıcı)", "Hulk Fling (Launch Players)", "Hulk Fling (Запуск игроков)", function(state) hulkFlingActive = state end)

local hitboxActive = false
addToggleButton(pageMain, "Hulk Hitbox Büyütücü", "Hulk Hitbox Extender (Big Target)", "Увеличенный хитбокс Халка", function(state) hitboxActive = state end)

RunService.RenderStepped:Connect(function()
    if hitboxActive then
        pcall(function()
            for _, player in pairs(Players:GetPlayers()) do
                if player ~= localPlayer and player.Character then
                    local char = player.Character
                    if isCharacterHulk(char) then
                        local head = char:FindFirstChild("Head")
                        local torso = char:FindFirstChild("UpperTorso") or char:FindFirstChild("Torso")
                        if head then head.Size = Vector3.new(6, 6, 6) head.Transparency = 0.6 head.CanCollide = false end
                        if torso then torso.Size = Vector3.new(7, 8, 7) torso.Transparency = 0.8 end
                    end
                end
            end
        end)
    end
end)

-- HULK FLING ÇALIŞTIRICI (Sadece yerel oyuncu Hulk ise veya Hulk karakteri etrafındaki herkesi fırlatır)
RunService.Heartbeat:Connect(function()
    if hulkFlingActive then
        pcall(function()
            local myChar = localPlayer.Character
            if myChar and isCharacterHulk(myChar) then
                local myRoot = myChar:FindFirstChild("HumanoidRootPart")
                if myRoot then
                    for _, player in pairs(Players:GetPlayers()) do
                        if player ~= localPlayer and player.Character then
                            local enemyChar = player.Character
                            local enemyRoot = enemyChar:FindFirstChild("HumanoidRootPart")
                            if enemyRoot and not isCharacterHulk(enemyChar) then
                                local dist = (enemyRoot.Position - myRoot.Position).Magnitude
                                if dist <= 18 then
                                    -- Harita dışına / gökyüzüne fırlatma kuvveti (Velocity & AngularVelocity)
                                    enemyRoot.AssemblyLinearVelocity = Vector3.new(math.random(-300, 300), 500, math.random(-300, 300))
                                    enemyRoot.AssemblyAngularVelocity = Vector3.new(5000, 5000, 5000)
                                end
                            end
                        end
                    end
                end
            end
        end)
    end
end)

local maxCombatDist = 150
RunService.RenderStepped:Connect(function()
    local targetHulk = nil
    local shortestDist = math.huge
    pcall(function()
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= localPlayer and player.Character then
                local char = player.Character
                if isCharacterHulk(char) and char:FindFirstChild("Head") and char:FindFirstChild("Humanoid") and char.Humanoid.Health > 0 then
                    local head = char.Head
                    local myRoot = localPlayer.Character and localPlayer.Character:FindFirstChild("HumanoidRootPart")
                    if myRoot then
                        local dist = (head.Position - myRoot.Position).Magnitude
                        if dist <= maxCombatDist then
                            local isVisible = true
                            if not wallbangActive then
                                local rayParams = RaycastParams.new()
                                rayParams.FilterDescendantsInstances = {localPlayer.Character, camera}
                                rayParams.FilterType = Enum.RaycastFilterType.Blacklist
                                local rayResult = workspace:Raycast(camera.CFrame.Position, head.Position - camera.CFrame.Position, rayParams)
                                isVisible = not (rayResult and not rayResult.Instance:IsDescendantOf(char))
                            end

                            if isVisible then
                                local passFov = true
                                if fovLockActive then
                                    local sp, onScreen = camera:WorldToViewportPoint(head.Position)
                                    if onScreen then
                                        local mDist = (Vector2.new(sp.X, sp.Y) - Vector2.new(camera.ViewportSize.X/2, camera.ViewportSize.Y/2)).Magnitude
                                        if mDist > 100 then passFov = false end
                                    else passFov = false end
                                end
                                if passFov and dist < shortestDist then shortestDist = dist targetHulk = head end
                            end
                        end
                    end
                end
            end
        end
    end)
    if aimbotActive and targetHulk then camera.CFrame = CFrame.new(camera.CFrame.Position, targetHulk.Position) end
end)

RunService.RenderStepped:Connect(function()
    if triggerbotActive then
        pcall(function()
            local mouseTarget = mouse.Target
            if mouseTarget and mouseTarget.Parent then
                local char = mouseTarget.Parent
                if isCharacterHulk(char) and char:FindFirstChild("Humanoid") and char.Humanoid.Health > 0 then mouse1click() end
            end
        end)
    end
end)
-- 2. ESP TAB
local espKalanlarActive = false
local espHulkActive = false
addToggleButton(pageVisuals, "Oyuncu ESP", "Players ESP", "ESP Игроков", function(state) espKalanlarActive = state end)
addToggleButton(pageVisuals, "Hulk ESP", "Hulk ESP", "ESP Халка", function(state) espHulkActive = state end)

local fovFrame = Instance.new("Frame")
fovFrame.Size = UDim2.new(0, 200, 0, 200)
fovFrame.AnchorPoint = Vector2.new(0.5, 0.5)
fovFrame.Position = UDim2.new(0.5, 0, 0.5, 0)
fovFrame.BackgroundTransparency = 1
fovFrame.Visible = false
fovFrame.Parent = screenGui
local fovCorner = Instance.new("UICorner") fovCorner.CornerRadius = UDim.new(1, 0) fovCorner.Parent = fovFrame
local fovStroke = Instance.new("UIStroke") fovStroke.Thickness = 1.5 fovStroke.Color = Color3.fromRGB(0, 255, 128) fovStroke.Parent = fovFrame

addToggleButton(pageVisuals, "FOV Çemberi", "FOV Circle", "Круг FOV", function(state) fovFrame.Visible = state end)

local crosshairFrame = Instance.new("Frame")
crosshairFrame.Size = UDim2.new(0, 10, 0, 10)
crosshairFrame.AnchorPoint = Vector2.new(0.5, 0.5)
crosshairFrame.Position = UDim2.new(0.5, 0, 0.5, 0)
crosshairFrame.BackgroundTransparency = 1
crosshairFrame.Visible = false
crosshairFrame.Parent = screenGui

local c1 = Instance.new("Frame") c1.Size = UDim2.new(1, 0, 0, 2) c1.Position = UDim2.new(0, 0, 0.5, -1) c1.BackgroundColor3 = Color3.fromRGB(0, 255, 128) c1.BorderSizePixel = 0 c1.Parent = crosshairFrame
local c2 = Instance.new("Frame") c2.Size = UDim2.new(0, 2, 1, 0) c2.Position = UDim2.new(0.5, -1, 0, 0) c2.BackgroundColor3 = Color3.fromRGB(0, 255, 128) c2.BorderSizePixel = 0 c2.Parent = crosshairFrame

addToggleButton(pageVisuals, "Özel Nişangah (Crosshair)", "Custom Crosshair", "Свой Прицел", function(state) crosshairFrame.Visible = state end)

RunService.RenderStepped:Connect(function()
    pcall(function()
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= localPlayer and player.Character then
                local char = player.Character
                local hrp = char:FindFirstChild("HumanoidRootPart")
                local myHrp = localPlayer.Character and localPlayer.Character:FindFirstChild("HumanoidRootPart")
                if hrp and myHrp then
                    local dist = math.floor((hrp.Position - myHrp.Position).Magnitude)
                    local isHulk = isCharacterHulk(char)
                    local billboard = char:FindFirstChild("ModESPBillboard")
                    if not billboard then
                        billboard = Instance.new("BillboardGui")
                        billboard.Name = "ModESPBillboard"
                        billboard.Size = UDim2.new(0, 100, 0, 40)
                        billboard.StudsOffset = Vector3.new(0, 3, 0)
                        billboard.AlwaysOnTop = true
                        local txt = Instance.new("TextLabel")
                        txt.Name = "ESPText" txt.Size = UDim2.new(1, 0, 1, 0) txt.BackgroundTransparency = 1 txt.TextColor3 = Color3.fromRGB(255, 255, 255) txt.TextStrokeTransparency = 0 txt.TextSize = 12 txt.Font = Enum.Font.Code txt.Parent = billboard
                        billboard.Parent = char
                    end
                    local textLabel = billboard:FindFirstChild("ESPText")
                    local show = false
                    if isHulk and espHulkActive then textLabel.TextColor3 = Color3.fromRGB(0, 255, 0) textLabel.Text = "HULK\n[" .. dist .. "m]" show = true
                    elseif not isHulk and espKalanlarActive then textLabel.TextColor3 = Color3.fromRGB(255, 80, 80) textLabel.Text = player.Name .. "\n[" .. dist .. "m]" show = true end
                    billboard.Enabled = show
                end
            end
        end
    end)
end)

-- 3. MOVEMENT TAB
local speedValue = 16
local speedActive = false
addToggleButton(pageMovement, "Hız Hilesi (Speed Hack)", "Speed Hack Active", "Спидхак (Скорость)", function(state) speedActive = state end)

local sliderFrame = Instance.new("Frame") sliderFrame.Size = UDim2.new(0, 410, 0, 50) sliderFrame.BackgroundTransparency = 1 sliderFrame.Parent = pageMovement
local sliderLabel = Instance.new("TextLabel") sliderLabel.Size = UDim2.new(1, 0, 0, 20) sliderLabel.BackgroundTransparency = 1 sliderLabel.Text = "Hız Değeri: 16 (Max: 255)" sliderLabel.TextColor3 = Color3.fromRGB(200, 200, 200) sliderLabel.TextSize = 12 sliderLabel.Font = Enum.Font.Code sliderLabel.TextXAlignment = Enum.TextXAlignment.Left sliderLabel.Parent = sliderFrame

local sliderTrack = Instance.new("Frame") sliderTrack.Size = UDim2.new(1, 0, 0, 6) sliderTrack.Position = UDim2.new(0, 0, 0, 30) sliderTrack.BackgroundColor3 = Color3.fromRGB(50, 50, 50) sliderTrack.BorderSizePixel = 0 sliderTrack.Parent = sliderFrame
local trackCorner = Instance.new("UICorner") trackCorner.CornerRadius = UDim.new(1, 0) trackCorner.Parent = sliderTrack
local sliderFill = Instance.new("Frame") sliderFill.Size = UDim2.new(16/255, 0, 1, 0) sliderFill.BackgroundColor3 = Color3.fromRGB(0, 255, 128) sliderFill.BorderSizePixel = 0 sliderFill.Parent = sliderTrack
local fillCorner = Instance.new("UICorner") fillCorner.CornerRadius = UDim.new(1, 0) fillCorner.Parent = sliderFill
local sliderButton = Instance.new("TextButton") sliderButton.Size = UDim2.new(0, 20, 0, 20) sliderButton.Position = UDim2.new(16/255, -10, 0, -7) sliderButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255) sliderButton.Text = "" sliderButton.Parent = sliderTrack
local btnCorner = Instance.new("UICorner") btnCorner.CornerRadius = UDim.new(1, 0) btnCorner.Parent = sliderButton

local sliding = false
sliderButton.MouseButton1Down:Connect(function() sliding = true end)
UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then sliding = false end end)
UserInputService.InputChanged:Connect(function(input)
    if sliding and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local pos = input.Position.X local absPos = sliderTrack.AbsolutePosition.X local absSize = sliderTrack.AbsoluteSize.X
        local rel = math.clamp((pos - absPos) / absSize, 0, 1)
        sliderFill.Size = UDim2.new(rel, 0, 1, 0) sliderButton.Position = UDim2.new(rel, -10, 0, -7)
        speedValue = math.floor(rel * 254 + 1)
        if currentLang == "TR" then sliderLabel.Text = "Hız Değeri: " .. speedValue .. " (Max: 255)"
        elseif currentLang == "EN" then sliderLabel.Text = "Speed Value: " .. speedValue .. " (Max: 255)"
        else sliderLabel.Text = "Скорость: " .. speedValue .. " (Макс: 255)" end
    end
end)

local noclipActive = false
addToggleButton(pageMovement, "Duvarlardan Geçme (Noclip)", "Noclip", "Проход сквозь стены (Noclip)", function(state) noclipActive = state end)

local infJumpActive = false
addToggleButton(pageMovement, "Sınırsız Zıplama (Air Jump)", "Infinite Jump (Air Jump)", "Бесконечный прыжок", function(state) infJumpActive = state end)

local lowGravityActive = false
addToggleButton(pageMovement, "Düşük Yerçekimi (Moon Jump)", "Low Gravity (Moon Jump)", "Низкая гравитация", function(state) lowGravityActive = state end)

local function createButton(parent, nameTR, nameEN, nameRU, color, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 410, 0, 35)
    btn.BackgroundColor3 = color
    btn.BorderSizePixel = 0
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.TextSize = 13
    btn.Font = Enum.Font.Code
    btn.Text = nameTR
    btn.Parent = parent

    btn:SetAttribute("TextTR", nameTR)
    btn:SetAttribute("TextEN", nameEN)
    btn:SetAttribute("TextRU", nameRU)
    table.insert(translatableElements, btn)

    local corner = Instance.new("UICorner") corner.CornerRadius = UDim.new(0, 4) corner.Parent = btn
    btn.MouseButton1Click:Connect(function() playClickSound() callback() end)
end

createButton(pageMovement, "Blink (Anlık İleri Işınlanma)", "Blink (Instant Forward Teleport)", "Блинк (Телепорт вперед)", Color3.fromRGB(0, 120, 120), function()
    pcall(function()
        local char = localPlayer.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            local root = char.HumanoidRootPart
            root.CFrame = root.CFrame + (camera.CFrame.LookVector * 25)
        end
    end)
end)

UserInputService.JumpRequest:Connect(function()
    if infJumpActive then
        pcall(function()
            local char = localPlayer.Character
            if char and char:FindFirstChildOfClass("Humanoid") then char:FindFirstChildOfClass("Humanoid"):ChangeState(Enum.HumanoidStateType.Jumping) end
        end)
    end
end)

-- FLY SPEED SLIDER
local flySpeedVal = 50
local flySpeedFrame = Instance.new("Frame") flySpeedFrame.Size = UDim2.new(0, 410, 0, 50) flySpeedFrame.BackgroundTransparency = 1 flySpeedFrame.Parent = pageMovement
local flySpeedLabel = Instance.new("TextLabel") flySpeedLabel.Size = UDim2.new(1, 0, 0, 20) flySpeedLabel.BackgroundTransparency = 1 flySpeedLabel.Text = "Uçuş Hızı: 50 (Max: 200)" flySpeedLabel.TextColor3 = Color3.fromRGB(200, 200, 200) flySpeedLabel.TextSize = 12 flySpeedLabel.Font = Enum.Font.Code flySpeedLabel.TextXAlignment = Enum.TextXAlignment.Left flySpeedLabel.Parent = flySpeedFrame

local flyTrack = Instance.new("Frame") flyTrack.Size = UDim2.new(1, 0, 0, 6) flyTrack.Position = UDim2.new(0, 0, 0, 30) flyTrack.BackgroundColor3 = Color3.fromRGB(50, 50, 50) flyTrack.BorderSizePixel = 0 flyTrack.Parent = flySpeedFrame
local ftCorner2 = Instance.new("UICorner") ftCorner2.CornerRadius = UDim.new(1, 0) ftCorner2.Parent = flyTrack
local flyFill = Instance.new("Frame") flyFill.Size = UDim2.new(50/200, 0, 1, 0) flyFill.BackgroundColor3 = Color3.fromRGB(0, 255, 128) flyFill.BorderSizePixel = 0 flyFill.Parent = flyTrack
local ffCorner2 = Instance.new("UICorner") ffCorner2.CornerRadius = UDim.new(1, 0) ffCorner2.Parent = flyFill
local flySliderBtn = Instance.new("TextButton") flySliderBtn.Size = UDim2.new(0, 20, 0, 20) flySliderBtn.Position = UDim2.new(50/200, -10, 0, -7) flySliderBtn.BackgroundColor3 = Color3.fromRGB(255, 255, 255) flySliderBtn.Text = "" flySliderBtn.Parent = flyTrack
local fbCorner2 = Instance.new("UICorner") fbCorner2.CornerRadius = UDim.new(1, 0) fbCorner2.Parent = flySliderBtn

local flySliding = false
flySliderBtn.MouseButton1Down:Connect(function() flySliding = true end)
UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then flySliding = false end end)
UserInputService.InputChanged:Connect(function(input)
    if flySliding and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local pos = input.Position.X local absPos = flyTrack.AbsolutePosition.X local absSize = flyTrack.AbsoluteSize.X
        local rel = math.clamp((pos - absPos) / absSize, 0, 1)
        flyFill.Size = UDim2.new(rel, 0, 1, 0) flySliderBtn.Position = UDim2.new(rel, -10, 0, -7)
        flySpeedVal = math.floor(rel * 190 + 10)
        if currentLang == "TR" then flySpeedLabel.Text = "Uçuş Hızı: " .. flySpeedVal .. " (Max: 200)"
        elseif currentLang == "EN" then flySpeedLabel.Text = "Fly Speed: " .. flySpeedVal .. " (Max: 200)"
        else flySpeedLabel.Text = "Скорость полета: " .. flySpeedVal .. " (Макс: 200)" end
    end
end)

local flyActive = false
addToggleButton(pageMovement, "Fly (Uçuş)", "Fly", "Полет (Fly)", function(state)
    flyActive = state
    pcall(function()
        if flyActive and localPlayer.Character then
            local humanoid = localPlayer.Character:FindFirstChildOfClass("Humanoid")
            if humanoid then humanoid.PlatformStand = true end
        elseif localPlayer.Character then
            local humanoid = localPlayer.Character:FindFirstChildOfClass("Humanoid")
            if humanoid then humanoid.PlatformStand = false end
        end
    end)
end)

-- FREE CAMERA SYSTEM
local freeCamActive = false
local freeCamPos = Vector3.new(0, 50, 0)
local freeCamSpeed = 60

addToggleButton(pageMovement, "Serbest Kamera (Free Camera)", "Free Camera", "Свободная камера", function(state)
    freeCamActive = state
    pcall(function()
        if freeCamActive then
            local char = localPlayer.Character
            if char and char:FindFirstChild("HumanoidRootPart") then
                freeCamPos = char.HumanoidRootPart.Position + Vector3.new(0, 5, 0)
            end
            camera.CameraType = Enum.CameraType.Scriptable
        else
            camera.CameraType = Enum.CameraType.Custom
            camera.CameraSubject = localPlayer.Character and localPlayer.Character:FindFirstChildOfClass("Humanoid")
        end
    end)
end)

RunService.RenderStepped:Connect(function(dt)
    if freeCamActive then
        pcall(function()
            local moveVector = Vector3.new(0, 0, 0)
            if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveVector = moveVector + camera.CFrame.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveVector = moveVector - camera.CFrame.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.A) then moveVector = moveVector - camera.CFrame.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveVector = moveVector + camera.CFrame.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.Space) then moveVector = moveVector + Vector3.new(0, 1, 0) end
            if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then moveVector = moveVector - Vector3.new(0, 1, 0) end

            freeCamPos = freeCamPos + (moveVector * freeCamSpeed * dt)
            camera.CFrame = CFrame.new(freeCamPos, freeCamPos + camera.CFrame.LookVector)
        end)
    end
end)

RunService.RenderStepped:Connect(function()
    if flyActive and localPlayer.Character and not freeCamActive then
        pcall(function()
            local root = localPlayer.Character:FindFirstChild("HumanoidRootPart")
            local humanoid = localPlayer.Character:FindFirstChildOfClass("Humanoid")
            if root and humanoid then
                humanoid.PlatformStand = true
                local camCFrame = camera.CFrame
                local moveDir = humanoid.MoveDirection
                local velocity = Vector3.new(0, 0, 0)
                if moveDir.Magnitude > 0 then 
                    velocity = (camCFrame.LookVector * (moveDir.Z * -1) + camCFrame.RightVector * moveDir.X) * flySpeedVal 
                end
                root.AssemblyLinearVelocity = velocity
                root.CFrame = CFrame.new(root.Position, root.Position + camCFrame.LookVector)
            end
        end)
    end
end)

RunService.Heartbeat:Connect(function()
    pcall(function()
        if localPlayer.Character then
            local humanoid = localPlayer.Character:FindFirstChildOfClass("Humanoid")
            if humanoid and speedActive and not flyActive and not freeCamActive then humanoid.WalkSpeed = speedValue end
            if noclipActive then
                for _, part in pairs(localPlayer.Character:GetDescendants()) do if part:IsA("BasePart") then part.CanCollide = false end end
            end
            if lowGravityActive then workspace.Gravity = 30 else workspace.Gravity = 196.2 end
        end
    end)
end)

-- 4. TP & SPAWNER TAB
createButton(pageWorld, "Güvenli Bölgeye Işınlan (Safe Zone)", "Teleport to Safe Zone", "Телепорт в безопасную зону", Color3.fromRGB(0, 100, 200), function()
    local char = localPlayer.Character if char and char:FindFirstChild("HumanoidRootPart") then char.HumanoidRootPart.CFrame = CFrame.new(0, 150, 0) end
end)

createButton(pageWorld, "Rastgele Kasaya / Sandığa Işınlan", "TP to Random Chest", "ТП к случайному сундуку", Color3.fromRGB(180, 120, 0), function()
    local chests = {}
    for _, obj in pairs(Workspace:GetDescendants()) do
        local name = obj.Name:lower()
        if (name:find("chest") or name:find("kasa") or name:find("box") or name:find("crate")) and obj:IsA("BasePart") then table.insert(chests, obj)
        elseif obj:IsA("ProximityPrompt") and obj.Parent and obj.Parent:IsA("BasePart") then table.insert(chests, obj.Parent) end
    end
    if #chests > 0 then
        local rc = chests[math.random(1, #chests)]
        local char = localPlayer.Character if char and char:FindFirstChild("HumanoidRootPart") then char.HumanoidRootPart.CFrame = rc.CFrame + Vector3.new(0, 3, 0) end
    end
end)
-- OBJECT SPAWNER
local spawnerBoxFrame = Instance.new("Frame")
spawnerBoxFrame.Size = UDim2.new(0, 410, 0, 155)
spawnerBoxFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
spawnerBoxFrame.BorderColor3 = Color3.fromRGB(0, 255, 128)
spawnerBoxFrame.BorderSizePixel = 1
spawnerBoxFrame.Parent = pageWorld

local sbCorner = Instance.new("UICorner") sbCorner.CornerRadius = UDim.new(0, 4) sbCorner.Parent = spawnerBoxFrame

local spawnerTitle = Instance.new("TextLabel")
spawnerTitle.Size = UDim2.new(1, 0, 0, 25)
spawnerTitle.BackgroundTransparency = 1
spawnerTitle.TextColor3 = Color3.fromRGB(0, 255, 128)
spawnerTitle.TextSize = 12
spawnerTitle.Font = Enum.Font.Code
spawnerTitle.Text = "  [ NESNE / MODEL ÜRETİCİ (OTO DÖNGÜ) ]"
spawnerTitle.TextXAlignment = Enum.TextXAlignment.Left
spawnerTitle.Parent = spawnerBoxFrame

local spawnerInput = Instance.new("TextBox")
spawnerInput.Size = UDim2.new(0, 275, 0, 35)
spawnerInput.Position = UDim2.new(0, 8, 0, 30)
spawnerInput.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
spawnerInput.TextColor3 = Color3.fromRGB(255, 255, 255)
spawnerInput.TextSize = 12
spawnerInput.Font = Enum.Font.Code
spawnerInput.PlaceholderText = "Model adı veya ID yaz..."
spawnerInput.Text = ""
spawnerInput.Parent = spawnerBoxFrame

local siCorner = Instance.new("UICorner") siCorner.CornerRadius = UDim.new(0, 4) siCorner.Parent = spawnerInput

local spawnToggleBtn = Instance.new("TextButton")
spawnToggleBtn.Size = UDim2.new(0, 110, 0, 35)
spawnToggleBtn.Position = UDim2.new(0, 290, 0, 30)
spawnToggleBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
spawnToggleBtn.BorderSizePixel = 0
spawnToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
spawnToggleBtn.TextSize = 12
spawnToggleBtn.Font = Enum.Font.Code
spawnToggleBtn.Text = "Üretici [ KAPALI ]"
spawnToggleBtn.Parent = spawnerBoxFrame

local stCorner = Instance.new("UICorner") stCorner.CornerRadius = UDim.new(0, 4) stCorner.Parent = spawnToggleBtn

local delayLabel = Instance.new("TextLabel")
delayLabel.Size = UDim2.new(1, -20, 0, 20)
delayLabel.Position = UDim2.new(0, 10, 0, 72)
delayLabel.BackgroundTransparency = 1
delayLabel.Text = "Üretim Gecikmesi: 0.5s (500ms)"
delayLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
delayLabel.TextSize = 11
delayLabel.Font = Enum.Font.Code
delayLabel.TextXAlignment = Enum.TextXAlignment.Left
delayLabel.Parent = spawnerBoxFrame

local delayTrack = Instance.new("Frame")
delayTrack.Size = UDim2.new(1, -20, 0, 6)
delayTrack.Position = UDim2.new(0, 10, 0, 100)
delayTrack.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
delayTrack.BorderSizePixel = 0
delayTrack.Parent = spawnerBoxFrame

local dtCorner = Instance.new("UICorner") dtCorner.CornerRadius = UDim.new(1, 0) dtCorner.Parent = delayTrack

local delayFill = Instance.new("Frame")
delayFill.Size = UDim2.new(0.5, 0, 1, 0)
delayFill.BackgroundColor3 = Color3.fromRGB(0, 255, 128)
delayFill.BorderSizePixel = 0
delayFill.Parent = delayTrack

local dfCorner = Instance.new("UICorner") dfCorner.CornerRadius = UDim.new(1, 0) dfCorner.Parent = delayFill

local delayButton = Instance.new("TextButton")
delayButton.Size = UDim2.new(0, 16, 0, 16)
delayButton.Position = UDim2.new(0.5, -8, 0, -5)
delayButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
delayButton.Text = ""
delayButton.Parent = delayTrack

local dbCorner = Instance.new("UICorner") dbCorner.CornerRadius = UDim.new(1, 0) dbCorner.Parent = delayButton

local spawnActiveLoop = false
local spawnDelayTime = 0.5
local delaySliding = false

delayButton.MouseButton1Down:Connect(function() delaySliding = true end)
UserInputService.InputEnded:Connect(function() delaySliding = false end)
UserInputService.InputChanged:Connect(function(input)
    if delaySliding and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local pos = input.Position.X local absPos = delayTrack.AbsolutePosition.X local absSize = delayTrack.AbsoluteSize.X
        local rel = math.clamp((pos - absPos) / absSize, 0, 1)
        delayFill.Size = UDim2.new(rel, 0, 1, 0) delayButton.Position = UDim2.new(rel, -8, 0, -5)
        spawnDelayTime = math.round((rel * 1.95 + 0.05) * 100) / 100
        if currentLang == "TR" then delayLabel.Text = string.format("Üretim Gecikmesi: %.2fs (%dms)", spawnDelayTime, math.floor(spawnDelayTime * 1000))
        elseif currentLang == "EN" then delayLabel.Text = string.format("Spawn Delay: %.2fs (%dms)", spawnDelayTime, math.floor(spawnDelayTime * 1000))
        else delayLabel.Text = string.format("Задержка спавна: %.2fs (%dms)", spawnDelayTime, math.floor(spawnDelayTime * 1000)) end
    end
end)

local function spawnTargetObject(nameOrId)
    pcall(function()
        local char = localPlayer.Character
        local spawnPos = char and char:FindFirstChild("HumanoidRootPart") and (char.HumanoidRootPart.CFrame + char.HumanoidRootPart.CFrame.LookVector * 5) or CFrame.new(0, 10, 0)
        
        local foundObj = ReplicatedStorage:FindFirstChild(nameOrId) or Workspace:FindFirstChild(nameOrId)
        if foundObj then
            local clone = foundObj:Clone()
            if clone:IsA("Model") then
                if clone.PrimaryPart then
                    clone:SetPrimaryPartCFrame(spawnPos)
                else
                    local part = clone:FindFirstChildWhichIsA("BasePart")
                    if part then part.CFrame = spawnPos end
                end
                clone.Parent = Workspace
            elseif clone:IsA("BasePart") then
                clone.CFrame = spawnPos
                clone.Parent = Workspace
            end
        else
            local numId = tonumber(nameOrId)
            if numId then
                local success, model = pcall(function()
                    return InsertService:LoadAsset(numId)
                end)
                if success and model then
                    for _, child in pairs(model:GetChildren()) do
                        child.Parent = Workspace
                        if child:IsA("Model") then
                            if child.PrimaryPart then child:SetPrimaryPartCFrame(spawnPos) end
                        elseif child:IsA("BasePart") then
                            child.CFrame = spawnPos
                        end
                    end
                else
                    local part = Instance.new("Part")
                    part.Size = Vector3.new(4, 4, 4)
                    part.Position = spawnPos.Position + Vector3.new(0, 3, 0)
                    part.BrickColor = BrickColor.random()
                    part.Material = Enum.Material.Neon
                    part.Name = tostring(nameOrId)
                    part.Parent = Workspace
                end
            else
                local part = Instance.new("Part")
                part.Size = Vector3.new(4, 4, 4)
                part.Position = spawnPos.Position + Vector3.new(0, 3, 0)
                part.BrickColor = BrickColor.random()
                part.Material = Enum.Material.Neon
                part.Name = nameOrId
                part.Parent = Workspace
            end
        end
    end)
end

spawnToggleBtn.MouseButton1Click:Connect(function()
    playClickSound()
    spawnActiveLoop = not spawnActiveLoop
    if spawnActiveLoop then
        spawnToggleBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 60)
        spawnToggleBtn.Text = (currentLang == "TR") and "Üretici [ AÇIK ]" or ((currentLang == "EN") and "Spawner [ ON ]" or "Спавнер [ ВКЛ ]")
        task.spawn(function()
            while spawnActiveLoop do
                local textVal = spawnerInput.Text
                if textVal ~= "" then spawnTargetObject(textVal) end
                task.wait(spawnDelayTime)
            end
        end)
    else
        spawnToggleBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        spawnToggleBtn.Text = (currentLang == "TR") and "Üretici [ KAPALI ]" or ((currentLang == "EN") and "Spawner [ OFF ]" or "Спавнер [ ВЫКЛ ]")
    end
end)

-- CHAT MESSAGE SENDER & AUTO LOOP (SOHBET SPAMMER)
local chatBoxFrame = Instance.new("Frame")
chatBoxFrame.Size = UDim2.new(0, 410, 0, 115)
chatBoxFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
chatBoxFrame.BorderColor3 = Color3.fromRGB(0, 255, 128)
chatBoxFrame.BorderSizePixel = 1
chatBoxFrame.Parent = pageWorld

local cbCorner = Instance.new("UICorner") cbCorner.CornerRadius = UDim.new(0, 4) cbCorner.Parent = chatBoxFrame

local chatTitle = Instance.new("TextLabel")
chatTitle.Size = UDim2.new(1, 0, 0, 25)
chatTitle.BackgroundTransparency = 1
chatTitle.TextColor3 = Color3.fromRGB(0, 255, 128)
chatTitle.TextSize = 12
chatTitle.Font = Enum.Font.Code
chatTitle.Text = "  [ OTOMATİK SOHBET MESAJ GÖNDERİCİ (SOHBET SPAM) ]"
chatTitle.TextXAlignment = Enum.TextXAlignment.Left
chatTitle.Parent = chatBoxFrame

local chatInput = Instance.new("TextBox")
chatInput.Size = UDim2.new(0, 275, 0, 35)
chatInput.Position = UDim2.new(0, 8, 0, 30)
chatInput.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
chatInput.TextColor3 = Color3.fromRGB(255, 255, 255)
chatInput.TextSize = 12
chatInput.Font = Enum.Font.Code
chatInput.PlaceholderText = "Sohbete yazılacak mesajı yaz..."
chatInput.Text = ""
chatInput.Parent = chatBoxFrame

local ciCorner = Instance.new("UICorner") ciCorner.CornerRadius = UDim.new(0, 4) ciCorner.Parent = chatInput

local sendChatBtn = Instance.new("TextButton")
sendChatBtn.Size = UDim2.new(0, 110, 0, 35)
sendChatBtn.Position = UDim2.new(0, 290, 0, 30)
sendChatBtn.BackgroundColor3 = Color3.fromRGB(0, 90, 60)
sendChatBtn.BorderSizePixel = 0
sendChatBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
sendChatBtn.TextSize = 12
sendChatBtn.Font = Enum.Font.Code
sendChatBtn.Text = "Sohbet Döngü [ KAPALI ]"
sendChatBtn.Parent = chatBoxFrame

local scCorner = Instance.new("UICorner") scCorner.CornerRadius = UDim.new(0, 4) scCorner.Parent = sendChatBtn

local function sendChatMessage(msg)
    pcall(function()
        if TextChatService.ChatVersion == Enum.ChatVersion.TextChatService then
            local channel = TextChatService.TextChannels.RBXGeneral
            if channel then channel:SendAsync(msg) end
        else
            local sayEvent = game:GetService("ReplicatedStorage"):FindFirstChild("DefaultChatSystemChatEvents")
            if sayEvent and sayEvent:FindFirstChild("SayMessageRequest") then
                sayEvent.SayMessageRequest:FireServer(msg, "All")
            end
        end
    end)
end

local chatLoopActive = false
sendChatBtn.MouseButton1Click:Connect(function()
    playClickSound()
    chatLoopActive = not chatLoopActive
    if chatLoopActive then
        sendChatBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 60)
        sendChatBtn.Text = (currentLang == "TR") and "Sohbet Döngü [ AÇIK ]" or ((currentLang == "EN") and "Chat Loop [ ON ]" or "Чат Цикл [ ВКЛ ]")
        task.spawn(function()
            while chatLoopActive do
                local msg = chatInput.Text
                if msg ~= "" then sendChatMessage(msg) end
                task.wait(2)
            end
        end)
    else
        sendChatBtn.BackgroundColor3 = Color3.fromRGB(0, 90, 60)
        sendChatBtn.Text = (currentLang == "TR") and "Sohbet Döngü [ KAPALI ]" or ((currentLang == "EN") and "Chat Loop [ OFF ]" or "Чат Цикл [ ВЫКЛ ]")
    end
end)

-- 5. SETTINGS TAB
local function createSettingsHeader(textTR, textEN, textRU)
    local lbl = Instance.new("TextLabel")
    lbl.Size = UDim2.new(0, 410, 0, 25)
    lbl.BackgroundTransparency = 1
    lbl.TextColor3 = Color3.fromRGB(0, 255, 128)
    lbl.TextSize = 14
    lbl.Font = Enum.Font.Code
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Text = "--- " .. textTR .. " ---"
    lbl.Parent = pageSettings

    lbl:SetAttribute("TextTR", textTR)
    lbl:SetAttribute("TextEN", textEN)
    lbl:SetAttribute("TextRU", textRU)
    table.insert(translatableElements, lbl)
    return lbl
end

createSettingsHeader("OTOMATİK IŞINLANMA (YOU VS HULK)", "AUTO TELEPORT (YOU VS HULK)", "АВТО-ТЕЛЕПОРТ (YOU VS HULK)")

addToggleButton(pageSettings, "Otomatik You vs Hulk'a Git", "Auto Teleport to You vs Hulk", "Авто-телепорт в You vs Hulk", function(state)
    if state and game.PlaceId ~= TARGET_PLACE_ID then
        pcall(function() TeleportService:Teleport(TARGET_PLACE_ID, localPlayer) end)
    end
end)

-- MÜZİK ÇALAR & ŞARKI SEÇİMİ
createSettingsHeader("MÜZİK ÇALAR VE ŞARKI SEÇİMİ", "MUSIC PLAYER & TRACK SELECTOR", "МУЗЫКАЛЬНЫЙ ПЛЕЕР")

local backgroundMusic = Instance.new("Sound")
backgroundMusic.Name = "UltimateBackgroundMusic"
backgroundMusic.SoundId = "rbxassetid://142376088"
backgroundMusic.Volume = 0.5
backgroundMusic.Looped = true
backgroundMusic.Parent = SoundService

addToggleButton(pageSettings, "Müziği Başlat / Durdur", "Play / Stop Music", "Воспроизвести / Остановить музыку", function(state)
    if state then
        pcall(function() backgroundMusic:Play() end)
    else
        pcall(function() backgroundMusic:Stop() end)
    end
end)

local musicPickerFrame = Instance.new("Frame")
musicPickerFrame.Size = UDim2.new(0, 410, 0, 150)
musicPickerFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
musicPickerFrame.BorderColor3 = Color3.fromRGB(0, 255, 128)
musicPickerFrame.BorderSizePixel = 1
musicPickerFrame.Parent = pageSettings

local mpLayout = Instance.new("UIGridLayout")
mpLayout.CellSize = UDim2.new(0, 195, 0, 32)
mpLayout.CellPadding = UDim2.new(0, 10, 0, 8)
mpLayout.Parent = musicPickerFrame

local function addMusicOption(songName, assetId)
    local b = Instance.new("TextButton")
    b.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    b.Text = "🎵 " .. songName
    b.TextColor3 = Color3.fromRGB(255, 255, 255)
    b.TextSize = 11
    b.Font = Enum.Font.Code
    b.Parent = musicPickerFrame
    
    local c = Instance.new("UICorner") c.CornerRadius = UDim.new(0, 4) c.Parent = b
    
    b.MouseButton1Click:Connect(function()
        playClickSound()
        pcall(function()
            local isPlaying = backgroundMusic.IsPlaying
            backgroundMusic:Stop()
            backgroundMusic.SoundId = "rbxassetid://" .. tostring(assetId)
            if isPlaying then
                backgroundMusic:Play()
            end
        end)
    end)
end

addMusicOption("It's Raining Tacos", 142376088)
addMusicOption("OMFG - HELLO", 110788401793874)
addMusicOption("You broke my heart", 71792887446304)
addMusicOption("No name Triad", 10494950381281)

-- Müzik Ses Seviyesi Slider
local musicSliderFrame = Instance.new("Frame") musicSliderFrame.Size = UDim2.new(0, 410, 0, 50) musicSliderFrame.BackgroundTransparency = 1 musicSliderFrame.Parent = pageSettings
local musicSliderLabel = Instance.new("TextLabel") musicSliderLabel.Size = UDim2.new(1, 0, 0, 20) musicSliderLabel.BackgroundTransparency = 1 musicSliderLabel.Text = "Müzik Ses Seviyesi: 50%" musicSliderLabel.TextColor3 = Color3.fromRGB(200, 200, 200) musicSliderLabel.TextSize = 12 musicSliderLabel.Font = Enum.Font.Code musicSliderLabel.TextXAlignment = Enum.TextXAlignment.Left musicSliderLabel.Parent = musicSliderFrame

local musicSliderTrack = Instance.new("Frame") musicSliderTrack.Size = UDim2.new(1, 0, 0, 6) musicSliderTrack.Position = UDim2.new(0, 0, 0, 30) musicSliderTrack.BackgroundColor3 = Color3.fromRGB(50, 50, 50) musicSliderTrack.BorderSizePixel = 0 musicSliderTrack.Parent = musicSliderFrame
local mtCorner = Instance.new("UICorner") mtCorner.CornerRadius = UDim.new(1, 0) mtCorner.Parent = musicSliderTrack
local musicSliderFill = Instance.new("Frame") musicSliderFill.Size = UDim2.new(0.5, 0, 1, 0) musicSliderFill.BackgroundColor3 = Color3.fromRGB(0, 255, 128) musicSliderFill.BorderSizePixel = 0 musicSliderFill.Parent = musicSliderTrack
local mfCorner = Instance.new("UICorner") mfCorner.CornerRadius = UDim.new(1, 0) mfCorner.Parent = musicSliderFill
local musicSliderBtn = Instance.new("TextButton") musicSliderBtn.Size = UDim2.new(0, 20, 0, 20) musicSliderBtn.Position = UDim2.new(0.5, -10, 0, -7) musicSliderBtn.BackgroundColor3 = Color3.fromRGB(255, 255, 255) musicSliderBtn.Text = "" musicSliderBtn.Parent = musicSliderTrack
local mbCorner = Instance.new("UICorner") mbCorner.CornerRadius = UDim.new(1, 0) mbCorner.Parent = musicSliderBtn

local musicSliding = false
musicSliderBtn.MouseButton1Down:Connect(function() musicSliding = true end)
UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then musicSliding = false end end)
UserInputService.InputChanged:Connect(function(input)
    if musicSliding and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local pos = input.Position.X local absPos = musicSliderTrack.AbsolutePosition.X local absSize = musicSliderTrack.AbsoluteSize.X
        local rel = math.clamp((pos - absPos) / absSize, 0, 1)
        musicSliderFill.Size = UDim2.new(rel, 0, 1, 0) musicSliderBtn.Position = UDim2.new(rel, -10, 0, -7)
        backgroundMusic.Volume = rel
        local volPercent = math.floor(rel * 100)
        if currentLang == "TR" then musicSliderLabel.Text = "Müzik Ses Seviyesi: " .. volPercent .. "%"
        elseif currentLang == "EN" then musicSliderLabel.Text = "Music Volume: " .. volPercent .. "%"
        else musicSliderLabel.Text = "Громкость музыки: " .. volPercent .. "%" end
    end
end)

createSettingsHeader("DİL SEÇİMİ / LANGUAGE", "LANGUAGE / DİL SEÇİMİ", "ВЫБОР ЯЗЫКА / LANGUAGE")

local langPickerFrame = Instance.new("Frame") langPickerFrame.Size = UDim2.new(0, 410, 0, 45) langPickerFrame.BackgroundTransparency = 1 langPickerFrame.Parent = pageSettings
local lpLayout = Instance.new("UIListLayout") lpLayout.FillDirection = Enum.FillDirection.Horizontal lpLayout.Padding = UDim.new(0, 10) lpLayout.Parent = langPickerFrame

-- GLOBAL TRANSLATION UPDATE FUNCTION
local panicBtn, soundBtn, perfBtn, infoLabel

local function updateLanguage(langCode)
    currentLang = langCode
    if langCode == "TR" then
        titleLabel.Text = "  YOU VS HULK | ULTIMATE MOD MENU (v9.49 [TÜRKÇE])"
        if panicBtn then panicBtn.Text = "Acil Durum Tuşu (Menüyü Gizle)" end
        if soundBtn then soundBtn.Text = "Ses Efektleri: Açık" end
        if perfBtn then perfBtn.Text = "Performans Artırıcı [ KAPALI ]" end
        if infoLabel then
            infoLabel.Text = [[=== MENÜ BİLGİSİ ===

Yazar / Oluşturan: black
Sürüm: v9.49 Ultimate (Hulk Fling Eklendi)
Oyun: You vs Hulk
--- DURUM ---
• Hulk Fling (Oyuncu Fırlatma) özelliği aktif edildi.
]]
        end
    elseif langCode == "EN" then
        titleLabel.Text = "  YOU VS HULK | ULTIMATE MOD MENU (v9.49 [ENGLISH])"
        if panicBtn then panicBtn.Text = "Panic Button (Hide Menu)" end
        if soundBtn then soundBtn.Text = "Sound Effects: On" end
        if perfBtn then perfBtn.Text = "Performance Booster [ OFF ]" end
        if infoLabel then
            infoLabel.Text = [[=== MENU INFORMATION ===

Author / Creator: black
Version: v9.49 Ultimate (Hulk Fling Added)
Game: You vs Hulk

--- STATUS ---
• Hulk Fling (Player Launcher) feature enabled.
]]
        end
    else
        titleLabel.Text = "  YOU VS HULK | ULTIMATE MOD MENU (v9.49 [РУССКИЙ])"
        if panicBtn then panicBtn.Text = "Кнопка паники (Скрыть меню)" end
        if soundBtn then soundBtn.Text = "Звуковые эффекты: ВКЛ" end
        if perfBtn then perfBtn.Text = "Ускоритель [ ВЫКЛ ]" end
        if infoLabel then
            infoLabel.Text = [[=== ИНФОРМАЦИЯ О МЕНЮ ===

Автор / Создатель: black
Версия: v9.49 Ultimate (Добавлен Hulk Fling)
Игра: You vs Hulk

--- СТАТУС ---
• Функция Hulk Fling включена.
]]
        end
    end

    for _, btn in pairs(tabButtons) do
        if langCode == "TR" then btn.Text = btn:GetAttribute("NameTR")
        elseif langCode == "EN" then btn.Text = btn:GetAttribute("NameEN")
        else btn.Text = btn:GetAttribute("NameRU") end
    end

    for _, el in pairs(translatableElements) do
        if el:IsA("TextButton") then
            local base = (langCode == "TR") and el:GetAttribute("TextTR") or ((langCode == "EN") and el:GetAttribute("TextEN") or el:GetAttribute("TextRU"))
            if el.Text:find("AÇIK") or el.Text:find("ON") or el.Text:find("ВКЛ") then
                local onSuffix = (langCode == "TR
