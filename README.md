-- Carregar Rayfield UI
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- Criar Janela
local Window = Rayfield:CreateWindow({
   Name = "Menu Custom",
   LoadingTitle = "Carregando...",
   LoadingSubtitle = "by Kaio",
   ConfigurationSaving = { Enabled = false }
})

-- Abas
local PlayerTab = Window:CreateTab("Player", 4483362458)
local VisualTab = Window:CreateTab("Visual", 4483362464)
local AimbotTab = Window:CreateTab("Aimbot", 4483362470)
local MovementTab = Window:CreateTab("Movement", 4483362475)

-- Serviços e variáveis globais
local player = game.Players.LocalPlayer
local runService = game:GetService("RunService")
local uis = game:GetService("UserInputService")

-- Variáveis de funcionalidades
local flyNoclip = false
local walkSpeedEnabled = false
local walkSpeedValue = 32
local espEnabled = false
local espLineEnabled = false
local aimbotEnabled = false
local fovValue = 100
local speedTPEnabled = false
local tpDistance = 6 -- distância de cada mini tp
local tpDelay = 0.02 -- delay entre mini tps

-- Movimentação Fly Noclip
local movement = {W=false,A=false,S=false,D=false,Q=false,E=false}
uis.InputBegan:Connect(function(input, processed)
    if processed then return end
    if movement[input.KeyCode.Name] ~= nil then
        movement[input.KeyCode.Name] = true
    end
end)
uis.InputEnded:Connect(function(input)
    if movement[input.KeyCode.Name] ~= nil then
        movement[input.KeyCode.Name] = false
    end
end)

-- Loop Fly Noclip
runService.RenderStepped:Connect(function()
    if flyNoclip and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local hrp = player.Character.HumanoidRootPart
        local camCF = workspace.CurrentCamera.CFrame
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
end)

-- Fly Noclip Toggle
PlayerTab:CreateToggle({
   Name = "Fly Noclip",
   CurrentValue = false,
   Flag = "FlyNoclip",
   Callback = function(Value)
      flyNoclip = Value
   end,
})

-------------------------------------------------------------------
-- WALK SPEED
-------------------------------------------------------------------
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

-------------------------------------------------------------------
-- ESP (Box + Line)
-------------------------------------------------------------------
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
-- AIMBOT MOBILE
-------------------------------------------------------------------
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

-- Função para achar alvo mais próximo no FOV
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
                    closestPlayer = plr
                end
            end
        end
    end
    return closestPlayer
end

-- Loop aimbot
runService.RenderStepped:Connect(function()
    if aimbotEnabled and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local target = getClosestTarget()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            workspace.CurrentCamera.CFrame = CFrame.new(workspace.CurrentCamera.CFrame.Position, target.Character.Head.Position)
        end
        if aimbotEnabled then
            fovCircle.Position = Vector2.new(workspace.CurrentCamera.ViewportSize.X/2, workspace.CurrentCamera.ViewportSize.Y/2)
            fovCircle.Radius = fovValue
        end
    end
end)

-------------------------------------------------------------------
-- SPEED TP (mini teleports)
-------------------------------------------------------------------
local tpDirection = {W=false,A=false,S=false,D=false}

uis.InputBegan:Connect(function(input, processed)
    if processed then return end
    if tpDirection[input.KeyCode.Name] ~= nil then
        tpDirection[input.KeyCode.Name] = true
    end
end)
uis.InputEnded:Connect(function(input)
    if tpDirection[input.KeyCode.Name] ~= nil then
        tpDirection[input.KeyCode.Name] = false
    end
end)

MovementTab:CreateToggle({
    Name = "Speed TP",
    CurrentValue = false,
    Flag = "SpeedTPToggle",
    Callback = function(Value)
        speedTPEnabled = Value
    end
})

MovementTab:CreateSlider({
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

-- Loop Speed TP
runService.RenderStepped:Connect(function()
    if speedTPEnabled and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local hrp = player.Character.HumanoidRootPart
        local camCF = workspace.CurrentCamera.CFrame
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
end)
