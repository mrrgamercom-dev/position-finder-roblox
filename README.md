--// Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer

--// Executor-friendly GUI parent
local parentGui = gethui and gethui() or game:GetService("CoreGui")

-- Remove existing UI if re-executed
if parentGui:FindFirstChild("PositionFinderUI") then
    parentGui.PositionFinderUI:Destroy()
end

--// Create ScreenGui
local gui = Instance.new("ScreenGui")
gui.Name = "PositionFinderUI"
gui.ResetOnSpawn = false
gui.Parent = parentGui

--// Main Frame
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 300, 0, 170)
frame.Position = UDim2.new(0.5, -150, 0.5, -85)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BorderSizePixel = 0
frame.Active = true -- Critical for dragging
frame.Parent = gui

-- Rounded corners
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = frame

-- Title
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 30)
title.BackgroundTransparency = 1
title.Text = "Position Finder"
title.Font = Enum.Font.GothamBold
title.TextSize = 16
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Parent = frame

-- Position Label
local posLabel = Instance.new("TextLabel")
posLabel.Size = UDim2.new(1, -20, 0, 40)
posLabel.Position = UDim2.new(0, 10, 0, 35)
posLabel.BackgroundTransparency = 1
posLabel.Font = Enum.Font.Code
posLabel.TextSize = 14
posLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
posLabel.TextWrapped = true
posLabel.Text = "Vector3: Loading..."
posLabel.Parent = frame

-- CFrame Label
local cfLabel = Instance.new("TextLabel")
cfLabel.Size = UDim2.new(1, -20, 0, 40)
cfLabel.Position = UDim2.new(0, 10, 0, 75)
cfLabel.BackgroundTransparency = 1
cfLabel.Font = Enum.Font.Code
cfLabel.TextSize = 14
cfLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
cfLabel.TextWrapped = true
cfLabel.Text = "CFrame: Loading..."
cfLabel.Parent = frame

-- Button Helper Function
local function createButton(text, position, color)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0.45, -5, 0, 30)
    button.Position = position
    button.BackgroundColor3 = color
    button.Text = text
    button.Font = Enum.Font.GothamBold
    button.TextSize = 14
    button.TextColor3 = Color3.new(1, 1, 1)
    button.Parent = frame

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = button

    return button
end

-- Buttons
local copyPosBtn = createButton("Copy Vector3", UDim2.new(0, 10, 1, -40), Color3.fromRGB(70, 130, 180))
local copyCFBtn = createButton("Copy CFrame", UDim2.new(0.55, 5, 1, -40), Color3.fromRGB(60, 179, 113))

--// Dragging Logic
local dragging, dragInput, dragStart, startPos

local function update(input)
    local delta = input.Position - dragStart
    frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

frame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = frame.Position

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

frame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        update(input)
    end
end)

--// Character Data Logic
local function getHRP()
    return player.Character and player.Character:FindFirstChild("HumanoidRootPart")
end

-- Live position update
RunService.RenderStepped:Connect(function()
    local hrp = getHRP()
    if hrp then
        local pos = hrp.Position
        local cf = hrp.CFrame

        posLabel.Text = string.format("Vector3.new(%.2f, %.2f, %.2f)", pos.X, pos.Y, pos.Z)
        cfLabel.Text = string.format("CFrame.new(%.2f, %.2f, %.2f)", cf.X, cf.Y, cf.Z)
    end
end)

-- Clipboard copy functionality
local function copyToClipboard(text, button)
    if setclipboard then
        setclipboard(text)
        local original = button.Text
        button.Text = "Copied!"
        task.delay(1, function() button.Text = original end)
    else
        button.Text = "Not Supported"
        task.delay(1, function() button.Text = original end)
    end
end

copyPosBtn.MouseButton1Click:Connect(function()
    local hrp = getHRP()
    if hrp then
        local pos = hrp.Position
        copyToClipboard(string.format("Vector3.new(%.2f, %.2f, %.2f)", pos.X, pos.Y, pos.Z), copyPosBtn)
    end
end)

copyCFBtn.MouseButton1Click:Connect(function()
    local hrp = getHRP()
    if hrp then
        local cf = hrp.CFrame
        copyToClipboard(string.format("CFrame.new(%.2f, %.2f, %.2f)", cf.X, cf.Y, cf.Z), copyCFBtn)
    end
end)

