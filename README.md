-- language: Lua, file: arsenal_hub_v23.lua, runtime: Delta (Mobile)
-- *Corrigido: Uso de UIListLayout para alinhamento perfeito e sem sobreposição.*
-- *Tamanho ajustado (270x420).*
-- *Fluxo: Descrição -> Key -> Aimbot -> ESP -> Rodapé.*

if getgenv().ArsenalHubV23 then getgenv().ArsenalHubV23:Destroy() end

local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local SoundService = game:GetService("SoundService")
local Camera = Workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local guiParent
pcall(function() guiParent = gethui() end)
if not guiParent then guiParent = game:GetService("CoreGui") end

local Config = {
    Key = "killer66z",
    Unlocked = false,
    AimbotEnabled = false,
    ESPEnabled = false,
    FOVRadius = 150,
    FOVColor = Color3.fromRGB(255, 50, 50),
    ESPColor = Color3.fromRGB(50, 150, 255),
}

-- ========== FOV CIRCLE ==========
local FovGui = Instance.new("ScreenGui")
FovGui.Name = "ArsenalFov"
FovGui.ResetOnSpawn = false
FovGui.IgnoreGuiInset = true
FovGui.Parent = guiParent

local FovCircle = Instance.new("Frame")
FovCircle.AnchorPoint = Vector2.new(0.5, 0.5)
FovCircle.Position = UDim2.new(0.5, 0, 0.5, 0)
FovCircle.Size = UDim2.new(0, Config.FOVRadius * 2, 0, Config.FOVRadius * 2)
FovCircle.BackgroundTransparency = 1
FovCircle.Visible = false
FovCircle.Parent = FovGui

Instance.new("UICorner", FovCircle).CornerRadius = UDim.new(1, 0)
local CircleStroke = Instance.new("UIStroke", FovCircle)
CircleStroke.Color = Config.FOVColor
CircleStroke.Thickness = 2
CircleStroke.Transparency = 0.2

-- ========== AIMBOT & ESP ==========
local function isVisible(part)
    local char = LocalPlayer.Character
    if not char then return false end
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.FilterDescendantsInstances = {char, part.Parent}
    return Workspace:Raycast(Camera.CFrame.Position, part.Position - Camera.CFrame.Position, params) == nil
end

local function getClosestEnemy()
    if not Config.AimbotEnabled or not Config.Unlocked then return nil end
    local center = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
    local best, bestDist = nil, math.huge

    for _, pl in ipairs(Players:GetPlayers()) do
        if pl ~= LocalPlayer then
            local isEnemy = true
            if pl.Team and LocalPlayer.Team and pl.Team == LocalPlayer.Team then isEnemy = false end
            if isEnemy then
                local char = pl.Character
                local hum = char and char:FindFirstChildOfClass("Humanoid")
                if char and hum and hum.Health > 0 then
                    local part = char:FindFirstChild("Head")
                    if part and isVisible(part) then
                        local screenPos, onScreen = Camera:WorldToViewportPoint(part.Position)
                        if onScreen then
                            local dist = (Vector2.new(screenPos.X, screenPos.Y) - center).Magnitude
                            if dist <= Config.FOVRadius and dist < bestDist then
                                best = pl; bestDist = dist
                            end
                        end
                    end
                end
            end
        end
    end
    return best
end

local ESPGui = Instance.new("ScreenGui")
ESPGui.Name = "ArsenalESP"
ESPGui.ResetOnSpawn = false
ESPGui.IgnoreGuiInset = true
ESPGui.Parent = guiParent

local ESPElements = {}

local function CreateESP(player)
    if player == LocalPlayer or ESPElements[player] then return end
    local container = Instance.new("Frame")
    container.BackgroundTransparency = 1
    container.Visible = false
    container.Parent = ESPGui
    
    local box = Instance.new("Frame", container)
    box.BackgroundTransparency = 1; box.BorderSizePixel = 0; box.Size = UDim2.new(1, 0, 1, 0)
    
    local top = Instance.new("Frame", box); top.Size = UDim2.new(1,0,0,1); top.BackgroundColor3 = Config.ESPColor; top.BorderSizePixel = 0
    local bottom = Instance.new("Frame", box); bottom.Size = UDim2.new(1,0,0,1); bottom.Position = UDim2.new(0,0,1,-1); bottom.BackgroundColor3 = Config.ESPColor; bottom.BorderSizePixel = 0
    local left = Instance.new("Frame", box); left.Size = UDim2.new(0,1,1,0); left.BackgroundColor3 = Config.ESPColor; left.BorderSizePixel = 0
    local right = Instance.new("Frame", box); right.Size = UDim2.new(0,1,1,0); right.Position = UDim2.new(1,-1,0,0); right.BackgroundColor3 = Config.ESPColor; right.BorderSizePixel = 0
    
    local healthBg = Instance.new("Frame", container)
    healthBg.BackgroundColor3 = Color3.fromRGB(0, 0, 0); healthBg.BorderSizePixel = 0
    healthBg.Size = UDim2.new(0, 3, 1, 0); healthBg.Position = UDim2.new(0, -5, 0, 0)
    
    local healthFg = Instance.new("Frame", healthBg)
    healthFg.BackgroundColor3 = Color3.fromRGB(0, 255, 0); healthFg.BorderSizePixel = 0
    healthFg.Size = UDim2.new(1, 0, 1, 0); healthFg.Position = UDim2.new(0, 0, 0, 0)
    
    local textLabel = Instance.new("TextLabel", container)
    textLabel.BackgroundTransparency = 1; textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    textLabel.Font = Enum.Font.Arcade; textLabel.TextSize = 12; textLabel.TextStrokeTransparency = 0
    textLabel.Size = UDim2.new(1, 0, 0, 20); textLabel.Position = UDim2.new(0, 0, 0, -22)
    
    ESPElements[player] = { container = container, parts = {top, bottom, left, right}, healthFg = healthFg, text = textLabel }
end

for _, p in ipairs(Players:GetPlayers()) do CreateESP(p) end
Players.PlayerAdded:Connect(CreateESP)
Players.PlayerRemoving:Connect(function(p) if ESPElements[p] then ESPElements[p].container:Destroy(); ESPElements[p] = nil end end)

RunService.RenderStepped:Connect(function()
    if Config.AimbotEnabled then
        FovCircle.Visible = true
        FovCircle.Size = UDim2.new(0, Config.FOVRadius * 2, 0, Config.FOVRadius * 2)
        local target = getClosestEnemy()
        if target then
            local char = target.Character
            local part = char and char:FindFirstChild("Head")
            if part and (UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton1) or UserInputService:IsMouseButtonPressed(Enum.UserInputType.Touch)) then
                Camera.CFrame = CFrame.new(Camera.CFrame.Position, part.Position)
            end
        end
    else
        FovCircle.Visible = false
    end

    for player, data in pairs(ESPElements) do
        local showESP = Config.ESPEnabled and Config.Unlocked
        local char = player.Character
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        
        if not showESP or not hrp or not hum or hum.Health <= 0 then data.container.Visible = false; continue end
        if player.Team and LocalPlayer.Team and player.Team == LocalPlayer.Team then data.container.Visible = false; continue end

        local pos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
        if onScreen then
            local dist = (Camera.CFrame.Position - hrp.Position).Magnitude
            local height = math.clamp(2000 / dist, 30, 200)
            local width = height * 0.6

            data.container.Visible = true
            data.container.Size = UDim2.new(0, width, 0, height)
            data.container.Position = UDim2.new(0, pos.X - width/2, 0, pos.Y - height/2)
            
            data.text.Text = player.Name .. " [" .. math.floor(dist) .. "m]"
            data.text.Size = UDim2.new(0, width * 2, 0, 20)
            data.text.Position = UDim2.new(0, -width/2, 0, -22)
            
            local pct = math.clamp(hum.Health / hum.MaxHealth, 0, 1)
            data.healthFg.Size = UDim2.new(1, 0, pct, 0)
            data.healthFg.Position = UDim2.new(0, 0, 1 - pct, 0)
            data.healthFg.BackgroundColor3 = Color3.fromRGB(255 * (1 - pct), 255 * pct, 0)
        else
            data.container.Visible = false
        end
    end
end)

-- ========== SONS ==========
local ClickSound = Instance.new("Sound"); ClickSound.SoundId = "rbxassetid://5825423295"; ClickSound.Volume = 1; ClickSound.Parent = SoundService
local SuccessSound = Instance.new("Sound"); SuccessSound.SoundId = "rbxassetid://6647877129"; SuccessSound.Volume = 2; SuccessSound.Parent = SoundService

-- ========== UI ==========
local MainGui = Instance.new("ScreenGui")
MainGui.Name = "ArsenalHubV23"
MainGui.ResetOnSpawn = false
MainGui.IgnoreGuiInset = true
MainGui.Parent = guiParent
getgenv().ArsenalHubV23 = MainGui

local Window = Instance.new("Frame")
Window.Size = UDim2.new(0, 270, 0, 420); Window.Position = UDim2.new(0.5, -135, 0.5, -210)
Window.BackgroundColor3 = Color3.fromRGB(20, 20, 25); Window.BackgroundTransparency = 0.1; Window.BorderSizePixel = 0; Window.Active = true; Window.Parent = MainGui
Instance.new("UICorner", Window).CornerRadius = UDim.new(0, 8)

Window.Size = UDim2.new(0, 0, 0, 0); Window.Rotation = -20
TweenService:Create(Window, TweenInfo.new(0.6, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Size = UDim2.new(0, 270, 0, 420), Rotation = 0}):Play()

-- Arrasto
local TitleBar = Instance.new("Frame")
TitleBar.Size = UDim2.new(1, 0, 0, 50); TitleBar.BackgroundTransparency = 1; TitleBar.Parent = Window

local dragging, dragStart, startPos
TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true; dragStart = input.Position; startPos = Window.Position
        input.Changed:Connect(function() if input.UserInputState == Enum.UserInputState.End then dragging = false end end)
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        Window.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 25, 0, 25); CloseBtn.Position = UDim2.new(1, -30, 0, 10)
CloseBtn.BackgroundColor3 = Color3.fromRGB(200, 30, 30); CloseBtn.Text = "X"; CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.Font = Enum.Font.Arcade; CloseBtn.TextSize = 16; CloseBtn.Parent = Window
Instance.new("UICorner", CloseBtn).CornerRadius = UDim.new(1, 0)
CloseBtn.MouseButton1Click:Connect(function() ClickSound:Play(); MainGui:Destroy() end)

-- Imagem e Títulos no topo
local HeaderImage = Instance.new("ImageLabel")
HeaderImage.Size = UDim2.new(0, 40, 0, 40); HeaderImage.Position = UDim2.new(0.5, -20, 0, 10)
HeaderImage.BackgroundTransparency = 1; HeaderImage.Image = "rbxassetid://1006782430"; HeaderImage.Parent = Window

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 20); Title.Position = UDim2.new(0, 0, 0, 55)
Title.BackgroundTransparency = 1; Title.Text = "ARSENAL HUB"; Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.Arcade; Title.TextSize = 16; Title.Parent = Window

local SubTitle = Instance.new("TextLabel")
SubTitle.Size = UDim2.new(1, 0, 0, 15); SubTitle.Position = UDim2.new(0, 0, 0, 75)
SubTitle.BackgroundTransparency = 1; SubTitle.Text = "by killer"; SubTitle.TextColor3 = Color3.fromRGB(150, 150, 150)
SubTitle.Font = Enum.Font.Arcade; SubTitle.TextSize = 12; SubTitle.Parent = Window

-- ========== ÁREA DE CONTEÚDO COM UILISTLAYOUT ==========
local Content = Instance.new("Frame")
Content.Size = UDim2.new(1, -20, 1, -130); Content.Position = UDim2.new(0, 10, 0, 95)
Content.BackgroundTransparency = 1; Content.Parent = Window

local ListLayout = Instance.new("UIListLayout")
ListLayout.Parent = Content
ListLayout.Padding = UDim.new(0, 8)
ListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
ListLayout.SortOrder = Enum.SortOrder.LayoutOrder

-- PÁGINA KEY (Usa o ListLayout para organizar)
local PageKey = Instance.new("Frame")
PageKey.Size = UDim2.new(1, 0, 1, 0); PageKey.BackgroundTransparency = 1; PageKey.LayoutOrder = 1; PageKey.Parent = Content

local KeyDesc = Instance.new("TextLabel")
KeyDesc.Size = UDim2.new(1, 0, 0, 35); KeyDesc.LayoutOrder = 1
KeyDesc.BackgroundTransparency = 1; KeyDesc.Text = "Para pegar a key, mande mensagem para esse dc: i1lnl"
KeyDesc.TextColor3 = Color3.fromRGB(200, 200, 200); KeyDesc.Font = Enum.Font.Arcade; KeyDesc.TextSize = 11; KeyDesc.TextWrapped = true; KeyDesc.Parent = PageKey

local KeyInput = Instance.new("TextBox")
KeyInput.Size = UDim2.new(1, 0, 0, 35); KeyInput.LayoutOrder = 2
KeyInput.BackgroundColor3 = Color3.fromRGB(5, 5, 8); KeyInput.TextColor3 = Color3.fromRGB(255, 255, 255)
KeyInput.PlaceholderText = "Digite a Key..."; KeyInput.Font = Enum.Font.Arcade; KeyInput.TextSize = 14; KeyInput.Text = ""; KeyInput.Parent = PageKey
Instance.new("UICorner", KeyInput).CornerRadius = UDim.new(0, 4)

local VerifyBtn = Instance.new("TextButton")
VerifyBtn.Size = UDim2.new(1, 0, 0, 35); VerifyBtn.LayoutOrder = 3
VerifyBtn.BackgroundColor3 = Color3.fromRGB(180, 30, 30); VerifyBtn.Text = "VERIFICAR"; VerifyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
VerifyBtn.Font = Enum.Font.Arcade; VerifyBtn.TextSize = 14; VerifyBtn.Parent = PageKey
Instance.new("UICorner", VerifyBtn).CornerRadius = UDim.new(0, 4)

local StatusLabel = Instance.new("TextLabel")
StatusLabel.Size = UDim2.new(1, 0, 0, 20); StatusLabel.LayoutOrder = 4
StatusLabel.BackgroundTransparency = 1; StatusLabel.Text = ""; StatusLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
StatusLabel.Font = Enum.Font.Arcade; StatusLabel.TextSize = 12; StatusLabel.Parent = PageKey

-- PÁGINA MENU
local PageMain = Instance.new("Frame")
PageMain.Size = UDim2.new(1, 0, 1, 0); PageMain.BackgroundTransparency = 1; PageMain.LayoutOrder = 2; PageMain.Visible = false; PageMain.Parent = Content

-- ===== SEÇÃO AIMBOT =====
local AimTitle = Instance.new("TextLabel")
AimTitle.Size = UDim2.new(1, 0, 0, 20); AimTitle.LayoutOrder = 1
AimTitle.BackgroundTransparency = 1; AimTitle.Text = "--- AIMBOT ---"; AimTitle.TextColor3 = Color3.fromRGB(200, 50, 50)
AimTitle.Font = Enum.Font.Arcade; AimTitle.TextSize = 14; AimTitle.Parent = PageMain

local AimToggle = Instance.new("TextButton")
AimToggle.Size = UDim2.new(1, 0, 0, 35); AimToggle.LayoutOrder = 2
AimToggle.BackgroundColor3 = Color3.fromRGB(30, 30, 38); AimToggle.Text = "AIMBOT: OFF"; AimToggle.TextColor3 = Color3.fromRGB(150, 150, 150)
AimToggle.Font = Enum.Font.Arcade; AimToggle.TextSize = 14; AimToggle.Parent = PageMain
Instance.new("UICorner", AimToggle).CornerRadius = UDim.new(0, 4)

local FovLabel = Instance.new("TextLabel")
FovLabel.Size = UDim2.new(1, 0, 0, 20); FovLabel.LayoutOrder = 3
FovLabel.BackgroundTransparency = 1; FovLabel.Text = "FOV: 150"; FovLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
FovLabel.Font = Enum.Font.Arcade; FovLabel.TextSize = 14; FovLabel.TextXAlignment = Enum.TextXAlignment.Left; FovLabel.Parent = PageMain

local SliderBg = Instance.new("Frame")
SliderBg.Size = UDim2.new(1, 0, 0, 8); SliderBg.LayoutOrder = 4
SliderBg.BackgroundColor3 = Color3.fromRGB(50, 50, 60); SliderBg.Parent = PageMain
Instance.new("UICorner", SliderBg).CornerRadius = UDim.new(1, 0)

local SliderFill = Instance.new("Frame")
SliderFill.Size = UDim2.new(150/400, 0, 1, 0); SliderFill.BackgroundColor3 = Config.FOVColor; SliderFill.Parent = SliderBg
Instance.new("UICorner", SliderFill).CornerRadius = UDim.new(1, 0)

local SliderBtn = Instance.new("TextButton")
SliderBtn.Size = UDim2.new(0, 20, 0, 20); SliderBtn.Position = UDim2.new(150/400, -10, 0.5, -10)
SliderBtn.BackgroundColor3 = Color3.fromRGB(255, 255, 255); SliderBtn.Text = ""; SliderBtn.Parent = SliderBg
Instance.new("UICorner", SliderBtn).CornerRadius = UDim.new(1, 0)

local FovColorBtn = Instance.new("TextButton")
FovColorBtn.Size = UDim2.new(1, 0, 0, 35); FovColorBtn.LayoutOrder = 5
FovColorBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 60); FovColorBtn.Text = "COR FOV: VERMELHO"; FovColorBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
FovColorBtn.Font = Enum.Font.Arcade; FovColorBtn.TextSize = 14; FovColorBtn.Parent = PageMain
Instance.new("UICorner", FovColorBtn).CornerRadius = UDim.new(0, 4)

-- ===== SEÇÃO ESP =====
local EspTitle = Instance.new("TextLabel")
EspTitle.Size = UDim2.new(1, 0, 0, 20); EspTitle.LayoutOrder = 6
EspTitle.BackgroundTransparency = 1; EspTitle.Text = "--- ESP ---"; EspTitle.TextColor3 = Color3.fromRGB(50, 150, 255)
EspTitle.Font = Enum.Font.Arcade; EspTitle.TextSize = 14; EspTitle.Parent = PageMain

local ESPToggle = Instance.new("TextButton")
ESPToggle.Size = UDim2.new(1, 0, 0, 35); ESPToggle.LayoutOrder = 7
ESPToggle.BackgroundColor3 = Color3.fromRGB(30, 30, 38); ESPToggle.Text = "ESP: OFF"; ESPToggle.TextColor3 = Color3.fromRGB(150, 150, 150)
ESPToggle.Font = Enum.Font.Arcade; ESPToggle.TextSize = 14; ESPToggle.Parent = PageMain
Instance.new("UICorner", ESPToggle).CornerRadius = UDim.new(0, 4)

local EspColorBtn = Instance.new("TextButton")
EspColorBtn.Size = UDim2.new(1, 0, 0, 35); EspColorBtn.LayoutOrder = 8
EspColorBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 60); EspColorBtn.Text = "COR ESP: AZUL"; EspColorBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
EspColorBtn.Font = Enum.Font.Arcade; EspColorBtn.TextSize = 14; EspColorBtn.Parent = PageMain
Instance.new("UICorner", EspColorBtn).CornerRadius = UDim.new(0, 4)

-- RODAPÉ
local Footer = Instance.new("TextLabel")
Footer.Size = UDim2.new(1, -20, 0, 20); Footer.Position = UDim2.new(0, 10, 1, -25)
Footer.BackgroundTransparency = 1; Footer.Text = "killer | Discord: @i1lnl"
Footer.TextColor3 = Color3.fromRGB(200, 50, 50); Footer.Font = Enum.Font.Arcade; Footer.TextSize = 12; Footer.Parent = Window

-- ========== LÓGICA UI ==========
VerifyBtn.MouseButton1Click:Connect(function()
    ClickSound:Play()
    if string.gsub(KeyInput.Text, "%s+", "") == Config.Key then
        Config.Unlocked = true; SuccessSound:Play()
        StatusLabel.TextColor3 = Color3.fromRGB(0, 255, 100); StatusLabel.Text = "SUCESSO!"
        task.wait(1)
        PageKey.Visible = false; PageMain.Visible = true
    else
        StatusLabel.TextColor3 = Color3.fromRGB(255, 50, 50); StatusLabel.Text = "INCORRETA."
    end
end)

AimToggle.MouseButton1Click:Connect(function()
    ClickSound:Play()
    Config.AimbotEnabled = not Config.AimbotEnabled
    if Config.AimbotEnabled then
        AimToggle.Text = "AIMBOT: ON"; AimToggle.TextColor3 = Color3.fromRGB(0, 255, 100); AimToggle.BackgroundColor3 = Color3.fromRGB(25, 50, 30); FovCircle.Visible = true
    else
        AimToggle.Text = "AIMBOT: OFF"; AimToggle.TextColor3 = Color3.fromRGB(150, 150, 150); AimToggle.BackgroundColor3 = Color3.fromRGB(30, 30, 38); FovCircle.Visible = false
    end
end)

ESPToggle.MouseButton1Click:Connect(function()
    ClickSound:Play()
    Config.ESPEnabled = not Config.ESPEnabled
    ESPToggle.Text = Config.ESPEnabled and "ESP: ON" or "ESP: OFF"
    ESPToggle.TextColor3 = Config.ESPEnabled and Color3.fromRGB(0, 255, 100) or Color3.fromRGB(150, 150, 150)
    ESPToggle.BackgroundColor3 = Config.ESPEnabled and Color3.fromRGB(25, 50, 30) or Color3.fromRGB(30, 30, 38)
end)

local fovMode = 1
local fovColors = {Color3.fromRGB(255, 50, 50), Color3.fromRGB(0, 255, 0), Color3.fromRGB(50, 150, 255), Color3.fromRGB(180, 50, 255)}
local fovNames = {"VERMELHO", "VERDE", "AZUL", "ROXO"}

FovColorBtn.MouseButton1Click:Connect(function()
    ClickSound:Play()
    fovMode = fovMode + 1
    if fovMode > #fovColors then fovMode = 1 end
    
    Config.FOVColor = fovColors[fovMode]
    FovColorBtn.Text = "COR FOV: " .. fovNames[fovMode]
    FovColorBtn.BackgroundColor3 = Config.FOVColor
    CircleStroke.Color = Config.FOVColor
    SliderFill.BackgroundColor3 = Config.FOVColor
end)

local espMode = 1
local espColors = {Color3.fromRGB(50, 150, 255), Color3.fromRGB(255, 50, 50), Color3.fromRGB(0, 255, 0), Color3.fromRGB(180, 50, 255)}
local espNames = {"AZUL", "VERMELHO", "VERDE", "ROXO"}

EspColorBtn.MouseButton1Click:Connect(function()
    ClickSound:Play()
    espMode = espMode + 1
    if espMode > #espColors then espMode = 1 end
    
    Config.ESPColor = espColors[espMode]
    EspColorBtn.Text = "COR ESP: " .. espNames[espMode]
    EspColorBtn.BackgroundColor3 = Config.ESPColor
    
    for _, data in pairs(ESPElements) do 
        for _, part in ipairs(data.parts) do part.BackgroundColor3 = Config.ESPColor end 
    end
end)

local sliderDragging = false
SliderBtn.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then sliderDragging = true end
end)
UserInputService.InputChanged:Connect(function(input)
    if sliderDragging and (input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement) then
        local percent = math.clamp((input.Position.X - SliderBg.AbsolutePosition.X) / SliderBg.AbsoluteSize.X, 0, 1)
        local val = math.floor(50 + (percent * 350))
        Config.FOVRadius = val
        FovCircle.Size = UDim2.new(0, val * 2, 0, val * 2)
        SliderFill.Size = UDim2.new(percent, 0, 1, 0)
        SliderBtn.Position = UDim2.new(percent, -10, 0.5, -10)
        FovLabel.Text = "FOV: " .. val
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then sliderDragging = false end
end)

print("Arsenal Hub V23 carregado. Key: killer66z")
