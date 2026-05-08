--[[ 
    VazaxLUA - UNIVERSAL PUNCH FLING (FE R6/R15)
    Creator: VAZAX
    Credits: Assisted by AI Gemini
    
    WARNING: Script ini menggunakan manipulasi physics FE. 
    Gunakan dengan bijak untuk menghindari kick dari server.
]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

-- GUI Container
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "VazaxUniversal_FE"
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
ScreenGui.ResetOnSpawn = false

-- States
local punchActive = false
local spinActive = false
local hitboxActive = false
local hbSize = 10

-- 1. FLOATING "V" BUTTON
local MiniBtn = Instance.new("Frame")
MiniBtn.Size = UDim2.new(0, 50, 0, 50)
MiniBtn.Position = UDim2.new(0, 10, 0, 10)
MiniBtn.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
MiniBtn.Parent = ScreenGui
Instance.new("UICorner", MiniBtn).CornerRadius = UDim.new(1, 0)

local VText = Instance.new("TextButton")
VText.Size = UDim2.new(1, 0, 1, 0)
VText.Text = "V"
VText.TextColor3 = Color3.fromRGB(160, 32, 240)
VText.TextSize = 30
VText.BackgroundTransparency = 1
VText.Font = Enum.Font.GothamBold
VText.Parent = MiniBtn

-- Draggable Logic
local dragging, dragInput, dragStart, startPos
MiniBtn.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true; dragStart = input.Position; startPos = MiniBtn.Position
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        MiniBtn.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
UserInputService.InputEnded:Connect(function(input) dragging = false end)

-- 2. MAIN GUI WINDOW (PURPLE & BLACK)
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 220, 0, 320)
MainFrame.Position = UDim2.new(0.5, -110, 0.5, -160)
MainFrame.BackgroundColor3 = Color3.fromRGB(15, 0, 25)
MainFrame.BorderSizePixel = 0
MainFrame.Visible = false
MainFrame.Parent = ScreenGui
Instance.new("UICorner", MainFrame)

-- Close Button
local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 30, 0, 30)
CloseBtn.Position = UDim2.new(1, -35, 0, 5)
CloseBtn.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
CloseBtn.Text = "X"
CloseBtn.TextColor3 = Color3.fromRGB(0, 0, 0)
CloseBtn.Parent = MainFrame
CloseBtn.MouseButton1Click:Connect(function() MainFrame.Visible = false end)

-- Scrolling Frame
local Scroll = Instance.new("ScrollingFrame")
Scroll.Size = UDim2.new(1, -10, 1, -60)
Scroll.Position = UDim2.new(0, 5, 0, 40)
Scroll.BackgroundTransparency = 1
Scroll.CanvasSize = UDim2.new(0, 0, 2, 0)
Scroll.ScrollBarThickness = 3
Scroll.Parent = MainFrame
local Layout = Instance.new("UIListLayout", Scroll)
Layout.Padding = UDim.new(0, 8); Layout.HorizontalAlignment = "Center"

VText.MouseButton1Click:Connect(function() MainFrame.Visible = not MainFrame.Visible end)

-- Toggle Generator
local function createToggle(name, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 180, 0, 40)
    btn.BackgroundColor3 = Color3.fromRGB(30, 0, 50)
    btn.Text = name .. ": Disable"
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Font = "Gotham"
    btn.Parent = Scroll
    Instance.new("UICorner", btn)
    
    local active = false
    btn.MouseButton1Click:Connect(function()
        active = not active
        btn.Text = name .. (active and ": Enable" or ": Disable")
        btn.BackgroundColor3 = active and Color3.fromRGB(160, 32, 240) or Color3.fromRGB(30, 0, 50)
        callback(active)
    end)
end

-- FEATURES SETUP
createToggle("Punch Damage", function(v) punchActive = v end)
createToggle("Spin Fling", function(v) spinActive = v end)

local hbInput = Instance.new("TextBox")
hbInput.Size = UDim2.new(0, 180, 0, 35)
hbInput.PlaceholderText = "Hitbox Size (1-50)"
hbInput.Text = "10"
hbInput.BackgroundColor3 = Color3.fromRGB(40,40,40)
hbInput.TextColor3 = Color3.fromRGB(255,255,255)
hbInput.Parent = Scroll

createToggle("Hitbox Expander", function(v) 
    hitboxActive = v 
    hbSize = math.clamp(tonumber(hbInput.Text) or 10, 1, 50)
end)

-- CREDIT (English as requested)
local Credits = Instance.new("TextLabel")
Credits.Size = UDim2.new(0, 180, 0, 60)
Credits.Text = "Creator: VAZAX\nAssisted by AI Gemini"
Credits.TextColor3 = Color3.fromRGB(160, 32, 240)
Credits.BackgroundTransparency = 1
Credits.TextSize = 12
Credits.Parent = Scroll

-- 3. PUNCH BUTTON (HAND ICON)
local PunchBtn = Instance.new("ImageButton")
PunchBtn.Size = UDim2.new(0, 70, 0, 70)
PunchBtn.Position = UDim2.new(1, -150, 1, -150)
PunchBtn.Image = "rbxassetid://12296135476"
PunchBtn.BackgroundColor3 = Color3.fromRGB(0,0,0)
PunchBtn.BackgroundTransparency = 0.5
PunchBtn.Parent = ScreenGui
Instance.new("UICorner", PunchBtn).CornerRadius = UDim.new(1,0)

-- PHYSICS LOGIC (FE COMPATIBLE)
RunService.Heartbeat:Connect(function()
    local char = LocalPlayer.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    -- Spin Fling Logic (Invisible Physics)
    if spinActive then
        local velocity = hrp.Velocity
        hrp.Velocity = velocity.Unit * 1000 + Vector3.new(0, 1000, 0) -- High Velocity
        hrp.RotVelocity = Vector3.new(0, 10000, 0) -- Ultra Spin
    end

    -- Hitbox Logic
    if hitboxActive then
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                local targetHRP = p.Character.HumanoidRootPart
                targetHRP.Size = Vector3.new(hbSize, hbSize, hbSize)
                targetHRP.Transparency = 0.8
                targetHRP.CanCollide = false
            end
        end
    end
end)

-- Punch Fling Action
PunchBtn.MouseButton1Click:Connect(function()
    if not punchActive then return end
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    
    for _, v in pairs(Players:GetPlayers()) do
        if v ~= LocalPlayer and v.Character and v.Character:FindFirstChild("HumanoidRootPart") then
            local dist = (char.HumanoidRootPart.Position - v.Character.HumanoidRootPart.Position).Magnitude
            if dist < (hbSize + 5) then
                -- FE Fling Method
                local target = v.Character.HumanoidRootPart
                for i = 1, 10 do
                    target.Velocity = Vector3.new(0, 5000, 0)
                    task.wait()
                end
            end
        end
    end
end)

