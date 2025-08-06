local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Criar ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "FloatingButtonGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

-- Botão flutuante com animações
local button = Instance.new("Frame")
button.Name = "FloatingButton"
button.Size = UDim2.new(0, 60, 0, 60)
button.Position = UDim2.new(0, 20, 0.5, -30) -- Posição: lado esquerdo, centralizado
button.AnchorPoint = Vector2.new(0, 0.5)
button.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
button.BorderSizePixel = 0
button.ZIndex = 100
button.Parent = screenGui

local buttonCorner = Instance.new("UICorner")
buttonCorner.CornerRadius = UDim.new(0, 12)
buttonCorner.Parent = button

local buttonStroke = Instance.new("UIStroke")
buttonStroke.Color = Color3.fromRGB(255, 255, 255)
buttonStroke.Thickness = 2
buttonStroke.Parent = button

local image = Instance.new("ImageLabel")
image.Size = UDim2.new(0, 36, 0, 36)
image.Position = UDim2.new(0.5, 0, 0.5, 0)
image.AnchorPoint = Vector2.new(0.5, 0.5)
image.BackgroundTransparency = 1
image.Image = "rbxassetid://6031075938"
image.ImageColor3 = Color3.new(1, 1, 1)
image.ZIndex = 101
image.Parent = button

-- Botão transparente para clique/hover
local clickDetector = Instance.new("TextButton")
clickDetector.Size = UDim2.new(1, 0, 1, 0)
clickDetector.BackgroundTransparency = 1
clickDetector.Text = ""
clickDetector.ZIndex = 102
clickDetector.Parent = button

-- Som do clique do botão flutuante
local clickSound = Instance.new("Sound")
clickSound.SoundId = "rbxassetid://15675059323"
clickSound.Volume = 0.5
clickSound.Parent = clickDetector

-- Animações do botão flutuante
local originalButtonSize = button.Size
local originalButtonColor = button.BackgroundColor3
local hoverButtonSize = UDim2.new(0, 65, 0, 65)
local hoverButtonColor = Color3.fromRGB(60, 60, 60)
local clickButtonSize = UDim2.new(0, 55, 0, 55)

-- Hover no botão flutuante
clickDetector.MouseEnter:Connect(function()
	local colorTween = TweenService:Create(button, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {
		BackgroundColor3 = hoverButtonColor
	})
	local sizeTween = TweenService:Create(button, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {
		Size = hoverButtonSize
	})
	local strokeTween = TweenService:Create(buttonStroke, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {
		Color = Color3.fromRGB(90, 150, 220)
	})
	colorTween:Play()
	sizeTween:Play()
	strokeTween:Play()
end)

clickDetector.MouseLeave:Connect(function()
	local colorTween = TweenService:Create(button, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {
		BackgroundColor3 = originalButtonColor
	})
	local sizeTween = TweenService:Create(button, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {
		Size = originalButtonSize
	})
	local strokeTween = TweenService:Create(buttonStroke, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {
		Color = Color3.fromRGB(70, 130, 200)
	})
	colorTween:Play()
	sizeTween:Play()
	strokeTween:Play()
end)

local GuiManager = {}
GuiManager.spawnedFrame = nil
GuiManager.isMinimized = false
GuiManager.activeConnections = {}
GuiManager.activeTweens = {}

function GuiManager:CleanUp()
	for _, tween in pairs(self.activeTweens) do
		if tween then
			tween:Cancel()
		end
	end
	self.activeTweens = {}

	for _, connection in pairs(self.activeConnections) do
		if connection then
			connection:Disconnect()
		end
	end
	self.activeConnections = {}
end

function GuiManager:AddConnection(connection)
	table.insert(self.activeConnections, connection)
end

function GuiManager:AddTween(tween)
	table.insert(self.activeTweens, tween)
	return tween
end

-- Função melhorada para criar botões com animações uniformes
function GuiManager:CreateButton(parent, size, position, text, color, zIndex, textColor)
	local btn = Instance.new("TextButton")
	btn.Size = size or UDim2.new(0, 120, 0, 45)
	btn.Position = position or UDim2.new(0, 0, 0, 0)
	btn.BackgroundColor3 = color or Color3.fromRGB(60, 60, 60)
	btn.Text = text or "Botão"
	btn.TextColor3 = textColor or Color3.new(1, 1, 1)
	btn.TextScaled = true
	btn.Font = Enum.Font.GothamBold
	btn.BorderSizePixel = 0
	btn.ZIndex = zIndex or 8
	btn.AutoButtonColor = false
	btn.Parent = parent

	local btnCorner = Instance.new("UICorner")
	btnCorner.CornerRadius = UDim.new(0, 8)
	btnCorner.Parent = btn

	-- Stroke para o botão
	local stroke = Instance.new("UIStroke")
	stroke.Color = Color3.fromRGB(
		math.clamp(color.R * 255 + 20, 0, 255),
		math.clamp(color.G * 255 + 20, 0, 255),
		math.clamp(color.B * 255 + 20, 0, 255)
	)
	stroke.Thickness = 2
	stroke.Transparency = 0.2
	stroke.Parent = btn

	-- Cores para animações
	local originalColor = btn.BackgroundColor3
	local originalSize = btn.Size
	local hoverColor = Color3.new(
		math.clamp(originalColor.R + 0.1, 0, 1),
		math.clamp(originalColor.G + 0.1, 0, 1),
		math.clamp(originalColor.B + 0.1, 0, 1)
	)
	local hoverSize = UDim2.new(originalSize.X.Scale, originalSize.X.Offset + 5, originalSize.Y.Scale, originalSize.Y.Offset + 2)
	local clickSize = UDim2.new(originalSize.X.Scale, originalSize.X.Offset - 3, originalSize.Y.Scale, originalSize.Y.Offset - 1)

	-- Animação de hover (entrada)
	local enterConnection = btn.MouseEnter:Connect(function()
		if self.spawnedFrame and self.spawnedFrame.Parent then
			local colorTween = self:AddTween(TweenService:Create(btn, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {
				BackgroundColor3 = hoverColor
			}))
			local sizeTween = self:AddTween(TweenService:Create(btn, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {
				Size = hoverSize
			}))
			colorTween:Play()
			sizeTween:Play()
		end
	end)

	-- Animação de hover (saída)
	local leaveConnection = btn.MouseLeave:Connect(function()
		if self.spawnedFrame and self.spawnedFrame.Parent then
			local colorTween = self:AddTween(TweenService:Create(btn, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {
				BackgroundColor3 = originalColor
			}))
			local sizeTween = self:AddTween(TweenService:Create(btn, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {
				Size = originalSize
			}))
			colorTween:Play()
			sizeTween:Play()
		end
	end)

	self:AddConnection(enterConnection)
	self:AddConnection(leaveConnection)

	-- Som do clique do botão
	local buttonSound = Instance.new("Sound")
	buttonSound.SoundId = "rbxassetid://15675059323"
	buttonSound.Volume = 0.5
	buttonSound.Parent = btn

	-- Animação de clique
	local clickConnection = btn.MouseButton1Click:Connect(function()
		buttonSound:Play()

		-- Animação de "press down"
		local pressDownTween = self:AddTween(TweenService:Create(btn, TweenInfo.new(0.1, Enum.EasingStyle.Quad), {
			Size = clickSize,
			BackgroundColor3 = Color3.new(
				math.clamp(originalColor.R - 0.1, 0, 1),
				math.clamp(originalColor.G - 0.1, 0, 1),
				math.clamp(originalColor.B - 0.1, 0, 1)
			)
		}))
		pressDownTween:Play()

		-- Animação de "release"
		pressDownTween.Completed:Connect(function()
			local releaseColorTween = self:AddTween(TweenService:Create(btn, TweenInfo.new(0.15, Enum.EasingStyle.Quad), {
				BackgroundColor3 = hoverColor
			}))
			local releaseSizeTween = self:AddTween(TweenService:Create(btn, TweenInfo.new(0.15, Enum.EasingStyle.Quad), {
				Size = hoverSize
			}))
			releaseColorTween:Play()
			releaseSizeTween:Play()
		end)
	end)

	self:AddConnection(clickConnection)

	return btn
end

-- Função para tornar o GUI arrastável (sistema de drag)
local function makeDraggable(frame, dragHandle)
	local dragging = false
	local dragStart = nil
	local startPos = nil

	local function update(input)
		local delta = input.Position - dragStart
		local newPosition = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
		frame.Position = newPosition
	end

	dragHandle.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			-- (Opcional) bloqueia drag se estiver minimizado
			if GuiManager.isMinimized then
				return
			end

			dragging = true
			dragStart = input.Position
			startPos = frame.Position
		end
	end)

	dragHandle.InputChanged:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
			if dragging then
				update(input)
			end
		end
	end)

	dragHandle.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			dragging = false
		end
	end)
end


-- Função para criar frame principal do painel
local function createMainFrame()
	local frame = Instance.new("Frame")
	frame.Size = UDim2.new(0, 450, 0, 380)
	frame.Position = UDim2.new(0.5, 0, 0.5, 0)
	frame.AnchorPoint = Vector2.new(0.5, 0.5)
	frame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
	frame.BorderSizePixel = 0
	frame.ZIndex = 10
	frame.ClipsDescendants = true
	frame.BackgroundTransparency = 0
	frame.Parent = screenGui

	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 14)
	corner.Parent = frame

	local stroke = Instance.new("UIStroke")
	stroke.Color = Color3.fromRGB(200, 200, 200)
	stroke.Thickness = 2
	stroke.Parent = frame

	-- Armazenar o tamanho original para o drag
	frame:SetAttribute("OriginalSize", frame.Size)

	return frame
end

-- Criar barra de título
local function createTitleBar(parent)
	local titleBar = Instance.new("Frame")
	titleBar.Size = UDim2.new(1, 0, 0, 38)
	titleBar.Position = UDim2.new(0, 0, 0, 0)
	titleBar.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
	titleBar.BorderSizePixel = 0
	titleBar.ZIndex = 12
	titleBar.Parent = parent

	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 14)
	corner.Parent = titleBar

	local titleLabel = Instance.new("TextLabel")
	titleLabel.Size = UDim2.new(1, -90, 1, 0)
	titleLabel.Position = UDim2.new(0, 12, 0, 0)
	titleLabel.BackgroundTransparency = 1
	titleLabel.Text = "🎮 Hub secreto"
	titleLabel.TextColor3 = Color3.new(1, 1, 1)
	titleLabel.TextScaled = true
	titleLabel.Font = Enum.Font.GothamBold
	titleLabel.TextXAlignment = Enum.TextXAlignment.Left
	titleLabel.ZIndex = 13
	titleLabel.Parent = titleBar

	-- Tornar a barra de título arrastável
	makeDraggable(parent, titleBar)

	return titleBar
end

-- Função para tocar som de abertura
local function playOpenGuiSound()
	local sound = Instance.new("Sound")
	sound.SoundId = "rbxassetid://8968249849"
	sound.Volume = 0.6
	sound.Parent = screenGui
	sound:Play()
	sound.Ended:Connect(function()
		sound:Destroy()
	end)
end

-- Clique no botão flutuante para abrir/fechar painel
clickDetector.MouseButton1Click:Connect(function()
	-- Animação de clique no botão flutuante
	local clickTween = TweenService:Create(button, TweenInfo.new(0.1, Enum.EasingStyle.Quad), {
		Size = clickButtonSize,
		BackgroundColor3 = Color3.fromRGB(20, 20, 20)
	})
	clickTween:Play()

	clickTween.Completed:Connect(function()
		local releaseTween = TweenService:Create(button, TweenInfo.new(0.15, Enum.EasingStyle.Quad), {
			Size = hoverButtonSize,
			BackgroundColor3 = hoverButtonColor
		})
		releaseTween:Play()
	end)

	clickSound:Play()

	if GuiManager.spawnedFrame then
		GuiManager:CleanUp()

		local tweenOut = TweenService:Create(GuiManager.spawnedFrame, TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.In), {
			Size = UDim2.new(0, 0, 0, 0),
			BackgroundTransparency = 1
		})
		tweenOut:Play()
		tweenOut.Completed:Connect(function()
			if GuiManager.spawnedFrame then
				GuiManager.spawnedFrame:Destroy()
				GuiManager.spawnedFrame = nil
				GuiManager.isMinimized = false
			end
		end)
		return
	end

	GuiManager.spawnedFrame = createMainFrame()
	playOpenGuiSound()

	local titleBar = createTitleBar(GuiManager.spawnedFrame)

	-- Função melhorada para criar botões da janela com animações uniformes
	local function createWindowButton(parent, text, bgColor, position, textColor)
		local btn = Instance.new("TextButton")
		btn.Size = UDim2.new(0, 28, 0, 28)
		btn.Position = position
		btn.AnchorPoint = Vector2.new(0.5, 0.5)
		btn.BackgroundColor3 = bgColor
		btn.Text = text
		btn.TextColor3 = textColor or Color3.new(1, 1, 1)
		btn.TextScaled = true
		btn.Font = Enum.Font.GothamBold
		btn.BorderSizePixel = 0
		btn.ZIndex = 14
		btn.AutoButtonColor = false
		btn.Parent = parent

		local corner = Instance.new("UICorner")
		corner.CornerRadius = UDim.new(0, 5)
		corner.Parent = btn

		-- Stroke para o botão da janela
		local stroke = Instance.new("UIStroke")
		stroke.Color = Color3.fromRGB(
			math.clamp(bgColor.R * 255 + 30, 0, 255),
			math.clamp(bgColor.G * 255 + 30, 0, 255),
			math.clamp(bgColor.B * 255 + 30, 0, 255)
		)
		stroke.Thickness = 1.5
		stroke.Transparency = 0.3
		stroke.Parent = btn

		-- Cores e tamanhos para animações
		local originalColor = btn.BackgroundColor3
		local originalSize = btn.Size
		local hoverColor = Color3.new(
			math.clamp(originalColor.R + 0.15, 0, 1),
			math.clamp(originalColor.G + 0.15, 0, 1),
			math.clamp(originalColor.B + 0.15, 0, 1)
		)
		local hoverSize = UDim2.new(0, 32, 0, 32)
		local clickSize = UDim2.new(0, 24, 0, 24)

		-- Animações de hover
		local enterConn = btn.MouseEnter:Connect(function()
			local colorTween = GuiManager:AddTween(TweenService:Create(btn, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {
				BackgroundColor3 = hoverColor
			}))
			local sizeTween = GuiManager:AddTween(TweenService:Create(btn, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {
				Size = hoverSize
			}))
			colorTween:Play()
			sizeTween:Play()
		end)

		local leaveConn = btn.MouseLeave:Connect(function()
			local colorTween = GuiManager:AddTween(TweenService:Create(btn, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {
				BackgroundColor3 = originalColor
			}))
			local sizeTween = GuiManager:AddTween(TweenService:Create(btn, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {
				Size = originalSize
			}))
			colorTween:Play()
			sizeTween:Play()
		end)

		GuiManager:AddConnection(enterConn)
		GuiManager:AddConnection(leaveConn)

		-- Som e animação de clique
		local sound = Instance.new("Sound")
		sound.SoundId = "rbxassetid://15675059323"
		sound.Volume = 0.5
		sound.Parent = btn

		local clickConn = btn.MouseButton1Click:Connect(function()
			sound:Play()

			-- Animação de clique
			local pressDownTween = GuiManager:AddTween(TweenService:Create(btn, TweenInfo.new(0.1, Enum.EasingStyle.Quad), {
				Size = clickSize,
				BackgroundColor3 = Color3.new(
					math.clamp(originalColor.R - 0.1, 0, 1),
					math.clamp(originalColor.G - 0.1, 0, 1),
					math.clamp(originalColor.B - 0.1, 0, 1)
				)
			}))
			pressDownTween:Play()

			pressDownTween.Completed:Connect(function()
				local releaseTween = GuiManager:AddTween(TweenService:Create(btn, TweenInfo.new(0.15, Enum.EasingStyle.Quad), {
					Size = hoverSize,
					BackgroundColor3 = hoverColor
				}))
				releaseTween:Play()
			end)
		end)

		GuiManager:AddConnection(clickConn)

		return btn
	end

	-- Botões minimizar e fechar com animações
	local minimizeButton = createWindowButton(titleBar, "-", Color3.fromRGB(255, 193, 7), UDim2.new(1, -64, 0.5, 0), Color3.fromRGB(0, 0, 0))
	local closeButton = createWindowButton(titleBar, "X", Color3.fromRGB(220, 53, 69), UDim2.new(1, -32, 0.5, 0))

	-- Criar containers das abas
	local function createContainers(parent)
		local tabContainer = Instance.new("Frame")
		tabContainer.Size = UDim2.new(1, -24, 0, 60)
		tabContainer.Position = UDim2.new(0, 12, 0, 46)
		tabContainer.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
		tabContainer.BorderSizePixel = 0
		tabContainer.ZIndex = 15
		tabContainer.Visible = true
		tabContainer.Parent = parent

		local tabCorner = Instance.new("UICorner")
		tabCorner.CornerRadius = UDim.new(0, 10)
		tabCorner.Parent = tabContainer

		local tabStroke = Instance.new("UIStroke")
		tabStroke.Color = Color3.fromRGB(100, 100, 100)
		tabStroke.Thickness = 1
		tabStroke.Transparency = 0.3
		tabStroke.Parent = tabContainer

		local tabLayout = Instance.new("UIListLayout")
		tabLayout.FillDirection = Enum.FillDirection.Horizontal
		tabLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
		tabLayout.VerticalAlignment = Enum.VerticalAlignment.Center
		tabLayout.Padding = UDim.new(0, 4)
		tabLayout.Parent = tabContainer

		local tabPadding = Instance.new("UIPadding")
		tabPadding.PaddingTop = UDim.new(0, 8)
		tabPadding.PaddingBottom = UDim.new(0, 8)
		tabPadding.PaddingLeft = UDim.new(0, 8)
		tabPadding.PaddingRight = UDim.new(0, 8)
		tabPadding.Parent = tabContainer

		local contentContainer = Instance.new("Frame")
		contentContainer.Size = UDim2.new(1, -24, 1, -118)
		contentContainer.Position = UDim2.new(0, 12, 0, 114)
		contentContainer.BackgroundTransparency = 1
		contentContainer.ZIndex = 11
		contentContainer.ClipsDescendants = true
		contentContainer.Visible = true
		contentContainer.Parent = parent

		return tabContainer, contentContainer
	end

	local tabContainer, contentContainer = createContainers(GuiManager.spawnedFrame)

	-- Sistema de abas
	local tabs = {"Scripts", "Hubs", "Stats", "Configuração"}
	local tabButtons = {}
	local tabContents = {}
	local activeTab = 1

	-- Função para criar botão de aba com animações melhoradas
	local function createTabButton(parent, text, index)
		local tabBtn = Instance.new("TextButton")
		tabBtn.Size = UDim2.new(0, 100, 0, 40)
		tabBtn.BackgroundColor3 = (index == 1) and Color3.fromRGB(70, 130, 200) or Color3.fromRGB(55, 55, 55)
		tabBtn.Text = text
		tabBtn.TextColor3 = Color3.new(1, 1, 1)
		tabBtn.TextScaled = true
		tabBtn.Font = Enum.Font.GothamBold
		tabBtn.BorderSizePixel = 0
		tabBtn.ZIndex = 16
		tabBtn.AutoButtonColor = false
		tabBtn.Visible = true
		tabBtn.Parent = parent

		local textPadding = Instance.new("UIPadding")
		textPadding.PaddingLeft = UDim.new(0, 6)
		textPadding.PaddingRight = UDim.new(0, 6)
		textPadding.PaddingTop = UDim.new(0, 2)
		textPadding.PaddingBottom = UDim.new(0, 2)
		textPadding.Parent = tabBtn

		local corner = Instance.new("UICorner")
		corner.CornerRadius = UDim.new(0, 8)
		corner.Parent = tabBtn

		local stroke = Instance.new("UIStroke")
		stroke.Color = (index == 1) and Color3.fromRGB(90, 150, 220) or Color3.fromRGB(80, 80, 80)
		stroke.Thickness = 2
		stroke.Transparency = 0.2
		stroke.Parent = tabBtn

		-- Som de clique
		local sound = Instance.new("Sound")
		sound.SoundId = "rbxassetid://15675059323"
		sound.Volume = 0.3
		sound.Parent = tabBtn

		-- Cores e tamanhos para animações das abas
		local originalSize = tabBtn.Size
		local hoverSize = UDim2.new(0, 105, 0, 42)
		local clickSize = UDim2.new(0, 98, 0, 38)

		-- Efeitos hover melhorados para abas
		local enterConn = tabBtn.MouseEnter:Connect(function()
			if GuiManager.spawnedFrame and GuiManager.spawnedFrame.Parent then
				local isActive = (activeTab == index)
				local hoverColor = isActive and Color3.fromRGB(90, 150, 220) or Color3.fromRGB(75, 75, 75)

				local colorTween = GuiManager:AddTween(TweenService:Create(tabBtn, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {
					BackgroundColor3 = hoverColor
				}))
				local sizeTween = GuiManager:AddTween(TweenService:Create(tabBtn, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {
					Size = hoverSize
				}))
				colorTween:Play()
				sizeTween:Play()
			end
		end)

		local leaveConn = tabBtn.MouseLeave:Connect(function()
			if GuiManager.spawnedFrame and GuiManager.spawnedFrame.Parent then
				local targetColor = (activeTab == index) and Color3.fromRGB(70, 130, 200) or Color3.fromRGB(55, 55, 55)
				local colorTween = GuiManager:AddTween(TweenService:Create(tabBtn, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {
					BackgroundColor3 = targetColor
				}))
				local sizeTween = GuiManager:AddTween(TweenService:Create(tabBtn, TweenInfo.new(0.2, Enum.EasingStyle.Quad), {
					Size = originalSize
				}))
				colorTween:Play()
				sizeTween:Play()
			end
		end)

		GuiManager:AddConnection(enterConn)
		GuiManager:AddConnection(leaveConn)

		tabBtn.MouseButton1Click:Connect(function()
			sound:Play()

			-- Animação de clique na aba
			local pressDownTween = GuiManager:AddTween(TweenService:Create(tabBtn, TweenInfo.new(0.1, Enum.EasingStyle.Quad), {
				Size = clickSize,
				BackgroundColor3 = Color3.fromRGB(50, 110, 180)
			}))
			pressDownTween:Play()

			pressDownTween.Completed:Connect(function()
				local releaseTween = GuiManager:AddTween(TweenService:Create(tabBtn, TweenInfo.new(0.15, Enum.EasingStyle.Quad), {
					Size = originalSize,
					BackgroundColor3 = Color3.fromRGB(70, 130, 200)
				}))
				releaseTween:Play()
			end)

			activeTab = index

			-- Atualizar aparência das abas
			for i, btn in pairs(tabButtons) do
				local isActive = (i == activeTab)
				local targetColor = isActive and Color3.fromRGB(70, 130, 200) or Color3.fromRGB(55, 55, 55)
				local targetStroke = isActive and Color3.fromRGB(90, 150, 220) or Color3.fromRGB(80, 80, 80)

				GuiManager:AddTween(TweenService:Create(btn, TweenInfo.new(0.3), {
					BackgroundColor3 = targetColor
				})):Play()

				if btn:FindFirstChild("UIStroke") then
					GuiManager:AddTween(TweenService:Create(btn.UIStroke, TweenInfo.new(0.3), {
						Color = targetStroke
					})):Play()
				end
			end

			-- Mostrar conteúdo da aba ativa
			for i, content in pairs(tabContents) do
				content.Visible = (i == activeTab)
			end
		end)

		return tabBtn
	end

	-- Criar botões das abas
	for i, tabName in pairs(tabs) do
		tabButtons[i] = createTabButton(tabContainer, tabName, i)
	end

	-- Função para criar conteúdo de aba
	local function createTabContent(parent, index)
		local content = Instance.new("ScrollingFrame")
		content.Size = UDim2.new(1, 0, 1, 0)
		content.Position = UDim2.new(0, 0, 0, 0)
		content.BackgroundTransparency = 1
		content.BorderSizePixel = 0
		content.ZIndex = 11
		content.ScrollBarThickness = 6
		content.ScrollBarImageColor3 = Color3.fromRGB(100, 100, 100)
		content.CanvasSize = UDim2.new(0, 0, 0, 0)
		content.Visible = (index == 1)
		content.Parent = parent

		local layout = Instance.new("UIListLayout")
		layout.SortOrder = Enum.SortOrder.LayoutOrder
		layout.Padding = UDim.new(0, 8)
		layout.Parent = content

		local padding = Instance.new("UIPadding")
		padding.PaddingTop = UDim.new(0, 8)
		padding.PaddingBottom = UDim.new(0, 8)
		padding.PaddingLeft = UDim.new(0, 8)
		padding.PaddingRight = UDim.new(0, 8)
		padding.Parent = content

		-- Auto-redimensionar canvas
		layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
			content.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 32)
		end)

		return content
	end

	-- Criar conteúdos das abas
	for i = 1, #tabs do
		tabContents[i] = createTabContent(contentContainer, i)
	end

	-- Adicionar conteúdo de exemplo com botões animados
	local function addExampleContent()
		local contents = {
			{text = "📜 Scripts Disponíveis", emoji = "📜"},
			{text = "🌐 Script Hubs", emoji = "🌐"},
			{text = "📊 Estatísticas do Jogador", emoji = "📊"},
			{text = "⚙️ Configurações", emoji = "⚙️"}
		}

		for i, contentData in pairs(contents) do
			local label = Instance.new("TextLabel")
			label.Size = UDim2.new(1, -16, 0, 40)
			label.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
			label.Text = contentData.text
			label.TextColor3 = Color3.new(1, 1, 1)
			label.TextScaled = true
			label.Font = Enum.Font.GothamBold
			label.ZIndex = 12
			label.Parent = tabContents[i]

			local corner = Instance.new("UICorner")
			corner.CornerRadius = UDim.new(0, 8)
			corner.Parent = label

			-- Adicionar botões de exemplo usando a função CreateButton com animações
			if i == 1 then -- Scripts
				local executeBtn = GuiManager:CreateButton(tabContents[i], UDim2.new(0, 180, 0, 35), UDim2.new(0, 0, 0, 0), 
					"▶️ Executar Script", Color3.fromRGB(40, 167, 69))
				local loadBtn = GuiManager:CreateButton(tabContents[i], UDim2.new(0, 180, 0, 35), UDim2.new(0, 0, 0, 0), 
					"📁 Carregar Script", Color3.fromRGB(23, 162, 184))
			elseif i == 2 then -- Hubs
				local infiniteYieldBtn = GuiManager:CreateButton(tabContents[i], UDim2.new(0, 180, 0, 35), UDim2.new(0, 0, 0, 0), 
					"🚀 Infinite Yield", Color3.fromRGB(108, 117, 125))
				local darkHubBtn = GuiManager:CreateButton(tabContents[i], UDim2.new(0, 180, 0, 35), UDim2.new(0, 0, 0, 0), 
					"🌙 Dark Hub", Color3.fromRGB(52, 58, 64))
			elseif i == 3 then -- Stats
				local speedBtn = GuiManager:CreateButton(tabContents[i], UDim2.new(0, 180, 0, 35), UDim2.new(0, 0, 0, 0), 
					"⚡ Aumentar Speed", Color3.fromRGB(255, 193, 7), nil, Color3.fromRGB(0, 0, 0))
				local jumpBtn = GuiManager:CreateButton(tabContents[i], UDim2.new(0, 180, 0, 35), UDim2.new(0, 0, 0, 0), 
					"🦘 Aumentar Jump", Color3.fromRGB(255, 87, 34))
			elseif i == 4 then -- Configurações
				local themeBtn = GuiManager:CreateButton(tabContents[i], UDim2.new(0, 180, 0, 35), UDim2.new(0, 0, 0, 0), 
					"🎨 Mudar Tema", Color3.fromRGB(156, 39, 176))
				local resetBtn = GuiManager:CreateButton(tabContents[i], UDim2.new(0, 180, 0, 35), UDim2.new(0, 0, 0, 0), 
					"🔄 Reset Config", Color3.fromRGB(244, 67, 54))
			end
		end
	end

	addExampleContent()

	-- Funcionalidade dos botões da janela
	local minimizeConn = minimizeButton.MouseButton1Click:Connect(function()
		GuiManager.isMinimized = not GuiManager.isMinimized
		if GuiManager.isMinimized then
			contentContainer.Visible = false
			tabContainer.Visible = false
			minimizeButton.Text = "+"

			local tween = GuiManager:AddTween(TweenService:Create(GuiManager.spawnedFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quad), {
				Size = UDim2.new(0, 450, 0, 38)
			}))
			tween:Play()
		else
			contentContainer.Visible = true
			tabContainer.Visible = true
			minimizeButton.Text = "−"

			local tween = GuiManager:AddTween(TweenService:Create(GuiManager.spawnedFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quad), {
				Size = UDim2.new(0, 450, 0, 380)
			}))
			tween:Play()
		end
	end)

	local closeConn = closeButton.MouseButton1Click:Connect(function()
		GuiManager:CleanUp()

		local tweenOut = TweenService:Create(GuiManager.spawnedFrame, TweenInfo.new(0.25, Enum.EasingStyle.Back, Enum.EasingDirection.In), {
			Size = UDim2.new(0, 0, 0, 0),
			BackgroundTransparency = 1
		})
		tweenOut:Play()
		tweenOut.Completed:Connect(function()
			if GuiManager.spawnedFrame then
				GuiManager.spawnedFrame:Destroy()
				GuiManager.spawnedFrame = nil
				GuiManager.isMinimized = false
			end
		end)
	end)

	GuiManager:AddConnection(minimizeConn)
	GuiManager:AddConnection(closeConn)
end)
