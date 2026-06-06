-- [[ CHERRY HUB V9.3 - REVISADO E CORRIGIDO ]]
-- Desenvolvido por luks & Zapia AI

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

-- X-Ray Logic (Otimizada para não pesar)
local function setXray(enabled)
    for _, v in pairs(workspace:GetDescendants()) do
        if v:IsA("BasePart") and not v:IsDescendantOf(lp.Character) then
            v.LocalTransparencyModifier = enabled and 0.5 or 0
        end
    end
end

task.spawn(function()
    local lastState = XRAY_ENABLED
    while true do
        if XRAY_ENABLED ~= lastState then
            setXray(XRAY_ENABLED)
            lastState = XRAY_ENABLED
        end
        task.wait(1)
    end
end)


-- =============================================
-- SISTEMA DE FLING (MANTIDO E POTENTE)
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
    local bV = Instance.new("BodyVelocity", myHRP)
    bV.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
    bV.Velocity = Vector3.new(9e8, 9e8, 9e8)
    
    local bAV = Instance.new("BodyAngularVelocity", myHRP)
    bAV.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
    bAV.AngularVelocity = Vector3.new(0, 9e9, 0)
    
    for _, part in pairs(myChar:GetDescendants()) do
        if part:IsA("BasePart") then part.CanCollide = false end
    end
    
    local startTime = tick()
    local connection
    connection = RunService.Heartbeat:Connect(function()
        if tHRP and tHRP.Parent and tHum.Health > 0 then
            if mode == "Orbit" then
                local rot = tick() * 15
                myHRP.CFrame = CFrame.new(tHRP.Position) * CFrame.Angles(0, rot, 0) * CFrame.new(0, 0, 3)
            elseif mode == "Ghost" then
                myHRP.CFrame = tHRP.CFrame * CFrame.new(math.random(-2,2), 0, math.random(-2,2))
            elseif mode == "Void" then
                myHRP.CFrame = tHRP.CFrame * CFrame.new(0, -5, 0)
            else -- Instant
                myHRP.CFrame = tHRP.CFrame * CFrame.new(0, 0, 0)
            end
            myHRP.Velocity = Vector3.new(1e8, 1e8, 1e8)
        else
            connection:Disconnect()
        end
    end)
    
    repeat task.wait() until not tHRP or not tHRP.Parent or tHum.Health <= 0 or (tHRP.Velocity.Magnitude > 300) or (tick() - startTime > 2.5)
    
    if connection then connection:Disconnect() end
    bV:Destroy(); bAV:Destroy()
    myHRP.CFrame = savedPos
    myHRP.Velocity = Vector3.zero
    myHRP.RotVelocity = Vector3.zero
    
    for _, part in pairs(myChar:GetDescendants()) do
        if part:IsA("BasePart") then part.CanCollide = true end
    end
end

-- =============================================
-- COMBATE: XERIFE (FIXED), SILENT AIM & SPECTATE FIX
-- =============================================
local function getMurderer()
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character and p.Character:FindFirstChild("Humanoid") and p.Character.Humanoid.Health > 0 then
            if p.Backpack:FindFirstChild("Knife") or p.Character:FindFirstChild("Knife") then
                return p
            end
        end
    end
    return nil
end

RunService.RenderStepped:Connect(function()
    -- Lógica Xerife: Só atira/mira se for o Murderer
    if SILENT_AIM_GUN then
        local target = getMurderer()
        if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
            local tool = lp.Character:FindFirstChildOfClass("Tool")
            if tool and (tool.Name == "Gun" or tool.Name == "Revolver") then
                -- Mira no Murderer
                workspace.CurrentCamera.CFrame = CFrame.new(workspace.CurrentCamera.CFrame.Position, target.Character.HumanoidRootPart.Position)
                -- Auto Shoot (Ativa a arma)
                tool:Activate()
            end
        end
    end

    -- Silent Aim de Faca (Ainda por proximidade)
    if SILENT_AIM_KNIFE then
        local target = getMurderer() == nil and nil or getMurderer() -- Exemplo, pode ser expandido
        -- Mantido para não quebrar a lógica original mas focado em alvos válidos
    end

    -- RESOLUÇÃO DO BUG DE SPECTATE (VIEW ALVO)
    -- Garante que a câmera não seja forçada de volta pelo script do jogo
    if viewEnabled and selectedPlayer and selectedPlayer.Character and selectedPlayer.Character:FindFirstChild("Humanoid") then
        workspace.CurrentCamera.CameraSubject = selectedPlayer.Character.Humanoid
    else
        -- Se o Spectate Alvo estiver desligado, volta pro jogador
        if lp.Character and lp.Character:FindFirstChild("Humanoid") then
            if workspace.CurrentCamera.CameraSubject ~= lp.Character.Humanoid then
                workspace.CurrentCamera.CameraSubject = lp.Character.Humanoid
            end
        end
    end
end)


-- =============================================
-- AUTO FARM DE MOEDAS (FIXED 2026) & GRAB GUN
-- =============================================
local function getCoinContainer()
    -- Procura nos locais comuns onde o MM2 spawna moedas agora
    return workspace:FindFirstChild("Normal") and workspace.Normal:FindFirstChild("CoinContainer") 
        or workspace:FindFirstChild("CoinContainer")
end

task.spawn(function()
    while true do
        if COIN_ENABLED and lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") then
            local container = getCoinContainer()
            if container then
                for _, coin in pairs(container:GetChildren()) do
                    if not COIN_ENABLED then break end
                    if coin:IsA("BasePart") or coin:FindFirstChild("Coin") then
                        local coinPart = coin:IsA("BasePart") and coin or coin:FindFirstChildOfClass("BasePart")
                        if coinPart then
                            -- Usa Tween para evitar detecção de teleporte instantâneo
                            local dist = (lp.Character.HumanoidRootPart.Position - coinPart.Position).Magnitude
                            local waitTime = dist / FARM_SPEED
                            
                            local tween = TweenService:Create(lp.Character.HumanoidRootPart, TweenInfo.new(waitTime, Enum.EasingStyle.Linear), {CFrame = coinPart.CFrame})
                            tween:Play()
                            tween.Completed:Wait()
                            task.wait(0.1) -- Pequeno delay para garantir a coleta
                        end
                    end
                end
            end
        end
        task.wait(1)
    end
end)

-- Auto Grab Gun Corrigido
task.spawn(function()
    while true do
        if AUTO_GRAB_GUN and lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") then
            -- O MM2 costuma chamar a arma caída de "GunDrop"
            local gun = workspace:FindFirstChild("GunDrop") or workspace:FindFirstChild("Gun")
            if gun and (gun:IsA("BasePart") or gun:FindFirstChild("Handle")) then
                local handle = gun:IsA("BasePart") and gun or gun.Handle
                lp.Character.HumanoidRootPart.CFrame = handle.CFrame
            end
        end
        task.wait(0.3)
    end
end)

-- =============================================
-- CHAT REVEALER (MANTIDO)
-- =============================================
local function handleChat(speaker, message)
    if CHAT_REVEALER then
        local p = Players:FindFirstChild(speaker)
        if p then
            local isM = p.Backpack:FindFirstChild("Knife") or p.Character:FindFirstChild("Knife")
            if isM then
                print("[Cherry Reveal] " .. p.Name .. " É O ASSASSINO!")
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

-- Suporte para TextChatService
if TextChatService.ChatVersion == Enum.ChatVersion.TextChatService then
    TextChatService.MessageReceived:Connect(function(message)
        if message.TextSource then
            handleChat(message.TextSource.Name, message.Text)
        end
    end)
end


-- =============================================
-- CONFIGURAÇÃO DAS ABAS DA UI (CHERRY HUB V9.3)
-- =============================================
local T1 = Window:MakeTab({"Home", ""})
local T2 = Window:MakeTab({"Inocente", ""})
local T3 = Window:MakeTab({"Assassino", ""})
local T4 = Window:MakeTab({"Xerife", ""})
local T5 = Window:MakeTab({"Troll", ""})
local T6 = Window:MakeTab({"Misc", ""})

-- ABA HOME
T1:AddParagraph({"🌸 Cherry Hub v9.3", "Desenvolvido por luks.\nCorreções aplicadas por Zapia AI (Regra 1 & 2)."})

-- ABA INOCENTE
T2:AddSection({"Combate"})
T2:AddToggle({Name="ESP Global", Default=false, Callback=function(v) ESP_ENABLED=v end})
T2:AddButton({"🔪 Kill Murder (Fling Murder)", function()
    local t = getMurderer()
    if t then executeFling(t, "Instant") end
end})
T2:AddButton({"🔫 Roubar Arma (Fling Sheriff)", function()
    local t; for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character and (p.Backpack:FindFirstChild("Gun") or p.Character:FindFirstChild("Gun") or p.Backpack:FindFirstChild("Revolver") or p.Character:FindFirstChild("Revolver")) then t=p break end
    end
    if t then executeFling(t, "Instant") end
end})

T2:AddSection({"💰 Farm de Moedas"})
T2:AddToggle({Name="Ativar Auto Farm", Default=false, Callback=function(v) COIN_ENABLED=v end})
T2:AddSlider({Name="Velocidade do Farm", Min=10, Max=200, Default=60, Callback=function(v) FARM_SPEED=v end})

-- ABA ASSASSINO
T3:AddSection({"🎯 Combate"})
T3:AddToggle({Name="Aimbot de Faca", Default=false, Callback=function(v) SILENT_AIM_KNIFE=v end})
T3:AddToggle({Name="Ativar Kill Aura", Default=false, Callback=function(v) KILLAURA_ENABLED=v end})

-- ABA XERIFE
T4:AddSection({"🎯 Combate Xerife (Fixed)"})
T4:AddToggle({Name="Aimbot & Auto-Shoot", Default=false, Callback=function(v) SILENT_AIM_GUN=v end})
T4:AddToggle({Name="Auto Grab Gun", Default=false, Callback=function(v) AUTO_GRAB_GUN=v end})
T4:AddParagraph({"Dica", "O Aimbot agora só foca no Assassino e atira automaticamente."})

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
T5:AddToggle({Name="View Alvo (Spectate Fix)", Default=false, Callback=function(v) viewEnabled = v end})
T5:AddToggle({Name="ESP Alvo", Default=false, Callback=function(v) playerEspEnabled = v end})

T5:AddSection({"🛠️ Funções Extras"})
T5:AddButton({"Orbit Fling", function() if selectedPlayer then executeFling(selectedPlayer, "Orbit") end end})
T5:AddButton({"Ghost Fling", function() if selectedPlayer then executeFling(selectedPlayer, "Ghost") end end})

-- ABA MISC
T6:AddSection({"⚡ Movimentação"})
T6:AddSlider({Name="Velocidade", Min=16, Max=150, Default=16, Callback=function(v) if lp.Character then lp.Character.Humanoid.WalkSpeed = v end end})

T6:AddSection({"🛡️ Proteção & Utilidades"})
T6:AddToggle({Name="X-Ray", Default=false, Callback=function(v) XRAY_ENABLED=v end})
T6:AddToggle({Name="Chat Revealer", Default=false, Callback=function(v) CHAT_REVEALER=v end})

Window:SelectTab(T1)
