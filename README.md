local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local Camera = workspace.CurrentCamera

local ScreenGui = Instance.new("ScreenGui")
local Frame = Instance.new("Frame")
local StatusLabel = Instance.new("TextLabel")
local TargetLabel = Instance.new("TextLabel")
local InfoLabel = Instance.new("TextLabel")
local MinimizeButton = Instance.new("TextButton")
local CloseButton = Instance.new("TextButton")

ScreenGui.Name = "MXP_Studios"
ScreenGui.Parent = game.CoreGui
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

Frame.Name = "MainFrame"
Frame.Parent = ScreenGui
Frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
Frame.BorderSizePixel = 0
Frame.Position = UDim2.new(0.02, 0, 0.02, 0)
Frame.Size = UDim2.new(0, 220, 0, 140)
Frame.Active = true
Frame.Draggable = true
Frame.Visible = true

StatusLabel.Parent = Frame
StatusLabel.BackgroundTransparency = 1
StatusLabel.Position = UDim2.new(0, 5, 0, 5)
StatusLabel.Size = UDim2.new(1, -10, 0, 20)
StatusLabel.Font = Enum.Font.GothamBold
StatusLabel.Text = "Aimlock: DESATIVADO"
StatusLabel.TextColor3 = Color3.fromRGB(255, 50, 50)
StatusLabel.TextSize = 14
StatusLabel.TextXAlignment = Enum.TextXAlignment.Left

TargetLabel.Parent = Frame
TargetLabel.BackgroundTransparency = 1
TargetLabel.Position = UDim2.new(0, 5, 0, 30)
TargetLabel.Size = UDim2.new(1, -10, 0, 20)
TargetLabel.Font = Enum.Font.Gotham
TargetLabel.Text = "Alvo: Nenhum"
TargetLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
TargetLabel.TextSize = 12
TargetLabel.TextXAlignment = Enum.TextXAlignment.Left

InfoLabel.Parent = Frame
InfoLabel.BackgroundTransparency = 1
InfoLabel.Position = UDim2.new(0, 5, 0, 55)
InfoLabel.Size = UDim2.new(1, -10, 0, 70)
InfoLabel.Font = Enum.Font.Code
InfoLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
InfoLabel.TextSize = 13
InfoLabel.TextXAlignment = Enum.TextXAlignment.Left
InfoLabel.TextYAlignment = Enum.TextYAlignment.Top
InfoLabel.Text = [[
[F] — Aimlock
[G] — Noclip
[E] — Auto Interact
]]

MinimizeButton.Parent = Frame
MinimizeButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
MinimizeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
MinimizeButton.Font = Enum.Font.GothamBold
MinimizeButton.TextSize = 12
MinimizeButton.Text = "-"
MinimizeButton.Size = UDim2.new(0, 25, 0, 20)
MinimizeButton.Position = UDim2.new(1, -55, 0, 5)
MinimizeButton.BorderSizePixel = 0

CloseButton.Parent = Frame
CloseButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseButton.Font = Enum.Font.GothamBold
CloseButton.TextSize = 12
CloseButton.Text = "X"
CloseButton.Size = UDim2.new(0, 25, 0, 20)
CloseButton.Position = UDim2.new(1, -28, 0, 5)
CloseButton.BorderSizePixel = 0

local BillboardGui = Instance.new("BillboardGui")
BillboardGui.Size = UDim2.new(0, 200, 0, 50)
BillboardGui.StudsOffset = Vector3.new(0, 4, 0)
BillboardGui.AlwaysOnTop = true
BillboardGui.Enabled = false

local HealthText = Instance.new("TextLabel")
HealthText.Parent = BillboardGui
HealthText.BackgroundTransparency = 1
HealthText.Size = UDim2.new(1, 0, 1, 0)
HealthText.Font = Enum.Font.GothamBold
HealthText.TextColor3 = Color3.fromRGB(255, 0, 0)
HealthText.TextStrokeTransparency = 0
HealthText.TextSize = 22
HealthText.Text = ""

local Aimlock = {
	Enabled = false,
	Target = nil,
	Key = Enum.KeyCode.F,
	AimPart = "Head",
	Smoothing = 0.25,
	MaxDistance = 60,
	Line = nil,
	Connection = nil
}

local NoclipEnabled = false
local AutoInteractEnabled = false
local Minimized = false

MinimizeButton.MouseButton1Click:Connect(function()
	Minimized = not Minimized
	for _, child in ipairs(Frame:GetChildren()) do
		if child ~= MinimizeButton and child ~= CloseButton then
			child.Visible = not Minimized
		end
	end
end)

CloseButton.MouseButton1Click:Connect(function()
	ScreenGui:Destroy()
end)

local function GetClosestPlayer()
	local closest, dist = nil, Aimlock.MaxDistance
	local char = LocalPlayer.Character
	if not char or not char:FindFirstChild("HumanoidRootPart") then return nil end
	for _, p in pairs(Players:GetPlayers()) do
		if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
			local d = (p.Character.HumanoidRootPart.Position - char.HumanoidRootPart.Position).Magnitude
			if d < dist then
				dist = d
				closest = p
			end
		end
	end
	return closest
end

local function CreateLine()
	if Aimlock.Line then Aimlock.Line:Remove() end
	local line = Drawing.new("Line")
	line.Visible = false
	line.Color = Color3.fromRGB(0, 255, 255)
	line.Thickness = 2
	Aimlock.Line = line
end

local function AimlockLoop()
	if not Aimlock.Enabled then
		if Aimlock.Line then Aimlock.Line.Visible = false end
		BillboardGui.Enabled = false
		return
	end
	local target = Aimlock.Target
	if not target or not target.Character or not target.Character:FindFirstChild(Aimlock.AimPart) then
		Aimlock.Target = GetClosestPlayer()
		return
	end
	local part = target.Character[Aimlock.AimPart]
	local humanoid = target.Character:FindFirstChildOfClass("Humanoid")
	if not humanoid or humanoid.Health <= 0 then
		Aimlock.Target = GetClosestPlayer()
		return
	end
	BillboardGui.Enabled = true
	BillboardGui.Parent = part
	HealthText.Text = string.format("❤️ %d / %d", math.floor(humanoid.Health), math.floor(humanoid.MaxHealth))
	local cam = Camera
	local cf = cam.CFrame
	local newCF = CFrame.new(cf.Position, part.Position)
	cam.CFrame = cf:Lerp(newCF, Aimlock.Smoothing)
	local pos, visible = cam:WorldToViewportPoint(part.Position)
	if Aimlock.Line then
		if visible then
			Aimlock.Line.Visible = true
			Aimlock.Line.From = Vector2.new(cam.ViewportSize.X / 2, cam.ViewportSize.Y / 2)
			Aimlock.Line.To = Vector2.new(pos.X, pos.Y)
		else
			Aimlock.Line.Visible = false
		end
	end
	StatusLabel.Text = "Aimlock: ATIVADO"
	StatusLabel.TextColor3 = Color3.fromRGB(50, 255, 50)
	TargetLabel.Text = "Alvo: " .. target.Name
end

local function ToggleAimlock()
	Aimlock.Enabled = not Aimlock.Enabled
	if Aimlock.Enabled then
		Aimlock.Target = GetClosestPlayer()
		CreateLine()
		Aimlock.Connection = RunService.RenderStepped:Connect(AimlockLoop)
	else
		if Aimlock.Connection then Aimlock.Connection:Disconnect() end
		if Aimlock.Line then Aimlock.Line:Remove() end
		Aimlock.Target = nil
		BillboardGui.Enabled = false
		StatusLabel.Text = "Aimlock: DESATIVADO"
		StatusLabel.TextColor3 = Color3.fromRGB(255, 50, 50)
		TargetLabel.Text = "Alvo: Nenhum"
	end
end

UIS.InputBegan:Connect(function(input, gp)
	if gp then return end
	if input.KeyCode == Enum.KeyCode.F then
		ToggleAimlock()
	elseif input.KeyCode == Enum.KeyCode.G then
		NoclipEnabled = not NoclipEnabled
		StatusLabel.Text = "Noclip: " .. (NoclipEnabled and "ATIVADO" or "DESATIVADO")
	elseif input.KeyCode == Enum.KeyCode.E then
		AutoInteractEnabled = not AutoInteractEnabled
		StatusLabel.Text = "Auto Interact: " .. (AutoInteractEnabled and "ATIVADO" or "DESATIVADO")
	end
end)

print("✅ MXP Studios carregado — F (Aimlock), G (Noclip), E (Auto Interact)")
