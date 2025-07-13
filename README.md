local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local localPlayer = Players.LocalPlayer
local camera = workspace.CurrentCamera

local aimbotEnabled = false
local FOV_RADIUS = 150

-- Cria ScreenGui e coloca no PlayerGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "LUCCAHUB"
screenGui.Parent = localPlayer:WaitForChild("PlayerGui")

local frame = Instance.new("Frame", screenGui)
frame.Size = UDim2.new(0, 220, 0, 130)
frame.Position = UDim2.new(0, 10, 0, 10)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BorderSizePixel = 0
frame.BackgroundTransparency = 0.3
frame.Active = true
frame.Draggable = true

local titleLabel = Instance.new("TextLabel", frame)
titleLabel.Size = UDim2.new(1, 0, 0, 20)
titleLabel.Position = UDim2.new(0, 0, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.TextColor3 = Color3.new(1, 1, 1)
titleLabel.Text = "LUCCAHUB               V 1.0"
titleLabel.Font = Enum.Font.SourceSansBold
titleLabel.TextSize = 18
titleLabel.TextXAlignment = Enum.TextXAlignment.Center

local toggleButton = Instance.new("TextButton", frame)
toggleButton.Size = UDim2.new(0, 200, 0, 40)
toggleButton.Position = UDim2.new(0, 10, 0, 25)
toggleButton.Text = "Ligar Aimbot"
toggleButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
toggleButton.TextColor3 = Color3.new(1, 1, 1)
toggleButton.Font = Enum.Font.SourceSansBold
toggleButton.TextSize = 22

local statusLabel = Instance.new("TextLabel", frame)
statusLabel.Size = UDim2.new(0, 200, 0, 20)
statusLabel.Position = UDim2.new(0, 10, 0, 70)
statusLabel.BackgroundTransparency = 1
statusLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
statusLabel.Text = "Status: OFF"
statusLabel.Font = Enum.Font.SourceSans
statusLabel.TextSize = 18
statusLabel.TextXAlignment = Enum.TextXAlignment.Left

local function getClosestEnemy()
    local closest = nil
    local minDist = FOV_RADIUS
    local mousePos = UserInputService:GetMouseLocation()

    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= localPlayer and plr.Team ~= localPlayer.Team and plr.Character and plr.Character:FindFirstChild("Head") then
            local humanoid = plr.Character:FindFirstChild("Humanoid")
            if humanoid and humanoid.Health > 0 then
                local headPos = plr.Character.Head.Position
                local screenPos, onScreen = camera:WorldToViewportPoint(headPos)
                if onScreen then
                    local dist = (Vector2.new(screenPos.X, screenPos.Y) - Vector2.new(mousePos.X, mousePos.Y)).Magnitude
                    if dist < minDist then
                        minDist = dist
                        closest = plr
                    end
                end
            end
        end
    end

    return closest
end

toggleButton.MouseButton1Click:Connect(function()
    aimbotEnabled = not aimbotEnabled
    if aimbotEnabled then
        toggleButton.Text = "Desligar Aimbot"
        statusLabel.Text = "Status: ON"
        statusLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
    else
        toggleButton.Text = "Ligar Aimbot"
        statusLabel.Text = "Status: OFF"
        statusLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
    end
end)

RunService.RenderStepped:Connect(function()
    if not aimbotEnabled then return end

    local success, err = pcall(function()
        local target = getClosestEnemy()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            local headPos = target.Character.Head.Position
            local camPos = camera.CFrame.Position
            camera.CFrame = CFrame.new(camPos, headPos)
        end
    end)

    if not success then
        warn("Erro no aimbot: ", err)
    end
end)
