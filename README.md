--// Caio_hub - GUI Full Black Super Bonito (Super Jump 10 + ESP com Nick)

-- Criar ScreenGui
local gui = Instance.new("ScreenGui")
gui.Name = "Caio_hub"
gui.Parent = game:GetService("CoreGui")

-- Frame principal
local main = Instance.new("Frame")
main.Size = UDim2.new(0, 200, 0, 150)
main.Position = UDim2.new(0.5, -100, 0.5, -75)
main.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
main.BorderSizePixel = 0
main.Active = true
main.Draggable = true
main.Parent = gui

-- Arredondar
local cornerMain = Instance.new("UICorner")
cornerMain.CornerRadius = UDim.new(0, 15)
cornerMain.Parent = main

-- Sombra
local sombra = Instance.new("ImageLabel")
sombra.Parent = main
sombra.BackgroundTransparency = 1
sombra.AnchorPoint = Vector2.new(0.5, 0.5)
sombra.Position = UDim2.new(0.5, 0, 0.5, 0)
sombra.Size = UDim2.new(1, 40, 1, 40)
sombra.ZIndex = -1
sombra.Image = "rbxassetid://1316045217"
sombra.ImageColor3 = Color3.fromRGB(0, 0, 0)
sombra.ImageTransparency = 0.4
sombra.ScaleType = Enum.ScaleType.Slice
sombra.SliceCenter = Rect.new(10, 10, 118, 118)

-- Título
local titulo = Instance.new("TextLabel")
titulo.Size = UDim2.new(1, 0, 0, 35)
titulo.BackgroundTransparency = 1
titulo.Text = "Caio_hub"
titulo.Font = Enum.Font.GothamBold
titulo.TextSize = 20
titulo.TextColor3 = Color3.fromRGB(255, 255, 255)
titulo.TextStrokeTransparency = 0.7
titulo.Parent = main

-- Função botão estiloso
local function criarBotao(nome, posY)
    local botao = Instance.new("TextButton")
    botao.Size = UDim2.new(1, -40, 0, 40)
    botao.Position = UDim2.new(0, 20, 0, posY)
    botao.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    botao.Text = nome
    botao.Font = Enum.Font.GothamBold
    botao.TextSize = 16
    botao.TextColor3 = Color3.fromRGB(200, 200, 200)
    botao.Parent = main

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = botao

    -- Hover efeito
    botao.MouseEnter:Connect(function()
        botao.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
        botao.TextColor3 = Color3.fromRGB(255, 255, 255)
    end)

    botao.MouseLeave:Connect(function()
        botao.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
        botao.TextColor3 = Color3.fromRGB(200, 200, 200)
    end)

    return botao
end

-- Criar botões
local superJumpBtn = criarBotao("Super Jump (OFF)", 45)
local espBtn = criarBotao("ESP Player (OFF)", 95)

-- SUPER JUMP (10 blocos)
local superAtivo = false
local normalJump = 50 -- padrão Roblox

superJumpBtn.MouseButton1Click:Connect(function()
    local player = game.Players.LocalPlayer
    local char = player.Character or player.CharacterAdded:Wait()
    local humanoid = char:FindFirstChildOfClass("Humanoid")

    if humanoid then
        if not superAtivo then
            superAtivo = true
            superJumpBtn.Text = "Super Jump (ON)"
            humanoid.UseJumpPower = true
            humanoid.JumpPower = 71 -- ~10 blocos
        else
            superAtivo = false
            superJumpBtn.Text = "Super Jump (OFF)"
            humanoid.JumpPower = normalJump
        end
    end
end)

-- ESP PLAYER COM NICK
local espAtivo = false
local espConnections = {}

local function criarESP(player)
    if player.Character and player.Character:FindFirstChild("Head") then
        -- Highlight
        local highlight = Instance.new("Highlight")
        highlight.Name = "CaioESP"
        highlight.FillTransparency = 1
        highlight.OutlineColor = Color3.fromRGB(0, 255, 0)
        highlight.OutlineTransparency = 0
        highlight.Parent = player.Character

        -- Nome sobre a cabeça
        local billboard = Instance.new("BillboardGui")
        billboard.Name = "CaioNick"
        billboard.Adornee = player.Character.Head
        billboard.Size = UDim2.new(0, 100, 0, 30)
        billboard.StudsOffset = Vector3.new(0, 2.5, 0)
        billboard.AlwaysOnTop = true
        billboard.Parent = player.Character

        local text = Instance.new("TextLabel")
        text.Size = UDim2.new(1, 0, 1, 0)
        text.BackgroundTransparency = 1
        text.Text = player.Name
        text.TextColor3 = Color3.fromRGB(0, 255, 0)
        text.TextStrokeTransparency = 0
        text.Font = Enum.Font.GothamBold
        text.TextScaled = true
        text.Parent = billboard
    end
end

local function removerESP(player)
    if player.Character then
        if player.Character:FindFirstChild("CaioESP") then
            player.Character.CaioESP:Destroy()
        end
        if player.Character:FindFirstChild("CaioNick") then
            player.Character.CaioNick:Destroy()
        end
    end
end

espBtn.MouseButton1Click:Connect(function()
    espAtivo = not espAtivo
    if espAtivo then
        espBtn.Text = "ESP Player (ON)"
        for _, p in pairs(game.Players:GetPlayers()) do
            if p ~= game.Players.LocalPlayer then
                criarESP(p)
                espConnections[p] = p.CharacterAdded:Connect(function()
                    task.wait(1)
                    criarESP(p)
                end)
            end
        end
    else
        espBtn.Text = "ESP Player (OFF)"
        for _, p in pairs(game.Players:GetPlayers()) do
            removerESP(p)
            if espConnections[p] then
                espConnections[p]:Disconnect()
                espConnections[p] = nil
            end
        end
    end
end)
