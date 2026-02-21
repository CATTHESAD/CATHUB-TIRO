local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- CONFIGURAÇÕES
local Config = {
    Aimbot = false,
    ESP = false,
    Smoothness = 0, -- Trava instantânea
    FOV = 200,
    Shooting = false, -- Detecta se está atirando
    MenuVisible = true
}

-- 1. INTERFACE CATHUD (TEMA SANGUE)
local ScreenGui = Instance.new("ScreenGui", LocalPlayer.PlayerGui)
ScreenGui.Name = "CATHUD_Shooting"
ScreenGui.ResetOnSpawn = false
ScreenGui.DisplayOrder = 999999

-- Botão Gota Vermelha
local OpenBtn = Instance.new("TextButton", ScreenGui)
OpenBtn.Size = UDim2.new(0, 50, 0, 50)
OpenBtn.Position = UDim2.new(0.02, 0, 0.3, 0)
OpenBtn.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
OpenBtn.Text = "●"
OpenBtn.TextColor3 = Color3.fromRGB(255, 0, 0)
OpenBtn.TextSize = 35
Instance.new("UICorner", OpenBtn).CornerRadius = UDim.new(0.5, 0)
local BtnStroke = Instance.new("UIStroke", OpenBtn)
BtnStroke.Thickness = 2; BtnStroke.Color = Color3.fromRGB(200, 0, 0)

local DropShadow = Instance.new("TextLabel", OpenBtn)
DropShadow.Text = "▼"; DropShadow.Size = UDim2.new(1, 0, 1, 0); DropShadow.Position = UDim2.new(0, 0, 0.3, 0)
DropShadow.BackgroundTransparency = 1; DropShadow.TextColor3 = Color3.fromRGB(255, 0, 0); DropShadow.TextSize = 20

-- Menu Principal
local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Size = UDim2.new(0, 200, 0, 160); MainFrame.Position = UDim2.new(0.02, 60, 0.3, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 15); MainFrame.Active = true; MainFrame.Draggable = true
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)
local MainStroke = Instance.new("UIStroke", MainFrame)
MainStroke.Thickness = 2; MainStroke.Color = Color3.fromRGB(255, 0, 0)

local Title = Instance.new("TextLabel", MainFrame)
Title.Text = "CATHUD"; Title.Size = UDim2.new(1, 0, 0, 40); Title.BackgroundTransparency = 1
Title.TextColor3 = Color3.fromRGB(255, 0, 0); Title.Font = Enum.Font.GothamBold; Title.TextSize = 18

OpenBtn.MouseButton1Click:Connect(function()
    Config.MenuVisible = not Config.MenuVisible
    MainFrame.Visible = Config.MenuVisible
    OpenBtn.Text = Config.MenuVisible and "X" or "●"
    DropShadow.Visible = not Config.MenuVisible
end)

local function CreateToggle(name, pos, callback)
    local btn = Instance.new("TextButton", MainFrame)
    btn.Text = name .. ": OFF"; btn.Size = UDim2.new(0.9, 0, 0, 40); btn.Position = pos
    btn.BackgroundColor3 = Color3.fromRGB(20, 20, 20); btn.TextColor3 = Color3.fromRGB(200, 0, 0)
    btn.Font = Enum.Font.GothamSemibold; btn.TextSize = 13
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)
    local state = false
    btn.MouseButton1Click:Connect(function()
        state = not state
        btn.Text = name .. ": " .. (state and "ON" or "OFF")
        btn.BackgroundColor3 = state and Color3.fromRGB(180, 0, 0) or Color3.fromRGB(20, 20, 20)
        btn.TextColor3 = state and Color3.new(1, 1, 1) or Color3.fromRGB(200, 0, 0)
        callback(state)
    end)
end

-- 2. ESP INTELIGENTE (VERDE/VERMELHO)
local function ApplyESP(player)
    if player == LocalPlayer then return end
    local bGui = Instance.new("BillboardGui", ScreenGui)
    bGui.Name = "ESP_" .. player.Name; bGui.AlwaysOnTop = true; bGui.Size = UDim2.new(4, 0, 5.5, 0); bGui.Enabled = false
    local frame = Instance.new("Frame", bGui); frame.Size = UDim2.new(1, 0, 1, 0); frame.BackgroundTransparency = 0.5; frame.BorderSizePixel = 0

    RunService.RenderStepped:Connect(function()
        if Config.ESP and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local hum = player.Character:FindFirstChildOfClass("Humanoid")
            if hum and hum.Health > 0 then
                local isAlly = (player.Team == LocalPlayer.Team) and (player.Team ~= nil)
                frame.BackgroundColor3 = isAlly and Color3.new(0, 1, 0) or Color3.new(1, 0, 0)
                bGui.Adornee = player.Character.HumanoidRootPart
                bGui.Enabled = true
            else bGui.Enabled = false end
        else bGui.Enabled = false end
    end)
end

for _, p in pairs(Players:GetPlayers()) do ApplyESP(p) end
Players.PlayerAdded:Connect(ApplyESP)

-- 3. DETECÇÃO DE TIRO (BOTÃO ESQUERDO)
UserInputService.InputBegan:Connect(function(i, processed)
    if not processed and i.UserInputType == Enum.UserInputType.MouseButton1 then
        Config.Shooting = true
    end
end)
UserInputService.InputEnded:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 then
        Config.Shooting = false
    end
end)

-- 4. AIMBOT ULTRA AGRESSIVO (LOCK-ON NO TIRO)
RunService:BindToRenderStep("AimbotUpdate", Enum.RenderPriority.Camera.Value + 1, function()
    -- SÓ TRAVA SE O AIMBOT ESTIVER ON E VOCÊ ESTIVER ATIRANDO (BOTÃO ESQUERDO)
    if Config.Aimbot and Config.Shooting then
        local target, shortestDist = nil, Config.FOV
        for _, p in pairs(Players:GetPlayers()) do
            local isAlly = (p.Team == LocalPlayer.Team) and (p.Team ~= nil)
            if p ~= LocalPlayer and not isAlly and p.Character and p.Character:FindFirstChild("Head") then
                local hum = p.Character:FindFirstChildOfClass("Humanoid")
                if hum and hum.Health > 0 then
                    local pos, onScreen = Camera:WorldToViewportPoint(p.Character.Head.Position)
                    if onScreen then
                        local mousePos = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
                        local dist = (Vector2.new(pos.X, pos.Y) - mousePos).Magnitude
                        if dist < shortestDist then shortestDist = dist; target = p.Character.Head end
                    end
                end
            end
        end
        if target then Camera.CFrame = CFrame.lookAt(Camera.CFrame.Position, target.Position) end
    end
end)

CreateToggle("Trigger Aim", UDim2.new(0.05, 0, 0.25, 0), function(v) Config.Aimbot = v end)
CreateToggle("Team ESP", UDim2.new(0.05, 0, 0.6, 0), function(v) Config.ESP = v end)
