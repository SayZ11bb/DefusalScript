# DefusalScript
Mod of defusal script

-- this is a Modified versio, The script is still under development.

-- Créditos: serq47 -- Creator
-- Código original por: serq47
-- Versão: 1.4.1 -- Mod
Ty for use My Mod version
-- Modified by Sayurizzw

-- Verifica se o exploit suporta Drawing API
local function API_Check()
    return Drawing and "Yes" or "No"
end

if API_Check() == "No" then
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Erro",
        Text = "Seu exploit não suporta Drawing API!",
        Duration = math.huge,
        Button1 = "OK"
    })
    return
end

-- Serviços do Roblox
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- Configurações Globais
_G.ESPVisible = true
_G.TeamCheck = true
_G.AimbotEnabled = false
_G.TextColor = Color3.fromRGB(255, 80, 10)
_G.TextSize = 14
_G.Outline = true
_G.OutlineColor = Color3.fromRGB(0, 0, 0)
_G.TextTransparency = 0.7
_G.TextFont = Drawing.Fonts.UI
_G.DisableKey = Enum.KeyCode.Q
_G.AimbotKey = Enum.KeyCode.E
_G.FOV = 37
_G.FOVCircleVisible = true
_G.AimbotSmoothness = 5
_G.MaxAimbotDistance = 300
_G.HealthBarSize = 40 -- Tamanho da barra de vida
_G.FOVColor = Color3.fromRGB(0, 0, 255) -- Cor do FOV

-- Criar GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = LocalPlayer:FindFirstChildOfClass("PlayerGui")
ScreenGui.Name = "ESP_Aimbot_GUI"

local espButton = Instance.new("TextButton")
espButton.Size = UDim2.new(0, 100, 0, 40)
espButton.Position = UDim2.new(0, 50, 0, 100)
espButton.Text = "Toggle ESP"
espButton.Parent = ScreenGui
espButton.BackgroundColor3 = Color3.fromRGB(60, 20, 60)

local aimbotButton = Instance.new("TextButton")
aimbotButton.Size = UDim2.new(0, 100, 0, 40)
aimbotButton.Position = UDim2.new(0, 50, 0, 160)
aimbotButton.Text = "Aimbot"
aimbotButton.Parent = ScreenGui
aimbotButton.BackgroundColor3 = Color3.fromRGB(160, 32, 240)

local fovButton = Instance.new("TextButton")
fovButton.Size = UDim2.new(0, 200, 0, 50)
fovButton.Position = UDim2.new(0, 50, 0, 220)
fovButton.Text = "Toggle FOV Circle"
fovButton.Parent = ScreenGui
fovButton.BackgroundColor3 = Color3.fromRGB(40, 90, 255)

-- Criar Círculo do FOV
local FOVCircle = Drawing.new("Circle")
FOVCircle.Radius = _G.FOV
FOVCircle.Thickness = 1
FOVCircle.Transparency = 0.5
FOVCircle.Color = _G.FOVColor
FOVCircle.Visible = _G.FOVCircleVisible

RunService.RenderStepped:Connect(function()
    FOVCircle.Position = UserInputService:GetMouseLocation()
end)

-- Função para criar ESP com barra de vida
local ESPObjects = {}

local function CreateESP(player)
    if player == LocalPlayer then return end

    local ESP = Drawing.new("Text")
    local HealthBar = Drawing.new("Line")

    local function UpdateESP()
        if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then
            ESP.Visible = false
            HealthBar.Visible = true
            return
        end

        local root = player.Character:FindFirstChild("HumanoidRootPart")
        local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
        if not root or not humanoid or humanoid.Health <= 0 then
            ESP.Visible = false
            HealthBar.Visible = false
            return
        end

        local vector, onScreen = Camera:WorldToViewportPoint(root.Position)
        ESP.Position = Vector2.new(vector.X, vector.Y - 25)
        ESP.Text = ("(%d) %s"):format((root.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude, player.Name)
        ESP.Color = _G.TextColor
        ESP.Visible = _G.ESPVisible and onScreen

        local hpPercent = humanoid.Health / humanoid.MaxHealth
        HealthBar.From = Vector2.new(vector.X - 20, vector.Y + 15)
        HealthBar.To = Vector2.new(vector.X - 20 + (_G.HealthBarSize * hpPercent), vector.Y + 15)
        HealthBar.Color = Color3.fromRGB(255 - (hpPercent * 255), hpPercent * 255, 0)
        HealthBar.Thickness = 3
        HealthBar.Visible = _G.ESPVisible and onScreen
    end

    -- Conectar a eventos de atualização
    RunService.RenderStepped:Connect(UpdateESP)
    player.CharacterAdded:Connect(UpdateESP)
    player.AncestryChanged:Connect(function()
        ESP.Visible = false
        HealthBar.Visible = false
    end)

    ESPObjects[player] = { ESP = ESP, HealthBar = HealthBar }
end

-- Atualiza o ESP para todos os jogadores
for _, player in ipairs(Players:GetPlayers()) do
    CreateESP(player)
end

-- Destruir objetos ESP de jogadores removidos
Players.PlayerRemoving:Connect(function(player)
    local espObj = ESPObjects[player]
    if espObj then
        espObj.ESP:Remove()
        espObj.HealthBar:Remove()
        ESPObjects[player] = nil
    end
end)

-- Aimbot Melhorado
local function Aimbot()
    if not _G.AimbotEnabled then return end

    local closestPlayer, minDistance = nil, _G.FOV
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local head = player.Character.Head
            local screenPos, onScreen = Camera:WorldToViewportPoint(head.Position)
            local mousePos = UserInputService:GetMouseLocation()
            local distance = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude

            if onScreen and distance < minDistance and (head.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude <= _G.MaxAimbotDistance then
                minDistance = distance
                closestPlayer = player
            end
        end
    end

    if closestPlayer then
        local targetPosition = closestPlayer.Character.Head.Position
        local smoothness = math.clamp(minDistance / _G.FOV, 0.1, 1)
        Camera.CFrame = Camera.CFrame:Lerp(CFrame.new(Camera.CFrame.Position, targetPosition), smoothness)
    end
end

-- Rodar Aimbot a cada frame
RunService.RenderStepped:Connect(Aimbot)

-- Botões da GUI
espButton.MouseButton1Click:Connect(function()
    _G.ESPVisible = not _G.ESPVisible
end)

aimbotButton.MouseButton1Click:Connect(function()
    _G.AimbotEnabled = not _G.AimbotEnabled
end)

fovButton.MouseButton1Click:Connect(function()
    _G.FOVCircleVisible = not _G.FOVCircleVisible
    FOVCircle.Visible = _G.FOVCircleVisible
end)

-- Teclas para ativar/desativar Aimbot e ESP
UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == _G.DisableKey then
        _G.ESPVisible = not _G.ESPVisible
    elseif input.KeyCode == _G.AimbotKey then
        _G.AimbotEnabled = not _G.AimbotEnabled
    end
end)
