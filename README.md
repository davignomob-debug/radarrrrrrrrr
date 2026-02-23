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

if GetSafeGui():FindFirstChild("BrainrotTester_V3") then
    GetSafeGui():FindFirstChild("BrainrotTester_V3"):Destroy()
end

local sg = Instance.new("ScreenGui")
sg.Name = "BrainrotTester_V3"
sg.ResetOnSpawn = false
sg.Parent = GetSafeGui()

-- [[ UI DE TESTE ]] --
local Main = Instance.new("Frame", sg)
Main.Size = UDim2.fromOffset(280, 220)
Main.Position = UDim2.new(0.5, -140, 0.5, -110)
Main.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 12)
local stroke = Instance.new("UIStroke", Main)
stroke.Color = Color3.fromRGB(0, 255, 255)
stroke.Thickness = 2

local Title = Instance.new("TextLabel", Main)
Title.Size = UDim2.new(1, 0, 0, 45)
Title.BackgroundTransparency = 1
Title.Text = "TESTADOR DE ESTEIRA"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 16

-- [[ BOTÃO 1: SERVER HOP ]] --
local HopBtn = Instance.new("TextButton", Main)
HopBtn.Size = UDim2.new(0.85, 0, 0, 50)
HopBtn.Position = UDim2.new(0.075, 0, 0.25, 0)
HopBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
HopBtn.Text = "PULAR SERVIDOR"
HopBtn.TextColor3 = Color3.fromRGB(0, 255, 255)
HopBtn.Font = Enum.Font.GothamBold
HopBtn.TextSize = 14
Instance.new("UICorner", HopBtn).CornerRadius = UDim.new(0, 8)

-- [[ BOTÃO 2: AUTO COLETAR TUDO ]] --
local FarmBtn = Instance.new("TextButton", Main)
FarmBtn.Size = UDim2.new(0.85, 0, 0, 50)
FarmBtn.Position = UDim2.new(0.075, 0, 0.55, 0)
FarmBtn.BackgroundColor3 = Color3.fromRGB(30, 60, 30)
FarmBtn.Text = "AUTO COLETAR: OFF"
FarmBtn.TextColor3 = Color3.fromRGB(150, 255, 150)
FarmBtn.Font = Enum.Font.GothamBold
FarmBtn.TextSize = 14
Instance.new("UICorner", FarmBtn).CornerRadius = UDim.new(0, 8)

-- LÓGICA SERVER HOP
HopBtn.MouseButton1Click:Connect(function()
    local success, result = pcall(function()
        return HttpService:JSONDecode(game:HttpGet("https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Desc&limit=50"))
    end)
    if success and result.data then
        TeleportService:TeleportToPlaceInstance(game.PlaceId, result.data[math.random(1, #result.data)].id)
    end
end)

-- [[ LÓGICA DE TESTE DE COLETA ]] --
_G.TestFarm = false
FarmBtn.MouseButton1Click:Connect(function()
    _G.TestFarm = not _G.TestFarm
    FarmBtn.Text = _G.TestFarm and "AUTO COLETAR: ON" or "AUTO COLETAR: OFF"
    FarmBtn.BackgroundColor3 = _G.TestFarm and Color3.fromRGB(50, 100, 50) or Color3.fromRGB(30, 60, 30)
end)

task.spawn(function()
    while true do task.wait(0.05) -- Loop super rápido para não perder nada
        if _G.TestFarm then
            pcall(function()
                local char = LocalPlayer.Character
                local hrp = char and char:FindFirstChild("HumanoidRootPart")
                
                if hrp then
                    for _, v in pairs(game:GetDescendants()) do
                        -- Procura por QUALQUER ProximityPrompt ativo no mapa
                        if v:IsA("ProximityPrompt") then
                            v.HoldDuration = 0 -- Coleta instantânea
                            
                            -- Verifica se o item está perto da área da esteira (opcional, aqui pega tudo)
                            local itemPos = v.Parent:IsA("BasePart") and v.Parent.Position or v.Parent:FindFirstChildWhichIsA("BasePart").Position
                            
                            if itemPos then
                                -- Teleporta, clica e volta rápido
                                local originalPos = hrp.CFrame
                                hrp.CFrame = CFrame.new(itemPos)
                                task.wait(0.05)
                                fireproximityprompt(v)
                                -- task.wait(0.05) -- Opcional: hrp.CFrame = originalPos (para voltar)
                            end
                        end
                    end
                end
            end)
        end
    end
end)

-- FECHAR E DRAG
local Close = Instance.new("TextButton", Main)
Close.Size = UDim2.fromOffset(30, 30); Close.Position = UDim2.new(1, -35, 0, 5)
Close.BackgroundTransparency = 1; Close.Text = "X"; Close.TextColor3 = Color3.fromRGB(255, 50, 50); Close.Font = Enum.Font.GothamBold; Close.MouseButton1Click:Connect(function() sg:Destroy() end)

local dragging, dragInput, dragStart, startPos
Main.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true; dragStart = input.Position; startPos = Main.Position end end)
UserInputService.InputChanged:Connect(function(input) if input == dragInput and dragging then local delta = input.Position - dragStart; Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y) end end)
Main.InputChanged:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseMovement then dragInput = input end end)
UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end end)
