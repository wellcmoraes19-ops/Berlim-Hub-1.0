for _,v in pairs(game.CoreGui:GetChildren()) do
	if v:IsA("ScreenGui") and v.Name == "DevPainel" then
		v:Destroy()
	end
end
local lightMode = false
--==================================================
-- SERVICES
--==================================================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local player = Players.LocalPlayer

--==================================================
-- CHARACTER
--==================================================
local char, humanoid, root
local function loadChar()
    char = player.Character or player.CharacterAdded:Wait()
    humanoid = char:WaitForChild("Humanoid")
    root = char:WaitForChild("HumanoidRootPart")
end
loadChar()
player.CharacterAdded:Connect(loadChar)

--==================================================
-- GUI BASE
--==================================================
local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "DevPainel"
gui.ResetOnSpawn = false

-- FPS
local fpsLabel = Instance.new("TextLabel", gui)
fpsLabel.Size = UDim2.new(0,120,0,28)
fpsLabel.Position = UDim2.new(0,10,0,10)
fpsLabel.BackgroundColor3 = Color3.fromRGB(20,20,20)
fpsLabel.TextColor3 = Color3.new(1,1,1)
fpsLabel.Font = Enum.Font.GothamBold
fpsLabel.TextSize = 14
fpsLabel.BorderSizePixel = 0
fpsLabel.Text = "FPS: 0"
Instance.new("UICorner", fpsLabel).CornerRadius = UDim.new(0,8)

do
    local fps,last = 0,tick()
    RunService.RenderStepped:Connect(function()
        fps += 1
        if tick()-last >= 1 then
            fpsLabel.Text = "FPS: "..fps
            fps = 0
            last = tick()
        end
    end)
end

--==================================================
-- MAIN PANEL
--==================================================
local main = Instance.new("Frame", gui)
main.Size = UDim2.new(0,560,0,380)
main.Position = UDim2.new(0.5,-280,0.5,-190)
main.BackgroundColor3 = Color3.fromRGB(18,18,18)
main.BorderSizePixel = 0
main.Visible = false
Instance.new("UICorner", main).CornerRadius = UDim.new(0,14)

--==================================================
-- OPEN BUTTON
--==================================================
-- BOTÃO DE ABRIR PAINEL COM IMAGEM
local openBtn = Instance.new("ImageButton")
openBtn.Size = UDim2.new(0,60,0,60)
openBtn.Position = UDim2.new(0,15,0.5,-30)
openBtn.Parent = gui
openBtn.BackgroundTransparency = 0


-- COLOQUE AQUI O ID DO SEU DECAL (imagem do Berlim)
openBtn.Image = "rbxassetid://5323699019"

-- Toggle do painel
openBtn.MouseButton1Click:Connect(function()
    main.Visible = not main.Visible
end)

--==================================================
-- DRAG MAIN PANEL
--==================================================
do
    local dragging,startPos,dragStart
    main.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            startPos = main.Position
            dragStart = i.Position
        end
    end)
    UIS.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
    end)
    UIS.InputChanged:Connect(function(i)
        if dragging and i.UserInputType == Enum.UserInputType.MouseMovement then
            local d = i.Position - dragStart
            main.Position = UDim2.new(
                startPos.X.Scale,startPos.X.Offset+d.X,
                startPos.Y.Scale,startPos.Y.Offset+d.Y
            )
        end
    end)
end

--==================================================
-- MENU / TABS
--==================================================
local menu = Instance.new("Frame", main)
menu.Size = UDim2.new(0,150,1,0)
menu.BackgroundColor3 = Color3.fromRGB(22,22,22)
menu.BorderSizePixel = 0
Instance.new("UICorner", menu).CornerRadius = UDim.new(0,14)

local title = Instance.new("TextLabel", main)
title.Size = UDim2.new(1,-160,0,40)
title.Position = UDim2.new(0,160,0,5)
title.BackgroundTransparency = 1
title.Text = "PRINCIPAL"
title.Font = Enum.Font.GothamBold
title.TextSize = 18
title.TextColor3 = Color3.new(1,1,1)

local tabs = {}
local function createTab(name)
    local f = Instance.new("Frame", main)
    f.Size = UDim2.new(1,-160,1,-60)
    f.Position = UDim2.new(0,160,0,55)
    f.BackgroundTransparency = 1
    f.Visible = false
    tabs[name] = f
    return f
end

local principal = createTab("Principal")
local antis = createTab("Anti")
local tp = createTab("TP")
local troll = createTab("Troll")
local config = createTab("Config")
local esp = createTab("ESP")
local notas = createTab("Notas")
principal.Visible = true


local function menuBtn(name,y)
    local b = Instance.new("TextButton", menu)
    b.Size = UDim2.new(1,0,0,44)
    b.Position = UDim2.new(0,0,0,y)
    b.Text = name
    b.Font = Enum.Font.GothamBold
    b.TextSize = 14
    b.TextColor3 = Color3.new(1,1,1)
    b.BackgroundColor3 = Color3.fromRGB(30,30,30)
    b.BorderSizePixel = 0
    b.MouseButton1Click:Connect(function()
        for _,v in pairs(tabs) do v.Visible=false end
        tabs[name].Visible=true
        title.Text = string.upper(name)
    end)
end

menuBtn("Principal",10)
menuBtn("Anti",60)
menuBtn("TP",110)
menuBtn("Troll",160)
menuBtn("Config",210)
menuBtn("ESP",260)
menuBtn("Notas",310)


--==================================================
-- UI HELPERS
--==================================================
local function inputBox(parent,text,y,callback)
    local box = Instance.new("TextBox", parent)
    box.Size = UDim2.new(0,260,0,36)
    box.Position = UDim2.new(0,20,0,y)
    box.PlaceholderText = text
    box.BackgroundColor3 = Color3.fromRGB(35,35,35)
    box.TextColor3 = Color3.new(1,1,1)
    box.Font = Enum.Font.Gotham
    box.TextSize = 14
    box.BorderSizePixel = 0
    Instance.new("UICorner", box).CornerRadius = UDim.new(0,8)
    box.FocusLost:Connect(function()
        local v = tonumber(box.Text)
        if v then callback(v) end
    end)
end

local function makeButton(parent,text,y,color,callback)
    local b = Instance.new("TextButton", parent)
    b.Size = UDim2.new(0,260,0,38)
    b.Position = UDim2.new(0,20,0,y)
    b.Text = text
    b.Font = Enum.Font.GothamBold
    b.TextSize = 14
    b.TextColor3 = Color3.new(1,1,1)
    b.BackgroundColor3 = color
    b.BorderSizePixel = 0
    Instance.new("UICorner", b).CornerRadius = UDim.new(0,8)
    b.MouseButton1Click:Connect(callback)
    return b
end

--==================================================
-- PRINCIPAL
--==================================================x(v)
inputBox(principal,"WalkSpeed",20,function(v) humanoid.WalkSpeed=v end)
inputBox(principal,"JumpPower",70,function(v) humanoid.JumpPower=v end)

makeButton(principal,"Reset Speed / Jump",120,Color3.fromRGB(45,45,45),function()
    humanoid.WalkSpeed=16
    humanoid.JumpPower=50
end)

makeButton(principal,"Fly",170,Color3.fromRGB(80,80,80),function()
    loadstring(game:HttpGet("https://raw.githubusercontent.com/XNEOFF/FlyGuiV3/main/FlyGuiV3.txt"))()
end)

local VirtualUser = game:GetService("VirtualUser")

--==================================================
-- ABA ANTI ORGANIZADA
--==================================================
--==================================================
-- ABA ANTI COM BOTÕES ON/OFF
--==================================================
local baseY = 20       -- posição inicial do primeiro botão
local spacing = 10     -- espaço entre os botões
local buttonHeight = 38

-- Função helper para criar botão toggle
local function makeToggleButton(parent, text, y, colorOff, colorOn, stateVar)
    local btn = Instance.new("TextButton", parent)
    btn.Size = UDim2.new(0,260,0,buttonHeight)
    btn.Position = UDim2.new(0,20,0,y)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 14
    btn.TextColor3 = Color3.new(1,1,1)
    btn.BackgroundColor3 = colorOff
    btn.BorderSizePixel = 0
    btn.Text = text..": OFF"
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0,8)

    btn.MouseButton1Click:Connect(function()
        stateVar.value = not stateVar.value
        if stateVar.value then
            btn.Text = text..": ON"
            btn.BackgroundColor3 = colorOn
        else
            btn.Text = text..": OFF"
            btn.BackgroundColor3 = colorOff
        end
    end)
    return btn
end

-- Variáveis de estado
local antiSitVar = {value = false}
local antiVoidVar = {value = false}
local antiAfkVar = {value = false}
local antiAfkConexao

-- Anti Sit
makeToggleButton(antis,"Anti Sit",baseY,Color3.fromRGB(60,60,60),Color3.fromRGB(0,170,0),antiSitVar)
RunService.Stepped:Connect(function()
    if antiSitVar.value and humanoid then
        humanoid.Sit = false
    end
end)

-- Anti Void
makeToggleButton(antis,"Anti Void",baseY + buttonHeight + spacing,Color3.fromRGB(60,60,60),Color3.fromRGB(0,170,0),antiVoidVar)
RunService.Heartbeat:Connect(function()
    if not antiVoidVar.value or not root then return end
    if root.Position.Y > 0 then
        lastSafePosition = root.Position
    end
    if root.Position.Y < -50 and lastSafePosition then
        root.CFrame = CFrame.new(lastSafePosition + Vector3.new(0,5,0))
    end
end)

-- Anti-AFK
makeToggleButton(antis,"Anti-AFK",baseY + (buttonHeight + spacing)*2,Color3.fromRGB(60,60,60),Color3.fromRGB(0,170,0),antiAfkVar)
Players.LocalPlayer.Idled:Connect(function()
    if antiAfkVar.value then
        local VirtualUser = game:GetService("VirtualUser")
        VirtualUser:CaptureController()
        VirtualUser:ClickButton2(Vector2.new())
    end
end)



--==================================================
-- PLAYER LIST
--==================================================
local function playerList(parent,y)
    local scroll = Instance.new("ScrollingFrame", parent)
    scroll.Size = UDim2.new(0,260,0,180)
    scroll.Position = UDim2.new(0,20,0,y)
    scroll.BackgroundColor3 = Color3.fromRGB(30,30,30)
    scroll.ScrollBarThickness = 6
    scroll.BorderSizePixel = 0
    Instance.new("UICorner", scroll).CornerRadius = UDim.new(0,8)

    local layout = Instance.new("UIListLayout", scroll)
    layout.Padding = UDim.new(0,4)

    local selected
    local function refresh()
        for _,v in ipairs(scroll:GetChildren()) do
            if v:IsA("TextButton") then v:Destroy() end
        end
        for _,plr in ipairs(Players:GetPlayers()) do
            if plr~=player then
                local b = Instance.new("TextButton", scroll)
                b.Size = UDim2.new(1,-8,0,32)
                b.Text = plr.Name
                b.Font = Enum.Font.Gotham
                b.TextSize = 13
                b.TextColor3 = Color3.new(1,1,1)
                b.BackgroundColor3 = Color3.fromRGB(45,45,45)
                b.BorderSizePixel = 0
                Instance.new("UICorner", b).CornerRadius = UDim.new(0,6)
                b.MouseButton1Click:Connect(function() selected=plr end)
            end
        end
        task.wait()
        scroll.CanvasSize = UDim2.new(0,0,0,layout.AbsoluteContentSize.Y+6)
    end

    refresh()
    Players.PlayerAdded:Connect(refresh)
    Players.PlayerRemoving:Connect(refresh)
    return function() return selected end
end

--==================================================
-- TP
--==================================================
local getTP = playerList(tp,20)
makeButton(tp,"Teleportar",210,Color3.fromRGB(0,120,255),function()
    local plr = getTP()
    if plr and plr.Character then
        local r = plr.Character:FindFirstChild("HumanoidRootPart")
        if r then root.CFrame = r.CFrame * CFrame.new(0,0,3) end
    end
end)

--==================================================
-- TROLL FUNCIONAL (VOID + FLY)
--==================================================
local getTroll = playerList(troll,20)
local trollActive = false

makeButton(troll,"ATIVAR TROLL",210,Color3.fromRGB(200,60,60),function()
    if trollActive then return end
    local plr = getTroll()
    if not plr or not plr.Character then return end

    local tr = plr.Character:FindFirstChild("HumanoidRootPart")
    local th = plr.Character:FindFirstChild("Humanoid")
    if not tr or not th then return end

    local tool = char:FindFirstChildOfClass("Tool")
    if not tool then
        warn("Você precisa estar com o sofá na mão!")
        return
    end

    trollActive = true
    local startCFrame = root.CFrame

    task.spawn(function()
        while trollActive do
            -- Fica embaixo do alvo (-4 Y) empurrando com o sofá
            root.CFrame = tr.CFrame * CFrame.new(0,-4,0)

            -- Quando o alvo sentar
            if th.Sit then
                -- Ativa Fly
                loadstring(game:HttpGet("https://raw.githubusercontent.com/XNEOFF/FlyGuiV3/main/FlyGuiV3.txt"))()

                -- Teleporta você pro void (altura segura)
				root.CFrame = CFrame.new(tr.Position.X, tr.Position.Y - 30, tr.Position.Z)

				-- Espera um pouquinho para garantir que o alvo sente
				task.wait(0.3)

-- Desequipa sofá
tool.Parent = char
humanoid:UnequipTools()

-- Volta imediatamente para posição inicial
root.CFrame = startCFrame


                trollActive = false
            end

            RunService.RenderStepped:Wait()
        end
    end)
end)

makeButton(troll,"DESATIVAR TROLL",260,Color3.fromRGB(90,90,90),function()
    trollActive = false
end)
--==================================================
-- CONFIG
--==================================================

local lightMode = false

-- FUNÇÃO ATUALIZAR CORES
local function updateTheme()
    if lightMode then
        -- Fundo principal
        main.BackgroundColor3 = Color3.fromRGB(235,235,235)
        menu.BackgroundColor3 = Color3.fromRGB(210,210,210)
        title.TextColor3 = Color3.fromRGB(0,0,0)

        -- Atualiza botões do menu
        for _,v in pairs(menu:GetChildren()) do
            if v:IsA("TextButton") then
                v.BackgroundColor3 = Color3.fromRGB(190,190,190)
                v.TextColor3 = Color3.fromRGB(0,0,0)
            end
        end
    else
        main.BackgroundColor3 = Color3.fromRGB(18,18,18)
        menu.BackgroundColor3 = Color3.fromRGB(22,22,22)
        title.TextColor3 = Color3.fromRGB(255,255,255)

        for _,v in pairs(menu:GetChildren()) do
            if v:IsA("TextButton") then
                v.BackgroundColor3 = Color3.fromRGB(30,30,30)
                v.TextColor3 = Color3.fromRGB(255,255,255)
            end
        end
    end
end

-- MODO CLARO (AGORA MAIS PRA CIMA)
makeButton(config,"Modo Claro",20,Color3.fromRGB(200,200,200),function()
    lightMode = not lightMode
    updateTheme()
end)

makeButton(config,"Fechar Painel",70,Color3.fromRGB(120,40,40),function()
    main.Visible = false
end)

makeButton(config,"Resetar Personagem",120,Color3.fromRGB(40,120,200),function()
    humanoid.Health = 0
end)

-- CRÉDITOS
local creditos = Instance.new("TextLabel", config)
creditos.Size = UDim2.new(0,260,0,30)
creditos.Position = UDim2.new(0,20,0,170)
creditos.BackgroundTransparency = 1
creditos.Text = "Créditos: Chat GPT & Berlim"
creditos.Font = Enum.Font.Gotham
creditos.TextSize = 13
creditos.TextColor3 = Color3.new(1,1,1)

-- BETA
local beta = Instance.new("TextLabel", config)
beta.Size = UDim2.new(0,260,0,25)
beta.Position = UDim2.new(0,20,0,195)
beta.BackgroundTransparency = 1
beta.Text = "Status: Em fase BETA"
beta.Font = Enum.Font.Gotham
beta.TextSize = 12
beta.TextColor3 = Color3.fromRGB(255,170,0)

-- KILL PAINEL
makeButton(config,"KILL PAINEL",230,Color3.fromRGB(150,0,0),function()
    gui:Destroy()
end)
--==================================================
-- ESP VISUAL MELHORADO (DEV SAFE)
--==================================================
local spectating = false
local getSpectate = playerList(esp,80)

makeButton(esp,"Spectator",270,Color3.fromRGB(0,120,255),function()
	local plr = getSpectate()
	if not plr or not plr.Character then return end
	
	spectating = not spectating
	
	if spectating then
		workspace.CurrentCamera.CameraSubject = plr.Character:FindFirstChild("Humanoid")
	else
		workspace.CurrentCamera.CameraSubject = humanoid
	end
end)
local espEnabled = false
local espObjects = {}

local function clearESP()

    -- Remove objetos salvos
    for _,v in pairs(espObjects) do
        if v and v.Parent then
            v:Destroy()
        end
    end
    espObjects = {}

    -- Segurança extra: remove qualquer Highlight ou Billboard
    for _,plr in ipairs(Players:GetPlayers()) do
        if plr.Character then
            for _,obj in ipairs(plr.Character:GetDescendants()) do
                if obj:IsA("Highlight") or obj:IsA("BillboardGui") then
                    obj:Destroy()
                end
            end
        end
    end
end

local function createESP(plr)
	if plr == player then return end
	if not plr.Character then return end
	
	local char = plr.Character
	local hrp = char:FindFirstChild("HumanoidRootPart")
	if not hrp then return end

	local highlight = Instance.new("Highlight")
	highlight.FillTransparency = 1
	highlight.OutlineColor = plr.TeamColor.Color
	highlight.OutlineTransparency = 0
	highlight.Parent = char

	table.insert(espObjects, highlight)
end

local function refreshESP()
    clearESP()
    if not espEnabled then return end
    for _,plr in ipairs(Players:GetPlayers()) do
        createESP(plr)
    end
end

--==================================================
-- ESP TOGGLE
--==================================================
local espButton

espButton = makeButton(esp,"ESP: OFF",20,Color3.fromRGB(120,0,0),function()
    espEnabled = not espEnabled
    
    if espEnabled then
        espButton.Text = "ESP: ON"
        espButton.BackgroundColor3 = Color3.fromRGB(0,170,0)
        refreshESP()
    else
        espButton.Text = "ESP: OFF"
        espButton.BackgroundColor3 = Color3.fromRGB(120,0,0)
        clearESP()
    end
end)
--==================================================
-- NOTAS
--==================================================

local notasFrame = Instance.new("Frame", notas)
notasFrame.Size = UDim2.new(0.9,0,0.75,0)
notasFrame.Position = UDim2.new(0.05,0,0.12,0)
notasFrame.BackgroundColor3 = Color3.fromRGB(30,30,30)
notasFrame.BorderSizePixel = 0
Instance.new("UICorner", notasFrame).CornerRadius = UDim.new(0,12)

notasTexto = Instance.new("TextLabel", notasFrame)
notasTexto.Size = UDim2.new(1,-20,1,-20)
notasTexto.Position = UDim2.new(0,10,0,10)
notasTexto.BackgroundTransparency = 1
notasTexto.TextWrapped = true
notasTexto.TextXAlignment = Enum.TextXAlignment.Center
notasTexto.TextYAlignment = Enum.TextYAlignment.Top
notasTexto.Font = Enum.Font.Gotham
notasTexto.TextSize = 16
notasTexto.TextColor3 = Color3.fromRGB(255,255,255)

notasTexto.Text = [[
📌 NOTAS DO PAINEL

Este painel está em desenvolvimento.

Futuramente haverá atualizações que irão adicionar:

• 🌌 Sistema de Skybox
• 👻 Invisível melhorado
• 🎨 Aba Visual
• ⚙ Melhorias gerais
Aguarde as próximas versões.
by Berlim
]]
