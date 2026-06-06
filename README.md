local redzlib = loadstring(game:HttpGet("https://raw.githubusercontent.com/minhdepzai-v/LibraryRobloc/refs/heads/main/RedzLibrary.lua"))()

local Window = redzlib:MakeWindow({
  Title = "Cherry Hub",
  SubTitle = "v3.4 - Full Update",
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

-- CONFIGURACOES GLOBBAIS
local ESP_ENABLED           = false
local HITBOX_ENABLED        = false
local KILLAURA_ENABLED      = false
local COIN_ENABLED          = false
local FARM_SPEED            = 60
local HITBOX_SIZE           = 10
local KILLAURA_RADIUS       = 10
local SILENT_AIM_ENABLED    = false
local AIMBOT_ENABLED        = false
local ESP_ASSASSINO_ENABLED = false

local selectedPlayer  = nil
local viewEnabled     = false
local flingTargetLoop = false
local playerEspEnabled = false

-- =============================================
-- SISTEMA DE ESP
-- =============================================
local function removeESP(p)
    if not p or not p.Character then return end
    if p.Character:FindFirstChild("CherryHighlight") then
        p.Character.CherryHighlight:Destroy()
    end
    if p.Character:FindFirstChild("CherryESPBill") then
        p.Character.CherryESPBill:Destroy()
    end
end

local function applyESP(p, color, label)
    if not p or not p.Character then return end
    local hrp  = p.Character:FindFirstChild("HumanoidRootPart")
    local head = p.Character:FindFirstChild("Head")
    if not hrp or not head then return end

    -- Highlight
    if not p.Character:FindFirstChild("CherryHighlight") then
        local h = Instance.new("Highlight")
        h.Name               = "CherryHighlight"
        h.FillColor          = color
        h.OutlineColor       = Color3.new(1, 1, 1)
        h.FillTransparency   = 0.3
        h.OutlineTransparency = 0
        h.DepthMode          = Enum.HighlightDepthMode.AlwaysOnTop
        h.Parent             = p.Character
    else
        local h = p.Character.CherryHighlight
        h.FillColor = color
    end

    -- Billboard com nome, role e distancia
    local myHRP = lp.Character and lp.Character:FindFirstChild("HumanoidRootPart")
    local dist  = myHRP and math.floor((myHRP.Position - hrp.Position).Magnitude) or 0

    local bill = p.Character:FindFirstChild("CherryESPBill")
    if not bill then
        bill             = Instance.new("BillboardGui")
        bill.Name        = "CherryESPBill"
        bill.Adornee     = head
        bill.AlwaysOnTop = true
        bill.Size        = UDim2.new(0, 130, 0, 55)
        bill.StudsOffset = Vector3.new(0, 2.5, 0)
        bill.ResetOnSpawn = false
        bill.Parent      = p.Character

        local frame = Instance.new("Frame")
        frame.BackgroundTransparency = 1
        frame.Size   = UDim2.new(1, 0, 1, 0)
        frame.Parent = bill

        local nameLabel = Instance.new("TextLabel")
        nameLabel.Name                 = "NameLabel"
        nameLabel.BackgroundTransparency = 1
        nameLabel.Size                 = UDim2.new(1, 0, 0.5, 0)
        nameLabel.Position             = UDim2.new(0, 0, 0, 0)
        nameLabel.Font                 = Enum.Font.GothamBold
        nameLabel.TextSize             = 14
        nameLabel.TextStrokeTransparency = 0
        nameLabel.TextStrokeColor3     = Color3.new(0, 0, 0)
        nameLabel.TextColor3           = color
        nameLabel.Text                 = p.Name
        nameLabel.Parent               = frame

        local infoLabel = Instance.new("TextLabel")
        infoLabel.Name                 = "InfoLabel"
        infoLabel.BackgroundTransparency = 1
        infoLabel.Size                 = UDim2.new(1, 0, 0.5, 0)
        infoLabel.Position             = UDim2.new(0, 0, 0.5, 0)
        infoLabel.Font                 = Enum.Font.Gotham
        infoLabel.TextSize             = 12
        infoLabel.TextStrokeTransparency = 0
        infoLabel.TextStrokeColor3     = Color3.new(0, 0, 0)
        infoLabel.TextColor3           = Color3.new(1, 1, 1)
        infoLabel.Text                 = label .. " | " .. dist .. "m"
        infoLabel.Parent               = frame
    else
        local frame = bill:FindFirstChildOfClass("Frame")
        if frame then
            local info  = frame:FindFirstChild("InfoLabel")
            local nameL = frame:FindFirstChild("NameLabel")
            if info  then info.Text       = label .. " | " .. dist .. "m" end
            if nameL then nameL.TextColor3 = color end
        end
    end
end

-- LOOP DO ESP
task.spawn(function()
    while task.wait(0.25) do
        for _, p in pairs(Players:GetPlayers()) do
            if p == lp or not p.Character then continue end

            local k = p.Backpack:FindFirstChild("Knife")   or p.Character:FindFirstChild("Knife")
            local g = p.Backpack:FindFirstChild("Gun")     or p.Character:FindFirstChild("Gun")
                   or p.Backpack:FindFirstChild("Revolver") or p.Character:FindFirstChild("Revolver")

            if ESP_ENABLED then
                -- Aba Inocente: so Murder e Xerife
                if k then
                    applyESP(p, Color3.fromRGB(255, 50, 50), "Assassino")
                elseif g then
                    applyESP(p, Color3.fromRGB(50, 150, 255), "Xerife")
                else
                    removeESP(p)
                end
            elseif ESP_ASSASSINO_ENABLED then
                -- Aba Assassino: Inocentes e Xerife
                if g then
                    applyESP(p, Color3.fromRGB(50, 150, 255), "Xerife")
                elseif not k then
                    applyESP(p, Color3.fromRGB(180, 180, 180), "Inocente")
                else
                    removeESP(p)
                end
            elseif playerEspEnabled and p == selectedPlayer then
                applyESP(p, Color3.fromRGB(255, 255, 0), "Alvo")
            else
                removeESP(p)
            end
        end
    end
end)

-- =============================================
-- FLING MELHORADO (TRACKING AGRESSIVO)
-- =============================================
local function executeFling(targetPlayer)
    if not targetPlayer then return end
    local myChar = lp.Character
    local myHRP  = myChar and myChar:FindFirstChild("HumanoidRootPart")
    if not myHRP then return end
    local tChar = targetPlayer.Character
    local tHRP  = tChar and tChar:FindFirstChild("HumanoidRootPart")
    if not tHRP then return end

    local savedPos = myHRP.CFrame

    local bv = Instance.new("BodyVelocity")
    bv.MaxForce = Vector3.new(1, 1, 1) * math.huge
    bv.Velocity = Vector3.new(0, 0, 0)
    bv.Parent   = myHRP

    local bav = Instance.new("BodyAngularVelocity")
    bav.MaxTorque       = Vector3.new(1, 1, 1) * math.huge
    bav.AngularVelocity = Vector3.new(9e8, 9e8, 9e8)
    bav.Parent          = myHRP

    local angle     = 0
    local startTime = tick()

    repeat
        tHRP = targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart")
        if not tHRP then break end

        angle = angle + 180

        -- Predicao agressiva: compensa corrida e pulo
        local vel  = tHRP.Velocity
        local pred = vel * 0.35
        if vel.Y > 0 then
            pred = pred + Vector3.new(0, vel.Y * 0.2, 0)
        end
        local pPos = tHRP.Position + pred

        myHRP.CFrame = CFrame.new(pPos) * CFrame.Angles(math.rad(angle), math.rad(angle * 0.5), 0)

        local sx = math.random(0,1) == 0 and 9e8 or -9e8
        local sz = math.random(0,1) == 0 and 9e8 or -9e8
        bv.Velocity = Vector3.new(sx, 9e8, sz)

        myHRP.AssemblyLinearVelocity  = Vector3.new(9e8, 9e8, 9e8)
        myHRP.AssemblyAngularVelocity = Vector3.new(9e8, 9e8, 9e8)

        task.wait()
    until (tHRP and tHRP.Velocity.Magnitude > 800)
       or tick() > startTime + 2.5
       or not targetPlayer.Parent

    bv:Destroy()
    bav:Destroy()

    for i = 1, 8 do
        myHRP.CFrame                  = savedPos
        myHRP.AssemblyLinearVelocity  = Vector3.new(0, 0, 0)
        myHRP.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
        task.wait()
    end
end

-- =============================================
-- ROUBAR ARMA (Aba Inocente)
-- =============================================
local function roubarArma()
    local xerife = nil
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character then
            local g = p.Character:FindFirstChild("Gun")
                   or p.Backpack:FindFirstChild("Gun")
                   or p.Character:FindFirstChild("Revolver")
                   or p.Backpack:FindFirstChild("Revolver")
            if g then xerife = p break end
        end
    end
    if not xerife then return end

    -- Fling no Xerife para derruba-lo
    executeFling(xerife)
    task.wait(1)

    -- Tenta pegar a arma que caiu no chao
    local myHRP = lp.Character and lp.Character:FindFirstChild("HumanoidRootPart")
    if not myHRP then return end
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("Tool") and (obj.Name == "Gun" or obj.Name == "Revolver") then
            if obj.Parent == workspace or obj.Parent:IsA("Model") then
                myHRP.CFrame = obj:GetPivot()
                task.wait(0.3)
                break
            end
        end
    end
end

-- =============================================
-- COIN FARM SYSTEM (TWEEN)
-- =============================================
local coinCollected = {}
local isTweening = false

local function findCoins()
    local c = {}
    local names = {"MainCoin", "CoinVisual", "Coin", "Coin_Server"}
    for _, o in ipairs(workspace:GetDescendants()) do
        if o:IsA("BasePart") and table.find(names, o.Name) then
            if o.Parent and not coinCollected[o:GetDebugId()] then table.insert(c, o) end
        end
    end
    return c
end

local function safeTeleport(target)
    local hrp = lp.Character and lp.Character:FindFirstChild("HumanoidRootPart")
    if not hrp or not target.Parent then return end
    isTweening = true
    local dist = (hrp.Position - target.Position).Magnitude
    local time = dist / FARM_SPEED
    local tween = TweenService:Create(hrp, TweenInfo.new(time, Enum.EasingStyle.Linear), {
        CFrame = CFrame.new(target.Position + Vector3.new(0, 1, 0))
    })
    tween:Play()
    tween.Completed:Connect(function()
        coinCollected[target:GetDebugId()] = true
        isTweening = false
    end)
    repeat task.wait() until not isTweening
end

local function startCoinFarm()
    coinCollected = {}
    task.spawn(function()
        while COIN_ENABLED do
            if not isTweening and lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") then
                local coins = findCoins()
                if #coins > 0 then
                    table.sort(coins, function(a, b)
                        return (lp.Character.HumanoidRootPart.Position - a.Position).Magnitude
                             < (lp.Character.HumanoidRootPart.Position - b.Position).Magnitude
                    end)
                    safeTeleport(coins[1])
                end
            end
            task.wait(0.1)
        end
    end)
end

-- =============================================
-- COMBAT SYSTEM (HITBOX)
-- =============================================
local function applyHitbox()
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character then
            local r = p.Character:FindFirstChild("HumanoidRootPart")
            if r then
                r.Size        = Vector3.new(HITBOX_SIZE, HITBOX_SIZE, HITBOX_SIZE)
                r.Transparency = 0.7
                r.CanCollide  = false
            end
        end
    end
end

local hitboxConn = nil
local function startHitbox()
    if hitboxConn then hitboxConn:Disconnect() end
    hitboxConn = RunService.Heartbeat:Connect(function()
        if HITBOX_ENABLED then applyHitbox() end
    end)
end

local function stopHitbox()
    HITBOX_ENABLED = false
    if hitboxConn then hitboxConn:Disconnect() hitboxConn = nil end
    for _, p in pairs(Players:GetPlayers()) do
        if p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
            p.Character.HumanoidRootPart.Size        = Vector3.new(2, 2, 1)
            p.Character.HumanoidRootPart.Transparency = 0
        end
    end
end

-- =============================================
-- KILL AURA SYSTEM
-- =============================================
local killAuraConn = nil
local function startKillAura()
    if killAuraConn then killAuraConn:Disconnect() end
    killAuraConn = RunService.Heartbeat:Connect(function()
        if not KILLAURA_ENABLED then return end
        local c   = lp.Character
        local hrp = c and c:FindFirstChild("HumanoidRootPart")
        if not hrp then return end
        local k = c:FindFirstChild("Knife") or lp.Backpack:FindFirstChild("Knife")
        if not k then return end
        if k.Parent == lp.Backpack then c.Humanoid:EquipTool(k) end
        local h = k:FindFirstChild("Handle")
        if not h then return end
        local cl, cd = nil, KILLAURA_RADIUS
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= lp and p.Character and p.Character.Humanoid.Health > 0 then
                local d = (hrp.Position - p.Character.HumanoidRootPart.Position).Magnitude
                if d < cd then cd = d; cl = p end
            end
        end
        if cl then h.CFrame = cl.Character.HumanoidRootPart.CFrame end
    end)
end

-- =============================================
-- XERIFE: SILENT AIM + AIMBOT
-- =============================================
local function getMurder()
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character then
            local k = p.Character:FindFirstChild("Knife") or p.Backpack:FindFirstChild("Knife")
            if k then return p end
        end
    end
    return nil
end

local silentAimConn = nil
local function startSilentAim()
    if silentAimConn then silentAimConn:Disconnect() end
    silentAimConn = RunService.RenderStepped:Connect(function()
        if not SILENT_AIM_ENABLED then return end
        local murder = getMurder()
        if not murder or not murder.Character then return end
        local head = murder.Character:FindFirstChild("Head")
        if not head then return end
        workspace.CurrentCamera.CFrame = CFrame.new(
            workspace.CurrentCamera.CFrame.Position,
            head.Position
        )
    end)
end

local aimbotConn = nil
local function startAimbot()
    if aimbotConn then aimbotConn:Disconnect() end
    aimbotConn = RunService.RenderStepped:Connect(function()
        if not AIMBOT_ENABLED then return end
        local murder = getMurder()
        if not murder or not murder.Character then return end
        local head = murder.Character:FindFirstChild("Head")
        if not head then return end
        local cam = workspace.CurrentCamera
        local screenPos, onScreen = cam:WorldToScreenPoint(head.Position)
        if onScreen then
            mousemoverel(
                screenPos.X - (cam.ViewportSize.X / 2),
                screenPos.Y - (cam.ViewportSize.Y / 2)
            )
        end
    end)
end

-- =============================================
-- PLAYER LIST & CAMERA
-- =============================================
local function getPlayerNames()
    local n = {}
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp then table.insert(n, p.Name) end
    end
    return n
end

RunService.RenderStepped:Connect(function()
    if viewEnabled and selectedPlayer and selectedPlayer.Character and selectedPlayer.Character:FindFirstChild("Humanoid") then
        workspace.CurrentCamera.CameraSubject = selectedPlayer.Character.Humanoid
    else
        if lp.Character and lp.Character:FindFirstChild("Humanoid") then
            workspace.CurrentCamera.CameraSubject = lp.Character.Humanoid
        end
    end
    if playerEspEnabled and selectedPlayer then
        applyESP(selectedPlayer, Color3.fromRGB(255, 255, 0), "Alvo")
    end
end)

-- =============================================
-- UI TABS
-- =============================================
local T1 = Window:MakeTab({"Home", ""})
local T2 = Window:MakeTab({"Inocente", ""})
local T3 = Window:MakeTab({"Assassino", ""})
local T4 = Window:MakeTab({"Xerife", ""})
local T5 = Window:MakeTab({"Troll", ""})
local T6 = Window:MakeTab({"Misc", ""})

T1:AddParagraph({"Cherry Hub v3.4", "Full Update: ESP, Fling, Xerife e Inocente."})

-- ABA INOCENTE
T2:AddSection({"Combate"})
T2:AddToggle({Name="ESP (Murder e Xerife)", Default=false, Callback=function(v) ESP_ENABLED=v end})
T2:AddButton({"Roubar Arma do Xerife", function()
    roubarArma()
end})
T2:AddButton({"Fling Murder", function()
    local t
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character and (p.Backpack:FindFirstChild("Knife") or p.Character:FindFirstChild("Knife")) then
            t = p break
        end
    end
    if t then executeFling(t) end
end})

T2:AddSection({"Farm de Moedas"})
T2:AddToggle({Name="Ativar Auto Farm", Default=false, Callback=function(v) COIN_ENABLED=v; if v then startCoinFarm() end end})
T2:AddSlider({Name="Velocidade do Farm", Min=10, Max=200, Default=60, Callback=function(v) FARM_SPEED=v end})
T2:AddParagraph({"Aviso de Seguranca", "Velocidade alta pode causar kick do servidor."})

-- ABA ASSASSINO
T3:AddSection({"ESP Assassino"})
T3:AddToggle({Name="ESP (Inocentes e Xerife)", Default=false, Callback=function(v)
    ESP_ASSASSINO_ENABLED = v
end})

T3:AddSection({"Hitbox"})
T3:AddSlider({Name="Tamanho da Hitbox", Min=1, Max=50, Default=10, Callback=function(v) HITBOX_SIZE=v end})
T3:AddToggle({Name="Ativar Hitbox", Default=false, Callback=function(v)
    HITBOX_ENABLED = v
    if v then startHitbox() else stopHitbox() end
end})

T3:AddSection({"Kill Aura"})
T3:AddSlider({Name="Raio da Aura", Min=1, Max=60, Default=10, Callback=function(v) KILLAURA_RADIUS=v end})
T3:AddToggle({Name="Ativar Kill Aura", Default=false, Callback=function(v)
    KILLAURA_ENABLED = v
    if v then startKillAura() end
end})

-- ABA XERIFE
T4:AddSection({"Mira"})
T4:AddToggle({Name="Silent Aim (Auto mira Murder)", Default=false, Callback=function(v)
    SILENT_AIM_ENABLED = v
    if v then
        startSilentAim()
    else
        if silentAimConn then silentAimConn:Disconnect() silentAimConn = nil end
    end
end})

T4:AddToggle({Name="Aimbot Murder", Default=false, Callback=function(v)
    AIMBOT_ENABLED = v
    if v then
        startAimbot()
    else
        if aimbotConn then aimbotConn:Disconnect() aimbotConn = nil end
    end
end})

T4:AddSection({"Acoes"})
T4:AddButton({"Auto Atirar no Murder", function()
    local murder = getMurder()
    if not murder or not murder.Character then return end
    local myChar = lp.Character
    if not myChar then return end
    local gun = myChar:FindFirstChild("Gun")      or lp.Backpack:FindFirstChild("Gun")
             or myChar:FindFirstChild("Revolver") or lp.Backpack:FindFirstChild("Revolver")
    if not gun then return end
    if gun.Parent == lp.Backpack then myChar.Humanoid:EquipTool(gun) end
    task.wait(0.2)
    local handle = gun:FindFirstChild("Handle")
    local murderHead = murder.Character:FindFirstChild("Head")
    if handle and murderHead then
        handle.CFrame = murderHead.CFrame
    end
end})

T4:AddParagraph({"Info", "Silent Aim redireciona a camera. Aimbot move o mouse. Use um de cada vez."})

-- ABA TROLL
T5:AddSection({"Selecionar Alvo"})
local pDropdown = T5:AddDropdown({
    Name = "Escolher Player",
    Options = getPlayerNames(),
    Default = "",
    Callback = function(v) selectedPlayer = Players:FindFirstChild(v) end
})

Players.PlayerAdded:Connect(function() pDropdown:SetOptions(getPlayerNames()) end)
Players.PlayerRemoving:Connect(function() pDropdown:SetOptions(getPlayerNames()) end)

T5:AddSection({"Acoes no Alvo"})
T5:AddToggle({Name="Fling Alvo Infinito", Default=false, Callback=function(v)
    flingTargetLoop = v
    task.spawn(function()
        while flingTargetLoop do
            if selectedPlayer then executeFling(selectedPlayer) end
            task.wait(0.3)
        end
    end)
end})

T5:AddToggle({Name="ESP Alvo", Default=false, Callback=function(v)
    playerEspEnabled = v
    if not v and selectedPlayer then removeESP(selectedPlayer) end
end})

T5:AddToggle({Name="View Alvo", Default=false, Callback=function(v) viewEnabled = v end})

T5:AddSection({"Caos Total"})
T5:AddButton({"Fling All Players", function()
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character then executeFling(p) task.wait(0.1) end
    end
end})

-- ABA MISC
T6:AddSection({"Movimentacao"})
T6:AddSlider({Name="Velocidade", Min=16, Max=150, Default=16, Callback=function(v)
    if lp.Character then lp.Character.Humanoid.WalkSpeed = v end
end})

T6:AddSlider({Name="Pulo", Min=50, Max=300, Default=50, Callback=function(v)
    if lp.Character then lp.Character.Humanoid.JumpPower = v end
end})

-- Inicializacao
Window:SelectTab(T1)
