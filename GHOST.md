--[[
    GHOST FPS KILLER - Painel com interface espectral.
    Estética: fundo preto/cinza fantasma, título branco com brilho azulado,
    botões ON/OFF com efeito spectral e borda pulsante.
    by LennonTheGoat
]]

local Players = game:GetService("Players")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

-- ====== DIMENSIONAMENTO =======
local SCALE = 0.75
local PANEL_WIDTH, PANEL_HEIGHT = math.floor(230*SCALE), math.floor(110*SCALE)
local PANEL_RADIUS = math.floor(16*SCALE)
local TITLE_HEIGHT = math.floor(36*SCALE)
local BTN_WIDTH = math.floor(0.9*PANEL_WIDTH)
local BTN_HEIGHT = math.floor(40*SCALE)
local BTN_RADIUS = math.floor(10*SCALE)
local BTN_FONT_SIZE = math.floor(18*SCALE)
local TITLE_FONT_SIZE = math.floor(22*SCALE)
local ICON_SIZE = math.floor(18*SCALE)
local BTN_ICON_PAD = math.floor(10*SCALE)
local BTN_Y0 = math.floor(45*SCALE)

-- Remove painel antigo
local old = playerGui:FindFirstChild("GhostFpsKillerPanel")
if old then old:Destroy() end

-- ========== PAINEL UI ==========
local gui = Instance.new("ScreenGui")
gui.Name = "GhostFpsKillerPanel"
gui.ResetOnSpawn = false
gui.Parent = playerGui

local main = Instance.new("Frame", gui)
main.Name = "MainPanel"
main.Size = UDim2.new(0, PANEL_WIDTH, 0, PANEL_HEIGHT)
main.Position = UDim2.new(1, -PANEL_WIDTH-15, 0, 15)
main.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
main.BackgroundTransparency = 0.15
main.BorderSizePixel = 0
main.Active = true
Instance.new("UICorner", main).CornerRadius = UDim.new(0, PANEL_RADIUS)

-- Gradiente espectral
local gradient = Instance.new("UIGradient", main)
gradient.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, Color3.fromRGB(10, 10, 10)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(60, 60, 70))
}
gradient.Rotation = 90

-- Borda com efeito pulsante
local uiStroke = Instance.new("UIStroke", main)
uiStroke.Thickness = 2
uiStroke.Color = Color3.fromRGB(180, 220, 255)
uiStroke.Transparency = 0.4

task.spawn(function()
    while main.Parent do
        TweenService:Create(uiStroke, TweenInfo.new(2, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {Transparency = 0.1}):Play()
        task.wait(2)
        TweenService:Create(uiStroke, TweenInfo.new(2, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {Transparency = 0.5}):Play()
        task.wait(2)
    end
end)

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

-- Título espectral
local title = Instance.new("TextLabel", main)
title.Name = "Title"
title.Size = UDim2.new(1, 0, 0, TITLE_HEIGHT)
title.Position = UDim2.new(0,0,0,0)
title.Text = "GHOST FPS KILLER"
title.Font = Enum.Font.GothamBlack
title.TextSize = TITLE_FONT_SIZE
title.BackgroundTransparency = 1
title.TextColor3 = Color3.fromRGB(220, 240, 255)
title.TextStrokeTransparency = 0.3
title.TextStrokeColor3 = Color3.fromRGB(140, 200, 255)

-- Funções UI
local function createCircleIcon(parent, y, on)
    local icon = Instance.new("Frame", parent)
    icon.Size = UDim2.new(0, ICON_SIZE, 0, ICON_SIZE)
    icon.Position = UDim2.new(0, BTN_ICON_PAD, 0, y + math.floor((BTN_HEIGHT-ICON_SIZE)/2))
    icon.BackgroundTransparency = 1
    local circle = Instance.new("ImageLabel", icon)
    circle.Size = UDim2.new(1, 0, 1, 0)
    circle.BackgroundTransparency = 1
    circle.Image = "rbxassetid://10137946418"
    circle.ImageColor3 = (on and Color3.fromRGB(140,255,200)) or Color3.fromRGB(200,80,80)
    return icon, circle
end

local function makeToggleBtn(parent, label, y, callback)
    local btn = Instance.new("TextButton", parent)
    btn.Size = UDim2.new(0, BTN_WIDTH, 0, BTN_HEIGHT)
    btn.Position = UDim2.new(0, math.floor((PANEL_WIDTH-BTN_WIDTH)/2), 0, y)
    btn.BackgroundColor3 = Color3.fromRGB(30,30,30)
    btn.BackgroundTransparency = 0.2
    btn.Text = label
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = BTN_FONT_SIZE
    btn.TextColor3 = Color3.fromRGB(240,240,240)
    btn.BorderSizePixel = 0
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, BTN_RADIUS)

    local icon, circle = createCircleIcon(parent, y, false)
    icon.ZIndex = btn.ZIndex+1
    btn.ZIndex = btn.ZIndex+2

    local state = false
    local function updateVisual()
        local goal = {}
        if state then
            goal.BackgroundColor3 = Color3.fromRGB(50, 100, 70)
            circle.ImageColor3 = Color3.fromRGB(140,255,200) -- verde spectral
        else
            goal.BackgroundColor3 = Color3.fromRGB(70, 30, 30)
            circle.ImageColor3 = Color3.fromRGB(200,80,80) -- vermelho esmaecido
        end
        TweenService:Create(btn, TweenInfo.new(0.4, Enum.EasingStyle.Sine), goal):Play()
    end

    btn.MouseButton1Click:Connect(function()
        state = not state
        callback(state, btn)
        updateVisual()
    end)
    icon.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            state = not state
            callback(state, btn)
            updateVisual()
        end
    end)

    updateVisual()
    return btn, function(v)
        state = v
        callback(state, btn)
        updateVisual()
    end
end

-- Botão Ghost FPS Killer
local btnFPSDevourer, setFPSDevourerState = makeToggleBtn(main, "Ativar Ghost FPS Killer", BTN_Y0, function(on)
    if on then
        -- aqui liga o loop (mantém o original)
    else
        -- aqui desliga
    end
end)
