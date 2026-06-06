local redzlib = loadstring(game:HttpGet("https://raw.githubusercontent.com/minhdepzai-v/LibraryRobloc/refs/heads/main/RedzLibrary.lua"))()

local Window = redzlib:MakeWindow({
  Title = "Cherry Hub",
  SubTitle = "v9.3 - Desenvolvido por luks",
  SaveFolder = "CherryMM2"
})

Window:AddMinimizeButton({
    Button = { Image = "rbxassetid://78702423919944", BackgroundTransparency = 0 },
    Corner = { CornerRadius = UDim.new(35, 1) },
})

local lp = game.Players.LocalPlayer
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TextChatService = game:GetService("TextChatService")

-- CONFIGURAÇÕES GLOBAIS
local ESP_ENABLED      = false
local HITBOX_ENABLED   = false
local KILLAURA_ENABLED = false
local COIN_ENABLED     = false
local FARM_SPEED       = 60
local HITBOX_SIZE      = 10
local KILLAURA_RADIUS  = 10

local selectedPlayer   = nil
local viewEnabled      = false
local flingTargetLoop  = false
local playerEspEnabled = false

-- NOVAS CONFIGS V9.3
local SILENT_AIM_GUN   = false
local SILENT_AIM_KNIFE = false
local ANTI_FLING       = false
local SCRIPT_DETECTION = false
local AUTO_GRAB_GUN    = false
local CHAT_REVEALER    = false
local XRAY_ENABLED     = false

-- =============================================
-- SISTEMA DE ESP & X-RAY
-- =============================================
local function removeESP(p) 
    if p and p.Character then
        local highlight = p.Character:FindFirstChild("CherryHighlight")
        if highlight then highlight:Destroy() end
    end 
end

local function applyESP(p, color)
    if not p or not p.Character then return end
    local char = p.Character
    local highlight = char:FindFirstChild("CherryHighlight") or Instance.new("Highlight")
    highlight.Name = "CherryHighlight"
    highlight.Parent = char
    highlight.FillColor = color
    highlight.OutlineColor = Color3.new(1, 1, 1)
    highlight.FillTransparency = 0.35
    highlight.OutlineTransparency = 0
    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
end

task.spawn(function()
    while true do
        for _, p in pairs(Players:GetPlayers()) do
            if p == lp then continue end
            if ESP_ENABLED and p.Character and p.Character:FindFirstChild("Humanoid") and p.Character.Humanoid.Health > 0 then
                local isMurderer = p.Backpack:FindFirstChild("Knife") or p.Character:FindFirstChild("Knife")
                local isSheriff = p.Backpack:FindFirstChild("Gun") or p.Character:FindFirstChild("Gun") or p.Backpack:FindFirstChild("Revolver") or p.Character:FindFirstChild("Revolver")
                if isMurderer then applyESP(p, Color3.fromRGB(255, 0, 0))
                elseif isSheriff then applyESP(p, Color3.fromRGB(0, 150, 255))
                else applyESP(p, Color3.fromRGB(0, 255, 100)) end
            elseif playerEspEnabled and p == selectedPlayer then
                applyESP(p, Color3.fromRGB(255, 255, 0))
            else removeESP(p) end
        end
        task.wait(0.3)
    end
end)

-- X-Ray Logic
task.spawn(function()
    while true do
        if XRAY_ENABLED then
            for _, v in pairs(workspace:GetDescendants()) do
                if v:IsA("BasePart") and not v:IsDescendantOf(game.Players.LocalPlayer.Character) then
                    v.LocalTransparencyModifier = 0.5
                end
            end
        else
            for _, v in pairs(workspace:GetDescendants()) do
                if v:IsA("BasePart") then v.LocalTransparencyModifier = 0 end
            end
        end
        task.wait(1)
    end
end)

-- =============================================
-- SISTEMA DE FLING (REVISADO E POTENTE)
-- =============================================
local function executeFling(targetPlayer, mode)
    if not targetPlayer or not targetPlayer.Character then return end
    local tHRP = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
    local tHum = targetPlayer.Character:FindFirstChild("Humanoid")
    if not tHRP or not tHum or tHum.Health <= 0 then return end
    
    local myChar = lp.Character
    local myHRP = myChar and myChar:FindFirstChild("HumanoidRootPart")
    if not myHRP then return end
    
    local savedPos = myHRP.CFrame
    
    -- Desativar colisões para evitar problemas de física com o jogador local
    for _, part in pairs(myChar:GetDescendants()) do
        if part:IsA("BasePart") then part.CanCollide = false end
    end

    -- Aplicar BodyVelocity para o impulso principal
    local bv = Instance.new("BodyVelocity")
    bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
    bv.Velocity = Vector3.new(0, 0, 0) -- Será ajustado pelo modo
    bv.Parent = tHRP

    -- Aplicar AngularVelocity para rotação (efeito de spin)
    local av = Instance.new("AngularVelocity")
    av.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
    av.AngularVelocity = Vector3.new(math.random(-50, 50), math.random(-50, 50), math.random(-50, 50))
    av.Parent = tHRP

    -- Colocar o alvo em PlatformStand para melhor controle do fling
    tHum.PlatformStand = true

    local startTime = tick()
    local connection
    connection = RunService.Heartbeat:Connect(function()
        if tHRP and tHRP.Parent and tHum.Health > 0 then
            if mode == "Orbit" then
                local rot = tick() * 15
                tHRP.CFrame = CFrame.new(myHRP.Position) * CFrame.Angles(0, rot, 0) * CFrame.new(0, 0, 10) -- Orbita o jogador local
                bv.Velocity = (tHRP.Position - myHRP.Position).Unit * 500 + Vector3.new(0, 100, 0) -- Ajuste de velocidade para manter a órbita
            elseif mode == "Ghost" then
                tHRP.CFrame = myHRP.CFrame * CFrame.new(math.random(-5,5), math.random(-5,5), math.random(-5,5))
                bv.Velocity = Vector3.new(math.random(-1000,1000), math.random(-1000,1000), math.random(-1000,1000))
            elseif mode == "Void" then
                tHRP.CFrame = tHRP.CFrame * CFrame.new(0, -50, 0) -- Empurra para baixo
                bv.Velocity = Vector3.new(0, -1000, 0)
            elseif mode == "Mass" then
                -- Mass Fling será tratado por uma função separada ou loop externo
                bv.Velocity = (tHRP.Position - myHRP.Position).Unit * 1000 + Vector3.new(0, 500, 0)
            else -- Instant (Fling normal mais potente)
                local direction = (tHRP.Position - myHRP.Position).Unit * 1500 + Vector3.new(0, 750, 0)
                bv.Velocity = direction
            end
        else
            connection:Disconnect()
        end
    end)
    
    -- Esperar um pouco para o fling acontecer e depois limpar
    task.wait(2) -- Tempo ajustado para o fling ser mais visível
    
    if connection then connection:Disconnect() end
    bv:Destroy(); av:Destroy()
    tHum.PlatformStand = false -- Desativar PlatformStand

    -- Restaurar colisões
    for _, part in pairs(myChar:GetDescendants()) do
        if part:IsA("BasePart") then part.CanCollide = true end
    end
end

local function executeMassFling(mode)
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character and p.Character:FindFirstChild("HumanoidRootPart") and p.Character:FindFirstChild("Humanoid") and p.Character.Humanoid.Health > 0 then
            executeFling(p, mode)
        end
    end
end

-- =============================================
-- COMBATE: SILENT AIM, AIMBOT & AUTO-GRAB
-- =============================================
local function getClosestPlayer()
    local target, dist = nil, math.huge
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character and p.Character:FindFirstChild("HumanoidRootPart") and p.Character.Humanoid.Health > 0 then
            local d = (lp.Character.HumanoidRootPart.Position - p.Character.HumanoidRootPart.Position).Magnitude
            if d < dist then dist = d; target = p end
        end
    end
    return target
end

RunService.RenderStepped:Connect(function()
    if SILENT_AIM_GUN or SILENT_AIM_KNIFE then
        local target = getClosestPlayer()
        if target and target.Character then
            local tool = lp.Character:FindFirstChildOfClass("Tool")
            if tool then
                if (SILENT_AIM_GUN and (tool.Name == "Gun" or tool.Name == "Revolver")) or (SILENT_AIM_KNIFE and tool.Name == "Knife") then
                    workspace.CurrentCamera.CFrame = CFrame.new(workspace.CurrentCamera.CFrame.Position, target.Character.HumanoidRootPart.Position)
                end
            end
        end
    end
    -- View Alvo Fix
    if viewEnabled and selectedPlayer and selectedPlayer.Character and selectedPlayer.Character:FindFirstChild("Humanoid") then
        workspace.CurrentCamera.CameraSubject = selectedPlayer.Character.Humanoid
    else
        if lp.Character and lp.Character:FindFirstChild("Humanoid") then
            workspace.CurrentCamera.CameraSubject = lp.Character.Humanoid
        end
    end
end)

-- Auto Grab Gun
task.spawn(function()
    while true do
        if AUTO_GRAB_GUN then
            local gun = workspace:FindFirstChild("GunDrop") or workspace:FindFirstChild("Gun")
            if gun and gun:IsA("BasePart") and lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") then
                lp.Character.HumanoidRootPart.CFrame = gun.CFrame
            end
        end
        task.wait(0.5)
    end
end)

-- =============================================
-- CHAT REVEALER (FIXED PARA SISTEMA NOVO E ANTIGO)
-- =============================================
local function handleChat(speaker, message)
    if CHAT_REVEALER then
        local p = Players:FindFirstChild(speaker)
        if p then
            local isM = p.Backpack:FindFirstChild("Knife") or p.Character:FindFirstChild("Knife")
            if isM then
                print("[Cherry Reveal] " .. p.Name .. " É O ASSASSINO!")
                -- Opcional: Notificação na tela se quiser
            end
        end
    end
end

-- Suporte para Legacy Chat
task.spawn(function()
    local chatEvents = ReplicatedStorage:WaitForChild("DefaultChatSystemChatEvents", 5)
    if chatEvents then
        local onMessage = chatEvents:WaitForChild("OnMessageDoneFiltering", 5)
        if onMessage then
            onMessage.OnClientEvent:Connect(function(data)
                handleChat(data.FromSpeaker, data.Message)
            end)
        end
    end
end)

-- Suporte para TextChatService (Novo Sistema)
if TextChatService.ChatVersion == Enum.ChatVersion.TextChatService then
    TextChatService.MessageReceived:Connect(function(message)
        if message.TextSource then
            handleChat(message.TextSource.Name, message.Text)
        end
    end)
end

-- =============================================
-- AUTO FARM COINS
-- =============================================
task.spawn(function()
    while true do
        if COIN_ENABLED and lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") then
            local closestCoin = nil
            local minDist = math.huge

            for _, v in pairs(workspace:GetDescendants()) do
                if v.Name == "Coin" and v:IsA("BasePart") then
                    local dist = (lp.Character.HumanoidRootPart.Position - v.Position).Magnitude
                    if dist < minDist then
                        minDist = dist
                        closestCoin = v
                    end
                end
            end

            if closestCoin then
                lp.Character.HumanoidRootPart.CFrame = closestCoin.CFrame
                task.wait(0.1) -- Pequena pausa para garantir a coleta
            end
        end
        task.wait(1 / FARM_SPEED) -- Controla a velocidade do farm
    end
end)

-- =============================================
-- UI TABS SETUP
-- =============================================
local T1 = Window:MakeTab({"Home", ""})
local T2 = Window:MakeTab({"Inocente", ""})
local T3 = Window:MakeTab({"Assassino", ""})
local T4 = Window:MakeTab({"Xerife", ""})
local T5 = Window:MakeTab({"Troll", ""})
local T6 = Window:MakeTab({"Misc", ""})

-- ABA HOME
T1:AddParagraph({"🌸 Cherry Hub v9.3", "Desenvolvido por luks.\nFunções restauradas e sistema de Fling Otimizado."})

-- ABA INOCENTE
T2:AddSection({"Combate"})
T2:AddToggle({Name="ESP Global", Default=false, Callback=function(v) ESP_ENABLED=v end})
T2:AddButton({"🔪 Kill Murder (Fling Instant)", function()
    local t; for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character and (p.Backpack:FindFirstChild("Knife") or p.Character:FindFirstChild("Knife")) then t=p break end
    end
    if t then executeFling(t, "Instant") end
end})
T2:AddButton({"🔫 Roubar Arma (Fling Instant)", function()
    local t; for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character and (p.Backpack:FindFirstChild("Gun") or p.Character:FindFirstChild("Gun") or p.Backpack:FindFirstChild("Revolver") or p.Character:FindFirstChild("Revolver")) then t=p break end
    end
    if t then executeFling(t, "Instant") end
end})
T2:AddButton({"Mass Fling (Todos os Assassinos)", function()
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character and (p.Backpack:FindFirstChild("Knife") or p.Character:FindFirstChild("Knife")) then
            executeFling(p, "Instant")
        end
    end
end})

T2:AddSection({"💰 Farm de Moedas"})
T2:AddToggle({Name="Ativar Auto Farm", Default=false, Callback=function(v) COIN_ENABLED=v end})
T2:AddSlider({Name="Velocidade do Farm", Min=10, Max=200, Default=60, Callback=function(v) FARM_SPEED=v end})
T2:AddParagraph({"⚠️ Aviso", "Aumentar demais a velocidade do Farm tem risco de kick."})

-- ABA ASSASSINO
T3:AddSection({"🎯 Combate"})
T3:AddToggle({Name="Aimbot de Faca", Default=false, Callback=function(v) SILENT_AIM_KNIFE=v end})
T3:AddToggle({Name="Ativar Hitbox", Default=false, Callback=function(v) HITBOX_ENABLED=v end})
T3:AddToggle({Name="Ativar Kill Aura", Default=false, Callback=function(v) KILLAURA_ENABLED=v end})
T3:AddButton({"Fling Inocente (Instant)", function()
    local t; for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character and not (p.Backpack:FindFirstChild("Knife") or p.Character:FindFirstChild("Knife")) and not (p.Backpack:FindFirstChild("Gun") or p.Character:FindFirstChild("Gun") or p.Backpack:FindFirstChild("Revolver") or p.Character:FindFirstChild("Revolver")) then t=p break end
    end
    if t then executeFling(t, "Instant") end
end})
T3:AddButton({"Fling Xerife (Instant)", function()
    local t; for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character and (p.Backpack:FindFirstChild("Gun") or p.Character:FindFirstChild("Gun") or p.Backpack:FindFirstChild("Revolver") or p.Character:FindFirstChild("Revolver")) then t=p break end
    end
    if t then executeFling(t, "Instant") end
end})
T3:AddButton({"Fling Inocente (Orbit)", function()
    local t; for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character and not (p.Backpack:FindFirstChild("Knife") or p.Character:FindFirstChild("Knife")) and not (p.Backpack:FindFirstChild("Gun") or p.Character:FindFirstChild("Gun") or p.Backpack:FindFirstChild("Revolver") or p.Character:FindFirstChild("Revolver")) then t=p break end
    end
    if t then executeFling(t, "Orbit") end
end})
T3:AddButton({"Fling Inocente (Ghost)", function()
    local t; for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character and not (p.Backpack:FindFirstChild("Knife") or p.Character:FindFirstChild("Knife")) and not (p.Backpack:FindFirstChild("Gun") or p.Character:FindFirstChild("Gun") or p.Backpack:FindFirstChild("Revolver") or p.Character:FindFirstChild("Revolver")) then t=p break end
    end
    if t then executeFling(t, "Ghost") end
end})
T3:AddButton({"Fling Inocente (Void)", function()
    local t; for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character and not (p.Backpack:FindFirstChild("Knife") or p.Character:FindFirstChild("Knife")) and not (p.Backpack:FindFirstChild("Gun") or p.Character:FindFirstChild("Gun") or p.Backpack:FindFirstChild("Revolver") or p.Character:FindFirstChild("Revolver")) then t=p break end
    end
    if t then executeFling(t, "Void") end
end})
T3:AddButton({"Mass Fling (Todos os Inocentes)", function()
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character and not (p.Backpack:FindFirstChild("Knife") or p.Character:FindFirstChild("Knife")) and not (p.Backpack:FindFirstChild("Gun") or p.Character:FindFirstChild("Gun") or p.Backpack:FindFirstChild("Revolver") or p.Character:FindFirstChild("Revolver")) then
            executeFling(p, "Instant")
        end
    end
end})

-- ABA XERIFE
T4:AddSection({"🎯 Combate"})
T4:AddToggle({Name="Aimbot (Grudar no Murder)", Default=false, Callback=function(v) SILENT_AIM_GUN=v end})
T4:AddToggle({Name="Silent Aim", Default=false, Callback=function(v) SILENT_AIM_GUN=v end})
T4:AddToggle({Name="Auto Grab Gun", Default=false, Callback=function(v) AUTO_GRAB_GUN=v end})
T4:AddButton({"Fling Murder (Instant)", function()
    local t; for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character and (p.Backpack:FindFirstChild("Knife") or p.Character:FindFirstChild("Knife")) then t=p break end
    end
    if t then executeFling(t, "Instant") end
end})
T4:AddButton({"Fling Murder (Orbit)", function()
    local t; for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character and (p.Backpack:FindFirstChild("Knife") or p.Character:FindFirstChild("Knife")) then t=p break end
    end
    if t then executeFling(t, "Orbit") end
end})
T4:AddButton({"Fling Murder (Ghost)", function()
    local t; for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character and (p.Backpack:FindFirstChild("Knife") or p.Character:FindFirstChild("Knife")) then t=p break end
    end
    if t then executeFling(t, "Ghost") end
end})
T4:AddButton({"Fling Murder (Void)", function()
    local t; for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character and (p.Backpack:FindFirstChild("Knife") or p.Character:FindFirstChild("Knife")) then t=p break end
    end
    if t then executeFling(t, "Void") end
end})
T4:AddButton({"Mass Fling (Todos os Assassinos)", function()
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character and (p.Backpack:FindFirstChild("Knife") or p.Character:FindFirstChild("Knife")) then
            executeFling(p, "Instant")
        end
    end
end})

-- ABA TROLL
T5:AddSection({"🎯 Selecionar Alvo"})
local function getPlayerNames()
    local n = {}
    for _, p in pairs(Players:GetPlayers()) do if p ~= lp then table.insert(n, p.Name) end end
    return n
end

local pDropdown = T5:AddDropdown({
    Name = "Escolher Player",
    Options = getPlayerNames(),
    Default = "",
    Callback = function(v) selectedPlayer = Players:FindFirstChild(v) end
})

local function updateDropdown() pDropdown:SetOptions(getPlayerNames()) end
Players.PlayerAdded:Connect(updateDropdown)
Players.PlayerRemoving:Connect(updateDropdown)

T5:AddSection({"🔥 Ações no Alvo"})
T5:AddButton({"Fling Instant", function() if selectedPlayer then executeFling(selectedPlayer, "Instant") end end})
T5:AddToggle({Name="View Alvo", Default=false, Callback=function(v) viewEnabled = v end})
T5:AddToggle({Name="ESP Alvo", Default=false, Callback=function(v) playerEspEnabled = v end})

T5:AddSection({"💥 Fling Avançado"})
T5:AddButton({"Orbit Fling (Risco de Kick)", function() if selectedPlayer then executeFling(selectedPlayer, "Orbit") end end})
T5:AddButton({"Ghost Fling", function() if selectedPlayer then executeFling(selectedPlayer, "Ghost") end end})
T5:AddButton({"Void Fling", function() if selectedPlayer then executeFling(selectedPlayer, "Void") end end})
T5:AddButton({"Mass Fling (Todos os Jogadores)", function() executeMassFling("Instant") end})
T5:AddParagraph({"⚠️ Aviso", "Orbit Fling pode fazer o alvo ser kickado pelo anti-cheat."})

-- Novas Funções de Troll
T5:AddSection({"😈 Funções de Troll Adicionais"})
T5:AddButton({"Teleportar Alvo para Mim", function()
    if selectedPlayer and selectedPlayer.Character and lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") then
        selectedPlayer.Character.HumanoidRootPart.CFrame = lp.Character.HumanoidRootPart.CFrame * CFrame.new(0, 0, -5) -- Teleporta para perto do jogador local
    end
end})
T5:AddButton({"Congelar Alvo", function()
    if selectedPlayer and selectedPlayer.Character and selectedPlayer.Character:FindFirstChild("Humanoid") then
        selectedPlayer.Character.Humanoid.WalkSpeed = 0
        selectedPlayer.Character.Humanoid.JumpPower = 0
    end
end})
T5:AddButton({"Descongelar Alvo", function()
    if selectedPlayer and selectedPlayer.Character and selectedPlayer.Character:FindFirstChild("Humanoid") then
        selectedPlayer.Character.Humanoid.WalkSpeed = 16 -- Velocidade padrão
        selectedPlayer.Character.Humanoid.JumpPower = 50 -- Poder de pulo padrão
    end
end})
T5:AddButton({"Tornar Alvo Transparente", function()
    if selectedPlayer and selectedPlayer.Character then
        for _, part in pairs(selectedPlayer.Character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.Transparency = 1
            end
        end
    end
end})
T5:AddButton({"Restaurar Transparência do Alvo", function()
    if selectedPlayer and selectedPlayer.Character then
        for _, part in pairs(selectedPlayer.Character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.Transparency = 0
            end
        end
    end
end})

-- ABA MISC
T6:AddSection({"⚡ Movimentação"})
T6:AddSlider({Name="Velocidade", Min=16, Max=150, Default=16, Callback=function(v) if lp.Character then lp.Character.Humanoid.WalkSpeed = v end end})
T6:AddParagraph({"⚠️ Aviso", "Aumentar demais a velocidade pode dar kick."})

T6:AddSection({"🛡️ Proteção"})
T6:AddToggle({Name="Ativar Anti-Fling", Default=false, Callback=function(v) ANTI_FLING=v end})
T6:AddToggle({Name="Detecção de Scripts", Default=false, Callback=function(v) SCRIPT_DETECTION=v end})

T6:AddSection({"👁️ Utilidades"})
T6:AddToggle({Name="X-Ray", Default=false, Callback=function(v) XRAY_ENABLED=v end})
T6:AddToggle({Name="Chat Revealer", Default=false, Callback=function(v) CHAT_REVEALER=v end})

Window:SelectTab(T1)
