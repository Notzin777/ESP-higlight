local UserInputService = game:GetService("UserInputService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local ESP_ENABLED = false
local HIGHLIGHT_NAME = "PurpleESP_Highlight"

local function toggleESP()
    ESP_ENABLED = not ESP_ENABLED

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local existing = player.Character:FindFirstChild(HIGHLIGHT_NAME)
            if ESP_ENABLED then
                if not existing then
                    local highlight = Instance.new("Highlight")
                    highlight.Name = HIGHLIGHT_NAME
                    highlight.FillColor = Color3.fromRGB(128, 0, 128) -- Roxo
                    highlight.OutlineColor = Color3.fromRGB(200, 0, 200)
                    highlight.FillTransparency = 0.5
                    highlight.OutlineTransparency = 0
                    highlight.Adornee = player.Character
                    highlight.Parent = player.Character
                end
            else
                if existing then
                    existing:Destroy()
                end
            end
        end
    end
end

UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.F then
        toggleESP()
    end
end)

-- Se jogadores entrarem depois do script rodar
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(char)
        if ESP_ENABLED then
            -- Adiciona o highlight se ESP estiver ativo
            local highlight = Instance.new("Highlight")
            highlight.Name = HIGHLIGHT_NAME
            highlight.FillColor = Color3.fromRGB(128, 0, 128)
            highlight.OutlineColor = Color3.fromRGB(200, 0, 200)
            highlight.FillTransparency = 0.5
            highlight.OutlineTransparency = 0
            highlight.Adornee = char
            highlight.Parent = char
        end
    end)
end)
