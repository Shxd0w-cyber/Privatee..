-- Wait for the game to fully load
repeat task.wait() until game:IsLoaded()

-- Load the lightweight Kavo UI library
local Kavo = loadstring(game:HttpGet("https://raw.githubusercontent.com/xHeptc/Kavo-UI-Library/main/source.lua"))()

-- Create the Window (Compact layout for mobile screens)
local Window = Kavo:CreateTable("SLS Hub", "Made by Akai")

-- Create the Tabs
local MainTab = Window:NewTab("Main")
local CombatTab = Window:NewTab("Offense & Defense")

-- Create Sections
local MovementSection = MainTab:NewSection("Player Exploits")
local MatchSection = CombatTab:NewSection("Gameplay Modifiers")

-- State Variables
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
-- FEATURE LOGIC
-- ==========================================

-- 1. Infinite Stamina
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

-- 2. Auto Save & Hitbox Expansion
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

-- 3. Auto Tackle
task.spawn(function()
    while task.wait(0.1) do
        if AutoTackleEnabled then
            local Character = LocalPlayer.Character
            local RootPart = Character and Character:FindFirstChild("HumanoidRootPart")
            if RootPart then
                for _, player in pairs(Players:GetPlayers()) do
                    if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                        if player:GetAttribute("HasBall") and (player.Character.HumanoidRootPart.Position - RootPart.Position).Magnitude < 12 then
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

-- 4. Auto Dribble
task.spawn(function()
    while task.wait(0.1) do
        if AutoDribbleEnabled then
            local Character = LocalPlayer.Character
            local RootPart = Character and Character:FindFirstChild("HumanoidRootPart")
            if RootPart and LocalPlayer:GetAttribute("HasBall") then
                for _, player in pairs(Players:GetPlayers()) do
                    if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                        if (player.Character.HumanoidRootPart.Position - RootPart.Position).Magnitude < 10 then
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
-- KAVO UI TOGGLES & INTERFACE
-- ==========================================

MovementSection:NewToggle("Infinite Stamina", "Gives you unlimited sprint energy", function(state)
    InfiniteStaminaEnabled = state
end)

MatchSection:NewToggle("Goalkeeper Auto Save", "Automatically blocks incoming shots near the box", function(state)
    AutoSaveGoleiro = state
end)

MatchSection:NewToggle("Expand Ball Hitbox", "Makes the soccer ball bigger on your screen", function(state)
    HitboxExpansionEnabled = state
end)

MatchSection:NewSlider("Hitbox Size Selector", "Adjust the size of the ball hitbox", 50, 5, function(value)
    HitboxSize = value
end)

MatchSection:NewToggle("Auto Tackle", "Instantly slides and challenges ball carriers near you", function(state)
    AutoTackleEnabled = state
end)

MatchSection:NewToggle("Auto Dribble", "Evades incoming defenders automatically when you have the ball", function(state)
    AutoDribbleEnabled = state
end)

print("SLS Kavo Menu Injected Successfully | Made by Akai")
