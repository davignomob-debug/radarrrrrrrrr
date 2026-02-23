local Players = game:GetService("Players")
local CoreGui = game:GetService("CoreGui")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

local function GetSafeGui()
    if gethui then return gethui() end
    return CoreGui or LocalPlayer:WaitForChild("PlayerGui")
end

if GetSafeGui():FindFirstChild("SniperOptimized_V6") then
    GetSafeGui():FindFirstChild("SniperOptimized_V6"):Destroy()
end

local sg = Instance.new("ScreenGui")
sg.Name = "SniperOptimized_V6"
sg.Parent = GetSafeGui()

-- [[ UI MINIMALISTA ]] --
local Main = Instance.new("Frame", sg)
Main.Size = UDim2.fromOffset(220, 100)
Main.Position = UDim2.new(0.5, -110, 0.5, -50)
Main.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 8)
local stroke = Instance.new("UIStroke", Main)
stroke.Color = Color3.fromRGB(0, 255, 150)
stroke.Thickness = 1.5

local FarmBtn = Instance.new("TextButton", Main)
FarmBtn.Size = UDim2.new(0.9, 0, 0.7, 0)
FarmBtn.Position = UDim2.new(0.05, 0, 0.15, 0)
FarmBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
FarmBtn.Text = "SNIPER LISO: OFF"
FarmBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
FarmBtn.Font = Enum.Font.GothamBold
FarmBtn.TextSize = 14
Instance.new("UICorner", FarmBtn)

-- [[ LÓGICA ULTRA OTIMIZADA ]] --
_G.Hunting = false

local function stopFarm()
    _G.Hunting = false
    FarmBtn.Text = "SNIPER LISO: OFF"
    FarmBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
end

FarmBtn.MouseButton1Click:Connect(function()
    _G.Hunting = not _G.Hunting
    FarmBtn.Text = _G.Hunting and "BUSCANDO..." or "SNIPER LISO: OFF"
    FarmBtn.BackgroundColor3 = _G.Hunting and Color3.fromRGB(0, 100, 50) or Color3.fromRGB(20, 20, 20)
end)

-- Usamos Heartbeat para rodar em sincronia com o FPS do jogo, sem travar a Render
game:GetService("RunService").Heartbeat:Connect(function()
    if not _G.Hunting then return end
    
    local char = LocalPlayer.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    -- Em vez de GetDescendants (pesado), olhamos apenas objetos próximos ou em pastas comuns
    -- O pcall evita que erros de itens sumindo travem o script
    pcall(function()
        -- Workspace costuma ter uma pasta de "Dropped" ou "Items", mas vamos focar nos ProximityPrompts ativos
        for _, v in pairs(game:GetService("ProximityService"):GetProximityPrompts()) do
            if not _G.Hunting then break end
            
            if v.Enabled and v.Parent then
                local name = v.Parent.Name:lower()
                if name:find("common") or name:find("comum") then
                    v.HoldDuration = 0
                    local targetPos = v.Parent.Position
                    
                    -- Teleporte e Coleta
                    hrp.CFrame = CFrame.new(targetPos)
                    fireproximityprompt(v)
                    
                    stopFarm()
                    break
                end
            end
        end
    end)
end)

-- FECHAR E DRAG (Mantidos simples)
local Close = Instance.new("TextButton", Main)
Close.Size = UDim2.fromOffset(20, 20); Close.Position = UDim2.new(1, -22, 0, 2); Close.BackgroundTransparency = 1; Close.Text = "X"; Close.TextColor3 = Color3.fromRGB(255, 50, 50); Close.MouseButton1Click:Connect(function() _G.Hunting = false; sg:Destroy() end)

local dragging, dragInput, dragStart, startPos
Main.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true; dragStart = input.Position; startPos = Main.Position end end)
UserInputService.InputChanged:Connect(function(input) if input == dragInput and dragging then local delta = input.Position - dragStart; Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y) end end)
Main.InputChanged:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseMovement then dragInput = input end end)
UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end end)
