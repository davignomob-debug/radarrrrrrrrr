local Players = game:GetService("Players")
local CoreGui = game:GetService("CoreGui")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

local function GetSafeGui()
    if gethui then return gethui() end
    return CoreGui or LocalPlayer:WaitForChild("PlayerGui")
end

if GetSafeGui():FindFirstChild("CommonSniper_V4") then
    GetSafeGui():FindFirstChild("CommonSniper_V4"):Destroy()
end

local sg = Instance.new("ScreenGui")
sg.Name = "CommonSniper_V4"
sg.ResetOnSpawn = false
sg.Parent = GetSafeGui()

-- [[ UI COMPACTA ]] --
local Main = Instance.new("Frame", sg)
Main.Size = UDim2.fromOffset(250, 130)
Main.Position = UDim2.new(0.5, -125, 0.5, -65)
Main.BackgroundColor3 = Color3.fromRGB(20, 25, 20)
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 12)
local stroke = Instance.new("UIStroke", Main)
stroke.Color = Color3.fromRGB(100, 255, 100)
stroke.Thickness = 2

local Title = Instance.new("TextLabel", Main)
Title.Size = UDim2.new(1, 0, 0, 40)
Title.BackgroundTransparency = 1
Title.Text = "SNIPER: 1 COMUM"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 14

-- [[ BOTÃO ÚNICO: AUTO COLETAR ]] --
local FarmBtn = Instance.new("TextButton", Main)
FarmBtn.Size = UDim2.new(0.85, 0, 0, 55)
FarmBtn.Position = UDim2.new(0.075, 0, 0.4, 0)
FarmBtn.BackgroundColor3 = Color3.fromRGB(30, 45, 30)
FarmBtn.Text = "BUSCAR COMUM: OFF"
FarmBtn.TextColor3 = Color3.fromRGB(150, 255, 150)
FarmBtn.Font = Enum.Font.GothamBold
FarmBtn.TextSize = 15
Instance.new("UICorner", FarmBtn).CornerRadius = UDim.new(0, 10)

-- [[ LÓGICA DE CONTROLE ]] --
_G.CommonFarm = false

local function stopFarm()
    _G.CommonFarm = false
    FarmBtn.Text = "BUSCAR COMUM: OFF"
    FarmBtn.BackgroundColor3 = Color3.fromRGB(30, 45, 30)
end

FarmBtn.MouseButton1Click:Connect(function()
    _G.CommonFarm = not _G.CommonFarm
    FarmBtn.Text = _G.CommonFarm and "BUSCANDO... (1)" or "BUSCAR COMUM: OFF"
    FarmBtn.BackgroundColor3 = _G.CommonFarm and Color3.fromRGB(50, 150, 50) or Color3.fromRGB(30, 45, 30)
end)

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
                                    
                                    -- Para instantaneamente após a primeira captura
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
Close.Size = UDim2.fromOffset(25, 25); Close.Position = UDim2.new(1, -30, 0, 5)
Close.BackgroundTransparency = 1; Close.Text = "X"; Close.TextColor3 = Color3.fromRGB(255, 50, 50); Close.Font = Enum.Font.GothamBold
Close.MouseButton1Click:Connect(function() 
    _G.CommonFarm = false 
    sg:Destroy() 
end)

-- DRAG
local dragging, dragInput, dragStart, startPos
Main.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true; dragStart = input.Position; startPos = Main.Position end end)
UserInputService.InputChanged:Connect(function(input) if input == dragInput and dragging then local delta = input.Position - dragStart; Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y) end end)
Main.InputChanged:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseMovement then dragInput = input end end)
UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end end)
