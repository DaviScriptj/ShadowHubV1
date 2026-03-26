--[[
    SHADOW HUB V.1 - SCRIPT COMPLETO
    Funcionalidades:
    - Invisibilidade REAL (outros jogadores NÃO veem)
    - Hitbox 50 blocos (acerta players à distância)
    - ESP colorido (wallhack)
    - Menu estilo Shadow Hub com botão flutuante
    
    ⚠️ USE APENAS PARA TESTES COM ADMS ⚠️
]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local LocalPlayer = Players.LocalPlayer

-- Aguardar personagem
local character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")

-- ==================== CRIAR GUI ====================
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "ShadowHubV1"
screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
screenGui.ResetOnSpawn = false

-- Botão flutuante (bolinha estilo Shadow Hub)
local floatingBtn = Instance.new("TextButton")
floatingBtn.Size = UDim2.new(0, 55, 0, 55)
floatingBtn.Position = UDim2.new(0, 15, 0, 100)
floatingBtn.BackgroundColor3 = Color3.fromRGB(80, 40, 120)
floatingBtn.Text = "🌑"
floatingBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
floatingBtn.TextSize = 28
floatingBtn.TextScaled = true
floatingBtn.Font = Enum.Font.GothamBold
floatingBtn.Parent = screenGui

local btnCorner = Instance.new("UICorner")
btnCorner.CornerRadius = UDim.new(1, 0)
btnCorner.Parent = floatingBtn

local btnShadow = Instance.new("UIShadow")
btnShadow.Parent = floatingBtn

-- Menu Principal
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 340, 0, 460)
mainFrame.Position = UDim2.new(0.5, -170, 0.5, -230)
mainFrame.BackgroundColor3 = Color3.fromRGB(10, 10, 18)
mainFrame.BackgroundTransparency = 0.05
mainFrame.Visible = false
mainFrame.Parent = screenGui

local frameCorner = Instance.new("UICorner")
frameCorner.CornerRadius = UDim.new(0, 12)
frameCorner.Parent = mainFrame

local frameShadow = Instance.new("UIShadow")
frameShadow.Parent = mainFrame

-- Título
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 55)
title.Position = UDim2.new(0, 0, 0, 0)
title.BackgroundTransparency = 1
title.Text = "🌑 SHADOW HUB V.1 🌑"
title.TextColor3 = Color3.fromRGB(140, 80, 200)
title.TextSize = 22
title.Font = Enum.Font.GothamBold
title.Parent = mainFrame

-- Subtítulo
local subtitle = Instance.new("TextLabel")
subtitle.Size = UDim2.new(1, 0, 0, 30)
subtitle.Position = UDim2.new(0, 0, 0, 50)
subtitle.BackgroundTransparency = 1
subtitle.Text = "Modo Teste | Invisibilidade REAL"
subtitle.TextColor3 = Color3.fromRGB(130, 130, 150)
subtitle.TextSize = 11
subtitle.Font = Enum.Font.Gotham
subtitle.Parent = mainFrame

-- Função para criar botões
local function createButton(text, yPos, color, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.85, 0, 0, 48)
    btn.Position = UDim2.new(0.075, 0, 0, yPos)
    btn.BackgroundColor3 = color or Color3.fromRGB(25, 25, 35)
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.TextSize = 14
    btn.Font = Enum.Font.GothamSemibold
    btn.Parent = mainFrame
    
    local btnCorner2 = Instance.new("UICorner")
    btnCorner2.CornerRadius = UDim.new(0, 8)
    btnCorner2.Parent = btn
    
    btn.MouseButton1Click:Connect(callback)
    
    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundColor3 = Color3.fromRGB(65, 45, 95)}):Play()
    end)
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundColor3 = color or Color3.fromRGB(25, 25, 35)}):Play()
    end)
    
    return btn
end

-- ==================== INVISIBILIDADE REAL ====================
local invisivel = false
local originalProperties = {}

local function aplicarInvisibilidadeReal(ativo)
    local char = LocalPlayer.Character
    if not char then return end
    
    for _, part in ipairs(char:GetDescendants()) do
        if part:IsA("BasePart") then
            if ativo then
                if not originalProperties[part] then
                    originalProperties[part] = {
                        Transparency = part.Transparency,
                        LocalTransparencyModifier = part.LocalTransparencyModifier
                    }
                end
                part.Transparency = 1
                part.LocalTransparencyModifier = 1
                part.CanCollide = false
            else
                if originalProperties[part] then
                    part.Transparency = originalProperties[part].Transparency
                    part.LocalTransparencyModifier = originalProperties[part].LocalTransparencyModifier
                else
                    part.Transparency = 0
                    part.LocalTransparencyModifier = 0
                end
                part.CanCollide = true
            end
        end
    end
    
    for _, accessory in ipairs(char:GetChildren()) do
        if accessory:IsA("Accessory") and accessory:FindFirstChild("Handle") then
            if ativo then
                accessory.Handle.Transparency = 1
                accessory.Handle.LocalTransparencyModifier = 1
            else
                accessory.Handle.Transparency = 0
                accessory.Handle.LocalTransparencyModifier = 0
            end
        end
    end
    
    local clothing = char:FindFirstChild("Shirt")
    if clothing then
        clothing.Transparency = ativo and 1 or 0
    end
    local pants = char:FindFirstChild("Pants")
    if pants then
        pants.Transparency = ativo and 1 or 0
    end
    
    print(string.format("[SHADOW HUB] Invisibilidade REAL: %s", ativo and "ATIVA 🔴" or "DESATIVADA"))
    
    if ativo then
        LocalPlayer:Chat("🌑 INVISÍVEL - Ninguém me vê!")
    else
        LocalPlayer:Chat("🌑 VISÍVEL NOVAMENTE!")
    end
end

local function toggleInvisibilidade()
    invisivel = not invisivel
    aplicarInvisibilidadeReal(invisivel)
end

-- ==================== HITBOX EXTENDIDA (50 BLOCOS) ====================
local hitboxAtivo = false
local hitboxConnection = nil
local ultimoAtaque = 0

local function toggleHitbox()
    hitboxAtivo = not hitboxAtivo
    
    if hitboxAtivo then
        hitboxConnection = RunService.Heartbeat:Connect(function()
            local char = LocalPlayer.Character
            if not char then return end
            
            local rootPart = char:FindFirstChild("HumanoidRootPart")
            if not rootPart then return end
            
            local now = tick()
            if now - ultimoAtaque < 0.2 then return end
            
            for _, player in ipairs(Players:GetPlayers()) do
                if player ~= LocalPlayer then
                    local targetChar = player.Character
                    if targetChar and targetChar:FindFirstChild("Humanoid") then
                        local targetRoot = targetChar:FindFirstChild("HumanoidRootPart") or targetChar:FindFirstChild("Torso")
                        if targetRoot then
                            local distancia = (rootPart.Position - targetRoot.Position).Magnitude
                            
                            if distancia <= 50 then
                                local targetHumanoid = targetChar:FindFirstChild("Humanoid")
                                if targetHumanoid and targetHumanoid.Health > 0 then
                                    targetHumanoid:TakeDamage(10)
                                    ultimoAtaque = now
                                    
                                    local hitEffect = Instance.new("Part")
                                    hitEffect.Size = Vector3.new(1, 1, 1)
                                    hitEffect.Position = targetRoot.Position
                                    hitEffect.Anchored = true
                                    hitEffect.CanCollide = false
                                    hitEffect.BrickColor = BrickColor.new("Bright violet")
                                    hitEffect.Material = Enum.Material.Neon
                                    hitEffect.Parent = workspace
                                    game:GetService("Debris"):AddItem(hitEffect, 0.2)
                                    
                                    print(string.format("[SHADOW HUB] HIT! %s | Distância: %.1f", player.Name, distancia))
                                end
                            end
                        end
                    end
                end
            end
        end)
        print("[SHADOW HUB] HITBOX ATIVA - 50 blocos 💥")
    else
        if hitboxConnection then
            hitboxConnection:Disconnect()
            hitboxConnection = nil
        end
        print("[SHADOW HUB] HITBOX DESATIVADA")
    end
end

-- ==================== ESP COLORIDO ====================
local espAtivo = false
local espHighlights = {}

local function criarESP(alvo, cor)
    if not alvo or not alvo.Parent then return end
    if espHighlights[alvo] then
        espHighlights[alvo]:Destroy()
    end
    local highlight = Instance.new("Highlight")
    highlight.Parent = alvo
    highlight.FillColor = cor
    highlight.FillTransparency = 0.4
    highlight.OutlineColor = Color3.fromRGB(200, 150, 255)
    highlight.OutlineTransparency = 0.2
    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    espHighlights[alvo] = highlight
end

local function removerTodosESP()
    for alvo, highlight in pairs(espHighlights) do
        pcall(function() highlight:Destroy() end)
    end
    espHighlights = {}
end

local function toggleESP()
    espAtivo = not espAtivo
    
    if espAtivo then
        removerTodosESP()
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                criarESP(player.Character, Color3.fromRGB(140, 80, 200))
            end
        end
        
        Players.PlayerAdded:Connect(function(player)
            player.CharacterAdded:Connect(function(char)
                if espAtivo then
                    criarESP(char, Color3.fromRGB(140, 80, 200))
                end
            end)
        end)
        
        print("[SHADOW HUB] ESP ATIVO 🔍")
    else
        removerTodosESP()
        print("[SHADOW HUB] ESP DESATIVADO")
    end
end

-- ==================== RESETAR PERSONAGEM ====================
local function resetarPersonagem()
    local char = LocalPlayer.Character
    if char then
        char:BreakJoints()
    end
end

-- ==================== CRIAR BOTÕES DO MENU ====================
createButton("👻 INVISIBILIDADE REAL", 100, Color3.fromRGB(35, 25, 50), function()
    toggleInvisibilidade()
end)

createButton("💥 HITBOX 50 BLOCOS", 160, Color3.fromRGB(35, 25, 50), function()
    toggleHitbox()
end)

createButton("🔍 ESP COLORIDO", 220, Color3.fromRGB(35, 25, 50), function()
    toggleESP()
end)

createButton("🔄 RESETAR", 280, Color3.fromRGB(55, 35, 65), function()
    resetarPersonagem()
end)

createButton("❌ FECHAR", 390, Color3.fromRGB(65, 40, 70), function()
    mainFrame.Visible = false
end)

-- ==================== CONTROLE DO MENU ====================
local menuAberto = false

floatingBtn.MouseButton1Click:Connect(function()
    menuAberto = not menuAberto
    mainFrame.Visible = menuAberto
    local rotacao = menuAberto and 45 or 0
    TweenService:Create(floatingBtn, TweenInfo.new(0.25), {Rotation = rotacao}):Play()
end)

-- Arrastar botão flutuante
local dragging = false
local dragStart
local startPos

floatingBtn.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = floatingBtn.Position
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        floatingBtn.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

-- Reaplicar invisibilidade ao renascer
LocalPlayer.CharacterAdded:Connect(function(newChar)
    character = newChar
    humanoid = newChar:WaitForChild("Humanoid")
    task.wait(0.5)
    if invisivel then
        aplicarInvisibilidadeReal(true)
    end
end)
