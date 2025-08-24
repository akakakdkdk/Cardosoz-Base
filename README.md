local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer

local baseActive, basePart = false

-- Função para criar a base
local function createBase()
    if basePart then return end
    local char = player.Character or player.CharacterAdded:Wait()
    local root = char:WaitForChild("HumanoidRootPart")
    
    basePart = Instance.new("Part")
    basePart.Size = Vector3.new(6,1,6)
    basePart.Anchored = true
    basePart.CanCollide = true
    basePart.Material = Enum.Material.Neon
    basePart.Color = Color3.fromRGB(0,0,0)
    basePart.Parent = workspace

    local gui = Instance.new("SurfaceGui", basePart)
    gui.Face = Enum.NormalId.Top
    gui.AlwaysOnTop = true
    local frame = Instance.new("Frame", gui)
    frame.Size = UDim2.new(1,0,1,0)
    frame.BackgroundTransparency = 1
    local gradient = Instance.new("UIGradient", frame)
    gradient.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0,Color3.fromRGB(128,0,128)),
        ColorSequenceKeypoint.new(1,Color3.fromRGB(0,0,0))
    })
    gradient.Rotation = 90

    basePart.Position = root.Position - Vector3.new(0, root.Size.Y/2 + basePart.Size.Y/2, 0)
end

-- Atualizar posição da base
local function updateBase()
    local char = player.Character
    if basePart and char then
        local root = char:FindFirstChild("HumanoidRootPart")
        if root then
            basePart.Position = root.Position - Vector3.new(0, root.Size.Y/2 + basePart.Size.Y/2, 0)
        end
    end
end

-- Função para criar a GUI
local function createGUI()
    if player.PlayerGui:FindFirstChild("Cardosoz Base") then return end

    local gui = Instance.new("ScreenGui")
    gui.Name = "Cardosoz Base"
    gui.Parent = player:WaitForChild("PlayerGui")

    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0, 180, 0, 60)
    -- Aparece no canto direito
    frame.Position = UDim2.new(1, -190, 0.5, -30)
    frame.BackgroundColor3 = Color3.fromRGB(75,0,130)
    frame.BorderSizePixel = 0
    frame.Parent = gui

    local corner = Instance.new("UICorner", frame)
    corner.CornerRadius = UDim.new(0,12)

    local title = Instance.new("TextLabel", frame)
    title.Size = UDim2.new(1, -50, 0, 25)
    title.Position = UDim2.new(0, 10, 0, 5)
    title.BackgroundTransparency = 1
    title.Text = "Cardosoz Base"
    title.TextColor3 = Color3.fromRGB(255,255,255)
    title.TextScaled = true
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Font = Enum.Font.GothamBold

    -- Botão minimizar
    local minimize = false
    local minBtn = Instance.new("TextButton", frame)
    minBtn.Size = UDim2.new(0,25,0,25)
    minBtn.Position = UDim2.new(1, -30, 0, 5)
    minBtn.BackgroundColor3 = Color3.fromRGB(0,0,0)
    minBtn.TextColor3 = Color3.fromRGB(255,255,255)
    minBtn.Text = "-"
    minBtn.Font = Enum.Font.GothamBold
    minBtn.TextScaled = true
    local minCorner = Instance.new("UICorner", minBtn)
    minCorner.CornerRadius = UDim.new(0,5)

    minBtn.MouseButton1Click:Connect(function()
        minimize = not minimize
        for _,v in pairs(frame:GetChildren()) do
            if v ~= minBtn and v ~= title then
                v.Visible = not minimize
            end
        end
        frame.Size = minimize and UDim2.new(0,180,0,35) or UDim2.new(0,180,0,60)
    end)

    -- Botão ativar base
    local baseBtn = Instance.new("TextButton", frame)
    baseBtn.Size = UDim2.new(0,160,0,30)
    baseBtn.Position = UDim2.new(0,10,0,25)
    baseBtn.BackgroundColor3 = Color3.fromRGB(0,0,0)
    baseBtn.TextColor3 = Color3.fromRGB(255,255,255)
    baseBtn.Text = "Ativar Base"
    baseBtn.Font = Enum.Font.GothamBold
    baseBtn.TextScaled = true
    local baseCorner = Instance.new("UICorner", baseBtn)
    baseCorner.CornerRadius = UDim.new(0,8)

    baseBtn.MouseButton1Click:Connect(function()
        baseActive = not baseActive
        if baseActive then 
            createBase() 
            baseBtn.Text = "Desativar Base" 
        else 
            if basePart then basePart:Destroy() end
            basePart = nil
            baseBtn.Text = "Ativar Base"
        end
    end)

    -- Arrastar GUI
    local dragging, dragInput, mousePos, framePos = false
    frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            mousePos = input.Position
            framePos = frame.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    frame.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            dragInput = input
        end
    end)

    RunService.RenderStepped:Connect(function()
        if dragging and dragInput then
            local delta = dragInput.Position - mousePos
            frame.Position = UDim2.new(
                framePos.X.Scale,
                framePos.X.Offset + delta.X,
                framePos.Y.Scale,
                framePos.Y.Offset + delta.Y
            )
        end
        if baseActive then updateBase() end
    end)
end

-- Criar GUI quando o script inicia
createGUI()

-- Reconstruir GUI quando o personagem respawnar
player.CharacterAdded:Connect(function()
    wait(0.5)
    createGUI()
end)
