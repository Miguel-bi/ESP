-- ✅ ESP com Team Check, Nomes e Linhas + Botão ON/OFF
-- ⚠️ Para testes autorizados apenas! Use com responsabilidade.

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- Interface do botão
local Gui = Instance.new("ScreenGui", game:GetService("CoreGui"))
Gui.Name = "ESP_Toggle_GUI"

local Button = Instance.new("TextButton", Gui)
Button.Size = UDim2.new(0, 120, 0, 40)
Button.Position = UDim2.new(0, 10, 0, 10)
Button.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
Button.TextColor3 = Color3.new(1, 1, 1)
Button.Font = Enum.Font.SourceSansBold
Button.TextSize = 18
Button.Text = "ESP: ON"

-- Estados
local ESPEnabled = true
local ESPTable = {}

-- Criar ESP para jogador
local function CreateESP(player)
	if player == LocalPlayer or ESPTable[player] then return end

	local text = Drawing.new("Text")
	text.Size = 14
	text.Center = true
	text.Outline = true
	text.Font = 2
	text.Color = Color3.fromRGB(255, 255, 255)
	text.Visible = false

	local line = Drawing.new("Line")
	line.Thickness = 1.5
	line.Color = Color3.fromRGB(255, 0, 0)
	line.Visible = false

	ESPTable[player] = {Text = text, Line = line}
end

-- Atualização contínua
RunService.RenderStepped:Connect(function()
	if not ESPEnabled then
		for _, v in pairs(ESPTable) do
			v.Text.Visible = false
			v.Line.Visible = false
		end
		return
	end

	for _, player in pairs(Players:GetPlayers()) do
		if player ~= LocalPlayer then
			CreateESP(player)

			local esp = ESPTable[player]
			local char = player.Character
			local hrp = char and char:FindFirstChild("HumanoidRootPart")
			local hum = char and char:FindFirstChild("Humanoid")

			if hrp and hum and hum.Health > 0 then
				if player.Team ~= LocalPlayer.Team then
					local pos, visible = Camera:WorldToViewportPoint(hrp.Position)

					if visible then
						esp.Text.Position = Vector2.new(pos.X, pos.Y - 25)
						esp.Text.Text = player.Name
						esp.Text.Visible = true

						esp.Line.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
						esp.Line.To = Vector2.new(pos.X, pos.Y)
						esp.Line.Visible = true
					else
						esp.Text.Visible = false
						esp.Line.Visible = false
					end
				else
					esp.Text.Visible = false
					esp.Line.Visible = false
				end
			else
				esp.Text.Visible = false
				esp.Line.Visible = false
			end
		end
	end
end)

-- Botão ON/OFF
Button.MouseButton1Click:Connect(function()
	ESPEnabled = not ESPEnabled
	Button.Text = "ESP: " .. (ESPEnabled and "ON" or "OFF")
end)

-- Jogadores que entrarem ou saírem
Players.PlayerAdded:Connect(CreateESP)
for _, p in ipairs(Players:GetPlayers()) do
	CreateESP(p)
end

Players.PlayerRemoving:Connect(function(p)
	if ESPTable[p] then
		ESPTable[p].Text:Remove()
		ESPTable[p].Line:Remove()
		ESPTable[p] = nil
	end
end)
