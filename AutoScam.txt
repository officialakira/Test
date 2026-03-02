local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local PathfindingService = game:GetService("PathfindingService")
local StarterGui = game:GetService("StarterGui")
local lp = Players.LocalPlayer

local function block(plr)
    if not plr or plr == lp then return end
    pcall(function()
        StarterGui:SetCore("PromptBlockPlayer", plr)
    end)
end

_G.InstaPickup = true

local SCREEN_GUI = Instance.new("ScreenGui")
SCREEN_GUI.Name = "BrainrotTeleporter"
SCREEN_GUI.DisplayOrder = 999
SCREEN_GUI.ResetOnSpawn = false

local MAIN_FRAME = Instance.new("Frame")
MAIN_FRAME.Name = "MainFrame"
MAIN_FRAME.Size = UDim2.new(0, 500, 0, 600)
MAIN_FRAME.Position = UDim2.new(0.5, -250, 0.5, -300)
MAIN_FRAME.BackgroundColor3 = Color3.fromRGB(45, 20, 60)
MAIN_FRAME.BackgroundTransparency = 0.15
MAIN_FRAME.BorderSizePixel = 0
MAIN_FRAME.Active = true
MAIN_FRAME.Draggable = true
MAIN_FRAME.Parent = SCREEN_GUI

local CORNER = Instance.new("UICorner")
CORNER.CornerRadius = UDim.new(0, 12)
CORNER.Parent = MAIN_FRAME

local TITLE_BAR = Instance.new("Frame")
TITLE_BAR.Name = "TitleBar"
TITLE_BAR.Size = UDim2.new(1, 0, 0, 40)
TITLE_BAR.BackgroundColor3 = Color3.fromRGB(60, 25, 80)
TITLE_BAR.BackgroundTransparency = 0.2
TITLE_BAR.BorderSizePixel = 0
TITLE_BAR.Parent = MAIN_FRAME

local TITLE = Instance.new("TextLabel")
TITLE.Name = "Title"
TITLE.Size = UDim2.new(1, -10, 1, 0)
TITLE.Position = UDim2.new(0, 10, 0, 0)
TITLE.BackgroundTransparency = 1
TITLE.Text = "idk how to name"
TITLE.TextColor3 = Color3.fromRGB(255, 255, 255)
TITLE.TextScaled = true
TITLE.Font = Enum.Font.GothamBold
TITLE.TextXAlignment = Enum.TextXAlignment.Left
TITLE.Parent = TITLE_BAR

local SUBTITLE = Instance.new("TextLabel")
SUBTITLE.Name = "Subtitle"
SUBTITLE.Size = UDim2.new(1, -10, 0, 20)
SUBTITLE.Position = UDim2.new(0, 10, 0, 40)
SUBTITLE.BackgroundTransparency = 1
SUBTITLE.Text = "made by nanygata"
SUBTITLE.TextColor3 = Color3.fromRGB(200, 180, 255)
SUBTITLE.TextScaled = true
SUBTITLE.Font = Enum.Font.Gotham
SUBTITLE.TextXAlignment = Enum.TextXAlignment.Left
SUBTITLE.Parent = MAIN_FRAME

local SCROLL_FRAME = Instance.new("ScrollingFrame")
SCROLL_FRAME.Name = "ScrollFrame"
SCROLL_FRAME.Size = UDim2.new(1, -20, 1, -120)
SCROLL_FRAME.Position = UDim2.new(0, 10, 0, 70)
SCROLL_FRAME.BackgroundTransparency = 1
SCROLL_FRAME.BorderSizePixel = 0
SCROLL_FRAME.ScrollBarThickness = 8
SCROLL_FRAME.ScrollBarImageColor3 = Color3.fromRGB(120, 60, 160)
SCROLL_FRAME.CanvasSize = UDim2.new(0, 0, 0, 0)
SCROLL_FRAME.AutomaticCanvasSize = Enum.AutomaticSize.Y
SCROLL_FRAME.Parent = MAIN_FRAME

local UI_LIST_LAYOUT = Instance.new("UIListLayout")
UI_LIST_LAYOUT.Padding = UDim.new(0, 10)
UI_LIST_LAYOUT.Parent = SCROLL_FRAME

local CLOSE_BUTTON = Instance.new("TextButton")
CLOSE_BUTTON.Name = "CloseButton"
CLOSE_BUTTON.Size = UDim2.new(0, 30, 0, 30)
CLOSE_BUTTON.Position = UDim2.new(1, -35, 0, 5)
CLOSE_BUTTON.BackgroundColor3 = Color3.fromRGB(80, 30, 100)
CLOSE_BUTTON.BackgroundTransparency = 0.3
CLOSE_BUTTON.Text = "X"
CLOSE_BUTTON.TextColor3 = Color3.fromRGB(255, 255, 255)
CLOSE_BUTTON.TextScaled = true
CLOSE_BUTTON.Font = Enum.Font.GothamBold
CLOSE_BUTTON.Parent = TITLE_BAR

local CLOSE_CORNER = Instance.new("UICorner")
CLOSE_CORNER.CornerRadius = UDim.new(0, 8)
CLOSE_CORNER.Parent = CLOSE_BUTTON

local isTeleporting = false

local function extractValue(text)
    if not text then return 0 end
    local clean = text:gsub("[%$,/s]", ""):gsub(",", "")
    local mult = 1
    if clean:match("[kK]") then
        mult = 1e3
        clean = clean:gsub("[kK]", "")
    elseif clean:match("[mM]") then
        mult = 1e6
        clean = clean:gsub("[mM]", "")
    elseif clean:match("[bB]") then
        mult = 1e9
        clean = clean:gsub("[bB]", "")
    elseif clean:match("[tT]") then
        mult = 1e12
        clean = clean:gsub("[tT]", "")
    elseif clean:match("[qQ]") then
        mult = 1e15
        clean = clean:gsub("[qQ]", "")
    end
    local num = tonumber(clean)
    return num and num * mult or 0
end

local function formatValue(value)
    if value >= 1e15 then
        return string.format("%.1fQ", value / 1e15)
    elseif value >= 1e12 then
        return string.format("%.1fT", value / 1e12)
    elseif value >= 1e9 then
        return string.format("%.1fB", value / 1e9)
    elseif value >= 1e6 then
        return string.format("%.1fM", value / 1e6)
    elseif value >= 1e3 then
        return string.format("%.1fK", value / 1e3)
    else
        return tostring(math.floor(value))
    end
end

local function findAllBrainrots()
    local allPets = {}
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj.Name == "AnimalOverhead" then
            local petValue, petName, generation = 0, "Brainrot", "Gen 1"
            
            for _, child in ipairs(obj:GetChildren()) do
                if child:IsA("TextLabel") then
                    local text = child.Text
                    if text then
                        if child.Name == "DisplayName" and text ~= "" then
                            petName = text
                        elseif text:find("$") and text:find("/s") then
                            petValue = extractValue(text)
                        elseif child.Name == "Generation" then
                            generation = text
                        end
                    end
                end
            end
            
            local position = nil
            local current = obj.Parent
            while current and current ~= workspace do
                if current:IsA("BasePart") then
                    position = current.Position
                    break
                end
                current = current.Parent
            end
            
            if position and petValue > 0 then
                table.insert(allPets, {
                    position = position,
                    name = petName,
                    value = petValue,
                    formattedValue = formatValue(petValue),
                    generation = generation
                })
            end
        end
    end
    
    table.sort(allPets, function(a, b) 
        return a.value > b.value
    end)
    
    return allPets
end

local InstaPickupSystem = {}
InstaPickupSystem.currentPrompt = nil
InstaPickupSystem.currentDistance = math.huge
InstaPickupSystem.lastUpdate = 0

function InstaPickupSystem:parseMoneyPerSec(text)
    if not text then return 0 end
    local mult = 1
    local numberStr = text:match("[%d%.]+")
    if not numberStr then return 0 end
    if text:find("K") then mult = 1000
    elseif text:find("M") then mult = 1000000
    elseif text:find("B") then mult = 1000000000
    elseif text:find("T") then mult = 1000000000000
    elseif text:find("Q") then mult = 1000000000000000
    end
    return tonumber(numberStr) and tonumber(numberStr) * mult or 0
end

function InstaPickupSystem:getBrainrotValue(prompt)
    local parent = prompt.Parent
    if not parent or not parent.Parent then return 0 end
    local model = parent.Parent
    for _, desc in ipairs(model:GetDescendants()) do
        if desc:IsA("TextLabel") and desc.Name == "Rarity" then
            local gen = desc.Parent:FindFirstChild("Generation")
            if gen and gen:IsA("TextLabel") then
                return self:parseMoneyPerSec(gen.Text)
            end
        end
    end
    return 0
end

function InstaPickupSystem:getPromptPosition(prompt)
    local parent = prompt.Parent
    if parent:IsA("BasePart") then
        return parent.Position
    end
    if parent:IsA("Model") then
        local primary = parent.PrimaryPart or parent:FindFirstChildWhichIsA("BasePart")
        if primary then
            return primary.Position
        end
    end
    if parent:IsA("Attachment") then
        return parent.WorldPosition
    end
    return nil
end

function InstaPickupSystem:findHighestValuePrompt()
    local bestPrompt = nil
    local bestValue = 0
    local bestDist = math.huge
    local plots = workspace:FindFirstChild("Plots")
    if not plots then return nil, math.huge end
    
    for _, obj in pairs(plots:GetDescendants()) do
        if obj:IsA("ProximityPrompt") and obj.Enabled and obj.ActionText == "Steal" then
            local pos = self:getPromptPosition(obj)
            if pos then
                local dist = (self.hrp.Position - pos).Magnitude
                if dist <= obj.MaxActivationDistance then
                    local value = self:getBrainrotValue(obj)
                    if value > bestValue then
                        bestValue = value
                        bestPrompt = obj
                        bestDist = dist
                    end
                end
            end
        end
    end
    return bestPrompt, bestDist
end

function InstaPickupSystem:activatePrompt(prompt)
    prompt.RequiresLineOfSight = false
    fireproximityprompt(prompt, 20, math.huge)
    prompt:InputHoldBegin()
    prompt:InputHoldEnd()
end

function InstaPickupSystem:start()
    local function getCharacter()
        return lp.Character or lp.CharacterAdded:Wait()
    end
    
    local function getHRP()
        local char = getCharacter()
        return char:WaitForChild("HumanoidRootPart")
    end
    
    self.hrp = getHRP()
    local humanoid = getCharacter():WaitForChild("Humanoid")
    
    lp.CharacterAdded:Connect(function()
        self.hrp = getHRP()
        humanoid = getCharacter():WaitForChild("Humanoid")
    end)
    
    RunService.Heartbeat:Connect(function()
        local now = tick()
        if now - self.lastUpdate >= 0.05 then
            self.currentPrompt, self.currentDistance = self:findHighestValuePrompt()
            self.lastUpdate = now
        end
    end)
    
    task.spawn(function()
        while true do
            if _G.InstaPickup and humanoid.WalkSpeed > 25 then
                if self.currentPrompt and self.currentDistance <= self.currentPrompt.MaxActivationDistance then
                    self:activatePrompt(self.currentPrompt)
                    task.wait(1.5)
                else
                    task.wait(0.1)
                end
            else
                task.wait(0.5)
            end
        end
    end)
end

local function equipFlyingTool()
    local char = lp.Character
    if not char then return nil end
    
    local backpack = lp:FindFirstChild("Backpack")
    if not backpack then return nil end
    
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if not humanoid then return nil end
    
    local flyingCarpet = nil
    
    for _, item in ipairs(backpack:GetDescendants()) do
        if item:IsA("Tool") and item.Name == "Flying Carpet" then
            flyingCarpet = item
            break
        end
    end
    
    if flyingCarpet then
        humanoid:EquipTool(flyingCarpet)
        return "Flying Carpet"
    end
    
    local witchBroom = nil
    
    for _, item in ipairs(backpack:GetDescendants()) do
        if item:IsA("Tool") and item.Name == "Witch's Broom" then
            witchBroom = item
            break
        end
    end
    
    if witchBroom then
        humanoid:EquipTool(witchBroom)
        return "Witch's Broom"
    end
    
    return nil
end

local function teleportToWaypoints(waypoints)
    if isTeleporting then return end
    isTeleporting = true
    
    local char = lp.Character
    if not char then 
        isTeleporting = false
        return 
    end
    
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then 
        isTeleporting = false
        return 
    end
    
    local toolEquipped = equipFlyingTool()
    if toolEquipped then
        task.wait(0.1)
    end
    
    for _, wp in ipairs(waypoints) do
        if not isTeleporting then break end
        
        local elevatedPos = Vector3.new(wp.Position.X, wp.Position.Y + 5, wp.Position.Z)
        hrp.CFrame = CFrame.new(elevatedPos)
        task.wait(0.02)
    end
    
    local players = Players:GetPlayers()
    local nearestPlayer = nil
    local nearestDist = math.huge
    
    for _, player in ipairs(players) do
        if player ~= lp and player.Character then
            local targetHRP = player.Character:FindFirstChild("HumanoidRootPart")
            if targetHRP then
                local dist = (hrp.Position - targetHRP.Position).Magnitude
                if dist < nearestDist then
                    nearestDist = dist
                    nearestPlayer = player
                end
            end
        end
    end
    
    if nearestPlayer then
        block(nearestPlayer)
    end
    
    isTeleporting = false
end

local function teleportToPosition(targetPosition)
    local char = lp.Character
    if not char then return end
    
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    local path = PathfindingService:CreatePath({
        AgentRadius = 2,
        AgentHeight = 5,
        AgentCanJump = true
    })
    
    local elevatedTarget = Vector3.new(targetPosition.X, targetPosition.Y + 5, targetPosition.Z)
    path:ComputeAsync(hrp.Position, elevatedTarget)
    
    if path.Status == Enum.PathStatus.Success then
        local waypointsPath = path:GetWaypoints()
        teleportToWaypoints(waypointsPath)
    else
        hrp.CFrame = CFrame.new(elevatedTarget)
        
        local players = Players:GetPlayers()
        local nearestPlayer = nil
        local nearestDist = math.huge
        
        for _, player in ipairs(players) do
            if player ~= lp and player.Character then
                local targetHRP = player.Character:FindFirstChild("HumanoidRootPart")
                if targetHRP then
                    local dist = (hrp.Position - targetHRP.Position).Magnitude
                    if dist < nearestDist then
                        nearestDist = dist
                        nearestPlayer = player
                    end
                end
            end
        end
        
        if nearestPlayer then
            block(nearestPlayer)
        end
    end
end

local function getValueColor(value)
    if value >= 1e12 then
        return Color3.fromRGB(255, 215, 0)
    elseif value >= 1e9 then
        return Color3.fromRGB(192, 192, 192)
    elseif value >= 1e6 then
        return Color3.fromRGB(205, 127, 50)
    else
        return Color3.fromRGB(120, 255, 120)
    end
end

local function createPetButton(petInfo, index)
    local button = Instance.new("TextButton")
    button.Name = "PetButton_" .. index
    button.Size = UDim2.new(1, 0, 0, 80)
    button.BackgroundColor3 = Color3.fromRGB(70, 30, 90)
    button.BackgroundTransparency = 0.2
    button.Text = ""
    button.AutoButtonColor = true
    button.Parent = SCROLL_FRAME
    
    local buttonCorner = Instance.new("UICorner")
    buttonCorner.CornerRadius = UDim.new(0, 8)
    buttonCorner.Parent = button
    
    local rankFrame = Instance.new("Frame")
    rankFrame.Name = "RankFrame"
    rankFrame.Size = UDim2.new(0, 40, 1, 0)
    rankFrame.BackgroundColor3 = Color3.fromRGB(50, 20, 70)
    rankFrame.BackgroundTransparency = 0.3
    rankFrame.BorderSizePixel = 0
    rankFrame.Parent = button
    
    local rankCorner = Instance.new("UICorner")
    rankCorner.CornerRadius = UDim.new(0, 8)
    rankCorner.Parent = rankFrame
    
    local rankLabel = Instance.new("TextLabel")
    rankLabel.Name = "Rank"
    rankLabel.Size = UDim2.new(1, 0, 1, 0)
    rankLabel.BackgroundTransparency = 1
    rankLabel.Text = "#" .. index
    rankLabel.TextColor3 = Color3.fromRGB(255, 200, 100)
    rankLabel.TextScaled = true
    rankLabel.Font = Enum.Font.GothamBold
    rankLabel.Parent = rankFrame
    
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Name = "Name"
    nameLabel.Size = UDim2.new(0.5, -50, 0.4, 0)
    nameLabel.Position = UDim2.new(0, 50, 0, 5)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text = petInfo.name
    nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    nameLabel.TextScaled = true
    nameLabel.TextXAlignment = Enum.TextXAlignment.Left
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.Parent = button
    
    local genLabel = Instance.new("TextLabel")
    genLabel.Name = "Generation"
    genLabel.Size = UDim2.new(0.5, -10, 0.4, 0)
    genLabel.Position = UDim2.new(0.5, 0, 0, 5)
    genLabel.BackgroundTransparency = 1
    genLabel.Text = petInfo.generation
    genLabel.TextColor3 = Color3.fromRGB(180, 220, 255)
    genLabel.TextScaled = true
    genLabel.TextXAlignment = Enum.TextXAlignment.Right
    genLabel.Font = Enum.Font.Gotham
    genLabel.Parent = button
    
    local valueColor = getValueColor(petInfo.value)
    
    local valueLabel = Instance.new("TextLabel")
    valueLabel.Name = "Value"
    valueLabel.Size = UDim2.new(1, -60, 0.4, 0)
    valueLabel.Position = UDim2.new(0, 50, 0.4, 5)
    valueLabel.BackgroundTransparency = 1
    valueLabel.Text = "$" .. petInfo.formattedValue .. "/s"
    valueLabel.TextColor3 = valueColor
    valueLabel.TextScaled = true
    valueLabel.TextXAlignment = Enum.TextXAlignment.Left
    valueLabel.Font = Enum.Font.GothamBold
    valueLabel.Parent = button
    
    local teleportLabel = Instance.new("TextLabel")
    teleportLabel.Name = "TeleportLabel"
    teleportLabel.Size = UDim2.new(1, -60, 0.2, 0)
    teleportLabel.Position = UDim2.new(0, 50, 0.8, 0)
    teleportLabel.BackgroundTransparency = 1
    teleportLabel.Text = "Clique para teleportar"
    teleportLabel.TextColor3 = Color3.fromRGB(200, 180, 255)
    teleportLabel.TextScaled = true
    teleportLabel.Font = Enum.Font.Gotham
    teleportLabel.TextXAlignment = Enum.TextXAlignment.Left
    teleportLabel.Parent = button
    
    if index == 1 then
        button.BackgroundColor3 = Color3.fromRGB(90, 40, 130)
        local crown = Instance.new("TextLabel")
        crown.Name = "Crown"
        crown.Size = UDim2.new(0, 20, 0, 20)
        crown.Position = UDim2.new(1, -25, 0, 5)
        crown.BackgroundTransparency = 1
        crown.Text = "👑"
        crown.TextColor3 = Color3.fromRGB(255, 215, 0)
        crown.TextScaled = true
        crown.Parent = button
    elseif index == 2 then
        button.BackgroundColor3 = Color3.fromRGB(80, 35, 115)
    elseif index == 3 then
        button.BackgroundColor3 = Color3.fromRGB(70, 30, 100)
    end
    
    button.MouseEnter:Connect(function()
        if index > 3 then
            button.BackgroundColor3 = Color3.fromRGB(90, 40, 120)
        end
    end)
    
    button.MouseLeave:Connect(function()
        if index == 1 then
            button.BackgroundColor3 = Color3.fromRGB(90, 40, 130)
        elseif index == 2 then
            button.BackgroundColor3 = Color3.fromRGB(80, 35, 115)
        elseif index == 3 then
            button.BackgroundColor3 = Color3.fromRGB(70, 30, 100)
        else
            button.BackgroundColor3 = Color3.fromRGB(70, 30, 90)
        end
    end)
    
    button.MouseButton1Click:Connect(function()
        teleportToPosition(petInfo.position)
    end)
    
    return button
end

local function updatePetList()
    local allPets = findAllBrainrots()
    
    for _, child in ipairs(SCROLL_FRAME:GetChildren()) do
        if child:IsA("TextButton") then
            child:Destroy()
        end
    end
    
    local petCount = Instance.new("TextLabel")
    petCount.Name = "PetCount"
    petCount.Size = UDim2.new(1, 0, 0, 30)
    petCount.BackgroundTransparency = 1
    petCount.Text = "🎯 Pets encontrados: " .. #allPets
    petCount.TextColor3 = Color3.fromRGB(200, 180, 255)
    petCount.TextScaled = true
    petCount.Font = Enum.Font.GothamBold
    petCount.Parent = SCROLL_FRAME
    
    if #allPets == 0 then
        local noPetsLabel = Instance.new("TextLabel")
        noPetsLabel.Name = "NoPets"
        noPetsLabel.Size = UDim2.new(1, 0, 0, 100)
        noPetsLabel.BackgroundTransparency = 1
        noPetsLabel.Text = "Nenhum brainrot encontrado"
        noPetsLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
        noPetsLabel.TextScaled = true
        noPetsLabel.Font = Enum.Font.GothamBold
        noPetsLabel.Parent = SCROLL_FRAME
    else
        for i, pet in ipairs(allPets) do
            createPetButton(pet, i)
        end
        
        local bestValue = allPets[1].formattedValue
        local bestLabel = Instance.new("TextLabel")
        bestLabel.Name = "BestValue"
        bestLabel.Size = UDim2.new(1, 0, 0, 25)
        bestLabel.Position = UDim2.new(0, 0, 0, 35)
        bestLabel.BackgroundTransparency = 1
        bestLabel.Text = "🎖️ Melhor valor: $" .. bestValue .. "/s"
        bestLabel.TextColor3 = Color3.fromRGB(255, 215, 0)
        bestLabel.TextScaled = true
        bestLabel.Font = Enum.Font.GothamBold
        bestLabel.Parent = SCROLL_FRAME
    end
end

local REFRESH_BUTTON = Instance.new("TextButton")
REFRESH_BUTTON.Name = "RefreshButton"
REFRESH_BUTTON.Size = UDim2.new(0, 140, 0, 40)
REFRESH_BUTTON.Position = UDim2.new(0.5, -150, 1, -50)
REFRESH_BUTTON.BackgroundColor3 = Color3.fromRGB(80, 40, 120)
REFRESH_BUTTON.BackgroundTransparency = 0.2
REFRESH_BUTTON.Text = "🔄 Atualizar Lista"
REFRESH_BUTTON.TextColor3 = Color3.fromRGB(255, 255, 255)
REFRESH_BUTTON.TextScaled = true
REFRESH_BUTTON.Font = Enum.Font.GothamBold
REFRESH_BUTTON.Parent = MAIN_FRAME

local REFRESH_CORNER = Instance.new("UICorner")
REFRESH_CORNER.CornerRadius = UDim.new(0, 8)
REFRESH_CORNER.Parent = REFRESH_BUTTON

REFRESH_BUTTON.MouseButton1Click:Connect(function()
    updatePetList()
end)

local TELEPORT_BEST_BUTTON = Instance.new("TextButton")
TELEPORT_BEST_BUTTON.Name = "TeleportBestButton"
TELEPORT_BEST_BUTTON.Size = UDim2.new(0, 140, 0, 40)
TELEPORT_BEST_BUTTON.Position = UDim2.new(0.5, 10, 1, -50)
TELEPORT_BEST_BUTTON.BackgroundColor3 = Color3.fromRGB(40, 160, 200)
TELEPORT_BEST_BUTTON.BackgroundTransparency = 0.2
TELEPORT_BEST_BUTTON.Text = "🚀 Teleportar #1"
TELEPORT_BEST_BUTTON.TextColor3 = Color3.fromRGB(255, 255, 255)
TELEPORT_BEST_BUTTON.TextScaled = true
TELEPORT_BEST_BUTTON.Font = Enum.Font.GothamBold
TELEPORT_BEST_BUTTON.Parent = MAIN_FRAME

local TELEPORT_CORNER = Instance.new("UICorner")
TELEPORT_CORNER.CornerRadius = UDim.new(0, 8)
TELEPORT_CORNER.Parent = TELEPORT_BEST_BUTTON

TELEPORT_BEST_BUTTON.MouseButton1Click:Connect(function()
    local allPets = findAllBrainrots()
    if #allPets > 0 then
        teleportToPosition(allPets[1].position)
    end
end)

local AUTOGRAB_BUTTON = Instance.new("TextButton")
AUTOGRAB_BUTTON.Name = "AutoGrabButton"
AUTOGRAB_BUTTON.Size = UDim2.new(0, 140, 0, 40)
AUTOGRAB_BUTTON.Position = UDim2.new(0.5, -150, 1, -100)
AUTOGRAB_BUTTON.BackgroundColor3 = Color3.fromRGB(120, 40, 160)
AUTOGRAB_BUTTON.BackgroundTransparency = 0.2
AUTOGRAB_BUTTON.Text = "🤖 Auto Grab: ON"
AUTOGRAB_BUTTON.TextColor3 = Color3.fromRGB(255, 255, 255)
AUTOGRAB_BUTTON.TextScaled = true
AUTOGRAB_BUTTON.Font = Enum.Font.GothamBold
AUTOGRAB_BUTTON.Parent = MAIN_FRAME

local AUTOGRAB_CORNER = Instance.new("UICorner")
AUTOGRAB_CORNER.CornerRadius = UDim.new(0, 8)
AUTOGRAB_CORNER.Parent = AUTOGRAB_BUTTON

AUTOGRAB_BUTTON.MouseButton1Click:Connect(function()
    _G.InstaPickup = not _G.InstaPickup
    AUTOGRAB_BUTTON.Text = "🤖 Auto Grab: " .. (_G.InstaPickup and "ON" or "OFF")
    AUTOGRAB_BUTTON.BackgroundColor3 = _G.InstaPickup and Color3.fromRGB(120, 40, 160) or Color3.fromRGB(80, 40, 100)
end)

CLOSE_BUTTON.MouseButton1Click:Connect(function()
    SCREEN_GUI:Destroy()
end)

InstaPickupSystem:start()

local function initialize()
    updatePetList()
    
    if lp:FindFirstChild("PlayerGui") then
        SCREEN_GUI.Parent = lp:FindFirstChild("PlayerGui")
    else
        SCREEN_GUI.Parent = game:GetService("CoreGui")
    end
    
    task.spawn(function()
        while SCREEN_GUI.Parent do
            updatePetList()
            task.wait(3)
        end
    end)
end

initialize()