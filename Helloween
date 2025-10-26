-- MM2 Script Hub V1.4 (WindUI + merged external logic)
-- Author: Yuki

-- Load WindUI safely
local success, WindUI = pcall(function()
    print("Attempting to load WindUI...")
    local result = loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()
    print("WindUI loaded successfully.")
    return result
end)

if not success or not WindUI then
    warn("Failed to load WindUI: " .. tostring(WindUI))
    game:GetService("Players").LocalPlayer:Kick("Failed to load UI library. Please check your network or the script URL.")
    return
end

print("Script running on client at: " .. os.date("%H:%M:%S %d/%m/%Y"))

-- Services
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local VirtualUser = game:GetService("VirtualUser")

local plr = Players.LocalPlayer
local isMobile = UserInputService.TouchEnabled and not UserInputService.KeyboardEnabled
local windowSize = isMobile and UDim2.fromOffset(360, 420) or UDim2.fromOffset(600, 480)

-- =========================
-- WindUI SAFE FOLDER FIX
-- =========================
local ASSET_ROOT = "WindUI"          -- no leading slash
local APP_FOLDER  = ASSET_ROOT .. "/MM2"

pcall(function()
    if typeof(isfolder) == "function" and not isfolder(ASSET_ROOT) then makefolder(ASSET_ROOT) end
    if typeof(isfolder) == "function" and not isfolder(APP_FOLDER)  then makefolder(APP_FOLDER)  end
end)

-- Create Window
local Window = WindUI:CreateWindow({
    Title = "MM2 Script Hub V1.5",
    Icon = "skull",
    Author = "Made by Yuki",
    Folder = APP_FOLDER,
    Size = windowSize,
    Transparent = false,
    Theme = "Dark",
    SideBarWidth = 200,
    User = {
        Enabled = true,
        Anonymous = true,
        Callback = function() print("User profile triggered.") end,
    },
})

-- Tabs
local Tabs = {
    AutoFarmTab  = Window:Tab({ Title = "AutoFarm",  Icon = "coins" }),
    AntiAFKTab   = Window:Tab({ Title = "Anti-AFK",  Icon = "moon"  }),
    AntiStealTab = Window:Tab({ Title = "Anti-Steal",Icon = "shield"})
}

-- Header
Tabs.AutoFarmTab:Paragraph({
    Title = '<font color="#ffdcb5">MM2 Script Hub</font> <font color="#FA4007">| HALLOWEEN! 🎃</font>',
    Desc = "💻 Made by Yuki",
    Image = "zap",
    RichText = true,
})

-- ===== Character / parts =====
local character = plr.Character or plr.CharacterAdded:Wait()
local humPart = character:WaitForChild("HumanoidRootPart")
plr.CharacterAdded:Connect(function(char)
    character = char
    humPart = char:WaitForChild("HumanoidRootPart")
    visitedPositions = {}
    -- reattach sound to new root
    if collectSound then
        collectSound.Parent = humPart
    end
end)

-- ===== External-logic variables (merged) =====
local visitedPositions = {}
local collected = 0
local startTime = 0
local speed = 15
getgenv()._farmSpeed = speed

-- ===== Sound (from external script) =====
local collectSound = Instance.new("Sound")
collectSound.SoundId = "rbxassetid://12221967"
collectSound.Volume = 1
collectSound.Parent = humPart

-- ===== Helpers merged from external script =====
local function flyTo(pos, spd)
    if not humPart then return end
    local distance = (pos - humPart.Position).Magnitude
    local duration = math.max(0.05, distance / (spd or speed))
    local tweenInfo = TweenInfo.new(duration, Enum.EasingStyle.Linear)
    local goal = {CFrame = CFrame.new(pos)}
    local tween = TweenService:Create(humPart, tweenInfo, goal)
    tween:Play()
    tween.Completed:Wait()
end

-- Continuous noclip while farming (like RunService.Stepped in external)
local noclipEnabled = false
RunService.Stepped:Connect(function()
    if noclipEnabled and character then
        for _, v in ipairs(character:GetDescendants()) do
            if v:IsA("BasePart") then
                v.CanCollide = false
            end
        end
    end
end)

-- ===== AutoFarm Toggle (keeps your design, swaps core loop to external logic) =====
Tabs.AutoFarmTab:Toggle({
    Title = "Enable Coin/Ball Farm",
    Default = false,
    Callback = function(state)
        getgenv().farm = state
        if state then
            collected = 0
            startTime = tick()
            visitedPositions = {}
            noclipEnabled = true

            WindUI:Notify({
                Title = "AutoFarm",
                Content = "Started farming...",
                Icon = "check",
                Duration = 4,
            })

            -- UI live stats (unchanged behavior)
            task.spawn(function()
                while getgenv().farm do
                    local elapsed = tick() - startTime
                    Tabs.AutoFarmTab:Label("Collected: " .. collected)
                    Tabs.AutoFarmTab:Label("Time Active: " .. math.floor(elapsed) .. "s")
                    Tabs.AutoFarmTab:Label("Coins/Hour: " .. math.floor((collected / math.max(1, elapsed)) * 3600))
                    task.wait(0.5)
                end
            end)

            -- === MAIN FARM LOOP (ported from the other script) ===
            task.spawn(function()
                while getgenv().farm do
                    character = plr.Character or plr.CharacterAdded:Wait()
                    humPart = character:FindFirstChild("HumanoidRootPart")
                    if humPart then
                        local closest, shortest = nil, math.huge
                        for _, obj in ipairs(workspace:GetDescendants()) do
                            -- External script targets any Coin_Server; keep that, so it works broadly
                            -- (If you want BeachBall only, add: and obj:GetAttribute("CoinID") == "BeachBall")
                            if obj:IsA("BasePart") and obj.Name == "Coin_Server" and not visitedPositions[obj] then
                                local dist = (obj.Position - humPart.Position).Magnitude
                                if dist < shortest and dist < 250 then
                                    closest = obj
                                    shortest = dist
                                end
                            end
                        end

                        if closest and closest.Parent and closest:IsDescendantOf(workspace) then
                            flyTo(closest.Position, getgenv()._farmSpeed or speed)
                            if closest and closest.Parent and closest:IsDescendantOf(workspace) then
                                visitedPositions[closest] = true
                                collected += 1
                                collectSound:Play()
                            end
                        end
                    end
                    task.wait(0.1)
                end
            end)
        else
            noclipEnabled = false
            WindUI:Notify({
                Title = "AutoFarm",
                Content = "Stopped farming.",
                Icon = "x",
                Duration = 4,
            })
        end
    end,
})

-- Speed input (preserves your UI, updates merged logic speed too)
Tabs.AutoFarmTab:Input({
    Title = "Fly Speed",
    Desc = "How fast to fly to coins (default 15)",
    Placeholder = "e.g. 15",
    Callback = function(val)
        local num = tonumber(val)
        if num then
            speed = math.clamp(num, 5, 50)
            getgenv()._farmSpeed = speed
            WindUI:Notify({
                Title = "Speed Updated",
                Content = "Flying speed set to " .. speed,
                Icon = "zap",
                Duration = 3,
            })
        else
            WindUI:Notify({
                Title = "Invalid Input",
                Content = "Please enter a number.",
                Icon = "alert-triangle",
                Duration = 4,
            })
        end
    end,
})

-- Anti-AFK (your original UI button, logic compatible with the other script)
Tabs.AntiAFKTab:Button({
    Title = "Enable Anti-AFK",
    Callback = function()
        local GC = getconnections or get_signal_cons
        if GC then
            for _,v in pairs(GC(plr.Idled)) do
                if v.Disable then v:Disable() elseif v.Disconnect then v:Disconnect() end
            end
        else
            local vu = cloneref and cloneref(VirtualUser) or VirtualUser
            plr.Idled:Connect(function()
                vu:CaptureController()
                vu:ClickButton2(Vector2.new())
            end)
        end
        WindUI:Notify({
            Title = "Anti-AFK Enabled",
            Content = "You won't get kicked for idling!",
            Icon = "coffee",
            Duration = 5,
        })
    end,
})

-- Anti-Steal (unchanged visual)
Tabs.AntiStealTab:Paragraph({
    Title = "Anti-Steal System",
    Desc = "Protects your coins or data from being hijacked by other scripts or players. Toggle below to enable.",
    Image = "lock",
})

local antiStealActive = false
Tabs.AntiStealTab:Toggle({
    Title = "Enable Anti-Steal",
    Default = false,
    Callback = function(state)
        antiStealActive = state
        WindUI:Notify({
            Title = "Anti-Steal",
            Content = state and "Anti-Steal Enabled!" or "Anti-Steal Disabled.",
            Icon = state and "shield" or "x",
            Duration = 4,
        })
    end,
})

print("✅ MM2 GUI loaded. Press Right Ctrl to toggle it.")
