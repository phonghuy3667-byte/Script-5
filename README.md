--========================================================
-- PHHUYHUB V3 - SERVER
-- Roblox Studio / Own Game
--========================================================

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

--========================================================
-- CONFIG
--========================================================

local ADMIN_IDS = {
	[11645467051] = true,
}

local REMOTE_NAME = "PHHuyHub_V3"

--========================================================
-- REMOTE
--========================================================

local Remote = ReplicatedStorage:FindFirstChild(REMOTE_NAME)

if not Remote then
	Remote = Instance.new("RemoteEvent")
	Remote.Name = REMOTE_NAME
	Remote.Parent = ReplicatedStorage
end

--========================================================
-- ADMIN
--========================================================

local function isAdmin(player)
	return player ~= nil and ADMIN_IDS[player.UserId] == true
end

local function findPlayer(name)
	if not name or name == "" then
		return nil
	end

	name = tostring(name):lower()

	-- Exact username/display name
	for _, player in ipairs(Players:GetPlayers()) do
		if player.Name:lower() == name
			or player.DisplayName:lower() == name then
			return player
		end
	end

	-- Partial username/display name
	for _, player in ipairs(Players:GetPlayers()) do
		if player.Name:lower():sub(1, #name) == name
			or player.DisplayName:lower():sub(1, #name) == name then
			return player
		end
	end

	return nil
end

--========================================================
-- CHARACTER HELPERS
--========================================================

local function getCharacter(player)
	return player and player.Character
end

local function getHumanoid(player)
	local character = getCharacter(player)

	if not character then
		return nil
	end

	return character:FindFirstChildOfClass("Humanoid")
end

local function getRoot(player)
	local character = getCharacter(player)

	if not character then
		return nil
	end

	return character:FindFirstChild("HumanoidRootPart")
end

--========================================================
-- ADMIN ACTIONS
--========================================================

local function killPlayer(target)
	local humanoid = getHumanoid(target)

	if humanoid then
		humanoid.Health = 0
		return true
	end

	return false
end

local function respawnPlayer(target)
	if target then
		target:LoadCharacter()
		return true
	end

	return false
end

local function freezePlayer(target)
	local root = getRoot(target)

	if root then
		root.Anchored = true
		return true
	end

	return false
end

local function unfreezePlayer(target)
	local root = getRoot(target)

	if root then
		root.Anchored = false
		return true
	end

	return false
end

local function bringPlayer(sender, target)
	local senderRoot = getRoot(sender)
	local targetRoot = getRoot(target)

	if senderRoot and targetRoot then
		targetRoot.CFrame =
			senderRoot.CFrame * CFrame.new(0, 0, -5)

		return true
	end

	return false
end

local function teleportToPlayer(sender, target)
	local senderRoot = getRoot(sender)
	local targetRoot = getRoot(target)

	if senderRoot and targetRoot then
		senderRoot.CFrame =
			targetRoot.CFrame * CFrame.new(0, 0, -5)

		return true
	end

	return false
end

local function teleportToSpawn(player)
	local root = getRoot(player)

	if not root then
		return false
	end

	for _, object in ipairs(workspace:GetDescendants()) do
		if object:IsA("SpawnLocation") then
			root.CFrame =
				object.CFrame + Vector3.new(0, 5, 0)

			return true
		end
	end

	return false
end

--========================================================
-- COMMAND SYSTEM
--========================================================

local function executeCommand(sender, text)
	if not isAdmin(sender) then
		return
	end

	text = tostring(text or "")

	if text == "" then
		return
	end

	local args = string.split(text, " ")
	local command = string.lower(args[1] or "")
	local targetName = args[2]

	local target = findPlayer(targetName)

	if command == ";kill" then

		if target then
			killPlayer(target)
		end

	elseif command == ";kick" then

		if target and target ~= sender then
			target:Kick("Removed by game administrator.")
		end

	elseif command == ";respawn" then

		if target then
			respawnPlayer(target)
		end

	elseif command == ";tp" then

		if target then
			teleportToPlayer(sender, target)
		end

	elseif command == ";bring" then

		if target then
			bringPlayer(sender, target)
		end

	elseif command == ";freeze" then

		if target then
			freezePlayer(target)
		end

	elseif command == ";unfreeze" then

		if target then
			unfreezePlayer(target)
		end

	elseif command == ";spawn" then

		teleportToSpawn(sender)
	end
end

--========================================================
-- REMOTE HANDLER
--========================================================

Remote.OnServerEvent:Connect(function(player, action, data)

	if action == "AdminCheck" then

		Remote:FireClient(
			player,
			"AdminStatus",
			isAdmin(player)
		)

		return
	end

	if action == "Command" then

		executeCommand(
			player,
			tostring(data or "")
		)

		return
	end
end)

--========================================================
-- PLAYER JOIN
--========================================================

Players.PlayerAdded:Connect(function(player)

	task.delay(1, function()

		if player.Parent then
			Remote:FireClient(
				player,
				"AdminStatus",
				isAdmin(player)
			)
		end

	end)

end)

print("======================================")
print(" PHHUYHUB V3 SERVER ONLINE")
print(" Remote:", REMOTE_NAME)
print(" Admin ID:", 11645467051)
print("======================================")
--========================================================
-- PHHUYHUB V3 - CLIENT
-- Roblox Studio / Own Game
--========================================================

--========================================================
-- SERVICES
--========================================================

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")
local Stats = game:GetService("Stats")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

--========================================================
-- REMOTE
--========================================================

local Remote = ReplicatedStorage:WaitForChild(
	"PHHuyHub_V3",
	10
)

if not Remote then
	warn("PHHuyHub V3: RemoteEvent not found")
	return
end

--========================================================
-- CONFIG
--========================================================

local Config = {
	Speed = 16,
	Jump = 50,

	Fly = false,
	NoClip = false,
	InfiniteJump = false,

	PlayerESP = false,
	NPCESP = false,
	MonsterESP = false,
	ItemESP = false,

	HighFPS = false,
	HideEffects = false,

	Stealth = false,

	UIVisible = true,
	UIScale = 1,
	Theme = "CYBER",
}

--========================================================
-- THEME
--========================================================

local Themes = {
	CYBER = {
		Background = Color3.fromRGB(7, 9, 15),
		Panel = Color3.fromRGB(14, 18, 29),
		Card = Color3.fromRGB(17, 21, 34),
		Accent = Color3.fromRGB(0, 162, 255),
		Accent2 = Color3.fromRGB(0, 225, 255),
		Text = Color3.fromRGB(240, 245, 255),
		SubText = Color3.fromRGB(145, 160, 180),
	},

	MIDNIGHT = {
		Background = Color3.fromRGB(5, 6, 10),
		Panel = Color3.fromRGB(12, 13, 20),
		Card = Color3.fromRGB(18, 19, 28),
		Accent = Color3.fromRGB(90, 120, 255),
		Accent2 = Color3.fromRGB(130, 150, 255),
		Text = Color3.fromRGB(240, 240, 250),
		SubText = Color3.fromRGB(145, 145, 165),
	},

	PURPLE = {
		Background = Color3.fromRGB(9, 6, 15),
		Panel = Color3.fromRGB(18, 12, 27),
		Card = Color3.fromRGB(25, 17, 37),
		Accent = Color3.fromRGB(150, 70, 255),
		Accent2 = Color3.fromRGB(205, 110, 255),
		Text = Color3.fromRGB(245, 240, 255),
		SubText = Color3.fromRGB(165, 145, 180),
	},

	OCEAN = {
		Background = Color3.fromRGB(4, 12, 17),
		Panel = Color3.fromRGB(8, 21, 29),
		Card = Color3.fromRGB(12, 30, 39),
		Accent = Color3.fromRGB(0, 190, 180),
		Accent2 = Color3.fromRGB(0, 235, 220),
		Text = Color3.fromRGB(235, 255, 255),
		SubText = Color3.fromRGB(135, 170, 175),
	},
}

local Theme = Themes[Config.Theme]

--========================================================
-- CLEAN OLD UI
--========================================================

local old = PlayerGui:FindFirstChild("PHHuyHub_V3")

if old then
	old:Destroy()
end

--========================================================
-- HELPERS
--========================================================

local function create(className, properties, parent)
	local object = Instance.new(className)

	for property, value in pairs(properties or {}) do
		object[property] = value
	end

	object.Parent = parent

	return object
end

local function corner(object, radius)
	return create("UICorner", {
		CornerRadius = UDim.new(0, radius or 8),
	}, object)
end

local function stroke(object, color, transparency)
	return create("UIStroke", {
		Color = color or Theme.Accent,
		Transparency = transparency or 0.65,
		Thickness = 1,
	}, object)
end

local function tween(object, properties, duration)
	local info = TweenInfo.new(
		duration or 0.2,
		Enum.EasingStyle.Quart,
		Enum.EasingDirection.Out
	)

	TweenService:Create(
		object,
		info,
		properties
	):Play()
end

--========================================================
-- SCREEN GUI
--========================================================

local ScreenGui = create("ScreenGui", {
	Name = "PHHuyHub_V3",
	ResetOnSpawn = false,
	IgnoreGuiInset = true,
	ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
	DisplayOrder = 999,
}, PlayerGui)

--========================================================
-- FLOAT BUTTON
--========================================================

local Float = create("TextButton", {
	Name = "PHButton",
	Size = UDim2.fromOffset(56, 56),
	Position = UDim2.new(0, 18, 0.5, -28),
	BackgroundColor3 = Theme.Panel,
	Text = "PH",
	TextColor3 = Theme.Text,
	TextSize = 19,
	Font = Enum.Font.GothamBold,
	AutoButtonColor = false,
}, ScreenGui)

corner(Float, 28)
stroke(Float, Theme.Accent, 0.25)

--========================================================
-- MAIN WINDOW
--========================================================

local Main = create("Frame", {
	Name = "Main",
	Size = UDim2.new(0, 700, 0, 430),
	Position = UDim2.new(0.5, -350, 0.5, -215),
	BackgroundColor3 = Theme.Background,
	BorderSizePixel = 0,
}, ScreenGui)

corner(Main, 14)
stroke(Main, Theme.Accent, 0.35)

--========================================================
-- RESPONSIVE
--========================================================

local function updateResponsive()

	local camera = workspace.CurrentCamera

	if not camera then
		return
	end

	local viewport = camera.ViewportSize

	if viewport.X < 650 then

		Main.Size = UDim2.new(
			1,
			-20,
			0,
			math.min(520, viewport.Y - 30)
		)

		Main.Position = UDim2.new(
			0.5,
			0,
			0.5,
			0
		)

		Main.AnchorPoint = Vector2.new(0.5, 0.5)

	else

		Main.Size = UDim2.fromOffset(700, 430)

		Main.Position = UDim2.new(
			0.5,
			0,
			0.5,
			0
		)

		Main.AnchorPoint = Vector2.new(0.5, 0.5)
	end
end

updateResponsive()

workspace.CurrentCamera:GetPropertyChangedSignal(
	"ViewportSize"
):Connect(updateResponsive)

--========================================================
-- HEADER
--========================================================

local Header = create("Frame", {
	Size = UDim2.new(1, 0, 0, 55),
	BackgroundTransparency = 1,
}, Main)

local Title = create("TextLabel", {
	Position = UDim2.fromOffset(18, 7),
	Size = UDim2.new(1, -130, 0, 25),
	BackgroundTransparency = 1,
	Text = "PHHUYHUB V3",
	TextColor3 = Theme.Text,
	TextSize = 19,
	Font = Enum.Font.GothamBold,
	TextXAlignment = Enum.TextXAlignment.Left,
}, Header)

local Subtitle = create("TextLabel", {
	Position = UDim2.fromOffset(19, 31),
	Size = UDim2.new(1, -130, 0, 17),
	BackgroundTransparency = 1,
	Text = "PHONGHUY36  •  PRIVATE TEST",
	TextColor3 = Theme.SubText,
	TextSize = 10,
	Font = Enum.Font.Gotham,
	TextXAlignment = Enum.TextXAlignment.Left,
}, Header)

local Min = create("TextButton", {
	Position = UDim2.new(1, -80, 0, 13),
	Size = UDim2.fromOffset(28, 28),
	BackgroundColor3 = Theme.Card,
	Text = "—",
	TextColor3 = Theme.Text,
	TextSize = 17,
	Font = Enum.Font.GothamBold,
	AutoButtonColor = false,
}, Header)

corner(Min, 8)

local Close = create("TextButton", {
	Position = UDim2.new(1, -45, 0, 13),
	Size = UDim2.fromOffset(28, 28),
	BackgroundColor3 = Theme.Card,
	Text = "×",
	TextColor3 = Theme.Text,
	TextSize = 20,
	Font = Enum.Font.GothamBold,
	AutoButtonColor = false,
}, Header)

corner(Close, 8)

--========================================================
-- DRAG
--========================================================

local dragging = false
local dragStart
local startPosition

Header.InputBegan:Connect(function(input)

	if input.UserInputType == Enum.UserInputType.MouseButton1
		or input.UserInputType == Enum.UserInputType.Touch then

		dragging = true
		dragStart = input.Position
		startPosition = Main.Position
	end
end)

UserInputService.InputChanged:Connect(function(input)

	if not dragging then
		return
	end

	if input.UserInputType ~= Enum.UserInputType.MouseMovement
		and input.UserInputType ~= Enum.UserInputType.Touch then
		return
	end

	local delta = input.Position - dragStart

	Main.Position = UDim2.new(
		startPosition.X.Scale,
		startPosition.X.Offset + delta.X,
		startPosition.Y.Scale,
		startPosition.Y.Offset + delta.Y
	)
end)

UserInputService.InputEnded:Connect(function(input)

	if input.UserInputType == Enum.UserInputType.MouseButton1
		or input.UserInputType == Enum.UserInputType.Touch then

		dragging = false
	end
end)

--========================================================
-- SIDEBAR
--========================================================

local Sidebar = create("Frame", {
	Position = UDim2.fromOffset(10, 65),
	Size = UDim2.new(0, 145, 1, -75),
	BackgroundColor3 = Theme.Panel,
	BorderSizePixel = 0,
}, Main)

corner(Sidebar, 10)

local SideLayout = create("UIListLayout", {
	Padding = UDim.new(0, 6),
	HorizontalAlignment = Enum.HorizontalAlignment.Center,
	SortOrder = Enum.SortOrder.LayoutOrder,
}, Sidebar)

create("UIPadding", {
	PaddingTop = UDim.new(0, 10),
	PaddingBottom = UDim.new(0, 10),
}, Sidebar)

--========================================================
-- CONTENT
--========================================================

local Content = create("Frame", {
	Position = UDim2.fromOffset(165, 65),
	Size = UDim2.new(1, -175, 1, -75),
	BackgroundTransparency = 1,
}, Main)

--========================================================
-- TABS
--========================================================

local Tabs = {}
local Pages = {}

local function createPage(name)

	local page = create("ScrollingFrame", {
		Name = name,
		Size = UDim2.fromScale(1, 1),
		BackgroundTransparency = 1,
		BorderSizePixel = 0,
		ScrollBarThickness = 3,
		ScrollBarImageColor3 = Theme.Accent,
		Visible = false,
		CanvasSize = UDim2.new(0, 0, 0, 0),
		AutomaticCanvasSize = Enum.AutomaticSize.Y,
	}, Content)

	create("UIPadding", {
		PaddingLeft = UDim.new(0, 5),
		PaddingRight = UDim.new(0, 5),
		PaddingBottom = UDim.new(0, 10),
	}, page)

	local layout = create("UIListLayout", {
		Padding = UDim.new(0, 9),
		SortOrder = Enum.SortOrder.LayoutOrder,
	}, page)

	Pages[name] = page

	return page
end

local function createTab(name, order)

	local button = create("TextButton", {
		Size = UDim2.new(1, -18, 0, 39),
		BackgroundColor3 = Theme.Card,
		Text = name,
		TextColor3 = Theme.SubText,
		TextSize = 12,
		Font = Enum.Font.GothamMedium,
		AutoButtonColor = false,
		LayoutOrder = order,
	}, Sidebar)

	corner(button, 8)

	Tabs[name] = button

	return button
end

local tabNames = {
	"ESP",
	"PLAYER",
	"ADMIN",
	"PERFORMANCE",
	"STEALTH",
	"SETTINGS",
}

for index, name in ipairs(tabNames) do
	createTab(name, index)
	createPage(name)
end

local function showPage(name)

	for tabName, page in pairs(Pages) do
		page.Visible = tabName == name
	end

	for tabName, button in pairs(Tabs) do

		if tabName == name then
			button.BackgroundColor3 = Theme.Accent
			button.TextColor3 = Color3.new(1, 1, 1)
		else
			button.BackgroundColor3 = Theme.Card
			button.TextColor3 = Theme.SubText
		end
	end
end

for name, button in pairs(Tabs) do
	button.Activated:Connect(function()
		showPage(name)
	end)
end

showPage("ESP")

--========================================================
-- UI CARD
--========================================================

local function card(parent, title, description)

	local frame = create("Frame", {
		Size = UDim2.new(1, -5, 0, 80),
		BackgroundColor3 = Theme.Card,
		BorderSizePixel = 0,
	}, parent)

	corner(frame, 10)
	stroke(frame, Theme.Accent, 0.85)

	local titleLabel = create("TextLabel", {
		Position = UDim2.fromOffset(14, 9),
		Size = UDim2.new(1, -28, 0, 20),
		BackgroundTransparency = 1,
		Text = title,
		TextColor3 = Theme.Text,
		TextSize = 13,
		Font = Enum.Font.GothamBold,
		TextXAlignment = Enum.TextXAlignment.Left,
	}, frame)

	if description then

		create("TextLabel", {
			Position = UDim2.fromOffset(14, 31),
			Size = UDim2.new(1, -28, 0, 35),
			BackgroundTransparency = 1,
			Text = description,
			TextColor3 = Theme.SubText,
			TextSize = 10,
			Font = Enum.Font.Gotham,
			TextWrapped = true,
			TextXAlignment = Enum.TextXAlignment.Left,
			TextYAlignment = Enum.TextYAlignment.Top,
		}, frame)

	end

	return frame
end

--========================================================
-- TOGGLE
--========================================================

local function toggle(parent, title, key, callback)

	local frame = create("Frame", {
		Size = UDim2.new(1, -5, 0, 48),
		BackgroundColor3 = Theme.Card,
		BorderSizePixel = 0,
	}, parent)

	corner(frame, 8)

	local label = create("TextLabel", {
		Position = UDim2.fromOffset(12, 0),
		Size = UDim2.new(1, -75, 1, 0),
		BackgroundTransparency = 1,
		Text = title,
		TextColor3 = Theme.Text,
		TextSize = 12,
		Font = Enum.Font.GothamMedium,
		TextXAlignment = Enum.TextXAlignment.Left,
	}, frame)

	local button = create("TextButton", {
		Position = UDim2.new(1, -58, 0.5, -12),
		Size = UDim2.fromOffset(46, 24),
		BackgroundColor3 = Theme.Panel,
		Text = "",
		AutoButtonColor = false,
	}, frame)

	corner(button, 12)

	local knob = create("Frame", {
		Position = UDim2.fromOffset(3, 3),
		Size = UDim2.fromOffset(18, 18),
		BackgroundColor3 = Theme.SubText,
		BorderSizePixel = 0,
	}, button)

	corner(knob, 9)

	local function update()

		local state = Config[key] == true

		if state then
			button.BackgroundColor3 = Theme.Accent
			knob.Position = UDim2.new(1, -21, 0, 3)
		else
			button.BackgroundColor3 = Theme.Panel
			knob.Position = UDim2.fromOffset(3, 3)
		end

		if callback then
			callback(state)
		end
	end

	button.Activated:Connect(function()
		Config[key] = not Config[key]
		update()
	end)

	update()

	return frame
end

--========================================================
-- SLIDER
--========================================================

local function slider(parent, title, key, minValue, maxValue)

	local frame = create("Frame", {
		Size = UDim2.new(1, -5, 0, 70),
		BackgroundColor3 = Theme.Card,
		BorderSizePixel = 0,
	}, parent)

	corner(frame, 8)

	local label = create("TextLabel", {
		Position = UDim2.fromOffset(12, 8),
		Size = UDim2.new(1, -75, 0, 20),
		BackgroundTransparency = 1,
		Text = title,
		TextColor3 = Theme.Text,
		TextSize = 12,
		Font = Enum.Font.GothamMedium,
		TextXAlignment = Enum.TextXAlignment.Left,
	}, frame)

	local valueLabel = create("TextLabel", {
		Position = UDim2.new(1, -65, 0, 8),
		Size = UDim2.fromOffset(52, 20),
		BackgroundTransparency = 1,
		Text = tostring(Config[key]),
		TextColor3 = Theme.Accent2,
		TextSize = 11,
		Font = Enum.Font.GothamBold,
		TextXAlignment = Enum.TextXAlignment.Right,
	}, frame)

	local bar = create("Frame", {
		Position = UDim2.fromOffset(12, 40),
		Size = UDim2.new(1, -24, 0, 7),
		BackgroundColor3 = Theme.Panel,
		BorderSizePixel = 0,
	}, frame)

	corner(bar, 5)

	local fill = create("Frame", {
		Size = UDim2.new(
			(Config[key] - minValue) /
			(maxValue - minValue),
			0,
			1,
			0
		),
		BackgroundColor3 = Theme.Accent,
		BorderSizePixel = 0,
	}, bar)

	corner(fill, 5)

	local function setFromX(x)

		local percent = math.clamp(
			(x - bar.AbsolutePosition.X) /
			bar.AbsoluteSize.X,
			0,
			1
		)

		local value =
			minValue +
			(maxValue - minValue) * percent

		value = math.floor(value + 0.5)

		Config[key] = value

		valueLabel.Text = tostring(value)

		fill.Size = UDim2.new(percent, 0, 1, 0)
	end

	bar.InputBegan:Connect(function(input)

		if input.UserInputType == Enum.UserInputType.MouseButton1
			or input.UserInputType == Enum.UserInputType.Touch then

			setFromX(input.Position.X)

			local connection

			connection = UserInputService.InputChanged:Connect(function(move)

				if move.UserInputType == Enum.UserInputType.MouseMovement
					or move.UserInputType == Enum.UserInputType.Touch then

					setFromX(move.Position.X)
				end
			end)

			local ended

			ended = UserInputService.InputEnded:Connect(function(endInput)

				if endInput.UserInputType == Enum.UserInputType.MouseButton1
					or endInput.UserInputType == Enum.UserInputType.Touch then

					connection:Disconnect()
					ended:Disconnect()
				end
			end)
		end
	end)

	return frame
end

--========================================================
-- ESP PAGE
--========================================================

local ESPPage = Pages.ESP

card(
	ESPPage,
	"ESP DEBUG",
	"Developer visualization tools for your own game."
)

toggle(
	ESPPage,
	"Player ESP",
	"PlayerESP"
)

toggle(
	ESPPage,
	"NPC ESP",
	"NPCESP"
)

toggle(
	ESPPage,
	"Monster ESP",
	"MonsterESP"
)

toggle(
	ESPPage,
	"Item ESP",
	"ItemESP"
)

slider(
	ESPPage,
	"ESP Distance",
	"ESPDistance",
	50,
	2000
)

--========================================================
-- PLAYER PAGE
--========================================================

local PlayerPage = Pages.PLAYER

card(
	PlayerPage,
	"PLAYER CONTROL",
	"Movement testing for your own Roblox experience."
)

slider(
	PlayerPage,
	"WalkSpeed",
	"Speed",
	8,
	100
)

slider(
	PlayerPage,
	"JumpPower",
	"Jump",
	25,
	150
)

toggle(
	PlayerPage,
	"Fly",
	"Fly"
)

toggle(
	PlayerPage,
	"NoClip",
	"NoClip"
)

toggle(
	PlayerPage,
	"Infinite Jump",
	"InfiniteJump"
)

--========================================================
-- CHARACTER CONTROL
--========================================================

local function applyMovement()

	local character = LocalPlayer.Character

	if not character then
		return
	end

	local humanoid =
		character:FindFirstChildOfClass("Humanoid")

	if not humanoid then
		return
	end

	humanoid.WalkSpeed = Config.Speed
	humanoid.JumpPower = Config.Jump
end

LocalPlayer.CharacterAdded:Connect(function()

	task.wait(0.5)

	applyMovement()
end)

RunService.Heartbeat:Connect(function()

	applyMovement()

	if Config.NoClip then

		local character = LocalPlayer.Character

		if character then

			for _, object in ipairs(character:GetDescendants()) do

				if object:IsA("BasePart") then
					object.CanCollide = false
				end
			end
		end
	end
end)

UserInputService.JumpRequest:Connect(function()

	if Config.InfiniteJump then

		local humanoid =
			LocalPlayer.Character
			and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")

		if humanoid then
			humanoid:ChangeState(
				Enum.HumanoidStateType.Jumping
			)
		end
	end
end)

--========================================================
-- FLY
--========================================================

local FlyConnection

local function stopFly()

	if FlyConnection then
		FlyConnection:Disconnect()
		FlyConnection = nil
	end
end

local function startFly()

	stopFly()

	FlyConnection = RunService.RenderStepped:Connect(function()

		if not Config.Fly then
			return
		end

		local character = LocalPlayer.Character

		if not character then
			return
		end

		local root =
			character:FindFirstChild("HumanoidRootPart")

		local humanoid =
			character:FindFirstChildOfClass("Humanoid")

		if not root or not humanoid then
			return
		end

		local camera = workspace.CurrentCamera

		if not camera then
			return
		end

		local direction = humanoid.MoveDirection

		if direction.Magnitude > 0 then
			root.AssemblyLinearVelocity =
				direction * 50
		else
			root.AssemblyLinearVelocity =
				Vector3.zero
		end
	end)
end

startFly()

--========================================================
-- ADMIN PAGE
--========================================================

local AdminPage = Pages.ADMIN

card(
	AdminPage,
	"ADMIN SERVER",
	"Server-authoritative commands. Admin access is checked by UserId."
)

local CommandBox = create("TextBox", {
	Size = UDim2.new(1, -5, 0, 44),
	BackgroundColor3 = Theme.Card,
	Text = "",
	PlaceholderText = ";kill Name",
	PlaceholderColor3 = Theme.SubText,
	TextColor3 = Theme.Text,
	TextSize = 12,
	Font = Enum.Font.Gotham,
	ClearTextOnFocus = false,
}, AdminPage)

corner(CommandBox)
stroke(CommandBox, Theme.Accent, 0.8)

create("UIPadding", {
	PaddingLeft = UDim.new(0, 12),
	PaddingRight = UDim.new(0, 12),
}, CommandBox)

local SendCommand = create("TextButton", {
	Size = UDim2.new(1, -5, 0, 42),
	BackgroundColor3 = Theme.Accent,
	Text --========================================================
-- ADMIN PAGE - CONTINUED
--========================================================

local SendCommand = create("TextButton", {
	Size = UDim2.new(1, -5, 0, 42),
	BackgroundColor3 = Theme.Accent,
	Text = "SEND COMMAND",
	TextColor3 = Color3.new(1, 1, 1),
	TextSize = 12,
	Font = Enum.Font.GothamBold,
	AutoButtonColor = false,
}, AdminPage)

corner(SendCommand, 8)

local AdminStatus = create("TextLabel", {
	Size = UDim2.new(1, -5, 0, 35),
	BackgroundTransparency = 1,
	Text = "Ready.",
	TextColor3 = Theme.SubText,
	TextSize = 11,
	Font = Enum.Font.Gotham,
	TextWrapped = true,
	TextXAlignment = Enum.TextXAlignment.Left,
}, AdminPage)

local function sendAdminCommand()
	local command = CommandBox.Text

	if command == "" then
		AdminStatus.Text = "⚠ Nhập command trước."
		return
	end

	Remote:FireServer("Command", command)
	AdminStatus.Text = "✓ Đã gửi: " .. command
end

SendCommand.Activated:Connect(sendAdminCommand)

CommandBox.FocusLost:Connect(function(enterPressed)
	if enterPressed then
		sendAdminCommand()
	end
end)

Remote.OnClientEvent:Connect(function(action, data)
	if action == "AdminStatus" then
		AdminStatus.Text = tostring(data)
	end
end)

--========================================================
-- PERFORMANCE PAGE
--========================================================

local PerformancePage = Pages.PERFORMANCE

card(
	PerformancePage,
	"PERFORMANCE MONITOR",
	"Real-time client performance information."
)

local FPSLabel = create("TextLabel", {
	Size = UDim2.new(1, -5, 0, 42),
	BackgroundColor3 = Theme.Card,
	Text = "FPS: --",
	TextColor3 = Theme.Text,
	TextSize = 14,
	Font = Enum.Font.GothamBold,
	TextXAlignment = Enum.TextXAlignment.Left,
}, PerformancePage)

corner(FPSLabel)

local PingLabel = create("TextLabel", {
	Size = UDim2.new(1, -5, 0, 42),
	BackgroundColor3 = Theme.Card,
	Text = "Ping: --",
	TextColor3 = Theme.Text,
	TextSize = 14,
	Font = Enum.Font.GothamBold,
	TextXAlignment = Enum.TextXAlignment.Left,
}, PerformancePage)

corner(PingLabel)

local PlayersLabel = create("TextLabel", {
	Size = UDim2.new(1, -5, 0, 42),
	BackgroundColor3 = Theme.Card,
	Text = "Players: 0",
	TextColor3 = Theme.Text,
	TextSize = 14,
	Font = Enum.Font.GothamBold,
	TextXAlignment = Enum.TextXAlignment.Left,
}, PerformancePage)

corner(PlayersLabel)

toggle(
	PerformancePage,
	"High FPS Mode",
	"HighFPS",
	function(enabled)
		if enabled then
			RunService:Set3dRenderingEnabled(true)
		end
	end
)

toggle(
	PerformancePage,
	"Hide Visual Effects",
	"HideEffects"
)

--========================================================
-- FPS / PING MONITOR
--========================================================

local frames = 0
local fps = 0
local lastFPSUpdate = os.clock()

RunService.RenderStepped:Connect(function()
	frames += 1

	local now = os.clock()

	if now - lastFPSUpdate >= 1 then
		fps = frames
		frames = 0
		lastFPSUpdate = now

		FPSLabel.Text = "FPS: " .. tostring(fps)
	end
end)

task.spawn(function()
	while ScreenGui.Parent do
		task.wait(1)

		local ping = 0

		pcall(function()
			local network = Stats.Network
			local serverStats = network:FindFirstChild("ServerStatsItem")

			if serverStats then
				local dataPing = serverStats:FindFirstChild("Data Ping")

				if dataPing then
					ping = dataPing:GetValue()
				end
			end
		end)

		if ping == 0 then
			pcall(function()
				ping = LocalPlayer:GetNetworkPing() * 1000
			end)
		end

		PingLabel.Text = string.format(
			"Ping: %d ms",
			math.floor(ping + 0.5)
		)

		PlayersLabel.Text =
			"Players: " .. tostring(#Players:GetPlayers())
	end
end)

--========================================================
-- STEALTH PAGE
--========================================================

local StealthPage = Pages.STEALTH

card(
	StealthPage,
	"STEALTH TEST",
	"Local visibility test for your own Roblox experience."
)

local StealthInfo = create("TextLabel", {
	Size = UDim2.new(1, -5, 0, 55),
	BackgroundColor3 = Theme.Card,
	Text = "Stealth OFF\nCharacter đang hiển thị bình thường.",
	TextColor3 = Theme.Text,
	TextSize = 11,
	Font = Enum.Font.Gotham,
	TextWrapped = true,
	TextXAlignment = Enum.TextXAlignment.Left,
	TextYAlignment = Enum.TextYAlignment.Center,
}, StealthPage)

corner(StealthInfo)

local SavedTransparency = {}

local function setStealth(enabled)
	local character = LocalPlayer.Character

	if not character then
		return
	end

	if enabled then
		SavedTransparency = {}

		for _, object in ipairs(character:GetDescendants()) do
			if object:IsA("BasePart") or object:IsA("Decal") then
				SavedTransparency[object] = object.Transparency
				object.Transparency = 1
			elseif object:IsA("ParticleEmitter")
				or object:IsA("Trail")
				or object:IsA("Beam") then
				object.Enabled = false
			end
		end

		StealthInfo.Text =
			"Stealth ON\nLocal character visibility đã tắt."
	else
		for object, transparency in pairs(SavedTransparency) do
			if object and object.Parent then
				object.Transparency = transparency
			end
		end

		SavedTransparency = {}

		StealthInfo.Text =
			"Stealth OFF\nCharacter đang hiển thị bình thường."
	end
end

toggle(
	StealthPage,
	"Stealth Test",
	"Stealth",
	setStealth
)

--========================================================
-- SETTINGS PAGE
--========================================================

local SettingsPage = Pages.SETTINGS

card(
	SettingsPage,
	"SETTINGS",
	"UI configuration for PHHuyHub V3."
)

local ThemeButton = create("TextButton", {
	Size = UDim2.new(1, -5, 0, 44),
	BackgroundColor3 = Theme.Card,
	Text = "THEME: " .. Config.Theme,
	TextColor3 = Theme.Text,
	TextSize = 12,
	Font = Enum.Font.GothamBold,
	AutoButtonColor = false,
}, SettingsPage)

corner(ThemeButton)

local ScaleButton = create("TextButton", {
	Size = UDim2.new(1, -5, 0, 44),
	BackgroundColor3 = Theme.Card,
	Text = "UI SCALE: " .. tostring(Config.UIScale),
	TextColor3 = Theme.Text,
	TextSize = 12,
	Font = Enum.Font.GothamBold,
	AutoButtonColor = false,
}, SettingsPage)

corner(ScaleButton)

local ResetButton = create("TextButton", {
	Size = UDim2.new(1, -5, 0, 44),
	BackgroundColor3 = Theme.Accent,
	Text = "RESET SETTINGS",
	TextColor3 = Color3.new(1, 1, 1),
	TextSize = 12,
	Font = Enum.Font.GothamBold,
	AutoButtonColor = false,
}, SettingsPage)

corner(ResetButton)

--========================================================
-- THEME CYCLE
--========================================================

local ThemeOrder = {
	"CYBER",
	"MIDNIGHT",
	"PURPLE",
	"OCEAN",
}

local function nextTheme()
	local index = table.find(ThemeOrder, Config.Theme) or 1
	index += 1

	if index > #ThemeOrder then
		index = 1
	end

	Config.Theme = ThemeOrder[index]
	Theme = Themes[Config.Theme]

	ThemeButton.Text = "THEME: " .. Config.Theme
end

ThemeButton.Activated:Connect(nextTheme)

--========================================================
-- UI SCALE
--========================================================

local UIScaleObject = create("UIScale", {
	Scale = Config.UIScale,
}, Main)

local ScaleValues = {
	0.8,
	0.9,
	1,
	1.1,
	1.2,
}

local function nextScale()
	local current = table.find(ScaleValues, Config.UIScale) or 3

	current += 1

	if current > #ScaleValues then
		current = 1
	end

	Config.UIScale = ScaleValues[current]
	UIScaleObject.Scale = Config.UIScale

	ScaleButton.Text =
		"UI SCALE: " .. tostring(Config.UIScale)
end

ScaleButton.Activated:Connect(nextScale)

--========================================================
-- RESET
--========================================================

ResetButton.Activated:Connect(function()

	Config.Speed = 16
	Config.Jump = 50

	Config.Fly = false
	Config.NoClip = false
	Config.InfiniteJump = false

	Config.PlayerESP = false
	Config.NPCESP = false
	Config.MonsterESP = false
	Config.ItemESP = false

	Config.HighFPS = false
	Config.HideEffects = false

	Config.Stealth = false

	Config.UIScale = 1
	Config.Theme = "CYBER"

	Theme = Themes[Config.Theme]

	UIScaleObject.Scale = 1

	ThemeButton.Text = "THEME: CYBER"
	ScaleButton.Text = "UI SCALE: 1"

	setStealth(false)

	applyMovement()
end)

--========================================================
-- MINIMIZE / CLOSE
--========================================================

local Opened = true
local Closed = false

Min.Activated:Connect(function()

	if Closed then
		return
	end

	Opened = not Opened

	Content.Visible = Opened
	Sidebar.Visible = Opened

	if Opened then
		Main.Size = UDim2.fromOffset(700, 430)
		Min.Text = "—"
	else
		Main.Size = UDim2.fromOffset(270, 55)
		Min.Text = "+"
	end
end)

Close.Activated:Connect(function()

	Closed = true
	Opened = false

	Main.Visible = false
	Float.Visible = true
end)

--========================================================
-- FLOAT BUTTON
--========================================================

Float.Activated:Connect(function()

	if Closed then
		Closed = false
	end

	Main.Visible = true
	Opened = true

	Content.Visible = true
	Sidebar.Visible = true

	Main.Size = UDim2.fromOffset(700, 430)
	Min.Text = "—"
end)

--========================================================
-- FLOAT BUTTON DRAG
--========================================================

local floatDragging = false
local floatStart
local floatPosition

Float.InputBegan:Connect(function(input)

	if input.UserInputType == Enum.UserInputType.MouseButton1
		or input.UserInputType == Enum.UserInputType.Touch then

		floatDragging = true
		floatStart = input.Position
		floatPosition = Float.Position
	end
end)

UserInputService.InputChanged:Connect(function(input)

	if not floatDragging then
		return
	end

	if input.UserInputType ~= Enum.UserInputType.MouseMovement
		and input.UserInputType ~= Enum.UserInputType.Touch then
		return
	end

	local delta = input.Position - floatStart

	Float.Position = UDim2.new(
		floatPosition.X.Scale,
		floatPosition.X.Offset + delta.X,
		floatPosition.Y.Scale,
		floatPosition.Y.Offset + delta.Y
	)
end)

UserInputService.InputEnded:Connect(function(input)

	if input.UserInputType == Enum.UserInputType.MouseButton1
		or input.UserInputType == Enum.UserInputType.Touch then

		floatDragging = false
	end
end)

--========================================================
-- CHARACTER RESPAWN
--========================================================

LocalPlayer.CharacterAdded:Connect(function(character)

	task.wait(0.5)

	applyMovement()

	if Config.Stealth then
		setStealth(true)
	end
end)

--========================================================
-- INITIAL STATE
--========================================================

Main.Visible = true
Float.Visible = true
Content.Visible = true
Sidebar.Visible = true

Opened = true
Closed = false

print("PHHuyHub V3 Client loaded successfully.")
