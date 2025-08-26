local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer

-- === GUI ===
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AdminRainbowButton"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

local button = Instance.new("ImageButton")
button.Name = "FlyUpButton"
button.Size = UDim2.fromOffset(70, 70)
button.Position = UDim2.new(0.1, 0, 0.8, 0)
button.BackgroundColor3 = Color3.fromHSV(0, 1, 1)
button.AutoButtonColor = false
button.Image = ""
button.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(1, 0)
corner.Parent = button

-- Смайлик (⬆️ маленький, по центру)
local label = Instance.new("TextLabel")
label.Size = UDim2.fromScale(0.6, 0.6) -- уменьшенный
label.Position = UDim2.fromScale(0.2, 0.2) -- центрируем
label.BackgroundTransparency = 1
label.Text = "⬆️"
label.Font = Enum.Font.SourceSansBold
label.TextColor3 = Color3.fromRGB(255, 255, 255)
label.TextSize = 28 -- фиксированный размер
label.Parent = button

-- 🌈 Радужная анимация
task.spawn(function()
    local hue = 0
    while button.Parent do
        hue = (hue + 0.01) % 1
        button.BackgroundColor3 = Color3.fromHSV(hue, 1, 1)
        task.wait(0.05)
    end
end)

-- === Drag (тач и мышь) ===
local dragging = false
local dragStart, startPos

local function update(input)
    local delta = input.Position - dragStart
    button.Position = UDim2.new(
        startPos.X.Scale, startPos.X.Offset + delta.X,
        startPos.Y.Scale, startPos.Y.Offset + delta.Y
    )
end

button.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = button.Position

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        update(input)
    end
end)

-- === Платформа вкл/выкл ===
local platform = nil
local active = false

button.MouseButton1Click:Connect(function()
    active = not active

    if active then
        -- включаем платформу
        local character = player.Character
        if not character or not character:FindFirstChild("HumanoidRootPart") then 
            active = false
            return 
        end
        local root = character.HumanoidRootPart

        platform = Instance.new("Part")
        platform.Size = Vector3.new(6, 1, 6)
        platform.Anchored = true
        platform.CanCollide = true
        platform.Color = Color3.fromRGB(200, 200, 200)
        platform.Transparency = 0.4
        platform.Position = root.Position - Vector3.new(0, 3, 0)
        platform.Parent = workspace

        -- 🕊️ Голубь (декаль)
        local decal = Instance.new("Decal")
        decal.Face = Enum.NormalId.Top
        decal.Texture = "rbxassetid://47109339" -- белый голубь
        decal.Parent = platform

        task.spawn(function()
            while active and platform and platform.Parent and root.Parent do
                platform.Position = root.Position - Vector3.new(0, 3, 0)
                task.wait(0.03)
            end
        end)

    else
        -- выключаем платформу
        if platform then
            platform:Destroy()
            platform = nil
        end
    end
end)
