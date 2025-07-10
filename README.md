local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

local ESP_ENABLED = false
local HIGHLIGHT_NAME = "PurpleESP_Highlight"
local toggleKey = Enum.KeyCode.F -- Tecla padrão

-- Função para aplicar/remover ESP
local function setESPState(state)
    ESP_ENABLED = state
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local existing = player.Character:FindFirstChild(HIGHLIGHT_NAME)
            if state then
                if not existing then
                    local highlight = Instance.new("Highlight")
                    highlight.Name = HIGHLIGHT_NAME
                    highlight.FillColor = Color3.fromRGB(128, 0, 128)
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

-- Atualiza ESP quando novos jogadores entram
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(char)
        if ESP_ENABLED then
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

-- GUI Construction
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "NotzinESPMenu"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = game:GetService("Players").LocalPlayer:WaitForChild("PlayerGui")

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 350, 0, 250)
MainFrame.Position = UDim2.new(0.5, -175, 0.4, -125)
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 32)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 36)
Title.Position = UDim2.new(0, 0, 0, 0)
Title.BackgroundTransparency = 1
Title.Font = Enum.Font.GothamBold
Title.Text = "notzin"
Title.TextSize = 28
Title.TextColor3 = Color3.fromRGB(170, 85, 255)
Title.Parent = MainFrame

-- Abas
local tabNames = {"Main", "Esp", "Info"}
local tabButtons = {}
local tabContents = {}

local function showTab(idx)
    for i, c in ipairs(tabContents) do
        c.Visible = (i == idx)
        tabButtons[i].BackgroundColor3 = (i == idx) and Color3.fromRGB(80, 60, 100) or Color3.fromRGB(30, 30, 40)
    end
end

for i, name in ipairs(tabNames) do
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 110, 0, 28)
    btn.Position = UDim2.new(0, (i-1)*112, 0, 38)
    btn.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
    btn.Font = Enum.Font.GothamSemibold
    btn.Text = name
    btn.TextSize = 20
    btn.TextColor3 = Color3.fromRGB(230, 230, 255)
    btn.AutoButtonColor = true
    btn.Parent = MainFrame
    btn.MouseButton1Click:Connect(function() showTab(i) end)
    tabButtons[i] = btn

    local content = Instance.new("Frame")
    content.Size = UDim2.new(1, -10, 1, -75)
    content.Position = UDim2.new(0, 5, 0, 70)
    content.BackgroundTransparency = 1
    content.Visible = false
    content.Parent = MainFrame
    tabContents[i] = content
end

-- Main Tab (só exemplo, pode adicionar opções)
local mainLabel = Instance.new("TextLabel")
mainLabel.Size = UDim2.new(1, 0, 0, 32)
mainLabel.Position = UDim2.new(0, 0, 0, 0)
mainLabel.BackgroundTransparency = 1
mainLabel.Font = Enum.Font.Gotham
mainLabel.Text = "Bem-vindo ao menu notzin!"
mainLabel.TextSize = 20
mainLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
mainLabel.Parent = tabContents[1]

-- Info Tab
local infoLabel = Instance.new("TextLabel")
infoLabel.Size = UDim2.new(1, 0, 1, 0)
infoLabel.Position = UDim2.new(0, 0, 0, 0)
infoLabel.BackgroundTransparency = 1
infoLabel.Font = Enum.Font.Gotham
infoLabel.TextWrapped = true
infoLabel.Text = "Menu ESP feito por notzin.\n\nAba Esp: Ative/desative o ESP roxo nos jogadores.\nConfiguração de tecla também disponível."
infoLabel.TextSize = 18
infoLabel.TextColor3 = Color3.fromRGB(220, 220, 240)
infoLabel.Parent = tabContents[3]

-- ESP Tab
local EspTab = tabContents[2]

local toggleESPBtn = Instance.new("TextButton")
toggleESPBtn.Size = UDim2.new(0, 180, 0, 38)
toggleESPBtn.Position = UDim2.new(0, 8, 0, 10)
toggleESPBtn.BackgroundColor3 = Color3.fromRGB(70, 30, 90)
toggleESPBtn.Font = Enum.Font.GothamBold
toggleESPBtn.TextSize = 20
toggleESPBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleESPBtn.AutoButtonColor = true
toggleESPBtn.Parent = EspTab

local function updateToggleESPBtn()
    toggleESPBtn.Text = ESP_ENABLED and "Desativar ESP" or "Ativar ESP"
    toggleESPBtn.BackgroundColor3 = ESP_ENABLED and Color3.fromRGB(100, 40, 120) or Color3.fromRGB(70, 30, 90)
end

toggleESPBtn.MouseButton1Click:Connect(function()
    setESPState(not ESP_ENABLED)
    updateToggleESPBtn()
end)

updateToggleESPBtn()

-- Keybind setting
local keybindLabel = Instance.new("TextLabel")
keybindLabel.Size = UDim2.new(0, 160, 0, 28)
keybindLabel.Position = UDim2.new(0, 8, 0, 60)
keybindLabel.BackgroundTransparency = 1
keybindLabel.Font = Enum.Font.Gotham
keybindLabel.Text = "Tecla para toggle:"
keybindLabel.TextSize = 18
keybindLabel.TextColor3 = Color3.fromRGB(230, 210, 255)
keybindLabel.Parent = EspTab

local keybindBtn = Instance.new("TextButton")
keybindBtn.Size = UDim2.new(0, 70, 0, 28)
keybindBtn.Position = UDim2.new(0, 172, 0, 60)
keybindBtn.BackgroundColor3 = Color3.fromRGB(40, 20, 60)
keybindBtn.Font = Enum.Font.GothamSemibold
keybindBtn.TextSize = 18
keybindBtn.TextColor3 = Color3.fromRGB(230, 210, 255)
keybindBtn.Text = toggleKey.Name
keybindBtn.Parent = EspTab

local waitingForKey = false

keybindBtn.MouseButton1Click:Connect(function()
    if waitingForKey then return end
    keybindBtn.Text = "Pressione..."
    waitingForKey = true
end)

UserInputService.InputBegan:Connect(function(input, processed)
    if not processed then
        -- Keybind capture
        if waitingForKey and input.UserInputType == Enum.UserInputType.Keyboard then
            toggleKey = input.KeyCode
            keybindBtn.Text = toggleKey.Name
            waitingForKey = false
        -- Toggle ESP com tecla configurada
        elseif input.KeyCode == toggleKey and not waitingForKey and MainFrame.Visible then
            setESPState(not ESP_ENABLED)
            updateToggleESPBtn()
        -- Abrir/fechar menu com "P"
        elseif input.KeyCode == Enum.KeyCode.P and not waitingForKey then
            MainFrame.Visible = not MainFrame.Visible
        end
    end
end)

-- Inicializa exibindo Main tab
showTab(1)
MainFrame.Visible = true -- Menu começa visível
