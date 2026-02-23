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

if GetSafeGui():FindFirstChild("PipiPotatoHunter_V1") then
    GetSafeGui():FindFirstChild("PipiPotatoHunter_V1"):Destroy()
end

local sg = Instance.new("ScreenGui")
sg.Name = "PipiPotatoHunter_V1"
sg.ResetOnSpawn = false
sg.Parent = GetSafeGui()

-- [[ UI PIPI HUNTER ]] --
local Main = Instance.new("Frame", sg)
Main.Size = UDim2.fromOffset(280, 220)
Main.Position = UDim2.new(0.5, -140, 0.5, -110)
Main.BackgroundColor3 = Color3.fromRGB(30, 20, 10) -- Cor puxada pro marrom/batata
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 12)
local stroke = Instance.new("UIStroke", Main)
stroke.Color = Color3.fromRGB(255, 200, 50) -- Dourado/Amarelo
stroke.Thickness = 2.5

local Title = Instance.new("TextLabel", Main)
Title.Size = UDim2.new(1, 0, 0, 45)
Title.BackgroundTransparency = 1
Title.Text = "PIPI POTATO SEEKER"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 16

-- [[ BOTÃO 1: SERVER HOP ]] --
local HopBtn = Instance.new("TextButton", Main)
HopBtn.Size = UDim2.new(0.85, 0, 0, 50)
HopBtn.Position = UDim2.new(0.075, 0, 0.25, 0)
HopBtn.BackgroundColor3 = Color3.fromRGB(60, 40, 20)
HopBtn.Text = "PROCURAR EM OUTRO MAPA"
HopBtn.TextColor3 = Color3.fromRGB(255, 200, 50)
HopBtn.Font = Enum.Font.GothamBold
HopBtn.TextSize = 13
Instance.new("UICorner", HopBtn).CornerRadius = UDim.new(0, 8)

-- [[ BOTÃO 2: AUTO SNIPE ]] --
local FarmBtn = Instance.new("TextButton", Main)
FarmBtn.Size = UDim2.new(0.85, 0, 0, 50)
FarmBtn.Position = UDim2.new(0.075, 0, 0.55, 0)
FarmBtn.BackgroundColor3 = Color3.fromRGB(40, 60, 20)
FarmBtn.Text = "SNIPER PIPI: OFF"
FarmBtn.TextColor3 = Color3.fromRGB(150, 255, 100)
FarmBtn.Font = Enum.Font.GothamBold
FarmBtn.TextSize = 14
Instance.new("UICorner", FarmBtn).CornerRadius = UDim.new(0, 8)

-- LÓGICA SERVER HOP
HopBtn.MouseButton1Click:Connect(function()
    HopBtn.Text = "BUSCANDO SERVIDOR..."
    local success, result = pcall(function()
        return HttpService:JSONDecode(game:HttpGet("https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Desc&limit=50"))
    end)
    if success and result.data then
        TeleportService:TeleportToPlaceInstance(game.PlaceId, result.data[math.random(1, #result.data)].id)
    end
end)

-- [[ LÓGICA SNIPER FOCADO ]] --
_G.PipiSniper = false
FarmBtn.MouseButton1Click:Connect(function()
    _G.PipiSniper = not _G.PipiSniper
    FarmBtn.Text = _G.PipiSniper and "SNIPER PIPI: ATIVADO" or "SNIPER PIPI: OFF"
    FarmBtn.BackgroundColor3 = _G.PipiSniper and Color3.fromRGB(60, 100, 30) or Color3.fromRGB(40, 60, 20)
end)

task.spawn(function()
    while true do task.wait(0.01) -- Velocidade máxima de detecção
        if _G.PipiSniper then
            pcall(function()
                local char = LocalPlayer.Character
                local hrp = char and char:FindFirstChild("HumanoidRootPart")
                
                if hrp then
                    for _, v in pairs(game:GetDescendants()) do
                        -- Só avança se encontrar um ProximityPrompt
                        if v:IsA("ProximityPrompt") then
                            -- FILTRO DE NOME: Verifica se o nome do item ou do pai é "Pipi Potato"
                            local parentName = v.Parent.Name:lower()
                            if string.find(parentName, "pipi") or string.find(parentName, "potato") then
                                
                                v.HoldDuration = 0 -- Coleta instantânea
                                
                                -- Pega a posição do item
                                local targetPos = v.Parent:IsA("BasePart") and v.Parent.Position or v.Parent:FindFirstChildWhichIsA("BasePart").Position
                                
                                if targetPos then
                                    -- Teleporte Ninja: vai, pega e fica lá (ou volta se você quiser)
                                    hrp.CFrame = CFrame.new(targetPos)
                                    task.wait(0.05)
                                    fireproximityprompt(v)
                                    print("Pipi Potato detectado e coletado!")
                                end
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
