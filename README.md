local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

-- Configurações Highlight
local ESP_ENABLED = false
local HIGHLIGHT_NAME = "PurpleESP_Highlight"
local toggleKey = Enum.KeyCode.F -- Tecla padrão
local highlightColor = "Roxo" -- Inicial
local highlightColors = {
    ["Roxo"] = {fill = Color3.fromRGB(128,0,128), outline = Color3.fromRGB(200,0,200)},
    ["Amarelo"] = {fill = Color3.fromRGB(255,255,0), outline = Color3.fromRGB(255,255,90)},
    ["Verde"] = {fill = Color3.fromRGB(0,255,0), outline = Color3.fromRGB(90,255,90)},
    ["Azul"] = {fill = Color3.fromRGB(0,128,255), outline = Color3.fromRGB(0,200,255)},
}

-- Propriedades visuais highlight
local fillTransparency = 0.08
local outlineTransparency = 0.0

-- Speed/Jump Config
local walkSpeed = 16
local jumpPower = 50
local maxSpeed = 200
local maxJump = 400

-- Função para aplicar highlight a todos os jogadores (exceto você)
local function setESPState(state)
    ESP_ENABLED = state
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local existing = player.Character:FindFirstChild(HIGHLIGHT_NAME)
            if state then
                if not existing then
                    local highlight = Instance.new("Highlight")
                    highlight.Name = HIGHLIGHT_NAME
                    local color = highlightColors[highlightColor]
                    highlight.FillColor = color.fill
                    highlight.OutlineColor = color.outline
                    highlight.FillTransparency = fillTransparency
                    highlight.OutlineTransparency = outlineTransparency
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

-- Atualiza cor dos highlights ativos
local function updateHighlightColors()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local h = player.Character:FindFirstChild(HIGHLIGHT_NAME)
            if h and h:IsA("Highlight") then
                local color = highlightColors[highlightColor]
                h.FillColor = color.fill
                h.OutlineColor = color.outline
                h.FillTransparency = fillTransparency
                h.OutlineTransparency = outlineTransparency
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
            local color = highlightColors[highlightColor]
            highlight.FillColor = color.fill
            highlight.OutlineColor = color.outline
            highlight.FillTransparency = fillTransparency
            highlight.OutlineTransparency = outlineTransparency
            highlight.Adornee = char
            highlight.Parent = char
        end
    end)
end)

-- GUI Construction
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "NotzinESPMenu"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 410, 0, 290)
MainFrame.Position = UDim2.new(0.5, -205, 0.4, -145)
MainFrame.BackgroundColor3 = Color3.fromRGB(25,25,32)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.ZIndex = 2
MainFrame.Parent = ScreenGui

-- Arredondamento
local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 18)
UICorner.Parent = MainFrame

-- Título centralizado
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 36)
Title.Position = UDim2.new(0, 0, 0, 0)
Title.BackgroundTransparency = 1
Title.Font = Enum.Font.GothamBold
Title.Text = "Notzin"
Title.TextSize = 28
Title.TextColor3 = Color3.fromRGB(170, 85, 255)
Title.Parent = MainFrame
Title.ZIndex = 2

-- Barra lateral de abas
local sideBar = Instance.new("Frame")
sideBar.Size = UDim2.new(0, 92, 1, -36)
sideBar.Position = UDim2.new(0,0,0,36)
sideBar.BackgroundColor3 = Color3.fromRGB(30,30,45)
sideBar.BorderSizePixel = 0
sideBar.ZIndex = 2
sideBar.Parent = MainFrame

local sideBarCorner = Instance.new("UICorner")
sideBarCorner.CornerRadius = UDim.new(0, 14)
sideBarCorner.Parent = sideBar

-- Abas
local tabNames = {"Main", "Esp", "Info"}
local tabButtons = {}
local tabContents = {}

local function showTab(idx)
    for i, c in ipairs(tabContents) do
        c.Visible = (i == idx)
        tabButtons[i].BackgroundColor3 = (i == idx) and Color3.fromRGB(80, 60, 100) or Color3.fromRGB(30, 30, 45)
    end
end

for i, name in ipairs(tabNames) do
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, -16, 0, 38)
    btn.Position = UDim2.new(0, 8, 0, 12 + (i-1)*46)
    btn.BackgroundColor3 = Color3.fromRGB(30, 30, 45)
    btn.Font = Enum.Font.GothamSemibold
    btn.Text = name
    btn.TextSize = 20
    btn.TextColor3 = Color3.fromRGB(230, 230, 255)
    btn.AutoButtonColor = true
    btn.ZIndex = 2
    btn.Parent = sideBar
    btn.MouseButton1Click:Connect(function() showTab(i) end)
    tabButtons[i] = btn

    local content = Instance.new("Frame")
    content.Size = UDim2.new(1,-110,1,-42)
    content.Position = UDim2.new(0, 102, 0, 40)
    content.BackgroundTransparency = 1
    content.Visible = false
    content.ZIndex = 2
    content.Parent = MainFrame
    tabContents[i] = content
end

-- MAIN TAB: Speed/Jump sliders
local mainTab = tabContents[1]

local function createSlider(labelText, minValue, maxValue, startValue, posY, callback)
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0, 110, 0, 24)
    label.Position = UDim2.new(0, 10, 0, posY)
    label.BackgroundTransparency = 1
    label.Font = Enum.Font.Gotham
    label.Text = labelText .. ":"
    label.TextSize = 17
    label.TextColor3 = Color3.fromRGB(220,220,255)
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.ZIndex = 2
    label.Parent = mainTab

    local valueLabel = Instance.new("TextLabel")
    valueLabel.Size = UDim2.new(0, 60, 0, 24)
    valueLabel.Position = UDim2.new(0, 210, 0, posY)
    valueLabel.BackgroundTransparency = 1
    valueLabel.Font = Enum.Font.GothamBold
    valueLabel.Text = tostring(startValue)
    valueLabel.TextSize = 17
    valueLabel.TextColor3 = Color3.fromRGB(120,255,120)
    valueLabel.TextXAlignment = Enum.TextXAlignment.Left
    valueLabel.ZIndex = 2
    valueLabel.Parent = mainTab

    local sliderBG = Instance.new("Frame")
    sliderBG.Size = UDim2.new(0,170,0,8)
    sliderBG.Position = UDim2.new(0, 120, 0, posY+8)
    sliderBG.BackgroundColor3 = Color3.fromRGB(40,40,60)
    sliderBG.BorderSizePixel = 0
    sliderBG.ZIndex = 2
    sliderBG.Parent = mainTab
    local cornerBG = Instance.new("UICorner")
    cornerBG.CornerRadius = UDim.new(0,4)
    cornerBG.Parent = sliderBG

    local sliderFill = Instance.new("Frame")
    sliderFill.Size = UDim2.new((startValue-minValue)/(maxValue-minValue),0,1,0)
    sliderFill.BackgroundColor3 = Color3.fromRGB(110,85,200)
    sliderFill.BorderSizePixel = 0
    sliderFill.ZIndex = 2
    sliderFill.Parent = sliderBG
    local cornerFill = Instance.new("UICorner")
    cornerFill.CornerRadius = UDim.new(0,4)
    cornerFill.Parent = sliderFill

    local dragging = false

    sliderBG.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local x = math.clamp(input.Position.X - sliderBG.AbsolutePosition.X, 0, sliderBG.AbsoluteSize.X)
            local pct = x/sliderBG.AbsoluteSize.X
            local value = math.floor(minValue + (maxValue-minValue)*pct + 0.5)
            sliderFill.Size = UDim2.new(pct,0,1,0)
            valueLabel.Text = tostring(value)
            callback(value)
        end
    end)
end

createSlider("Speed", 8, maxSpeed, walkSpeed, 10, function(val)
    walkSpeed = val
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
        LocalPlayer.Character.Humanoid.WalkSpeed = walkSpeed
    end
end)
createSlider("Jump Power", 20, maxJump, jumpPower, 54, function(val)
    jumpPower = val
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
        LocalPlayer.Character.Humanoid.JumpPower = jumpPower
    end
end)

-- Ajusta valores ao spawnar
LocalPlayer.CharacterAdded:Connect(function(char)
    local hum = char:WaitForChild("Humanoid")
    hum.WalkSpeed = walkSpeed
    hum.JumpPower = jumpPower
end)

-- ESP TAB
local EspTab = tabContents[2]

local toggleESPBtn = Instance.new("TextButton")
toggleESPBtn.Size = UDim2.new(0, 180, 0, 38)
toggleESPBtn.Position = UDim2.new(0, 8, 0, 10)
toggleESPBtn.BackgroundColor3 = Color3.fromRGB(70, 30, 90)
toggleESPBtn.Font = Enum.Font.GothamBold
toggleESPBtn.TextSize = 20
toggleESPBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleESPBtn.AutoButtonColor = true
toggleESPBtn.ZIndex = 2
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

-- Seletor de cor do highlight
local colorLabel = Instance.new("TextLabel")
colorLabel.Size = UDim2.new(0, 140, 0, 26)
colorLabel.Position = UDim2.new(0, 8, 0, 62)
colorLabel.BackgroundTransparency = 1
colorLabel.Font = Enum.Font.Gotham
colorLabel.Text = "Cor do Highlight:"
colorLabel.TextSize = 18
colorLabel.TextColor3 = Color3.fromRGB(240, 230, 255)
colorLabel.TextXAlignment = Enum.TextXAlignment.Left
colorLabel.ZIndex = 2
colorLabel.Parent = EspTab

local colorDropdown = Instance.new("TextButton")
colorDropdown.Size = UDim2.new(0, 90, 0, 26)
colorDropdown.Position = UDim2.new(0, 150, 0, 62)
colorDropdown.BackgroundColor3 = Color3.fromRGB(40, 20, 60)
colorDropdown.Font = Enum.Font.GothamSemibold
colorDropdown.TextSize = 18
colorDropdown.TextColor3 = Color3.fromRGB(230, 210, 255)
colorDropdown.Text = highlightColor
colorDropdown.ZIndex = 3 -- ZIndex alto para ficar sobre qualquer outro
colorDropdown.Parent = EspTab

local dropOpen = false
local colorOptionsFrame = Instance.new("Frame")
colorOptionsFrame.Size = UDim2.new(1, 0, 0, 106)
colorOptionsFrame.Position = UDim2.new(0, 0, 0, 26)
colorOptionsFrame.BackgroundColor3 = Color3.fromRGB(30, 15, 45)
colorOptionsFrame.BorderSizePixel = 0
colorOptionsFrame.Visible = false
colorOptionsFrame.ZIndex = 4 -- Acima de tudo
colorOptionsFrame.Parent = colorDropdown
local colorCorner = Instance.new("UICorner")
colorCorner.CornerRadius = UDim.new(0,6)
colorCorner.Parent = colorOptionsFrame

for idx, cname in ipairs({"Roxo", "Amarelo", "Verde", "Azul"}) do
    local opt = Instance.new("TextButton")
    opt.Size = UDim2.new(1,0,0,24)
    opt.Position = UDim2.new(0,0,0,(idx-1)*26)
    opt.BackgroundColor3 = Color3.fromRGB(40, 20, 60)
    opt.Text = cname
    opt.Font = Enum.Font.Gotham
    opt.TextSize = 18
    opt.TextColor3 = Color3.fromRGB(230,230,255)
    opt.ZIndex = 5
    opt.Parent = colorOptionsFrame
    opt.MouseButton1Click:Connect(function()
        highlightColor = cname
        colorDropdown.Text = cname
        colorOptionsFrame.Visible = false
        dropOpen = false
        updateHighlightColors()
    end)
end
colorDropdown.MouseButton1Click:Connect(function()
    dropOpen = not dropOpen
    colorOptionsFrame.Visible = dropOpen
    colorOptionsFrame.ZIndex = 4
    colorDropdown.ZIndex = dropOpen and 5 or 3
end)

-- Keybind setting
local keybindLabel = Instance.new("TextLabel")
keybindLabel.Size = UDim2.new(0, 160, 0, 28)
keybindLabel.Position = UDim2.new(0, 8, 0, 102)
keybindLabel.BackgroundTransparency = 1
keybindLabel.Font = Enum.Font.Gotham
keybindLabel.Text = "Tecla para toggle:"
keybindLabel.TextSize = 18
keybindLabel.TextColor3 = Color3.fromRGB(230, 210, 255)
keybindLabel.ZIndex = 2
keybindLabel.Parent = EspTab

local keybindBtn = Instance.new("TextButton")
keybindBtn.Size = UDim2.new(0, 70, 0, 28)
keybindBtn.Position = UDim2.new(0, 172, 0, 102)
keybindBtn.BackgroundColor3 = Color3.fromRGB(40, 20, 60)
keybindBtn.Font = Enum.Font.GothamSemibold
keybindBtn.TextSize = 18
keybindBtn.TextColor3 = Color3.fromRGB(230, 210, 255)
keybindBtn.Text = toggleKey.Name
keybindBtn.ZIndex = 2
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

-- INFO TAB
local infoTab = tabContents[3]
local infoLabel = Instance.new("TextLabel")
infoLabel.Size = UDim2.new(1, 0, 1, 0)
infoLabel.Position = UDim2.new(0, 0, 0, 0)
infoLabel.BackgroundTransparency = 1
infoLabel.Font = Enum.Font.Gotham
infoLabel.TextWrapped = true
infoLabel.Text = "Menu ESP feito por Notzin.\n\nAba Esp: Ative/desative o ESP, escolha cor, configure tecla toggle.\nAba Main: Speed/JumpPower do seu personagem.\n\nMenu arredondado, abas laterais, abre/fecha com P."
infoLabel.TextSize = 18
infoLabel.TextColor3 = Color3.fromRGB(220, 220, 240)
infoLabel.ZIndex = 2
infoLabel.Parent = infoTab

-- Inicializa exibindo Main tab
showTab(1)
MainFrame.Visible = true -- Menu começa visível
