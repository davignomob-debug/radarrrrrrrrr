local Players = game:GetService("Players")
local ProximityPromptService = game:GetService("ProximityPromptService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

-- // LISTA DE BRAINROTS (CONFORME SOLICITADO)
local Brainrots = {
    "Nenhum",
    "Tim Cheese", "Lirililarila", "Fluri Flura", "Cacto Hipopotamo", "Pipi Potato", "Tric Trac Barabum", "Burbaloni Loliloli", -- Communs
    "Boneca Ambalabu", "Trippi Troppi", "Svinina Bombardino", "Bambini Crostini", "Avacodini Guffo", "Bandito Bobrito", "Tatatata Sahur" -- Uncommons
}

local AlvoSelecionado = "Nenhum"
local SniperAtivo = false

-- // CRIAR UI ARRASTÁVEL
local function CreateUI()
    local sg = Instance.new("ScreenGui", (gethui and gethui()) or game:GetService("CoreGui"))
    sg.Name = "BrainrotSelector_V10"

    local Main = Instance.new("Frame", sg)
    Main.Size = UDim2.fromOffset(250, 200)
    Main.Position = UDim2.new(0.5, -125, 0.4, 0)
    Main.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    Main.BorderSizePixel = 0
    Instance.new("UICorner", Main)
    local stroke = Instance.new("UIStroke", Main)
    stroke.Color = Color3.fromRGB(100, 100, 255)

    local Title = Instance.new("TextLabel", Main)
    Title.Size = UDim2.new(1, 0, 0, 30)
    Title.Text = "BRAINROT SNIPER"
    Title.TextColor3 = Color3.new(1, 1, 1)
    Title.BackgroundTransparency = 1
    Title.Font = Enum.Font.GothamBold

    -- Botão Ativar
    local Toggle = Instance.new("TextButton", Main)
    Toggle.Size = UDim2.new(0.9, 0, 0, 40)
    Toggle.Position = UDim2.new(0.05, 0, 0.2, 0)
    Toggle.Text = "STATUS: OFF"
    Toggle.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    Toggle.TextColor3 = Color3.new(1, 1, 1)
    Instance.new("UICorner", Toggle)

    -- Lista Arrastável (ScrollingFrame)
    local ListFrame = Instance.new("ScrollingFrame", Main)
    ListFrame.Size = UDim2.new(0.9, 0, 0.5, 0)
    ListFrame.Position = UDim2.new(0.05, 0, 0.45, 0)
    ListFrame.CanvasSize = UDim2.new(0, 0, 0, (#Brainrots * 25))
    ListFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    ListFrame.ScrollBarThickness = 4

    local Layout = Instance.new("UIListLayout", ListFrame)
    Layout.SortOrder = Enum.SortOrder.LayoutOrder

    -- Criar botões da lista
    for i, name in ipairs(Brainrots) do
        local b = Instance.new("TextButton", ListFrame)
        b.Size = UDim2.new(1, 0, 0, 25)
        b.Text = name
        b.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        b.TextColor3 = Color3.new(0.8, 0.8, 0.8)
        b.BorderSizePixel = 0
        
        b.MouseButton1Click:Connect(function()
            AlvoSelecionado = name:lower()
            Title.Text = "ALVO: " .. name:upper()
            for _, v in pairs(ListFrame:GetChildren()) do
                if v:IsA("TextButton") then v.TextColor3 = Color3.new(0.8, 0.8, 0.8) end
            end
            b.TextColor3 = Color3.fromRGB(0, 255, 150)
        end)
    end

    -- Lógica de Arrastar
    local dragging, dragInput, dragStart, startPos
    Main.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true; dragStart = input.Position; startPos = Main.Position end end)
    UserInputService.InputChanged:Connect(function(input) if input == dragInput and dragging then local delta = input.Position - dragStart; Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y) end end)
    Main.InputChanged:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseMovement then dragInput = input end end)
    UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end end)

    return Toggle
end

local ToggleBtn = CreateUI()

ToggleBtn.MouseButton1Click:Connect(function()
    if AlvoSelecionado == "nenhum" then 
        ToggleBtn.Text = "ESCOLHA UM ITEM!" 
        task.wait(1)
        ToggleBtn.Text = "STATUS: OFF"
        return 
    end
    SniperAtivo = not SniperAtivo
    ToggleBtn.Text = SniperAtivo and "BUSCANDO..." or "STATUS: OFF"
    ToggleBtn.BackgroundColor3 = SniperAtivo and Color3.fromRGB(0, 150, 100) or Color3.fromRGB(40, 40, 40)
end)

-- // LÓGICA DE SNIPER (BASEADA EM EVENTO - SEM LAG)
ProximityPromptService.PromptShown:Connect(function(prompt)
    if not SniperAtivo or AlvoSelecionado == "nenhum" then return end

    local item = prompt.Parent
    if not item then return end

    local nomeObj = item.Name:lower()
    local textoPrompt = (prompt.ObjectText or ""):lower()

    -- Verifica se o item que apareceu é o que você selecionou na lista
    if nomeObj:find(AlvoSelecionado) or textoPrompt:find(AlvoSelecionado) then
        SniperAtivo = false -- Desliga após achar
        ToggleBtn.Text = "COLETADO!"
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)

        pcall(function()
            local char = LocalPlayer.Character
            local hrp = char:FindFirstChild("HumanoidRootPart")
            local targetPart = item:IsA("BasePart") and item or item:FindFirstChildWhichIsA("BasePart")

            if hrp and targetPart then
                -- Teleporte e Coleta
                hrp.CFrame = targetPart.CFrame * CFrame.new(0, 2, 0)
                task.wait(0.05)
                prompt.HoldDuration = 0
                fireproximityprompt(prompt)
            end
        end)
    end
end)
