local player = game.Players.LocalPlayer
local camera = game.Workspace.CurrentCamera
local predictionTime = 0.15 -- 150 milliseconds
local lockRange = 100 -- Range to lock onto target
local isLocked = false
local target = nil

-- Create GUI components
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "CamlockGui"
screenGui.Parent = player:WaitForChild("PlayerGui")

local button = Instance.new("TextButton")
button.Size = UDim2.new(0, 120, 0, 60)
button.Position = UDim2.new(1, -130, 0, 10)
button.Text = "cam.cc" -- Updated button name
button.Name = "CamlockButton"
button.BackgroundColor3 = Color3.new(1, 1, 1)
button.Parent = screenGui

local dragHandle = Instance.new("Frame")
dragHandle.Size = UDim2.new(0, 20, 0, 60)
dragHandle.Position = UDim2.new(1, -20, 0, 0)
dragHandle.BackgroundColor3 = Color3.fromRGB(255, 0, 0) -- Red color
dragHandle.Parent = button

local notification = Instance.new("TextLabel")
notification.Size = UDim2.new(0, 200, 0, 50)
notification.Position = UDim2.new(1, -210, 1, -60)
notification.Text = "No target locked"
notification.TextColor3 = Color3.new(1, 1, 1)
notification.BackgroundColor3 = Color3.new(0, 0, 0)
notification.Visible = false
notification.Parent = screenGui

-- Dragging functionality
local dragging = false
local dragStart
local startPos

dragHandle.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = button.Position
    end
end)

dragHandle.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement and dragging then
        local delta = input.Position - dragStart
        button.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

dragHandle.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

-- Function to find the nearest target within the lockRange
local function findNearestTarget()
    local closest = nil
    local shortestDistance = lockRange

    for _, v in pairs(game.Players:GetPlayers()) do
        if v ~= player and v.Character and v.Character:FindFirstChild("Head") then
            local distance = (v.Character.Head.Position - player.Character.Head.Position).magnitude
            if distance < shortestDistance then
                closest = v
                shortestDistance = distance
            end
        end
    end

    return closest
end

-- Function to predict target's position after 150ms
local function predictPosition(target)
    if target and target.Character and target.Character:FindFirstChild("Head") then
        local head = target.Character.Head
        local velocity = head.Velocity
        local predictedPosition = head.Position + (velocity * predictionTime)
        return predictedPosition
    end
    return nil
end

-- Function to aimlock the camera to the predicted target position
local function aimlock()
    if target and target.Character and target.Character:FindFirstChild("Head") then
        local predictedPosition = predictPosition(target)
        if predictedPosition then
            local direction = (predictedPosition - camera.CFrame.Position).unit
            local newCFrame = CFrame.new(camera.CFrame.Position, camera.CFrame.Position + direction)
            camera.CFrame = newCFrame
        end
    end
end

-- Function to toggle camlock, only if the button name is correct
local function toggleCamlock()
    if button.Text == "cam.cc" then
        isLocked = not isLocked
        if isLocked then
            target = findNearestTarget()
            if target then
                notification.Text = "Locked onto: " .. target.Name
            else
                notification.Text = "No target found within range."
                isLocked = false
            end
        else
            notification.Text = "Camlock disabled."
        end
        notification.Visible = true
        wait(3)
        notification.Visible = false
    else
        notification.Text = "Error: Button name is incorrect."
        notification.Visible = true
        wait(3)
        notification.Visible = false
    end
end

-- Connect button click to toggleCamlock function
button.MouseButton1Click:Connect(toggleCamlock)

-- Update camera every frame
game:GetService("RunService").RenderStepped:Connect(function()
    if isLocked then
        aimlock()
    end
end)

-- Ensure the GUI persists across respawns
local function onCharacterAdded()
    if not player.PlayerGui:FindFirstChild("CamlockGui") then
        screenGui = Instance.new("ScreenGui")
        screenGui.Name = "CamlockGui"
        screenGui.Parent = player:WaitForChild("PlayerGui")

        button = Instance.new("TextButton")
        button.Size = UDim2.new(0, 120, 0, 60)
        button.Position = UDim2.new(1, -130, 0, 10)
        button.Text = "cam.cc"
        button.Name = "CamlockButton"
        button.BackgroundColor3 = Color3.new(1, 1, 1)
        button.Parent = screenGui

        dragHandle = Instance.new("Frame")
        dragHandle.Size = UDim2.new(0, 20, 0, 60)
        dragHandle.Position = UDim2.new(1, -20, 0, 0)
        dragHandle.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
        dragHandle.Parent = button

        notification = Instance.new("TextLabel")
        notification.Size = UDim2.new(0, 200, 0, 50)
        notification.Position = UDim2.new(1, -210, 1, -60)
        notification.Text = "No target locked"
        notification.TextColor3 = Color3.new(1, 1, 1)
        notification.BackgroundColor3 = Color3.new(0, 0, 0)
        notification.Visible = false
        notification.Parent = screenGui

        dragHandle.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                dragging = true
                dragStart = input.Position
                startPos = button.Position
            end
        end)

        dragHandle.InputChanged:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseMovement and dragging then
                local delta = input.Position - dragStart
                button.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
            end
        end)

        dragHandle.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                dragging = false
            end
        end)

        button.MouseButton1Click:Connect(toggleCamlock)
    end
end

player.CharacterAdded:Connect(onCharacterAdded)
