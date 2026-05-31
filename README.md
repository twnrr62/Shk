-- LocalScript dentro de StarterGui
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- ========== CONFIGURAÇÕES ==========
local settings = {
    aimbotEnabled = true,
    espEnabled = true,
    fovRadius = 150,
}

-- ========== CRIAR INTERFACE ==========
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AimbotSystem"
screenGui.IgnoreGuiInset = true
screenGui.Parent = player:WaitForChild("PlayerGui")

-- ========== CÍRCULO FOV PERFEITO (AMARELO) ==========
local fovCircle = Instance.new("Frame")
fovCircle.Size = UDim2.new(0, settings.fovRadius * 2, 0, settings.fovRadius * 2)
fovCircle.Position = UDim2.new(0.5, -settings.fovRadius, 0.5, -settings.fovRadius)
fovCircle.BackgroundColor3 = Color3.new(1, 0.85, 0)  -- Amarelo vibrante
fovCircle.BackgroundTransparency = 0.92
fovCircle.BorderSizePixel = 3
fovCircle.BorderColor3 = Color3.new(1, 0.8, 0)
fovCircle.ZIndex = 10
fovCircle.Parent = screenGui

-- Deixar 100% redondo
local circleCorner = Instance.new("UICorner")
circleCorner.CornerRadius = UDim.new(1, 0)  -- Isso faz o círculo perfeito!
circleCorner.Parent = fovCircle

-- Brilho externo (efeito de glow)
local glow = Instance.new("Frame")
glow.Size = UDim2.new(1, 12, 1, 12)
glow.Position = UDim2.new(0.5, -settings.fovRadius - 6, 0.5, -settings.fovRadius - 6)
glow.BackgroundColor3 = Color3.new(1, 0.85, 0)
glow.BackgroundTransparency = 0.95
glow.BorderSizePixel = 0
glow.ZIndex = 9
glow.Parent = screenGui

local glowCorner = Instance.new("UICorner")
glowCorner.CornerRadius = UDim.new(1, 0)
glowCorner.Parent = glow

-- Ponto central (estilo "Camera Lock" igual ao print)
local centerDot = Instance.new("Frame")
centerDot.Size = UDim2.new(0, 5, 0, 5)
centerDot.Position = UDim2.new(0.5, -2.5, 0.5, -2.5)
centerDot.BackgroundColor3 = Color3.new(1, 0.85, 0)
centerDot.BackgroundTransparency = 0
centerDot.BorderSizePixel = 1
centerDot.BorderColor3 = Color3.new(1, 1, 1)
centerDot.ZIndex = 11
centerDot.Parent = screenGui

local dotCorner = Instance.new("UICorner")
dotCorner.CornerRadius = UDim.new(1, 0)
dotCorner.Parent = centerDot

-- Anel interno (opcional, mais estilizado)
local innerRing = Instance.new("Frame")
innerRing.Size = UDim2.new(0.8, 0, 0.8, 0)
innerRing.Position = UDim2.new(0.1, 0, 0.1, 0)
innerRing.BackgroundTransparency = 1
innerRing.BorderSizePixel = 1
innerRing.BorderColor3 = Color3.new(1, 0.85, 0)
innerRing.ZIndex = 10
innerRing.Parent = fovCircle

local innerCorner = Instance.new("UICorner")
innerCorner.CornerRadius = UDim.new(1, 0)
innerCorner.Parent = innerRing

-- ========== PAINEL DE MENU (ESTILO HERMANOS'DEV) ==========
local menuFrame = Instance.new("Frame")
menuFrame.Size = UDim2.new(0, 260, 0, 320)
menuFrame.Position = UDim2.new(0, 10, 0.5, -160)
menuFrame.BackgroundColor3 = Color3.new(0.08, 0.08, 0.12)
menuFrame.BackgroundTransparency = 0.05
menuFrame.BorderSizePixel = 1
menuFrame.BorderColor3 = Color3.new(1, 0.85, 0)
menuFrame.Active = true
menuFrame.Draggable = true
menuFrame.Parent = screenGui

-- Arredondar menu
local menuCorner = Instance.new("UICorner")
menuCorner.CornerRadius = UDim.new(0, 8)
menuCorner.Parent = menuFrame

-- Título
local menuTitle = Instance.new("TextLabel")
menuTitle.Size = UDim2.new(1, 0, 0, 35)
menuTitle.Position = UDim2.new(0, 0, 0, 0)
menuTitle.BackgroundColor3 = Color3.new(0.15, 0.15, 0.2)
menuTitle.Text = "⚡ CAMERA LOCK SYSTEM ⚡"
menuTitle.TextColor3 = Color3.new(1, 0.85, 0)
menuTitle.Font = Enum.Font.GothamBold
menuTitle.TextSize = 13
menuTitle.Parent = menuFrame

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 8)
titleCorner.Parent = menuTitle

-- Botão Aimbot (Camera Lock)
local aimbotBtn = Instance.new("TextButton")
aimbotBtn.Size = UDim2.new(0.9, 0, 0, 40)
aimbotBtn.Position = UDim2.new(0.05, 0, 0, 50)
aimbotBtn.BackgroundColor3 = Color3.new(0.2, 0.6, 0.2)
aimbotBtn.Text = "🔒 CAMERA LOCK: ON"
aimbotBtn.TextColor3 = Color3.new(1, 1, 1)
aimbotBtn.Font = Enum.Font.GothamSemibold
aimbotBtn.TextSize = 12
aimbotBtn.Parent = menuFrame

local btnCorner = Instance.new("UICorner")
btnCorner.CornerRadius = UDim.new(0, 5)
btnCorner.Parent = aimbotBtn

-- Botão ESP
local espBtn = Instance.new("TextButton")
espBtn.Size = UDim2.new(0.9, 0, 0, 40)
espBtn.Position = UDim2.new(0.05, 0, 0, 100)
espBtn.BackgroundColor3 = Color3.new(0.2, 0.6, 0.2)
espBtn.Text = "👁️ ESP PLAYERS: ON"
espBtn.TextColor3 = Color3.new(1, 1, 1)
espBtn.Font = Enum.Font.GothamSemibold
espBtn.TextSize = 12
espBtn.Parent = menuFrame

local espCorner = Instance.new("UICorner")
espCorner.CornerRadius = UDim.new(0, 5)
espCorner.Parent = espBtn

-- Slider FOV
local fovLabel = Instance.new("TextLabel")
fovLabel.Size = UDim2.new(0.9, 0, 0, 25)
fovLabel.Position = UDim2.new(0.05, 0, 0, 155)
fovLabel.BackgroundTransparency = 1
fovLabel.Text = "🎯 FOV RADIUS: " .. settings.fovRadius
fovLabel.TextColor3 = Color3.new(1, 1, 0.7)
fovLabel.TextSize = 11
fovLabel.Font = Enum.Font.Gotham
fovLabel.TextXAlignment = Enum.TextXAlignment.Left
fovLabel.Parent = menuFrame

local fovInput = Instance.new("TextBox")
fovInput.Size = UDim2.new(0.4, 0, 0, 30)
fovInput.Position = UDim2.new(0.55, 0, 0, 152)
fovInput.BackgroundColor3 = Color3.new(0.15, 0.15, 0.2)
fovInput.Text = tostring(settings.fovRadius)
fovInput.TextColor3 = Color3.new(1, 0.85, 0)
fovInput.Font = Enum.Font.Gotham
fovInput.TextSize = 12
fovInput.Parent = menuFrame

local inputCorner = Instance.new("UICorner")
inputCorner.CornerRadius = UDim.new(0, 4)
inputCorner.Parent = fovInput

-- Status
local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(0.9, 0, 0, 30)
statusLabel.Position = UDim2.new(0.05, 0, 0, 200)
statusLabel.BackgroundColor3 = Color3.new(0.1, 0.1, 0.15)
statusLabel.Text = "● TEST MODE ACTIVE"
statusLabel.TextColor3 = Color3.new(0, 1, 0)
statusLabel.TextSize = 10
statusLabel.Font = Enum.Font.Gotham
statusLabel.Parent = menuFrame

local statusCorner = Instance.new("UICorner")
statusCorner.CornerRadius = UDim.new(0, 4)
statusCorner.Parent = statusLabel

-- Rodapé
local footer = Instance.new("TextLabel")
footer.Size = UDim2.new(1, 0, 0, 25)
footer.Position = UDim2.new(0, 0, 1, -25)
footer.BackgroundColor3 = Color3.new(0.05, 0.05, 0.08)
footer.Text = "DRAG MENU | HOLD TO MOVE"
footer.TextColor3 = Color3.new(0.5, 0.5, 0.5)
footer.TextSize = 9
footer.Font = Enum.Font.Gotham
footer.Parent = menuFrame

-- ========== AIMBOT INSTANTÂNEO ==========
local function getClosestTargetInFOV()
    local center = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)
    local closest = nil
    local closestDistance = settings.fovRadius
    
    for _, otherPlayer in ipairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character then
            local head = otherPlayer.Character:FindFirstChild("Head")
            if head and head.Parent and head.Parent:FindFirstChild("Humanoid") and head.Parent.Humanoid.Health > 0 then
                local headPos, onScreen = camera:WorldToViewportPoint(head.Position)
                if onScreen then
                    local screenPos = Vector2.new(headPos.X, headPos.Y)
                    local distToCenter = (screenPos - center).Magnitude
                    if distToCenter < closestDistance then
                        closestDistance = distToCenter
                        closest = otherPlayer
                    end
                end
            end
        end
    end
    return closest
end

-- Aimbot instantâneo (100% funcional)
RunService.RenderStepped:Connect(function()
    if not settings.aimbotEnabled then return end
    
    local target = getClosestTargetInFOV()
    if target and target.Character then
        local head = target.Character:FindFirstChild("Head")
        if head then
            -- Mira instantânea na cabeça
            camera.CFrame = CFrame.new(camera.CFrame.Position, head.Position)
        end
    end
end)

-- ========== ESP COM LINHA ==========
local espObjects = {}

local function updateESP()
    if not settings.espEnabled then
        for _, obj in pairs(espObjects) do pcall(function() obj:Destroy() end) end
        espObjects = {}
        return
    end
    
    for _, obj in pairs(espObjects) do pcall(function() obj:Destroy() end) end
    espObjects = {}
    
    local myChar = player.Character
    local myRoot = myChar and myChar:FindFirstChild("HumanoidRootPart")
    if not myRoot then return end
    
    for _, otherPlayer in ipairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character then
            local head = otherPlayer.Character:FindFirstChild("Head")
            local root = otherPlayer.Character:FindFirstChild("HumanoidRootPart")
            local hum = otherPlayer.Character:FindFirstChild("Humanoid")
            
            if head and root and hum and hum.Health > 0 then
                local headScreen, onScreen = camera:WorldToViewportPoint(head.Position)
                local myScreen = camera:WorldToViewportPoint(myRoot.Position)
                local distance = (myRoot.Position - root.Position).Magnitude
                
                if onScreen then
                    -- Nome + Distância
                    local label = Instance.new("TextLabel")
                    label.Size = UDim2.new(0, 140, 0, 22)
                    label.Position = UDim2.new(0, headScreen.X - 70, 0, headScreen.Y - 35)
                    label.BackgroundTransparency = 0.4
                    label.BackgroundColor3 = Color3.new(0, 0, 0)
                    label.Text = string.format("%s  |  %.0fm", otherPlayer.Name, distance)
                    label.TextColor3 = Color3.new(1, 0.85, 0)
                    label.TextSize = 11
                    label.Font = Enum.Font.GothamBold
                    label.BorderSizePixel = 1
                    label.BorderColor3 = Color3.new(1, 0.85, 0)
                    label.ZIndex = 10
                    label.Parent = screenGui
                    table.insert(espObjects, label)
                    
                    -- Linha de conexão
                    if myScreen then
                        local start = Vector2.new(myScreen.X, myScreen.Y)
                        local finish = Vector2.new(headScreen.X, headScreen.Y)
                        local diff = finish - start
                        local length = diff.Magnitude
                        local angle = math.atan2(diff.Y, diff.X)
                        
                        local line = Instance.new("Frame")
                        line.Size = UDim2.new(0, length, 0, 2)
                        line.Position = UDim2.new(0, start.X, 0, start.Y)
                        line.Rotation = math.deg(angle)
                        line.BackgroundColor3 = Color3.new(1, 0.85, 0)
                        line.BackgroundTransparency = 0.4
                        line.BorderSizePixel = 0
                        line.ZIndex = 5
                        line.Parent = screenGui
                        table.insert(espObjects, line)
                    end
                end
            end
        end
    end
end

-- ========== EVENTOS DOS BOTÕES ==========
aimbotBtn.MouseButton1Click:Connect(function()
    settings.aimbotEnabled = not settings.aimbotEnabled
    aimbotBtn.Text = settings.aimbotEnabled and "🔒 CAMERA LOCK: ON" or "🔒 CAMERA LOCK: OFF"
    aimbotBtn.BackgroundColor3 = settings.aimbotEnabled and Color3.new(0.2, 0.6, 0.2) or Color3.new(0.6, 0.2, 0.2)
    
    -- Feedback visual no círculo
    if settings.aimbotEnabled then
        fovCircle.BorderColor3 = Color3.new(0, 1, 0)
        centerDot.BackgroundColor3 = Color3.new(0, 1, 0)
    else
        fovCircle.BorderColor3 = Color3.new(1, 0.85, 0)
        centerDot.BackgroundColor3 = Color3.new(1, 0.85, 0)
    end
end)

espBtn.MouseButton1Click:Connect(function()
    settings.espEnabled = not settings.espEnabled
    espBtn.Text = settings.espEnabled and "👁️ ESP PLAYERS: ON" or "👁️ ESP PLAYERS: OFF"
    espBtn.BackgroundColor3 = settings.espEnabled and Color3.new(0.2, 0.6, 0.2) or Color3.new(0.6, 0.2, 0.2)
end)

fovInput.FocusLost:Connect(function()
    local newVal = tonumber(fovInput.Text)
    if newVal and newVal >= 60 and newVal <= 300 then
        settings.fovRadius = newVal
        fovLabel.Text = "🎯 FOV RADIUS: " .. settings.fovRadius
        fovCircle.Size = UDim2.new(0, settings.fovRadius * 2, 0, settings.fovRadius * 2)
        fovCircle.Position = UDim2.new(0.5, -settings.fovRadius, 0.5, -settings.fovRadius)
        glow.Size = UDim2.new(0, settings.fovRadius * 2 + 12, 0, settings.fovRadius * 2 + 12)
        glow.Position = UDim2.new(0.5, -settings.fovRadius - 6, 0.5, -settings.fovRadius - 6)
    else
        fovInput.Text = tostring(settings.fovRadius)
    end
end)

-- Atualizações contínuas
RunService.RenderStepped:Connect(updateESP)

-- Teclas de atalho
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.O then
        aimbotBtn.MouseButton1Click:Fire()
    elseif input.KeyCode == Enum.KeyCode.P then
        espBtn.MouseButton1Click:Fire()
    end
end)

print("✅ Sistema carregado! | Círculo AMARELO perfeito | O = Camera Lock | P = ESP")
