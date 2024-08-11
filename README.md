local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

-- Create the ScreenGui and Frame
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = game.CoreGui  -- Parent to CoreGui for better compatibility with executors
screenGui.Name = "HighlightGui"

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0.3, 0, 0.2, 0) -- Width: 30% of the screen, Height: 20% of the screen
mainFrame.Position = UDim2.new(0.35, 0, 0.4, 0) -- Centered position
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30) -- Dark background color
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui

-- Make the frame draggable
local dragToggle = nil
local dragInput = nil
local dragStart = nil
local startPos = nil

local function updateInput(input)
    local delta = input.Position - dragStart
    mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

mainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragToggle = true
        dragStart = input.Position
        startPos = mainFrame.Position

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragToggle = false
            end
        end)
    end
end)

mainFrame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

RunService.Heartbeat:Connect(function()
    if dragToggle then
        updateInput(dragInput)
    end
end)

-- Minimize Button
local minimizeButton = Instance.new("TextButton")
minimizeButton.Size = UDim2.new(0.1, 0, 0.2, 0) -- 10% width, 20% height of the mainFrame
minimizeButton.Position = UDim2.new(0.9, -5, 0, 5) -- Top right corner
minimizeButton.Text = "-"
minimizeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
minimizeButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
minimizeButton.BorderSizePixel = 0
minimizeButton.Parent = mainFrame

local isMinimized = false

minimizeButton.MouseButton1Click:Connect(function()
    if isMinimized then
        mainFrame.Size = UDim2.new(0.3, 0, 0.2, 0)
        minimizeButton.Text = "-"
    else
        mainFrame.Size = UDim2.new(0, 150, 0, 30)  -- Smaller size when minimized
        minimizeButton.Text = "+"
    end
    isMinimized = not isMinimized
end)

-- Toggle Button for Highlights
local toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(0.3, 0, 0.3, 0) -- Smaller toggle button
toggleButton.Position = UDim2.new(0.05, 0, 0.2, 0) -- Centered within the frame
toggleButton.Text = "Off"
toggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0) -- Red color
toggleButton.BorderSizePixel = 0
toggleButton.Parent = mainFrame

local scriptEnabled = false

-- Function to create or remove the highlight
local function toggleHighlight()
    if scriptEnabled then
        -- Add highlights
        for _, player in ipairs(Players:GetPlayers()) do
            if player.Character and not player.Character:FindFirstChild("Highlight") then
                createOutline(player.Character)
            end
        end
    else
        -- Remove all highlights
        for _, player in ipairs(Players:GetPlayers()) do
            if player.Character then
                local highlight = player.Character:FindFirstChild("Highlight")
                if highlight then
                    highlight:Destroy()
                end
            end
        end
    end
end

-- Function to create an outline
local function createOutline(character)
    local highlight = Instance.new("Highlight")
    highlight.Parent = character
    highlight.Adornee = character
    highlight.OutlineColor = Color3.fromRGB(255, 0, 0)  -- Change to your desired color
    highlight.OutlineTransparency = 0.5  -- Adjust transparency if needed
end

-- Toggle button functionality
toggleButton.MouseButton1Click:Connect(function()
    scriptEnabled = not scriptEnabled
    toggleHighlight()
    if scriptEnabled then
        toggleButton.Text = "On"
        toggleButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0) -- Green color
    else
        toggleButton.Text = "Off"
        toggleButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0) -- Red color
    end
end)

-- Continuously update highlights if the script is enabled
RunService.RenderStepped:Connect(function()
    if scriptEnabled then
        for _, player in ipairs(Players:GetPlayers()) do
            if player.Character and not player.Character:FindFirstChild("Highlight") then
                createOutline(player.Character)
            end
        end
    end
end)

-- Connect to players who join the game later
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        if scriptEnabled then
            createOutline(character)
        end
    end)
end)

-- Walk Speed Button
local walkSpeedButton = Instance.new("TextButton")
walkSpeedButton.Size = UDim2.new(0.3, 0, 0.2, 0) -- Size of the button
walkSpeedButton.Position = UDim2.new(0.05, 0, 0.6, 0) -- Positioned below the toggle button
walkSpeedButton.Text = "WalkSpeed"
walkSpeedButton.TextColor3 = Color3.fromRGB(255, 255, 255)
walkSpeedButton.BackgroundColor3 = Color3.fromRGB(0, 0, 255) -- Blue color
walkSpeedButton.BorderSizePixel = 0
walkSpeedButton.Parent = mainFrame

local walkSpeed = 16  -- Default WalkSpeed
walkSpeedButton.MouseButton1Click:Connect(function()
    walkSpeed = walkSpeed + 10  -- Increase WalkSpeed by 10
    local character = Players.LocalPlayer.Character
    if character then
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid.WalkSpeed = walkSpeed
        end
    end
end)

-- Jump Power Button
local jumpPowerButton = Instance.new("TextButton")
jumpPowerButton.Size = UDim2.new(0.3, 0, 0.2, 0) -- Size of the button
jumpPowerButton.Position = UDim2.new(0.6, 0, 0.6, 0) -- Positioned next to WalkSpeed button
jumpPowerButton.Text = "JumpPower"
jumpPowerButton.TextColor3 = Color3.fromRGB(255, 255, 255)
jumpPowerButton.BackgroundColor3 = Color3.fromRGB(255, 165, 0) -- Orange color
jumpPowerButton.BorderSizePixel = 0
jumpPowerButton.Parent = mainFrame

local jumpPower = 50  -- Default JumpPower
jumpPowerButton.MouseButton1Click:Connect(function()
    jumpPower = jumpPower + 10  -- Increase JumpPower by 10
    local character = Players.LocalPlayer.Character
    if character then
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid.JumpPower = jumpPower
        end
    end
end)
