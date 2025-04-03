-- Load libraries
local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
local SaveManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/SaveManager.lua"))()
local InterfaceManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/InterfaceManager.lua"))()

-- Global variables
local isFarming = false
local autoEat = false
local dummyFarmActive = false
local dummyFarmConnection = nil
local TeleportDropdown = nil

_G.attackAllNPCToggle = false
_G.dummyFarm5kEnabled = false
_G.killAura = false
_G.huntPlayers = false
_G.farmLowLevels = false

-- Create window
local Window = Fluent:CreateWindow({
    Title = "Moon HUB (Animal Simulator)",
    SubTitle = "by lk",
    TabWidth = 160,
    Size = UDim2.fromOffset(580, 460),
    Acrylic = true,
    Theme = "Dark",
    MinimizeKey = Enum.KeyCode.LeftControl
})

-- Create tabs
local Tabs = {
    Farm = Window:AddTab({ Title = "Farm", Icon = "package" }),
    PVP = Window:AddTab({ Title = "PvP", Icon = "sword" }),
    Teleport = Window:AddTab({ Title = "Teleport", Icon = "map-pin" }),
    Misc = Window:AddTab({ Title = "Misc", Icon = "box" }),
    Scripts = Window:AddTab({ Title = "Scripts", Icon = "code" }),
    Skins = Window:AddTab({ Title = "Skins", Icon = "shirt" }),
    Settings = Window:AddTab({ Title = "Settings", Icon = "settings" })
}

--[[
    FARM TAB FUNCTIONS
]]

-- Coin Farm function
local function coinFarmLoop()
    while isFarming and task.wait(0.1) do
        pcall(function()
            game:GetService("ReplicatedStorage").Events.CoinEvent:FireServer()
        end)
    end
end

-- Attack All NPCs function
local function attackAllNPCsLoop()
    while _G.attackAllNPCToggle and task.wait(0.01) do
        pcall(function()
            local npcsWithHealth = {}
            
            for _, npc in ipairs(workspace.NPC:GetDescendants()) do
                if npc:IsA("Humanoid") and npc.Health > 0 then
                    table.insert(npcsWithHealth, {
                        humanoid = npc,
                        health = npc.Health
                    })
                end
            end
            
            table.sort(npcsWithHealth, function(a, b)
                return a.health < b.health
            end)
            
            for _, npcData in ipairs(npcsWithHealth) do
                if _G.attackAllNPCToggle then
                    local args = {
                        [1] = npcData.humanoid,
                        [2] = 1
                    }
                    game:GetService("ReplicatedStorage").jdskhfsIIIllliiIIIdchgdIiIIIlIlIli:FireServer(unpack(args))
                end
            end
        end)
    end
end

-- Dummy Farm function
local function dummyFarmFunction()
    if dummyFarmConnection then
        dummyFarmConnection:Disconnect()
        dummyFarmConnection = nil
    end
    
    if dummyFarmActive then
        dummyFarmConnection = game:GetService("RunService").Heartbeat:Connect(function()
            pcall(function()
                local targetDummy = workspace.MAP.dummies:GetChildren()[1]
                if targetDummy and game.Players.LocalPlayer.Character then
                    local humanoid = targetDummy:FindFirstChild("Humanoid")
                    local rootPart = targetDummy:FindFirstChild("HumanoidRootPart")
                    local playerRoot = game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                    
                    if humanoid and rootPart and playerRoot then
                        playerRoot.CFrame = rootPart.CFrame * CFrame.new(0, 8, 0)
                        game:GetService("ReplicatedStorage").jdskhfsIIIllliiIIIdchgdIiIIIlIlIli:FireServer(humanoid, 1)
                    end
                end
            end)
        end)
    end
end

-- Dummy 5k Farm function
local function dummy5kFarmLoop()
    while _G.dummyFarm5kEnabled and task.wait() do
        pcall(function()
            local dummies = workspace.MAP["5k_dummies"]:GetChildren()
            local targetDummy = nil
            local shortestDistance = math.huge
            
            for _, dummy in pairs(dummies) do
                if dummy.Name == "Dummy2" then
                    if dummy:FindFirstChild("Humanoid") and dummy:FindFirstChild("HumanoidRootPart") then
                        local isOccupied = false
                        local dummyRoot = dummy.HumanoidRootPart
                        
                        for _, player in pairs(game.Players:GetPlayers()) do
                            if player.Character and player ~= game.Players.LocalPlayer then
                                local playerRoot = player.Character:FindFirstChild("HumanoidRootPart")
                                if playerRoot and (playerRoot.Position - dummyRoot.Position).Magnitude < 10 then
                                    isOccupied = true
                                    break
                                end
                            end
                        end
                        
                        if not isOccupied then
                            local distance = (game.Players.LocalPlayer.Character.HumanoidRootPart.Position - dummyRoot.Position).Magnitude
                            if distance < shortestDistance then
                                shortestDistance = distance
                                targetDummy = dummy
                            end
                        end
                    end
                end
            end
            
            if targetDummy and game.Players.LocalPlayer.Character then
                local humanoid = targetDummy:FindFirstChild("Humanoid")
                local rootPart = targetDummy:FindFirstChild("HumanoidRootPart")
                local playerRoot = game.Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                
                if humanoid and rootPart and playerRoot then
                    playerRoot.CFrame = rootPart.CFrame * CFrame.new(0, 8, 0)
                    game:GetService("ReplicatedStorage").jdskhfsIIIllliiIIIdchgdIiIIIlIlIli:FireServer(humanoid, 1)
                end
            end
        end)
    end
end

--[[
    FARM TAB UI
]]

-- Coin Farm Toggle
Tabs.Farm:AddToggle("CoinFarmToggle", {
    Title = "💰 Coin Farm",
    Description = "Automatically farms coins",
    Default = false,
    Callback = function(state)
        isFarming = state
        
        if isFarming then
            task.spawn(coinFarmLoop)
            Fluent:Notify({
                Title = "💰 Coin Farm Activated",
                Content = "Coin Farm has been activated.",
                SubContent = "Earning coins automatically.",
                Duration = 1
            })
        else
            Fluent:Notify({
                Title = "💰 Coin Farm Deactivated",
                Content = "Coin Farm has been deactivated.",
                SubContent = "Stopped farming coins.",
                Duration = 1
            })
        end
    end
})

-- Attack All Bosses Toggle
Tabs.Farm:AddToggle("AttackAllBossesToggle", {
    Title = "👹 Attack All Bosses",
    Description = "Automatically attacks all bosses",
    Default = false,
    Callback = function(state)
        _G.attackAllNPCToggle = state
        
        if state then
            task.spawn(attackAllNPCsLoop)
        end
        
        Fluent:Notify({
            Title = "👹 Attack All Bosses",
            Content = state and "Auto attack on all bosses has been activated!" or "Auto attack on all bosses has been deactivated!",
            Duration = 1
        })
    end
})

-- Dummy Farm Toggle
Tabs.Farm:AddToggle("DummyFarmToggle", {
    Title = "🧍🏻 Dummy Farm",
    Description = "Automatically farms dummies",
    Default = false,
    Callback = function(state)
        dummyFarmActive = state
        dummyFarmFunction()
        
        Fluent:Notify({
            Title = "🧍🏻 Dummy Farm " .. (state and "Activated" or "Deactivated"),
            Content = state and "Dummy Farm has been activated!" or "Dummy Farm has been deactivated!",
            Duration = 1
        })
    end
})

-- Dummy 5k Farm Toggle
Tabs.Farm:AddToggle("Dummy5kFarmToggle", {
    Title = "🧍🏻 Dummy 5k Farm",
    Description = "Automatically farms 5k dummies",
    Default = false,
    Callback = function(state)
        _G.dummyFarm5kEnabled = state
        
        if state then
            task.spawn(dummy5kFarmLoop)
        end
        
        Fluent:Notify({
            Title = "🧍🏻 Dummy 5k Farm " .. (state and "Activated" or "Deactivated"),
            Content = state and "Dummy 5k Farm has been activated!" or "Dummy 5k Farm has been deactivated!",
            Duration = 1
        })
    end
})

-- Free Radio Toggle
Tabs.Farm:AddToggle("FreeRadioToggle", {
    Title = "📻 Free Radio", 
    Description = nil,
    Default = false,
    Callback = function(state)
        local gui = game.Players.LocalPlayer:FindFirstChild("PlayerGui")
        if gui and gui:FindFirstChild("DRadio_Gui") then
            gui.DRadio_Gui.Enabled = state
        end
        
        Fluent:Notify({
            Title = "📻 Free Radio",
            Content = state and "Free Radio has been activated!" or "Free Radio has been deactivated!",
            Duration = 1
        })
    end
})

-- Visual 13x Exp Toggle
Tabs.Farm:AddToggle("Visual13xExpToggle", {
    Title = "🔍 Visual 13x Exp", 
    Description = nil,
    Default = false,
    Callback = function(state)
        local gui = game.Players.LocalPlayer:FindFirstChild("PlayerGui")
        if gui and gui:FindFirstChild("LevelBar") and gui.LevelBar:FindFirstChild("gamepassText") then
            gui.LevelBar.gamepassText.Visible = state
            if state then
                gui.LevelBar.gamepassText.Text = "13x exp"
            end
        end
        
        Fluent:Notify({
            Title = "🔍 Visual 13x Exp",
            Content = state and "13x Exp has been activated!" or "13x Exp has been deactivated!",
            Duration = 1
        })
    end
})

--[[
    PVP TAB FUNCTIONS
]]

-- Auto Eat function
local function autoEatLoop()
    local VirtualInputManager = game:GetService("VirtualInputManager")
    
    while autoEat and task.wait(1) do
        pcall(function()
            -- Select food slot
            VirtualInputManager:SendKeyEvent(true, "One", false, game)
            task.wait(0.1)
            VirtualInputManager:SendKeyEvent(false, "One", false, game)
            task.wait(0.1)

            -- Click at screen center
            local screenCenterX = workspace.CurrentCamera.ViewportSize.X * 0.5
            local screenCenterY = workspace.CurrentCamera.ViewportSize.Y * 0.7
            
            VirtualInputManager:SendMouseButtonEvent(screenCenterX, screenCenterY, 0, true, game, 0)
            task.wait(0.05)
            VirtualInputManager:SendMouseButtonEvent(screenCenterX, screenCenterY, 0, false, game, 0)
        end)
    end
end

-- Kill Aura function
local function killAuraLoop()
    local Players = game:GetService("Players")
    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    local localPlayer = Players.LocalPlayer

    while _G.killAura and task.wait(0.01) do
        pcall(function()
            for _, player in ipairs(Players:GetPlayers()) do
                if player ~= localPlayer and player.Character then
                    local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
                    if humanoid and humanoid.Health > 0 and not player.Character:FindFirstChild("SafeZoneShield") then
                        local args = {
                            [1] = humanoid,
                            [2] = 5
                        }
                        ReplicatedStorage.jdskhfsIIIllliiIIIdchgdIiIIIlIlIli:FireServer(unpack(args))
                    end
                end
            end
        end)
    end
end

-- Loop Kill All function
local function loopKillAllPlayers()
    local localPlayer = game.Players.LocalPlayer

    while _G.huntPlayers and task.wait() do
        pcall(function()
            for _, target in ipairs(game.Players:GetPlayers()) do
                if target ~= localPlayer and target.Character and target.Character:FindFirstChild("Humanoid") and 
                   target.Character.Humanoid.Health > 1 and not target.Character:FindFirstChild("SafeZoneShield") then

                    local targetRoot = target.Character:FindFirstChild("HumanoidRootPart")
                    local localRoot = localPlayer.Character and localPlayer.Character:FindFirstChild("HumanoidRootPart")

                    if targetRoot and localRoot then
                        if (localRoot.Position - targetRoot.Position).Magnitude > 10 then
                            localRoot.CFrame = targetRoot.CFrame
                        end

                        local startTime = tick()

                        while target.Character and target.Character:FindFirstChild("Humanoid") and 
                              target.Character.Humanoid.Health > 1 and _G.huntPlayers do

                            if tick() - startTime > 8 then
                                break
                            end

                            local carryArgs = {
                                [1] = target,
                                [2] = "request_accepted"
                            }
                            game:GetService("ReplicatedStorage").Events.CarryEvent:FireServer(unpack(carryArgs))

                            local attackArgs = {
                                [1] = target.Character.Humanoid,
                                [2] = 24
                            }
                            game:GetService("ReplicatedStorage").jdskhfsIIIllliiIIIdchgdIiIIIlIlIli:FireServer(unpack(attackArgs))

                            task.wait()
                        end
                    end
                end
            end
        end)
    end
end

-- Auto Kill Low Levels function
local function autoKillLowLevels()
    local lp = game.Players.LocalPlayer

    while _G.farmLowLevels and task.wait() do
        pcall(function()
            local best = nil
            for _, p in ipairs(game.Players:GetPlayers()) do
                if p ~= lp and p.Character and p:FindFirstChild("leaderstats") and 
                   p.leaderstats.Level.Value < lp.leaderstats.Level.Value and 
                   p.Character:FindFirstChild("HumanoidRootPart") and 
                   p.Character:FindFirstChild("Humanoid") and 
                   p.Character.Humanoid.Health > 1 and 
                   not p.Character:FindFirstChild("SafeZoneShield") and 
                   (not best or p.leaderstats.Level.Value < best.leaderstats.Level.Value) then 
                    best = p 
                end
            end

            if best and lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") then
                local lr, tr = lp.Character.HumanoidRootPart, best.Character.HumanoidRootPart
                if (lr.Position - tr.Position).Magnitude > 10 then 
                    lr.CFrame = tr.CFrame 
                end
                
                game:GetService("ReplicatedStorage").Events.CarryEvent:FireServer(best, "request_accepted")
                game:GetService("ReplicatedStorage").jdskhfsIIIllliiIIIdchgdIiIIIlIlIli:FireServer(best.Character.Humanoid, 24)
            end
        end)
    end
end

--[[
    PVP TAB UI
]]

-- Auto Eat Toggle
Tabs.PVP:AddToggle("AutoEatToggle", {
    Title = "🐟 Auto Eat (PC)",
    Description = "Enables or disables the Auto Eat function",
    Default = false,
    Callback = function(state)
        autoEat = state
        if autoEat then
            task.spawn(autoEatLoop)
        end
        
        Fluent:Notify({
            Title = "🐟 Auto Eat",
            Content = state and "Auto Eat has been activated." or "Auto Eat has been deactivated.",
            Duration = 1
        })
    end
})

-- Kill Aura Toggle
Tabs.PVP:AddToggle("KillAuraToggle", {
    Title = "⚔️ Kill Aura",
    Description = "Enables or disables the Kill Aura function",
    Default = false,
    Callback = function(state)
        _G.killAura = state
        if state then
            task.spawn(killAuraLoop)
        end
        
        Fluent:Notify({
            Title = "⚔️ Kill Aura " .. (state and "Activated" or "Deactivated"),
            Content = state and "Kill Aura is now active." or "Kill Aura is now inactive.",
            Duration = 1
        })
    end
})

-- Loop Kill All Toggle
Tabs.PVP:AddToggle("LoopKillAllToggle", {
    Title = "🤯 Loop Kill All Players",
    Description = "Automatically hunts and kills all players",
    Default = false,
    Callback = function(state)
        _G.huntPlayers = state
        if state then
            task.spawn(loopKillAllPlayers)
        end
        
        Fluent:Notify({
            Title = state and "🤯 Loop Kill Activated" or "🛑 Loop Kill Stopped",
            Content = state and "Now hunting all players!" or "Stopped hunting players.",
            Duration = 1
        })
    end
})

-- Auto Kill Low Levels Toggle
Tabs.PVP:AddToggle("AutoKillLowLevels", {
    Title = "😎 Auto Kill Low Levels",
    Description = "Automatically hunts players with a lower level than you",
    Default = false,
    Callback = function(state)
        _G.farmLowLevels = state
        if state then
            task.spawn(autoKillLowLevels)
        end
        
        Fluent:Notify({
            Title = state and "😎 Auto Kill Low Levels Activated" or "🛑 Auto Kill Low Levels Stopped",
            Content = state and "Hunting lower-level players!" or "Stopped hunting low-level players.",
            Duration = 1
        })
    end
})

Tabs.PVP:AddButton("FreeFireball", {
    Title = "🔥 Free Fireball",
    Description = "Click to get a fireball!",
    Callback = function()
        local tool = Instance.new("Tool")
        tool.Name = "Fireball"
        tool.RequiresHandle = false

        tool.Activated:Connect(function()
            local mouse = game.Players.LocalPlayer:GetMouse()
            local args = {
                [1] = mouse.Hit.p,
                [2] = "NewFireball"
            }
            game:GetService("ReplicatedStorage").SkillsInRS.RemoteEvent:FireServer(unpack(args))
        end)

        tool.Parent = game.Players.LocalPlayer.Backpack
        
        Fluent:Notify({
            Title = "🔥 Fireball Created",
            Content = "The Fireball has been added to your backpack!",
            Duration = 1
        })
    end
})

Tabs.PVP:AddButton("FreeLightningball", {
    Title = "⚡ Free Lightningball",
    Description = "Click to get a Lightning Ball!",
    Callback = function()
        local tool = Instance.new("Tool")
        tool.Name = "Lightning Ball"
        tool.RequiresHandle = false

        tool.Activated:Connect(function()
            local mouse = game.Players.LocalPlayer:GetMouse()
            for i = 1, 3 do
                local args = {
                    [1] = mouse.Hit.p,
                    [2] = "NewLightningball"
                }
                game:GetService("ReplicatedStorage").SkillsInRS.RemoteEvent:FireServer(unpack(args))
                task.wait(0.1)
            end
        end)

        tool.Parent = game.Players.LocalPlayer.Backpack
        
        Fluent:Notify({
            Title = "⚡ Lightningball Created",
            Content = "The Lightningball has been added to your backpack!",
            Duration = 1
        })
    end
})

-- Player Teleport Dropdown
local function updatePlayerList()
    local playerList = {}
    for _, player in pairs(game:GetService("Players"):GetPlayers()) do
        table.insert(playerList, player.Name)
    end
    
    if Tabs.PVP and Tabs.PVP:FindFirstChild("TeleportDropdown") then
        Tabs.PVP:FindFirstChild("TeleportDropdown"):SetValues(playerList)
    end
end

TeleportDropdown = Tabs.PVP:AddDropdown("TeleportDropdown", {
    Title = "🙋🏻 Teleport to Player",
    Description = "Select a player to teleport to",
    Values = {},
    Multi = false,
    Callback = function(selectedPlayer)
        local localPlayer = game:GetService("Players").LocalPlayer
        local targetPlayer = game:GetService("Players"):FindFirstChild(selectedPlayer)

        if targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
            if localPlayer.Character and localPlayer.Character:FindFirstChild("HumanoidRootPart") then
                localPlayer.Character:SetPrimaryPartCFrame(targetPlayer.Character.HumanoidRootPart.CFrame)
                
                Fluent:Notify({
                    Title = "🙋🏻 Teleport Successful",
                    Content = "You have successfully teleported to " .. targetPlayer.Name .. "!",
                    Duration = 3
                })
            end
        else
            Fluent:Notify({
                Title = "Error",
                Content = "Player not found or invalid target!",
                Duration = 3
            })
        end
    end
})

-- Update player list when players join/leave
game:GetService("Players").PlayerAdded:Connect(updatePlayerList)
game:GetService("Players").PlayerRemoving:Connect(updatePlayerList)
updatePlayerList()

--[[
    SETTINGS TAB
]]

Tabs.Settings:AddParagraph({
    Title = "UI Configuration",
    Content = "Customize the interface appearance and behavior"
})

SaveManager:SetLibrary(Fluent)
InterfaceManager:SetLibrary(Fluent)
SaveManager:IgnoreThemeSettings()
SaveManager:SetIgnoreIndexes({})
InterfaceManager:SetFolder("FluentScriptHub")
SaveManager:SetFolder("FluentScriptHub/moonanimalsim")
InterfaceManager:BuildInterfaceSection(Tabs.Settings)
SaveManager:BuildConfigSection(Tabs.Settings)

-- Select first tab and show notification
Window:SelectTab(1)
Fluent:Notify({
    Title = "Script Fully Loaded!",
    Content = "Happy using, remembering that all functions are undetectable!",
    Duration = 2
})
