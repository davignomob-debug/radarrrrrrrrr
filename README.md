local Players = game:GetService("Players")
local ProximityPromptService = game:GetService("ProximityPromptService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

-- // LISTA COMPLETA DE BRAINROTS
local Brainrots = {
    "Nenhum",
    "Tim Cheese", "Lirililarila", "Fluri Flura", "Cacto", "Hipopotamo", "Pipi Potato", "Tric Trac", "Barabum", "Burbaloni", "Loliloli",
    "Boneca", "Ambalabu", "Trippi Troppi", "Svinina", "Bombardino", "Bambini", "Crostini", "Avacodini", "Guffo", "Bandito", "Bobrito", "Tatatata", "Sahur"
}

local AlvoSelecionado = "Nenhum"
local FarmAtivo = false

-- // INTERFACE ARRASTÁVEL
local function CreateUI()
    local sg = Instance.new("ScreenGui", (gethui and gethui()) or game:GetService("CoreGui"))
    sg.Name = "AutoFarm_Infinite_V11"

    local Main = Instance.new("Frame", sg)
    Main.Size = UDim2.fromOffset(250, 220)
    Main.Position = UDim2.new(0.5, -125, 0.4, 0)
    Main.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
    Main.BorderSizePixel = 0
    Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 10)
    
    local stroke = Instance.new("UIStroke", Main)
    stroke.Color = Color3.fromRGB(0, 255, 127)
    stroke.Thickness = 2

    local Title = Instance.new("TextLabel", Main)
    Title.Size = UDim2.new(1, 0, 0, 35)
    Title.Text = "AUTO FARM INFINITO"
    Title.TextColor3 = Color3.new(1, 1, 1)
    Title.BackgroundTransparency = 1
    Title.Font = Enum.Font.GothamBold
    Title.TextSize = 14

    local Toggle = Instance.new("TextButton", Main)
    Toggle.Size = UDim2.new(0.9, 0, 0, 40)
    Toggle.Position = UDim2.new(0.05, 0, 0.2, 0)
    Toggle.Text = "AUTO FARM: OFF"
    Toggle.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    Toggle.TextColor3 = Color3.new(1, 1, 1)
    Toggle.Font = Enum.Font.GothamBold
    Instance.new("UICorner", Toggle)

    local ListFrame = Instance.new("ScrollingFrame", Main)
    ListFrame.Size = UDim2.new(0.9, 0, 0.45, 0)
    ListFrame.Position = UDim2.new(0.05, 0, 0.45, 0)
    ListFrame.CanvasSize = UDim2.new(0, 0, 0, (#Brainrots * 25))
    ListFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    ListFrame.ScrollBarThickness = 3
    Instance.new("UICorner", ListFrame)

    local Layout = Instance.new("UIListLayout", ListFrame)

    for i, name in ipairs(Brainrots) do
        local b = Instance.new("TextButton", ListFrame)
        b.Size = UDim2.new(1, 0, 0, 25)
        b.Text = name
        b.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        b.TextColor3 = Color3.new(0.7, 0.7, 0.7)
        b.Font = Enum.Font.Gotham
        b.BorderSizePixel = 0
        
        b.MouseButton1Click:Connect(function()
            AlvoSelecionado = name:lower()
            Title.Text = "ALVO: " .. name:upper()
            for _, v in pairs(ListFrame:GetChildren()) do
                if v:IsA("TextButton") then v.TextColor3 = Color3.new(0.7, 0.7, 0.7) end
            end
            b.TextColor3 = Color3.fromRGB(0, 255, 127)
        end)
    end

    -- Arrastar UI
    local dragging, dragInput, dragStart, startPos
    Main.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true; dragStart = input.Position; startPos = Main.Position end end)
    UserInputService.InputChanged:Connect(function(input) if input == dragInput and dragging then local delta = input.Position - dragStart; Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y) end end)
    Main.InputChanged:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseMovement then dragInput = input end end)
    UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end end)

    return Toggle
end

local ToggleBtn = CreateUI()

ToggleBtn.MouseButton1Click:Connect(function()
    FarmAtivo = not FarmAtivo
    ToggleBtn.Text = FarmAtivo and "FARMANDO... (ON)" or "AUTO FARM: OFF"
    ToggleBtn.BackgroundColor3 = FarmAtivo and Color3.fromRGB(0, 100, 50) or Color3.fromRGB(35, 35, 35)
end)

-- // LÓGICA INFINITA (DORMIR E FARMAR)
ProximityPromptService.PromptShown:Connect(function(prompt)
    if not FarmAtivo or AlvoSelecionado == "nenhum" then return end

    local item = prompt.Parent
    if not item then return end

    local nomeObj = item.Name:lower()
    local textoPrompt = (prompt.ObjectText or ""):lower()

    -- Verifica se o item é o alvo
    if nomeObj:find(AlvoSelecionado) or textoPrompt:find(AlvoSelecionado) then
        pcall(function()
            local char = LocalPlayer.Character
            local hrp = char:FindFirstChild("HumanoidRootPart")
            local targetPart = item:IsA("BasePart") and item or item:FindFirstChildWhichIsA("BasePart")

            if hrp and targetPart then
                -- Teleporte e Coleta
                local originalPos = hrp.CFrame -- Salva sua posição
                hrp.CFrame = targetPart.CFrame * CFrame.new(0, 2, 0)
                
                task.wait(0.1) -- Tempo para o servidor registrar
                prompt.HoldDuration = 0
                fireproximityprompt(prompt)
                
                -- O SEGREDO PARA IR DORMIR:
                -- Ele NÃO desativa o SniperAtivo. 
                -- Ele apenas espera um pouco e você continua pronto para o próximo!
                task.wait(0.5) 
            end
        end)
    end
end)
