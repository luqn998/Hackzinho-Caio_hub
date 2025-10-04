-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local hrp = character:WaitForChild("HumanoidRootPart")
local humanoid = character:WaitForChild("Humanoid")

-- Função para setup do Character
local function setupCharacter(char)
    character = char
    hrp = character:WaitForChild("HumanoidRootPart")
    humanoid = char:WaitForChild("Humanoid")
end

setupCharacter(character)
player.CharacterAdded:Connect(setupCharacter)

-- ======== GUI Full Black ========
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = player:WaitForChild("PlayerGui")
screenGui.Name = "CaioHubGui"
screenGui.ResetOnSpawn = false

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0,200,0,100)
mainFrame.Position = UDim2.new(0.5,-100,0.5,-50)
mainFrame.BackgroundColor3 = Color3.fromRGB(20,20,20) -- full black
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui
mainFrame.Active = true
mainFrame.Draggable = true
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0,15)
corner.Parent = mainFrame

-- Title
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1,0,0,30)
title.Position = UDim2.new(0,0,0,0)
title.BackgroundTransparency = 1
title.Text = "Caio_hub (1.0)"
title.Font = Enum.Font.GothamBold
title.TextSize = 18
title.TextColor3 = Color3.fromRGB(255,255,255)
title.Parent = mainFrame

-- Função para botão toggle estilizado
local function createToggleButton(name, position)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0,160,0,35)
    button.Position = position
    button.Text = name.." : OFF"
    button.Font = Enum.Font.GothamBold
    button.TextSize = 16
    button.TextColor3 = Color3.fromRGB(255,255,255)
    button.BackgroundColor3 = Color3.fromRGB(50,50,50)
    button.BorderSizePixel = 0
    button.Parent = mainFrame

    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0,10)
    btnCorner.Parent = button

    -- Hover
    button.MouseEnter:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(70,70,70)}):Play()
    end)
    button.MouseLeave:Connect(function()
        TweenService:Create(button, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(50,50,50)}):Play()
    end)

    local state = false
    button.MouseButton1Click:Connect(function()
        state = not state
        if state then
            button.Text = name.." : ON"
            TweenService:Create(button, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(0,255,128)}):Play()
        else
            button.Text = name.." : OFF"
            TweenService:Create(button, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(50,50,50)}):Play()
        end
    end)

    return button
end

-- ===== BOTÃO 3D Floor =====
local floorButton = createToggleButton("3D Floor", UDim2.new(0,20,0,50))
local floorActive = false
local platform

floorButton.MouseButton1Click:Connect(function()
    floorActive = not floorActive
    if floorActive then
        if not platform and hrp then
            platform = Instance.new("Part")
            platform.Size = Vector3.new(10,1,10)
            platform.Anchored = true
            platform.Position = hrp.Position - Vector3.new(0,3,0)
            platform.Color = Color3.fromRGB(50,50,50)
            platform.Material = Enum.Material.Neon
            platform.Parent = workspace
        end
    else
        if platform then
            platform:Destroy()
            platform = nil
        end
    end
end)

-- Atualiza posição da plataforma rapidamente
RunService.RenderStepped:Connect(function()
    if floorActive and platform and hrp then
        platform.Position = platform.Position:Lerp(hrp.Position - Vector3.new(0,3,0),0.2)
    end
end)
