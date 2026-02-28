-- FX HUB - РАБОЧАЯ ВЕРСИЯ
local player = game.Players.LocalPlayer
local mouse = player:GetMouse()
local camera = workspace.CurrentCamera

-- ========== ГЛАВНОЕ ОКНО ==========
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = player.PlayerGui
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 420, 0, 520)
mainFrame.Position = UDim2.new(0.5, -210, 0.5, -260)
mainFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui

-- ЗАГОЛОВОК
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 40)
titleBar.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
titleBar.Parent = mainFrame

local title = Instance.new("TextLabel")
title.Size = UDim2.new(0.7, 0, 1, 0)
title.Position = UDim2.new(0, 15, 0, 0)
title.BackgroundTransparency = 1
title.Text = "🎯 FX HUB"
title.TextColor3 = Color3.fromRGB(255, 255, 100)
title.TextSize = 22
title.TextXAlignment = Enum.TextXAlignment.Left
title.Font = Enum.Font.GothamBold
title.Parent = titleBar

-- КНОПКИ
local minimizeBtn = Instance.new("TextButton")
minimizeBtn.Size = UDim2.new(0, 35, 0, 30)
minimizeBtn.Position = UDim2.new(1, -40, 0.5, -15)
minimizeBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 100)
minimizeBtn.Text = "🗕"
minimizeBtn.TextColor3 = Color3.new(1, 1, 1)
minimizeBtn.TextSize = 18
minimizeBtn.Font = Enum.Font.GothamBold
minimizeBtn.Parent = titleBar

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 35, 0, 30)
closeBtn.Position = UDim2.new(1, -80, 0.5, -15)
closeBtn.BackgroundColor3 = Color3.fromRGB(220, 60, 60)
closeBtn.Text = "✖"
closeBtn.TextColor3 = Color3.new(1, 1, 1)
closeBtn.TextSize = 18
closeBtn.Font = Enum.Font.GothamBold
closeBtn.Parent = titleBar

-- ========== ПЕРЕМЕННЫЕ ==========
_G.AimbotEnabled = false
_G.AimbotRadius = 200
_G.AimbotSmoothness = 30
_G.WeakAimEnabled = false
_G.WeakAimPower = 0.20
_G.CrosshairSize = 70
_G.ESPEnabled = false
_G.FPSLevel = 0
_G.ShowCrosshair = true
_G.WallCheck = true
_G.SelectedTarget = nil
_G.TargetPart = "Head"
_G.WindowMinimized = false

-- ========== ПРОЗРАЧНЫЙ КВАДРАТ ==========
local crosshairGui = Instance.new("ScreenGui")
crosshairGui.Name = "CrosshairGui"
crosshairGui.Parent = player.PlayerGui
crosshairGui.ResetOnSpawn = false
crosshairGui.DisplayOrder = 1000
crosshairGui.Enabled = true
crosshairGui.IgnoreGuiInset = true

local function updateSquare()
    for _, v in pairs(crosshairGui:GetChildren()) do v:Destroy() end
    if not _G.ShowCrosshair then return end
    
    local s = _G.CrosshairSize
    local offsetY = 0
    local square = Instance.new("Frame")
    square.Name = "SquareCrosshair"
    square.Size = UDim2.new(0, s, 0, s)
    square.Position = UDim2.new(0.5, -s/2, 0.5, offsetY - s/2)
    square.BackgroundColor3 = Color3.new(0, 0, 0)
    square.BackgroundTransparency = 0.4
    square.BorderSizePixel = 0
    square.Parent = crosshairGui
end

spawn(function() while wait(0.1) do updateSquare() end end)

-- ========== СВОРАЧИВАНИЕ ==========
local tabsFrame = Instance.new("Frame")
tabsFrame.Size = UDim2.new(1, 0, 0, 40)
tabsFrame.Position = UDim2.new(0, 0, 0, 40)
tabsFrame.BackgroundColor3 = Color3.fromRGB(45, 45, 60)
tabsFrame.Parent = mainFrame

local contentFrame = Instance.new("ScrollingFrame")
contentFrame.Size = UDim2.new(1, 0, 1, -80)
contentFrame.Position = UDim2.new(0, 0, 0, 80)
contentFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
contentFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
contentFrame.ScrollBarThickness = 8
contentFrame.ScrollBarImageColor3 = Color3.fromRGB(150, 150, 200)
contentFrame.Parent = mainFrame

minimizeBtn.MouseButton1Click:Connect(function()
    _G.WindowMinimized = not _G.WindowMinimized
    if _G.WindowMinimized then
        mainFrame.Size = UDim2.new(0, 420, 0, 40)
        minimizeBtn.Text = "🗖"
        tabsFrame.Visible = false
        contentFrame.Visible = false
    else
        mainFrame.Size = UDim2.new(0, 420, 0, 520)
        minimizeBtn.Text = "🗕"
        tabsFrame.Visible = true
        contentFrame.Visible = true
    end
end)

closeBtn.MouseButton1Click:Connect(function() mainFrame.Visible = false end)

-- ========== ФУНКЦИИ FPS ==========
local function applyFPSBoost(level)
    if level == 0 then
        settings().Rendering.QualityLevel = 10
        game.Lighting.GlobalShadows = true
    elseif level == 1 then
        settings().Rendering.QualityLevel = 8
        game.Lighting.GlobalShadows = false
    elseif level == 2 then
        settings().Rendering.QualityLevel = 5
        game.Lighting.GlobalShadows = false
        game.Lighting.FogEnd = 9e9
    elseif level == 3 then
        settings().Rendering.QualityLevel = 3
        game.Lighting.GlobalShadows = false
        game.Lighting.FogEnd = 9e9
    elseif level == 4 then
        settings().Rendering.QualityLevel = 1
        game.Lighting.GlobalShadows = false
        game.Lighting.FogEnd = 9e9
    end
end

-- ========== КНОПКИ ==========
local function createButton(name, y, callback, selected)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.9, 0, 0, 38)
    btn.Position = UDim2.new(0.05, 0, 0, y)
    btn.BackgroundColor3 = selected and Color3.fromRGB(0, 180, 0) or Color3.fromRGB(55, 55, 70)
    btn.Text = name
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.TextSize = 16
    btn.Font = Enum.Font.Gotham
    btn.Parent = contentFrame
    btn.MouseButton1Click:Connect(callback)
    return y + 43
end

-- ========== ВКЛАДКИ ==========
local function showTab(tab)
    for _, v in pairs(contentFrame:GetChildren()) do v:Destroy() end
    local y = 10
    
    if tab == "aim" then
        local aimBtn = Instance.new("TextButton")
        aimBtn.Size = UDim2.new(0.9, 0, 0, 38)
        aimBtn.Position = UDim2.new(0.05, 0, 0, y)
        aimBtn.BackgroundColor3 = _G.AimbotEnabled and Color3.fromRGB(0, 180, 0) or Color3.fromRGB(55, 55, 70)
        aimBtn.Text = "🎯 АИМБОТ: " .. (_G.AimbotEnabled and "ВКЛ" or "ВЫКЛ")
        aimBtn.Parent = contentFrame
        aimBtn.MouseButton1Click:Connect(function() _G.AimbotEnabled = not _G.AimbotEnabled showTab("aim") end)
        y = y + 43
        
        local weakBtn = Instance.new("TextButton")
        weakBtn.Size = UDim2.new(0.9, 0, 0, 38)
        weakBtn.Position = UDim2.new(0.05, 0, 0, y)
        weakBtn.BackgroundColor3 = _G.WeakAimEnabled and Color3.fromRGB(0, 180, 0) or Color3.fromRGB(55, 55, 70)
        weakBtn.Text = "🔄 СЛАБЫЙ АИМ: " .. (_G.WeakAimEnabled and "ВКЛ" or "ВЫКЛ")
        weakBtn.Parent = contentFrame
        weakBtn.MouseButton1Click:Connect(function() _G.WeakAimEnabled = not _G.WeakAimEnabled showTab("aim") end)
        y = y + 43
        
        local powerLabel = Instance.new("TextLabel")
        powerLabel.Size = UDim2.new(0.9, 0, 0, 25)
        powerLabel.Position = UDim2.new(0.05, 0, 0, y)
        powerLabel.BackgroundTransparency = 1
        powerLabel.Text = "⚡ СИЛА: " .. string.format("%.2f", _G.WeakAimPower)
        powerLabel.TextColor3 = Color3.fromRGB(255, 255, 100)
        powerLabel.Parent = contentFrame
        y = y + 30
        
        local powers = {"0.10", "0.15", "0.20", "0.25", "0.30"}
        local powerValues = {0.10, 0.15, 0.20, 0.25, 0.30}
        for i = 1, 5 do
            y = createButton(powers[i], y, function() _G.WeakAimPower = powerValues[i] showTab("aim") end, math.abs(_G.WeakAimPower - powerValues[i]) < 0.01)
        end
        
        local partLabel = Instance.new("TextLabel")
        partLabel.Size = UDim2.new(0.9, 0, 0, 25)
        partLabel.Position = UDim2.new(0.05, 0, 0, y)
        partLabel.BackgroundTransparency = 1
        partLabel.Text = "🎯 ЧАСТЬ ТЕЛА:"
        partLabel.TextColor3 = Color3.fromRGB(255, 255, 100)
        partLabel.Parent = contentFrame
        y = y + 30
        
        local parts = {"Голова", "Тело", "Рука", "Нога"}
        local partValues = {"Head", "Torso", "Right Hand", "Right Leg"}
        for i = 1, 4 do
            y = createButton(parts[i], y, function() _G.TargetPart = partValues[i] showTab("aim") end, _G.TargetPart == partValues[i])
        end
        
        local radiusLabel = Instance.new("TextLabel")
        radiusLabel.Size = UDim2.new(0.9, 0, 0, 25)
        radiusLabel.Position = UDim2.new(0.05, 0, 0, y)
        radiusLabel.BackgroundTransparency = 1
        radiusLabel.Text = "📏 РАДИУС: " .. _G.AimbotRadius
        radiusLabel.TextColor3 = Color3.fromRGB(255, 255, 100)
        radiusLabel.Parent = contentFrame
        y = y + 30
        
        y = createButton("- 50", y, function() _G.AimbotRadius = math.max(50, _G.AimbotRadius - 50) showTab("aim") end)
        y = createButton("- 20", y, function() _G.AimbotRadius = math.max(50, _G.AimbotRadius - 20) showTab("aim") end)
        y = createButton("- 10", y, function() _G.AimbotRadius = math.max(50, _G.AimbotRadius - 10) showTab("aim") end)
        y = createButton("- 5", y, function() _G.AimbotRadius = math.max(50, _G.AimbotRadius - 5) showTab("aim") end)
        y = createButton("+ 5", y, function() _G.AimbotRadius = math.min(300, _G.AimbotRadius + 5) showTab("aim") end)
        y = createButton("+ 10", y, function() _G.AimbotRadius = math.min(300, _G.AimbotRadius + 10) showTab("aim") end)
        y = createButton("+ 20", y, function() _G.AimbotRadius = math.min(300, _G.AimbotRadius + 20) showTab("aim") end)
        y = createButton("+ 50", y, function() _G.AimbotRadius = math.min(300, _G.AimbotRadius + 50) showTab("aim") end)
        
        local smoothLabel = Instance.new("TextLabel")
        smoothLabel.Size = UDim2.new(0.9, 0, 0, 25)
        smoothLabel.Position = UDim2.new(0.05, 0, 0, y)
        smoothLabel.BackgroundTransparency = 1
        smoothLabel.Text = "🌀 ПЛАВНОСТЬ: " .. _G.AimbotSmoothness .. "%"
        smoothLabel.TextColor3 = Color3.fromRGB(255, 255, 100)
        smoothLabel.Parent = contentFrame
        y = y + 30
        
        y = createButton("- 5%", y, function() _G.AimbotSmoothness = math.max(5, _G.AimbotSmoothness - 5) showTab("aim") end)
        y = createButton("+ 5%", y, function() _G.AimbotSmoothness = math.min(100, _G.AimbotSmoothness + 5) showTab("aim") end)
        
        local crossLabel = Instance.new("TextLabel")
        crossLabel.Size = UDim2.new(0.9, 0, 0, 25)
        crossLabel.Position = UDim2.new(0.05, 0, 0, y)
        crossLabel.BackgroundTransparency = 1
        crossLabel.Text = "⬛ РАЗМЕР: " .. _G.CrosshairSize
        crossLabel.TextColor3 = Color3.fromRGB(255, 255, 100)
        crossLabel.Parent = contentFrame
        y = y + 30
        
        y = createButton("- 5", y, function() _G.CrosshairSize = math.max(30, _G.CrosshairSize - 5) showTab("aim") end)
        y = createButton("+ 5", y, function() _G.CrosshairSize = math.min(200, _G.CrosshairSize + 5) showTab("aim") end)
        
        local wallBtn = Instance.new("TextButton")
        wallBtn.Size = UDim2.new(0.9, 0, 0, 38)
        wallBtn.Position = UDim2.new(0.05, 0, 0, y)
        wallBtn.BackgroundColor3 = _G.WallCheck and Color3.fromRGB(0, 180, 0) or Color3.fromRGB(55, 55, 70)
        wallBtn.Text = "🧱 СТЕНЫ: " .. (_G.WallCheck and "ВКЛ" or "ВЫКЛ")
        wallBtn.Parent = contentFrame
        wallBtn.MouseButton1Click:Connect(function() _G.WallCheck = not _G.WallCheck showTab("aim") end)
        y = y + 43
        
        contentFrame.CanvasSize = UDim2.new(0, 0, 0, y + 20)
    end
    
    if tab == "target" then
        if _G.SelectedTarget then
            local cur = Instance.new("TextLabel")
            cur.Size = UDim2.new(0.9, 0, 0, 35)
            cur.Position = UDim2.new(0.05, 0, 0, y)
            cur.BackgroundColor3 = Color3.fromRGB(0, 100, 0)
            cur.Text = "👤 ТЕКУЩАЯ: " .. _G.SelectedTarget.Name
            cur.TextColor3 = Color3.fromRGB(255, 255, 0)
            cur.Parent = contentFrame
            y = y + 40
        end
        
        y = createButton("🔄 СБРОСИТЬ ЦЕЛЬ", y, function() _G.SelectedTarget = nil showTab("target") end)
        
        for _, v in pairs(game.Players:GetPlayers()) do
            if v ~= player then
                y = createButton("👤 " .. v.Name, y, function() _G.SelectedTarget = v showTab("target") end, _G.SelectedTarget == v)
            end
        end
        contentFrame.CanvasSize = UDim2.new(0, 0, 0, y + 20)
    end
    
    if tab == "esp" then
        local espBtn = Instance.new("TextButton")
        espBtn.Size = UDim2.new(0.9, 0, 0, 38)
        espBtn.Position = UDim2.new(0.05, 0, 0, y)
        espBtn.BackgroundColor3 = _G.ESPEnabled and Color3.fromRGB(0, 180, 0) or Color3.fromRGB(55, 55, 70)
        espBtn.Text = "👁️ ESP: " .. (_G.ESPEnabled and "ВКЛ" or "ВЫКЛ")
        espBtn.Parent = contentFrame
        espBtn.MouseButton1Click:Connect(function() _G.ESPEnabled = not _G.ESPEnabled showTab("esp") end)
        y = y + 43
        contentFrame.CanvasSize = UDim2.new(0, 0, 0, y + 20)
    end
    
    if tab == "fps" then
        local levels = {"⚪ ВЫКЛ", "🟢 СЛАБО", "🟡 СРЕДНЕ", "🟠 СИЛЬНО", "🔴 МАКС"}
        for i = 0, 4 do
            y = createButton(levels[i+1], y, function() _G.FPSLevel = i applyFPSBoost(i) showTab("fps") end, _G.FPSLevel == i)
        end
        
        local crossBtn = Instance.new("TextButton")
        crossBtn.Size = UDim2.new(0.9, 0, 0, 38)
        crossBtn.Position = UDim2.new(0.05, 0, 0, y)
        crossBtn.BackgroundColor3 = _G.ShowCrosshair and Color3.fromRGB(0, 180, 0) or Color3.fromRGB(55, 55, 70)
        crossBtn.Text = "⬛ КВАДРАТ: " .. (_G.ShowCrosshair and "ВКЛ" or "ВЫКЛ")
        crossBtn.Parent = contentFrame
        crossBtn.MouseButton1Click:Connect(function() _G.ShowCrosshair = not _G.ShowCrosshair showTab("fps") end)
        y = y + 43
        contentFrame.CanvasSize = UDim2.new(0, 0, 0, y + 20)
    end
end

-- ========== ВКЛАДКИ МЕНЮ ==========
local tabs = {
    aim = Instance.new("TextButton"), 
    target = Instance.new("TextButton"),
    esp = Instance.new("TextButton"), 
    fps = Instance.new("TextButton")
}

local function selectTab(name)
    for n, tab in pairs(tabs) do
        tab.BackgroundColor3 = n == name and Color3.fromRGB(70, 70, 100) or Color3.fromRGB(45, 45, 60)
    end
    showTab(name)
end

tabs.aim.Size = UDim2.new(0.25, 0, 1, 0)
tabs.aim.Position = UDim2.new(0, 0, 0, 0)
tabs.aim.Text = "🎯 АИМ"
tabs.aim.Parent = tabsFrame
tabs.aim.MouseButton1Click:Connect(function() selectTab("aim") end)

tabs.target.Size = UDim2.new(0.25, 0, 1, 0)
tabs.target.Position = UDim2.new(0.25, 0, 0, 0)
tabs.target.Text = "👤 ЦЕЛЬ"
tabs.target.Parent = tabsFrame
tabs.target.MouseButton1Click:Connect(function() selectTab("target") end)

tabs.esp.Size = UDim2.new(0.25, 0, 1, 0)
tabs.esp.Position = UDim2.new(0.5, 0, 0, 0)
tabs.esp.Text = "👁️ ЕСП"
tabs.esp.Parent = tabsFrame
tabs.esp.MouseButton1Click:Connect(function() selectTab("esp") end)

tabs.fps.Size = UDim2.new(0.25, 0, 1, 0)
tabs.fps.Position = UDim2.new(0.75, 0, 0, 0)
tabs.fps.Text = "⚡ ФПС"
tabs.fps.Parent = tabsFrame
tabs.fps.MouseButton1Click:Connect(function() selectTab("fps") end)

selectTab("aim")

-- ========== ESP ==========
spawn(function()
    while wait(0.5) do
        if _G.ESPEnabled then
            for _, v in pairs(game.Players:GetPlayers()) do
                if v ~= player and v.Character and not v.Character:FindFirstChild("ESP_Highlight") then
                    local h = Instance.new("Highlight")
                    h.Name = "ESP_Highlight"
                    h.FillColor = Color3.new(1, 0, 0)
                    h.FillTransparency = 0.3
                    h.Parent = v.Character
                end
            end
        else
            for _, v in pairs(game.Players:GetPlayers()) do
                if v.Character and v.Character:FindFirstChild("ESP_Highlight") then
                    v.Character.ESP_Highlight:Destroy()
                end
            end
        end
    end
end)

-- ========== ФУНКЦИИ ==========
local function getTargetPart(character)
    if not character then return nil end
    if _G.TargetPart == "Head" then return character:FindFirstChild("Head")
    elseif _G.TargetPart == "Torso" then return character:FindFirstChild("Torso") or character:FindFirstChild("UpperTorso")
    elseif _G.TargetPart == "Right Hand" then return character:FindFirstChild("Right Hand") or character:FindFirstChild("RightHand")
    elseif _G.TargetPart == "Right Leg" then return character:FindFirstChild("Right Leg") or character:FindFirstChild("RightLeg")
    end
    return character:FindFirstChild("Head") or character:FindFirstChild("HumanoidRootPart")
end

local function getPlayerInSquare()
    local screenCenter = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)
    local squareHalf = _G.CrosshairSize / 2
    local squareMin = Vector2.new(screenCenter.X - squareHalf, screenCenter.Y - squareHalf)
    local squareMax = Vector2.new(screenCenter.X + squareHalf, screenCenter.Y + squareHalf)
    
    for _, v in pairs(game.Players:GetPlayers()) do
        if v ~= player and v.Character and v.Character:FindFirstChild("Head") then
            local head = v.Character.Head
            local headPos, onScreen = camera:WorldToViewportPoint(head.Position)
            if onScreen then
                local headScreenPos = Vector2.new(headPos.X, headPos.Y)
                if headScreenPos.X >= squareMin.X and headScreenPos.X <= squareMax.X and
                   headScreenPos.Y >= squareMin.Y and headScreenPos.Y <= squareMax.Y then
                    if _G.WallCheck then
                        local ray = Ray.new(camera.CFrame.Position, (head.Position - camera.CFrame.Position).Unit * 1000)
                        local hit = workspace:FindPartOnRay(ray, player.Character)
                        if hit and (hit:IsDescendantOf(v.Character) or hit == head) then return v end
                    else return v end
                end
            end
        end
    end
    return nil
end

local function isInRadius(character, target)
    if not character or not target or not target.Character then return false end
    local dist = (character.HumanoidRootPart.Position - target.Character.HumanoidRootPart.Position).Magnitude
    return dist <= _G.AimbotRadius
end

-- ========== АИМБОТ ==========
spawn(function()
    while wait(0.01) do
        if _G.AimbotEnabled then
            local char = player.Character
            if char and char:FindFirstChild("HumanoidRootPart") then
                local target = nil
                local targetInSquare = getPlayerInSquare()
                
                if targetInSquare and isInRadius(char, targetInSquare) then
                    target = targetInSquare
                end
                
                if not target and _G.SelectedTarget and _G.SelectedTarget.Character then
                    if isInRadius(char, _G.SelectedTarget) then
                        target = _G.SelectedTarget
                    end
                end
                
                if target and target.Character then
                    local targetPart = getTargetPart(target.Character)
                    if targetPart then
                        local power = _G.WeakAimEnabled and _G.WeakAimPower or (_G.AimbotSmoothness / 100)
                        local lookAt = CFrame.new(camera.CFrame.Position, targetPart.Position)
                        camera.CFrame = camera.CFrame:Lerp(lookAt, power)
                    end
                end
            end
        end
    end
end)

print("✅ FX HUB ЗАГРУЖЕН!")
print("🎯 Прицел по центру")
