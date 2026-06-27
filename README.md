--[[
    HUNTER Launcher v4.9 - CORREÇÃO DE MIRA E ANTI-AIM
    - Corrigido problema de mirar em objetos no chão
    - Adicionada opção Anti-Aim (desvia de headshots)
    - Círculo do FOV grande com borda BRANCA e sem o ponto central vermelho
    - Mecanismo de mira configurado para ignorar alvos fora do raio do círculo
    - Suavidade funcionando (Fraco/Médio/Forte)
    - ESP apenas com destaque visual (sem nomes)
    - Persistência TOTAL após morte/mudança de mapa
    - Aba de Créditos embutida internamente (sem abrir nova janela)
]]

local player = game:GetService("Players").LocalPlayer
local gui = Instance.new("ScreenGui")
gui.Name = "HunterUI"
gui.Parent = player:WaitForChild("PlayerGui")
gui.ResetOnSpawn = false

-- ====== VARIÁVEIS GLOBAIS ======
local aimbotActive = false
local espActive = false
local fovActive = false
local antiAimActive = false
local targetPart = "Head"
local aimSmoothness = 0.3
local fovRadius = 200
local fovCircle = nil
local isMinimized = false
local currentTarget = nil  
local heartbeatConnection = nil
local espObjects = {}
local onCreditsPage = false

-- ====== JANELA PRINCIPAL ======
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 280, 0, 380) -- Aumentado para caber o novo botão
mainFrame.Position = UDim2.new(0.5, -140, 0.5, -190)
mainFrame.BackgroundColor3 = Color3.new(0.1, 0.02, 0.02)
mainFrame.BorderColor3 = Color3.new(0.6, 0, 0)
mainFrame.BorderSizePixel = 2
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = gui

-- ====== BARRA DE TÍTULO ======
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 35)
titleBar.BackgroundColor3 = Color3.new(0.15, 0.03, 0.03)
titleBar.BorderSizePixel = 0
titleBar.Parent = mainFrame

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -90, 1, 0) 
titleLabel.Position = UDim2.new(0, 10, 0, 0)
titleLabel.Text = "⚡ HUNTER v4.9"
titleLabel.TextColor3 = Color3.new(0.9, 0.1, 0.1)
titleLabel.TextSize = 18
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.BackgroundTransparency = 1
titleLabel.Parent = titleBar

local creditsToggleBtn = Instance.new("TextButton")
creditsToggleBtn.Size = UDim2.new(0, 30, 0, 30)
creditsToggleBtn.Position = UDim2.new(1, -70, 0, 2)
creditsToggleBtn.Text = "👥"
creditsToggleBtn.TextColor3 = Color3.new(1, 1, 1)
creditsToggleBtn.TextSize = 16
creditsToggleBtn.Font = Enum.Font.GothamBold
creditsToggleBtn.BackgroundColor3 = Color3.new(0.2, 0.05, 0.05)
creditsToggleBtn.BorderColor3 = Color3.new(0.3, 0, 0)
creditsToggleBtn.BorderSizePixel = 1
creditsToggleBtn.Parent = titleBar

local minimizeBtn = Instance.new("TextButton")
minimizeBtn.Size = UDim2.new(0, 30, 0, 30)
minimizeBtn.Position = UDim2.new(1, -35, 0, 2)
minimizeBtn.Text = "—"
minimizeBtn.TextColor3 = Color3.new(1, 1, 1)
minimizeBtn.TextSize = 20
minimizeBtn.Font = Enum.Font.GothamBold
minimizeBtn.BackgroundColor3 = Color3.new(0.2, 0.05, 0.05)
minimizeBtn.BorderColor3 = Color3.new(0.3, 0, 0)
minimizeBtn.BorderSizePixel = 1
minimizeBtn.Parent = titleBar

-- Painel original das funções (Cheats)
local content = Instance.new("Frame")
content.Size = UDim2.new(1, -20, 1, -55)
content.Position = UDim2.new(0, 10, 0, 45)
content.BackgroundTransparency = 1
content.Parent = mainFrame

-- Painel interno exclusivo dos Créditos
local creditsContent = Instance.new("Frame")
creditsContent.Size = UDim2.new(1, -20, 1, -55)
creditsContent.Position = UDim2.new(0, 10, 0, 45)
creditsContent.BackgroundTransparency = 1
creditsContent.Visible = false
creditsContent.Parent = mainFrame

-- ====== FUNÇÃO PARA CRIAR BOTÕES ======
local function createButton(parent, text, yPos)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0, 250, 0, 35)
    frame.Position = UDim2.new(0, 0, 0, yPos)
    frame.BackgroundTransparency = 1
    frame.Parent = parent
    
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0, 160, 0, 35)
    label.Position = UDim2.new(0, 0, 0, 0)
    label.Text = text
    label.TextColor3 = Color3.new(1, 1, 1)
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.TextSize = 14
    label.Font = Enum.Font.GothamBold
    label.BackgroundTransparency = 1
    label.Parent = frame
    
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 70, 0, 30)
    btn.Position = UDim2.new(0, 180, 0, 2)
    btn.Text = "OFF"
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.TextSize = 14
    btn.Font = Enum.Font.GothamBold
    btn.BackgroundColor3 = Color3.new(0.5, 0, 0)
    btn.BorderColor3 = Color3.new(0.3, 0, 0)
    btn.BorderSizePixel = 1
    btn.Parent = frame
    
    return btn, label
end

-- ====== CRIAR BOTÕES ======
local yPos = 0

-- 1. Seletor de alvo
local targetFrame = Instance.new("Frame")
targetFrame.Size = UDim2.new(0, 250, 0, 35)
targetFrame.Position = UDim2.new(0, 0, 0, yPos)
targetFrame.BackgroundTransparency = 1
targetFrame.Parent = content

local targetLabel = Instance.new("TextLabel")
targetLabel.Size = UDim2.new(0, 160, 0, 35)
targetLabel.Text = "🎯 Alvo: Cabeça"
targetLabel.TextColor3 = Color3.new(1, 1, 0)
targetLabel.TextSize = 14
targetLabel.Font = Enum.Font.GothamBold
targetLabel.TextXAlignment = Enum.TextXAlignment.Left
targetLabel.BackgroundTransparency = 1
targetLabel.Parent = targetFrame

local targetBtn = Instance.new("TextButton")
targetBtn.Size = UDim2.new(0, 70, 0, 30)
targetBtn.Position = UDim2.new(0, 180, 0, 2)
targetBtn.Text = "Trocar"
targetBtn.TextColor3 = Color3.new(1, 1, 1)
targetBtn.TextSize = 12
targetBtn.Font = Enum.Font.GothamBold
targetBtn.BackgroundColor3 = Color3.new(0.2, 0.1, 0.1)
targetBtn.BorderColor3 = Color3.new(0.3, 0, 0)
targetBtn.BorderSizePixel = 1
targetBtn.Parent = targetFrame

targetBtn.MouseButton1Click:Connect(function()
    if targetPart == "Head" then
        targetPart = "Torso"
        targetLabel.Text = "🎯 Alvo: Torso"
    else
        targetPart = "Head"
        targetLabel.Text = "🎯 Alvo: Cabeça"
    end
end)

yPos = yPos + 40

-- 2. Aimbot
local aimbotBtn, aimbotLabel = createButton(content, "🎯 Aimbot", yPos)
yPos = yPos + 40

-- 3. ESP
local espBtn, espLabel = createButton(content, "📡 ESP", yPos)
yPos = yPos + 40

-- 4. FOV Circle
local fovBtn, fovLabel = createButton(content, "⭕ FOV Circle", yPos)
yPos = yPos + 40

-- 5. Anti-Aim (NOVO)
local antiAimBtn, antiAimLabel = createButton(content, "🔄 Anti-Aim", yPos)
yPos = yPos + 40

-- 6. Suavidade do Aimbot
local smoothFrame = Instance.new("Frame")
smoothFrame.Size = UDim2.new(0, 250, 0, 35)
smoothFrame.Position = UDim2.new(0, 0, 0, yPos)
smoothFrame.BackgroundTransparency = 1
smoothFrame.Parent = content

local smoothLabel = Instance.new("TextLabel")
smoothLabel.Size = UDim2.new(0, 160, 0, 35)
smoothLabel.Text = "⚙️ Suavidade: Média"
smoothLabel.TextColor3 = Color3.new(1, 1, 0)
smoothLabel.TextSize = 13
smoothLabel.Font = Enum.Font.GothamBold
smoothLabel.TextXAlignment = Enum.TextXAlignment.Left
smoothLabel.BackgroundTransparency = 1
smoothLabel.Parent = smoothFrame

local smoothBtn = Instance.new("TextButton")
smoothBtn.Size = UDim2.new(0, 70, 0, 30)
smoothBtn.Position = UDim2.new(0, 180, 0, 2)
smoothBtn.Text = "Mudar"
smoothBtn.TextColor3 = Color3.new(1, 1, 1)
smoothBtn.TextSize = 12
smoothBtn.Font = Enum.Font.GothamBold
smoothBtn.BackgroundColor3 = Color3.new(0.2, 0.1, 0.1)
smoothBtn.BorderColor3 = Color3.new(0.3, 0, 0)
smoothBtn.BorderSizePixel = 1
smoothBtn.Parent = smoothFrame

yPos = yPos + 40

-- 7. Tamanho do FOV
local fovSizeFrame = Instance.new("Frame")
fovSizeFrame.Size = UDim2.new(0, 250, 0, 35)
fovSizeFrame.Position = UDim2.new(0, 0, 0, yPos)
fovSizeFrame.BackgroundTransparency = 1
fovSizeFrame.Parent = content

local fovSizeLabel = Instance.new("TextLabel")
fovSizeLabel.Size = UDim2.new(0, 160, 0, 35)
fovSizeLabel.Text = "📐 Tamanho FOV: 200px"
fovSizeLabel.TextColor3 = Color3.new(1, 1, 0)
fovSizeLabel.TextSize = 13
fovSizeLabel.Font = Enum.Font.GothamBold
fovSizeLabel.TextXAlignment = Enum.TextXAlignment.Left
fovSizeLabel.BackgroundTransparency = 1
fovSizeLabel.Parent = fovSizeFrame

local fovSizeBtn = Instance.new("TextButton")
fovSizeBtn.Size = UDim2.new(0, 70, 0, 30)
fovSizeBtn.Position = UDim2.new(0, 180, 0, 2)
fovSizeBtn.Text = "Mudar"
fovSizeBtn.TextColor3 = Color3.new(1, 1, 1)
fovSizeBtn.TextSize = 12
fovSizeBtn.Font = Enum.Font.GothamBold
fovSizeBtn.BackgroundColor3 = Color3.new(0.2, 0.1, 0.1)
fovSizeBtn.BorderColor3 = Color3.new(0.3, 0, 0)
fovSizeBtn.BorderSizePixel = 1
fovSizeBtn.Parent = fovSizeFrame

local smoothNames = {"Forte", "Média", "Fraco"}
local smoothValues = {0.05, 0.3, 0.6}
local smoothIndex = 2

local fovSizes = {150, 200, 250, 300}
local fovNames = {"150px", "200px", "250px", "300px"}
local fovIndex = 2

smoothBtn.MouseButton1Click:Connect(function()
    smoothIndex = smoothIndex + 1
    if smoothIndex > 3 then smoothIndex = 1 end
    smoothLabel.Text = "⚙️ Suavidade: " .. smoothNames[smoothIndex]
    aimSmoothness = smoothValues[smoothIndex]
end)

fovSizeBtn.MouseButton1Click:Connect(function()
    fovIndex = fovIndex + 1
    if fovIndex > 4 then fovIndex = 1 end
    
    fovSizeLabel.Text = "📐 Tamanho FOV: " .. fovNames[fovIndex]
    fovRadius = fovSizes[fovIndex]
    
    if fovActive and fovCircle then
        fovCircle.Size = UDim2.new(0, fovRadius * 2, 0, fovRadius * 2)
        fovCircle.Position = UDim2.new(0.5, -fovRadius, 0.5, -fovRadius)
    end
end)

-- ====== DESIGN E MONTAGEM DA ABA INTERNA DE CRÉDITOS ======
local function createCreditLabel(text, yPosition, color)
    local lbl = Instance.new("TextLabel")
    lbl.Size = UDim2.new(1, 0, 0, 30)
    lbl.Position = UDim2.new(0, 5, 0, yPosition)
    lbl.Text = text
    lbl.TextColor3 = color or Color3.new(1, 1, 1)
    lbl.TextSize = 14
    lbl.Font = Enum.Font.GothamBold
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.BackgroundTransparency = 1
    lbl.Parent = creditsContent
    return lbl
end

createCreditLabel("👑 Criado por: huntereyes7", 10, Color3.new(1, 1, 1))
createCreditLabel("📸 Instagram: @loro_malzao", 45, Color3.new(1, 1, 1))
createCreditLabel("💬 Discord: https://discord.gg/V3jVNHWX2s", 80, Color3.new(0.4, 0.6, 1))

local copyDiscordBtn = Instance.new("TextButton")
copyDiscordBtn.Size = UDim2.new(1, 0, 0, 35)
copyDiscordBtn.Position = UDim2.new(0, 0, 0, 130)
copyDiscordBtn.Text = "📋 Copiar Discord"
copyDiscordBtn.TextColor3 = Color3.new(1, 1, 1)
copyDiscordBtn.TextSize = 14
copyDiscordBtn.Font = Enum.Font.GothamBold
copyDiscordBtn.BackgroundColor3 = Color3.new(0.2, 0.05, 0.05)
copyDiscordBtn.BorderColor3 = Color3.new(0.4, 0, 0)
copyDiscordBtn.BorderSizePixel = 1
copyDiscordBtn.Parent = creditsContent

copyDiscordBtn.MouseButton1Click:Connect(function()
    if setclipboard then
        setclipboard("https://discord.gg/V3jVNHWX2s")
        copyDiscordBtn.Text = "✅ Copiado!"
        task.wait(2)
        copyDiscordBtn.Text = "📋 Copiar Discord"
    else
        copyDiscordBtn.Text = "❌ Sem suporte no Executor"
    end
end)

local thankYouLabel = Instance.new("TextLabel")
thankYouLabel.Size = UDim2.new(1, 0, 0, 40)
thankYouLabel.Position = UDim2.new(0, 0, 1, -45)
thankYouLabel.Text = "❤️ Obrigado por usar meu script! Espero que você aproveite. 🚀"
thankYouLabel.TextColor3 = Color3.new(0.7, 0.7, 0.7)
thankYouLabel.TextSize = 11
thankYouLabel.Font = Enum.Font.Gotham
thankYouLabel.TextWrapped = true
thankYouLabel.BackgroundTransparency = 1
thankYouLabel.Parent = creditsContent

-- ====== FUNÇÃO DE ALTERNAR ENTRE AS ABAS ======
creditsToggleBtn.MouseButton1Click:Connect(function()
    if isMinimized then return end 
    
    onCreditsPage = not onCreditsPage
    if onCreditsPage then
        content.Visible = false          
        creditsContent.Visible = true    
        creditsToggleBtn.Text = "🎯"     
        titleLabel.Text = "👥 CRÉDITOS"
    else
        creditsContent.Visible = false   
        content.Visible = true           
        creditsToggleBtn.Text = "👥"     
        titleLabel.Text = "⚡ HUNTER v4.9"
    end
end)

-- ====== FUNÇÃO MINIMIZAR ======
minimizeBtn.MouseButton1Click:Connect(function()
    isMinimized = not isMinimized
    minimizeBtn.Text = isMinimized and "+" or "—"
    
    if isMinimized then
        content.Visible = false
        creditsContent.Visible = false
        mainFrame.Size = UDim2.new(0, 280, 0, 35)
    else
        mainFrame.Size = UDim2.new(0, 280, 0, 380)
        if onCreditsPage then
            creditsContent.Visible = true
        else
            content.Visible = true
        end
    end
end)

-- ====== MECANISMO DE TRAVA RESTRITA AO RAIO DO FOV (CORRIGIDO) ======
local function getClosestTarget()
    local camera = game:GetService("Workspace").CurrentCamera
    if not camera then return nil, nil, nil, math.huge end
    
    local viewportSize = camera.ViewportSize
    local center = Vector2.new(viewportSize.X / 2, viewportSize.Y / 2)
    
    local closestPlayer = nil
    local closestDistance = fovRadius
    local closestPart = nil
    local targetPos = nil
    
    -- Verifica se o alvo está realmente vivo e é um jogador válido
    for _, v in pairs(game:GetService("Players"):GetPlayers()) do
        if v ~= player and v.Character and v.Character:FindFirstChild("HumanoidRootPart") then
            local char = v.Character
            local humanoid = char:FindFirstChild("Humanoid")
            
            -- Verifica se o jogador está vivo
            if humanoid and humanoid.Health > 0 and humanoid.Health <= humanoid.MaxHealth then
                local target = nil
                
                -- Tenta pegar a parte específica do alvo
                if targetPart == "Head" then
                    target = char:FindFirstChild("Head")
                else
                    target = char:FindFirstChild("Torso") or char:FindFirstChild("UpperTorso")
                end
                
                -- Fallback para HumanoidRootPart se a parte específica não existir
                if not target then
                    target = char:FindFirstChild("HumanoidRootPart")
                end
                
                -- Verifica se o target existe e está na hierarquia
                if target and target.Parent and target.Parent == char then
                    local worldPos = target.Position
                    local pos, onScreen = camera:WorldToViewportPoint(worldPos)
                    
                    -- Verifica se está na tela e se a distância é válida
                    if onScreen and pos.Z > 0 then
                        local distance = (Vector2.new(pos.X, pos.Y) - center).Magnitude
                        
                        -- Verifica se o jogador está dentro do FOV
                        if distance < closestDistance then
                            closestDistance = distance
                            closestPlayer = v
                            closestPart = target
                            targetPos = worldPos
                        end
                    end
                end
            end
        end
    end
    
    return closestPlayer, closestPart, targetPos, closestDistance
end

-- ====== FUNÇÃO ANTI-AIM ======
local function toggleAntiAim(state)
    antiAimActive = state
    antiAimBtn.Text = state and "ON" or "OFF"
    antiAimBtn.BackgroundColor3 = state and Color3.new(0, 0.6, 0) or Color3.new(0.5, 0, 0)
    
    if state then
        -- Anti-aim: Fica se movendo aleatoriamente para dificultar headshots
        game:GetService("RunService").Heartbeat:Connect(function()
            if not antiAimActive then return end
            if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then return end
            
            local hrp = player.Character.HumanoidRootPart
            local randomYaw = math.rad(math.random(-30, 30))
            local randomPitch = math.rad(math.random(-10, 10))
            
            -- Aplica uma rotação aleatória sutil
            hrp.CFrame = hrp.CFrame * CFrame.Angles(randomPitch, randomYaw, 0)
        end)
    end
end

antiAimBtn.MouseButton1Click:Connect(function()
    toggleAntiAim(not antiAimActive)
end)

-- ====== FUNÇÃO ATIVADORA DO SISTEMA DE MIRA (CORRIGIDA) ======
local function toggleAimbot(state)
    aimbotActive = state
    aimbotBtn.Text = state and "ON" or "OFF"
    aimbotBtn.BackgroundColor3 = state and Color3.new(0, 0.6, 0) or Color3.new(0.5, 0, 0)
    
    if heartbeatConnection then
        heartbeatConnection:Disconnect()
        heartbeatConnection = nil
    end
    
    if state then
        heartbeatConnection = game:GetService("RunService").Heartbeat:Connect(function()
            if not aimbotActive then return end
            
            local camera = game:GetService("Workspace").CurrentCamera
            if not camera then return end
            
            local cameraPos = camera.CFrame.Position
            
            -- Obtém o alvo mais próximo
            local targetPlayer, targetPart, targetPos, distance = getClosestTarget()
            
            -- Verifica se o alvo é válido e está dentro do FOV
            if not targetPos or not targetPart or distance > fovRadius then
                currentTarget = nil
                return
            end
            
            -- Verifica se o alvo ainda está vivo (segunda verificação)
            if targetPlayer and targetPlayer.Character then
                local humanoid = targetPlayer.Character:FindFirstChild("Humanoid")
                if not humanoid or humanoid.Health <= 0 then
                    currentTarget = nil
                    return
                end
            else
                currentTarget = nil
                return
            end
            
            currentTarget = targetPlayer
            
            -- Calcula a direção para o alvo
            local direction = (targetPos - cameraPos).Unit
            local yaw = math.atan2(-direction.X, -direction.Z)
            local pitch = math.asin(direction.Y)
            
            -- Aplica a mira com suavidade
            local targetCFrame = CFrame.new(cameraPos) * CFrame.Angles(0, yaw, 0) * CFrame.Angles(pitch, 0, 0)
            local currentCFrame = camera.CFrame
            local lerpFactor = aimSmoothness
            
            if lerpFactor <= 0.1 then
                camera.CFrame = targetCFrame
            else
                camera.CFrame = currentCFrame:Lerp(targetCFrame, lerpFactor * 2)
            end
        end)
    end
end

aimbotBtn.MouseButton1Click:Connect(function()
    toggleAimbot(not aimbotActive)
end)

-- ====== ESP SIMPLES (MANTIDO) ======
local function clearESP()
    for _, obj in pairs(espObjects) do
        pcall(function() 
            if obj and obj.Parent then
                obj:Destroy() 
            end
        end)
    end
    espObjects = {}
end

local function createSimpleESP(char)
    if not char or not char:FindFirstChild("HumanoidRootPart") or char == player.Character then 
        return nil 
    end
    
    local humanoid = char:FindFirstChild("Humanoid")
    if not humanoid or humanoid.Health <= 0 then
        return nil
    end
    
    local highlight = Instance.new("Highlight")
    highlight.Name = "ESP_Highlight"
    highlight.FillColor = Color3.new(1, 0, 0)
    highlight.FillTransparency = 0.4
    highlight.OutlineColor = Color3.new(1, 1, 0)
    highlight.OutlineTransparency = 0.2
    highlight.Parent = char
    
    return highlight
end

local function toggleESP(state)
    espActive = state
    espBtn.Text = state and "ON" or "OFF"
    espBtn.BackgroundColor3 = state and Color3.new(0, 0.6, 0) or Color3.new(0.5, 0, 0)
    
    clearESP()
    
    if state then
        for _, v in pairs(game:GetService("Players"):GetPlayers()) do
            if v ~= player and v.Character then
                local esp = createSimpleESP(v.Character)
                if esp then
                    table.insert(espObjects, esp)
                end
            end
        end
        
        game
