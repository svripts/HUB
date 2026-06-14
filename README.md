-- ============================================================
-- LOGGED BY @DARKHUb
-- FEAR DUEL BAT AIMBOT (COMPLETE - All features)
-- ============================================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local LP = Players.LocalPlayer

-- Settings (from log)
local AIMBOT_SPEED = 56.5
local HIT_DISTANCE = 5
local SWING_COOLDOWN = 0.08
local BEHIND_DISTANCE = -3
local BASE_PREDICTION = 0.15
local FACING_PREDICTION = 1.5
local PREDICTION_RADIUS_LIMIT = 10

-- State
local aimbotActive = false
local hittingCooldown = false
local aimbotConnection = nil

-- Bat tool names (from log)
local BAT_NAMES = {
    "Bat", "Slap", "Iron Slap", "Gold Slap", "Diamond Slap",
    "Emerald Slap", "Ruby Slap", "Dark Matter Slap", "Flame Slap",
    "Nuclear Slap", "Galaxy Slap", "Glitched Slap"
}

-- ============================================================
-- GUI CREATION
-- ============================================================

-- Clean up old GUI if exists
pcall(function()
    local old = game:GetService("CoreGui"):FindFirstChild("BlackskillAimbotGUI")
    if old then old:Destroy() end
end)

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "BlackskillAimbotGUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = game:GetService("CoreGui")

-- Main Frame
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 200, 0, 80)
MainFrame.Position = UDim2.new(0.5, -100, 0.85, -40)
MainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 18)
MainFrame.BackgroundTransparency = 0.1
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

-- Corner
local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 12)
MainCorner.Parent = MainFrame

-- Stroke
local MainStroke = Instance.new("UIStroke")
MainStroke.Thickness = 1.5
MainStroke.Color = Color3.fromRGB(100, 100, 120)
MainStroke.Parent = MainFrame

-- Title Label (Logged by @blackskillisue)
local TitleLabel = Instance.new("TextLabel")
TitleLabel.Size = UDim2.new(1, -10, 0, 22)
TitleLabel.Position = UDim2.new(0, 8, 0, 6)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "Logged by @blackskillisue"
TitleLabel.TextColor3 = Color3.fromRGB(180, 180, 200)
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.TextSize = 10
TitleLabel.TextXAlignment = Enum.TextXAlignment.Center
TitleLabel.Parent = MainFrame

-- Divider Line
local Divider = Instance.new("Frame")
Divider.Size = UDim2.new(1, -16, 0, 1)
Divider.Position = UDim2.new(0, 8, 0, 30)
Divider.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
Divider.BorderSizePixel = 0
Divider.Parent = MainFrame

-- Aimbot Label
local AimbotLabel = Instance.new("TextLabel")
AimbotLabel.Size = UDim2.new(0, 80, 0, 28)
AimbotLabel.Position = UDim2.new(0, 12, 0, 38)
AimbotLabel.BackgroundTransparency = 1
AimbotLabel.Text = "Bat Aimbot"
AimbotLabel.TextColor3 = Color3.fromRGB(220, 220, 240)
AimbotLabel.Font = Enum.Font.GothamBold
AimbotLabel.TextSize = 14
AimbotLabel.TextXAlignment = Enum.TextXAlignment.Left
AimbotLabel.Parent = MainFrame

-- Pill (Toggle Background)
local Pill = Instance.new("Frame")
Pill.Size = UDim2.new(0, 44, 0, 24)
Pill.Position = UDim2.new(1, -56, 0.5, -12)
Pill.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
Pill.BorderSizePixel = 0
Pill.Parent = MainFrame

local PillCorner = Instance.new("UICorner")
PillCorner.CornerRadius = UDim.new(0, 12)
PillCorner.Parent = Pill

local PillStroke = Instance.new("UIStroke")
PillStroke.Thickness = 1
PillStroke.Color = Color3.fromRGB(55, 55, 70)
PillStroke.Parent = Pill

-- Dot (Toggle Indicator)
local Dot = Instance.new("Frame")
Dot.Size = UDim2.new(0, 16, 0, 16)
Dot.Position = UDim2.new(0, 3, 0.5, -8)
Dot.BackgroundColor3 = Color3.fromRGB(80, 80, 100)
Dot.BorderSizePixel = 0
Dot.Parent = Pill

local DotCorner = Instance.new("UICorner")
DotCorner.CornerRadius = UDim.new(0, 8)
DotCorner.Parent = Dot

-- Status Label
local StatusLabel = Instance.new("TextLabel")
StatusLabel.Size = UDim2.new(0, 50, 0, 16)
StatusLabel.Position = UDim2.new(1, -56, 0, 58)
StatusLabel.BackgroundTransparency = 1
StatusLabel.Text = "OFF"
StatusLabel.TextColor3 = Color3.fromRGB(150, 150, 170)
StatusLabel.Font = Enum.Font.GothamBold
StatusLabel.TextSize = 9
StatusLabel.TextXAlignment = Enum.TextXAlignment.Right
StatusLabel.Parent = MainFrame

-- Click Button (invisible overlay for toggle)
local ToggleButton = Instance.new("TextButton")
ToggleButton.Size = UDim2.new(0, 60, 0, 28)
ToggleButton.Position = UDim2.new(1, -68, 0.5, -14)
ToggleButton.BackgroundTransparency = 1
ToggleButton.Text = ""
ToggleButton.Parent = MainFrame

-- ============================================================
-- ANIMATION FUNCTIONS
-- ============================================================

local function animateToggle(on)
    local targetPillColor = on and Color3.fromRGB(70, 100, 130) or Color3.fromRGB(35, 35, 45)
    local targetDotPos = on and UDim2.new(1, -19, 0.5, -8) or UDim2.new(0, 3, 0.5, -8)
    local targetDotColor = on and Color3.fromRGB(200, 220, 255) or Color3.fromRGB(80, 80, 100)
    local targetStatusText = on and "ON" or "OFF"
    local targetStatusColor = on and Color3.fromRGB(150, 200, 255) or Color3.fromRGB(150, 150, 170)
    
    TweenService:Create(Pill, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {
        BackgroundColor3 = targetPillColor
    }):Play()
    
    TweenService:Create(Dot, TweenInfo.new(0.2, Enum.EasingStyle.Back), {
        Position = targetDotPos,
        BackgroundColor3 = targetDotColor
    }):Play()
    
    TweenService:Create(StatusLabel, TweenInfo.new(0.15), {
        TextColor3 = targetStatusColor
    }):Play()
    
    StatusLabel.Text = targetStatusText
end

-- ============================================================
-- BAT AIMBOT FUNCTIONS
-- ============================================================

-- Get current bat tool
local function getBat()
    local char = LP.Character
    if not char then return nil end
    
    for _, name in ipairs(BAT_NAMES) do
        local tool = char:FindFirstChild(name)
        if tool and tool:IsA("Tool") then
            return tool
        end
    end
    
    local backpack = LP:FindFirstChild("Backpack")
    if backpack then
        for _, name in ipairs(BAT_NAMES) do
            local tool = backpack:FindFirstChild(name)
            if tool and tool:IsA("Tool") then
                local hum = char:FindFirstChildOfClass("Humanoid")
                if hum then
                    pcall(function() hum:EquipTool(tool) end)
                end
                return tool
            end
        end
    end
    return nil
end

-- Swing the bat
local function swingBat(targetChar)
    if hittingCooldown then return end
    hittingCooldown = true
    
    pcall(function()
        local bat = getBat()
        if bat then
            -- Touch handle for hit registration
            local handle = bat:FindFirstChild("Handle")
            if handle and targetChar then
                local targetHead = targetChar:FindFirstChild("Head")
                if targetHead then
                    pcall(function()
                        firetouchinterest(handle, targetHead, 0)
                        firetouchinterest(handle, targetHead, 1)
                    end)
                end
            end
            
            local remote = bat:FindFirstChildWhichIsA("RemoteEvent")
            if remote then
                pcall(function() remote:FireServer() end)
            else
                pcall(function() bat:Activate() end)
            end
        end
    end)
    
    task.delay(SWING_COOLDOWN, function()
        hittingCooldown = false
    end)
end

-- Get target parts based on enemy position (Head/Chest/Legs)
local function getTargetParts(targetChar, myPos)
    local targetHead = targetChar:FindFirstChild("Head")
    local targetHRP = targetChar:FindFirstChild("HumanoidRootPart")
    local targetUpper = targetChar:FindFirstChild("UpperTorso")
    local targetLower = targetChar:FindFirstChild("LowerTorso")
    
    if not targetHead or not targetHRP then
        return targetHRP, targetHRP
    end
    
    local headY = targetHead.Position.Y
    local myY = myPos.Y
    
    -- DYNAMIC HITBOX (from log)
    if headY > myY + 3 then
        -- Enemy ABOVE you -> target HEAD
        return targetHead, targetHead
    elseif myY > headY + 3 then
        -- Enemy BELOW you -> target LEGS
        return targetLower or targetHRP, targetLower or targetHRP
    else
        -- Enemy SAME LEVEL -> target CHEST
        return targetUpper or targetHRP, targetUpper or targetHRP
    end
end

-- Get closest enemy with full data
local function getClosestEnemy()
    local char = LP.Character
    if not char then return nil, math.huge, nil, nil, nil end
    
    local myHRP = char:FindFirstChild("HumanoidRootPart")
    if not myHRP then return nil, math.huge, nil, nil, nil end
    
    local myPos = myHRP.Position
    local closest, closestDist = nil, math.huge
    local closestChar, closestHRP, closestHead, closestTargetPart = nil, nil, nil, nil
    
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LP then
            local targetChar = player.Character
            if targetChar then
                local targetHRP = targetChar:FindFirstChild("HumanoidRootPart")
                local targetHum = targetChar:FindFirstChildOfClass("Humanoid")
                
                if targetHRP and targetHum and targetHum.Health > 0 then
                    local dist = (myHRP.Position - targetHRP.Position).Magnitude
                    if dist < closestDist then
                        closestDist = dist
                        closest = player
                        closestChar = targetChar
                        closestHRP = targetHRP
                        closestHead = targetChar:FindFirstChild("Head")
                        
                        -- Get appropriate target part based on position
                        local targetPart, _ = getTargetParts(targetChar, myPos)
                        closestTargetPart = targetPart
                    end
                end
            end
        end
    end
    
    return closest, closestDist, closestChar, closestHRP, closestHead, closestTargetPart
end

-- Main aimbot function (COMPLETE with all log features)
local function startAimbot()
    if aimbotConnection then return end
    
    aimbotActive = true
    aimbotConnection = RunService.Heartbeat:Connect(function()
        if not aimbotActive then return end
        
        local char = LP.Character
        if not char then return end
        
        local myHRP = char:FindFirstChild("HumanoidRootPart")
        if not myHRP then return end
        
        local myPos = myHRP.Position
        
        local target, dist, targetChar, targetHRP, targetHead, targetPart = getClosestEnemy()
        
        if target and targetChar and targetHRP and targetPart then
            -- Get enemy's current velocity
            local currentVel = targetHRP.AssemblyLinearVelocity
            local targetVelY = currentVel.Y
            
            -- Get positions
            local targetPos = targetPart.Position
            local headPos = targetHead and targetHead.Position or targetPos
            
            -- ============================================================
            -- FACING PREDICTION (enemy's look direction)
            -- ============================================================
            local facingOffset = targetHRP.CFrame.LookVector * FACING_PREDICTION
            local predictedPos = targetPos + facingOffset + (currentVel * BASE_PREDICTION)
            
            -- ============================================================
            -- PREDICTION RADIUS CLAMP (from log)
            -- ============================================================
            local predOffset = predictedPos - targetPos
            if predOffset.Magnitude > PREDICTION_RADIUS_LIMIT then
                predictedPos = targetPos + predOffset.Unit * PREDICTION_RADIUS_LIMIT
            end
            
            -- ============================================================
            -- HOLY HORIZONTAL APPROACH (from log)
            -- ============================================================
            local flatDir = Vector3.new(
                predictedPos.X - myPos.X,
                0,
                predictedPos.Z - myPos.Z
            )
            
            -- Move behind enemy (not into them)
            local standPos = predictedPos - (flatDir.Unit * BEHIND_DISTANCE)
            
            -- ============================================================
            -- Y TRACKING (Air tracking - from log) - UPDATED TO USE 20
            -- ============================================================
            local myY = myPos.Y
            local headY = headPos.Y
            
            -- Predict Y position based on enemy's vertical velocity
            local predictedY = headY + (targetVelY * BASE_PREDICTION)
            
            -- Clamp Y prediction (good addition)
            if math.abs(predictedY - headY) > 8 then
                predictedY = headY + (predictedY - headY) / math.abs(predictedY - headY) * 8
            end
            
            -- Calculate Y velocity for tracking - USING 20 INSTEAD OF 15
            local yVel = math.clamp((headY - myY) * 20, -120, 120)
            
            -- ============================================================
            -- FINAL AIM POINT
            -- ============================================================
            local finalAimPoint = Vector3.new(
                standPos.X,
                predictedY,
                standPos.Z
            )
            
            -- ============================================================
            -- MOVEMENT
            -- ============================================================
            local moveDir = finalAimPoint - myPos
            local finalDir = moveDir.Magnitude > 0.5 and moveDir.Unit or flatDir.Unit
            
            myHRP.AssemblyLinearVelocity = Vector3.new(
                finalDir.X * AIMBOT_SPEED,
                yVel,
                finalDir.Z * AIMBOT_SPEED
            )
            
            -- ============================================================
            -- SWING BAT
            -- ============================================================
            local headDist = headPos and (headPos - myPos).Magnitude or dist
            if headDist <= HIT_DISTANCE then
                swingBat(targetChar)
            end
            
        else
            -- No target, stop moving
            if myHRP then
                myHRP.AssemblyLinearVelocity = Vector3.zero
            end
        end
    end)
end

local function stopAimbot()
    aimbotActive = false
    if aimbotConnection then
        aimbotConnection:Disconnect()
        aimbotConnection = nil
    end
    
    local char = LP.Character
    if char then
        local myHRP = char:FindFirstChild("HumanoidRootPart")
        if myHRP then
            myHRP.AssemblyLinearVelocity = Vector3.zero
        end
    end
end

-- Toggle function
local function toggleAimbot()
    if aimbotActive then
        stopAimbot()
        animateToggle(false)
        print("[Blackskill Aimbot] OFF")
    else
        startAimbot()
        animateToggle(true)
        print("[Blackskill Aimbot] ON")
    end
end

-- Connect toggle button
ToggleButton.MouseButton1Click:Connect(toggleAimbot)

-- Keybind (E key from log)
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.E then
        toggleAimbot()
    end
end)

-- Optional: Right Control to hide/show GUI
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.RightControl then
        MainFrame.Visible = not MainFrame.Visible
    end
end)

print("[Logged by @darkhub] Fear Duel Bat Aimbot Loaded")
print("Press E to toggle | Right Control to hide GUI")
