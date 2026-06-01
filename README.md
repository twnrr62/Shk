-- LocalScript dentro de StarterGui
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer
local camera = workspace.CurrentCamera
local mouse = player:GetMouse()

-- ========== CONFIGURAÇÕES ==========
local settings = {
    aimbotEnabled = true,
    espEnabled = true,
    aimbotMode = "legit", -- "legit" ou "rage"
    fovRadius = 150,
    espTypes = {
        box = true,
        skeleton = true,
        name = true,
        distance = true,
        line = false
    },
    panelMinimized = false
}

-- ========== VARIÁVEIS DE CONTROLE ==========
local isShooting = false
local currentTarget = nil
local espObjects = {}
local originalCameraCFrame = nil

-- ========== CRIAR INTERFACE ==========
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AimbotSystem"
screenGui.IgnoreGuiInset = true
screenGui.Parent = player:WaitForChild("PlayerGui")

-- ========== CÍRCULO FOV ==========
local fovCircle = Instance.new("Frame")
fovCircle.Size = UDim2.new(0, settings.fovRadius * 2, 0, settings.fovRadius * 2)
fovCircle.Position = UDim2.new(0.5, -settings.fovRadius, 0.5, -settings.fovRadius)
fovCircle.BackgroundTransparency = 1
fovCircle.BorderSizePixel = 3
fovCircle.BorderColor3 = Color3.new(1, 0.85, 0)
fovCircle.ZIndex = 10
fovCircle.Parent = screenGui

local circleCorner = Instance.new("UICorner")
circleCorner.CornerRadius = UDim.new(1, 0)
circleCorner.Parent = fovCircle

-- ========== PAINEL UI ==========
local menuFrame = Instance.new("Frame")
menuFrame.Size = UDim2.new(0, 280, 0, 400)
menuFrame.Position = UDim2.new(0, 10, 0.5, -200)
menuFrame.BackgroundColor3 = Color3.new(0.08, 0.08, 0.12)
menuFrame.BackgroundTransparency = 0.05
menuFrame.BorderSizePixel = 1
menuFrame.BorderColor3 = Color3.new(1, 0.85, 0)
menuFrame.Active = true
menuFrame.Draggable = true
menuFrame.Parent = screenGui

local menuCorner = Instance.new("UICorner")
menuCorner.CornerRadius = UDim.new(0, 8)
menuCorner.Parent = menuFrame

-- Barra de título
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 35)
titleBar.Position = UDim2.new(0, 0, 0, 0)
titleBar.BackgroundColor3 = Color3.new(0.15, 0.15, 0.2)
titleBar.Parent = menuFrame

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 8)
titleCorner.Parent = titleBar

local menuTitle = Instance.new("TextLabel")
menuTitle.Size = UDim2.new(0.8, 0, 1, 0)
menuTitle.Position = UDim2.new(0, 10, 0, 0)
menuTitle.BackgroundTransparency = 1
menuTitle.Text = "⚡ AIMBOT + ESP v3 ⚡"
menuTitle.TextColor3 = Color3.new(1, 0.85, 0)
menuTitle.Font = Enum.Font.GothamBold
menuTitle.TextSize = 12
menuTitle.TextXAlignment = Enum.TextXAlignment.Left
menuTitle.Parent = titleBar

local minimizeBtn = Instance.new("TextButton")
minimizeBtn.Size = UDim2.new(0, 30, 1, 0)
minimizeBtn.Position = UDim2.new(1, -35, 0, 0)
minimizeBtn.BackgroundColor3 = Color3.new(0.25, 0.25, 0.3)
minimizeBtn.Text = "−"
minimizeBtn.TextColor3 = Color3.new(1, 1, 1)
minimizeBtn.TextSize = 18
minimizeBtn.Font = Enum.Font.GothamBold
minimizeBtn.Parent = titleBar

local minimizeCorner = Instance.new("UICorner")
minimizeCorner.CornerRadius = UDim.new(0, 4)
minimizeCorner.Parent = minimizeBtn

local contentContainer = Instance.new("Frame")
contentContainer.Size = UDim2.new(1, 0, 1, -35)
contentContainer.Position = UDim2.new(0, 0, 0, 35)
contentContainer.BackgroundTransparency = 1
contentContainer.Parent = menuFrame

-- Conteúdo do painel
local scrollY = 0

local aimbotSection = Instance.new("TextLabel")
aimbotSection.Size = UDim2.new(0.95, 0, 0, 25)
aimbotSection.Position = UDim2.new(0.025, 0, 0, scrollY)
aimbotSection.BackgroundColor3 = Color3.new(0.15, 0.15, 0.2)
aimbotSection.Text = "🎯 AIMBOT CONFIG"
aimbotSection.TextColor3 = Color3.new(1, 0.85, 0)
aimbotSection.TextSize = 11
aimbotSection.Font = Enum.Font.GothamBold
aimbotSection.Parent = contentContainer
scrollY = scrollY + 30

local aimbotBtn = Instance.new("TextButton")
aimbotBtn.Size = UDim2.new(0.45, 0, 0, 35)
aimbotBtn.Position = UDim2.new(0.025, 0, 0, scrollY)
aimbotBtn.BackgroundColor3 = Color3.new(0.2, 0.6, 0.2)
aimbotBtn.Text = "🔒 ON"
aimbotBtn.TextColor3 = Color3.new(1, 1, 1)
aimbotBtn.Font = Enum.Font.GothamSemibold
aimbotBtn.TextSize = 12
aimbotBtn.Parent = contentContainer

local aimbotBtnCorner = Instance.new("UICorner")
aimbotBtnCorner.CornerRadius = UDim.new(0, 4)
aimbotBtnCorner.Parent = aimbotBtn

local modeBtn = Instance.new("TextButton")
modeBtn.Size = UDim2.new(0.45, 0, 0, 35)
modeBtn.Position = UDim2.new(0.525, 0, 0, scrollY)
modeBtn.BackgroundColor3 = Color3.new(0.3, 0.5, 0.8)
modeBtn.Text = "MODO: LEGIT"
modeBtn.TextColor3 = Color3.new(1, 1, 1)
modeBtn.Font = Enum.Font.GothamSemibold
modeBtn.TextSize = 11
modeBtn.Parent = contentContainer

local modeBtnCorner = Instance.new("UICorner")
modeBtnCorner.CornerRadius = UDim.new(0, 4)
modeBtnCorner.Parent = modeBtn
scrollY = scrollY + 45

local fovLabel = Instance.new("TextLabel")
fovLabel.Size = UDim2.new(0.9, 0, 0, 20)
fovLabel.Position = UDim2.new(0.05, 0, 0, scrollY)
fovLabel.BackgroundTransparency = 1
fovLabel.Text = "🎯 FOV RADIUS: " .. settings.fovRadius
fovLabel.TextColor3 = Color3.new(1, 1, 0.7)
fovLabel.TextSize = 10
fovLabel.Font = Enum.Font.Gotham
fovLabel.TextXAlignment = Enum.TextXAlignment.Left
fovLabel.Parent = contentContainer

local fovInput = Instance.new("TextBox")
fovInput.Size = UDim2.new(0.3, 0, 0, 25)
fovInput.Position = UDim2.new(0.65, 0, 0, scrollY - 2)
fovInput.BackgroundColor3 = Color3.new(0.15, 0.15, 0.2)
fovInput.Text = tostring(settings.fovRadius)
fovInput.TextColor3 = Color3.new(1, 0.85, 0)
fovInput.Font = Enum.Font.Gotham
fovInput.TextSize = 11
fovInput.Parent = contentContainer

local inputCorner = Instance.new("UICorner")
inputCorner.CornerRadius = UDim.new(0, 4)
inputCorner.Parent = fovInput
scrollY = scrollY + 35

local espSection = Instance.new("TextLabel")
espSection.Size = UDim2.new(0.95, 0, 0, 25)
espSection.Position = UDim2.new(0.025, 0, 0, scrollY)
espSection.BackgroundColor3 = Color3.new(0.15, 0.15, 0.2)
espSection.Text = "👁️ ESP TYPES"
espSection.TextColor3 = Color3.new(1, 0.85, 0)
espSection.TextSize = 11
espSection.Font = Enum.Font.GothamBold
espSection.Parent = contentContainer
scrollY = scrollY + 30

local espTypes = {"box", "skeleton", "name", "distance", "line"}
local espButtons = {}
local espBtnY = scrollY

for i, espType in ipairs(espTypes) do
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.18, 0, 0, 30)
    btn.Position = UDim2.new(0.025 + ((i-1) * 0.19), 0, 0, espBtnY)
    btn.BackgroundColor3 = settings.espTypes[espType] and Color3.new(0.2, 0.6, 0.2) or Color3.new(0.5, 0.2, 0.2)
    btn.Text = string.upper(string.sub(espType, 1, 1)) .. string.sub(espType, 2, 3)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.TextSize = 9
    btn.Font = Enum.Font.GothamBold
    btn.Parent = contentContainer
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 4)
    btnCorner.Parent = btn
    
    espButtons[espType] = btn
    
    btn.MouseButton1Click:Connect(function()
        settings.espTypes[espType] = not settings.espTypes[espType]
        btn.BackgroundColor3 = settings.espTypes[espType] and Color3.new(0.2, 0.6, 0.2) or Color3.new(0.5, 0.2, 0.2)
    end)
end
scrollY = scrollY + 40

local espGlobalBtn = Instance.new("TextButton")
espGlobalBtn.Size = UDim2.new(0.9, 0, 0, 35)
espGlobalBtn.Position = UDim2.new(0.05, 0, 0, scrollY)
espGlobalBtn.BackgroundColor3 = Color3.new(0.2, 0.6, 0.2)
espGlobalBtn.Text = "👁️ ESP GLOBAL: ON"
espGlobalBtn.TextColor3 = Color3.new(1, 1, 1)
espGlobalBtn.Font = Enum.Font.GothamSemibold
espGlobalBtn.TextSize = 12
espGlobalBtn.Parent = contentContainer

local espGlobalCorner = Instance.new("UICorner")
espGlobalCorner.CornerRadius = UDim.new(0, 4)
espGlobalCorner.Parent = espGlobalBtn
scrollY = scrollY + 45

local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(0.9, 0, 0, 25)
statusLabel.Position = UDim2.new(0.05, 0, 0, scrollY)
statusLabel.BackgroundColor3 = Color3.new(0.1, 0.1, 0.15)
statusLabel.Text = "● ATIRE PARA MIRAR | FOV: " .. settings.fovRadius
statusLabel.TextColor3 = Color3.new(0, 1, 0)
statusLabel.TextSize = 9
statusLabel.Font = Enum.Font.Gotham
statusLabel.Parent = contentContainer

local statusCorner = Instance.new("UICorner")
statusCorner.CornerRadius = UDim.new(0, 4)
statusCorner.Parent = statusLabel

contentContainer.Size = UDim2.new(1, 0, 0, scrollY + 35)

-- ========== FUNÇÕES DO AIMBOT ==========

-- Encontrar jogador mais próximo
local function getClosestPlayer()
    local myChar = player.Character
    local myRoot = myChar and myChar:FindFirstChild("HumanoidRootPart")
    if not myRoot then return nil end
    
    local closest = nil
    local closestDist = math.huge
    
    for _, otherPlayer in ipairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character then
            local head = otherPlayer.Character:FindFirstChild("Head")
            local hum = otherPlayer.Character:FindFirstChild("Humanoid")
            if head and hum and hum.Health > 0 then
                local dist = (myRoot.Position - head.Position).Magnitude
                if dist < closestDist then
                    closestDist = dist
                    closest = otherPlayer
                end
            end
        end
    end
    return closest
end

-- Encontrar alvo no FOV (para modo Legit)
local function getTargetInFOV()
    local center = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)
    local closest = nil
    local closestDistance = settings.fovRadius
    
    for _, otherPlayer in ipairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character then
            local head = otherPlayer.Character:FindFirstChild("Head")
            local hum = otherPlayer.Character:FindFirstChild("Humanoid")
            if head and hum and hum.Health > 0 then
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

-- Função para mirar no alvo (apenas no tiro)
local function aimAtTarget(target)
    if not target or not target.Character then return end
    
    local head = target.Character:FindFirstChild("Head")
    if head then
        camera.CFrame = CFrame.new(camera.CFrame.Position, head.Position)
    end
end

-- Detectar clique do mouse para atirar
mouse.Button1Down:Connect(function()
    if not settings.aimbotEnabled then return end
    
    isShooting = true
    
    local target = nil
    
    if settings.aimbotMode == "rage" then
        -- RAGE: mira no jogador mais próximo independente da mira
        target = getClosestPlayer()
        if target then
            aimAtTarget(target)
        end
    else
        -- LEGIT: só mira se estiver dentro do FOV
        target = getTargetInFOV()
        if target then
            aimAtTarget(target)
        end
    end
    
    -- Reset após o tiro (delay para não travar a câmera)
    task.wait(0.05)
    isShooting = false
end)

-- ========== ESP COMPLETO ==========
local function getJoints(character)
    local joints = {
        head = character:FindFirstChild("Head"),
        torso = character:FindFirstChild("UpperTorso") or character:FindFirstChild("Torso"),
        leftArm = character:FindFirstChild("LeftArm"),
        rightArm = character:FindFirstChild("RightArm"),
        leftLeg = character:FindFirstChild("LeftLeg"),
        rightLeg = character:FindFirstChild("RightLeg"),
        humanoidRoot = character:FindFirstChild("HumanoidRootPart")
    }
    return joints
end

local function drawSkeleton(character, color, parent)
    local joints = getJoints(character)
    local lines = {}
    
    if not joints.humanoidRoot then return lines end
    
    local connections = {
        {joints.humanoidRoot, joints.torso},
        {joints.torso, joints.head},
        {joints.torso, joints.leftArm},
        {joints.torso, joints.rightArm},
        {joints.torso, joints.leftLeg},
        {joints.torso, joints.rightLeg}
    }
    
    for _, conn in ipairs(connections) do
        local part1, part2 = conn[1], conn[2]
        if part1 and part2 then
            local pos1, onScreen1 = camera:WorldToViewportPoint(part1.Position)
            local pos2, onScreen2 = camera:WorldToViewportPoint(part2.Position)
            
            if onScreen1 and onScreen2 then
                local start = Vector2.new(pos1.X, pos1.Y)
                local finish = Vector2.new(pos2.X, pos2.Y)
                local diff = finish - start
                local length = diff.Magnitude
                local angle = math.atan2(diff.Y, diff.X)
                
                local line = Instance.new("Frame")
                line.Size = UDim2.new(0, length, 0, 2)
                line.Position = UDim2.new(0, start.X, 0, start.Y)
                line.Rotation = math.deg(angle)
                line.BackgroundColor3 = color
                line.BackgroundTransparency = 0.3
                line.BorderSizePixel = 0
                line.ZIndex = 10
                line.Parent = parent
                table.insert(lines, line)
            end
        end
    end
    
    return lines
end

local function drawBox(character, color, parent)
    local root = character:FindFirstChild("HumanoidRootPart")
    local head = character:FindFirstChild("Head")
    
    if root and head then
        local rootPos, onScreen = camera:WorldToViewportPoint(root.Position)
        local headPos = camera:WorldToViewportPoint(head.Position)
        
        if onScreen then
            local height = math.abs(headPos.Y - rootPos.Y) + 20
            local width = height * 0.6
            local yPos = headPos.Y - 10
            
            local box = Instance.new("Frame")
            box.Size = UDim2.new(0, width, 0, height)
            box.Position = UDim2.new(0, rootPos.X - width/2, 0, yPos - height)
            box.BackgroundTransparency = 0.85
            box.BackgroundColor3 = color
            box.BorderSizePixel = 2
            box.BorderColor3 = color
            box.ZIndex = 10
            box.Parent = parent
            
            return {box}
        end
    end
    return {}
end

local function updateESP()
    for _, obj in pairs(espObjects) do
        pcall(function() obj:Destroy() end)
    end
    espObjects = {}
    
    if not settings.espEnabled then return end
    
    local myChar = player.Character
    if not myChar then return end
    
    for _, otherPlayer in ipairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character then
            local char = otherPlayer.Character
            local hum = char:FindFirstChild("Humanoid")
            local head = char:FindFirstChild("Head")
            
            if hum and hum.Health > 0 and head then
                local espColor = Color3.new(1, 0.85, 0)
                local distance = myChar and myChar:FindFirstChild("HumanoidRootPart") and (myChar.HumanoidRootPart.Position - head.Position).Magnitude or 0
                
                local playerESP = Instance.new("Folder")
                playerESP.Name = otherPlayer.Name
                playerESP.Parent = screenGui
                table.insert(espObjects, playerESP)
                
                if settings.espTypes.box then
                    local boxes = drawBox(char, espColor, playerESP)
                    for _, box in ipairs(boxes) do
                        table.insert(espObjects, box)
                    end
                end
                
                if settings.espTypes.skeleton then
                    local skeleton = drawSkeleton(char, espColor, playerESP)
                    for _, bone in ipairs(skeleton) do
                        table.insert(espObjects, bone)
                    end
                end
                
                if settings.espTypes.name or settings.espTypes.distance then
                    local headPos, onScreen = camera:WorldToViewportPoint(head.Position)
                    if onScreen then
                        local text = ""
                        if settings.espTypes.name and settings.espTypes.distance then
                            text = string.format("%s | %.0fm", otherPlayer.Name, distance)
                        elseif settings.espTypes.name then
                            text = otherPlayer.Name
                        elseif settings.espTypes.distance then
                            text = string.format("%.0fm", distance)
                        end
                        
                        local label = Instance.new("TextLabel")
                        label.Size = UDim2.new(0, 140, 0, 20)
                        label.Position = UDim2.new(0, headPos.X - 70, 0, headPos.Y - 30)
                        label.BackgroundTransparency = 0.5
                        label.BackgroundColor3 = Color3.new(0, 0, 0)
                        label.Text = text
                        label.TextColor3 = espColor
                        label.TextSize = 10
                        label.Font = Enum.Font.GothamBold
                        label.BorderSizePixel = 1
                        label.BorderColor3 = espColor
                        label.ZIndex = 10
                        label.Parent = playerESP
                        table.insert(espObjects, label)
                    end
                end
                
                if settings.espTypes.line and myChar and myChar:FindFirstChild("HumanoidRootPart") then
                    local myRoot = myChar.HumanoidRootPart
                    local myPos, myOn = camera:WorldToViewportPoint(myRoot.Position)
                    local headPos, headOn = camera:WorldToViewportPoint(head.Position)
                    
                    if myOn and headOn then
                        local start = Vector2.new(myPos.X, myPos.Y)
                        local finish = Vector2.new(headPos.X, headPos.Y)
                        local diff = finish - start
                        local length = diff.Magnitude
                        local angle = math.atan2(diff.Y, diff.X)
                        
                        local line = Instance.new("Frame")
                        line.Size = UDim2.new(0, length, 0, 2)
                        line.Position = UDim2.new(0, start.X, 0, start.Y)
                        line.Rotation = math.deg(angle)
                        line.BackgroundColor3 = espColor
                        line.BackgroundTransparency = 0.4
                        line.BorderSizePixel = 0
                        line.ZIndex = 5
                        line.Parent = playerESP
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
    aimbotBtn.Text = settings.aimbotEnabled and "🔒 ON" or "🔒 OFF"
    aimbotBtn.BackgroundColor3 = settings.aimbotEnabled and Color3.new(0.2, 0.6, 0.2) or Color3.new(0.6, 0.2, 0.2)
    fovCircle.BorderColor3 = settings.aimbotEnabled and Color3.new(0, 1, 0) or Color3.new(1, 0.85, 0)
end)

modeBtn.MouseButton1Click:Connect(function()
    settings.aimbotMode = settings.aimbotMode == "legit" and "rage" or "legit"
    modeBtn.Text = settings.aimbotMode == "legit" and "MODO: LEGIT" or "MODO: RAGE"
    modeBtn.BackgroundColor3 = settings.aimbotMode == "legit" and Color3.new(0.3, 0.5, 0.8) or Color3.new(0.8, 0.3, 0.3)
    statusLabel.Text = string.format("● ATIRE PARA MIRAR | %s | FOV: %d", string.upper(settings.aimbotMode), settings.fovRadius)
end)

espGlobalBtn.MouseButton1Click:Connect(function()
    settings.espEnabled = not settings.espEnabled
    espGlobalBtn.Text = settings.espEnabled and "👁️ ESP GLOBAL: ON" or "👁️ ESP GLOBAL: OFF"
    espGlobalBtn.BackgroundColor3 = settings.espEnabled and Color3.new(0.2, 0.6, 0.2) or Color3.new(0.6, 0.2, 0.2)
end)

fovInput.FocusLost:Connect(function()
    local newVal = tonumber(fovInput.Text)
    if newVal and newVal >= 50 and newVal <= 300 then
        settings.fovRadius = newVal
        fovLabel.Text = "🎯 FOV RADIUS: " .. settings.fovRadius
        fovCircle.Size = UDim2.new(0, settings.fovRadius * 2, 0, settings.fovRadius * 2)
        fovCircle.Position = UDim2.new(0.5, -settings.fovRadius, 0.5, -settings.fovRadius)
        statusLabel.Text = string.format("● ATIRE PARA MIRAR | %s | FOV: %d", string.upper(settings.aimbotMode), settings.fovRadius)
    else
        fovInput.Text = tostring(settings.fovRadius)
    end
end)

local minimized = false
minimizeBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    contentContainer.Visible = not minimized
    menuFrame.Size = minimized and UDim2.new(0, 280, 0, 35) or UDim2.new(0, 280, 0, 400)
    minimizeBtn.Text = minimized and "+" or "−"
end)

RunService.RenderStepped:Connect(updateESP)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.O then
        aimbotBtn.MouseButton1Click:Fire()
    elseif input.KeyCode == Enum.KeyCode.P then
        espGlobalBtn.MouseButton1Click:Fire()
    elseif input.KeyCode == Enum.KeyCode.M then
        modeBtn.MouseButton1Click:Fire()
    end
end)

print("✅ Sistema carregado! | AIMBOT ATIVADO NO CLICK | O=Toggle | P=ESP | M=Modo")
