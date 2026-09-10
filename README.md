--==================================================
-- PHHUYHUB V3 SERVER
-- Roblox Studio / Own Game
--==================================================

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local ADMIN_ID = 11645467051

local Remote = ReplicatedStorage:FindFirstChild("PHHuyHub_V3")

if not Remote then
	Remote = Instance.new("RemoteEvent")
	Remote.Name = "PHHuyHub_V3"
	Remote.Parent = ReplicatedStorage
end

local function isAdmin(player)
	return player.UserId == ADMIN_ID
end

local function findPlayer(text)
	if not text then return nil end

	text = tostring(text):lower()

	for _, player in ipairs(Players:GetPlayers()) do
		if player.Name:lower() == text
			or player.DisplayName:lower() == text then
			return player
		end
	end

	for _, player in ipairs(Players:GetPlayers()) do
		if player.Name:lower():sub(1, #text) == text then
			return player
		end
	end

	return nil
end

local function humanoid(player)
	local character = player and player.Character
	return character and character:FindFirstChildOfClass("Humanoid")
end

local function root(player)
	local character = player and player.Character
	return character and character:FindFirstChild("HumanoidRootPart")
end

local function kill(player)
	local hum = humanoid(player)

	if hum then
		hum.Health = 0
	end
end

local function respawn(player)
	if player then
		player:LoadCharacter()
	end
end

local function freeze(player, enabled)
	local r = root(player)

	if r then
		r.Anchored = enabled
	end
end

local function bring(sender, target)
	local a = root(sender)
	local b = root(target)

	if a and b then
		b.CFrame = a.CFrame * CFrame.new(0, 0, -5)
	end
end

local function teleportToSpawn(player)
	local r = root(player)

	if not r then return end

	local spawnLocation

	for _, obj in ipairs(workspace:GetDescendants()) do
		if obj:IsA("SpawnLocation") then
			spawnLocation = obj
			break
		end
	end

	if spawnLocation then
		r.CFrame =
			spawnLocation.CFrame + Vector3.new(0, 4, 0)
	end
end

local function executeCommand(sender, text)
	if not isAdmin(sender) then
		return
	end

	local args = string.split(text, " ")
	local cmd = string.lower(args[1] or "")
	local target = findPlayer(args[2])

	if cmd == ";kill" and target then

		kill(target)

	elseif cmd == ";kick" and target and target ~= sender then

		target:Kick(
			"Removed by the game administrator."
		)

	elseif cmd == ";respawn" and target then

		respawn(target)

	elseif cmd == ";bring" and target then

		bring(sender, target)

	elseif cmd == ";freeze" and target then

		freeze(target, true)

	elseif cmd == ";unfreeze" and target then

		freeze(target, false)

	elseif cmd == ";tp" and target then

		local senderRoot = root(sender)
		local targetRoot = root(target)

		if senderRoot and targetRoot then
			senderRoot.CFrame =
				targetRoot.CFrame *
				CFrame.new(0, 0, -5)
		end

	elseif cmd == ";spawn" then

		teleportToSpawn(sender)
	end
end

Remote.OnServerEvent:Connect(function(player, action, data)

	if action == "AdminCheck" then

		Remote:FireClient(
			player,
			"AdminStatus",
			isAdmin(player)
		)

	elseif action == "Command" then

		executeCommand(
			player,
			tostring(data or "")
		)
	end
end)

print("[PHHuyHub V3] Server Ready")
--==================================================
-- PHHUYHUB V3 | PART 1/2
-- Roblox Studio / Own Game
--==================================================

local Players=game:GetService("Players")
local Rep=game:GetService("ReplicatedStorage")
local UIS=game:GetService("UserInputService")
local Run=game:GetService("RunService")
local Lighting=game:GetService("Lighting")
local Stats=game:GetService("Stats")

local LP=Players.LocalPlayer
local PG=LP:WaitForChild("PlayerGui")
local Remote=Rep:WaitForChild("PHHuyHub_V3")

local old=PG:FindFirstChild("PHHuyHub_V3")
if old then old:Destroy() end

local S={
	PlayerESP=true,Name=true,Health=true,Tracer=true,
	Box=true,Skeleton=true,NPC=false,Item=false,Monster=false,
	Speed=true,Jump=true,Fly=true,NoClip=false,InfJump=false,
	LowGraphics=false,HighFPS=false,
	Stealth=false,HideName=false,HideEffects=false,
	Animation=true
}

local V={
	Speed=32,
	Jump=70,
	FlySpeed=55,
	Distance=1000
}

local COL={
	BG=Color3.fromRGB(3,7,15),
	PANEL=Color3.fromRGB(5,12,24),
	CARD=Color3.fromRGB(8,18,35),
	CARD2=Color3.fromRGB(10,25,46),
	BLUE=Color3.fromRGB(0,175,255),
	CYAN=Color3.fromRGB(0,235,255),
	TEXT=Color3.fromRGB(235,248,255),
	MUTED=Color3.fromRGB(115,155,180),
	GREEN=Color3.fromRGB(0,235,160)
}

local function corner(x,r)
	local c=Instance.new("UICorner")
	c.CornerRadius=UDim.new(0,r)
	c.Parent=x
end

local function stroke(x,t)
	local s=Instance.new("UIStroke")
	s.Color=COL.BLUE
	s.Thickness=1
	s.Transparency=t or .6
	s.Parent=x
end

local function txt(p,v,z,b)
	local x=Instance.new("TextLabel")
	x.BackgroundTransparency=1
	x.Text=v
	x.TextColor3=COL.TEXT
	x.TextSize=z or 12
	x.Font=b and Enum.Font.GothamBold or Enum.Font.Gotham
	x.TextXAlignment=Enum.TextXAlignment.Left
	x.Parent=p
	return x
end

local GUI=Instance.new("ScreenGui")
GUI.Name="PHHuyHub_V3"
GUI.ResetOnSpawn=false
GUI.IgnoreGuiInset=true
GUI.ZIndexBehavior=Enum.ZIndexBehavior.Sibling
GUI.Parent=PG

--==================================================
-- FLOATING BUTTON
--==================================================

local Float=Instance.new("TextButton")
Float.Size=UDim2.fromOffset(62,62)
Float.Position=UDim2.new(0,15,.5,-31)
Float.BackgroundColor3=COL.PANEL
Float.Text="PH"
Float.TextColor3=COL.CYAN
Float.TextSize=21
Float.Font=Enum.Font.GothamBlack
Float.AutoButtonColor=false
Float.Parent=GUI
corner(Float,22)
stroke(Float,.1)

--==================================================
-- MAIN
--==================================================

local Main=Instance.new("Frame")
Main.AnchorPoint=Vector2.new(.5,.5)
Main.Position=UDim2.fromScale(.5,.5)
Main.Size=UDim2.fromScale(.92,.84)
Main.BackgroundColor3=COL.BG
Main.BorderSizePixel=0
Main.Parent=GUI
corner(Main,18)
stroke(Main,.08)

--==================================================
-- HEADER
--==================================================

local Header=Instance.new("Frame")
Header.Size=UDim2.new(1,0,0,66)
Header.BackgroundColor3=COL.PANEL
Header.BorderSizePixel=0
Header.Parent=Main
corner(Header,18)

local Logo=txt(Header,"PHHuyHub",25,true)
Logo.Position=UDim2.fromOffset(20,7)
Logo.Size=UDim2.fromOffset(300,30)
Logo.TextColor3=COL.CYAN

local Sub=txt(Header,"PLAY  •  FARM  •  DOMINATE",9,true)
Sub.Position=UDim2.fromOffset(22,40)
Sub.Size=UDim2.fromOffset(250,16)
Sub.TextColor3=COL.MUTED

local Stat=Instance.new("Frame")
Stat.AnchorPoint=Vector2.new(1,0)
Stat.Position=UDim2.new(1,-100,0,10)
Stat.Size=UDim2.fromOffset(205,45)
Stat.BackgroundColor3=COL.CARD
Stat.Parent=Header
corner(Stat,9)
stroke(Stat,.75)

local FPS=txt(Stat,"FPS: --",10,true)
FPS.Position=UDim2.fromOffset(10,5)
FPS.Size=UDim2.fromOffset(85,17)
FPS.TextColor3=COL.GREEN

local Ping=txt(Stat,"Ping: --",10,true)
Ping.Position=UDim2.fromOffset(10,24)
Ping.Size=UDim2.fromOffset(85,17)
Ping.TextColor3=COL.CYAN

local Count=txt(Stat,"Player: --",10,true)
Count.Position=UDim2.fromOffset(100,14)
Count.Size=UDim2.fromOffset(95,18)

local Min=Instance.new("TextButton")
Min.Size=UDim2.fromOffset(42,42)
Min.Position=UDim2.new(1,-94,0,12)
Min.BackgroundColor3=COL.CARD
Min.Text="—"
Min.TextColor3=COL.TEXT
Min.TextSize=20
Min.Parent=Header
corner(Min,10)
stroke(Min,.7)

local Close=Instance.new("TextButton")
Close.Size=UDim2.fromOffset(42,42)
Close.Position=UDim2.new(1,-48,0,12)
Close.BackgroundColor3=COL.CARD
Close.Text="×"
Close.TextColor3=COL.TEXT
Close.TextSize=25
Close.Parent=Header
corner(Close,10)
stroke(Close,.7)

--==================================================
-- SIDEBAR
--==================================================

local Side=Instance.new("Frame")
Side.Position=UDim2.new(0,0,0,66)
Side.Size=UDim2.new(0,155,1,-66)
Side.BackgroundColor3=COL.PANEL
Side.BorderSizePixel=0
Side.Parent=Main

local SP=Instance.new("UIPadding")
SP.PaddingTop=UDim.new(0,12)
SP.PaddingLeft=UDim.new(0,10)
SP.PaddingRight=UDim.new(0,10)
SP.Parent=Side

local SL=Instance.new("UIListLayout")
SL.Padding=UDim.new(0,7)
SL.Parent=Side

local Content=Instance.new("Frame")
Content.Position=UDim2.new(0,155,0,66)
Content.Size=UDim2.new(1,-155,1,-66)
Content.BackgroundTransparency=1
Content.Parent=Main

local Pages={}
local Tabs={}

local TabData={
	{"ESP","⌂"},
	{"PLAYER","●"},
	{"ADMIN","♛"},
	{"PERFORMANCE","▥"},
	{"STEALTH","◉"},
	{"SETTINGS","⚙"}
}

local function page(name)
	local p=Instance.new("ScrollingFrame")
	p.Name=name
	p.Size=UDim2.fromScale(1,1)
	p.BackgroundTransparency=1
	p.BorderSizePixel=0
	p.ScrollBarThickness=2
	p.ScrollBarImageColor3=COL.BLUE
	p.AutomaticCanvasSize=Enum.AutomaticSize.Y
	p.CanvasSize=UDim2.new()
	p.Visible=false
	p.Parent=Content

	local pad=Instance.new("UIPadding")
	pad.PaddingTop=UDim.new(0,14)
	pad.PaddingBottom=UDim.new(0,20)
	pad.PaddingLeft=UDim.new(0,14)
	pad.PaddingRight=UDim.new(0,14)
	pad.Parent=p

	local list=Instance.new("UIListLayout")
	list.Padding=UDim.new(0,8)
	list.Parent=p

	Pages[name]=p
	return p
end

for _,d in ipairs(TabData) do
	page(d[1])

	local b=Instance.new("TextButton")
	b.Name=d[1]
	b.Size=UDim2.new(1,0,0,45)
	b.BackgroundColor3=COL.CARD
	b.Text=d[2].."   "..d[1]
	b.TextColor3=COL.TEXT
	b.TextSize=12
	b.Font=Enum.Font.GothamBold
	b.AutoButtonColor=false
	b.Parent=Side
	corner(b,10)
	stroke(b,.82)
	Tabs[d[1]]=b

	b.Activated:Connect(function()
		for n,p in pairs(Pages) do
			p.Visible=n==d[1]
		end
		for n,t in pairs(Tabs) do
			t.BackgroundColor3=
				n==d[1]
				and Color3.fromRGB(0,75,110)
				or COL.CARD
		end
	end)
end

--==================================================
-- CARD
--==================================================

local function card(p,title,sub)
	local c=Instance.new("Frame")
	c.Size=UDim2.new(1,0,0,66)
	c.BackgroundColor3=COL.CARD
	c.Parent=p
	corner(c,11)
	stroke(c,.72)

	local a=txt(c,title,16,true)
	a.Position=UDim2.fromOffset(15,8)
	a.Size=UDim2.new(1,-30,0,24)
	a.TextColor3=COL.CYAN

	local b=txt(c,sub or "",10)
	b.Position=UDim2.fromOffset(16,36)
	b.Size=UDim2.new(1,-30,0,18)
	b.TextColor3=COL.MUTED

	return c
end

--==================================================
-- TOGGLE
--==================================================

local function toggle(p,name,key,callback)

	local r=Instance.new("TextButton")
	r.Size=UDim2.new(1,0,0,40)
	r.BackgroundColor3=COL.CARD2
	r.Text=""
	r.AutoButtonColor=false
	r.Parent=p
	corner(r,8)

	local t=txt(r,name,11,true)
	t.Position=UDim2.fromOffset(12,0)
	t.Size=UDim2.new(1,-70,1,0)

	local sw=Instance.new("Frame")
	sw.AnchorPoint=Vector2.new(1,.5)
	sw.Position=UDim2.new(1,-10,.5,0)
	sw.Size=UDim2.fromOffset(43,22)
	sw.BackgroundColor3=Color3.fromRGB(25,43,61)
	sw.Parent=r
	corner(sw,12)

	local dot=Instance.new("Frame")
	dot.Size=UDim2.fromOffset(16,16)
	dot.Position=UDim2.fromOffset(3,3)
	dot.BackgroundColor3=COL.MUTED
	dot.Parent=sw
	corner(dot,10)

	local function update()
		if S[key] then
			sw.BackgroundColor3=Color3.fromRGB(0,105,150)
			dot.Position=UDim2.new(1,-19,0,3)
			dot.BackgroundColor3=COL.CYAN
		else
			sw.BackgroundColor3=Color3.fromRGB(25,43,61)
			dot.Position=UDim2.fromOffset(3,3)
			dot.BackgroundColor3=COL.MUTED
		end
	end

	r.Activated:Connect(function()
		S[key]=not S[key]
		update()
		if callback then callback(S[key]) end
	end)

	update()
	return r
end

--==================================================
-- SLIDER
--==================================================

local function slider(p,name,key,min,max)

	local h=Instance.new("Frame")
	h.Size=UDim2.new(1,0,0,52)
	h.BackgroundTransparency=1
	h.Parent=p

	local n=txt(h,name,10)
	n.Size=UDim2.new(.7,0,0,18)

	local val=txt(h,tostring(V[key]),10,true)
	val.AnchorPoint=Vector2.new(1,0)
	val.Position=UDim2.new(1,0,0,0)
	val.Size=UDim2.fromOffset(50,18)
	val.TextXAlignment=Enum.TextXAlignment.Right

	local bar=Instance.new("Frame")
	bar.Position=UDim2.new(0,0,0,29)
	bar.Size=UDim2.new(1,0,0,5)
	bar.BackgroundColor3=Color3.fromRGB(25,47,70)
	bar.Parent=h
	corner(bar,5)

	local fill=Instance.new("Frame")
	fill.BackgroundColor3=COL.BLUE
	fill.Parent=bar
	corner(fill,5)

	local knob=Instance.new("Frame")
	knob.Size=UDim2.fromOffset(11,11)
	knob.AnchorPoint=Vector2.new(.5,.5)
	knob.BackgroundColor3=COL.CYAN
	knob.Parent=bar
	corner(knob,8)

	local function update()
		local x=(V[key]-min)/(max-min)
		fill.Size=UDim2.new(x,0,1,0)
		knob.Position=UDim2.new(x,0,.5,0)
		val.Text=tostring(V[key])
	end

	local drag=false

	bar.InputBegan:Connect(function(i)
		if i.UserInputType==Enum.UserInputType.MouseButton1
			or i.UserInputType==Enum.UserInputType.Touch then
			drag=true
		end
	end)

	UIS.InputChanged:Connect(function(i)
		if not drag then return end
		if i.UserInputType==Enum.UserInputType.MouseMovement
			or i.UserInputType==Enum.UserInputType.Touch then

			local q=math.clamp(
				(i.Position.X-bar.AbsolutePosition.X)/
				bar.AbsoluteSize.X,0,1)

			V[key]=math.floor(min+(max-min)*q+.5)
			update()
		end
	end)

	UIS.InputEnded:Connect(function(i)
		if i.UserInputType==Enum.UserInputType.MouseButton1
			or i.UserInputType==Enum.UserInputType.Touch then
			drag=false
		end
	end)

	update()
end

--==================================================
-- ESP PAGE
--==================================================

do
	local p=Pages.ESP

	card(p,"◎  ESP","Hiển thị người chơi, NPC, vật phẩm...")

	toggle(p,"Player ESP","PlayerESP")
	toggle(p,"Tên + Khoảng cách","Name")
	toggle(p,"Hiển thị HP","Health")
	toggle(p,"Đường kẻ (Tracer)","Tracer")
	toggle(p,"Khung (Box)","Box")
	toggle(p,"Skeleton (Xương)","Skeleton")

	slider(p,"Khoảng cách tối đa","Distance",100,3000)

	toggle(p,"ESP NPC","NPC")
	toggle(p,"ESP Vật phẩm","Item")
	toggle(p,"ESP Quái","Monster")
end

--==================================================
-- PLAYER PAGE
--==================================================

do
	local p=Pages.PLAYER

	card(p,"●  PLAYER","Tăng tốc, nhảy, noclip...")

	toggle(p,"Speed","Speed",function(on)
		local c=LP.Character
		local h=c and c:FindFirstChildOfClass("Humanoid")
		if h then h.WalkSpeed=on and V.Speed or 16 end
	end)

	slider(p,"Speed Value","Speed",16,100)

	toggle(p,"Jump","Jump",function(on)
		local c=LP.Character
		local h=c and c:FindFirstChildOfClass("Humanoid")
		if h then h.JumpPower=on and V.Jump or 50 end
	end)

	slider(p,"Jump Value","Jump",50,150)

	toggle(p,"Fly","Fly")
	slider(p,"Fly Speed","FlySpeed",20,120)

	toggle(p,"NoClip","NoClip")
	toggle(p,"Infinite Jump","InfJump")
end

--==================================================
-- PLACEHOLDER PAGES
-- PART 2 FILLS THESE
--==================================================

card(Pages.ADMIN,"♛  ADMIN SERVER","Lệnh quản trị server")
card(Pages.PERFORMANCE,"▥  PERFORMANCE","Hiệu suất & thông tin hệ thống")
card(Pages.STEALTH,"◉  STEALTH TEST","Kiểm thử visibility")
card(Pages.SETTINGS,"⚙  SETTINGS","Cài đặt & tùy chỉnh")

Pages.ESP.Visible=true
Tabs.ESP.BackgroundColor3=Color3.fromRGB(0,75,110)

print("[PHHuyHub V3] PART 1 READY")
--==================================================
-- PHHUYHUB V3 | PART 2/2
--==================================================

--==================================================
-- ADMIN
--==================================================

do
	local p=Pages.ADMIN

	for _,x in ipairs(p:GetChildren()) do
		if x:IsA("Frame") then x:Destroy() end
	end

	card(p,"♛  ADMIN SERVER","Lệnh quản trị & công cụ server")

	local Status=txt(p,"● Đang kiểm tra quyền...",11,true)
	Status.Size=UDim2.new(1,0,0,25)
	Status.TextColor3=COL.MUTED

	local Box=Instance.new("TextBox")
	Box.Size=UDim2.new(1,0,0,45)
	Box.BackgroundColor3=COL.CARD
	Box.TextColor3=COL.TEXT
	Box.PlaceholderColor3=COL.MUTED
	Box.PlaceholderText="Nhập lệnh..."
	Box.Text=""
	Box.TextSize=12
	Box.Font=Enum.Font.Code
	Box.ClearTextOnFocus=false
	Box.Parent=p
	corner(Box,9)
	stroke(Box,.72)

	local Send=Instance.new("TextButton")
	Send.Size=UDim2.new(1,0,0,45)
	Send.BackgroundColor3=Color3.fromRGB(0,70,105)
	Send.Text="➤  GỬI LỆNH SERVER"
	Send.TextColor3=COL.TEXT
	Send.TextSize=12
	Send.Font=Enum.Font.GothamBold
	Send.Parent=p
	corner(Send,9)
	stroke(Send,.5)

	Send.Activated:Connect(function()
		if Box.Text~="" then
			Remote:FireServer("Command",Box.Text)
			Box.Text=""
		end
	end)

	local Help=txt(p,
		";kill Name\n"..
		";kick Name\n"..
		";respawn Name\n"..
		";tp Name\n"..
		";bring Name\n"..
		";freeze Name\n"..
		";unfreeze Name\n"..
		";spawn",
		11)

	Help.Size=UDim2.new(1,0,0,170)
	Help.TextColor3=COL.MUTED

	Remote.OnClientEvent:Connect(function(action,value)
		if action=="AdminStatus" then
			if value then
				Status.Text="● ADMIN AUTHORIZED"
				Status.TextColor3=COL.GREEN
				Logo.Text="PHHuyHub  •  ADMIN"
			else
				Status.Text="● USER"
				Status.TextColor3=COL.MUTED
			end
		end
	end)

	Remote:FireServer("AdminCheck")
end

--==================================================
-- PERFORMANCE
--==================================================

do
	local p=Pages.PERFORMANCE

	for _,x in ipairs(p:GetChildren()) do
		if x:IsA("Frame") then x:Destroy() end
	end

	card(p,"▥  PERFORMANCE","Hiệu suất & thông tin hệ thống")

	local BigFPS=txt(p,"FPS     --",17,true)
	BigFPS.Size=UDim2.new(1,0,0,32)
	BigFPS.TextColor3=COL.GREEN

	local BigPing=txt(p,"Ping    --",12,true)
	BigPing.Size=UDim2.new(1,0,0,28)
	BigPing.TextColor3=COL.CYAN

	toggle(p,"High FPS","HighFPS")

	toggle(p,"Low Graphics","LowGraphics",function(on)
		Lighting.GlobalShadows=not on
		if on then Lighting.Brightness=1 end
	end)

	local Mode=txt(p,"Performance Mode: ACTIVE",11)
	Mode.Size=UDim2.new(1,0,0,30)
	Mode.TextColor3=COL.MUTED

	local frames=0
	local start=os.clock()

	Run.RenderStepped:Connect(function()
		frames+=1

		if os.clock()-start>=1 then
			local f=math.floor(frames)

			FPS.Text="FPS: "..f
			BigFPS.Text="FPS     "..f
			Count.Text="Player: "..#Players:GetPlayers()

			frames=0
			start=os.clock()
		end
	end)

	task.spawn(function()
		while GUI.Parent do
			task.wait(1)

			local ok,v=pcall(function()
				return Stats.Network.ServerStatsItem["Data Ping"]:GetValue()
			end)

			if ok then
				local q=math.floor(v)

				Ping.Text="Ping: "..q.."ms"
				BigPing.Text="Ping    "..q.."ms"
			end
		end
	end)
end

--==================================================
-- STEALTH
--==================================================

do
	local p=Pages.STEALTH

	for _,x in ipairs(p:GetChildren()) do
		if x:IsA("Frame") then x:Destroy() end
	end

	card(p,"◉  STEALTH TEST",
		"Kiểm thử visibility phía client")

	local function visibility(on)

		local c=LP.Character
		if not c then return end

		for _,x in ipairs(c:GetDescendants()) do
			if x:IsA("BasePart") then
				x.LocalTransparencyModifier=on and 1 or 0
			elseif x:IsA("Decal") then
				x.Transparency=on and 1 or 0
			end
		end
	end

	toggle(p,"Stealth (Local Test)","Stealth",visibility)

	toggle(p,"Ẩn tên","HideName",function(on)
		local c=LP.Character
		local h=c and c:FindFirstChildOfClass("Humanoid")

		if h then
			h.DisplayDistanceType=
				on
				and Enum.HumanoidDisplayDistanceType.None
				or Enum.HumanoidDisplayDistanceType.Viewer
		end
	end)

	toggle(p,"Ẩn hiệu ứng","HideEffects")

	local info=txt(p,
		"Stealth là kiểm thử visibility phía client\n"..
		"trong game Roblox của bạn.",
		11)

	info.Size=UDim2.new(1,0,0,55)
	info.TextColor3=COL.MUTED
end

--==================================================
-- SETTINGS
--==================================================

do
	local p=Pages.SETTINGS

	for _,x in ipairs(p:GetChildren()) do
		if x:IsA("Frame") then x:Destroy() end
	end

	card(p,"⚙  SETTINGS","Cài đặt & tùy chỉnh")

	toggle(p,"UI Animation","Animation")

	local theme=txt(p,"Theme       NEON BLUE",12,true)
	theme.Size=UDim2.new(1,0,0,35)
	theme.TextColor3=COL.CYAN

	local lang=txt(p,"Language    Tiếng Việt",12,true)
	lang.Size=UDim2.new(1,0,0,35)

	local device=txt(p,"Device       Mobile • Tablet • PC",11)
	device.Size=UDim2.new(1,0,0,35)
	device.TextColor3=COL.MUTED

	local Reset=Instance.new("TextButton")
	Reset.Size=UDim2.new(1,0,0,45)
	Reset.BackgroundColor3=COL.CARD
	Reset.Text="↻  RESET CÀI ĐẶT"
	Reset.TextColor3=COL.CYAN
	Reset.TextSize=12
	Reset.Font=Enum.Font.GothamBold
	Reset.Parent=p
	corner(Reset,9)
	stroke(Reset,.7)

	Reset.Activated:Connect(function()
		S.Speed=false
		S.Jump=false
		S.Fly=false
		S.NoClip=false
		S.InfJump=false

		local c=LP.Character
		local h=c and c:FindFirstChildOfClass("Humanoid")

		if h then
			h.WalkSpeed=16
			h.JumpPower=50
		end
	end)
end

--==================================================
-- OPEN / CLOSE
--==================================================

local Opened=true

Float.Activated:Connect(function()
	Opened=not Opened
	Main.Visible=Opened
end)

Close.Activated:Connect(function()
	Opened=false
	Main.Visible=false
end)

Min.Activated:Connect(function()
	Main.Visible=false
end)

--==================================================
-- DRAG MOBILE / PC
--==================================================

local dragging=false
local dragStart
local startPos

Header.InputBegan:Connect(function(i)
	if i.UserInputType==Enum.UserInputType.MouseButton1
		or i.UserInputType==Enum.UserInputType.Touch then

		dragging=true
		dragStart=i.Position
		startPos=Main.Position
	end
end)

UIS.InputChanged:Connect(function(i)
	if not dragging then return end

	if i.UserInputType==Enum.UserInputType.MouseMovement
		or i.UserInputType==Enum.UserInputType.Touch then

		local d=i.Position-dragStart

		Main.Position=UDim2.new(
			startPos.X.Scale,
			startPos.X.Offset+d.X,
			startPos.Y.Scale,
			startPos.Y.Offset+d.Y
		)
	end
end)

UIS.InputEnded:Connect(function(i)
	if i.UserInputType==Enum.UserInputType.MouseButton1
		or i.UserInputType==Enum.UserInputType.Touch then
		dragging=false
	end
end)

--==================================================
-- MOBILE RESPONSIVE
--==================================================

local function Layout()

	local cam=workspace.CurrentCamera
	if not cam then return end

	local w=cam.ViewportSize.X

	if w<=700 then

		Main.Size=UDim2.fromScale(.97,.93)

		Side.Size=UDim2.new(0,58,1,-66)

		Content.Position=UDim2.new(0,58,0,66)

		Content.Size=UDim2.new(1,-58,1,-66)

		Sub.Visible=false
		Stat.Visible=false

		for _,d in ipairs(TabData) do
			Tabs[d[1]].Text=d[2]
			Tabs[d[1]].TextSize=17
		end

	elseif w<=1000 then

		Main.Size=UDim2.fromScale(.95,.89)

		Side.Size=UDim2.new(0,100,1,-66)

		Content.Position=UDim2.new(0,100,0,66)

		Content.Size=UDim2.new(1,-100,1,-66)

	else

		Main.Size=UDim2.fromScale(.92,.84)

		Side.Size=UDim2.new(0,155,1,-66)

		Content.Position=UDim2.new(0,155,0,66)

		Content.Size=UDim2.new(1,-155,1,-66)

		Sub.Visible=true
		Stat.Visible=true

		for _,d in ipairs(TabData) do
			Tabs[d[1]].Text=d[2].."   "..d[1]
			Tabs[d[1]].TextSize=12
		end
	end
end

workspace.CurrentCamera:
	GetPropertyChangedSignal("ViewportSize"):
	Connect(Layout)

Layout()

--==================================================
-- NOCLIP
--==================================================

Run.Stepped:Connect(function()

	if not S.NoClip then return end

	local c=LP.Character

	if c then
		for _,x in ipairs(c:GetDescendants()) do
			if x:IsA("BasePart") then
				x.CanCollide=false
			end
		end
	end
end)

--==================================================
-- INFINITE JUMP
--==================================================

UIS.JumpRequest:Connect(function()

	if not S.InfJump then return end

	local c=LP.Character
	local h=c and c:FindFirstChildOfClass("Humanoid")

	if h then
		h:ChangeState(
			Enum.HumanoidStateType.Jumping
		)
	end
end)

--==================================================
-- FLY
--==================================================

Run.RenderStepped:Connect(function()

	if not S.Fly then return end

	local c=LP.Character
	if not c then return end

	local r=c:FindFirstChild("HumanoidRootPart")
	local h=c:FindFirstChildOfClass("Humanoid")

	if r and h then

		local d=h.MoveDirection

		r.AssemblyLinearVelocity=Vector3.new(
			d.X*V.FlySpeed,
			0,
			d.Z*V.FlySpeed
		)
	end
end)

--==================================================
-- LIVE SPEED / JUMP
--==================================================

Run.Heartbeat:Connect(function()

	local c=LP.Character
	local h=c and c:FindFirstChildOfClass("Humanoid")

	if not h then return end

	if S.Speed then
		h.WalkSpeed=V.Speed
	end

	if S.Jump then
		h.JumpPower=V.Jump
	end
end)

print("================================")
print(" PHHUYHUB V3 • NEON BLUE")
print(" CLIENT FULLY LOADED")
print("================================")
