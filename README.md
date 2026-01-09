-- POLO PREMIUN (LocalScript)
-- Pega este LocalScript en StarterPlayer > StarterPlayerScripts
-- Hecho para uso LEGÍTIMO dentro de tus propios juegos en Roblox Studio.

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer

-- Esperar al personaje
local function waitCharacter()
    local char = player.Character
    if not char or not char.Parent then
        char = player.CharacterAdded:Wait()
    end
    return char
end

-- --- Config / Estado ---
local state = {
    fly = false,
    speedBoost = false,
    jumpBoost = false,
    flySpeed = 60,
    speedValue = 30,
    jumpPowerValue = 100,
}

local activeObjects = {} -- guardamos BodyVelocity/BodyGyro u otros objetos por personaje

-- --- UI CREATION ---
local function createGui()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "POLO_PREMIUN_GUI"
    screenGui.ResetOnSpawn = false
    screenGui.Parent = player:WaitForChild("PlayerGui")

    local main = Instance.new("Frame")
    main.Name = "Main"
    main.Size = UDim2.new(0, 260, 0, 180)
    main.Position = UDim2.new(0.02, 0, 0.02, 0)
    main.BackgroundColor3 = Color3.fromRGB(22,22,22)
    main.BackgroundTransparency = 0
    main.BorderSizePixel = 0
    main.Parent = screenGui

    local title = Instance.new("TextLabel")
    title.Name = "Title"
    title.Size = UDim2.new(1, -10, 0, 30)
    title.Position = UDim2.new(0, 5, 0, 5)
    title.BackgroundTransparency = 1
    title.Text = "POLO PREMIUN"
    title.Font = Enum.Font.SourceSansBold
    title.TextSize = 20
    title.TextColor3 = Color3.fromRGB(255, 215, 0)
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = main

    local function makeToggle(name, ypos)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0, 120, 0, 36)
        btn.Position = UDim2.new(0, 10 + (ypos * 130), 0, 40 + (ypos * 40))
        btn.BackgroundColor3 = Color3.fromRGB(40,40,40)
        btn.BorderSizePixel = 0
        btn.TextColor3 = Color3.fromRGB(255,255,255)
        btn.Font = Enum.Font.SourceSans
        btn.TextSize = 16
        btn.Text = name .. ": OFF"
        btn.Parent = main
        return btn
    end

    local flyBtn = makeToggle("Fly", 0)
    local speedBtn = makeToggle("Speed", 0.7)
    local jumpBtn = makeToggle("Jump", 1.4)

    local info = Instance.new("TextLabel")
    info.Size = UDim2.new(1, -10, 0, 24)
    info.Position = UDim2.new(0, 5, 1, -30)
    info.BackgroundTransparency = 1
    info.Text = "F = toggle Fly | WASD move | E up Q down"
    info.Font = Enum.Font.SourceSans
    info.TextSize = 14
    info.TextColor3 = Color3.fromRGB(200,200,200)
    info.TextXAlignment = Enum.TextXAlignment.Left
    info.Parent = main

    return {
        ScreenGui = screenGui,
        FlyBtn = flyBtn,
        SpeedBtn = speedBtn,
        JumpBtn = jumpBtn,
        Title = title,
        Info = info,
    }
end

local gui = createGui()

-- --- CORE FEATURES ---
local function enableFly(character)
    if state.fly then return end
    state.fly = true

    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    local bv = Instance.new("BodyVelocity")
    bv.Name = "PoloFly_BV"
    bv.MaxForce = Vector3.new(1e5, 1e5, 1e5)
    bv.Velocity = Vector3.new(0,0,0)
    bv.Parent = hrp

    local bg = Instance.new("BodyGyro")
    bg.Name = "PoloFly_BG"
    bg.MaxTorque = Vector3.new(1e5, 1e5, 1e5)
    bg.CFrame = hrp.CFrame
    bg.Parent = hrp

    activeObjects.BV = bv
    activeObjects.BG = bg

    gui.FlyBtn.Text = "Fly: ON"
    gui.Title.Text = "POLO PREMIUN — Fly: ON"
end

local function disableFly(character)
    if not state.fly then return end
    state.fly = false

    if activeObjects.BV and activeObjects.BV.Parent then activeObjects.BV:Destroy() end
    if activeObjects.BG and activeObjects.BG.Parent then activeObjects.BG:Destroy() end
    activeObjects.BV = nil
    activeObjects.BG = nil

    gui.FlyBtn.Text = "Fly: OFF"
    gui.Title.Text = "POLO PREMIUN"
end

local function toggleFly()
    local char = player.Character or player.CharacterAdded:Wait()
    if state.fly then
        disableFly(char)
    else
        enableFly(char)
    end
end

local function enableSpeed(character)
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    humanoid.WalkSpeed = state.speedValue
    state.speedBoost = true
    gui.SpeedBtn.Text = "Speed: ON"
end

local function disableSpeed(character)
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    humanoid.WalkSpeed = 16 -- valor por defecto de Roblox
    state.speedBoost = false
    gui.SpeedBtn.Text = "Speed: OFF"
end

local function toggleSpeed()
    local char = player.Character or player.CharacterAdded:Wait()
    if state.speedBoost then
        disableSpeed(char)
    else
        enableSpeed(char)
    end
end

local function enableJump(character)
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    humanoid.JumpPower = state.jumpPowerValue
    state.jumpBoost = true
    gui.JumpBtn.Text = "Jump: ON"
end

local function disableJump(character)
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    humanoid.JumpPower = 50 -- valor por defecto (puede variar)
    state.jumpBoost = false
    gui.JumpBtn.Text = "Jump: OFF"
end

local function toggleJump()
    local char = player.Character or player.CharacterAdded:Wait()
    if state.jumpBoost then
        disableJump(char)
    else
        enableJump(char)
    end
end

-- --- FLY MOVEMENT (se ejecuta cada frame si fly=true) ---
RunService.Heartbeat:Connect(function(delta)
    if not state.fly then return end
    local char = player.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    local cam = workspace.CurrentCamera
    if not hrp or not cam then return end
    local bv = activeObjects.BV
    local bg = activeObjects.BG
    if not bv or not bg then return end

    -- Obtener dirección según WASD
    local moveVec = Vector3.new(0,0,0)
    local forward = UserInputService:IsKeyDown(Enum.KeyCode.W) and 1 or 0
    local backward = UserInputService:IsKeyDown(Enum.KeyCode.S) and 1 or 0
    local left = UserInputService:IsKeyDown(Enum.KeyCode.A) and 1 or 0
    local right = UserInputService:IsKeyDown(Enum.KeyCode.D) and 1 or 0
    local up = UserInputService:IsKeyDown(Enum.KeyCode.E) and 1 or 0
    local down = UserInputService:IsKeyDown(Enum.KeyCode.Q) and 1 or 0

    local camLook = Vector3.new(cam.CFrame.LookVector.X, 0, cam.CFrame.LookVector.Z)
    if camLook.Magnitude > 0 then camLook = camLook.Unit end
    local camRight = Vector3.new(cam.CFrame.RightVector.X, 0, cam.CFrame.RightVector.Z)
    if camRight.Magnitude > 0 then camRight = camRight.Unit end

    if forward == 1 then moveVec = moveVec + camLook end
    if backward == 1 then moveVec = moveVec - camLook end
    if left == 1 then moveVec = moveVec - camRight end
    if right == 1 then moveVec = moveVec + camRight end

    local verticalVel = (up - down) * (state.flySpeed * 0.8)
    if moveVec.Magnitude > 0 then
        moveVec = moveVec.Unit * state.flySpeed
    else
        moveVec = Vector3.new(0,0,0)
    end

    local finalVel = Vector3.new(moveVec.X, verticalVel, moveVec.Z)
    bv.Velocity = finalVel

    -- Orientar hacia la cámara horizontalmente
    bg.CFrame = CFrame.new(hrp.Position, hrp.Position + Vector3.new(cam.CFrame.LookVector.X, 0, cam.CFrame.LookVector.Z))
end)

-- --- BUTTON EVENTS ---
gui.FlyBtn.MouseButton1Click:Connect(function()
    toggleFly()
end)
gui.SpeedBtn.MouseButton1Click:Connect(function()
    toggleSpeed()
end)
gui.JumpBtn.MouseButton1Click:Connect(function()
    toggleJump()
end)

-- --- Keybind para Fly (tecla F) ---
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.F then
        toggleFly()
    end
end)

-- --- Mantener estado cuando reaparece el personaje ---
local function onCharacterAdded(char)
    -- limpieza de objetos antiguos
    if activeObjects.BV and activeObjects.BV.Parent then activeObjects.BV:Destroy() end
    if activeObjects.BG and activeObjects.BG.Parent then activeObjects.BG:Destroy() end
    activeObjects = {}

    -- aplicar speed/jump si estaban activos
    if state.speedBoost then
        local humanoid = char:FindFirstChildOfClass("Humanoid")
        if humanoid then humanoid.WalkSpeed = state.speedValue end
    end
    if state.jumpBoost then
        local humanoid = char:FindFirstChildOfClass("Humanoid")
        if humanoid then humanoid.JumpPower = state.jumpPowerValue end
    end

    -- si fly estaba activo, volver a habilitarlo
    if state.fly then
        enableFly(char)
    end
end

player.CharacterAdded:Connect(onCharacterAdded)

-- Si ya hay personaje al ejecutar script, aplicar estados actuales (por ejemplo después de reinicio en Studio)
local initialChar = player.Character or player.CharacterAdded:Wait()
if state.speedBoost then
    local h = initialChar:FindFirstChildOfClass("Humanoid")
    if h then h.WalkSpeed = state.speedValue end
end
if state.jumpBoost then
    local h = initialChar:FindFirstChildOfClass("Humanoid")
    if h then h.JumpPower = state.jumpPowerValue end
end

print("POLO PREMIUN cargado. Presiona F para alternar Fly. Usa los botones en pantalla para Speed y Jump.")
