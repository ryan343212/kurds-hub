-- Menu bonito, com abas, Speed, Infinite Jump e nome "Kurds Hub" no topo esquerdo
local aimbotAtivo = false
local espEsqueletoAtivo = false
local espickAtivo = false
local infiniteJumpAtivo = false
local speedAtivo = false
local speedValor = 16 -- valor padrão Roblox
local teclaAtivar = Enum.KeyCode.E
local teclaMenu = Enum.KeyCode.K
local fov = 120
local fovMin = 10
local fovMax = 300
local distanciaMax = 1000
local distanciaMin = 50
local distanciaMaxLimit = 3000
local mouseDireitoAtivo = false

-- Funções de aimbot e ESP (iguais aos exemplos anteriores)
local function encontrarAlvo()
    local players = game.Players:GetPlayers()
    local camera = workspace.CurrentCamera
    local mouse = game.Players.LocalPlayer:GetMouse()
    local localChar = game.Players.LocalPlayer.Character
    local localHead = localChar and localChar:FindFirstChild("Head")
    if not localHead then return nil end
    local alvo = nil
    local menorDist = math.huge
    for _, player in ipairs(players) do
        if player ~= game.Players.LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local pos, visivel = camera:WorldToScreenPoint(player.Character.Head.Position)
            local distTela = (Vector2.new(pos.X, pos.Y) - Vector2.new(mouse.X, mouse.Y)).magnitude
            local distReal = (localHead.Position - player.Character.Head.Position).Magnitude
            if visivel and distTela < menorDist and distTela <= fov and distReal <= distanciaMax then
                menorDist = distTela
                alvo = player
            end
        end
    end
    return alvo
end

local function mirarNoAlvo(alvo)
    if alvo and alvo.Character and alvo.Character:FindFirstChild("Head") then
        local camera = workspace.CurrentCamera
        camera.CFrame = CFrame.new(camera.CFrame.Position, alvo.Character.Head.Position)
    end
end

local UserInputService = game:GetService("UserInputService")
UserInputService.InputBegan:Connect(function(input, processado)
    if not processado then
        if input.KeyCode == teclaAtivar then
            aimbotAtivo = not aimbotAtivo
        end
        if input.UserInputType == Enum.UserInputType.MouseButton2 then
            mouseDireitoAtivo = true
        end
        if input.KeyCode == teclaMenu then
            gui.Enabled = not gui.Enabled
        end
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton2 then
        mouseDireitoAtivo = false
    end
end)

-- ESP Esqueleto
local function limparEsqueleto()
    for _, obj in pairs(workspace:GetChildren()) do
        if obj.Name == "SkeletonESP" and obj:IsA("Folder") then
            obj:Destroy()
        end
    end
end
local function mostrarEsqueleto()
    limparEsqueleto()
    if not espEsqueletoAtivo then return end
    local skeletonFolder = Instance.new("Folder", workspace)
    skeletonFolder.Name = "SkeletonESP"
    for _, player in ipairs(game.Players:GetPlayers()) do
        if player ~= game.Players.LocalPlayer and player.Character then
            local char = player.Character
            local parts = {"Head", "Torso", "LeftArm", "RightArm", "LeftLeg", "RightLeg"}
            for _, partName in ipairs(parts) do
                local part = char:FindFirstChild(partName)
                if part then
                    local box = Instance.new("BoxHandleAdornment")
                    box.Adornee = part
                    box.Size = part.Size + Vector3.new(0.1,0.1,0.1)
                    box.Color3 = Color3.new(0,1,0)
                    box.Transparency = 0.5
                    box.AlwaysOnTop = true
                    box.ZIndex = 10
                    box.Parent = skeletonFolder
                end
            end
        end
    end
end

-- ESP Nick
local function limparNickESP()
    for _, player in ipairs(game.Players:GetPlayers()) do
        if player ~= game.Players.LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local head = player.Character.Head
            if head:FindFirstChild("NickBillboard") then
                head.NickBillboard:Destroy()
            end
        end
    end
end
local function mostrarNickESP()
    limparNickESP()
    if not espNickAtivo then return end
    for _, player in ipairs(game.Players:GetPlayers()) do
        if player ~= game.Players.LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local head = player.Character.Head
            if not head:FindFirstChild("NickBillboard") then
                local billboard = Instance.new("BillboardGui")
                billboard.Name = "NickBillboard"
                billboard.Adornee = head
                billboard.Size = UDim2.new(0, 200, 0, 50)
                billboard.StudsOffset = Vector3.new(0, 2.5, 0)
                billboard.AlwaysOnTop = true
                billboard.Parent = head
                local label = Instance.new("TextLabel", billboard)
                label.Size = UDim2.new(1, 0, 1, 0)
                label.BackgroundTransparency = 1
                label.Text = player.Name
                label.TextColor3 = Color3.fromRGB(255, 255, 0)
                label.TextStrokeTransparency = 0.5
                label.TextScaled = true
                label.Font = Enum.Font.SourceSansBold
            end
        end
    end
end

-- Infinite Jump
local jumpConnection = nil
local function ativarInfiniteJump()
    if jumpConnection then jumpConnection:Disconnect() end
    jumpConnection = UserInputService.JumpRequest:Connect(function()
        if infiniteJumpAtivo then
            local char = game.Players.LocalPlayer.Character
            if char and char:FindFirstChild("Humanoid") then
                char.Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
            end
        end
    end)
end
local function desativarInfiniteJump()
    if jumpConnection then jumpConnection:Disconnect() jumpConnection = nil end
end

-- Speed Hack
local function aplicarSpeed()
    local char = game.Players.LocalPlayer.Character
    if char and char:FindFirstChild("Humanoid") then
        char.Humanoid.WalkSpeed = speedAtivo and speedValor or 16
    end
end
game.Players.LocalPlayer.CharacterAdded:Connect(function(char)
    wait(1)
    aplicarSpeed()
end)

game:GetService("RunService").RenderStepped:Connect(function()
    -- ESP
    if espEsqueletoAtivo then mostrarEsqueleto() else limparEsqueleto() end
    if espNickAtivo then mostrarNickESP() else limparNickESP() end
    -- Speed
    aplicarSpeed()
    -- Infinite Jump
    if infiniteJumpAtivo then
        ativarInfiniteJump()
    else
        desativarInfiniteJump()
    end
    -- Aimbot
    if aimbotAtivo and mouseDireitoAtivo then
        local alvo = encontrarAlvo()
        mirarNoAlvo(alvo)
    end
end)

-- GUI MENU (com abas e nome)
local player = game.Players.LocalPlayer
local gui = Instance.new("ScreenGui")
gui.Name = "AimbotMenu"
gui.ResetOnSpawn = false
gui.Parent = game:GetService("CoreGui")

local circulo = Instance.new("ImageButton", gui)
circulo.Name = "CirculoMenu"
circulo.Size = UDim2.new(0, 60, 0, 60)
circulo.Position = UDim2.new(0, 20, 0.7, 0)
circulo.BackgroundTransparency = 1
circulo.Image = "rbxassetid://3570695787"
circulo.ImageColor3 = Color3.fromRGB(60, 180, 255)

local dragging = false
local dragInput, mousePos, framePos

circulo.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        mousePos = input.Position
        framePos = circulo.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)
circulo.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - mousePos
        local newPos = UDim2.new(
            framePos.X.Scale,
            framePos.X.Offset + delta.X,
            framePos.Y.Scale,
            framePos.Y.Offset + delta.Y
        )
        circulo.Position = newPos
        if abaMain.Visible then
            abaMain.Position = UDim2.new(
                newPos.X.Scale,
                newPos.X.Offset + 90,
                newPos.Y.Scale,
                newPos.Y.Offset
            )
        end
    end
end)

-- Main menu frame
local abaMain = Instance.new("Frame", gui)
abaMain.Name = "AbaMain"
abaMain.Size = UDim2.new(0, 350, 0, 380)
abaMain.Position = UDim2.new(0, 100, 0.7, 0)
abaMain.BackgroundColor3 = Color3.fromRGB(40,60,100)
abaMain.BorderSizePixel = 0
abaMain.Visible = false

local cornerMain = Instance.new("UICorner", abaMain)
cornerMain.CornerRadius = UDim.new(0,18)

-- Nome do menu no topo esquerdo
local titulo = Instance.new("TextLabel", abaMain)
titulo.Name = "TituloKurdsHub"
titulo.Size = UDim2.new(0, 200, 0, 36)
titulo.Position = UDim2.new(0, 12, 0, 7)
titulo.BackgroundTransparency = 1
titulo.Text = "Kurds Hub"
titulo.TextColor3 = Color3.fromRGB(210,255,80)
titulo.Font = Enum.Font.GothamBold
titulo.TextSize = 32
titulo.TextXAlignment = Enum.TextXAlignment.Left

local tabBar = Instance.new("Frame", abaMain)
tabBar.Size = UDim2.new(1,0,0,50)
tabBar.Position = UDim2.new(0,0,0,0)
tabBar.BackgroundColor3 = Color3.fromRGB(60,90,160)
tabBar.BorderSizePixel = 0
local cornerTab = Instance.new("UICorner", tabBar)
cornerTab.CornerRadius = UDim.new(0,18)

local tabAimbotBtn = Instance.new("TextButton", tabBar)
tabAimbotBtn.Size = UDim2.new(0.33, -7, 1, -10)
tabAimbotBtn.Position = UDim2.new(0,7,0,5)
tabAimbotBtn.Text = "Aimbot"
tabAimbotBtn.BackgroundColor3 = Color3.fromRGB(100,180,255)
tabAimbotBtn.Font = Enum.Font.GothamBold
tabAimbotBtn.TextSize = 22
tabAimbotBtn.TextColor3 = Color3.new(0,0,0)
tabAimbotBtn.AutoButtonColor = false
local cornerAimbot = Instance.new("UICorner", tabAimbotBtn)
cornerAimbot.CornerRadius = UDim.new(0,12)

local tabESPBtn = Instance.new("TextButton", tabBar)
tabESPBtn.Size = UDim2.new(0.33, -7, 1, -10)
tabESPBtn.Position = UDim2.new(0.33,0,0,5)
tabESPBtn.Text = "ESP"
tabESPBtn.BackgroundColor3 = Color3.fromRGB(200,255,140)
tabESPBtn.Font = Enum.Font.GothamBold
tabESPBtn.TextSize = 22
tabESPBtn.TextColor3 = Color3.new(0,0,0)
tabESPBtn.AutoButtonColor = false
local cornerESP = Instance.new("UICorner", tabESPBtn)
cornerESP.CornerRadius = UDim.new(0,12)

local tabExtrasBtn = Instance.new("TextButton", tabBar)
tabExtrasBtn.Size = UDim2.new(0.33, -7, 1, -10)
tabExtrasBtn.Position = UDim2.new(0.66,0,0,5)
tabExtrasBtn.Text = "Extras"
tabExtrasBtn.BackgroundColor3 = Color3.fromRGB(255,180,120)
tabExtrasBtn.Font = Enum.Font.GothamBold
tabExtrasBtn.TextSize = 22
tabExtrasBtn.TextColor3 = Color3.new(0,0,0)
tabExtrasBtn.AutoButtonColor = false
local cornerExtras = Instance.new("UICorner", tabExtrasBtn)
cornerExtras.CornerRadius = UDim.new(0,12)

-- Abas
local abaAimbot = Instance.new("Frame", abaMain)
abaAimbot.Size = UDim2.new(1,0,1,-50)
abaAimbot.Position = UDim2.new(0,0,0,50)
abaAimbot.BackgroundTransparency = 1
abaAimbot.Visible = true

local abaESP = Instance.new("Frame", abaMain)
abaESP.Size = UDim2.new(1,0,1,-50)
abaESP.Position = UDim2.new(0,0,0,50)
abaESP.BackgroundTransparency = 1
abaESP.Visible = false

local abaExtras = Instance.new("Frame", abaMain)
abaExtras.Size = UDim2.new(1,0,1,-50)
abaExtras.Position = UDim2.new(0,0,0,50)
abaExtras.BackgroundTransparency = 1
abaExtras.Visible = false

tabAimbotBtn.MouseButton1Click:Connect(function()
    abaAimbot.Visible = true
    abaESP.Visible = false
    abaExtras.Visible = false
end)
tabESPBtn.MouseButton1Click:Connect(function()
    abaAimbot.Visible = false
    abaESP.Visible = true
    abaExtras.Visible = false
end)
tabExtrasBtn.MouseButton1Click:Connect(function()
    abaAimbot.Visible = false
    abaESP.Visible = false
    abaExtras.Visible = true
end)

-- Aimbot Aba conteudo
local btnAimbot = Instance.new("TextButton", abaAimbot)
btnAimbot.Size = UDim2.new(1,-40,0,40)
btnAimbot.Position = UDim2.new(0,20,0,12)
btnAimbot.Text = "Aimbot: DESLIGADO"
btnAimbot.BackgroundColor3 = Color3.fromRGB(100,180,255)
btnAimbot.TextColor3 = Color3.fromRGB(0,0,0)
btnAimbot.Font = Enum.Font.GothamBold
btnAimbot.TextSize = 18
local btnAimbotCorner = Instance.new("UICorner", btnAimbot)
btnAimbotCorner.CornerRadius = UDim.new(0,8)
btnAimbot.MouseButton1Click:Connect(function()
    aimbotAtivo = not aimbotAtivo
    btnAimbot.Text = "Aimbot: " .. (aimbotAtivo and "LIGADO" or "DESLIGADO")
end)

local fovLabel = Instance.new("TextLabel", abaAimbot)
fovLabel.Size = UDim2.new(0, 120, 0, 30)
fovLabel.Position = UDim2.new(0, 30, 0, 70)
fovLabel.Text = "FOV: " .. tostring(fov)
fovLabel.BackgroundTransparency = 1
fovLabel.TextColor3 = Color3.fromRGB(255,255,255)
fovLabel.Font = Enum.Font.GothamBold
fovLabel.TextSize = 16

local sliderFrame = Instance.new("Frame", abaAimbot)
sliderFrame.Size = UDim2.new(0, 170, 0, 10)
sliderFrame.Position = UDim2.new(0, 160, 0, 80)
sliderFrame.BackgroundColor3 = Color3.fromRGB(140, 180, 250)
sliderFrame.BorderSizePixel = 0
local sliderFrameCorner = Instance.new("UICorner", sliderFrame)
sliderFrameCorner.CornerRadius = UDim.new(0,5)

local sliderButton = Instance.new("Frame", sliderFrame)
sliderButton.Size = UDim2.new(0, 10, 0, 30)
sliderButton.Position = UDim2.new(0, math.floor((fov-fovMin)/(fovMax-fovMin)*(sliderFrame.Size.X.Offset-10)), 0, -10)
sliderButton.BackgroundColor3 = Color3.fromRGB(100,180,255)
sliderButton.BorderSizePixel = 0
sliderButton.ZIndex = 2
local sliderBtnCorner = Instance.new("UICorner", sliderButton)
sliderBtnCorner.CornerRadius = UDim.new(0,5)

local draggingSlider = false
sliderButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        draggingSlider = true
    end
end)
sliderButton.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        draggingSlider = false
    end
end)
sliderFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        draggingSlider = true
        local x = math.clamp(input.Position.X - sliderFrame.AbsolutePosition.X, 0, sliderFrame.Size.X.Offset-10)
        sliderButton.Position = UDim2.new(0, x, 0, -10)
        fov = math.floor((x / (sliderFrame.Size.X.Offset-10)) * (fovMax - fovMin) + fovMin)
        fovLabel.Text = "FOV: " .. tostring(fov)
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if draggingSlider and input.UserInputType == Enum.UserInputType.MouseMovement then
        local x = math.clamp(input.Position.X - sliderFrame.AbsolutePosition.X, 0, sliderFrame.Size.X.Offset-10)
        sliderButton.Position = UDim2.new(0, x, 0, -10)
        fov = math.floor((x / (sliderFrame.Size.X.Offset-10)) * (fovMax - fovMin) + fovMin)
        fovLabel.Text = "FOV: " .. tostring(fov)
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        draggingSlider = false
    end
end)

local distLabel = Instance.new("TextLabel", abaAimbot)
distLabel.Size = UDim2.new(0, 120, 0, 30)
distLabel.Position = UDim2.new(0, 30, 0, 110)
distLabel.Text = "Distância: " .. tostring(distanciaMax)
distLabel.BackgroundTransparency = 1
distLabel.TextColor3 = Color3.fromRGB(255,255,255)
distLabel.Font = Enum.Font.GothamBold
distLabel.TextSize = 16

local distSliderFrame = Instance.new("Frame", abaAimbot)
distSliderFrame.Size = UDim2.new(0, 170, 0, 10)
distSliderFrame.Position = UDim2.new(0, 160, 0, 120)
distSliderFrame.BackgroundColor3 = Color3.fromRGB(180, 255, 200)
distSliderFrame.BorderSizePixel = 0
local distSliderFrameCorner = Instance.new("UICorner", distSliderFrame)
distSliderFrameCorner.CornerRadius = UDim.new(0,5)

local distSliderButton = Instance.new("Frame", distSliderFrame)
distSliderButton.Size = UDim2.new(0, 10, 0, 30)
distSliderButton.Position = UDim2.new(0, math.floor((distanciaMax-distanciaMin)/(distanciaMaxLimit-distanciaMin)*(distSliderFrame.Size.X.Offset-10)), 0, -10)
distSliderButton.BackgroundColor3 = Color3.fromRGB(100, 200, 100)
distSliderButton.BorderSizePixel = 0
distSliderButton.ZIndex = 2
local distSliderBtnCorner = Instance.new("UICorner", distSliderButton)
distSliderBtnCorner.CornerRadius = UDim.new(0,5)

local draggingDistSlider = false
distSliderButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        draggingDistSlider = true
    end
end)
distSliderButton.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        draggingDistSlider = false
    end
end)
distSliderFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        draggingDistSlider = true
        local x = math.clamp(input.Position.X - distSliderFrame.AbsolutePosition.X, 0, distSliderFrame.Size.X.Offset-10)
        distSliderButton.Position = UDim2.new(0, x, 0, -10)
        distanciaMax = math.floor((x / (distSliderFrame.Size.X.Offset-10)) * (distanciaMaxLimit - distanciaMin) + distanciaMin)
        distLabel.Text = "Distância: " .. tostring(distanciaMax)
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if draggingDistSlider and input.UserInputType == Enum.UserInputType.MouseMovement then
        local x = math.clamp(input.Position.X - distSliderFrame.AbsolutePosition.X, 0, distSliderFrame.Size.X.Offset-10)
        distSliderButton.Position = UDim2.new(0, x, 0, -10)
        distanciaMax = math.floor((x / (distSliderFrame.Size.X.Offset-10)) * (distanciaMaxLimit - distanciaMin) + distanciaMin)
        distLabel.Text = "Distância: " .. tostring(distanciaMax)
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        draggingDistSlider = false
    end
end)

-- ESP Aba conteudo
local btnESP = Instance.new("TextButton", abaESP)
btnESP.Size = UDim2.new(1,-40,0,40)
btnESP.Position = UDim2.new(0,20,0,12)
btnESP.Text = "ESP Esqueleto: DESLIGADO"
btnESP.BackgroundColor3 = Color3.fromRGB(180,255,180)
btnESP.TextColor3 = Color3.fromRGB(0,0,0)
btnESP.Font = Enum.Font.GothamBold
btnESP.TextSize = 18
local btnESPCorner = Instance.new("UICorner", btnESP)
btnESPCorner.CornerRadius = UDim.new(0,8)
btnESP.MouseButton1Click:Connect(function()
    espEsqueletoAtivo = not espEsqueletoAtivo
    btnESP.Text = "ESP Esqueleto: " .. (espEsqueletoAtivo and "LIGADO" or "DESLIGADO")
    if espEsqueletoAtivo then mostrarEsqueleto() else limparEsqueleto() end
end)

local btnNickESP = Instance.new("TextButton", abaESP)
btnNickESP.Size = UDim2.new(1,-40,0,40)
btnNickESP.Position = UDim2.new(0,20,0,62)
btnNickESP.Text = "ESP Nick: DESLIGADO"
btnNickESP.BackgroundColor3 = Color3.fromRGB(255,255,120)
btnNickESP.TextColor3 = Color3.fromRGB(0,0,0)
btnNickESP.Font = Enum.Font.GothamBold
btnNickESP.TextSize = 18
local btnNickESPCorner = Instance.new("UICorner", btnNickESP)
btnNickESPCorner.CornerRadius = UDim.new(0,8)
btnNickESP.MouseButton1Click:Connect(function()
    espNickAtivo = not espNickAtivo
    btnNickESP.Text = "ESP Nick: " .. (espNickAtivo and "LIGADO" or "DESLIGADO")
    if espNickAtivo then mostrarNickESP() else limparNickESP() end
end)

-- Aba Extras: Infinite Jump e Speed
local btnInfJump = Instance.new("TextButton", abaExtras)
btnInfJump.Size = UDim2.new(1,-40,0,40)
btnInfJump.Position = UDim2.new(0,20,0,12)
btnInfJump.Text = "Infinite Jump: DESLIGADO"
btnInfJump.BackgroundColor3 = Color3.fromRGB(230,180,255)
btnInfJump.TextColor3 = Color3.fromRGB(0,0,0)
btnInfJump.Font = Enum.Font.GothamBold
btnInfJump.TextSize = 18
local btnInfJumpCorner = Instance.new("UICorner", btnInfJump)
btnInfJumpCorner.CornerRadius = UDim.new(0,8)
btnInfJump.MouseButton1Click:Connect(function()
    infiniteJumpAtivo = not infiniteJumpAtivo
    btnInfJump.Text = "Infinite Jump: " .. (infiniteJumpAtivo and "LIGADO" or "DESLIGADO")
end)

local btnSpeed = Instance.new("TextButton", abaExtras)
btnSpeed.Size = UDim2.new(1,-40,0,40)
btnSpeed.Position = UDim2.new(0,20,0,62)
btnSpeed.Text = "Speed: DESLIGADO"
btnSpeed.BackgroundColor3 = Color3.fromRGB(255,180,120)
btnSpeed.TextColor3 = Color3.fromRGB(0,0,0)
btnSpeed.Font = Enum.Font.GothamBold
btnSpeed.TextSize = 18
local btnSpeedCorner = Instance.new("UICorner", btnSpeed)
btnSpeedCorner.CornerRadius = UDim.new(0,8)
btnSpeed.MouseButton1Click:Connect(function()
    speedAtivo = not speedAtivo
    btnSpeed.Text = "Speed: " .. (speedAtivo and "LIGADO" or "DESLIGADO")
    aplicarSpeed()
end)

local speedLabel = Instance.new("TextLabel", abaExtras)
speedLabel.Size = UDim2.new(0, 120, 0, 30)
speedLabel.Position = UDim2.new(0, 30, 0, 112)
speedLabel.Text = "Velocidade: " .. tostring(speedValor)
speedLabel.BackgroundTransparency = 1
speedLabel.TextColor3 = Color3.fromRGB(255,255,255)
speedLabel.Font = Enum.Font.GothamBold
speedLabel.TextSize = 16

local speedSliderFrame = Instance.new("Frame", abaExtras)
speedSliderFrame.Size = UDim2.new(0, 170, 0, 10)
speedSliderFrame.Position = UDim2.new(0, 160, 0, 122)
speedSliderFrame.BackgroundColor3 = Color3.fromRGB(255, 220, 120)
speedSliderFrame.BorderSizePixel = 0
local speedSliderFrameCorner = Instance.new("UICorner", speedSliderFrame)
speedSliderFrameCorner.CornerRadius = UDim.new(0,5)

local speedSliderButton = Instance.new("Frame", speedSliderFrame)
speedSliderButton.Size = UDim2.new(0, 10, 0, 30)
speedSliderButton.Position = UDim2.new(0, math.floor((speedValor-16)/(200-16)*(speedSliderFrame.Size.X.Offset-10)), 0, -10)
speedSliderButton.BackgroundColor3 = Color3.fromRGB(255,180,120)
speedSliderButton.BorderSizePixel = 0
speedSliderButton.ZIndex = 2
local speedSliderBtnCorner = Instance.new("UICorner", speedSliderButton)
speedSliderBtnCorner.CornerRadius = UDim.new(0,5)

local draggingSpeedSlider = false
speedSliderButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        draggingSpeedSlider = true
    end
end)
speedSliderButton.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        draggingSpeedSlider = false
    end
end)
speedSliderFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        draggingSpeedSlider = true
        local x = math.clamp(input.Position.X - speedSliderFrame.AbsolutePosition.X, 0, speedSliderFrame.Size.X.Offset-10)
        speedSliderButton.Position = UDim2.new(0, x, 0, -10)
        speedValor = math.floor((x / (speedSliderFrame.Size.X.Offset-10)) * (200 - 16) + 16)
        speedLabel.Text = "Velocidade: " .. tostring(speedValor)
        aplicarSpeed()
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if draggingSpeedSlider and input.UserInputType == Enum.UserInputType.MouseMovement then
        local x = math.clamp(input.Position.X - speedSliderFrame.AbsolutePosition.X, 0, speedSliderFrame.Size.X.Offset-10)
        speedSliderButton.Position = UDim2.new(0, x, 0, -10)
        speedValor = math.floor((x / (speedSliderFrame.Size.X.Offset-10)) * (200 - 16) + 16)
        speedLabel.Text = "Velocidade: " .. tostring(speedValor)
        aplicarSpeed()
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        draggingSpeedSlider = false
    end
end)

local btnFechar = Instance.new("TextButton", abaMain)
btnFechar.Size = UDim2.new(1, -40, 0, 32)
btnFechar.Position = UDim2.new(0, 20, 1, -42)
btnFechar.Text = "Fechar"
btnFechar.BackgroundColor3 = Color3.fromRGB(255,120,120)
btnFechar.TextColor3 = Color3.fromRGB(0,0,0)
btnFechar.Font = Enum.Font.GothamBold
btnFechar.TextSize = 16
local btnFecharCorner = Instance.new("UICorner", btnFechar)
btnFecharCorner.CornerRadius = UDim.new(0,8)
btnFechar.MouseButton1Click:Connect(function()
    abaMain.Visible = false
end)

circulo.MouseButton1Click:Connect(function()
    if not dragging then
        abaMain.Visible = not abaMain.Visible
        btnAimbot.Text = "Aimbot: " .. (aimbotAtivo and "LIGADO" or "DESLIGADO")
        btnESP.Text = "ESP Esqueleto: " .. (espEsqueletoAtivo and "LIGADO" or "DESLIGADO")
        btnNickESP.Text = "ESP Nick: " .. (espNickAtivo and "LIGADO" or "DESLIGADO")
        btnInfJump.Text = "Infinite Jump: " .. (infiniteJumpAtivo and "LIGADO" or "DESLIGADO")
        btnSpeed.Text = "Speed: " .. (speedAtivo and "LIGADO" or "DESLIGADO")
        abaMain.Position = UDim2.new(
            circulo.Position.X.Scale,
            circulo.Position.X.Offset + 90,
            circulo.Position.Y.Scale,
            circulo.Position.Y.Offset
        )
    end
end)
