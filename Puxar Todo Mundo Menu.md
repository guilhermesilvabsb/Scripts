--========================================================
--  BRING UI COMPLETA v3.1 (com proteção anti-spam)
--  PC + MOBILE | Selecionar Todos | Sem Minimizar | Logs
--========================================================

-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local localPlayer = Players.LocalPlayer
local playerGui = localPlayer:WaitForChild("PlayerGui")

--===============================
-- Sistema anti-spam (não permite múltiplas execuções)
--===============================
if playerGui:FindFirstChild("BringUI") then
    warn("[BringUI] Já está carregado! Cancelando execução.")
    return
end

-- Configs
local bringEnabled = true
local distance = 5                -- 1..20
local clearCooldown = 1           -- limpeza de tools por jogador
local lerpAlpha = 0.45            -- suavidade
local freezeEnabled = false       -- congela ao desativar

-- Estados
local lastCleared = {}
local selectedPlayers = {}

--=====================
-- Funções de posição
--=====================

local function computeBringPosition()
	local char = localPlayer.Character
	if not char then return nil end
	local hrp = char:FindFirstChild("HumanoidRootPart")
	if not hrp then return nil end
	return hrp.Position - hrp.CFrame.LookVector * distance
end

local bringPosition = computeBringPosition()
local function updateBringPosition()
	bringPosition = computeBringPosition()
end

--=======================
-- UI BASE
--=======================

local ui = Instance.new("ScreenGui")
ui.Name = "BringUI"
ui.ResetOnSpawn = false
ui.Parent = playerGui

---------------------------------------
-- Logs discretos (toast)
---------------------------------------

local logHolder = Instance.new("Frame", ui)
logHolder.Size = UDim2.new(0, 300, 0, 160)
logHolder.Position = UDim2.new(0, 12, 1, -170)
logHolder.BackgroundTransparency = 1
logHolder.ClipsDescendants = true

local function newLog(text)
	local msg = Instance.new("TextLabel", logHolder)
	msg.Size = UDim2.new(1, 0, 0, 24)
	msg.Position = UDim2.new(0, 0, 1, 0)
	msg.BackgroundColor3 = Color3.fromRGB(18,18,18)
	msg.BackgroundTransparency = 0.1
	msg.TextColor3 = Color3.fromRGB(220,220,220)
	msg.Font = Enum.Font.Gotham
	msg.TextSize = 14
	msg.TextXAlignment = Enum.TextXAlignment.Left
	msg.Text = "  "..text
	msg.BorderSizePixel = 0

	local corner = Instance.new("UICorner", msg)
	corner.CornerRadius = UDim.new(0,6)

	msg:TweenPosition(UDim2.new(0,0,1,-26), "Out", "Quad", 0.25, true)

	task.delay(2.2, function()
		if msg and msg.Parent then
			msg:TweenPosition(UDim2.new(0,0,1,0), "In", "Quad", 0.2, true)
			task.wait(0.2)
			if msg then msg:Destroy() end
		end
	end)
end

---------------------------------------
-- Painel principal
---------------------------------------

local frame = Instance.new("Frame", ui)
frame.Size = UDim2.new(0, 360, 0, 360)
frame.Position = UDim2.new(0.5, -180, 0.5, -180)
frame.BackgroundColor3 = Color3.fromRGB(20,20,20)
frame.Active = true
frame.Draggable = true

Instance.new("UICorner", frame).CornerRadius = UDim.new(0,12)

-- Fade overlay
local fadeOverlay = Instance.new("Frame", frame)
fadeOverlay.Size = UDim2.new(1,0,1,0)
fadeOverlay.BackgroundTransparency = 1

-- Título
local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, -20, 0, 40)
title.Position = UDim2.new(0,10,0,0)
title.BackgroundTransparency = 1
title.Text = "Puxar Todo Mundo"
title.Font = Enum.Font.GothamBold
title.TextSize = 40
title.TextColor3 = Color3.fromRGB(255,255,255)
title.TextXAlignment = Enum.TextXAlignment.Left

-- Toggle Bring
local toggleBtn = Instance.new("TextButton", frame)
toggleBtn.Size = UDim2.new(1,-20,0,40)
toggleBtn.Position = UDim2.new(0,10,0,50)
toggleBtn.BackgroundColor3 = Color3.fromRGB(40,40,40)
toggleBtn.TextColor3 = Color3.fromRGB(255,255,255)
toggleBtn.Font = Enum.Font.Gotham
toggleBtn.TextSize = 16
toggleBtn.Text = "Bring: ON"
toggleBtn.AutoButtonColor = false
Instance.new("UICorner", toggleBtn).CornerRadius = UDim.new(0,10)

-- Reposicionar
local repositionBtn = Instance.new("TextButton", frame)
repositionBtn.Size = UDim2.new(1,-20,0,36)
repositionBtn.Position = UDim2.new(0,10,0,100)
repositionBtn.BackgroundColor3 = Color3.fromRGB(40,40,40)
repositionBtn.TextColor3 = Color3.fromRGB(255,255,255)
repositionBtn.Font = Enum.Font.Gotham
repositionBtn.TextSize = 14
repositionBtn.Text = "Atualizar posição (R)"
repositionBtn.AutoButtonColor = false
Instance.new("UICorner", repositionBtn).CornerRadius = UDim.new(0,8)

-- Checkbox: Congelar ao desativar
local freezeToggle = Instance.new("TextButton", frame)
freezeToggle.Size = UDim2.new(1,-20,0,30)
freezeToggle.Position = UDim2.new(0,10,0,140)
freezeToggle.BackgroundColor3 = Color3.fromRGB(40,40,40)
freezeToggle.TextColor3 = Color3.fromRGB(255,255,255)
freezeToggle.Font = Enum.Font.Gotham
freezeToggle.TextSize = 14
freezeToggle.Text = "[ ] Congelar ao desativar"
freezeToggle.AutoButtonColor = false
Instance.new("UICorner", freezeToggle).CornerRadius = UDim.new(0,8)

freezeToggle.MouseButton1Click:Connect(function()
	freezeEnabled = not freezeEnabled
	freezeToggle.Text = (freezeEnabled and "[✔] " or "[ ] ") .. "Congelar ao desativar"
end)

------------------------------------------------------
-- Botão Selecionar Todos / Desmarcar Todos
------------------------------------------------------

local selectAllBtn = Instance.new("TextButton", frame)
selectAllBtn.Size = UDim2.new(1,-20,0,32)
selectAllBtn.Position = UDim2.new(0,10,0,176)
selectAllBtn.BackgroundColor3 = Color3.fromRGB(40,40,40)
selectAllBtn.TextColor3 = Color3.fromRGB(255,255,255)
selectAllBtn.Font = Enum.Font.Gotham
selectAllBtn.TextSize = 14
selectAllBtn.Text = "Selecionar Todos"
selectAllBtn.AutoButtonColor = false
Instance.new("UICorner", selectAllBtn).CornerRadius = UDim.new(0,8)

local allSelected = false

selectAllBtn.MouseButton1Click:Connect(function()
	allSelected = not allSelected
	selectAllBtn.Text = allSelected and "Desmarcar Todos" or "Selecionar Todos"

	for _, plr in ipairs(Players:GetPlayers()) do
		if plr ~= localPlayer then
			selectedPlayers[plr] = allSelected
		end
	end

	refreshPlayerList()
	newLog(allSelected and "Todos selecionados" or "Todos desmarcados")
end)

---------------------------------------
-- Distância
---------------------------------------

local distLabel = Instance.new("TextLabel", frame)
distLabel.Size = UDim2.new(1,-20,0,24)
distLabel.Position = UDim2.new(0,10,0,214)
distLabel.BackgroundTransparency = 1
distLabel.TextColor3 = Color3.fromRGB(200,200,200)
distLabel.Font = Enum.Font.Gotham
distLabel.TextSize = 14
distLabel.Text = "Distância: "..distance

-- Slider
local sliderBack = Instance.new("Frame", frame)
sliderBack.Size = UDim2.new(1,-20,0,10)
sliderBack.Position = UDim2.new(0,10,0,238)
sliderBack.BackgroundColor3 = Color3.fromRGB(50,50,50)
Instance.new("UICorner", sliderBack).CornerRadius = UDim.new(0,5)

local sliderFill = Instance.new("Frame", sliderBack)
sliderFill.Size = UDim2.new(distance/20,0,1,0)
sliderFill.BackgroundColor3 = Color3.fromRGB(0,170,255)
Instance.new("UICorner", sliderFill).CornerRadius = UDim.new(0,5)

---------------------------------------
-- Lista de jogadores
---------------------------------------

local playerList = Instance.new("ScrollingFrame", frame)
playerList.Size = UDim2.new(1,-20,0,90)
playerList.Position = UDim2.new(0,10,0,260)
playerList.CanvasSize = UDim2.new(0,0,0,0)
playerList.BackgroundColor3 = Color3.fromRGB(25,25,25)
playerList.ScrollBarThickness = 6
playerList.BorderSizePixel = 0
Instance.new("UICorner", playerList).CornerRadius = UDim.new(0,8)

local UIListLayout = Instance.new("UIListLayout", playerList)
UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
UIListLayout.Padding = UDim.new(0,4)

--------------------------------------
-- Atualizar lista de jogadores
--------------------------------------

function refreshPlayerList()
	for _,c in ipairs(playerList:GetChildren()) do
		if not c:IsA("UIListLayout") then
			c:Destroy()
		end
	end

	for _, plr in ipairs(Players:GetPlayers()) do
		if plr ~= localPlayer then
			local item = Instance.new("Frame", playerList)
			item.Size = UDim2.new(1,-4,0,26)
			item.BackgroundColor3 = Color3.fromRGB(30,30,30)
			Instance.new("UICorner", item).CornerRadius = UDim.new(0,6)

			local check = Instance.new("TextButton", item)
			check.Size = UDim2.new(0,26,1,0)
			check.Position = UDim2.new(0,2,0,0)
			check.BackgroundColor3 = Color3.fromRGB(50,50,50)
			check.Text = selectedPlayers[plr] and "✔" or ""
			check.TextColor3 = Color3.fromRGB(255,255,255)
			check.Font = Enum.Font.GothamBold
			check.TextSize = 16
			check.AutoButtonColor = false
			Instance.new("UICorner", check).CornerRadius = UDim.new(0,6)

			local name = Instance.new("TextLabel", item)
			name.Size = UDim2.new(1,-36,1,0)
			name.Position = UDim2.new(0,36,0,0)
			name.BackgroundTransparency = 1
			name.Text = plr.Name
			name.Font = Enum.Font.Gotham
			name.TextSize = 14
			name.TextColor3 = Color3.fromRGB(245,245,245)
			name.TextXAlignment = Enum.TextXAlignment.Left

			check.MouseButton1Click:Connect(function()
				selectedPlayers[plr] = not selectedPlayers[plr]
				check.Text = selectedPlayers[plr] and "✔" or ""
			end)
		end
	end

	task.wait()
	playerList.CanvasSize = UDim2.new(0,0,0,UIListLayout.AbsoluteContentSize.Y + 6)
end

refreshPlayerList()
Players.PlayerAdded:Connect(refreshPlayerList)
Players.PlayerRemoving:Connect(function(plr)
	selectedPlayers[plr] = nil
	lastCleared[plr] = nil
	refreshPlayerList()
end)

---------------------------------------
-- Slider (PC + mobile)
---------------------------------------

local dragging = false

sliderBack.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1
	or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
	end
end)

sliderBack.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1
	or input.UserInputType == Enum.UserInputType.Touch then
		dragging = false
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.Touch then
		dragging = false
	end
end)

RunService.RenderStepped:Connect(function()
	if dragging then
		local posX = UserInputService:GetMouseLocation().X
		local relX = math.clamp(posX - sliderBack.AbsolutePosition.X, 0, sliderBack.AbsoluteSize.X)
		local frac = relX / math.max(1, sliderBack.AbsoluteSize.X)

		local newDist = math.clamp(math.floor(frac * 20 + 0.5), 1, 20)
		if newDist ~= distance then
			distance = newDist
			distLabel.Text = "Distância: "..distance
			sliderFill.Size = UDim2.new(distance/20,0,1,0)
			updateBringPosition()
			newLog("Distância alterada para "..distance)
		end
	end
end)

---------------------------------------
-- Toggle Bring (ON/OFF)
---------------------------------------

toggleBtn.MouseButton1Click:Connect(function()
	bringEnabled = not bringEnabled
	toggleBtn.Text = "Bring: " .. (bringEnabled and "ON" or "OFF")
	newLog("Bring " .. (bringEnabled and "ativado" or "desativado"))

	if not bringEnabled then
		if freezeEnabled then
			local pos = computeBringPosition()
			if pos then
				for plr,sel in pairs(selectedPlayers) do
					if sel and plr.Character then
						local hrp = plr.Character:FindFirstChild("HumanoidRootPart")
						if hrp then
							hrp.CFrame = CFrame.new(pos)
							hrp.Anchored = true
						end
					end
				end
			end
		end
	else
		for plr,sel in pairs(selectedPlayers) do
			if sel and plr.Character then
				local hrp = plr.Character:FindFirstChild("HumanoidRootPart")
				if hrp then hrp.Anchored = false end
			end
		end
	end
end)

---------------------------------------
-- Reposicionar
---------------------------------------

repositionBtn.MouseButton1Click:Connect(function()
	updateBringPosition()
	newLog("Posição atualizada")
end)

---------------------------------------
-- Teclas de atalho
---------------------------------------

UserInputService.InputBegan:Connect(function(input, gp)
	if gp then return end

	if input.KeyCode == Enum.KeyCode.F then
		toggleBtn.MouseButton1Click:Fire()
	elseif input.KeyCode == Enum.KeyCode.R then
		repositionBtn.MouseButton1Click:Fire()
	end
end)

---------------------------------------
-- Loop principal do Bring
---------------------------------------

RunService.RenderStepped:Connect(function()
	if not bringEnabled then return end

	bringPosition = computeBringPosition()
	if not bringPosition then return end

	for plr,selected in pairs(selectedPlayers) do
		if selected and plr.Character then
			local char = plr.Character
			local hrp = char:FindFirstChild("HumanoidRootPart")
			if hrp then

				if tick() - (lastCleared[plr] or 0) >= clearCooldown then
					pcall(function()
						if plr:FindFirstChild("Backpack") then
							plr.Backpack:ClearAllChildren()
						end
						for _,tool in ipairs(char:GetChildren()) do
							if tool:IsA("Tool") then tool:Destroy() end
						end
					end)
					lastCleared[plr] = tick()
				end

				hrp.Anchored = false
				hrp.CFrame = hrp.CFrame:Lerp(CFrame.new(bringPosition), lerpAlpha)
			end
		end
	end
end)

---------------------------------------
-- Respawn
---------------------------------------

localPlayer.CharacterAdded:Connect(function()
	task.wait(0.2)
	updateBringPosition()
end)

---------------------------------------
-- Inicial
---------------------------------------

toggleBtn.Text = "Bring: "..(bringEnabled and "ON" or "OFF")
distLabel.Text = "Distância: "..distance
sliderFill.Size = UDim2.new(distance/20,0,1,0)

newLog("Painel carregado ✔")
print("[BringUI] Carregado v3.1")
