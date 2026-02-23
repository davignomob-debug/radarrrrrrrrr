local Players = game:GetService("Players")
local CoreGui = game:GetService("CoreGui")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

local function GetSafeGui()
    if gethui then return gethui() end
    return CoreGui or LocalPlayer:WaitForChild("PlayerGui")
end

-- Limpeza de UI antiga
if GetSafeGui():FindFirstChild("PipiCommonSniper") then
    GetSafeGui():FindFirstChild("PipiCommonSniper"):Destroy()
end

local sg = Instance.new("ScreenGui")
sg.Name = "PipiCommonSniper"
sg.ResetOnSpawn = false
sg.Parent = GetSafeGui()

-- [[ UI MINIMALISTA ]] --
local Main = Instance.new("Frame", sg)
Main.Size = UDim2.fromOffset(250, 120)
Main.Position = UDim2.new(0.5, -125, 0.5, -60)
Main.BackgroundColor3 = Color3.fromRGB(20, 30, 20)
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 10)
local stroke = Instance.new("UIStroke", Main)
stroke.Color = Color3.fromRGB(150, 255, 100)
stroke.Thickness = 2

-- [[ BOTÃO ÚNICO ]] --
local FarmBtn = Instance.new("TextButton", Main)
FarmBtn.Size = UDim2.new(0.85, 0, 0, 60)
FarmBtn.Position = UDim2.new(0.075, 0, 0.3, 0)
FarmBtn.BackgroundColor3 = Color3.fromRGB(30, 50, 30)
FarmBtn.Text = "SNIPER: OFF"
FarmBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
FarmBtn.Font = Enum.Font.GothamBold
FarmBtn.TextSize = 14
Instance.new("UICorner", FarmBtn)

-- [[ LÓGICA DE CONTROLE ]] --
_G.Hunting = false

local function stopFarm()
    _G.Hunting = false
    FarmBtn.Text = "SNIPER: OFF"
    FarmBtn.BackgroundColor3 = Color3.fromRGB(30, 50, 30)
end

FarmBtn.MouseButton1Click:Connect(function()
    _G.Hunting = not _G.Hunting
    FarmBtn.Text = _G.Hunting and "BUSCANDO PIPI/COMUM..." or "SNIPER: OFF"
    FarmBtn.BackgroundColor3 = _G.Hunting and Color3.fromRGB(50, 100, 50) or Color3.fromRGB(30, 50, 30)
end)

-- Loop de busca (Igual ao que funcionou antes)
task.spawn(function()
    while true do 
        task.wait(0.01) -- Velocidade para não perder o item
        if _G.Hunting then
            pcall(function()
                local char = LocalPlayer.Character
                local hrp = char and char:FindFirstChild("HumanoidRootPart")
                
                if hrp then
                    -- Varre o jogo em busca do item
                    for _, v in pairs(game:GetDescendants()) do
                        if not _G.Hunting then break end

                        if v:IsA("ProximityPrompt") then
                            local itemName = tostring(v.Parent.Name):lower()
                            
                            -- FILTRO ESPECÍFICO: Pipi, Kiwi ou Comum
                            if itemName:find("pipi") or itemName:find("kiwi") or itemName:find("comum") or itemName:find("common") then
                                
                                v.HoldDuration = 0
                                local targetPart = v.Parent:IsA("BasePart") and v.Parent or v.Parent:FindFirstChildWhichIsA("BasePart")
                                
                                if targetPart then
                                    -- Teleporte e Coleta
                                    hrp.CFrame = targetPart.CFrame
                                    task.wait(0.05)
                                    fireproximityprompt(v)
                                    
                                    -- Para tudo após 1 captura
                                    stopFarm()
                                    break
                                end
                            end
                        end
                    end
                end
            end)
        end
    end
end)

-- Botão Fechar
local Close = Instance.new("TextButton", Main)
Close.Size = UDim2.fromOffset(25, 25); Close.Position = UDim2.new(1, -30, 0, 5)
Close.BackgroundTransparency = 1; Close.Text = "X"; Close.TextColor3 = Color3.fromRGB(255, 80, 80); Close.Font = Enum.Font.GothamBold
Close.MouseButton1Click:Connect(function() _G.Hunting = false; sg:Destroy() end)

-- Sistema de Arrastar (Drag)
local dragging, dragInput, dragStart, startPos
Main.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true; dragStart = input.Position; startPos = Main.Position end end)
UserInputService.InputChanged:Connect(function(input) if input == dragInput and dragging then local delta = input.Position - dragStart; Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y) end end)
Main.InputChanged:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseMovement then dragInput = input end end)
UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end end)
