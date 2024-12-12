local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

local CamlockState = false
local AutoShootState = false
local clickInterval = 0.1
local Prediction = 0.1457102
local HorizontalPrediction = 0.16458
local VerticalPrediction = 0.1422
local enemy = nil
local Minimized = false

getgenv().Key = "q"

function FindNearestEnemy()
    local ClosestDistance, ClosestPlayer = math.huge, nil
    local CenterPosition = Vector2.new(
        game:GetService("GuiService"):GetScreenResolution().X / 2,
        game:GetService("GuiService"):GetScreenResolution().Y / 2
    )

    for _, Player in ipairs(Players:GetPlayers()) do
        if Player ~= LocalPlayer then
            local Character = Player.Character
            if Character and Character:FindFirstChild("HumanoidRootPart") and Character.Humanoid.Health > 0 then
                local Position, IsVisibleOnViewport = workspace.CurrentCamera:WorldToViewportPoint(Character.HumanoidRootPart.Position)

                if IsVisibleOnViewport then
                    local Distance = (CenterPosition - Vector2.new(Position.X, Position.Y)).Magnitude
                    if Distance < ClosestDistance then
                        ClosestPlayer = Character.HumanoidRootPart
                        ClosestDistance = Distance
                    end
                end
            end
        end
    end

    return ClosestPlayer
end

RunService.Heartbeat:Connect(function()
    if CamlockState and enemy then
        local camera = workspace.CurrentCamera
        local predictedPosition = enemy.Position + Vector3.new(
            enemy.Velocity.X * HorizontalPrediction,
            enemy.Velocity.Y * VerticalPrediction,
            enemy.Velocity.Z * Prediction
        )
        camera.CFrame = CFrame.new(camera.CFrame.p, predictedPosition)
    end
end)

local function autoShoot()
    local tool = LocalPlayer.Character:FindFirstChildOfClass("Tool")
    if tool and tool:FindFirstChild("Handle") and enemy then
        local ray = Ray.new(LocalPlayer.Character.Head.Position, (enemy.Position - LocalPlayer.Character.Head.Position).unit * 500)
        local hitPart, hitPosition = workspace:FindPartOnRay(ray, LocalPlayer.Character)
        if hitPart and hitPart:IsDescendantOf(enemy.Parent) then
            tool:Activate()
        end
    end
end

local function startAutoShoot()
    while AutoShootState do
        autoShoot()
        wait(clickInterval)
    end
end

local function createButton(text, parent, size, pos)
    local button = Instance.new("TextButton")
    button.Parent = parent
    button.Text = text
    button.Size = size
    button.Position = pos
    button.BackgroundColor3 = Color3.fromRGB(255, 85, 85)
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.Font = Enum.Font.GothamBold
    button.TextScaled = true
    button.TextWrapped = true
    button.BorderSizePixel = 0

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = button

    return button
end

local function createMinimizeButton(parent, position)
    local button = createButton("-", parent, UDim2.new(0, 20, 0, 20), position)
    button.Text = "-"
    button.BackgroundColor3 = Color3.fromRGB(255, 85, 85)
    return button
end

local function createCloseButton(parent, position)
    local button = createButton("X", parent, UDim2.new(0, 20, 0, 20), position)
    button.Text = "X"
    button.BackgroundColor3 = Color3.fromRGB(255, 85, 85)
    return button
end

local CamLockGui = Instance.new("ScreenGui")
local CamLockFrame = Instance.new("Frame")
local UICornerCamLock = Instance.new("UICorner")

CamLockGui.Name = "CamLockGui"
CamLockGui.Parent = game.CoreGui

CamLockFrame.Parent = CamLockGui
CamLockFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
CamLockFrame.BorderSizePixel = 0
CamLockFrame.Position = UDim2.new(1, -240, 0, 10)
CamLockFrame.Size = UDim2.new(0, 202, 0, 100)
CamLockFrame.Active = true
CamLockFrame.Draggable = true

UICornerCamLock.CornerRadius = UDim.new(0, 10)
UICornerCamLock.Parent = CamLockFrame

local CamLockTitleLabel = Instance.new("TextLabel")
CamLockTitleLabel.Parent = CamLockFrame
CamLockTitleLabel.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
CamLockTitleLabel.BackgroundTransparency = 1
CamLockTitleLabel.Position = UDim2.new(0, -10, 0, 3)
CamLockTitleLabel.Size = UDim2.new(1, 0, 0, 30)
CamLockTitleLabel.Font = Enum.Font.GothamBold
CamLockTitleLabel.Text = "CamLock"
CamLockTitleLabel.TextColor3 = Color3.fromRGB(255, 85, 85)
CamLockTitleLabel.TextSize = 24
CamLockTitleLabel.TextScaled = true

local CamLockButton = createButton("Toggle CamLock", CamLockFrame, UDim2.new(0, 170, 0, 44), UDim2.new(0.1, 0, 0.4, 0))
CamLockButton.MouseButton1Click:Connect(function()
    CamlockState = not CamlockState
    if CamlockState then
        CamLockButton.Text = "CamLock ON"
        enemy = FindNearestEnemy()
    else
        CamLockButton.Text = "CamLock OFF"
        enemy = nil
    end
end)

local MinimizeCamLockButton = createMinimizeButton(CamLockFrame, UDim2.new(0.9, -30, 0, 5))
local CloseCamLockButton = createCloseButton(CamLockFrame, UDim2.new(0.9, -60, 0, 5))

CloseCamLockButton.MouseButton1Click:Connect(function()
    CamLockGui:Destroy()
end)

MinimizeCamLockButton.MouseButton1Click:Connect(function()
    Minimized = not Minimized
    if Minimized then
        CamLockFrame.Size = UDim2.new(0, 100, 0, 40)
        CamLockTitleLabel.Visible = false
        CamLockButton.Visible = false
        MinimizeCamLockButton.Text = "+"
    else
        CamLockFrame.Size = UDim2.new(0, 202, 0, 100)
        CamLockTitleLabel.Visible = true
        CamLockButton.Visible = true
        MinimizeCamLockButton.Text = "-"
    end
end)

local AutoShootGui = Instance.new("ScreenGui")
local AutoShootFrame = Instance.new("Frame")
local UICornerAutoShoot = Instance.new("UICorner")

AutoShootGui.Name = "AutoShootGui"
AutoShootGui.Parent = game.CoreGui

AutoShootFrame.Parent = AutoShootGui
AutoShootFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
AutoShootFrame.BorderSizePixel = 0
AutoShootFrame.Position = UDim2.new(1, -240, 0, 150)
AutoShootFrame.Size = UDim2.new(0, 202, 0, 100)
AutoShootFrame.Active = true
AutoShootFrame.Draggable = true

UICornerAutoShoot.CornerRadius = UDim.new(0, 10)
UICornerAutoShoot.Parent = AutoShootFrame

local AutoShootTitleLabel = Instance.new("TextLabel")
AutoShootTitleLabel.Parent = AutoShootFrame
AutoShootTitleLabel.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
AutoShootTitleLabel.BackgroundTransparency = 1
AutoShootTitleLabel.Position = UDim2.new(0, -10, 0, 3)
AutoShootTitleLabel.Size = UDim2.new(1, 0, 0, 30)
AutoShootTitleLabel.Font = Enum.Font.GothamBold
AutoShootTitleLabel.Text = "AutoShoot"
AutoShootTitleLabel.TextColor3 = Color3.fromRGB(255, 85, 85)
AutoShootTitleLabel.TextSize = 24
AutoShootTitleLabel.TextScaled = true

local AutoShootButton = createButton("Toggle AutoShoot", AutoShootFrame, UDim2.new(0, 170, 0, 44), UDim2.new(0.1, 0, 0.4, 0))
AutoShootButton.MouseButton1Click:Connect(function()
    AutoShootState = not AutoShootState
    if AutoShootState then
        AutoShootButton.Text = "AutoShoot ON"
        spawn(startAutoShoot)
    else
        AutoShootButton.Text = "AutoShoot OFF"
    end
end)

local MinimizeAutoShootButton = createMinimizeButton(AutoShootFrame, UDim2.new(0.9, -30, 0, 5))
local CloseAutoShootButton = createCloseAutoShootButton.MouseButton1Click:Connect(function()
    AutoShootGui:Destroy()
end)

MinimizeAutoShootButton.MouseButton1Click:Connect(function()
    Minimized = not Minimized
    if Minimized then
        AutoShootFrame.Size = UDim2.new(0, 100, 0, 40)
        AutoShootTitleLabel.Visible = false
        AutoShootButton.Visible = false
        MinimizeAutoShootButton.Text = "+"
    else
        AutoShootFrame.Size = UDim2.new(0, 202, 0, 100)
        AutoShootTitleLabel.Visible = true
        AutoShootButton.Visible = true
        MinimizeAutoShootButton.Text = "-"
    end
end)
