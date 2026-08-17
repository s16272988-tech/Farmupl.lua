local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local player = Players.LocalPlayer
local backpack = player:WaitForChild("Backpack")

local running = false
local boxesCollected = 0
local myKey = "SCRIPTSROBLOX2026youjil"

local farmPoint = Vector3.new(-24.719234, 17.219492, -70.369278)
local sellPoint = Vector3.new(3.648493, 15.170154, -61.144489)

local usaFarmPoints = {
    Vector3.new(6880.591309, 17.219496, 115.590675),
    Vector3.new(4316.914062, 18.251337, 100.704872),
    Vector3.new(2784.545654, 18.152271, 119.353958), -- 3
    Vector3.new(1553.964966, 18.248333, 114.918900), -- 4
    Vector3.new(169.624664, 18.210472, 91.254753),    -- 5
    Vector3.new(-19.489838, 17.240204, 75.024414),    -- 6
    Vector3.new(-13.665200, 17.291885, -54.005737),   -- 7
    Vector3.new(-1.449235, 17.291885, -63.920403)     -- 8
}

local function teleportTo(targetPos)
    local rootPart = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if rootPart then rootPart.CFrame = CFrame.new(targetPos) end
end

local function tweenToUsa()
    local rootPart = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if not rootPart then return end

    for index, targetPos in ipairs(usaFarmPoints) do
        local distance = (rootPart.Position - targetPos).Magnitude
        local speed = 230
        
        if index == 3 or index == 4 then
            speed = 200
        elseif index >= 5 and index <= 7 then
            speed = 100
        end

        local timeToTravel = distance / speed
        local info = TweenInfo.new(timeToTravel, Enum.EasingStyle.Linear)
        local tween = TweenService:Create(rootPart, info, {CFrame = CFrame.new(targetPos)})
        tween:Play()
        tween.Completed:Wait()
    end
end

local function fireAllNearby()
    local rootPart = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if not rootPart then return end
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("ProximityPrompt") and (obj.Parent:IsA("BasePart") and (obj.Parent.Position - rootPart.Position).Magnitude <= 50) then
            fireproximityprompt(obj)
        end
    end
end

-- Создание GUI
local screenGui
local keyFrame, mainFrame, countLabel

pcall(function()
    screenGui = Instance.new("ScreenGui")
    screenGui.Name = "FarmControlHub"
    
    if syn and syn.protect_gui then
        syn.protect_gui(screenGui)
        screenGui.Parent = CoreGui
    else
        pcall(function() screenGui.Parent = CoreGui end)
        if not screenGui.Parent then
            screenGui.Parent = player:WaitForChild("PlayerGui")
        end
    end

    -- 1. ОКНО КЛЮЧА
    keyFrame = Instance.new("Frame", screenGui)
    keyFrame.Size = UDim2.new(0, 280, 0, 160)
    keyFrame.Position = UDim2.new(0.5, -140, 0.5, -80)
    keyFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    keyFrame.Active = true
    keyFrame.Draggable = true
    Instance.new("UICorner", keyFrame).CornerRadius = UDim.new(0, 12)

    local keyTitle = Instance.new("TextLabel", keyFrame)
    keyTitle.Size = UDim2.new(1, 0, 0, 40)
    keyTitle.Text = "ENTER KEY"
    keyTitle.TextColor3 = Color3.new(1, 1, 1)
    keyTitle.Font = Enum.Font.GothamBold
    keyTitle.TextSize = 16
    keyTitle.BackgroundTransparency = 1

    local keyBox = Instance.new("TextBox", keyFrame)
    keyBox.Size = UDim2.new(0.88, 0, 0, 35)
    keyBox.Position = UDim2.new(0.06, 0, 0.35, 0)
    keyBox.PlaceholderText = "Введите ключ..."
    keyBox.Text = ""
    keyBox.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    keyBox.TextColor3 = Color3.new(1, 1, 1)
    keyBox.Font = Enum.Font.Gotham
    keyBox.TextSize = 14
    Instance.new("UICorner", keyBox).CornerRadius = UDim.new(0, 8)

    local checkBtn = Instance.new("TextButton", keyFrame)
    checkBtn.Size = UDim2.new(0.88, 0, 0, 35)
    checkBtn.Position = UDim2.new(0.06, 0, 0.65, 0)
    checkBtn.BackgroundColor3 = Color3.fromRGB(52, 152, 219)
    checkBtn.Text = "SUBMIT"
    checkBtn.TextColor3 = Color3.new(1, 1, 1)
    checkBtn.Font = Enum.Font.GothamBold
    checkBtn.TextSize = 14
    Instance.new("UICorner", checkBtn).CornerRadius = UDim.new(0, 8)

    -- 2. ОСНОВНОЙ ХАБ (увеличена высота под обновленные кнопки)
    mainFrame = Instance.new("Frame", screenGui)
    mainFrame.Size = UDim2.new(0, 260, 0, 165)
    mainFrame.Position = UDim2.new(0, 50, 0, 150)
    mainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    mainFrame.Active = true
    mainFrame.Draggable = true
    mainFrame.Visible = false
    Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 12)

    local title = Instance.new("TextLabel", mainFrame)
    title.Size = UDim2.new(1, 0, 0, 30)
    title.Text = "AUTO FARM HUB"
    title.TextColor3 = Color3.new(1, 1, 1)
    title.Font = Enum.Font.GothamBold
    title.TextSize = 16
    title.BackgroundTransparency = 1

    countLabel = Instance.new("TextLabel", mainFrame)
    countLabel.Size = UDim2.new(1, 0, 0, 20)
    countLabel.Position = UDim2.new(0, 0, 0, 28)
    countLabel.Text = "Boxes: 0"
    countLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    countLabel.Font = Enum.Font.Gotham
    countLabel.TextSize = 14
    countLabel.BackgroundTransparency = 1

    local function createButton(text, color, pos, size)
        local btn = Instance.new("TextButton", mainFrame)
        btn.Size = size or UDim2.new(0.42, 0, 0, 32)
        btn.Position = pos
        btn.BackgroundColor3 = color
        btn.Text = text
        btn.TextColor3 = Color3.new(1, 1, 1)
        btn.Font = Enum.Font.GothamBold
        btn.TextSize = 13
        Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)
        return btn
    end

    -- START и STOP подняты выше
    local startBtn = createButton("START", Color3.fromRGB(46, 204, 113), UDim2.new(0.06, 0, 0.35, 0))
    local stopBtn = createButton("STOP", Color3.fromRGB(231, 76, 60), UDim2.new(0.52, 0, 0.35, 0))
    
    -- Кнопка TP TO USA FARM сделана больше и шире
    local usaBtn = createButton("TP TO USA FARM", Color3.fromRGB(155, 89, 182), UDim2.new(0.06, 0, 0.58, 0), UDim2.new(0.88, 0, 0, 38))

    checkBtn.MouseButton1Click:Connect(function()
        if keyBox.Text == myKey then
            keyFrame.Visible = false
            mainFrame.Visible = true
        else
            keyBox.Text = ""
            keyBox.PlaceholderText = "НЕВЕРНЫЙ КЛЮЧ!"
        end
    end)

    startBtn.MouseButton1Click:Connect(function() running = true end)
    stopBtn.MouseButton1Click:Connect(function() running = false end)
    usaBtn.MouseButton1Click:Connect(function()
        task.spawn(tweenToUsa)
    end)
end)

-- Цикл фарма
task.spawn(function()
    while true do
        if running then
            teleportTo(farmPoint)
            fireAllNearby()
            task.wait(0.9)
            teleportTo(sellPoint)
            fireAllNearby()
            task.wait(0.9)
        end
        if countLabel then
            countLabel.Text = "Boxes: " .. tostring(boxesCollected)
        end
        task.wait(0.1)
    end
end)

backpack.ChildAdded:Connect(function(child)
    if (child.Name:lower():find("box") or child:IsA("Tool")) then
        boxesCollected += 1
    end
end)
# Farmupl.lua
