-- Variáveis principais
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")

local ESPEnabled = false
local ESPObjects = {}

-- Criar botão na tela (usando Drawing API)
local button = Drawing.new("Text")
button.Text = "[ESP] Clique F2 para Ativar/Desativar"
button.Size = 16
button.Center = true
button.Outline = true
button.Color = Color3.fromRGB(0, 255, 0)
button.Position = Vector2.new(200, 50)
button.Visible = true

-- Alternar ESP com tecla F2
game:GetService("UserInputService").InputBegan:Connect(function(input, gpe)
    if gpe then return end
    if input.KeyCode == Enum.KeyCode.F2 then
        ESPEnabled = not ESPEnabled
        button.Color = ESPEnabled and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
    end
end)

-- Criar ESP para um jogador
local function CreateESP(player)
    if player == LocalPlayer then return end

    local espText = Drawing.new("Text")
    espText.Size = 14
    espText.Center = true
    espText.Outline = true
    espText.Color = Color3.fromRGB(255, 0, 0)
    espText.Visible = false

    ESPObjects[player] = espText

    RunService.RenderStepped:Connect(function()
        if not ESPEnabled or not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then
            espText.Visible = false
            return
        end

        -- Team Check
        if player.Team == LocalPlayer.Team then
            espText.Visible = false
            return
        end

        local pos, onScreen = Camera:WorldToViewportPoint(player.Character.HumanoidRootPart.Position)
        if onScreen then
            espText.Position = Vector2.new(pos.X, pos.Y - 20)
            espText.Text = player.Name
            espText.Visible = true
        else
            espText.Visible = false
        end
    end)
end

-- Criar ESP para jogadores existentes
for _, player in ipairs(Players:GetPlayers()) do
    CreateESP(player)
end

-- Criar ESP para novos jogadores
Players.PlayerAdded:Connect(function(player)
    CreateESP(player)
end)

-- Remover ESP quando jogador sair
Players.PlayerRemoving:Connect(function(player)
    if ESPObjects[player] then
        ESPObjects[player]:Remove()
        ESPObjects[player] = nil
    end
end)
