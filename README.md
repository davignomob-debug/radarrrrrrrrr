local Players = game:GetService("Players")
local ProximityPromptService = game:GetService("ProximityPromptService")
local LocalPlayer = Players.LocalPlayer

-- // CONFIGURAÇÃO
local ALVOS = {"pipi", "kiwi", "comum", "common"}
_G.SniperAtivo = false

-- // UI ULTRA LEVE (CORRIGIDA)
local function CreateUI()
    local sg = Instance.new("ScreenGui")
    sg.Name = "FinalSniper"
    sg.ResetOnSpawn = false
    -- Tenta colocar no CoreGui (Executores) ou PlayerGui (Roblox)
    local parent = (gethui and gethui()) or game:GetService("CoreGui") or LocalPlayer:WaitForChild("PlayerGui")
    sg.Parent = parent

    local btn = Instance.new("TextButton")
    btn.Size = UDim2.fromOffset(160, 45)
    btn.Position = UDim2.new(0.5, -80, 0.1, 0)
    btn.Text = "SNIPER: OFF"
    btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 14
    btn.Parent = sg
    
    local corner = Instance.new("UICorner", btn)
    
    return btn
end

local mainBtn = CreateUI()

-- // LOGICA DE ATIVAR/DESATIVAR
mainBtn.MouseButton1Click:Connect(function()
    _G.SniperAtivo = not _G.SniperAtivo
    mainBtn.Text = _G.SniperAtivo and "SCANNEANDO..." or "SNIPER: OFF"
    mainBtn.BackgroundColor3 = _G.SniperAtivo and Color3.fromRGB(0, 180, 100) or Color3.fromRGB(40, 40, 40)
end)

-- // EVENTO QUE DETECTA O ITEM SEM LAG
ProximityPromptService.PromptShown:Connect(function(prompt)
    if not _G.SniperAtivo then return end
    
    local char = LocalPlayer.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    -- Pega o nome do item ou o texto do botão
    local item = prompt.Parent
    local nomeObjeto = (item and item.Name or ""):lower()
    local textoBotao = (prompt.ObjectText or ""):lower()
    
    local encontrou = false
    for _, alvo in ipairs(ALVOS) do
        if nomeObjeto:find(alvo) or textoBotao:find(alvo) then
            encontrou = true
            break
        end
    end

    if encontrou then
        -- Desativa na hora para evitar compras duplas
        _G.SniperAtivo = false
        mainBtn.Text = "PEGOU! (OFF)"
        mainBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)

        -- Teleporte e Coleta Instantânea
        -- Usamos pcall para garantir que se o item sumir o jogo não trave
        pcall(function()
            local targetPos = item:IsA("BasePart") and item.CFrame or item:FindFirstChildWhichIsA("BasePart").CFrame
            
            -- Move o personagem para cima do item
            hrp.CFrame = targetPos * CFrame.new(0, 2, 0)
            
            -- Força a coleta
            task.wait(0.05) -- Pequeno delay para o servidor validar sua posição
            prompt.HoldDuration = 0
            fireproximityprompt(prompt)
        end)
    end
end)
