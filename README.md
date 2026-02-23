local Players = game:GetService("Players")
local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")
local CoreGui = game:GetService("CoreGui")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

local function GetSafeGui()
    if gethui then return gethui() end
    return CoreGui or LocalPlayer:WaitForChild("PlayerGui")
end

if GetSafeGui():FindFirstChild("RareSniper_V1") then
    GetSafeGui():FindFirstChild("RareSniper_V1"):Destroy()
end

local sg = Instance.new("ScreenGui")
sg.Name = "RareSniper_V1"
sg.ResetOnSpawn = false
sg.Parent = GetSafeGui()

local Main = Instance.new("Frame", sg)
Main.Size = UDim2.fromOffset(280, 220)
Main.Position = UDim2.new(0.5, -140, 0.5, -110)
Main.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 12)
local stroke = Instance.new("UIStroke", Main)
stroke.Color = Color3.fromRGB(0, 150, 255) -- Azul Rare
stroke.Thickness = 2

local Title = Instance.new("TextLabel", Main)
Title.Size = UDim2.new(1, 0, 0, 45)
Title.BackgroundTransparency = 1
Title.Text = "RARE ITEM SNIPER"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 16

local HopBtn = Instance.new("TextButton", Main)
HopBtn.Size = UDim2.new(0.85, 0, 0, 50)
HopBtn.Position = UDim2.new(0.075, 0, 0.25, 0)
HopBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
HopBtn.Text = "PULAR SERVIDOR"
HopBtn.TextColor3 = Color3.fromRGB(0, 150, 255)
HopBtn.Font = Enum.Font.GothamBold
HopBtn.TextSize = 13
Instance.new("UICorner", HopBtn).CornerRadius = UDim.new(0, 8)

local FarmBtn = Instance.new("TextButton", Main)
FarmBtn.Size = UDim2.new(0.85, 0, 0, 50)
FarmBtn.Position = UDim2.new(0.075, 0, 0.55, 0)
FarmBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
FarmBtn.Text = "SNIPER 1 RARE: OFF"
FarmBtn.TextColor3 = Color3.fromRGB(200, 200, 200)
FarmBtn.Font = Enum.Font.GothamBold
FarmBtn.TextSize = 14
Instance.new("UICorner", FarmBtn).CornerRadius = UDim.new(0, 8)

-- LÓGICA DE CONTROLE
_G.SniperActive = false

FarmBtn.MouseButton1Click:Connect(function()
    _G.SniperActive = not _G.SniperActive
    
    if _G.SniperActive then
        FarmBtn.Text = "BUSCANDO RARE..."
        FarmBtn.BackgroundColor3 = Color3.fromRGB(0, 80, 150)
        FarmBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    else
        FarmBtn.Text = "SNIPER 1 RARE: OFF"
        FarmBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        FarmBtn.TextColor3 = Color3.fromRGB(200, 200, 200)
    end
end)

task.spawn(function()
    while true do
        task.wait() -- Roda o mais rápido possível
        
        -- Interrompe instantaneamente se for desligado
        if not _G.SniperActive then continue end
        
        pcall(function()
            local char = LocalPlayer.Character
            local hrp = char and char:FindFirstChild("HumanoidRootPart")
            
            if hrp then
                for _, v in pairs(game:GetDescendants()) do
                    -- Verifica se o Sniper ainda está ativo dentro do loop (parada instantânea)
                    if not _G.SniperActive then break end
                    
                    if v:IsA("ProximityPrompt") then
                        local name = v.Parent.Name:lower()
                        
                        -- Filtra por itens "Rare" (ou Pipi Potato/outros raros)
                        if string.find(name, "rare") or string.find(name, "pipi") then
                            
                            v.HoldDuration = 0
                            local targetPos = v.Parent:IsA("BasePart") and v.Parent.Position or v.Parent:FindFirstChildWhichIsA("BasePart").Position
                            
                            if targetPos then
                                -- Teleporta e compra
                                hrp.CFrame = CFrame.new(targetPos)
                                task.wait(0.05)
                                fireproximityprompt(v)
                                
                                -- DESATIVA APÓS COMPRAR 1
                                _G.SniperActive = false
                                FarmBtn.Text = "COMPRADO! (OFF)"
                                FarmBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
                                print("Rare comprado com sucesso. Sniper desligado.")
                                break
                            end
                        end
                    end
                end
            end
        end)
    end
end)

-- SERVER HOP
HopBtn.MouseButton1Click:Connect(function()
    local success, result = pcall(function()
        return HttpService:JSONDecode(game:HttpGet("https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Desc&limit=50"))
    end)
    if success and result.data then
        TeleportService:TeleportToPlaceInstance(game.PlaceId, result.data[math.random(1, #result.data)].id)
    end
end)

-- FECHAR E DRAG
local Close = Instance.new("TextButton", Main)
Close.Size = UDim2.fromOffset(30, 30); Close.Position = UDim2.new(1, -35, 0, 5); Close.BackgroundTransparency = 1; Close.Text = "X"; Close.TextColor3 = Color3.fromRGB(255, 50, 50); Close.Font = Enum.Font.GothamBold; Close.MouseButton1Click:Connect(function() _G.SniperActive = false; sg:Destroy() end)

local dragging, dragInput, dragStart, startPos
Main.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true; dragStart = input.Position; startPos = Main.Position end end)
UserInputService.InputChanged:Connect(function(input) if input == dragInput and dragging then local delta = input.Position - dragStart; Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y) end end)
Main.InputChanged:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseMovement then dragInput = input end end)
UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end end)
