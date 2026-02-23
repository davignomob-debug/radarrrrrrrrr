local Players = game:GetService("Players")
local CoreGui = game:GetService("CoreGui")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

local function GetSafeGui()
    if gethui then return gethui() end
    return CoreGui or LocalPlayer:WaitForChild("PlayerGui")
end

-- Deleta qualquer painel antigo para não bugar
if GetSafeGui():FindFirstChild("SniperOnly_V5") then
    GetSafeGui():FindFirstChild("SniperOnly_V5"):Destroy()
end

local sg = Instance.new("ScreenGui")
sg.Name = "SniperOnly_V5"
sg.ResetOnSpawn = false
sg.Parent = GetSafeGui()

-- [[ FRAME PRINCIPAL - BEM PEQUENO ]] --
local Main = Instance.new("Frame", sg)
Main.Size = UDim2.fromOffset(240, 110) -- Tamanho reduzido
Main.Position = UDim2.new(0.5, -120, 0.5, -55)
Main.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 10)
local stroke = Instance.new("UIStroke", Main)
stroke.Color = Color3.fromRGB(255, 255, 255)
stroke.Thickness = 1.5

-- TÍTULO
local Title = Instance.new("TextLabel", Main)
Title.Size = UDim2.new(1, 0, 0, 35)
Title.BackgroundTransparency = 1
Title.Text = "SNIPER COMUM"
Title.TextColor3 = Color3.fromRGB(200, 200, 200)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 13

-- [[ O ÚNICO BOTÃO ]] --
local FarmBtn = Instance.new("TextButton", Main)
FarmBtn.Size = UDim2.new(0.85, 0, 0, 50)
FarmBtn.Position = UDim2.new(0.075, 0, 0.38, 0)
FarmBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
FarmBtn.Text = "BUSCAR: OFF"
FarmBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
FarmBtn.Font = Enum.Font.GothamBold
FarmBtn.TextSize = 14
Instance.new("UICorner", FarmBtn).CornerRadius = UDim.new(0, 8)

-- LÓGICA DE CONTROLE
_G.CommonFarm = false

local function stopFarm()
    _G.CommonFarm = false
    FarmBtn.Text = "BUSCAR: OFF"
    FarmBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
end

FarmBtn.MouseButton1Click:Connect(function()
    _G.CommonFarm = not _G.CommonFarm
    if _G.CommonFarm then
        FarmBtn.Text = "BUSCANDO..."
        FarmBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 70)
    else
        stopFarm()
    end
end)

-- LOOP DE BUSCA
task.spawn(function()
    while true do 
        task.wait(0.01)
        if _G.CommonFarm then
            pcall(function()
                local char = LocalPlayer.Character
                local hrp = char and char:FindFirstChild("HumanoidRootPart")
                
                if hrp then
                    for _, v in pairs(game:GetDescendants()) do
                        if not _G.CommonFarm then break end

                        if v:IsA("ProximityPrompt") then
                            local itemName = tostring(v.Parent.Name):lower()
                            -- Filtra apenas itens comuns
                            if itemName:find("common") or itemName:find("comum") then
                                
                                v.HoldDuration = 0
                                local itemPos = v.Parent:IsA("BasePart") and v.Parent.Position or v.Parent:FindFirstChildWhichIsA("BasePart").Position
                                
                                if itemPos then
                                    hrp.CFrame = CFrame.new(itemPos)
                                    task.wait(0.05)
                                    fireproximityprompt(v)
                                    
                                    -- Para após pegar 1
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

-- FECHAR
local Close = Instance.new("TextButton", Main)
Close.Size = UDim2.fromOffset(20, 20); Close.Position = UDim2.new(1, -25, 0, 5)
Close.BackgroundTransparency = 1; Close.Text = "X"; Close.TextColor3 = Color3.fromRGB(255, 80, 80); Close.Font = Enum.Font.GothamBold
Close.MouseButton1Click:Connect(function() 
    _G.CommonFarm = false 
    sg:Destroy() 
end)

-- ARRASTAR
local dragging, dragInput, dragStart, startPos
Main.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true; dragStart = input.Position; startPos = Main.Position end end)
UserInputService.InputChanged:Connect(function(input) if input == dragInput and dragging then local delta = input.Position - dragStart; Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y) end end)
Main.InputChanged:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseMovement then dragInput = input end end)
UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end end)
