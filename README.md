-- Caio_Hub V2.1 - GUI completo com ícone arrastável com texto "Caio"
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
local modifiedParts = {}
local function clamp(v,a,b) return math.max(a,math.min(b,v)) end

------------------------------------------------------------
-- Anti Kick / Anti AFK / Anti Rejoin
------------------------------------------------------------
pcall(function()
	if player.Kick then
		player.Kick = function(...) warn("[Caio_Hub] Kick bloqueado!") end
	end
end)

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

spawn(function()
	while task.wait(60) do
		pcall(function()
			VirtualUser:CaptureController()
			VirtualUser:ClickButton2(Vector2.new())
		end)
	end
end)

------------------------------------------------------------
-- Tela de Apresentação
------------------------------------------------------------
local introGui = Instance.new("ScreenGui")
introGui.Name = "CaioHubIntro"
introGui.ResetOnSpawn = false
introGui.Parent = player:WaitForChild("PlayerGui")

local introFrame = Instance.new("Frame")
introFrame.Size = UDim2.new(1,0,1,0)
introFrame.BackgroundColor3 = Color3.fromRGB(20,0,30)
introFrame.Parent = introGui

local introTitle = Instance.new("TextLabel")
introTitle.Size = UDim2.new(1,0,0,100)
introTitle.Position = UDim2.new(0,0,0.4,0)
introTitle.BackgroundTransparency = 1
introTitle.Text = "🌟 Caio Hub Apresenta 🌟"
introTitle.Font = Enum.Font.Arcade
introTitle.TextSize = 38
introTitle.TextColor3 = Color3.fromRGB(255,200,0)
introTitle.Parent = introFrame

local glow = Instance.new("UIStroke")
glow.Thickness = 2
glow.Color = Color3.fromRGB(255,255,0)
glow.Transparency = 0.2
glow.Parent = introTitle

-- Animação pulsante
spawn(function()
	while introGui.Parent do
		for i=0,1,0.03 do
			introTitle.TextTransparency = i
			glow.Transparency = 0.2 + i*0.6
			task.wait(0.03)
		end
		for i=1,0,-0.03 do
			introTitle.TextTransparency = i
			glow.Transparency = 0.2 + i*0.6
			task.wait(0.03)
		end
	end
end)

task.wait(4)
introGui:Destroy()

------------------------------------------------------------
-- GUI Principal
------------------------------------------------------------
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "CaioHubGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

-- Ícone arrastável com texto "Caio"
local icon = Instance.new("TextButton")
icon.Size = UDim2.new(0,100,0,40)
icon.Position = UDim2.new(0,10,0,50)
icon.BackgroundColor3 = Color3.fromRGB(50,0,100)
icon.Text = "Caio"
icon.TextColor3 = Color3.fromRGB(255,255,255)
icon.Font = Enum.Font.Arcade
icon.TextSize = 22
icon.Parent = screenGui

local iconCorner = Instance.new("UICorner", icon)
iconCorner.CornerRadius = UDim.new(0,10)

local iconStroke = Instance.new("UIStroke", icon)
iconStroke.Color = Color3.fromRGB(255,220,0)
iconStroke.Thickness = 2

-- Menu principal
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0,320,0,300)
mainFrame.Position = UDim2.new(0,120,0,50)
mainFrame.BackgroundColor3 = Color3.fromRGB(25,0,40)
mainFrame.Visible = true
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui

local mainCorner = Instance.new("UICorner", mainFrame)
mainCorner.CornerRadius = UDim.new(0,18)

local mainStroke = Instance.new("UIStroke", mainFrame)
mainStroke.Color = Color3.fromRGB(255,220,0)
mainStroke.Thickness = 4

local titleLabel = Instance.new("TextLabel", mainFrame)
titleLabel.Size = UDim2.new(1,-20,0,44)
titleLabel.Position = UDim2.new(0,10,0,8)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "Caio_Hub (V2.1)"
titleLabel.TextColor3 = Color3.fromRGB(255,255,150)
titleLabel.Font = Enum.Font.Arcade
titleLabel.TextSize = 22
titleLabel.TextXAlignment = Enum.TextXAlignment.Left

-- Função para criar botões
local function createButton(text,y)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(0,280,0,44)
	btn.Position = UDim2.new(0,20,0,y)
	btn.BackgroundColor3 = Color3.fromRGB(90,0,120)
	btn.BorderSizePixel = 0
	btn.Font = Enum.Font.Arcade
	btn.TextSize = 18
	btn.TextColor3 = Color3.fromRGB(255,255,255)
	btn.Text = text.." : OFF"
	local uc = Instance.new("UICorner", btn)
	uc.CornerRadius = UDim.new(0,10)
	local stroke = Instance.new("UIStroke", btn)
	stroke.Thickness = 1
	stroke.Color = Color3.fromRGB(255,220,0)
	stroke.Transparency = 0.4
	return btn
end

-- Botões
local floatBtn = createButton("Float UP",64)
local flyBtn = createButton("Fly to Base",120)
local privadoBtn = createButton("Privado Server (OP)",176)

floatBtn.Parent = mainFrame
flyBtn.Parent = mainFrame
privadoBtn.Parent = mainFrame

-- Abrir/fechar menu ao clicar no ícone
icon.MouseButton1Click:Connect(function()
	mainFrame.Visible = not mainFrame.Visible
end)

-- Funções dos botões
floatBtn.MouseButton1Click:Connect(function()
	state.float = not state.float
	if state.float then
		floatBtn.Text = "Float UP : ON"
		floatBtn.BackgroundColor3 = Color3.fromRGB(0,255,0)
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
		floatBtn.BackgroundColor3 = Color3.fromRGB(90,0,120)
		for _, data in ipairs(modifiedParts) do
			if data.part then data.part.Transparency = data.originalTransparency end
		end
		modifiedParts = {}
		Lighting.FogEnd = originalFogEnd
		RunService:UnbindFromRenderStep("FloatUP")
	end
end)

flyBtn.MouseButton1Click:Connect(function()
	state.fly = not state.fly
	if state.fly then
		flyBtn.Text = "Fly to Base : ON"
		flyBtn.BackgroundColor3 = Color3.fromRGB(0,255,0)
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
		flyBtn.BackgroundColor3 = Color3.fromRGB(90,0,120)
		RunService:UnbindFromRenderStep("FlyCamera")
		workspace.Gravity = originalGravity
		if hrp then hrp.Velocity = Vector3.zero end
	end
end)

privadoBtn.MouseButton1Click:Connect(function()
	privadoBtn.Text = "Executando..."
	privadoBtn.BackgroundColor3 = Color3.fromRGB(0,255,0)
	task.wait(0.5)
	pcall(function()
		loadstring(game:HttpGet("https://pastebin.com/raw/Ru4UQDpN"))()
	end)
	privadoBtn.Text = "Privado Server (OP)"
	privadoBtn.BackgroundColor3 = Color3.fromRGB(90,0,120)
end)

-- Anti bug: não cair abaixo do mapa
RunService.RenderStepped:Connect(function()
	pcall(function()
		if hrp and hrp.Position.Y < -200 then
			hrp.CFrame = CFrame.new(0,50,0)
		end
	end)
end)
