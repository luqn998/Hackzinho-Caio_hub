-- // SERVIÇOS
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local hrp = character:WaitForChild("HumanoidRootPart")
local humanoid = character:WaitForChild("Humanoid")

player.CharacterAdded:Connect(function(char)
	character = char
	hrp = char:WaitForChild("HumanoidRootPart")
	humanoid = char:WaitForChild("Humanoid")
end)

-- // GUI
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "CaioHub"
gui.ResetOnSpawn = false

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 250, 0, 150)
frame.Position = UDim2.new(0.05, 0, 0.35, 0)
frame.BackgroundColor3 = Color3.fromRGB(30,30,30)
frame.Active = true
frame.Draggable = true

-- Borda amarela
local border = Instance.new("UIStroke", frame)
border.Thickness = 2
border.Color = Color3.fromRGB(255,255,0)
border.ApplyStrokeMode = Enum.ApplyStrokeMode.Border

local uiCorner = Instance.new("UICorner", frame)
uiCorner.CornerRadius = UDim.new(0,15)

local title = Instance.new("TextLabel", frame)
title.Text = "Caio_Hub (V1)"
title.Size = UDim2.new(1,0,0,35)
title.BackgroundTransparency = 1
title.TextColor3 = Color3.fromRGB(255,255,0)
title.Font = Enum.Font.GothamBold
title.TextSize = 20
title.TextStrokeTransparency = 0.7

-- // BOTÃO CREATOR
local function createButton(name, y)
	local btn = Instance.new("TextButton", frame)
	btn.Size = UDim2.new(0, 200, 0, 40)
	btn.Position = UDim2.new(0, 25, 0, y)
	btn.BackgroundColor3 = Color3.fromRGB(50,50,50)
	btn.Text = name .. " : OFF"
	btn.TextColor3 = Color3.fromRGB(255,255,255)
	btn.Font = Enum.Font.GothamBold
	btn.TextSize = 16
	local corner = Instance.new("UICorner", btn)
	corner.CornerRadius = UDim.new(0,10)
	return btn
end

-- // BOTÕES
local floorBtn = createButton("3D Floor", 50)
local flyBtn = createButton("Fly V2", 110)

-----------------------------------------------
-- BOTÃO 1: 3D FLOOR (PLATAFORMA + RAIO-X)
-----------------------------------------------
local floorActive = false
local platform
local originalTransparencies = {}
local originalFogEnd = Lighting.FogEnd

local function applyXray()
	for _, obj in ipairs(workspace:GetDescendants()) do
		if obj:IsA("BasePart") then
			originalTransparencies[obj] = obj.Transparency
			obj.Transparency = math.clamp(obj.Transparency + 0.7,0,1)
		end
	end
	Lighting.FogEnd = 999999
end

local function removeXray()
	for obj, trans in pairs(originalTransparencies) do
		if obj and obj.Parent then
			obj.Transparency = trans
		end
	end
	Lighting.FogEnd = originalFogEnd
	originalTransparencies = {}
end

floorBtn.MouseButton1Click:Connect(function()
	floorActive = not floorActive
	if floorActive then
		floorBtn.Text = "3D Floor : ON"
		floorBtn.BackgroundColor3 = Color3.fromRGB(0,255,128)

		applyXray()

		if not platform then
			platform = Instance.new("Part")
			platform.Size = Vector3.new(20,1,20)
			platform.Anchored = true
			platform.Material = Enum.Material.Neon
			platform.Color = Color3.fromRGB(80,80,80)
			platform.Transparency = 0.3
			platform.Name = "FloorPlatform"
			platform.Parent = workspace
		end

		RunService.RenderStepped:Connect(function()
			if platform and hrp and floorActive then
				platform.CFrame = hrp.CFrame * CFrame.new(0,-4,0)
			end
		end)
	else
		floorBtn.Text = "3D Floor : OFF"
		floorBtn.BackgroundColor3 = Color3.fromRGB(50,50,50)
		if platform then
			platform:Destroy()
			platform = nil
		end
		removeXray()
	end
end)

-----------------------------------------------
-- BOTÃO 2: FLY V2 (VAI ONDE VOCÊ OLHA)
-----------------------------------------------
local flying = false
local flySpeed = 25

flyBtn.MouseButton1Click:Connect(function()
	flying = not flying
	if flying then
		flyBtn.Text = "Fly V2 : ON"
		flyBtn.BackgroundColor3 = Color3.fromRGB(0,255,128)

		RunService:BindToRenderStep("FlyV2", Enum.RenderPriority.Character.Value + 1, function()
			if hrp then
				local cam = workspace.CurrentCamera
				local dir = cam.CFrame.LookVector
				hrp.Velocity = dir * flySpeed
			end
		end)
	else
		flyBtn.Text = "Fly V2 : OFF"
		flyBtn.BackgroundColor3 = Color3.fromRGB(50,50,50)
		RunService:UnbindFromRenderStep("FlyV2")
		if hrp then hrp.Velocity = Vector3.zero end
	end
end)

-----------------------------------------------
-- ANTI-BUG / ANTI-KICK / ANTI-BACK
-----------------------------------------------
RunService.RenderStepped:Connect(function()
	pcall(function()
		if hrp and hrp.Position.Y < -50 then
			hrp.CFrame = CFrame.new(Vector3.new(0,10,0))
		end
	end)
end)
