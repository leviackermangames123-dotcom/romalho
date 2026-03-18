local player = game.Players.LocalPlayer
local camera = workspace.CurrentCamera
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

-- CONFIG
local sprintSpeed = 24
local normalSpeed = 16
local zoomFOV = 50
local normalFOV = 70
local aimStrength = 0.08 -- quanto maior, mais forte

-- ESTADOS
local fpsAtivo = false
local sprintAtivo = false
local zoomAtivo = false
local aimAssistAtivo = false
local indicadorAtivo = false

-- FUNÇÃO HUMANOID
local function getHumanoid()
	if player.Character then
		return player.Character:FindFirstChild("Humanoid")
	end
end

-- PEGAR PLAYER MAIS PRÓXIMO DO CENTRO
local function getClosestPlayer()
	local closest = nil
	local shortest = math.huge

	for _, plr in pairs(game.Players:GetPlayers()) do
		if plr ~= player and plr.Character and plr.Character:FindFirstChild("Head") then
			local head = plr.Character.Head
			local pos, onScreen = camera:WorldToViewportPoint(head.Position)

			if onScreen then
				local dist = (Vector2.new(pos.X, pos.Y) - camera.ViewportSize/2).Magnitude
				if dist < shortest then
					shortest = dist
					closest = head
				end
			end
		end
	end

	return closest
end

-- LOOP
RunService.RenderStepped:Connect(function()
	if fpsAtivo and player.Character then
		camera.CameraType = Enum.CameraType.Custom
		camera.CameraSubject = getHumanoid()
	end

	camera.FieldOfView = zoomAtivo and zoomFOV or normalFOV

	-- AIM ASSIST SUAVE
	if aimAssistAtivo then
		local target = getClosestPlayer()
		if target then
			camera.CFrame = camera.CFrame:Lerp(
				CFrame.new(camera.CFrame.Position, target.Position),
				aimStrength
			)
		end
	end
end)

-- CONTROLES
UIS.InputBegan:Connect(function(input, gp)
	if gp then return end

	if input.KeyCode == Enum.KeyCode.LeftShift and sprintAtivo then
		local hum = getHumanoid()
		if hum then hum.WalkSpeed = sprintSpeed end
	end
end)

UIS.InputEnded:Connect(function(input)
	if input.KeyCode == Enum.KeyCode.LeftShift then
		local hum = getHumanoid()
		if hum then hum.WalkSpeed = normalSpeed end
	end
end)

-- UI
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))

-- CROSSHAIR DINÂMICA
local crosshair = Instance.new("TextLabel", gui)
crosshair.Size = UDim2.new(0, 20, 0, 20)
crosshair.Position = UDim2.new(0.5, -10, 0.5, -10)
crosshair.Text = "+"
crosshair.TextScaled = true
crosshair.TextColor3 = Color3.fromRGB(255,255,255)
crosshair.BackgroundTransparency = 1

-- INDICADOR DIRECIONAL
local indicador = Instance.new("TextLabel", gui)
indicador.Size = UDim2.new(0, 200, 0, 30)
indicador.Position = UDim2.new(0.5, -100, 0, 50)
indicador.TextColor3 = Color3.fromRGB(255,255,255)
indicador.BackgroundTransparency = 1
indicador.TextScaled = true

RunService.RenderStepped:Connect(function()
	if indicadorAtivo then
		local target = getClosestPlayer()
		if target then
			indicador.Text = "Inimigo na mira"
		else
			indicador.Text = ""
		end
	else
		indicador.Text = ""
	end
end)

-- PAINEL FLAMENGO
local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 220, 0, 260)
frame.Position = UDim2.new(0, 20, 0.3, 0)
frame.BackgroundColor3 = Color3.fromRGB(180, 0, 0)

local stroke = Instance.new("UIStroke", frame)
stroke.Color = Color3.fromRGB(0,0,0)
stroke.Thickness = 3

local layout = Instance.new("UIListLayout", frame)
layout.Padding = UDim.new(0, 8)

-- BOTÃO
local function criarBotao(nome, callback)
	local estado = false

	local btn = Instance.new("TextButton", frame)
	btn.Size = UDim2.new(1, -10, 0, 35)
	btn.BackgroundColor3 = Color3.fromRGB(0,0,0)
	btn.TextColor3 = Color3.fromRGB(255,255,255)
	btn.Text = nome .. ": OFF"

	btn.MouseButton1Click:Connect(function()
		estado = not estado
		btn.Text = nome .. ": " .. (estado and "ON" or "OFF")
		callback(estado)
	end)
end

-- BOTÕES
criarBotao("FPS", function(v) fpsAtivo = v end)
criarBotao("Sprint", function(v) sprintAtivo = v end)
criarBotao("Zoom", function(v) zoomAtivo = v end)
criarBotao("Aim Assist", function(v) aimAssistAtivo = v end)
criarBotao("Indicador", function(v) indicadorAtivo = v end)
