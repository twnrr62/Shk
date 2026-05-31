--[[
    COLOQUE ISSO EM UM LocalScript DENTRO DE StarterGui
    Além disso, crie um ModuleScript em ReplicatedStorage chamado "ESPModule"
]]

-- LocalScript (Aimbot + ESP Toggle)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- Configurações
local aimbotEnabled = true
local espEnabled = true
local fovRadius = 150  -- pixels
local aimSmoothness = 0.2
local circleVisible = true

-- Criar círculo FOV na tela
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "TestTools"
screenGui.Parent = player:WaitForChild("PlayerGui")

local fovCircle = Instance.new("Frame")
fovCircle.Size = UDim2.new(0, fovRadius*2, 0, fovRadius*2)
fovCircle.Position = UDim2.new(0.5, -fovRadius, 0.5, -fovRadius)
fovCircle.BackgroundColor3 = Color3.new(1, 0, 0)
fovCircle.BackgroundTransparency = 0.8
fovCircle.BorderSizePixel = 2
fovCircle.BorderColor3 = Color3.new(1, 0, 0)
fovCircle.Visible = circleVisible
fovCircle.Parent = screenGui

-- Criar ESP (exemplo simples, você vai expandir)
local espContainer = Instance.new("Folder", screenGui)
espContainer.Name = "ESP"

-- Função para obter cabeça do alvo
local function getClosestTargetInFOV()
    local center = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)
    local closest = nil
    local closestDist = fovRadius
    
    for _, otherPlayer in ipairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character and otherPlayer.Character:FindFirstChild("Head") then
            local headPos, onScreen = camera:WorldToViewportPoint(otherPlayer.Character.Head.Position)
            if onScreen then
                local screenPos = Vector2.new(headPos.X, headPos.Y)
                local distToCenter = (screenPos - center).Magnitude
                if distToCenter < closestDist then
                    closestDist = distToCenter
                    closest = otherPlayer
                end
            end
        end
    end
    return closest
end

-- ESP com Box, Nome, Articulações e Distância
local function updateESP()
    if not espEnabled then
        for _, obj in ipairs(espContainer:GetChildren()) do
            obj:Destroy()
        end
        return
    end
    
    -- Limpar ESP antigo (mantendo apenas players atuais)
    for _, obj in ipairs(espContainer:GetChildren()) do
        if obj:GetAttribute("player") and not Players:FindFirstChild(obj:GetAttribute("player")) then
            obj:Destroy()
        end
    end
    
    for _, otherPlayer in ipairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character then
            local char = otherPlayer.Character
            local root = char:FindFirstChild("HumanoidRootPart")
            local head = char:FindFirstChild("Head")
            if root and head then
                local pos, onScreen = camera:WorldToViewportPoint(root.Position)
                if onScreen then
                    -- Box ESP (simplificado)
                    local box = Instance.new("Frame")
                    box.Size = UDim2.new(0, 100, 0, 120)
                    box.Position = UDim2.new(0, pos.X - 50, 0, pos.Y - 60)
                    box.BackgroundTransparency = 0.7
                    box.BackgroundColor3 = Color3.new(0, 1, 0)
                    box.BorderSizePixel = 1
                    box.Parent = espContainer
                    box:SetAttribute("player", otherPlayer.Name)
                    
                    -- Nome
                    local nameLabel = Instance.new("TextLabel")
                    nameLabel.Text = otherPlayer.Name
                    nameLabel.Size = UDim2.new(1, 0, 0, 20)
                    nameLabel.Position = UDim2.new(0, 0, 1, 0)
                    nameLabel.BackgroundTransparency = 1
                    nameLabel.TextColor3 = Color3.new(1, 1, 1)
                    nameLabel.TextStrokeTransparency = 0.5
                    nameLabel.Parent = box
                    
                    -- Distância
                    local dist = (camera.CFrame.Position - root.Position).Magnitude
                    local distLabel = Instance.new("TextLabel")
                    distLabel.Text = string.format("%.1fm", dist)
                    distLabel.Size = UDim2.new(1, 0, 0, 20)
                    distLabel.Position = UDim2.new(0, 0, 0, -20)
                    distLabel.BackgroundTransparency = 1
                    distLabel.TextColor3 = Color3.new(1, 1, 0)
                    distLabel.Parent = box
                    
                    -- Articulações (exemplo: braços)
                    local rightArm = char:FindFirstChild("Right Arm")
                    if rightArm then
                        local armPos, armOn = camera:WorldToViewportPoint(rightArm.Position)
                        if armOn then
                            local jointPoint = Instance.new("Frame")
                            jointPoint.Size = UDim2.new(0, 5, 0, 5)
                            jointPoint.Position = UDim2.new(0, armPos.X - 2.5, 0, armPos.Y - 2.5)
                            jointPoint.BackgroundColor3 = Color3.new(1, 0, 1)
                            jointPoint.BackgroundTransparency = 0.3
                            jointPoint.Parent = espContainer
                            jointPoint:SetAttribute("player", otherPlayer.Name)
                        end
                    end
                end
            end
        end
    end
end

-- Aimbot (movimentação da câmera)
RunService.RenderStepped:Connect(function()
    if aimbotEnabled then
        local target = getClosestTargetInFOV()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            local headPos = target.Character.Head.Position
            local camPos = camera.CFrame.Position
            local lookVector = (headPos - camPos).Unit
            local newCFrame = CFrame.new(camPos, camPos + lookVector)
            camera.CFrame = camera.CFrame:Lerp(newCFrame, aimSmoothness)
        end
    end
end)

-- Atualizar ESP a cada frame
RunService.RenderStepped:Connect(updateESP)

-- Toggle com tecla (exemplo: "P" para ESP, "O" para Aimbot)
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.P then
        espEnabled = not espEnabled
        print("ESP toggled:", espEnabled)
    elseif input.KeyCode == Enum.KeyCode.O then
        aimbotEnabled = not aimbotEnabled
        print("Aimbot toggled:", aimbotEnabled)
    end
end)
