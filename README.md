local Players = game:GetService("Players")
local camera = workspace.CurrentCamera

local targetPlayer = Players:FindFirstChild("NomeDoJogador")

if targetPlayer and targetPlayer.Character then
    local head = targetPlayer.Character:WaitForChild("Head")

    camera.CameraType = Enum.CameraType.Scriptable

    game:GetService("RunService").RenderStepped:Connect(function()
        camera.CFrame = CFrame.new(
            head.Position + Vector3.new(0, 2, 8),
            head.Position
        )
    end)
end
