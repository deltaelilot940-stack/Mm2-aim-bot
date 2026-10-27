-- Services

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

local localPlayer = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- System State

local systemEnabled = true
local DETECTION_RADIUS = 35 -- Studs range to engage knife wielders

local FIRE_COOLDOWN = 0.3 -- Revolver fire rate

local lastShotTime = 0

-- UI Setup

local playerGui = localPlayer:WaitForChild("PlayerGui")

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AutoDefenseHUD"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

-- Top Status Banner

local statusBanner = Instance.new("Frame")
statusBanner.Name = "StatusBanner"
statusBanner.Size = UDim2.new(0, 420, 0, 56)
statusBanner.Position = UDim2.new(0.5, -210, 0, 20)
statusBanner.BackgroundColor3 = Color3.fromRGB(18, 20, 28)
statusBanner.BackgroundTransparency = 0.2
statusBanner.BorderSizePixel = 0
statusBanner.Parent = screenGui

local bannerCorner = Instance.new("UICorner")
bannerCorner.CornerRadius = UDim.new(0, 10)
bannerCorner.Parent = statusBanner

local bannerStroke = Instance.new("UIStroke")
bannerStroke.Color = Color3.fromRGB(50, 150, 255)
bannerStroke.Thickness = 1.5
bannerStroke.Parent = statusBanner

local statusIcon = Instance.new("TextLabel")
statusIcon.Name = "Icon"
statusIcon.Size = UDim2.new(0, 44, 1, 0)
statusIcon.BackgroundTransparency = 1
statusIcon.Text = "🛡️"
statusIcon.TextSize = 24
statusIcon.Parent = statusBanner

local statusTitle = Instance.new("TextLabel")
statusTitle.Name = "Title"
statusTitle.Size = UDim2.new(1, -145, 0.5, 0)
statusTitle.Position = UDim2.new(0, 48, 0.08, 0)
statusTitle.BackgroundTransparency = 1
statusTitle.Text = "AUTO-DEFENSE: ACTIVE"
statusTitle.TextColor3 = Color3.fromRGB(80, 200, 255)
statusTitle.Font = Enum.Font.GothamBold
statusTitle.TextSize = 13
statusTitle.TextXAlignment = Enum.TextXAlignment.Left
statusTitle.Parent = statusBanner

local statusDesc = Instance.new("TextLabel")
statusDesc.Name = "Desc"
statusDesc.Size = UDim2.new(1, -145, 0.45, 0)
statusDesc.Position = UDim2.new(0, 48, 0.52, 0)
statusDesc.BackgroundTransparency = 1
statusDesc.Text = "Scanning perimeter (35 studs) for knife threats..."
statusDesc.TextColor3 = Color3.fromRGB(180, 190, 205)
statusDesc.Font = Enum.Font.Gotham
statusDesc.TextSize = 11
statusDesc.TextXAlignment = Enum.TextXAlignment.Left
statusDesc.Parent = statusBanner

-- Toggle Button (Right side of banner)

local toggleBtn = Instance.new("TextButton")
toggleBtn.Name = "ToggleBtn"
toggleBtn.Size = UDim2.new(0, 85, 0, 34)
toggleBtn.Position = UDim2.new(1, -95, 0.5, -17)
toggleBtn.BackgroundColor3 = Color3.fromRGB(30, 80, 150)
toggleBtn.BorderSizePixel = 0
toggleBtn.Text = "ENABLED [V]"
toggleBtn.TextColor3 = Color3.fromRGB(220, 240, 255)
toggleBtn.Font = Enum.Font.GothamBold
toggleBtn.TextSize = 11
toggleBtn.Parent = statusBanner

local toggleBtnCorner = Instance.new("UICorner")
toggleBtnCorner.CornerRadius = UDim.new(0, 6)
toggleBtnCorner.Parent = toggleBtn

local function toggleSystem()
    systemEnabled = not systemEnabled
    localPlayer:SetAttribute("AutoDefenseEnabled", systemEnabled)
    if systemEnabled then
        toggleBtn.BackgroundColor3 = Color3.fromRGB(30, 80, 150)
        toggleBtn.Text = "ENABLED [V]"
        bannerStroke.Color = Color3.fromRGB(50, 150, 255)
        statusIcon.Text = "🛡️"
        statusTitle.Text = "AUTO-DEFENSE: ACTIVE"
        statusTitle.TextColor3 = Color3.fromRGB(80, 200, 255)
        statusDesc.Text = "Scanning perimeter (35 studs) for knife threats..."
        statusDesc.TextColor3 = Color3.fromRGB(180, 190, 205)
    else
        toggleBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 70)
        toggleBtn.Text = "DISABLED [V]"
        bannerStroke.Color = Color3.fromRGB(120, 120, 130)
        statusIcon.Text = "⚠️"
        statusTitle.Text = "AUTO-DEFENSE: DISABLED"
        statusTitle.TextColor3 = Color3.fromRGB(180, 180, 190)
        statusDesc.Text = "System paused. Press V or click button to re-enable."
        statusDesc.TextColor3 = Color3.fromRGB(140, 140, 150)
    end
end

toggleBtn.MouseButton1Click:Connect(toggleSystem)
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
        if input.KeyCode == Enum.KeyCode.V then
            toggleSystem()
        end
    end)
    
    -- Lock-on Reticle

    local reticle = Instance.new("Frame")
    reticle.Name = "Reticle"
    reticle.Size = UDim2.new(0, 80, 0, 80)
    reticle.AnchorPoint = Vector2.new(0.5, 0.5)
    reticle.BackgroundTransparency = 1
    reticle.Visible = false
    reticle.Parent = screenGui
    
    local function makeBracket(pos, size)
        local f = Instance.new("Frame")
        f.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
        f.BorderSizePixel = 0
        f.Position = pos
        f.Size = size
        f.Parent = reticle
        return f
    end
    
    makeBracket(UDim2.new(0, 0, 0, 0), UDim2.new(0, 18, 0, 3))
    makeBracket(UDim2.new(0, 0, 0, 0), UDim2.new(0, 3, 0, 18))
    makeBracket(UDim2.new(1, -18, 0, 0), UDim2.new(0, 18, 0, 3))
    makeBracket(UDim2.new(1, -3, 0, 0), UDim2.new(0, 3, 0, 18))
    makeBracket(UDim2.new(0, 0, 1, -3), UDim2.new(0, 18, 0, 3))
    makeBracket(UDim2.new(0, 0, 1, -18), UDim2.new(0, 3, 0, 18))
    makeBracket(UDim2.new(1, -18, 1, -3), UDim2.new(0, 18, 0, 3))
    makeBracket(UDim2.new(1, -3, 1, -18), UDim2.new(0, 3, 0, 18))
    
    local centerDot = Instance.new("Frame")
    centerDot.Size = UDim2.new(0, 6, 0, 6)
    centerDot.AnchorPoint = Vector2.new(0.5, 0.5)
    centerDot.Position = UDim2.new(0.5, 0, 0.5, 0)
    centerDot.BackgroundColor3 = Color3.fromRGB(255, 60, 60)
    centerDot.BorderSizePixel = 0
    centerDot.Parent = reticle
    local dotCorner = Instance.new("UICorner")
    dotCorner.CornerRadius = UDim.new(1, 0)
    dotCorner.Parent = centerDot
    
    local targetLabel = Instance.new("TextLabel")
    targetLabel.Name = "TargetLabel"
    targetLabel.Size = UDim2.new(0, 240, 0, 20)
    targetLabel.AnchorPoint = Vector2.new(0.5, 0)
    targetLabel.Position = UDim2.new(0.5, 0, 1, 8)
    targetLabel.BackgroundTransparency = 1
    targetLabel.Text = "[LOCKED: KNIFE THREAT]"
    targetLabel.TextColor3 = Color3.fromRGB(255, 60, 60)
    targetLabel.Font = Enum.Font.GothamBold
    targetLabel.TextSize = 11
    targetLabel.Parent = reticle
    
    -- Helper: Check if model has a knife

    local function hasKnife(model)
        if not model or not model:IsA("Model") then return false end
            
            -- Equipped knife tool

            for _, item in ipairs(model:GetChildren()) do
                if item:IsA("Tool") and string.find(item.Name:lower(), "knife") then
                    return true
                end
            end
            
            -- Attributes

            if model:GetAttribute("HasKnife") == true or model:GetAttribute("WeaponType") == "Knife" then
                return true
            end
            
            -- If it's a Player, check backpack

            local targetPlayer = Players:GetPlayerFromCharacter(model)
            if targetPlayer and targetPlayer:FindFirstChild("Backpack") then
                for _, item in ipairs(targetPlayer.Backpack:GetChildren()) do
                    if item:IsA("Tool") and string.find(item.Name:lower(), "knife") then
                        return true
                    end
                end
            end
            
            -- Named Knife child

            if model:FindFirstChild("Knife") then
                return true
            end
            
            return false
        end
        
        -- Helper: Find nearest knife threat

        local function findNearestKnifeThreat(localHrp)
            local closestModel = nil
            local closestDist = DETECTION_RADIUS
            
            for _, obj in ipairs(workspace:GetChildren()) do
                if obj:IsA("Model") and obj ~= localPlayer.Character then
                    local hum = obj:FindFirstChildOfClass("Humanoid")
                    local hrp = obj:FindFirstChild("HumanoidRootPart")
                    if hum and hum.Health > 0 and hrp then
                        local dist = (hrp.Position - localHrp.Position).Magnitude
                        if dist <= closestDist then
                            if hasKnife(obj) then
                                closestDist = dist
                                closestModel = obj
                            end
                        end
                    end
                end
            end
            return closestModel, closestDist
        end
        
        -- Helper: Auto-equip Gun

        local function ensureGunEquipped(character, humanoid)
            local currentTool = character:FindFirstChildOfClass("Tool")
            if currentTool and string.find(currentTool.Name:lower(), "gun") then
                return currentTool
            end
            
            local backpack = localPlayer:FindFirstChild("Backpack")
            if backpack then
                for _, item in ipairs(backpack:GetChildren()) do
                    if item:IsA("Tool") and string.find(item.Name:lower(), "gun") then
                        humanoid:EquipTool(item)
                        return item
                    end
                end
            end
            return nil
        end
        
        -- Camera Lock RenderStep

        RunService:BindToRenderStep("AutoDefenseLock", Enum.RenderPriority.Camera.Value + 1, function()
            local char = localPlayer.Character
            if not char or not systemEnabled then
                reticle.Visible = false
                return
            end
            
            local hum = char:FindFirstChildOfClass("Humanoid")
            local hrp = char:FindFirstChild("HumanoidRootPart")
            if not hum or hum.Health <= 0 or not hrp then
                reticle.Visible = false
                return
            end
            
            local target, dist = findNearestKnifeThreat(hrp)
            
            if target then
                local targetHrp = target:FindFirstChild("HumanoidRootPart")
                local targetHead = target:FindFirstChild("Head") or targetHrp
                local targetHum = target:FindFirstChildOfClass("Humanoid")
                
                if targetHrp and targetHead and targetHum and targetHum.Health > 0 then
                    local aimPos = targetHead.Position
                    
                    -- 1. Auto-Equip Gun

                    ensureGunEquipped(char, hum)
                    
                    -- 2. Lock Camera onto Target

                    local camPos = camera.CFrame.Position
                    camera.CFrame = CFrame.new(camPos, aimPos)
                    
                    -- 3. Turn Character towards Target

                    local flatDiff = Vector3.new(aimPos.X - hrp.Position.X, 0, aimPos.Z - hrp.Position.Z)
                    if flatDiff.Magnitude > 0.1 then
                        hrp.CFrame = CFrame.new(hrp.Position, hrp.Position + flatDiff)
                    end
                    
                    -- 4. Auto-Shot when holding Gun

                    local holdingGun = char:FindFirstChildOfClass("Tool")
                    if holdingGun and string.find(holdingGun.Name:lower(), "gun") then
                        if (tick() - lastShotTime >= FIRE_COOLDOWN) then
                            local fireEvent = holdingGun:FindFirstChild("FireWeapon")
                            if fireEvent then
                                lastShotTime = tick()
                                fireEvent:FireServer(aimPos)
                            end
                        end
                    end
                    
                    -- 5. Update HUD Reticle

                    local screenPos, onScreen = camera:WorldToViewportPoint(aimPos)
                    if onScreen then
                        reticle.Position = UDim2.new(0, screenPos.X, 0, screenPos.Y)
                        reticle.Visible = true
                        targetLabel.Text = string.format("[LOCKED: %s | %.0f STUDS | HP: %.0f]", target.Name, dist, targetHum.Health)
                    else
                        reticle.Visible = false
                    end
                    
                    -- Update Banner to Alert

                    bannerStroke.Color = Color3.fromRGB(255, 60, 60)
                    statusBanner.BackgroundColor3 = Color3.fromRGB(35, 15, 15)
                    statusIcon.Text = "🎯"
                    statusTitle.Text = "THREAT ENGAGED: " .. string.upper(target.Name)
                    statusTitle.TextColor3 = Color3.fromRGB(255, 70, 70)
                    statusDesc.Text = string.format("Knife threat detected %.1f studs away! Auto-firing...", dist)
                    statusDesc.TextColor3 = Color3.fromRGB(255, 190, 190)
                    return
                end
            end
            
            -- Standby state

            reticle.Visible = false
            if systemEnabled then
                bannerStroke.Color = Color3.fromRGB(50, 150, 255)
                statusBanner.BackgroundColor3 = Color3.fromRGB(18, 20, 28)
                statusIcon.Text = "🛡️"
                statusTitle.Text = "AUTO-DEFENSE: ACTIVE"
                statusTitle.TextColor3 = Color3.fromRGB(80, 200, 255)
                statusDesc.Text = "Scanning perimeter (35 studs) for knife threats..."
                statusDesc.TextColor3 = Color3.fromRGB(180, 190, 205)
            end
        end)
        
        -- Attribute Sync for external HUDs

        localPlayer:SetAttribute("AutoDefenseEnabled", systemEnabled)
        localPlayer:GetAttributeChangedSignal("AutoDefenseEnabled"):Connect(function()
            local val = localPlayer:GetAttribute("AutoDefenseEnabled")
            if val ~= nil and val ~= systemEnabled then
                toggleSystem()
            end
        end)
        -- RoleHighlightController (With Integrated Auto-Defense Control)

        local Players = game:GetService("Players")
        local UserInputService = game:GetService("UserInputService")
        local RunService = game:GetService("RunService")
        local TweenService = game:GetService("TweenService")
        local SoundService = game:GetService("SoundService")
        
        local localPlayer = Players.LocalPlayer
        local camera = workspace.CurrentCamera
        local screenGui = script.Parent
        
        -- Color Configurations (Strictly following user specifications)

        local COLOR_NO_WEAPON = Color3.fromRGB(46, 213, 115)  -- GREEN

        local COLOR_GUN       = Color3.fromRGB(0, 168, 255)   -- BLUE

        local COLOR_KNIFE     = Color3.fromRGB(255, 45, 85)   -- RED

        
        local config = {
        alwaysOnTop = true,
        soundFeedback = true,
        }
        
        local activeHighlights = {} -- [Model] = { Highlight = instance, Conns = {...}, Role = string, Weapon = string, Color = Color3 }

        local selectedTarget = nil
        
        -- Helper to find ancestor character Model with Humanoid

        local function getCharacterFromHit(hitInstance)
            if not hitInstance then return nil end
                local curr = hitInstance
                while curr and curr ~= workspace do
                    if curr:IsA("Model") and curr:FindFirstChildOfClass("Humanoid") then
                        return curr
                    end
                    curr = curr.Parent
                end
                return nil
            end
            
            -- Weapon Classification Logic for both Players and NPCs

            local function classifyEntityWeapons(model)
                local tools = {}
                
                -- Check Model children and descendants for tools (equipped or stored in NPC inventory)

                for _, item in ipairs(model:GetChildren()) do
                    if item:IsA("Tool") then
                        table.insert(tools, item)
                    end
                end
                for _, item in ipairs(model:GetDescendants()) do
                    if item:IsA("Tool") and not table.find(tools, item) then
                        table.insert(tools, item)
                    end
                end
                
                -- If it's a Player, also check their Backpack

                local player = Players:GetPlayerFromCharacter(model)
                if player and player:FindFirstChild("Backpack") then
                    for _, item in ipairs(player.Backpack:GetChildren()) do
                        if item:IsA("Tool") and not table.find(tools, item) then
                            table.insert(tools, item)
                        end
                    end
                end
                
                local hasKnife = false
                local knifeName = nil
                local hasGun = false
                local gunName = nil
                
                for _, tool in ipairs(tools) do
                    local nameLower = string.lower(tool.Name)
                    
                    -- Knife identification

                    if nameLower:find("knife") or nameLower:find("blade") or nameLower:find("sword") or 
                        nameLower:find("dagger") or nameLower:find("machete") or nameLower:find("murder") or 
                        nameLower:find("katana") or tool:GetAttribute("WeaponType") == "Knife" then
                        hasKnife = true
                        knifeName = tool.Name
                        -- Gun identification

                    elseif nameLower:find("gun") or nameLower:find("pistol") or nameLower:find("revolver") or 
                        nameLower:find("rifle") or nameLower:find("shotgun") or nameLower:find("sniper") or 
                        nameLower:find("sheriff") or nameLower:find("firearm") or nameLower:find("blaster") or 
                        tool:GetAttribute("WeaponType") == "Gun" then
                        hasGun = true
                        gunName = tool.Name
                    end
                end
                
                -- Priority: Knife (Red) > Gun (Blue) > Unarmed (Green)

                if hasKnife then
                    return "Knife", knifeName or "Knife", COLOR_KNIFE
                elseif hasGun then
                    return "Gun", gunName or "Gun", COLOR_GUN
                else
                    return "Unarmed", "None", COLOR_NO_WEAPON
                end
            end
            
            -- Play UI audio feedback

            local function playFeedback(soundId, pitch)
                if not config.soundFeedback then return end
                    local s = Instance.new("Sound")
                    s.SoundId = soundId
                    s.Volume = 0.5
                    s.PlaybackSpeed = pitch or 1
                    s.Parent = SoundService
                    s:Play()
                    game:GetService("Debris"):AddItem(s, 1.5)
                end
                
                -- ==================== UI CONSTRUCTION ====================

                
                -- 1. Main HUD Frame (Adjusted height for Auto-Defense button)

                local hudFrame = Instance.new("Frame")
                hudFrame.Name = "HUDFrame"
                hudFrame.Size = UDim2.new(0, 310, 0, 262)
                hudFrame.Position = UDim2.new(1, -330, 0, 45)
                hudFrame.BackgroundColor3 = Color3.fromRGB(18, 22, 30)
                hudFrame.BackgroundTransparency = 0.12
                hudFrame.BorderSizePixel = 0
                hudFrame.Parent = screenGui
                
                local hudCorner = Instance.new("UICorner")
                hudCorner.CornerRadius = UDim.new(0, 10)
                hudCorner
