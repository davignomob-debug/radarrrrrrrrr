local Players = game:GetService("Players")
local CoreGui = game:GetService("CoreGui")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

-- Função para colocar na CoreGui ou PlayerGui
local function GetSafeGui()
    if gethui then return gethui() end
    return CoreGui or LocalPlayer:WaitForChild("PlayerGui")
end

-- Limpeza de versões anteriores
local old = GetSafeGui():FindFirstChild("SimplePanel_V1")
if old then old:Destroy() end

local sg = Instance.new("ScreenGui")
sg.Name = "SimplePanel_V1"
sg.ResetOnSpawn = false
sg.Parent = GetSafeGui()

-- Frame Principal (Pequeno e centralizado)
local Main = Instance.new("Frame")
Main.Size = UDim2.fromOffset(250, 150)
Main.Position = UDim2.new(0.5, -125, 0.5, -75)
Main.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
Main.BorderSizePixel = 0
Main.Parent = sg

local MainCorner = Instance.new("UICorner", Main)
MainCorner.CornerRadius = UDim.new(0, 12)

local MainStroke = Instance.new("UIStroke", Main)
MainStroke.Color = Color3.fromRGB(0, 255, 255)
MainStroke.Thickness = 2

-- Botão Único
local ActionBtn = Instance.new("TextButton")
ActionBtn.Size = UDim2.new(0.8, 0, 0.4, 0)
ActionBtn.Position = UDim2.new(0.1, 0, 0.3, 0)
ActionBtn.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
ActionBtn.Text = "EXECUTAR"
ActionBtn.TextColor3 = Color3.fromRGB(0, 255, 255)
ActionBtn.Font = Enum.Font.GothamBold
ActionBtn.TextSize = 18
ActionBtn.Parent = Main

local BtnCorner = Instance.new("UICorner", ActionBtn)
BtnCorner.CornerRadius = UDim.new(0, 8)

-- Efeito de Clique
ActionBtn.MouseButton1Click:Connect(function()
    print("Botão acionado!")
    -- COLOQUE SEU COMANDO AQUI ABAIXO:
    
end)

-- Botão Fechar (X no topo)
local Close = Instance.new("TextButton")
Close.Size = UDim2.fromOffset(25, 25)
Close.Position = UDim2.new(1, -30, 0, 5)
Close.BackgroundTransparency = 1
Close.Text = "X"
Close.TextColor3 = Color3.fromRGB(255, 50, 50)
Close.Font = Enum.Font.GothamBold
Close.TextSize = 16
Close.Parent = Main
Close.MouseButton1Click:Connect(function() sg:Destroy() end)

-- Sistema de Arrastar (Draggable)
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
