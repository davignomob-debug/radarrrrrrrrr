local Players = game:GetService("Players")
local CoreGui = game:GetService("CoreGui")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

local function GetSafeGui()
    if gethui then return gethui() end
    return CoreGui or LocalPlayer:WaitForChild("PlayerGui")
end

if GetSafeGui():FindFirstChild("SniperUltra_V8") then
    GetSafeGui():FindFirstChild("SniperUltra_V8"):Destroy()
end

local sg = Instance.new("ScreenGui", GetSafeGui())
sg.Name = "SniperUltra_V8"

local Main = Instance.new("Frame", sg)
Main.Size = UDim2.fromOffset(220, 100)
Main.Position = UDim2.new(0.5, -110, 0.5, -50)
Main.BackgroundColor3 = Color3.fromRGB(30, 0, 0) -- Vermelho Alerta
Instance.new("UICorner", Main)

local FarmBtn = Instance.new("TextButton", Main)
FarmBtn.Size = UDim2.new(0.9, 0, 0.7, 0)
FarmBtn.Position = UDim2.new(0.05, 0, 0.15, 0)
FarmBtn.Text = "SNIPER ATIVAR"
FarmBtn.BackgroundColor3 = Color3.fromRGB(50, 0, 0)
FarmBtn.TextColor3 = Color3.new(1, 1, 1)
FarmBtn.Font = Enum.Font.GothamBold
Instance.new("UICorner", FarmBtn)

_G.SniperActive = false

FarmBtn.MouseButton1Click:Connect(function()
    _G.SniperActive = not _G.SniperActive
    FarmBtn.Text = _G.SniperActive and "BUSCANDO..." or "SNIPER ATIVAR"
    FarmBtn.BackgroundColor3 = _G.SniperActive and Color3.fromRGB(200, 50, 50) or Color3.fromRGB(50, 0, 0)
end)

task.spawn(function()
    while true do
        task.wait(0.05) -- Um pouco mais lento para não travar, mas ainda rápido
        if _G.SniperActive then
            local success, err = pcall(function()
                local char = LocalPlayer.Character
                local hrp = char and char:FindFirstChild("HumanoidRootPart")
                if not hrp then return end

                -- Procura por botões de interação no mapa todo
                for _, v in pairs(game:GetDescendants()) do
                    if not _G.SniperActive then break end
                    
                    if v:IsA("ProximityPrompt") then
                        -- Pega o nome para conferir
                        local n = tostring(v.Parent.Name):lower()
                        
                        -- FILTRO AMPLO (Tenta pegar qualquer coisa que pareça Pipi, Kiwi ou Common)
                        if n:find("pipi") or n:find("kiwi") or n:find("com") or n:find("potat") then
                            
                            v.HoldDuration = 0
                            local target = v.Parent:IsA("BasePart") and v.Parent or v.Parent:FindFirstChildWhichIsA("BasePart")
                            
                            if target then
                                -- TELEPORTE
                                hrp.CFrame = target.CFrame
                                task.wait(0.1)
                                fireproximityprompt(v)
                                
                                -- DESLIGA NA HORA
                                _G.SniperActive = false
                                FarmBtn.Text = "PEGOU! (OFF)"
                                break
                            end
                        end
                    end
                end
            end)
        end
    end
end)

-- Fechar e Arrastar (padrão)
local Close = Instance.new("TextButton", Main)
Close.Size = UDim2.fromOffset(20, 20); Close.Position = UDim2.new(1, -22, 0, 2); Close.Text = "X"; Close.TextColor3 = Color3.new(1,0,0); Close.BackgroundTransparency = 1; Close.MouseButton1Click:Connect(function() _G.SniperActive = false; sg:Destroy() end)
local dragging, dragInput, dragStart, startPos; Main.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true; dragStart = input.Position; startPos = Main.Position end end); UserInputService.InputChanged:Connect(function(input) if input == dragInput and dragging then local delta = input.Position - dragStart; Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y) end end); Main.InputChanged:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseMovement then dragInput = input end end); UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end end)
