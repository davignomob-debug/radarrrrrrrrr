local Players = game:GetService("Players")
local ProximityPromptService = game:GetService("ProximityPromptService")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

-- // LISTA DE BRAINROTS
local Brainrots = {
    "Nenhum",
    "Tim Cheese", "Lirililarila", "Fluri Flura", "Cacto", "Hipopotamo", "Pipi Potato", "Tric Trac", "Barabum", "Burbaloni", "Loliloli",
    "Boneca", "Ambalabu", "Trippi Troppi", "Svinina", "Bombardino", "Bambini", "Crostini", "Avacodini", "Guffo", "Bandito", "Bobrito", "Tatatata", "Sahur"
}

local AlvoSelecionado = "nenhum"
local FarmAtivo = false

-- // ANTI-AFK
for _, v in pairs(getconnections(LocalPlayer.Idled)) do v:Disable() end

-- // UI (SIMPLIFICADA E FUNCIONAL)
local sg = Instance.new("ScreenGui", (gethui and gethui()) or game:GetService("CoreGui"))
local main = Instance.new("Frame", sg)
main.Size = UDim2.fromOffset(250, 230)
main.Position = UDim2.new(0.5, -125, 0.4, 0)
main.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
main.Active = true
main.Draggable = true
Instance.new("UICorner", main)

local title = Instance.new("TextLabel", main)
title.Size = UDim2.new(1, 0, 0, 30)
title.Text = "BRAINROT GOD-FARM"
title.TextColor3 = Color3.new(1, 1, 1)
title.BackgroundTransparency = 1

local toggle = Instance.new("TextButton", main)
toggle.Size = UDim2.new(0.9, 0, 0, 40)
toggle.Position = UDim2.new(0.05, 0, 0.15, 0)
toggle.Text = "STATUS: OFF"
toggle.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
toggle.TextColor3 = Color3.new(1, 1, 1)
Instance.new("UICorner", toggle)

local scroll = Instance.new("ScrollingFrame", main)
scroll.Size = UDim2.new(0.9, 0, 0.55, 0)
scroll.Position = UDim2.new(0.05, 0, 0.38, 0)
scroll.CanvasSize = UDim2.new(0, 0, 0, #Brainrots * 22)
scroll.BackgroundColor3 = Color3.fromRGB(15, 15, 18)
scroll.ScrollBarThickness = 2
Instance.new("UIListLayout", scroll)

for _, name in ipairs(Brainrots) do
    local b = Instance.new("TextButton", scroll)
    b.Size = UDim2.new(1, 0, 0, 20)
    b.Text = name
    b.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
    b.TextColor3 = Color3.new(0.7, 0.7, 0.7)
    b.BorderSizePixel = 0
    b.MouseButton1Click:Connect(function()
        AlvoSelecionado = name:lower()
        title.Text = "ALVO: " .. name:upper()
        for _, btn in pairs(scroll:GetChildren()) do if btn:IsA("TextButton") then btn.TextColor3 = Color3.new(0.7, 0.7, 0.7) end end
        b.TextColor3 = Color3.new(0, 1, 0.5)
    end)
end

toggle.MouseButton1Click:Connect(function()
    FarmAtivo = not FarmAtivo
    toggle.Text = FarmAtivo and "FARMANDO..." or "STATUS: OFF"
    toggle.BackgroundColor3 = FarmAtivo and Color3.fromRGB(0, 120, 60) or Color3.fromRGB(40, 40, 45)
end)

-- // LÓGICA DE TELEPORTE GLOBAL (FIX)
-- Função para interagir
local function capturar(prompt, part)
    local char = LocalPlayer.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if hrp then
        -- Teleporte forçado
        hrp.Velocity = Vector3.new(0,0,0)
        hrp.CFrame = part.CFrame * CFrame.new(0, 3, 0)
        
        -- Bypass de distância no momento do teleporte
        prompt.MaxActivationDistance = 999999
        prompt.RequiresLineOfSight = false
        
        task.wait(0.15) -- Delay essencial para carregar o item
        fireproximityprompt(prompt)
    end
end

-- Scanner de alta prioridade
RunService.Heartbeat:Connect(function()
    if not FarmAtivo or AlvoSelecionado == "nenhum" then return end
    
    -- Procuramos por ProximityPrompts no mapa inteiro de forma otimizada
    for _, prompt in ipairs(ProximityPromptService:GetProximityPrompts()) do
        local parent = prompt.Parent
        if parent then
            local nomeObj = parent.Name:lower()
            local textoObj = (prompt.ObjectText or ""):lower()
            
            -- Se o nome ou o texto bater com o selecionado
            if nomeObj:find(AlvoSelecionado) or textoObj:find(AlvoSelecionado) then
                local part = parent:IsA("BasePart") and parent or parent:FindFirstChildWhichIsA("BasePart")
                if part then
                    capturar(prompt, part)
                    task.wait(0.5) -- Espera o item sumir
                end
            end
        end
    end
end)
