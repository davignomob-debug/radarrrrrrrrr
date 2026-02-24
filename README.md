local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

-- // LISTA COMPLETA DE BRAINROTS
local Brainrots = {
    "Tim Cheese", "LiriliLarila", "Fluri Flura", "Cacto Hipopotamo",
    "Pipi Potato", "Tric Trac Barabum", "Burbaloni Loliloli", "Boneca Ambalabu",
    "Trippi Troppi", "Svinina Bombardino", "Bambini Crostini", "Avocadini Guffo",
    "Bandito Bobrito", "Tatatata Sahur",
    "Gangster Footera", "Tung Tung Sahur", "Cappuccino Assassino", "Pipi Kiwi",
    "Orangutini Ananassini", "Talpa Di Fero", "Espresso Signora",
    "Brr Brr Patapim", "Rhino Toasterino", "Brr Bicus Dicus",
    "Strawberrelli Flamingelli", "Bananito Delfinito", "Balerina Capucina", "Glorbo Fruttodrillo",
    "Blueberrinni Octopusini", "Chimpanzini Bananini", "Bombardiro Crocodilo", "Elefanto Cocofanto",
    "Bombombini Gusini", "Pandaccini Bananini", "Chef Crabracadabra",
    "Gorillo Watermelondrillo", "Frigo Camelo", "Girafa Celestre",
    "Ganganzelli Trulala", "Tigroligre Frutonni",
    "Tralaledon", "Esok", "La Vaca", "garama and madundung", Strawberry",
    "Capitano Clash Warnini", "Meowl"
}

local AlvosSelecionados = {}
local FarmAtivo = false

local function CreateUI()
    local sg = Instance.new("ScreenGui", (gethui and gethui()) or game:GetService("CoreGui"))
    sg.Name = "AutoFarm_Infinite_V11"

    local Main = Instance.new("Frame", sg)
    Main.Size = UDim2.fromOffset(320, 320)
    Main.Position = UDim2.new(0.5, -160, 0.4, 0)
    Main.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
    Main.BorderSizePixel = 0
    Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 10)

    local stroke = Instance.new("UIStroke", Main)
    stroke.Color = Color3.fromRGB(0, 255, 127)
    stroke.Thickness = 2

    -- // TÍTULO
    local Title = Instance.new("TextLabel", Main)
    Title.Size = UDim2.new(1, -40, 0, 45)
    Title.Text = "AUTO FARM INFINITO"
    Title.TextColor3 = Color3.new(1, 1, 1)
    Title.BackgroundTransparency = 1
    Title.Font = Enum.Font.GothamBold
    Title.TextSize = 18

    -- // BOTÃO FECHAR
    local CloseBtn = Instance.new("TextButton", Main)
    CloseBtn.Size = UDim2.fromOffset(30, 30)
    CloseBtn.Position = UDim2.new(1, -35, 0, 7)
    CloseBtn.Text = "X"
    CloseBtn.TextColor3 = Color3.new(1, 1, 1)
    CloseBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    CloseBtn.Font = Enum.Font.GothamBold
    CloseBtn.TextSize = 16
    CloseBtn.BorderSizePixel = 0
    Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(0, 6)

    CloseBtn.MouseEnter:Connect(function()
        CloseBtn.BackgroundColor3 = Color3.fromRGB(180, 30, 30)
    end)
    CloseBtn.MouseLeave:Connect(function()
        CloseBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    end)
    CloseBtn.MouseButton1Click:Connect(function()
        sg:Destroy()
        FarmAtivo = false
    end)

    -- // BOTÃO TOGGLE
    local Toggle = Instance.new("TextButton", Main)
    Toggle.Size = UDim2.new(0.9, 0, 0, 48)
    Toggle.Position = UDim2.new(0.05, 0, 0.17, 0)
    Toggle.Text = "AUTO FARM: OFF"
    Toggle.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    Toggle.TextColor3 = Color3.new(1, 1, 1)
    Toggle.Font = Enum.Font.GothamBold
    Toggle.TextSize = 17
    Instance.new("UICorner", Toggle)

    -- // LISTA
    local ListFrame = Instance.new("ScrollingFrame", Main)
    ListFrame.Size = UDim2.new(0.9, 0, 0.55, 0)
    ListFrame.Position = UDim2.new(0.05, 0, 0.4, 0)
    ListFrame.CanvasSize = UDim2.new(0, 0, 0, (#Brainrots * 34))
    ListFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    ListFrame.ScrollBarThickness = 4
    Instance.new("UICorner", ListFrame)
    Instance.new("UIListLayout", ListFrame)

    for i, name in ipairs(Brainrots) do
        local b = Instance.new("TextButton", ListFrame)
        b.Size = UDim2.new(1, 0, 0, 34)
        b.Text = name
        b.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        b.TextColor3 = Color3.new(0.7, 0.7, 0.7)
        b.Font = Enum.Font.Gotham
        b.TextSize = 15
        b.BorderSizePixel = 0

        b.MouseButton1Click:Connect(function()
            local lower = name:lower()
            if AlvosSelecionados[lower] then
                AlvosSelecionados[lower] = nil
                b.TextColor3 = Color3.new(0.7, 0.7, 0.7)
                b.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
            else
                AlvosSelecionados[lower] = true
                b.TextColor3 = Color3.fromRGB(0, 255, 127)
                b.BackgroundColor3 = Color3.fromRGB(20, 50, 35)
            end

            local count = 0
            for _ in pairs(AlvosSelecionados) do count = count + 1 end
            if count == 0 then
                Title.Text = "AUTO FARM INFINITO"
            elseif count == 1 then
                Title.Text = "1 SELECIONADO"
            else
                Title.Text = count .. " SELECIONADOS"
            end
        end)
    end

    -- // ARRASTAR
    local dragging, dragInput, dragStart, startPos
    Main.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true; dragStart = input.Position; startPos = Main.Position
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            local delta = input.Position - dragStart
            Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
    Main.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement then dragInput = input end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
    end)

    return Toggle
end

local ToggleBtn = CreateUI()

ToggleBtn.MouseButton1Click:Connect(function()
    FarmAtivo = not FarmAtivo
    ToggleBtn.Text = FarmAtivo and "FARMANDO... (ON)" or "AUTO FARM: OFF"
    ToggleBtn.BackgroundColor3 = FarmAtivo and Color3.fromRGB(0, 100, 50) or Color3.fromRGB(35, 35, 35)
end)

-- // BUSCA PROMPT
local function FindPromptDoAlvo()
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("ProximityPrompt") then
            local nomeObj = (obj.Parent and obj.Parent.Name or ""):lower()
            local textoPrompt = (obj.ObjectText or ""):lower()
            for alvo, _ in pairs(AlvosSelecionados) do
                if nomeObj:find(alvo) or textoPrompt:find(alvo) then
                    return obj
                end
            end
        end
    end
    return nil
end

-- // LOOP PRINCIPAL
task.spawn(function()
    while true do
        task.wait(0.1)
        if not FarmAtivo or next(AlvosSelecionados) == nil then continue end

        local prompt = FindPromptDoAlvo()
        if not prompt then continue end

        local item = prompt.Parent
        local targetPart = item:IsA("BasePart") and item or item:FindFirstChildWhichIsA("BasePart")
        if not targetPart then continue end

        local char = LocalPlayer.Character
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        if not hrp then continue end

        pcall(function()
            hrp.CFrame = targetPart.CFrame * CFrame.new(0, 2, 0)
            task.wait(0.05)
            prompt.HoldDuration = 0
            fireproximityprompt(prompt)
            task.wait(0.2)
        end)
    end
end)
