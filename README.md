-- Положить в StarterPlayer → StarterPlayerScripts

local Players      = game:GetService("Players")
local RepStorage   = game:GetService("ReplicatedStorage")
local ChatService  = game:GetService("Chat")
local TweenService = game:GetService("TweenService")
local RunService   = game:GetService("RunService")

local player    = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Утилита для обрезки пробелов
local function trim(s) return s:match("^%s*(.-)%s*$") end

-- Таблица цветов (для Part и GUI)
local colorMap = {
	red      = Color3.fromRGB(255,   0,   0), ["красный"]   = Color3.fromRGB(255,   0,   0),
	green    = Color3.fromRGB(  0, 255,   0), ["зелёный"]   = Color3.fromRGB(  0, 255,   0), ["зеленый"]=Color3.fromRGB(0,255,0),
	blue     = Color3.fromRGB(  0,   0, 255), ["синий"]     = Color3.fromRGB(  0,   0, 255),
	yellow   = Color3.fromRGB(255, 255,   0), ["жёлтый"]    = Color3.fromRGB(255, 255,   0), ["желтый"]=Color3.fromRGB(255,255,0),
	black    = Color3.fromRGB(  0,   0,   0), ["чёрный"]    = Color3.fromRGB(  0,   0,   0), ["черный"]=Color3.fromRGB(0,0,0),
	white    = Color3.fromRGB(255, 255, 255), ["белый"]     = Color3.fromRGB(255, 255, 255),
}

-- 1) GUI: ввод команд, кнопка «Ввести», меню и консоль логов
local screenGui = Instance.new("ScreenGui")
screenGui.Name         = "CommandInterface"
screenGui.ResetOnSpawn = false
screenGui.Parent       = playerGui

local frame = Instance.new("Frame", screenGui)
frame.Name                   = "CMDFrame"
frame.Size                   = UDim2.new(0, 450, 0, 60)
frame.Position               = UDim2.new(0.5, -225, 1, -70)
frame.BackgroundColor3       = Color3.fromRGB(25, 25, 25)
frame.BackgroundTransparency = 0.1
frame.BorderSizePixel        = 0
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 12)

local textBox = Instance.new("TextBox", frame)
textBox.Name             = "CommandBox"
textBox.Size             = UDim2.new(0, 330, 1, -12)
textBox.Position         = UDim2.new(0, 10, 0, 6)
textBox.PlaceholderText  = "Введите команды которые в консоли"
textBox.ClearTextOnFocus = false
textBox.Font             = Enum.Font.Gotham
textBox.TextSize         = 18
textBox.TextColor3       = Color3.fromRGB(230, 230, 230)
textBox.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
textBox.BorderSizePixel  = 0
Instance.new("UICorner", textBox).CornerRadius = UDim.new(0, 8)

local sendButton = Instance.new("TextButton", frame)
sendButton.Name             = "SendButton"
sendButton.Size             = UDim2.new(0, 90, 1, -12)
sendButton.Position         = UDim2.new(0, 350, 0, 6)
sendButton.Text             = "Ввести"
sendButton.Font             = Enum.Font.GothamBold
sendButton.TextSize         = 18
sendButton.TextColor3       = Color3.fromRGB(245, 245, 245)
sendButton.BackgroundColor3 = Color3.fromRGB(0, 135, 255)
sendButton.BorderSizePixel  = 0
Instance.new("UICorner", sendButton).CornerRadius = UDim.new(0, 8)
sendButton.MouseEnter:Connect(function()
	TweenService:Create(sendButton, TweenInfo.new(0.15), { BackgroundColor3 = Color3.fromRGB(0, 180, 255) }):Play()
end)
sendButton.MouseLeave:Connect(function()
	TweenService:Create(sendButton, TweenInfo.new(0.15), { BackgroundColor3 = Color3.fromRGB(0, 135, 255) }):Play()
end)

local menuButton = Instance.new("TextButton", screenGui)
menuButton.Name             = "MenuButton"
menuButton.Size             = UDim2.new(0, 24, 0, 24)
menuButton.Position         = UDim2.new(0.5, 225, 1, -110)
menuButton.Text             = "☰"
menuButton.Font             = Enum.Font.GothamBold
menuButton.TextSize         = 20
menuButton.TextColor3       = Color3.fromRGB(200, 200, 200)
menuButton.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
menuButton.BorderSizePixel  = 0
Instance.new("UICorner", menuButton).CornerRadius = UDim.new(0, 6)

local console = Instance.new("ScrollingFrame", screenGui)
console.Name                 = "ConsoleFrame"
console.Size                 = UDim2.new(0, 450, 0, 200)
console.Position             = UDim2.new(0.5, -225, 1, -280)
console.BackgroundColor3     = Color3.fromRGB(20, 20, 20)
console.BackgroundTransparency = 0.2
console.BorderSizePixel      = 0
console.Visible              = false
console.CanvasSize           = UDim2.new(0, 0, 0, 0)
console.ScrollBarThickness   = 4
Instance.new("UICorner", console).CornerRadius = UDim.new(0, 12)

local layout = Instance.new("UIListLayout", console)
layout.SortOrder = Enum.SortOrder.LayoutOrder
layout.Padding   = UDim.new(0, 4)
layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
	console.CanvasSize     = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 8)
	console.CanvasPosition = Vector2.new(0, console.CanvasSize.Y.Offset)
end)

local function addLog(text)
	local lbl = Instance.new("TextLabel", console)
	lbl.Size                   = UDim2.new(1, -10, 0, 20)
	lbl.Position               = UDim2.new(0, 5, 0, 0)
	lbl.BackgroundTransparency = 1
	lbl.Font                   = Enum.Font.Gotham
	lbl.TextSize               = 16
	lbl.TextColor3             = Color3.fromRGB(180, 240, 120)
	lbl.TextXAlignment         = Enum.TextXAlignment.Left
	lbl.Text                   = text
end

menuButton.MouseButton1Click:Connect(function()
	console.Visible = not console.Visible
end)

-- 2) Чат без изменений
local chatFolder = RepStorage:WaitForChild("DefaultChatSystemChatEvents", 5)
local sayEvent   = chatFolder and chatFolder:WaitForChild("SayMessageRequest", 5)
local function SendChatMessage(msg)
	if sayEvent then
		sayEvent:FireServer(msg, "All")
	else
		local head = (player.Character or player.CharacterAdded:Wait()):FindFirstChild("Head")
		if head then
			ChatService:Chat(head, msg, Enum.ChatColor.Blue)
		else
			addLog("Ошибка: Head не найден")
		end
	end
end

-- 3) Движение и базовые команды
local directions = {
	up    = Vector3.new(0, 0, -1),
	down  = Vector3.new(0, 0,  1),
	left  = Vector3.new(-1,0,  0),
	right = Vector3.new(1, 0,  0),
}

local originalSpeed
local function moveCharacter(dirVector, duration)
	local hum = (player.Character or player.CharacterAdded:Wait()):FindFirstChild("Humanoid")
	if not hum then addLog("Ошибка: Humanoid не найден"); return end
	local start = tick()
	task.spawn(function()
		while tick() - start <= duration do
			hum:Move(dirVector, true)
			RunService.RenderStepped:Wait()
		end
	end)
end

local function doJump(count)
	local hum = (player.Character or player.CharacterAdded:Wait()):FindFirstChild("Humanoid")
	if not hum then addLog("Ошибка: Humanoid не найден"); return end
	count = math.floor(count)
	addLog("> Jump("..count..")")
	task.spawn(function()
		for i = 1, count do
			hum.Jump = true
			task.wait(0.5)
		end
	end)
end

local function setSpeed(speed)
	local hum = (player.Character or player.CharacterAdded:Wait()):FindFirstChild("Humanoid")
	if not hum then addLog("Ошибка: Humanoid не найден"); return end
	originalSpeed = originalSpeed or hum.WalkSpeed
	hum.WalkSpeed = speed
	addLog("> Speed="..speed)
end

local function resetSpeed()
	local hum = (player.Character or player.CharacterAdded:Wait()):FindFirstChild("Humanoid")
	if not hum then addLog("Ошибка: Humanoid не найден"); return end
	if originalSpeed then
		hum.WalkSpeed = originalSpeed
		addLog("> Speed сброшена:"..originalSpeed)
		originalSpeed = nil
	else
		addLog("> оригинальная скорость неизвестна")
	end
end

local function teleportTo(arg)
	local x,y,z = arg:match("([%-%.%d]+)%s*,%s*([%-%.%d]+)%s*,%s*([%-%.%d]+)")
	if not x then addLog("> Teleport требует x,y,z"); return end
	local hrp = (player.Character or player.CharacterAdded:Wait()):FindFirstChild("HumanoidRootPart")
	if not hrp then addLog("Ошибка: HRP не найден"); return end
	hrp.CFrame = CFrame.new(tonumber(x), tonumber(y), tonumber(z))
	addLog("> Teleport="..x..","..y..","..z)
end

local function setSit(state)
	local hum = (player.Character or player.CharacterAdded:Wait()):FindFirstChild("Humanoid")
	if not hum then addLog("Ошибка: Humanoid не найден"); return end
	hum.Sit = state
	addLog("> "..(state and "Sit" or "Stand"))
end

-- 4) Создание форм: Box, Ball и Circle
local function createShape(argRaw)
	local parts = {}
	for v in string.gmatch(argRaw, "[^,]+") do
		table.insert(parts, trim(v))
	end
	if #parts < 3 then
		addLog("> Create требует: shape, color, anchor")
		return
	end

	local shape    = parts[1]:lower()
	local colorArg = parts[2]:lower()
	local ancArg   = parts[3]:lower()

	local col = colorMap[colorArg]
	if not col then
		addLog("> неизвестный цвет: "..parts[2])
		return
	end

	local anchored = false
	if ancArg == "anchored" or ancArg == "анкор" or ancArg == "true" or ancArg == "да" then
		anchored = true
	elseif ancArg == "unanchored" or ancArg == "неанкор" or ancArg == "false" or ancArg == "нет" then
		anchored = false
	else
		addLog("> неизвестный anchor: "..parts[3])
		return
	end

	local hrp = (player.Character or player.CharacterAdded:Wait()):FindFirstChild("HumanoidRootPart")
	local baseCFrame = hrp and hrp.CFrame * CFrame.new(0,5,0) or CFrame.new(0,5,0)

	if shape == "part" then
		local obj = Instance.new("Part", workspace)
		obj.Color    = col
		obj.Anchored = anchored
		obj.Size     = Vector3.new(4,1,2)
		obj.CFrame   = baseCFrame
		addLog("> Created Part: color="..parts[2])

	elseif shape == "ball" then
		local obj = Instance.new("Part", workspace)
		obj.Shape    = Enum.PartType.Ball
		obj.Color    = col
		obj.Anchored = anchored
		obj.Size     = Vector3.new(4,4,4)
		obj.CFrame   = baseCFrame
		addLog("> Created Ball: color="..parts[2])

	elseif shape == "circle" then
		-- Outer ring
		local outer = Instance.new("Part", workspace)
		outer.Shape            = Enum.PartType.Cylinder
		outer.Color            = col
		outer.Anchored         = anchored
		outer.Size             = Vector3.new(6,1,6)
		outer.CFrame           = baseCFrame * CFrame.Angles(math.rad(90),0,0)
		-- Inner hole
		local inner = Instance.new("Part", workspace)
		inner.Shape            = Enum.PartType.Cylinder
		inner.Color            = Color3.new(1,1,1)
		inner.Transparency     = 1
		inner.CanCollide       = false
		inner.Anchored         = anchored
		inner.Size             = Vector3.new(2,1.2,2)
		inner.CFrame           = baseCFrame * CFrame.Angles(math.rad(90),0,0)
		addLog("> Created Circle: color="..parts[2])

	else
		addLog("> неизвестная форма: "..shape)
	end
end

-- 6) Resize UI (для будущих нужд)
local uiInstances = {}
local function resizeUI(argRaw)
	local parts = {}
	for v in string.gmatch(argRaw, "[^,]+") do
		table.insert(parts, trim(v))
	end
	if #parts < 3 then
		addLog("> Resize требует: name, width, height")
		return
	end
	local name = parts[1]
	local w, h = tonumber(parts[2]), tonumber(parts[3])
	local obj = uiInstances[name]
	if not obj then
		addLog("> UI не найден: "..name)
		return
	end
	if not w or not h then
		addLog("> неверный размер")
		return
	end
	obj.Size = UDim2.new(0, w, 0, h)
end

-- 7) Обработка команд
local function processCommand(raw)
	local cmdRaw, argRaw = raw:match("^(%w+)%s*%((.-)%)$")
	if not cmdRaw then
		addLog("> неверный формат: "..raw)
		return
	end
	local cmd = cmdRaw:lower()

	if cmd == "chat" then
		local msg = argRaw:match('^"(.*)"$')
		if msg then
			addLog("> chat: "..msg)
			SendChatMessage(msg)
		else
			addLog("> chat требует строку в кавычках")
		end

	elseif directions[cmd] and tonumber(argRaw) then
		addLog("> "..cmd:upper().." "..argRaw.."с")
		moveCharacter(directions[cmd], tonumber(argRaw))

	elseif cmd == "jump"       and tonumber(argRaw) then doJump(tonumber(argRaw))
	elseif cmd == "speed"      and tonumber(argRaw) then setSpeed(tonumber(argRaw))
	elseif cmd == "resetspeed"                         then resetSpeed()
	elseif cmd == "teleport"    then teleportTo(argRaw)
	elseif cmd == "sit"                                 then setSit(true)
	elseif cmd == "stand"                               then setSit(false)

	elseif cmd == "create"     then createShape(argRaw)
	elseif cmd == "resize"     then resizeUI(argRaw)

	else
		addLog("> неизвестная команда: "..cmdRaw)
	end
end

-- 7) Подключение ввода
sendButton.MouseButton1Click:Connect(function()
	if textBox.Text ~= "" then
		processCommand(textBox.Text)
		textBox.Text = ""
	end
end)
textBox.FocusLost:Connect(function(enter)
	if enter and textBox.Text ~= "" then
		processCommand(textBox.Text)
		textBox.Text = ""
	end
end)

-- 8) Приветствие и список команд
task.delay(0.5, function()
	addLog("Консоль готова. Доступные команды:")
	addLog('chat("…")           — отправить в чат')
	addLog("Up(sec)            — вперёд")
	addLog("Down(sec)          — назад")
	addLog("Left(sec)          — влево")
	addLog("Right(sec)         — вправо")
	addLog("Jump(count)        — прыгнуть count раз")
	addLog("Speed(value)       — установить скорость")
	addLog("ResetSpeed()       — сбросить скорость")
	addLog("Teleport(x,y,z)    — телепорт")
	addLog("Sit()              — сесть")
	addLog("Stand()            — встать")
	addLog("Create(shape,color,anchor) — создать форму (part, ball, circle)")
	addLog("  color: red/красный, green/зелёный, blue/синий, yellow/жёлтый, black/чёрный, white/белый")
	addLog("  anchor: anchored/анкор (true), unanchored/неанкор (false)")
end)
