-- // SERVIÇOS
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
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

-------------------------------------------------
-- // VARIÁVEIS
-------------------------------------------------
local floorActive = false
local flying = false
local floatActive = false
local autoActive = false
local flySpeed = 24
local floatSpeed = 20
local autoSpeed = 24
local originalGravity = workspace.Gravity
local platform
local originalTransparencies = {}
local originalFogEnd = Lighting.FogEnd
local markedPosition = nil

-------------------------------------------------
-- // GUI PRINCIPAL
-------------------------------------------------
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "CaioHub"
gui.ResetOnSpawn = false

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 260, 0, 340)
frame.Position = UDim2.new(0.05, 0, 0.35, 0)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.Active = true
frame.Draggable = true

local stroke = Instance.new("UIStroke", frame)
stroke.Thickness = 3
stroke.Color = Color3.fromRGB(0, 128, 255)

local corner = Instance.new("UICorner", frame)
corner.CornerRadius = UDim.new(0, 14)

local title = Instance.new("TextLabel", frame)
title.Text = "Caio_Hub (V3)"
title.Size = UDim2.new(1, 0, 0, 35)
title.BackgroundTransparency = 1
title.TextColor3 = Color3.fromRGB(100, 180, 255)
title.Font = Enum.Font.Arcade
title.TextSize = 22

-------------------------------------------------
-- // FUNÇÃO PARA CRIAR BOTÕES
-------------------------------------------------
local function createButton(name, posY)
	local btn = Instance.new("TextButton", frame)
	btn.Size = UDim2.new(0, 210, 0, 40)
	btn.Position = UDim2.new(0, 25, 0, posY)
	btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
	btn.Text = name .. " : OFF"
	btn.TextColor3 = Color3.fromRGB(255, 255, 255)
	btn.Font = Enum.Font.Arcade
	btn.TextSize = 16
	local c = Instance.new("UICorner", btn)
	c.CornerRadius = UDim.new(0, 10)
	return btn
end

-- BOTÕES
local floorBtn = createButton("3rd Floor", 50)
local flyBtn = createButton("Fly to Base", 100)
local floatBtn = createButton("Float UP", 150)
local autoBtn = createButton("Auto Steal", 200)
local markBtn = createButton("Mark Position", 250)

-------------------------------------------------
-- // 3RD FLOOR
-------------------------------------------------
local function applyXray()
	for _, obj in ipairs(workspace:GetDescendants()) do
		if obj:IsA("BasePart") then
			originalTransparencies[obj] = obj.Transparency
			obj.Transparency = math.clamp(obj.Transparency + 0.7, 0, 1)
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
		floorBtn.Text = "3rd Floor : ON"
		floorBtn.BackgroundColor3 = Color3.fromRGB(0, 200, 255)
		applyXray()

		if not platform then
			platform = Instance.new("Part")
			platform.Size = Vector3.new(20, 1, 20)
			platform.Anchored = true
			platform.Material = Enum.Material.Neon
			platform.Color = Color3.fromRGB(80, 80, 80)
			platform.Transparency = 0.3
			platform.Name = "FloorPlatform"
			platform.Parent = workspace
		end

		RunService.RenderStepped:Connect(function()
			if platform and hrp and floorActive then
				platform.CFrame = hrp.CFrame * CFrame.new(0, -4, 0)
			end
		end)
	else
		floorBtn.Text = "3rd Floor : OFF"
		floorBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
		if platform then
			platform:Destroy()
			platform = nil
		end
		removeXray()
	end
end)

-------------------------------------------------
-- // FLY TO BASE
-------------------------------------------------
flyBtn.MouseButton1Click:Connect(function()
	flying = not flying
	if flying then
		flyBtn.Text = "Fly to Base : ON"
		flyBtn.BackgroundColor3 = Color3.fromRGB(0, 200, 255)
		workspace.Gravity = 0
		RunService:BindToRenderStep("FlyBase", Enum.RenderPriority.Character.Value + 1, function()
			if hrp and flying then
				local dir = workspace.CurrentCamera.CFrame.LookVector
				hrp.Velocity = dir * flySpeed
			end
		end)
	else
		flyBtn.Text = "Fly to Base : OFF"
		flyBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
		RunService:UnbindFromRenderStep("FlyBase")
		workspace.Gravity = originalGravity
		if hrp then hrp.Velocity = Vector3.zero end
	end
end)

-------------------------------------------------
-- // FLOAT UP
-------------------------------------------------
floatBtn.MouseButton1Click:Connect(function()
	floatActive = not floatActive
	if floatActive then
		floatBtn.Text = "Float UP : ON"
		floatBtn.BackgroundColor3 = Color3.fromRGB(0, 200, 255)

		RunService:BindToRenderStep("FloatUP", Enum.RenderPriority.Character.Value + 2, function()
			if hrp and floatActive then
				hrp.Velocity = Vector3.new(0, floatSpeed, 0)
			end
		end)
	else
		floatBtn.Text = "Float UP : OFF"
		floatBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
		RunService:UnbindFromRenderStep("FloatUP")
		if hrp then hrp.Velocity = Vector3.zero end
	end
end)

-------------------------------------------------
-- // MARK POSITION
-------------------------------------------------
markBtn.MouseButton1Click:Connect(function()
	markedPosition = hrp.Position
	markBtn.Visible = false
end)

-------------------------------------------------
-- // AUTO STEAL
-------------------------------------------------
autoBtn.MouseButton1Click:Connect(function()
	if not markedPosition then return end
	autoActive = not autoActive
	if autoActive then
		autoBtn.Text = "Auto Steal : ON"
		autoBtn.BackgroundColor3 = Color3.fromRGB(0, 200, 255)
		workspace.Gravity = 0

		RunService:BindToRenderStep("AutoSteal", Enum.RenderPriority.Character.Value + 3, function()
			if hrp and autoActive then
				local direction = (markedPosition - hrp.Position).Unit
				hrp.Velocity = direction * autoSpeed

				if (hrp.Position - markedPosition).Magnitude < 2 then
					workspace.Gravity = originalGravity
					autoActive = false
					autoBtn.Text = "Auto Steal : OFF"
					autoBtn.BackgroundColor3 = Color3.fromRGB(50,50,50)
					RunService:UnbindFromRenderStep("AutoSteal")
				end
			end
		end)
	else
		autoBtn.Text = "Auto Steal : OFF"
		autoBtn.BackgroundColor3 = Color3.fromRGB(50,50,50)
		RunService:UnbindFromRenderStep("AutoSteal")
		workspace.Gravity = originalGravity
		if hrp then hrp.Velocity = Vector3.zero end
	end
end)

-------------------------------------------------
-- // SEGURANÇA / ANTI-BUG
-------------------------------------------------
RunService.RenderStepped:Connect(function()
	pcall(function()
		if hrp and hrp.Position.Y < -50 then
			hrp.CFrame = CFrame.new(Vector3.new(0,10,0))
		end
	end)
end)
