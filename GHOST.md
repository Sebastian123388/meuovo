-- GHOST FPS KILLER - versão clean interface
-- Painel compacto, translúcido, sem sombra pesada.
-- Glow discreto nos botões + animação suave.

local Players = game:GetService("Players")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

-- Remove acessórios/roupas
local function removeAllAccessoriesFromCharacter()
    local character = player.Character
    if not character then return end
    for _, item in ipairs(character:GetChildren()) do
        if item:IsA("Accessory")
            or item:IsA("LayeredClothing")
            or item:IsA("Shirt")
            or item:IsA("ShirtGraphic")
            or item:IsA("Pants")
            or item:IsA("BodyColors")
            or item:IsA("CharacterMesh") then
            pcall(function() item:Destroy() end)
        end
    end
end
player.CharacterAdded:Connect(function()
    task.wait(0.2)
    removeAllAccessoriesFromCharacter()
end)
if player.Character then
    task.defer(removeAllAccessoriesFromCharacter)
end

-- FPS Killer
local FPSDevourer = {}
do
    FPSDevourer.running = false
    local TOOL_NAME = "Tung Bat"
    local function equipTungBat()
        local character = player.Character
        local backpack = player:FindFirstChild("Backpack")
        if not character or not backpack then return false end
        local tool = backpack:FindFirstChild(TOOL_NAME)
        if tool then tool.Parent = character return true end
        return false
    end
    local function unequipTungBat()
        local character = player.Character
        local backpack = player:FindFirstChild("Backpack")
        if not character or not backpack then return false end
        local tool = character:FindFirstChild(TOOL_NAME)
        if tool then tool.Parent = backpack return true end
        return false
    end

    function FPSDevourer:Start()
        if FPSDevourer.running then return end
        FPSDevourer.running = true
        FPSDevourer._stop = false
        task.spawn(function()
            while FPSDevourer.running and not FPSDevourer._stop do
                equipTungBat()
                task.wait(0.035)
                unequipTungBat()
                task.wait(0.035)
            end
        end)
    end
    function FPSDevourer:Stop()
        FPSDevourer.running = false
        FPSDevourer._stop = true
        unequipTungBat()
    end
    player.CharacterAdded:Connect(function()
        FPSDevourer.running = false
        FPSDevourer._stop = true
    end)
end

-- Remove painel antigo
local old = playerGui:FindFirstChild("GhostFPSKillerPanel")
if old then old:Destroy() end

-- Painel UI
local gui = Instance.new("ScreenGui")
gui.Name = "GhostFPSKillerPanel"
gui.ResetOnSpawn = false
gui.Parent = playerGui

local main = Instance.new("Frame", gui)
main.Name = "MainPanel"
main.Size = UDim2.new(0, 180, 0, 90)
main.Position = UDim2.new(1, -200, 0, 20)
main.BackgroundColor3 = Color3.fromRGB(30,30,30)
main.BackgroundTransparency = 0.15
main.BorderSizePixel = 0
main.Active = true
Instance.new("UICorner", main).CornerRadius = UDim.new(0, 12)

-- Glow em volta (bem leve)
local glow = Instance.new("UIStroke", main)
glow.Thickness = 2
glow.Color = Color3.fromRGB(0, 255, 140)
glow.Transparency = 0.4
glow.ApplyStrokeMode = Enum.ApplyStrokeMode.Border

-- Animação de entrada (fade + scale)
main.BackgroundTransparency = 1
main.Size = UDim2.new(0, 0, 0, 0)
TweenService:Create(main, TweenInfo.new(0.6, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), {
    Size = UDim2.new(0, 180, 0, 90),
    BackgroundTransparency = 0.15
}):Play()

-- Drag
do
    local dragging, dragInput, dragStart, startPos
    main.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = main.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then dragging = false end
            end)
        end
    end)
    main.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and input == dragInput then
            local delta = input.Position - dragStart
            main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
end

-- Título
local title = Instance.new("TextLabel", main)
title.Name = "Title"
title.Size = UDim2.new(1, 0, 0, 28)
title.Text = "GHOST FPS KILLER"
title.Font = Enum.Font.GothamBold
title.TextSize = 16
title.BackgroundTransparency = 1
title.TextColor3 = Color3.fromRGB(255,255,255)

-- Botão Toggle
local btn = Instance.new("TextButton", main)
btn.Size = UDim2.new(0.9, 0, 0, 32)
btn.Position = UDim2.new(0.05, 0, 0, 40)
btn.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
btn.Text = "Ativar"
btn.Font = Enum.Font.Gotham
btn.TextSize = 15
btn.TextColor3 = Color3.fromRGB(230,230,230)
btn.AutoButtonColor = false
Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)

-- Glow discreto no botão
local btnStroke = Instance.new("UIStroke", btn)
btnStroke.Thickness = 1.5
btnStroke.Color = Color3.fromRGB(0,255,140)
btnStroke.Transparency = 0.35

-- Hover efeito
btn.MouseEnter:Connect(function()
    TweenService:Create(btn, TweenInfo.new(0.25, Enum.EasingStyle.Sine), {BackgroundColor3 = Color3.fromRGB(35,35,35)}):Play()
end)
btn.MouseLeave:Connect(function()
    TweenService:Create(btn, TweenInfo.new(0.25, Enum.EasingStyle.Sine), {BackgroundColor3 = Color3.fromRGB(25,25,25)}):Play()
end)

-- Lógica do botão
local state = false
btn.MouseButton1Click:Connect(function()
    state = not state
    if state then
        btn.Text = "Rodando..."
        FPSDevourer:Start()
        TweenService:Create(btnStroke, TweenInfo.new(0.3), {Transparency = 0}):Play()
    else
        btn.Text = "Ativar"
        FPSDevourer:Stop()
        TweenService:Create(btnStroke, TweenInfo.new(0.3), {Transparency = 0.35}):Play()
    end
end)

-- Resetar no respawn
player.CharacterAdded:Connect(function()
    state = false
    btn.Text = "Ativar"
    FPSDevourer:Stop()
end)
