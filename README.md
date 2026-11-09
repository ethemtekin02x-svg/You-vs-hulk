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

local localPlayer = Players.LocalPlayer
local camera = workspace.CurrentCamera
local mouse = localPlayer:GetMouse()

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

-- 1. TOGGLE BUTTON (Kare ve Kompakt Tasarım)
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

-- KLAVYE KISAYOLU: INSERT TUŞU İLE MENÜYÜ AÇ/KAPAT
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
titleLabel.Text = "  YOU VS HULK — ULTIMATE MOD MENU (v9.20)"
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
local function createTab(name)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 45)
    btn.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    btn.BorderSizePixel = 0
    btn.TextColor3 = Color3.fromRGB(180, 180, 180)
    btn.TextSize = 13
    btn.Font = Enum.Font.Code
    btn.Text = name
    btn.Parent = tabListFrame

    local page = Instance.new("ScrollingFrame")
    page.Size = UDim2.new(1, 0, 1, 0)
    page.BackgroundTransparency = 1
    page.Visible = false
    page.CanvasSize = UDim2.new(0, 0, 0, 850)
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

local pageMain = createTab("Main & Combat")
local pageVisuals = createTab("ESP (Visuals)")
local pageMovement = createTab("Movement")
local pageWorld = createTab("TP & Spawner")
local pageSettings = createTab("Settings")
local pageInfo = createTab("Info")

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
watermarkText.Text = "black | v9.20 Ultimate"
watermarkText.Parent = watermarkFrame

-- UI HELPER FUNCTIONS
local function addToggleButton(parent, text, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 410, 0, 35)
    btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    btn.BorderSizePixel = 0
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.TextSize = 13
    btn.Font = Enum.Font.Code
    btn.Text = text .. " [ OFF ]"
    btn.Parent = parent

    local c = Instance.new("UICorner") c.CornerRadius = UDim.new(0, 4) c.Parent = btn

    local active = false
    btn.MouseButton1Click:Connect(function()
        playClickSound()
        active = not active
        if active then
            btn.BackgroundColor3 = Color3.fromRGB(0, 120, 60)
            btn.Text = text .. " [ ON ]"
        else
            btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
            btn.Text = text .. " [ OFF ]"
        end
        callback(active)
    end)
    return btn
end

local function createFeatureWidget(featureName, callback)
    local widget = Instance.new("TextButton")
    widget.Size = UDim2.new(0, 95, 0, 45)
    widget.Position = UDim2.new(0.5, -47, 0.5, -22)
    widget.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    widget.BorderColor3 = Color3.fromRGB(0, 255, 128)
    widget.BorderSizePixel = 2
    widget.TextColor3 = Color3.fromRGB(255, 255, 255)
    widget.TextSize = 10
    widget.Font = Enum.Font.Code
    widget.Text = featureName .. "\n[ OFF ]"
    widget.Parent = screenGui
    
    local c = Instance.new("UICorner") c.CornerRadius = UDim.new(0, 4) c.Parent = widget
    makeDraggable(widget)
    
    local active = false
    local clicks = 0
    widget.MouseButton1Click:Connect(function()
        playClickSound()
        clicks = clicks + 1
        task.delay(0.3, function() clicks = 0 end)
        if clicks >= 2 then widget:Destroy() return end
        active = not active
        if active then
            widget.BackgroundColor3 = Color3.fromRGB(0, 100, 50)
            widget.Text = featureName .. "\n[ ON ] (Dbl-tap del)"
        else
            widget.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
            widget.Text = featureName .. "\n[ OFF ] (Dbl-tap del)"
        end
        callback(active)
    end)
end

-- 1. MAIN & COMBAT
local function isCharacterHulk(char)
    if not char then return false end
    for _, obj in pairs(char:GetDescendants()) do
        if obj.Name:lower():find("hulk") then return true end
    end
    local root = char:FindFirstChild("HumanoidRootPart")
    return root and root.Size.Y > 3.2
end

local aimbotActive = false
addToggleButton(pageMain, "Hulk Head Aimbot", function(state) aimbotActive = state end)

local triggerbotActive = false
addToggleButton(pageMain, "Auto Triggerbot (Auto Shoot)", function(state) triggerbotActive = state end)

local fovLockActive = false
addToggleButton(pageMain, "FOV Target Lock", function(state) fovLockActive = state end)

local hitboxActive = false
addToggleButton(pageMain, "Hulk Hitbox Extender (Big Target)", function(state)
    hitboxActive = state
end)

RunService.RenderStepped:Connect(function()
    if hitboxActive then
        pcall(function()
            for _, player in pairs(Players:GetPlayers()) do
                if player ~= localPlayer and player.Character then
                    local char = player.Character
                    if isCharacterHulk(char) then
                        local head = char:FindFirstChild("Head")
                        local torso = char:FindFirstChild("UpperTorso") or char:FindFirstChild("Torso")
                        if head then
                            head.Size = Vector3.new(6, 6, 6)
                            head.Transparency = 0.6
                            head.CanCollide = false
                        end
                        if torso then
                            torso.Size = Vector3.new(7, 8, 7)
                            torso.Transparency = 0.8
                        end
                    end
                end
            end
        end)
    end
end)

local maxCombatDist = 150
local widgetBtn1 = Instance.new("TextButton")
widgetBtn1.Size = UDim2.new(0, 410, 0, 35)
widgetBtn1.BackgroundColor3 = Color3.fromRGB(0, 80, 80)
widgetBtn1.BorderSizePixel = 0
widgetBtn1.TextColor3 = Color3.fromRGB(255, 255, 255)
widgetBtn1.TextSize = 13
widgetBtn1.Font = Enum.Font.Code
widgetBtn1.Text = "Spawn Aimbot Floating Widget"
widgetBtn1.Parent = pageMain
local wc1 = Instance.new("UICorner") wc1.CornerRadius = UDim.new(0, 4) wc1.Parent = widgetBtn1
widgetBtn1.MouseButton1Click:Connect(function()
    playClickSound()
    createFeatureWidget("Aimbot Quick", function(state) aimbotActive = state end)
end)

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
                            local rayParams = RaycastParams.new()
                            rayParams.FilterDescendantsInstances = {localPlayer.Character, camera}
                            rayParams.FilterType = Enum.RaycastFilterType.Blacklist
                            rayParams.IgnoreWater = true
                            
                            local rayDirection = (head.Position - camera.CFrame.Position)
                            local rayResult = workspace:Raycast(camera.CFrame.Position, rayDirection, rayParams)
                            
                            local isVisible = true
                            if rayResult and not rayResult.Instance:IsDescendantOf(char) then
                                isVisible = false
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
                                
                                if passFov and dist < shortestDist then
                                    shortestDist = dist
                                    targetHulk = head
                                end
                            end
                        end
                    end
                end
            end
        end
    end)
    
    if aimbotActive and targetHulk then
        camera.CFrame = CFrame.new(camera.CFrame.Position, targetHulk.Position)
    end
end)

RunService.RenderStepped:Connect(function()
    if triggerbotActive then
        pcall(function()
            local mouseTarget = mouse.Target
            if mouseTarget and mouseTarget.Parent then
                local char = mouseTarget.Parent
                if isCharacterHulk(char) and char:FindFirstChild("Humanoid") and char.Humanoid.Health > 0 then
                    mouse1click()
                end
            end
        end)
    end
end)
-- 2. ESP TAB
local espKalanlarActive = false
local espHulkActive = false
addToggleButton(pageVisuals, "Players ESP", function(state) espKalanlarActive = state end)
addToggleButton(pageVisuals, "Hulk ESP", function(state) espHulkActive = state end)

local fovFrame = Instance.new("Frame")
fovFrame.Size = UDim2.new(0, 200, 0, 200)
fovFrame.Position = UDim2.new(0.5, -100, 0.5, -100)
fovFrame.BackgroundTransparency = 1
fovFrame.Visible = false
fovFrame.Parent = screenGui
local fovCorner = Instance.new("UICorner") fovCorner.CornerRadius = UDim.new(1, 0) fovCorner.Parent = fovFrame
local fovStroke = Instance.new("UIStroke") fovStroke.Thickness = 1.5 fovStroke.Color = Color3.fromRGB(0, 255, 128) fovStroke.Parent = fovFrame

addToggleButton(pageVisuals, "FOV Circle", function(state) fovFrame.Visible = state end)

local crosshairFrame = Instance.new("Frame")
crosshairFrame.Size = UDim2.new(0, 10, 0, 10)
crosshairFrame.Position = UDim2.new(0.5, -5, 0.5, -5)
crosshairFrame.BackgroundTransparency = 1
crosshairFrame.Visible = false
crosshairFrame.Parent = screenGui

local c1 = Instance.new("Frame") c1.Size = UDim2.new(1, 0, 0, 2) c1.Position = UDim2.new(0, 0, 0.5, -1) c1.BackgroundColor3 = Color3.fromRGB(0, 255, 128) c1.BorderSizePixel = 0 c1.Parent = crosshairFrame
local c2 = Instance.new("Frame") c2.Size = UDim2.new(0, 2, 1, 0) c2.Position = UDim2.new(0.5, -1, 0, 0) c2.BackgroundColor3 = Color3.fromRGB(0, 255, 128) c2.BorderSizePixel = 0 c2.Parent = crosshairFrame

addToggleButton(pageVisuals, "Custom Crosshair", function(state) crosshairFrame.Visible = state end)

local widgetBtn2 = Instance.new("TextButton")
widgetBtn2.Size = UDim2.new(0, 410, 0, 35)
widgetBtn2.BackgroundColor3 = Color3.fromRGB(0, 80, 80)
widgetBtn2.BorderSizePixel = 0
widgetBtn2.TextColor3 = Color3.fromRGB(255, 255, 255)
widgetBtn2.TextSize = 13
widgetBtn2.Font = Enum.Font.Code
widgetBtn2.Text = "Spawn ESP Floating Widget"
widgetBtn2.Parent = pageVisuals
local wc2 = Instance.new("UICorner") wc2.CornerRadius = UDim.new(0, 4) wc2.Parent = widgetBtn2
widgetBtn2.MouseButton1Click:Connect(function()
    playClickSound()
    createFeatureWidget("ESP Quick", function(state) espHulkActive = state espKalanlarActive = state end)
end)

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
                        txt.Name = "ESPText"
                        txt.Size = UDim2.new(1, 0, 1, 0)
                        txt.BackgroundTransparency = 1
                        txt.TextColor3 = Color3.fromRGB(255, 255, 255)
                        txt.TextStrokeTransparency = 0
                        txt.TextSize = 12
                        txt.Font = Enum.Font.Code
                        txt.Parent = billboard
                        billboard.Parent = char
                    end
                    local textLabel = billboard:FindFirstChild("ESPText")
                    local show = false
                    if isHulk and espHulkActive then
                        textLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
                        textLabel.Text = "HULK\n[" .. dist .. "m]"
                        show = true
                    elseif not isHulk and espKalanlarActive then
                        textLabel.TextColor3 = Color3.fromRGB(255, 80, 80)
                        textLabel.Text = player.Name .. "\n[" .. dist .. "m]"
                        show = true
                    end
                    billboard.Enabled = show
                end
            end
        end
    end)
end)

-- 3. MOVEMENT TAB
local speedValue = 16
local speedActive = false
addToggleButton(pageMovement, "Speed Hack Active", function(state) speedActive = state end)

local sliderFrame = Instance.new("Frame")
sliderFrame.Size = UDim2.new(0, 410, 0, 50)
sliderFrame.BackgroundTransparency = 1
sliderFrame.Parent = pageMovement

local sliderLabel = Instance.new("TextLabel")
sliderLabel.Size = UDim2.new(1, 0, 0, 20)
sliderLabel.BackgroundTransparency = 1
sliderLabel.Text = "Speed Value: 16 (Max: 255)"
sliderLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
sliderLabel.TextSize = 12
sliderLabel.Font = Enum.Font.Code
sliderLabel.TextXAlignment = Enum.TextXAlignment.Left
sliderLabel.Parent = sliderFrame

local sliderTrack = Instance.new("Frame")
sliderTrack.Size = UDim2.new(1, 0, 0, 6)
sliderTrack.Position = UDim2.new(0, 0, 0, 30)
sliderTrack.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
sliderTrack.BorderSizePixel = 0
sliderTrack.Parent = sliderFrame

local trackCorner = Instance.new("UICorner") trackCorner.CornerRadius = UDim.new(1, 0) trackCorner.Parent = sliderTrack

local sliderFill = Instance.new("Frame")
sliderFill.Size = UDim2.new(16/255, 0, 1, 0)
sliderFill.BackgroundColor3 = Color3.fromRGB(0, 255, 128)
sliderFill.BorderSizePixel = 0
sliderFill.Parent = sliderTrack

local fillCorner = Instance.new("UICorner") fillCorner.CornerRadius = UDim.new(1, 0) fillCorner.Parent = sliderFill

local sliderButton = Instance.new("TextButton")
sliderButton.Size = UDim2.new(0, 20, 0, 20)
sliderButton.Position = UDim2.new(16/255, -10, 0, -7)
sliderButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
sliderButton.Text = ""
sliderButton.Parent = sliderTrack

local btnCorner = Instance.new("UICorner") btnCorner.CornerRadius = UDim.new(1, 0) btnCorner.Parent = sliderButton

local sliding = false
sliderButton.MouseButton1Down:Connect(function() sliding = true end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        sliding = false
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if sliding and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local pos = input.Position.X
        local absPos = sliderTrack.AbsolutePosition.X
        local absSize = sliderTrack.AbsoluteSize.X
        local rel = math.clamp((pos - absPos) / absSize, 0, 1)
        
        sliderFill.Size = UDim2.new(rel, 0, 1, 0)
        sliderButton.Position = UDim2.new(rel, -10, 0, -7)
        
        speedValue = math.floor(rel * 254 + 1)
        sliderLabel.Text = "Speed Value: " .. speedValue .. " (Max: 255)"
    end
end)

local noclipActive = false
addToggleButton(pageMovement, "Noclip", function(state) noclipActive = state end)

local infJumpActive = false
addToggleButton(pageMovement, "Infinite Jump (Air Jump)", function(state) infJumpActive = state end)

UserInputService.JumpRequest:Connect(function()
    if infJumpActive then
        pcall(function()
            local char = localPlayer.Character
            if char and char:FindFirstChildOfClass("Humanoid") then
                char:FindFirstChildOfClass("Humanoid"):ChangeState(Enum.HumanoidStateType.Jumping)
            end
        end)
    end
end)
-- FLY HIZ AYARI (FLY SPEED SLIDER)
local flySpeedVal = 50
local flySpeedFrame = Instance.new("Frame")
flySpeedFrame.Size = UDim2.new(0, 410, 0, 50)
flySpeedFrame.BackgroundTransparency = 1
flySpeedFrame.Parent = pageMovement

local flySpeedLabel = Instance.new("TextLabel")
flySpeedLabel.Size = UDim2.new(1, 0, 0, 20)
flySpeedLabel.BackgroundTransparency = 1
flySpeedLabel.Text = "Fly Speed: 50 (Max: 200)"
flySpeedLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
flySpeedLabel.TextSize = 12
flySpeedLabel.Font = Enum.Font.Code
flySpeedLabel.TextXAlignment = Enum.TextXAlignment.Left
flySpeedLabel.Parent = flySpeedFrame

local flyTrack = Instance.new("Frame")
flyTrack.Size = UDim2.new(1, 0, 0, 6)
flyTrack.Position = UDim2.new(0, 0, 0, 30)
flyTrack.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
flyTrack.BorderSizePixel = 0
flyTrack.Parent = flySpeedFrame

local ftCorner2 = Instance.new("UICorner") ftCorner2.CornerRadius = UDim.new(1, 0) ftCorner2.Parent = flyTrack

local flyFill = Instance.new("Frame")
flyFill.Size = UDim2.new(50/200, 0, 1, 0)
flyFill.BackgroundColor3 = Color3.fromRGB(0, 255, 128)
flyFill.BorderSizePixel = 0
flyFill.Parent = flyTrack

local ffCorner2 = Instance.new("UICorner") ffCorner2.CornerRadius = UDim.new(1, 0) ffCorner2.Parent = flyFill

local flySliderBtn = Instance.new("TextButton")
flySliderBtn.Size = UDim2.new(0, 20, 0, 20)
flySliderBtn.Position = UDim2.new(50/200, -10, 0, -7)
flySliderBtn.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
flySliderBtn.Text = ""
flySliderBtn.Parent = flyTrack

local fbCorner2 = Instance.new("UICorner") fbCorner2.CornerRadius = UDim.new(1, 0) fbCorner2.Parent = flySliderBtn

local flySliding = false
flySliderBtn.MouseButton1Down:Connect(function() flySliding = true end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        flySliding = false
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if flySliding and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local pos = input.Position.X
        local absPos = flyTrack.AbsolutePosition.X
        local absSize = flyTrack.AbsoluteSize.X
        local rel = math.clamp((pos - absPos) / absSize, 0, 1)
        
        flyFill.Size = UDim2.new(rel, 0, 1, 0)
        flySliderBtn.Position = UDim2.new(rel, -10, 0, -7)
        
        flySpeedVal = math.floor(rel * 190 + 10)
        flySpeedLabel.Text = "Fly Speed: " .. flySpeedVal .. " (Max: 200)"
    end
end)

local flyActive = false
addToggleButton(pageMovement, "ShiftLock Smooth Fly", function(state)
    flyActive = state
end)

RunService.RenderStepped:Connect(function()
    if flyActive and localPlayer.Character then
        pcall(function()
            local root = localPlayer.Character:FindFirstChild("HumanoidRootPart")
            local humanoid = localPlayer.Character:FindFirstChildOfClass("Humanoid")
            if root and humanoid then
                humanoid.PlatformStand = true
                local camCFrame = camera.CFrame
                local moveDir = humanoid.MoveDirection
                
                if moveDir.Magnitude > 0 then
                    root.Velocity = camCFrame.LookVector * (moveDir.Z * -flySpeedVal) + camCFrame.RightVector * (moveDir.X * flySpeedVal) + Vector3.new(0, 0, 0)
                else
                    root.Velocity = Vector3.new(0, 0, 0)
                end
                root.CFrame = CFrame.new(root.Position, root.Position + camCFrame.LookVector)
            end
        end)
    else
        pcall(function()
            if localPlayer.Character then
                local humanoid = localPlayer.Character:FindFirstChildOfClass("Humanoid")
                if humanoid then
                    humanoid.PlatformStand = false
                end
            end
        end)
    end
end)

local widgetBtnFly = Instance.new("TextButton")
widgetBtnFly.Size = UDim2.new(0, 410, 0, 35)
widgetBtnFly.BackgroundColor3 = Color3.fromRGB(0, 90, 60)
widgetBtnFly.BorderSizePixel = 0
widgetBtnFly.TextColor3 = Color3.fromRGB(255, 255, 255)
widgetBtnFly.TextSize = 13
widgetBtnFly.Font = Enum.Font.Code
widgetBtnFly.Text = "Spawn Fly Floating Widget"
widgetBtnFly.Parent = pageMovement
local wcf = Instance.new("UICorner") wcf.CornerRadius = UDim.new(0, 4) wcf.Parent = widgetBtnFly
widgetBtnFly.MouseButton1Click:Connect(function()
    playClickSound()
    createFeatureWidget("Fly Quick", function(state) flyActive = state end)
end)

local widgetBtn3 = Instance.new("TextButton")
widgetBtn3.Size = UDim2.new(0, 410, 0, 35)
widgetBtn3.BackgroundColor3 = Color3.fromRGB(80, 0, 80)
widgetBtn3.BorderSizePixel = 0
widgetBtn3.TextColor3 = Color3.fromRGB(255, 255, 255)
widgetBtn3.TextSize = 13
widgetBtn3.Font = Enum.Font.Code
widgetBtn3.Text = "Spawn Speed Floating Widget"
widgetBtn3.Parent = pageMovement
local wc3 = Instance.new("UICorner") wc3.CornerRadius = UDim.new(0, 4) wc3.Parent = widgetBtn3
widgetBtn3.MouseButton1Click:Connect(function()
    playClickSound()
    createFeatureWidget("Speed Quick", function(state) speedActive = state end)
end)

RunService.Heartbeat:Connect(function()
    pcall(function()
        if localPlayer.Character then
            local humanoid = localPlayer.Character:FindFirstChildOfClass("Humanoid")
            if humanoid and speedActive and not flyActive then humanoid.WalkSpeed = speedValue end
            if noclipActive then
                for _, part in pairs(localPlayer.Character:GetDescendants()) do
                    if part:IsA("BasePart") then part.CanCollide = false end
                end
            end
        end
    end)
end)

-- 4. TP & SPAWNER TAB
local function createButton(name, color, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 410, 0, 35)
    btn.BackgroundColor3 = color
    btn.BorderSizePixel = 0
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.TextSize = 13
    btn.Font = Enum.Font.Code
    btn.Text = name
    btn.Parent = pageWorld
    local corner = Instance.new("UICorner") corner.CornerRadius = UDim.new(0, 4) corner.Parent = btn
    btn.MouseButton1Click:Connect(function() playClickSound() callback() end)
end

createButton("Teleport to Safe Zone", Color3.fromRGB(0, 100, 200), function()
    local char = localPlayer.Character
    if char and char:FindFirstChild("HumanoidRootPart") then char.HumanoidRootPart.CFrame = CFrame.new(0, 150, 0) end
end)

createButton("TP to Random Chest", Color3.fromRGB(180, 120, 0), function()
    local chests = {}
    for _, obj in pairs(Workspace:GetDescendants()) do
        local name = obj.Name:lower()
        if (name:find("chest") or name:find("kasa") or name:find("box") or name:find("crate")) and obj:IsA("BasePart") then
            table.insert(chests, obj)
        elseif obj:IsA("ProximityPrompt") and obj.Parent and obj.Parent:IsA("BasePart") then
            table.insert(chests, obj.Parent)
        end
    end
    if #chests > 0 then
        local rc = chests[math.random(1, #chests)]
        local char = localPlayer.Character
        if char and char:FindFirstChild("HumanoidRootPart") then char.HumanoidRootPart.CFrame = rc.CFrame + Vector3.new(0, 3, 0) end
    end
end)

-- CHAT OBJECT SPAWNER
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
spawnerTitle.Text = "  [ CHAT OBJECT SPAWNER (AUTO LOOP & SPEED) ]"
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
spawnToggleBtn.Text = "Spawner [ OFF ]"
spawnToggleBtn.Parent = spawnerBoxFrame

local stCorner = Instance.new("UICorner") stCorner.CornerRadius = UDim.new(0, 4) stCorner.Parent = spawnToggleBtn

local delayLabel = Instance.new("TextLabel")
delayLabel.Size = UDim2.new(1, -20, 0, 20)
delayLabel.Position = UDim2.new(0, 10, 0, 72)
delayLabel.BackgroundTransparency = 1
delayLabel.Text = "Spawn Delay (Speed): 0.5s (500ms)"
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
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        delaySliding = false
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if delaySliding and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local pos = input.Position.X
        local absPos = delayTrack.AbsolutePosition.X
        local absSize = delayTrack.AbsoluteSize.X
        local rel = math.clamp((pos - absPos) / absSize, 0, 1)
        
        delayFill.Size = UDim2.new(rel, 0, 1, 0)
        delayButton.Position = UDim2.new(rel, -8, 0, -5)
        
        spawnDelayTime = math.round((rel * 1.95 + 0.05) * 100) / 100
        delayLabel.Text = string.format("Spawn Delay (Speed): %.2fs (%dms)", spawnDelayTime, math.floor(spawnDelayTime * 1000))
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
                clone:SetPrimaryPartCFrame(spawnPos)
                clone.Parent = Workspace
            elseif clone:IsA("BasePart") then
                clone.CFrame = spawnPos
                clone.Parent = Workspace
            end
        else
            local numId = tonumber(nameOrId)
            if numId then
                local model = InsertService:LoadAsset(numId)
                if model then
                    for _, child in pairs(model:GetChildren()) do
                        child.Parent = Workspace
                        if child:IsA("Model") then
                            child:SetPrimaryPartCFrame(spawnPos)
                        elseif child:IsA("BasePart") then
                            child.CFrame = spawnPos
                        end
                    end
                end
            else
                local part = Instance.new("Part")
                part.Size = Vector3.new(4, 4, 4)
                part.Position = spawnPos.Position + Vector3.new(0, 3, 0)
                part.BrickColor = BrickColor.random()
                part.Material = Enum.Material.Neon
                part.Name = nameOrId
                part.Parent = Workspace
                
                local mesh = Instance.new("SpecialMesh")
                mesh.MeshType = Enum.MeshType.Brick
                mesh.Parent = part
            end
        end
    end)
end

spawnToggleBtn.MouseButton1Click:Connect(function()
    playClickSound()
    spawnActiveLoop = not spawnActiveLoop
    if spawnActiveLoop then
        spawnToggleBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 60)
        spawnToggleBtn.Text = "Spawner [ ON ]"
        
        task.spawn(function()
            while spawnActiveLoop do
                local textVal = spawnerInput.Text
                if textVal ~= "" then
                    spawnTargetObject(textVal)
                end
                task.wait(spawnDelayTime)
            end
        end)
    else
        spawnToggleBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        spawnToggleBtn.Text = "Spawner [ OFF ]"
    end
end)

-- 5. SETTINGS TAB
local function createSettingsHeader(text)
    local lbl = Instance.new("TextLabel")
    lbl.Size = UDim2.new(0, 410, 0, 25)
    lbl.BackgroundTransparency = 1
    lbl.TextColor3 = Color3.fromRGB(0, 255, 128)
    lbl.TextSize = 14
    lbl.Font = Enum.Font.Code
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Text = "--- " .. text .. " ---"
    lbl.Parent = pageSettings
end

createSettingsHeader("APPEARANCE & THEME")

local colorPickerFrame = Instance.new("Frame")
colorPickerFrame.Size = UDim2.new(0, 410, 0, 120)
colorPickerFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
colorPickerFrame.BorderColor3 = Color3.fromRGB(0, 255, 128)
colorPickerFrame.BorderSizePixel = 1
colorPickerFrame.Visible = false
colorPickerFrame.Parent = pageSettings

local cpLayout = Instance.new("UIGridLayout")
cpLayout.CellSize = UDim2.new(0, 125, 0, 30)
cpLayout.CellPadding = UDim2.new(0, 5, 0, 5)
cpLayout.Parent = colorPickerFrame

local function addColorOption(name, col)
    local b = Instance.new("TextButton")
    b.BackgroundColor3 = col
    b.Text = name
    b.TextColor3 = Color3.fromRGB(255, 255, 255)
    b.TextSize = 11
    b.Font = Enum.Font.Code
    b.Parent = colorPickerFrame
    
    local c = Instance.new("UICorner") c.CornerRadius = UDim.new(0, 4) c.Parent = b
    
    b.MouseButton1Click:Connect(function()
        playClickSound()
        mainFrame.BackgroundColor3 = col
        toggleBtn.BackgroundColor3 = col
    end)
end

addColorOption("Classic Dark", Color3.fromRGB(20, 20, 20))
addColorOption("Deep Red", Color3.fromRGB(120, 0, 0))
addColorOption("Navy Blue", Color3.fromRGB(0, 50, 120))
addColorOption("Emerald Green", Color3.fromRGB(0, 100, 50))
addColorOption("Cyber Purple", Color3.fromRGB(75, 0, 130))
addColorOption("Sunset Orange", Color3.fromRGB(200, 80, 0))

local paletteBtn = Instance.new("TextButton")
paletteBtn.Size = UDim2.new(0, 410, 0, 35)
paletteBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
paletteBtn.BorderSizePixel = 0
paletteBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
paletteBtn.TextSize = 13
paletteBtn.Font = Enum.Font.Code
paletteBtn.Text = "🎨 Open/Close Color Palette"
paletteBtn.Parent = pageSettings

local pc = Instance.new("UICorner") pc.CornerRadius = UDim.new(0, 4) pc.Parent = paletteBtn
paletteBtn.MouseButton1Click:Connect(function()
    playClickSound()
    colorPickerFrame.Visible = not colorPickerFrame.Visible
end)

createSettingsHeader("CONTROLS & PERFORMANCE")
addToggleButton(pageSettings, "Lock Menu Position", function(state) isMenuLocked = state end)

local soundBtn = Instance.new("TextButton")
soundBtn.Size = UDim2.new(0, 410, 0, 35)
soundBtn.BackgroundColor3 = Color3.fromRGB(0, 80, 100)
soundBtn.BorderSizePixel = 0
soundBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
soundBtn.TextSize = 13
soundBtn.Font = Enum.Font.Code
soundBtn.Text = "🔊 Sound Effects: On"
soundBtn.Parent = pageSettings
local sbc = Instance.new("UICorner") sbc.CornerRadius = UDim.new(0, 4) sbc.Parent = soundBtn
soundBtn.MouseButton1Click:Connect(function()
    soundEffectsEnabled = not soundEffectsEnabled
    if soundEffectsEnabled then soundBtn.Text = "🔊 Sound Effects: On" playClickSound() else soundBtn.Text = "🔇 Sound Effects: Off" end
end)
-- FOV Slider
local fovSliderFrame = Instance.new("Frame")
fovSliderFrame.Size = UDim2.new(0, 410, 0, 50)
fovSliderFrame.BackgroundTransparency = 1
fovSliderFrame.Parent = pageSettings

local fovSliderLabel = Instance.new("TextLabel")
fovSliderLabel.Size = UDim2.new(1, 0, 0, 20)
fovSliderLabel.BackgroundTransparency = 1
fovSliderLabel.Text = "Camera FOV: 70"
fovSliderLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
fovSliderLabel.TextSize = 12
fovSliderLabel.Font = Enum.Font.Code
fovSliderLabel.TextXAlignment = Enum.TextXAlignment.Left
fovSliderLabel.Parent = fovSliderFrame

local fovSliderTrack = Instance.new("Frame")
fovSliderTrack.Size = UDim2.new(1, 0, 0, 6)
fovSliderTrack.Position = UDim2.new(0, 0, 0, 30)
fovSliderTrack.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
fovSliderTrack.BorderSizePixel = 0
fovSliderTrack.Parent = fovSliderFrame

local ftCorner = Instance.new("UICorner") ftCorner.CornerRadius = UDim.new(1, 0) ftCorner.Parent = fovSliderTrack

local fovSliderFill = Instance.new("Frame")
fovSliderFill.Size = UDim2.new((70-50)/70, 0, 1, 0)
fovSliderFill.BackgroundColor3 = Color3.fromRGB(0, 255, 128)
fovSliderFill.BorderSizePixel = 0
fovSliderFill.Parent = fovSliderTrack

local ffCorner = Instance.new("UICorner") ffCorner.CornerRadius = UDim.new(1, 0) ffCorner.Parent = fovSliderFill

local fovSliderBtn = Instance.new("TextButton")
fovSliderBtn.Size = UDim2.new(0, 20, 0, 20)
fovSliderBtn.Position = UDim2.new((70-50)/70, -10, 0, -7)
fovSliderBtn.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
fovSliderBtn.Text = ""
fovSliderBtn.Parent = fovSliderTrack

local fbCorner = Instance.new("UICorner") fbCorner.CornerRadius = UDim.new(1, 0) fbCorner.Parent = fovSliderBtn

local fovSliding = false
fovSliderBtn.MouseButton1Down:Connect(function() fovSliding = true end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        fovSliding = false
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if fovSliding and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local pos = input.Position.X
        local absPos = fovSliderTrack.AbsolutePosition.X
        local absSize = fovSliderTrack.AbsoluteSize.X
        local rel = math.clamp((pos - absPos) / absSize, 0, 1)
        
        fovSliderFill.Size = UDim2.new(rel, 0, 1, 0)
        fovSliderBtn.Position = UDim2.new(rel, -10, 0, -7)
        
        local targetFov = math.floor(rel * 70 + 50)
        camera.FieldOfView = targetFov
        fovSliderLabel.Text = "Camera FOV: " .. targetFov
    end
end)

addToggleButton(pageSettings, "Quick Auto Respawn", function(state)
    if state then
        pcall(function()
            localPlayer.CharacterAdded:Connect(function(char)
                local hum = char:WaitForChild("Humanoid")
                hum.Died:Connect(function()
                    task.wait(0.5)
                    pcall(function() localPlayer:LoadCharacter() end)
                end)
            end)
        end)
    end
end)

local perfBtn = Instance.new("TextButton")
perfBtn.Size = UDim2.new(0, 410, 0, 35)
perfBtn.BackgroundColor3 = Color3.fromRGB(100, 80, 0)
perfBtn.BorderSizePixel = 0
perfBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
perfBtn.TextSize = 13
perfBtn.Font = Enum.Font.Code
perfBtn.Text = "⚡ Performance Boost [ OFF ]"
perfBtn.Parent = pageSettings
local pfc = Instance.new("UICorner") pfc.CornerRadius = UDim.new(0, 4) pfc.Parent = perfBtn

local performanceActive = false
perfBtn.MouseButton1Click:Connect(function()
    playClickSound()
    performanceActive = not performanceActive
    pcall(function()
        if performanceActive then
            perfBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 60)
            perfBtn.Text = "⚡ Performance Boost [ ON ]"
            Lighting.GlobalShadows = false
            Lighting.Brightness = 2
            for _, v in pairs(Workspace:GetDescendants()) do
                if v:IsA("BasePart") then v.Material = Enum.Material.SmoothPlastic v.Reflectance = 0 end
            end
        else
            perfBtn.BackgroundColor3 = Color3.fromRGB(100, 80, 0)
            perfBtn.Text = "⚡ Performance Boost [ OFF ]"
            Lighting.GlobalShadows = true
            Lighting.Brightness = 1
            for _, v in pairs(Workspace:GetDescendants()) do
                if v:IsA("BasePart") then v.Material = Enum.Material.Plastic v.Reflectance = 0 end
            end
        end
    end)
end)

local panicBtn = Instance.new("TextButton")
panicBtn.Size = UDim2.new(0, 410, 0, 35)
panicBtn.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
panicBtn.BorderSizePixel = 0
panicBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
panicBtn.TextSize = 13
panicBtn.Font = Enum.Font.Code
panicBtn.Text = "🚨 Panic Button (Hide Menu)"
panicBtn.Parent = pageSettings
local pnc = Instance.new("UICorner") pnc.CornerRadius = UDim.new(0, 4) pnc.Parent = panicBtn
panicBtn.MouseButton1Click:Connect(function()
    playClickSound()
    mainFrame.Visible = false
    toggleBtn.Visible = not toggleBtn.Visible
end)

-- 6. INFO TAB
local infoLabel = Instance.new("TextLabel")
infoLabel.Size = UDim2.new(1, -20, 1, -20)
infoLabel.BackgroundTransparency = 1
infoLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
infoLabel.TextSize = 13
infoLabel.Font = Enum.Font.Code
infoLabel.TextXAlignment = Enum.TextXAlignment.Left
infoLabel.TextYAlignment = Enum.TextYAlignment.Top
infoLabel.TextWrapped = true
infoLabel.Text = [[=== MENU INFORMATION ===

Author / Creator: black
Version: v9.20 Ultimate (CoreGui Forced & Insert Key Hotkey)
Game: You vs Hulk

--- STATUS ---
• CoreGui injection prioritized so it bypasses UI hiding.
• Press 'Insert' key on your keyboard if the menu button is hidden.
]]
infoLabel.Parent = pageInfo
