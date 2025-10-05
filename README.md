-- // SERVIÇOS
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local hrp = character:WaitForChild("HumanoidRootPart")

player.CharacterAdded:Connect(function(char)
	character = char
	hrp = char:WaitForChild("HumanoidRootPart")
end)

-- // GUI
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "CaioHub"
gui.ResetOnSpawn = false

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 200, 0, 150)
frame.Position = UDim2.new(0.05, 0, 0.4, 0)
frame.BackgroundColor3 = Color3.fromRGB(25,25,25)
frame.Active = true
frame.Draggable = true

local uiCorner = Instance.new("UICorner", frame)
uiCorner.CornerRadius = UDim.new(0, 12)

local title = Instance.new("TextLabel", frame)
title.Text = "Caio Hub"
title.Size = UDim2.new(1, 0, 0, 30)
title.BackgroundTransparency = 1
title.TextColor3 = Color3.fromRGB(255,255,255)
title.Font = Enum.Font.GothamBold
title.TextSize = 18

-- // BOTÃO CREATOR
local function createButton(name, y)
	local btn = Instance.new("TextButton", frame)
	btn.Size = UDim2.new(0, 160, 0, 35)
	btn.Position = UDim2.new(0, 20, 0, y)
	btn.BackgroundColor3 = Color3.fromRGB(50,50,50)
	btn.Text = name .. " : OFF"
	btn.TextColor3 = Color3.fromRGB(255,255,255)
	btn.Font = Enum.Font.GothamBold
	btn.TextSize = 14
	local corner = Instance.new("UICorner", btn)
	corner.CornerRadius = UDim.new(0,8)
	return btn
end

-- // BOTÕES
local floorBtn = createButton("3D Floor", 40)
local flyBtn = createButton("Fly", 85)

-----------------------------------------------
-- BOTÃO 1: 3D FLOOR (PLATAFORMA + RAIO-X TEMPORÁRIO)
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
-- BOTÃO 2: FLY (DIREÇÃO DA CÂMERA)
-----------------------------------------------
local flying = false
local flySpeed = 25  -- VELOCIDADE ATUALIZADA

flyBtn.MouseButton1Click:Connect(function()
	flying = not flying
	if flying then
		flyBtn.Text = "Fly : ON"
		flyBtn.BackgroundColor3 = Color3.fromRGB(0,255,128)

		RunService:BindToRenderStep("FlyCam", Enum.RenderPriority.Camera.Value + 1, function()
			if hrp and workspace.CurrentCamera then
				local cam = workspace.CurrentCamera
				local dir = cam.CFrame.LookVector
				if dir.Magnitude > 0 then
					hrp.Velocity = dir.Unit * flySpeed
				end
			end
		end)
	else
		flyBtn.Text = "Fly : OFF"
		flyBtn.BackgroundColor3 = Color3.fromRGB(50,50,50)
		RunService:UnbindFromRenderStep("FlyCam")
		if hrp then hrp.Velocity = Vector3.zero end
	end
end)
