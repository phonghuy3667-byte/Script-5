--========================================================
-- PHONGHUY36 SERVER
-- Roblox Studio - dành cho game của bạn
--========================================================

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local REMOTE_NAME = "PHONGHUY36_REMOTE"
local FARM_RANGE = 5000

local Remote = ReplicatedStorage:FindFirstChild(REMOTE_NAME)

if not Remote then
    Remote = Instance.new("RemoteEvent")
    Remote.Name = REMOTE_NAME
    Remote.Parent = ReplicatedStorage
end

--========================================================
-- ADMIN
-- THAY USER ID CỦA BẠN VÀO ĐÂY
--========================================================

local ADMINS = {
    [123456789] = true,
}

local function isAdmin(player)
    return ADMINS[player.UserId] == true
end

--========================================================
-- PLAYER FINDER
--========================================================

local function findPlayer(text)
    if not text or text == "" then
        return nil
    end

    text = string.lower(text)

    for _, player in ipairs(Players:GetPlayers()) do
        if string.lower(player.Name) == text then
            return player
        end
    end

    for _, player in ipairs(Players:GetPlayers()) do
        if string.lower(player.DisplayName) == text then
            return player
        end
    end

    for _, player in ipairs(Players:GetPlayers()) do
        if string.lower(player.Name):sub(1, #text) == text then
            return player
        end
    end

    return nil
end

--========================================================
-- GET ROOT
--========================================================

local function getRoot(model)
    if not model then
        return nil
    end

    return model:FindFirstChild("HumanoidRootPart")
        or model.PrimaryPart
end

--========================================================
-- KILL FOLDER
--========================================================

local function killFolder(player, folderName)
    local folder = workspace:FindFirstChild(folderName)

    if not folder then
        return
    end

    local character = player.Character
    local root = getRoot(character)

    if not root then
        return
    end

    for _, npc in ipairs(folder:GetChildren()) do
        local humanoid = npc:FindFirstChildOfClass("Humanoid")
        local npcRoot = getRoot(npc)

        if humanoid and npcRoot and humanoid.Health > 0 then
            local distance = (npcRoot.Position - root.Position).Magnitude

            if distance <= FARM_RANGE then
                humanoid.Health = 0
            end
        end
    end
end

--========================================================
-- CHEST
--========================================================

local function collectChests(player)
    local folder = workspace:FindFirstChild("Chests")

    if not folder then
        return
    end

    local character = player.Character
    local root = getRoot(character)

    if not root then
        return
    end

    for _, chest in ipairs(folder:GetChildren()) do
        local chestRoot = getRoot(chest)

        if chestRoot then
            local distance = (chestRoot.Position - root.Position).Magnitude

            if distance <= FARM_RANGE then
                chest:SetAttribute("Collected", true)

                local prompt = chest:FindFirstChildWhichIsA(
                    "ProximityPrompt",
                    true
                )

                if prompt then
                    prompt.Enabled = false
                end

                -- Hệ thống reward riêng của game có thể xử lý ở đây.
            end
        end
    end
end

--========================================================
-- FARM LOOP
--========================================================

local farming = {}

local function startFarm(player)
    if farming[player] then
        return
    end

    farming[player] = true

    task.spawn(function()
        while farming[player] and player.Parent do

            killFolder(player, "Monsters")
            killFolder(player, "Bosses")
            collectChests(player)

            task.wait(0.25)
        end
    end)
end

local function stopFarm(player)
    farming[player] = nil
end

--========================================================
-- ADMIN COMMAND
--========================================================

local function executeCommand(player, message)
    if not isAdmin(player) then
        return
    end

    local command, targetName = message:match("^%s*(%w+)%s*:%s*(.+)%s*$")

    if not command or not targetName then
        return
    end

    command = string.lower(command)
    targetName = targetName:gsub("^%s+", ""):gsub("%s+$", "")

    local target = findPlayer(targetName)

    if command == "kill" then
        if target and target.Character then
            local humanoid =
                target.Character:FindFirstChildOfClass("Humanoid")

            if humanoid then
                humanoid.Health = 0
            end
        end

    elseif command == "kick" then
        if target and target ~= player then
            target:Kick("Removed by PHONGHUY36 Admin.")
        end
    end
end

--========================================================
-- REMOTE
--========================================================

Remote.OnServerEvent:Connect(function(player, action, data)

    if action == "FarmStart" then
        startFarm(player)

    elseif action == "FarmStop" then
        stopFarm(player)

    elseif action == "KillMonsters" then
        killFolder(player, "Monsters")

    elseif action == "KillBosses" then
        killFolder(player, "Bosses")

    elseif action == "CollectChests" then
        collectChests(player)

    elseif action == "Command" then
        executeCommand(player, tostring(data or ""))

    elseif action == "AdminCheck" then
        Remote:FireClient(player, "AdminStatus", isAdmin(player))
    end
end)

Players.PlayerRemoving:Connect(function(player)
    farming[player] = nil
end)
--========================================================
-- PHONGHUY36 CLIENT
-- Roblox Studio - Mobile Hub
--========================================================

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")
local TextChatService = game:GetService("TextChatService")

local Player = Players.LocalPlayer
local PlayerGui = Player:WaitForChild("PlayerGui")

local Remote = ReplicatedStorage:WaitForChild(
    "PHONGHUY36_REMOTE",
    10
)

if not Remote then
    warn("PHONGHUY36: RemoteEvent not found.")
    return
end

--========================================================
-- CONFIG
--========================================================

local RANGE = 5000

local State = {
    Monster = false,
    Boss = false,
    Farm = false,
    Chest = false,

    MonsterESP = false,
    BossESP = false,
    ChestESP = false,

    FullMap = false,
    NoClip = false,
    Performance = false,

    Speed = false,
    Jump = false,

    Admin = false,
}

--========================================================
-- GUI
--========================================================

local Gui = Instance.new("ScreenGui")
Gui.Name = "PHONGHUY36"
Gui.ResetOnSpawn = false
Gui.IgnoreGuiInset = true
Gui.Parent = PlayerGui

--========================================================
-- FLOAT BUTTON
--========================================================

local Float = Instance.new("TextButton")
Float.Name = "OpenButton"
Float.Size = UDim2.fromOffset(54,54)
Float.Position = UDim2.new(0,15,0.5,-27)
Float.Text = "$"
Float.TextSize = 25
Float.Font = Enum.Font.GothamBold
Float.TextColor3 = Color3.new(1,1,1)
Float.BackgroundColor3 = Color3.fromRGB(35,35,35)
Float.Parent = Gui

local FloatCorner = Instance.new("UICorner")
FloatCorner.CornerRadius = UDim.new(1,0)
FloatCorner.Parent = Float

--========================================================
-- MAIN
--========================================================

local Main = Instance.new("Frame")
Main.Name = "Main"
Main.Size = UDim2.fromOffset(390,460)
Main.Position = UDim2.new(0.5,-195,0.5,-230)
Main.BackgroundColor3 = Color3.fromRGB(24,24,27)
Main.BorderSizePixel = 0
Main.Visible = true
Main.Parent = Gui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0,14)
MainCorner.Parent = Main

--========================================================
-- TITLE
--========================================================

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1,-20,0,45)
Title.Position = UDim2.fromOffset(10,5)
Title.BackgroundTransparency = 1
Title.Text = "PHONGHUY36"
Title.TextSize = 23
Title.Font = Enum.Font.GothamBold
Title.TextColor3 = Color3.new(1,1,1)
Title.Parent = Main

local SubTitle = Instance.new("TextLabel")
SubTitle.Size = UDim2.new(1,-20,0,20)
SubTitle.Position = UDim2.fromOffset(10,42)
SubTitle.BackgroundTransparency = 1
SubTitle.Text = "FULL DEVELOPMENT HUB"
SubTitle.TextSize = 11
SubTitle.Font = Enum.Font.Gotham
SubTitle.TextColor3 = Color3.fromRGB(170,170,170)
SubTitle.Parent = Main

--========================================================
-- SIDEBAR
--========================================================

local Sidebar = Instance.new("Frame")
Sidebar.Size = UDim2.fromOffset(105,365)
Sidebar.Position = UDim2.fromOffset(10,75)
Sidebar.BackgroundColor3 = Color3.fromRGB(31,31,35)
Sidebar.BorderSizePixel = 0
Sidebar.Parent = Main

local SidebarCorner = Instance.new("UICorner")
SidebarCorner.CornerRadius = UDim.new(0,10)
SidebarCorner.Parent = Sidebar

local SideLayout = Instance.new("UIListLayout")
SideLayout.Padding = UDim.new(0,5)
SideLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
SideLayout.VerticalAlignment = Enum.VerticalAlignment.Top
SideLayout.Parent = Sidebar

--========================================================
-- CONTENT
--========================================================

local Content = Instance.new("ScrollingFrame")
Content.Size = UDim2.fromOffset(255,365)
Content.Position = UDim2.fromOffset(125,75)
Content.BackgroundColor3 = Color3.fromRGB(31,31,35)
Content.BorderSizePixel = 0
Content.ScrollBarThickness = 3
Content.CanvasSize = UDim2.new(0,0,0,900)
Content.Parent = Main

local ContentCorner = Instance.new("UICorner")
ContentCorner.CornerRadius = UDim.new(0,10)
ContentCorner.Parent = Content

local Layout = Instance.new("UIListLayout")
Layout.Padding = UDim.new(0,7)
Layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
Layout.Parent = Content

--========================================================
-- HELPERS
--========================================================

local Tabs = {}

local function clearContent()
    for _, child in ipairs(Content:GetChildren()) do
        if not child:IsA("UIListLayout") then
            child:Destroy()
        end
    end
end

local function button(text, callback)
    local b = Instance.new("TextButton")

    b.Size = UDim2.new(1,-16,0,42)
    b.BackgroundColor3 = Color3.fromRGB(43,43,48)
    b.BorderSizePixel = 0
    b.Text = text
    b.TextSize = 13
    b.Font = Enum.Font.GothamSemibold
    b.TextColor3 = Color3.new(1,1,1)
    b.AutoButtonColor = true
    b.Parent = Content

    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0,8)
    c.Parent = b

    b.MouseButton1Click:Connect(callback)

    return b
end

local function toggle(text, key, callback)
    local b

    local function refresh()
        b.Text = text .. " : " .. (State[key] and "ON" or "OFF")
    end

    b = button(text, function()
        State[key] = not State[key]

        refresh()

        if callback then
            callback(State[key])
        end
    end)

    refresh()
    return b
end

local function section(text)
    local label = Instance.new("TextLabel")

    label.Size = UDim2.new(1,-16,0,30)
    label.BackgroundTransparency = 1
    label.Text = "  " .. text
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.TextSize = 13
    label.Font = Enum.Font.GothamBold
    label.TextColor3 = Color3.fromRGB(190,190,190)
    label.Parent = Content
end

local function tab(name)
    local b = Instance.new("TextButton")

    b.Size = UDim2.new(1,-10,0,38)
    b.BackgroundColor3 = Color3.fromRGB(43,43,48)
    b.BorderSizePixel = 0
    b.Text = name
    b.TextSize = 12
    b.Font = Enum.Font.GothamSemibold
    b.TextColor3 = Color3.new(1,1,1)
    b.Parent = Sidebar

    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0,8)
    c.Parent = b

    return b
end

--========================================================
-- COMBAT
--========================================================

local function loadCombat()

    clearContent()

    section("COMBAT")

    toggle("AUTO KILL MONSTER","Monster",function(on)
        Remote:FireServer(on and "FarmStart" or "FarmStop")
    end)

    toggle("AUTO KILL BOSS","Boss",function(on)
        Remote:FireServer(on and "FarmStart" or "FarmStop")
    end)

    button("KILL ALL MONSTERS",function()
        Remote:FireServer("KillMonsters")
    end)

    button("KILL ALL BOSSES",function()
        Remote:FireServer("KillBosses")
    end)

    section("RANGE")

    local rangeLabel = Instance.new("TextLabel")
    rangeLabel.Size = UDim2.new(1,-16,0,35)
    rangeLabel.BackgroundTransparency = 1
    rangeLabel.Text = "Kill Range : " .. RANGE .. " studs"
    rangeLabel.TextSize = 13
    rangeLabel.Font = Enum.Font.Gotham
    rangeLabel.TextColor3 = Color3.new(1,1,1)
    rangeLabel.Parent = Content
end

--========================================================
-- FARM
--========================================================

local function loadFarm()

    clearContent()

    section("FARM SYSTEM")

    toggle("AUTO FARM ALL","Farm",function(on)
        Remote:FireServer(on and "FarmStart" or "FarmStop")
    end)

    toggle("AUTO COLLECT CHEST","Chest",function(on)
        Remote:FireServer(on and "FarmStart" or "FarmStop")
    end)

    button("COLLECT CHESTS NOW",function()
        Remote:FireServer("CollectChests")
    end)

    button("AUTO QUEST",function()
        warn("Connect your game's quest system here.")
    end)

    button("STOP ALL FARM",function()
        State.Farm = false
        State.Monster = false
        State.Boss = false
        State.Chest = false

        Remote:FireServer("FarmStop")
    end)
end

--========================================================
-- VISUAL
--========================================================

local ESPObjects = {}

local function removeESP()
    for _, object in pairs(ESPObjects) do
        if object then
            object:Destroy()
        end
    end

    table.clear(ESPObjects)
end

local function makeESP(model, label)

    if not model:IsA("Model") then
        return
    end

    local root =
        model:FindFirstChild("HumanoidRootPart")
        or model.PrimaryPart

    if not root then
        return
    end

    if root:FindFirstChild("PHONGHUY_ESP") then
        return
    end

    local gui = Instance.new("BillboardGui")
    gui.Name = "PHONGHUY_ESP"
    gui.Size = UDim2.fromOffset(150,35)
    gui.StudsOffset = Vector3.new(0,3,0)
    gui.AlwaysOnTop = true
    gui.Adornee = root
    gui.Parent = root

    local text = Instance.new("TextLabel")
    text.Size = UDim2.fromScale(1,1)
    text.BackgroundTransparency = 1
    text.Text = label
    text.TextSize = 12
    text.Font = Enum.Font.GothamBold
    text.TextColor3 = Color3.new(1,1,1)
    text.Parent = gui

    table.insert(ESPObjects,gui)
end

local function loadVisual()

    clearContent()

    section("ESP")

    toggle("MONSTER ESP","MonsterESP",function(on)
        removeESP()

        if on then
            local folder = workspace:FindFirstChild("Monsters")

            if folder then
                for _, model in ipairs(folder:GetChildren()) do
                    makeESP(model,"MONSTER")
                end
            end
        end
    end)

    toggle("BOSS ESP","BossESP",function(on)
        removeESP()

        if on then
            local folder = workspace:FindFirstChild("Bosses")

            if folder then
                for _, model in ipairs(folder:GetChildren()) do
                    makeESP(model,"BOSS")
                end
            end
        end
    end)

    toggle("CHEST ESP","ChestESP",function(on)
        removeESP()

        if on then
            local folder = workspace:FindFirstChild("Chests")

            if folder then
                for _, model in ipairs(folder:GetChildren()) do
                    makeESP(model,"CHEST")
                end
            end
        end
    end)

    toggle("FULL MAP / NO FOG","FullMap",function(on)

        if on then
            Lighting.FogEnd = 1000000
        else
            Lighting.FogEnd = 1000
        end
    end)
end

--========================================================
-- PLAYER
--========================================================

local function loadPlayer()

    clearContent()

    section("PLAYER")

    toggle("SPEED","Speed",function(on)

        local character = Player.Character
        local humanoid =
            character and character:FindFirstChildOfClass("Humanoid")

        if humanoid then
            humanoid.WalkSpeed = on and 32 or 16
        end
    end)

    toggle("JUMP","Jump",function(on)

        local character = Player.Character
        local humanoid =
            character and character:FindFirstChildOfClass("Humanoid")

        if humanoid then
            humanoid.UseJumpPower = true
            humanoid.JumpPower = on and 80 or 50
        end
    end)

    toggle("NOCLIP","NoClip")

    toggle("PERFORMANCE MODE","Performance",function(on)

        if on then
            settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
        else
            settings().Rendering.QualityLevel = Enum.QualityLevel.Automatic
        end
    end)
end

--========================================================
-- CHAT
--========================================================

local function loadChat()

    clearContent()

    section("ROBLOX CHAT")

    local input = Instance.new("TextBox")

    input.Size = UDim2.new(1,-16,0,45)
    input.PlaceholderText = "Nhập tin nhắn..."
    input.Text = ""
    input.TextSize = 13
    input.Font = Enum.Font.Gotham
    input.TextColor3 = Color3.new(1,1,1)
    input.BackgroundColor3 = Color3.fromRGB(43,43,48)
    input.Parent = Content

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0,8)
    corner.Parent = input

    button("SEND TO ROBLOX CHAT",function()

        local message = input.Text

        if message == "" then
            return
        end

        local channels = TextChatService:FindFirstChild("TextChannels")
        local general = channels and channels:FindFirstChild("RBXGeneral")

        if general then
            general:SendAsync(message)
            input.Text = ""
        end
    end)
end

--========================================================
-- ADMIN
--========================================================

local CommandBox

local function loadAdmin()

    clearContent()

    section("ADMIN")

    CommandBox = Instance.new("TextBox")

    CommandBox.Size = UDim2.new(1,-16,0,45)
    CommandBox.PlaceholderText = "kill: PlayerName"
    CommandBox.Text = ""
    CommandBox.TextSize = 13
    CommandBox.Font = Enum.Font.Gotham
    CommandBox.TextColor3 = Color3.new(1,1,1)
    CommandBox.BackgroundColor3 = Color3.fromRGB(43,43,48)
    CommandBox.Parent = Content

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0,8)
    corner.Parent = CommandBox

    button("EXECUTE COMMAND",function()

        if CommandBox.Text ~= "" then
            Remote:FireServer(
                "Command",
                CommandBox.Text
            )

            CommandBox.Text = ""
        end
    end)

    section("EXAMPLES")

    button("kill: PlayerName",function()
        CommandBox.Text = "kill: PlayerName"
    end)

    button("kick: PlayerName",function()
        CommandBox.Text = "kick: PlayerName"
    end)
end

--========================================================
-- TABS
--========================================================

local combatTab = tab("⚔ Combat")
local farmTab = tab("💰 Farm")
local visualTab = tab("👁 Visual")
local playerTab = tab("🏃 Player")
local chatTab = tab("💬 Chat")
local adminTab = tab("⚙ Admin")

combatTab.MouseButton1Click:Connect(loadCombat)
farmTab.MouseButton1Click:Connect(loadFarm)
visualTab.MouseButton1Click:Connect(loadVisual)
playerTab.MouseButton1Click:Connect(loadPlayer)
chatTab.MouseButton1Click:Connect(loadChat)
adminTab.MouseButton1Click:Connect(loadAdmin)

--========================================================
-- NOCLIP
--========================================================

RunService.Stepped:Connect(function()

    if not State.NoClip then
        return
    end

    local character = Player.Character

    if not character then
        return
    end

    for _, part in ipairs(character:GetDescendants()) do
        if part:IsA("BasePart") then
            part.CanCollide = false
        end
    end
end)

--========================================================
-- FLOAT TOGGLE
--========================================================

Float.MouseButton1Click:Connect(function()

    Main.Visible = not Main.Visible

    if Main.Visible then
        Main.Size = UDim2.fromOffset(350,420)

        TweenService:Create(
            Main,
            TweenInfo.new(0.18),
            {
                Size = UDim2.fromOffset(390,460)
            }
        ):Play()
    end
end)

--========================================================
-- DRAG
--========================================================

local dragging = false
local dragStart
local startPos

Title.InputBegan:Connect(function(input)

    if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then

        dragging = true
        dragStart = input.Position
        startPos = Main.Position
    end
end)

Title.InputEnded:Connect(function(input)

    if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then

        dragging = false
    end
end)

Title.InputChanged:Connect(function(input)

    if not dragging then
        return
    end

    if input.UserInputType ~= Enum.UserInputType.MouseMovement
        and input.UserInputType ~= Enum.UserInputType.Touch then
        return
    end

    local delta = input.Position - dragStart

    Main.Position = UDim2.new(
        startPos.X.Scale,
        startPos.X.Offset + delta.X,
        startPos.Y.Scale,
        startPos.Y.Offset + delta.Y
    )
end)

--========================================================
-- ADMIN STATUS
--========================================================

Remote.OnClientEvent:Connect(function(action, value)

    if action == "AdminStatus" then
        State.Admin = value

        if not value then
            adminTab.Visible = false
        end
    end
end)

Remote:FireServer("AdminCheck")

--========================================================
-- START
--========================================================

loadCombat()

print("PHONGHUY36 CLIENT LOADED")
