-- Carregar Rayfield UI
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- Criar Janela
local Window = Rayfield:CreateWindow({
   Name = "Kds Hub",
   LoadingTitle = "Carregando...",
   LoadingSubtitle = "by KDSS",
   ConfigurationSaving = { Enabled = false }
})

-- Abas
local PlayerTab = Window:CreateTab("Player", 4483362458)
local VisualTab = Window:CreateTab("Visual", 4483362464)
local AimbotTab = Window:CreateTab("Aimbot", 4483362470)

-- Serviços e variáveis globais
local player = game.Players.LocalPlayer
local runService = game:GetService("RunService")
local uis = game:GetService("UserInputService")

-- Variáveis Player
local flyNoclip = false
local walkSpeedEnabled = false
local walkSpeedValue = 32
local speedTPEnabled = false
local tpDistance = 5
local tpDelay = 0.01
local tpDirection = {W=false,A=false,S=false,D=false}
local movement = {W=false,A=false,S=false,D=false,Q=false,E=false}
local infinityJumpEnabled = false
local floatEnabled = false
local floatGravity = 0.5 -- quanto menor, mais devagar cai

-- Captura inputs
uis.InputBegan:Connect(function(input, processed)
    if processed then return end
    if tpDirection[input.KeyCode.Name] ~= nil then tpDirection[input.KeyCode.Name] = true end
    if movement[input.KeyCode.Name] ~= nil then movement[input.KeyCode.Name] = true end
end)

uis.InputEnded:Connect(function(input)
    if tpDirection[input.KeyCode.Name] ~= nil then tpDirection[input.KeyCode.Name] = false end
    if movement[input.KeyCode.Name] ~= nil then movement[input.KeyCode.Name] = false end
end)

-- Infinity Jump
uis.JumpRequest:Connect(function()
    if infinityJumpEnabled and player.Character and player.Character:FindFirstChild("Humanoid") then
        player.Character.Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
    end
end)

-------------------------------------------------------------------
-- Player Tab: Fly Noclip, WalkSpeed, Speed TP, Infinity Jump, Float
-------------------------------------------------------------------

-- Fly Noclip
PlayerTab:CreateToggle({
   Name = "Fly Noclip",
   CurrentValue = false,
   Flag = "FlyNoclip",
   Callback = function(Value)
      flyNoclip = Value
   end,
})

-- WalkSpeed
PlayerTab:CreateToggle({
   Name = "WalkSpeed",
   CurrentValue = false,
   Flag = "WalkSpeedToggle",
   Callback = function(Value)
      walkSpeedEnabled = Value
      if player.Character and player.Character:FindFirstChild("Humanoid") then
          player.Character.Humanoid.WalkSpeed = Value and walkSpeedValue or 16
      end
   end,
})

PlayerTab:CreateSlider({
   Name = "WalkSpeed Value",
   Range = {16, 100},
   Increment = 2,
   Suffix = "WalkSpeed",
   CurrentValue = walkSpeedValue,
   Flag = "WalkSpeedValue",
   Callback = function(Value)
      walkSpeedValue = Value
      if walkSpeedEnabled and player.Character and player.Character:FindFirstChild("Humanoid") then
          player.Character.Humanoid.WalkSpeed = walkSpeedValue
      end
   end,
})

-- Speed TP
PlayerTab:CreateToggle({
    Name = "Speed TP",
    CurrentValue = false,
    Flag = "SpeedTPToggle",
    Callback = function(Value)
        speedTPEnabled = Value
    end
})

PlayerTab:CreateSlider({
    Name = "TP Distance",
    Range = {1, 20},
    Increment = 1,
    Suffix = "Studs",
    CurrentValue = tpDistance,
    Flag = "TPDistance",
    Callback = function(Value)
        tpDistance = Value
    end
})

-- Infinity Jump
PlayerTab:CreateToggle({
    Name = "Infinity Jump",
    CurrentValue = false,
    Flag = "InfinityJumpToggle",
    Callback = function(Value)
        infinityJumpEnabled = Value
    end
})

-- Float
PlayerTab:CreateToggle({
    Name = "Float",
    CurrentValue = false,
    Flag = "FloatToggle",
    Callback = function(Value)
        floatEnabled = Value
    end
})

-- Loop Fly Noclip, Speed TP e Float
runService.RenderStepped:Connect(function()
    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local hrp = player.Character.HumanoidRootPart
        local humanoid = player.Character:FindFirstChild("Humanoid")
        local camCF = workspace.CurrentCamera.CFrame

        -- Fly Noclip
        if flyNoclip then
            local moveDir = Vector3.new()
            if movement.W then moveDir += camCF.LookVector end
            if movement.S then moveDir -= camCF.LookVector end
            if movement.A then moveDir -= camCF.RightVector end
            if movement.D then moveDir += camCF.RightVector end
            if movement.E then moveDir += camCF.UpVector end
            if movement.Q then moveDir -= camCF.UpVector end
            if moveDir.Magnitude > 0 then
                hrp.CFrame = hrp.CFrame + (moveDir.Unit * 5)
            end
            for _,v in pairs(player.Character:GetDescendants()) do
                if v:IsA("BasePart") then
                    v.CanCollide = false
                end
            end
        end

        -- Speed TP
        if speedTPEnabled then
            local moveVector = Vector3.new()
            if tpDirection.W then moveVector += camCF.LookVector end
            if tpDirection.S then moveVector -= camCF.LookVector end
            if tpDirection.A then moveVector -= camCF.RightVector end
            if tpDirection.D then moveVector += camCF.RightVector end
            if moveVector.Magnitude > 0 then
                hrp.CFrame = hrp.CFrame + (moveVector.Unit * tpDistance)
                task.wait(tpDelay)
            end
        end

        -- Float
        if floatEnabled and humanoid then
            if humanoid.FloorMaterial == Enum.Material.Air then
                humanoid:ChangeState(Enum.HumanoidStateType.Freefall)
                humanoid.Gravity = floatGravity
            else
                humanoid.Gravity = workspace.Gravity
            end
        end
    end
end)

-------------------------------------------------------------------
-- Visual Tab: ESP Box + ESP Line
-------------------------------------------------------------------
local espEnabled = false
local espLineEnabled = false

local function createESP(plr)
    if plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
        local box = Instance.new("BoxHandleAdornment")
        box.Size = Vector3.new(4, 6, 4)
        box.Color3 = Color3.fromRGB(0, 255, 0)
        box.Transparency = 0.3
        box.AlwaysOnTop = true
        box.ZIndex = 2
        box.Adornee = plr.Character.HumanoidRootPart
        box.Parent = plr.Character.HumanoidRootPart

        local line = Drawing.new("Line")
        line.Thickness = 1
        line.Color = Color3.fromRGB(255, 0, 0)

        runService.RenderStepped:Connect(function()
            if plr.Character and plr.Character:FindFirstChild("Head") and espEnabled then
                local headPos, onScreen = workspace.CurrentCamera:WorldToViewportPoint(plr.Character.Head.Position)
                if onScreen then
                    line.Visible = espLineEnabled
                    if espLineEnabled then
                        line.From = Vector2.new(workspace.CurrentCamera.ViewportSize.X/2, workspace.CurrentCamera.ViewportSize.Y)
                        line.To = Vector2.new(headPos.X, headPos.Y)
                    end
                else
                    line.Visible = false
                end
            else
                line.Visible = false
            end
        end)
    end
end

local function enableESP()
    for _,plr in pairs(game.Players:GetPlayers()) do
        if plr ~= player then
            createESP(plr)
        end
    end
end

game.Players.PlayerAdded:Connect(function(plr)
    plr.CharacterAdded:Connect(function()
        if espEnabled then
            task.wait(1)
            createESP(plr)
        end
    end)
end)

VisualTab:CreateToggle({
   Name = "ESP Box",
   CurrentValue = false,
   Flag = "ESPBox",
   Callback = function(Value)
      espEnabled = Value
      if espEnabled then enableESP() end
   end,
})

VisualTab:CreateToggle({
   Name = "ESP Line",
   CurrentValue = false,
   Flag = "ESPLine",
   Callback = function(Value)
      espLineEnabled = Value
   end,
})

-------------------------------------------------------------------
-- Aimbot Tab: Auto Head Aim + FOV
-------------------------------------------------------------------
local aimbotEnabled = false
local fovValue = 100
local fovCircle = Drawing.new("Circle")
fovCircle.Visible = false
fovCircle.Thickness = 1
fovCircle.Color = Color3.fromRGB(255,0,0)
fovCircle.Filled = false
fovCircle.NumSides = 100

AimbotTab:CreateToggle({
    Name = "Aimbot",
    CurrentValue = false,
    Flag = "AimbotToggle",
    Callback = function(Value)
        aimbotEnabled = Value
        fovCircle.Visible = Value
    end
})

AimbotTab:CreateSlider({
    Name = "FOV",
    Range = {1, 360},
    Increment = 1,
    Suffix = "°",
    CurrentValue = fovValue,
    Flag = "FOVSlider",
    Callback = function(Value)
        fovValue = Value
    end
})

-- Função para encontrar alvo dentro do FOV
local function getClosestTarget()
    local closestPlayer = nil
    local shortestDistance = fovValue
    for _, plr in pairs(game.Players:GetPlayers()) do
        if plr ~= player and plr.Character and plr.Character:FindFirstChild("Head") then
            local headPos, onScreen = workspace.CurrentCamera:WorldToViewportPoint(plr.Character.Head.Position)
            if onScreen then
                local dist = (Vector2.new(headPos.X, headPos.Y) - Vector2.new(workspace.CurrentCamera.ViewportSize.X/2, workspace.CurrentCamera.ViewportSize.Y/2)).Magnitude
                if dist <= shortestDistance then
                    shortestDistance = dist
