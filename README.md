local Players = game:GetService("Players")
local ProximityPromptService = game:GetService("ProximityPromptService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

-- // LISTA COMPLETA
local Brainrots = {
    "Nenhum",
    "Tim Cheese", "Lirililarila", "Fluri Flura", "Cacto", "Hipopotamo", "Pipi Potato", "Tric Trac", "Barabum", "Burbaloni", "Loliloli",
    "Boneca", "Ambalabu", "Trippi Troppi", "Svinina", "Bombardino", "Bambini", "Crostini", "Avacodini", "Guffo", "Bandito", "Bobrito", "Tatatata", "Sahur"
}

local AlvoSelecionado = "nenhum"
local FarmAtivo = false

-- // ANTI-AFK (Para você poder dormir)
local VirtualUser = game:GetService("VirtualUser")
LocalPlayer.Idled:Connect(function()
    VirtualUser:CaptureController()
    VirtualUser:ClickButton2(Vector2.new())
end)

-- // UI (Omitida parte visual repetida para focar na lógica, mantenha a anterior)
local function CreateUI()
    local sg = Instance.new("ScreenGui", (gethui and gethui()) or game:GetService("CoreGui"))
    local Main = Instance.new("Frame", sg)
    Main.Size = UDim2.fromOffset(250, 220)
    Main.Position = UDim2.new(0.5, -125, 0.4, 0)
    Main.BackgroundColor3 = Color3.fromRGB(10, 10, 15)
    Instance.new("UICorner", Main)
    
    local Title = Instance.new("TextLabel", Main)
    Title.Size = UDim2.new(1, 0, 0, 35)
    Title.Text = "GOD FARM (MAPA TODO)"
    Title.TextColor3 = Color3.new(1, 1, 1)
    Title.BackgroundTransparency = 1

    local Toggle = Instance.new("TextButton", Main)
    Toggle.Size = UDim2.new(0.9, 0, 0, 40)
    Toggle.Position = UDim2.new(0.05, 0, 0.2, 0)
    Toggle.Text = "STATUS: OFF"
    Instance.new("UICorner", Toggle)

    local ListFrame = Instance.new("ScrollingFrame", Main)
    ListFrame.Size = UDim2.new(0.9, 0, 0.45, 0)
    ListFrame.Position = UDim2.new(0.05, 0, 0.45, 0)
    ListFrame.CanvasSize = UDim2.new(0, 0, 0, (#Brainrots * 25))
    Instance.new("UIListLayout", ListFrame)

    for i, name in ipairs(Brainrots) do
        local b = Instance.new("TextButton", ListFrame)
        b.Size = UDim2.new(1, 0, 0, 25)
        b.Text = name
        b.MouseButton1Click:Connect(function()
            AlvoSelecionado = name:lower()
            Title.Text = "ALVO: " .. name:upper()
        end)
    end

    return Toggle
end

local ToggleBtn = CreateUI()
ToggleBtn.MouseButton1Click:Connect(function()
    FarmAtivo = not FarmAtivo
    ToggleBtn.Text = FarmAtivo and "FARM ATIVADO" or "STATUS: OFF"
end)

-- // LÓGICA DE TRACKING GLOBAL
RunService.Heartbeat:Connect(function()
    if not FarmAtivo or AlvoSelecionado == "nenhum" then return end

    -- Varre TUDO no Workspace (Ignora StreamingEnabled)
    for _, obj in ipairs(game.Workspace:GetDescendants()) do
        if not FarmAtivo then break end

        -- Se o objeto tem o nome do alvo
        if obj.Name:lower():find(AlvoSelecionado) then
            local prompt = obj:FindFirstChildWhichIsA("ProximityPrompt", true) or obj.Parent:FindFirstChildWhichIsA("ProximityPrompt", true)
            
            if prompt then
                pcall(function()
                    local char = LocalPlayer.Character
                    local hrp = char:FindFirstChild("HumanoidRootPart")
                    
                    -- Se achou a parte física do item
                    local targetPart = obj:IsA("BasePart") and obj or obj:FindFirstChildWhichIsA("BasePart") or obj.Parent:FindFirstChildWhichIsA("BasePart")
                    
                    if hrp and targetPart then
                        -- 1. TELEPORTE AGRESSIVO (Força o carregamento do item)
                        hrp.CFrame = targetPart.CFrame * CFrame.new(0, 3, 0)
                        
                        -- 2. BYPASS DE DISTÂNCIA
                        prompt.MaxActivationDistance = math.huge
                        prompt.RequiresLineOfSight = false
                        
                        -- 3. COLETA
                        task.wait(0.15) -- Delay crucial para o StreamingEnabled carregar o objeto
                        fireproximityprompt(prompt)
                        
                        -- 4. ESPERA O ITEM SUMIR PARA NÃO BUGAR
                        task.wait(0.5)
                    end
                end)
            end
        end
    end
end)
