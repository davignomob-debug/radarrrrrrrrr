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

local AlvoSelecionado = "nenhum"
local FarmAtivo = false

-- // INTERFACE ARRASTÁVEL
local function CreateUI()
    local sg = Instance.new("ScreenGui", (gethui and gethui()) or game:GetService("CoreGui"))
    sg.Name = "AutoFarm_GodMode_V12"

    local Main = Instance.new("Frame", sg)
    Main.Size = UDim2.fromOffset(250, 220)
    Main.Position = UDim2.new(0.5, -125, 0.4, 0)
    Main.BackgroundColor3 = Color3.fromRGB(10, 10, 15)
    Main.BorderSizePixel = 0
    Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 10)
    
    local stroke = Instance.new("UIStroke", Main)
    stroke.Color = Color3.fromRGB(255, 0, 100)
    stroke.Thickness = 2

    local Title = Instance.new("TextLabel", Main)
    Title.Size = UDim2.new(1, 0, 0, 35)
    Title.Text = "AUTO FARM (MAPA TODO)"
    Title.TextColor3 = Color3.new(1, 1, 1)
    Title.BackgroundTransparency = 1
    Title.Font = Enum.Font.GothamBold
    Title.TextSize = 14

    local Toggle = Instance.new("TextButton", Main)
    Toggle.Size = UDim2.new(0.9, 0, 0, 40)
    Toggle.Position = UDim2.new(0.05, 0, 0.2, 0)
    Toggle.Text = "STATUS: OFF"
    Toggle.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    Toggle.TextColor3 = Color3.new(1, 1, 1)
    Toggle.Font = Enum.Font.GothamBold
    Instance.new("UICorner", Toggle)

    local ListFrame = Instance.new("ScrollingFrame", Main)
    ListFrame.Size = UDim2.new(0.9, 0, 0.45, 0)
    ListFrame.Position = UDim2.new(0.05, 0, 0.45, 0)
    ListFrame.CanvasSize = UDim2.new(0, 0, 0, (#Brainrots * 25))
    ListFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    ListFrame.ScrollBarThickness = 2
    Instance.new("UICorner", ListFrame)

    local Layout = Instance.new("UIListLayout", ListFrame)

    for i, name in ipairs(Brainrots) do
        local b = Instance.new("TextButton", ListFrame)
        b.Size = UDim2.new(1, 0, 0, 25)
        b.Text = name
        b.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
        b.TextColor3 = Color3.new(0.6, 0.6, 0.6)
        b.Font = Enum.Font.Gotham
        b.BorderSizePixel = 0
        
        b.MouseButton1Click:Connect(function()
            AlvoSelecionado = name:lower()
            Title.Text = "ALVO: " .. name:upper()
            for _, v in pairs(ListFrame:GetChildren()) do
                if v:IsA("TextButton") then v.TextColor3 = Color3.new(0.6, 0.6, 0.6) end
            end
            b.TextColor3 = Color3.fromRGB(255, 0, 100)
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
    ToggleBtn.Text = FarmAtivo and "FARM ATIVADO" or "STATUS: OFF"
    ToggleBtn.BackgroundColor3 = FarmAtivo and Color3.fromRGB(150, 0, 50) or Color3.fromRGB(30, 30, 30)
end)

-- // MODIFICAÇÃO PARA MAPA TODO (INFINITE RANGE)
-- Esta função força o botão a aparecer mesmo se você estiver longe
local function BypassRange(prompt)
    prompt.MaxActivationDistance = math.huge -- Distância infinita
    prompt.RequiresLineOfSight = false -- Pega através de paredes
end

-- Monitora todos os botões do jogo
game:GetService("RunService").Stepped:Connect(function()
    if not FarmAtivo or AlvoSelecionado == "nenhum" then return end

    for _, prompt in ipairs(ProximityPromptService:GetProximityPrompts()) do
        if not FarmAtivo then break end
        
        local item = prompt.Parent
        if item then
            local nomeObj = item.Name:lower()
            local textoPrompt = (prompt.ObjectText or ""):lower()

            if nomeObj:find(AlvoSelecionado) or textoPrompt:find(AlvoSelecionado) then
                -- Aplica o Bypass de distância
                BypassRange(prompt)
                
                pcall(function()
                    local char = LocalPlayer.Character
                    local hrp = char:FindFirstChild("HumanoidRootPart")
                    local targetPart = item:IsA("BasePart") and item or item:FindFirstChildWhichIsA("BasePart")

                    if hrp and targetPart then
                        -- Teleporte e Coleta
                        hrp.CFrame = targetPart.CFrame * CFrame.new(0, 3, 0)
                        
                        task.wait(0.1)
                        prompt.HoldDuration = 0
                        fireproximityprompt(prompt)
                        
                        -- Espera um pouco para não dar spam no mesmo item
                        task.wait(0.4)
                    end
                end)
            end
        end
    end
end)
