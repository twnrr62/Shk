-- LocalScript dentro de StarterGui (ou em um ScreenGui)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- Configurações
local aimbotEnabled = true
local fovRadius = 120  -- pixels (raio do círculo - ajustável)
local aimSmoothness = 0.15

-- Criar interface
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AimbotTest"
screenGui.IgnoreGuiInset = true  -- Ignora insets da interface do Roblox
screenGui.Parent = player:WaitForChild("PlayerGui")

-- Círculo FOV (bem visível)
local fovCircle = Instance.new("Frame")
fovCircle.Size = UDim2.new(0, fovRadius * 2, 0, fovRadius * 2)
fovCircle.Position = UDim2.new(0.5, -fovRadius, 0.5, -fovRadius)  -- Centralizado
fovCircle.BackgroundColor3 = Color3.new(1, 0, 0)
fovCircle.BackgroundTransparency = 0.85
fovCircle.BorderSizePixel = 3
fovCircle.BorderColor3 = Color3.new(1, 0, 0)
fovCircle.Visible = true
fovCircle.ZIndex = 10  -- Fica acima de outros elementos
fovCircle.Parent = screenGui

-- Efeito de borda mais grossa (opcional: contorno interno)
local innerBorder = Instance.new("Frame")
innerBorder.Size = UDim2.new(1, -6, 1, -6)
innerBorder.Position = UDim2.new(0, 3, 0, 3)
innerBorder.BackgroundTransparency = 1
innerBorder.BorderSizePixel = 1
innerBorder.BorderColor3 = Color3.new(1, 1, 1)
innerBorder.Parent = fovCircle

-- Ponto central (bem visível)
local centerDot = Instance.new("Frame")
centerDot.Size = UDim2.new(0, 6, 0, 6)
centerDot.Position = UDim2.new(0.5, -3, 0.5, -3)
centerDot.BackgroundColor3 = Color3.new(1, 1, 1)
centerDot.BackgroundTransparency = 0
centerDot.BorderSizePixel = 1
centerDot.BorderColor3 = Color3.new(0, 0, 0)
centerDot.ZIndex = 10
centerDot.Parent = screenGui

-- Cruz de mira no centro (opcional, ajuda a mirar)
local crosshair = Instance.new("Frame")
crosshair.Size = UDim2.new(0, 2, 0, 16)
crosshair.Position = UDim2.new(0.5, -1, 0.5, -8)
crosshair.BackgroundColor3 = Color3.new(1, 1, 1)
crosshair.BackgroundTransparency = 0.3
crosshair.Parent = screenGui

local crosshair2 = Instance.new("Frame")
crosshair2.Size = UDim2.new(0, 16, 0, 2)
crosshair2.Position = UDim2.new(0.5, -8, 0.5, -1)
crosshair2.BackgroundColor3 = Color3.new(1, 1, 1)
crosshair2.BackgroundTransparency = 0.3
crosshair2.Parent = screenGui

-- Função para encontrar o inimigo mais próximo do centro
local function getClosestTargetInFOV()
    local center = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)
    local closest = nil
    local closestDistance = fovRadius
    
    for _, otherPlayer in ipairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character then
            local head = otherPlayer.Character:FindFirstChild("Head")
            if head then
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

-- Aimbot
RunService.RenderStepped:Connect(function()
    if not aimbotEnabled then return end
    
    local target = getClosestTargetInFOV()
    if target and target.Character and target.Character:FindFirstChild("Head") then
        local headPos = target.Character.Head.Position
        local camPos = camera.CFrame.Position
        local direction = (headPos - camPos).Unit
        local targetCFrame = CFrame.new(camPos, camPos + direction)
        
        camera.CFrame = camera.CFrame:Lerp(targetCFrame, aimSmoothness)
    end
end)

-- Tecla O para ligar/desligar
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.O then
        aimbotEnabled = not aimbotEnabled
        
        if aimbotEnabled then
            fovCircle.BorderColor3 = Color3.new(0, 1, 0)  -- Verde = ativo
            fovCircle.BackgroundColor3 = Color3.new(0, 1, 0)
            centerDot.BackgroundColor3 = Color3.new(0, 1, 0)
        else
            fovCircle.BorderColor3 = Color3.new(1, 0, 0)  -- Vermelho = inativo
            fovCircle.BackgroundColor3 = Color3.new(1, 0, 0)
            centerDot.BackgroundColor3 = Color3.new(1, 1, 1)
        end
    end
end)

-- Ajustar tamanho do círculo se a tela redimensionar (opcional)
camera:GetPropertyChangedSignal("ViewportSize"):Connect(function()
    fovCircle.Size = UDim2.new(0, fovRadius * 2, 0, fovRadius * 2)
    fovCircle.Position = UDim2.new(0.5, -fovRadius, 0.5, -fovRadius)
end)
