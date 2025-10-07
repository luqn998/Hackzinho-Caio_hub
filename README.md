-- Caio_Hub V2.1 - GUI completo sem ícone
-- LocalScript dentro de StarterGui ou PlayerGui

-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")
local StarterGui = game:GetService("StarterGui")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local hrp = character:WaitForChild("HumanoidRootPart")
local humanoid = character:WaitForChild("Humanoid")

player.CharacterAdded:Connect(function(char)
	character = char
	hrp = char:WaitForChild("HumanoidRootPart")
	humanoid = char:WaitForChild("Humanoid")
end)

-- Configurações
local flySpeed = 24
local floatSpeed = 20
local autoSpeed = 24
local originalGravity = workspace.Gravity
local originalFogEnd = Lighting.FogEnd

local platform
local originalTransparencies = {}
local markedPosition = nil

local state = {
	floor = false,
	fly = false,
	float = false,
	auto = false,
	fps = false
}

-- Anti-Kick / Anti-Rejoin básico
pcall(function()
	local mt = getrawmetatable(game)
	if mt then
		local oldNamecall = mt.__namecall
		setreadonly(mt,false)
		mt.__namecall = newcclosure(function(self,...)
			local method = getnamecallmethod()
			if method == "Kick" then return end
			return oldNamecall(self,...)
		end)
		setreadonly(mt,true)
	end
end)

Players.PlayerRemoving:Connect(function(plr)
	if plr == player then
		warn("PlayerRemoving detected (anti-rejoin)")
	end
end)

-- Funções auxiliares
local function clamp(v,a,b) return math.max(a,math.min(b,v)) end
local function applyXray()
	for _, obj in ipairs(Workspace:GetDescendants()) do
		if obj:IsA("BasePart") then
			if originalTransparencies[obj]==nil then
				originalTransparencies[obj]=obj.Transparency
			end
			obj.Transparency = clamp(obj.Transparency+0.7,0,1)
		end
	end
	Lighting.FogEnd=999999
end
local function removeXray()
	for obj,trans in pairs(originalTransparencies) do
		if obj and obj.Parent then
			pcall(function() obj.Transparency = trans end)
		end
	end
	originalTransparencies = {}
	Lighting.FogEnd = originalFogEnd
end

-- Cria ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "CaioHubGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

-- GUI Principal
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0,320,0,460)
mainFrame.Position = UDim2.new(0.05,0,0.2,0)
mainFrame.BackgroundColor3 = Color3.fromRGB(18,18,20)
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui
mainFrame.Active = true
mainFrame.Draggable = true

local mainCorner = Instance.new("UICorner", mainFrame)
mainCorner.CornerRadius = UDim.new(0,18)

local mainStroke = Instance.new("UIStroke", mainFrame)
mainStroke.Thickness = 3
mainStroke.Color = Color3.fromRGB(0,128,255)

-- Título
local title = Instance.new("TextLabel", mainFrame)
title.Size = UDim2.new(1,-20,0,44)
title.Position = UDim2.new(0,10,0,8)
title.BackgroundTransparency = 1
title.Text = "Caio_Hub (V2.1)"
title.TextColor3 = Color3.fromRGB(160,210,255)
title.Font = Enum.Font.Arcade
title.TextSize = 22
title.TextXAlignment = Enum.TextXAlignment.Left

-- Função criar botões
local function createButton(text,y)
	local btn = Instance.new("TextButton", mainFrame)
	btn.Size = UDim2.new(0,280,0,44)
	btn.Position = UDim2.new(0,20,0,y)
	btn.BackgroundColor3 = Color3.fromRGB(40,40,44)
	btn.BorderSizePixel = 0
	btn.Font = Enum.Font.Arcade
	btn.TextSize = 18
	btn.TextColor3 = Color3.fromRGB(230,230,230)
	btn.Text = text.." : OFF"
	local uc = Instance.new("UICorner", btn)
	uc.CornerRadius = UDim.new(0,10)
	local stroke = Instance.new("UIStroke", btn)
	stroke.Thickness = 1
	stroke.Color = Color3.fromRGB(80,140,255)
	stroke.Transparency = 0.4
	return btn
end

local floorBtn = createButton("3rd Floor",64)
local flyBtn = createButton("Fly to Base",120)
local floatBtn = createButton("Float UP",176)
local markBtn = createButton("Mark Position",232)
local autoBtn = createButton("Auto Steal",288)
local fpsBtn = createButton("FPS Boost",344)

-- 3rd Floor
floorBtn.MouseButton1Click:Connect(function()
	state.floor = not state.floor
	if state.floor then
		floorBtn.Text="3rd Floor : ON"
		floorBtn.BackgroundColor3=Color3.fromRGB(0,255,128)
		applyXray()
		if not platform then
			platform = Instance.new("Part")
			platform.Size=Vector3.new(20,1,20)
			platform.Anchored=true
			platform.Material=Enum.Material.Neon
			platform.Color=Color3.fromRGB(80,80,80)
			platform.Transparency=0.3
			platform.Name="FloorPlatform"
			platform.Parent=Workspace
		end
		RunService.RenderStepped:Connect(function()
			if platform and hrp and state.floor then
				platform.CFrame = hrp.CFrame*CFrame.new(0,-4,0)
			end
		end)
	else
		floorBtn.Text="3rd Floor : OFF"
		floorBtn.BackgroundColor3=Color3.fromRGB(40,40,44)
		if platform then platform:Destroy() platform=nil end
		removeXray()
	end
end)

-- Fly to Base
flyBtn.MouseButton1Click:Connect(function()
	state.fly = not state.fly
	if state.fly then
		flyBtn.Text="Fly to Base : ON"
		flyBtn.BackgroundColor3=Color3.fromRGB(0,255,128)
		workspace.Gravity=0
		RunService:BindToRenderStep("FlyFree",Enum.RenderPriority.Character.Value+1,function()
			if hrp and state.fly then
				local cam = workspace.CurrentCamera
				local dir = cam.CFrame.LookVector
				hrp.Velocity = dir*flySpeed
			end
		end)
	else
		flyBtn.Text="Fly to Base : OFF"
		flyBtn.BackgroundColor3=Color3.fromRGB(40,40,44)
		RunService:UnbindFromRenderStep("FlyFree")
		workspace.Gravity=originalGravity
		if hrp then hrp.Velocity=Vector3.zero end
	end
end)

-- Float UP
floatBtn.MouseButton1Click:Connect(function()
	state.float = not state.float
	if state.float then
		floatBtn.Text="Float UP : ON"
		floatBtn.BackgroundColor3=Color3.fromRGB(0,255,128)
		RunService:BindToRenderStep("FloatUP",Enum.RenderPriority.Character.Value+1,function()
			if hrp and state.float then
				hrp.Velocity = Vector3.new(0,floatSpeed,0)
			end
		end)
	else
		floatBtn.Text="Float UP : OFF"
		floatBtn.BackgroundColor3=Color3.fromRGB(40,40,44)
		RunService:UnbindFromRenderStep("FloatUP")
	end
end)

-- Mark Position
markBtn.MouseButton1Click:Connect(function()
	markedPosition = hrp.Position
	markBtn.Visible = false
end)

-- Auto Steal
autoBtn.MouseButton1Click:Connect(function()
	if markedPosition then
		state.auto = true
		autoBtn.Text="Auto Steal : ON"
		autoBtn.BackgroundColor3=Color3.fromRGB(0,255,128)
		spawn(function()
			while state.auto and markedPosition do
				local dir = (markedPosition-hrp.Position)
				local distance = dir.Magnitude
				local move = dir.Unit * autoSpeed
				hrp.Velocity = Vector3.new(move.X,0,move.Z)
				if distance<2 then
					workspace.Gravity=originalGravity
					hrp.Velocity=Vector3.zero
					state.auto=false
					autoBtn.Text="Auto Steal : OFF"
					autoBtn.BackgroundColor3=Color3.fromRGB(40,40,44)
					break
				end
				wait()
			end
		end)
	end
end)

-- FPS Boost
fpsBtn.MouseButton1Click:Connect(function()
	state.fps = not state.fps
	if state.fps then
		fpsBtn.Text="FPS Boost : ON"
		fpsBtn.BackgroundColor3=Color3.fromRGB(0,255,128)
		-- Remove acessórios e reduz gráficos
		for _,acc in pairs(character:GetChildren()) do
			if acc:IsA("Accessory") then acc:Destroy() end
		end
		Lighting.GlobalShadows=false
		Lighting.FogEnd=100000
		Lighting.Brightness=1
	else
		fpsBtn.Text="FPS Boost : OFF"
		fpsBtn.BackgroundColor3=Color3.fromRGB(40,40,44)
		Lighting.GlobalShadows=true
		Lighting.FogEnd=originalFogEnd
		Lighting.Brightness=2
	end
end)

-- Anti bug: não cair abaixo do mapa
RunService.RenderStepped:Connect(function()
	pcall(function()
		if hrp and hrp.Position.Y < -200 then
			hrp.CFrame = CFrame.new(0,50,0)
		end
	end)
end)
