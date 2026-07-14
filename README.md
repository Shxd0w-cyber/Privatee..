-- Add a tiny delay to ensure the executor is fully ready to handle UI rendering
task.wait(0.2)

local success, Rayfield = pcall(function()
   return loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
end)

if not success or not Rayfield then
   -- Alternative fallback mirror link if the main provider domain stutters
   Rayfield = loadstring(game:HttpGet('https://raw.githubusercontent.com/SiriusSoftwareLtd/Rayfield/main/source.lua'))()
end

-- Main Window Creation
local Window = Rayfield:CreateWindow({
   Name = "Super League Soccer | Made by Akai",
   LoadingTitle = "Loading SLS Script...",
   LoadingSubtitle = "Made by Akai",
   ConfigurationSaving = { Enabled = false },
   Discord = { Enabled = false },
   KeySystem = false
})

-- Tabs
local MainTab = Window:CreateTab("Main", 4483362458)
local CombatTab = Window:CreateTab("Defense & Offense", 4483362458)

-- State Variables (Toggles)
local InfiniteStaminaEnabled = false
local AutoSaveGoleiro = false
local HitboxExpansionEnabled = false
local AutoTackleEnabled = false
local AutoDribbleEnabled = false

local HitboxSize = 15

-- Roblox Services
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

-- ==========================================
-- 1. INFINITE STAMINA LOGIC
-- ==========================================
task.spawn(function()
    local Knit = require(ReplicatedStorage.Packages.Knit)
    local StaminaController = Knit.GetController("StaminaController")
    local originalConsume = StaminaController.Consume
    StaminaController.Consume = function(self, amount)
        if InfiniteStaminaEnabled then
            local States = require(ReplicatedStorage.Shared.States)
            local Defaults = require(ReplicatedStorage.Shared.Defaults.Movement)
            States.Stamina.Amount:set(Defaults.Stamina.Maximum)
            return true
        end
        return originalConsume(self, amount)
    end
end)

-- ==========================================
-- 2. AUTO SAVE AND HITBOX EXPANSION LOGIC
-- ==========================================
RunService.Heartbeat:Connect(function()
    local Character = LocalPlayer.Character
    local RootPart = Character and Character:FindFirstChild("HumanoidRootPart")
    if not RootPart then return end

    local Ball = workspace:FindFirstChild("Football") or workspace:FindFirstChild("Ball")
    if not Ball or not Ball:IsA("BasePart") then return end

    local Distance = (Ball.Position - RootPart.Position).Magnitude

    if HitboxExpansionEnabled then
        Ball.Size = Vector3.new(HitboxSize, HitboxSize, HitboxSize)
        Ball.CanCollide = false
    else
        Ball.Size = Vector3.new(2, 2, 2)
    end

    if AutoSaveGoleiro and Distance < 30 then
        local Knit = require(ReplicatedStorage.Packages.Knit)
        local ActionController = Knit.GetController("ActionController")
        
        if ActionController and not ActionController:IsOnCooldown("Save") then
            ActionController:RequestAction("Save")
            if Distance < 15 then
                Ball.CFrame = RootPart.CFrame * CFrame.new(0, 0, -2)
            end
        end
    end
end)

-- ==========================================
-- 3. AUTO TACKLE LOGIC
-- ==========================================
task.spawn(function()
    while task.wait(0.1) do
        if AutoTackleEnabled then
            local Character = LocalPlayer.Character
            local RootPart = Character and Character:FindFirstChild("HumanoidRootPart")
            
            if RootPart then
                for _, player in pairs(Players:GetPlayers()) do
                    if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                        local hasBall = player:GetAttribute("HasBall")
                        local distance = (player.Character.HumanoidRootPart.Position - RootPart.Position).Magnitude
                        
                        if hasBall and distance < 12 then
                            local Knit = require(ReplicatedStorage.Packages.Knit)
                            local ActionController = Knit.GetController("ActionController")
                            if ActionController and not ActionController:IsOnCooldown("Tackle") then
                                ActionController:RequestAction("Tackle")
                            end
                        end
                    end
                end
            end
        end
    end
end)

-- ==========================================
-- 4. AUTO DRIBBLE LOGIC
-- ==========================================
task.spawn(function()
    while task.wait(0.1) do
        if AutoDribbleEnabled then
            local Character = LocalPlayer.Character
            local RootPart = Character and Character:FindFirstChild("HumanoidRootPart")
            
            if RootPart and LocalPlayer:GetAttribute("HasBall") then
                for _, player in pairs(Players:GetPlayers()) do
                    if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                        local enemyDistance = (player.Character.HumanoidRootPart.Position - RootPart.Position).Magnitude
                        
                        if enemyDistance < 10 then
                            local Knit = require(ReplicatedStorage.Packages.Knit)
                            local ActionController = Knit.GetController("ActionController")
                            if ActionController and not ActionController:IsOnCooldown("Dribble") then
                                ActionController:RequestAction("Dribble")
                                task.wait(0.5)
                            end
                        end
                    end
                end
            end
        end
    end
end)

-- ==========================================
-- CONTROL PANEL ELEMENTS
-- ==========================================
MainTab:CreateToggle({
   Name = "Infinite Stamina",
   CurrentValue = false,
   Flag = "StaminaToggle",
   Callback = function(Value) InfiniteStaminaEnabled = Value end,
})

CombatTab:CreateToggle({
   Name = "Goalkeeper Auto Save (All Balls)",
   CurrentValue = false,
   Flag = "AutoSaveToggle",
   Callback = function(Value)
      AutoSaveGoleiro = Value
      if Value then
          Rayfield:Notify({Name = "GK Mode", Content = "Automated saves activated.", Duration = 3})
      end
   end,
})

CombatTab:CreateToggle({
   Name = "Expand Ball Hitbox",
   CurrentValue = false,
   Flag = "HitboxToggle",
   Callback = function(Value) HitboxExpansionEnabled = Value end,
})

CombatTab:CreateSlider({
   Name = "Hitbox Size Selector",
   Min = 5,
   Max = 50,
   CurrentValue = 15,
   Flag = "HitboxSlider",
   Callback = function(Value) HitboxSize = Value end,
})

CombatTab:CreateToggle({
   Name = "Auto Tackle (Automatic Steal)",
   CurrentValue = false,
   Flag = "TackleToggle",
   Callback = function(Value) AutoTackleEnabled = Value end,
})

CombatTab:CreateToggle({
   Name = "Auto Dribble (Evade Defender)",
   CurrentValue = false,
   Flag = "DribbleToggle",
   Callback = function(Value) AutoDribbleEnabled = Value end,
})

Rayfield:Notify({
   Name = "Successfully Injected!",
   Content = "SLS Hub is running cleanly.",
   Duration = 4,
})
