local Players = game:GetService("Players")
local Debris = game:GetService("Debris")
local TweenService = game:GetService("TweenService")
локальный RunService = игра:GetService("RunService")
локальный игрок = Игроки.ЛокальныйИгрок
local playerGui = player:WaitForChild("PlayerGui")

local BLOOD_ASSET = "rbxassetid://5069307706"
local NECK_SNAP_ID = "rbxassetid://114303647099968"
local GRAB_ANIM_ID = "rbxassetid://243663079"
local THROW_ANIM_ID = "rbxassetid://218504594"
local THROW_START_TIME = 0.5
local THROW_EARLY_RATIO = 0.7

local CODE_ACTIONS = {
	dismantle = "https://raw.githubusercontent.com/GUI-Offical/TestFile-Broken/refs/heads/main/Protected_8925751398564838.lua.txt",
}

local spinnerChars = {"|","/","-","\\"}

local GUI_NAMES = {
	"Grab_Intro",
	"Grab_LoadingGui",
	"Grab_ModalGui",
	"Grab_Popup",
	"Grab_ChoiceGui",
	"Grab_KillNotify",
	"Grab_CodeGui",
	"Grab_CodeButton",
}

локальная функция safeParentGui(gui)
	local ok, core = pcall(function() return game:GetService("CoreGui") end)
	Если все в порядке и ядро, то
		local suc, _ = pcall(function() gui.Parent = core end)
		Если получилось, то вернуть графический интерфейс.
	конец
	gui.Parent = playerGui
	возврат графического интерфейса
конец

локальная функция cleanupGuis()
	for _, name in ipairs(GUI_NAMES) do
		pcall(function()
			local core = game:GetService("CoreGui")
			local g = core:FindFirstChild(name)
			если g, то g:Уничтожить() конец
		конец)
		pcall(function()
			local g = playerGui:FindFirstChild(name)
			если g, то g:Уничтожить() конец
		конец)
	конец
конец

локальная функция popupNotification(text, duration, textColor, bgColor)
	длительность = длительность или 2
	textColor = textColor или Color3.fromRGB(255,255,255)
	bgColor = bgColor or Color3.fromRGB(0,0,0)

	local gui = Instance.new("ScreenGui")
	gui.Name = "Grab_Popup"
	gui.ResetOnSpawn = false
	safeParentGui(gui)

	local frame = Instance.new("Frame")
	frame.Size = UDim2.new(0, 360, 0, 56)
	frame.Position = UDim2.new(0.5, -180, 0.16, 0)
	frame.BackgroundColor3 = bgColor
	frame.BorderSizePixel = 0
	frame.Parent = gui

	локальная метка = Instance.new("TextLabel")
	label.Size = UDim2.new(1, -12, 1, -12)
	label.Position = UDim2.new(0, 6, 0, 6)
	label.BackgroundTransparency = 1
	label.Font = Enum.Font.Code
	label.TextSize = 18
	label.TextColor3 = textColor
	label.TextWrapped = true
	label.TextXAlignment = Enum.TextXAlignment.Center
	label.TextYAlignment = Enum.TextYAlignment.Center
	label.Text = текст
	label.Parent = frame

	task.delay(duration, function()
		if gui and gui.Parent then gui:Destroy() end
	конец)
конец

локальная функция setModelCollisions(model, canCollide)
	for _, d in ipairs(model:GetDescendants()) do
		если d:IsA("BasePart") тогда
			pcall(function() d.CanCollide = конец canCollide)
		конец
	конец
конец

local animFlags = {}
локальная функция animateWords(label, fullText, wordDelay)
	if not label or not label.Parent then return end
	animFlags[label] = (animFlags[label] or 0) + 1
	локальный токен = animFlags[label]

	локальные слова = {}
	for w in fullText:gmatch("%S+") do table.insert(words, w) end
	label.Text = ""
	task.spawn(function()
		for i, w in ipairs(words) do
			if animFlags[label] ~= token then return end
			if i == 1 then label.Text = w else label.Text = label.Text .. " " .. w end
			task.wait(wordDelay or 0.12)
		конец
		if animFlags[label] == token then animFlags[label] = nil end
	конец)
конец

local bloodTemplateType, bloodEmitterTemplate, bloodModelTemplate
локальная функция preloadBlood()
	local ok, objects = pcall(function() return game:GetObjects(BLOOD_ASSET) end)
	Если не ok, не содержит объектов или #objects == 0, то вернуть end
	локальный корень = объекты[1]
	локальный foundEmitter
	for _, d in ipairs(root:GetDescendants()) do
		if d:IsA("ParticleEmitter") then foundEmitter = d break end
	конец
	если foundEmitter тогда
		bloodEmitterTemplate = foundEmitter:Clone()
		bloodTemplateType = "emitter"
	еще
		for _, p in ipairs(root:GetDescendants()) do
			if p:IsA("BasePart") then p.CanCollide = false p.Massless = true end
		конец
		root.Parent = nil
		bloodModelTemplate = root
		bloodTemplateType = "model"
	конец
конец

локальная функция makeLoadingGui()
	local gui = Instance.new("ScreenGui")
	gui.Name = "Grab_LoadingGui"
	gui.ResetOnSpawn = false

	local frame = Instance.new("Frame")
	frame.AnchorPoint = Vector2.new(1, 1)
	frame.Position = UDim2.new(1, -8, 1, -8)
	frame.Size = UDim2.new(0, 300, 0, 72)
	frame.BackgroundTransparency = 0.6
	frame.BackgroundColor3 = Color3.fromRGB(0,0,0)
	frame.BorderSizePixel = 0
	frame.Parent = gui

	local title = Instance.new("TextLabel")
	title.Name = "Title"
	title.Size = UDim2.new(1, -12, 0, 28)
	title.Position = UDim2.new(0, 6, 0, 6)
	title.BackgroundTransparency = 1
	title.Font = Enum.Font.Code
	title.TextSize = 16
	title.TextColor3 = Color3.fromRGB(255 255 255)
	title.Text = "Загрузка инструмента Grab..."
	title.Parent = frame

	локальные сведения = Instance.new("TextLabel")
	details.Name = "Details"
	details.Size = UDim2.new(1, -12, 0, 32)
	details.Position = UDim2.new(0, 6, 0, 34)
	details.BackgroundTransparency = 1
	details.Font = Enum.Font.Code
	details.TextSize = 14
	details.TextColor3 = Color3.fromRGB(200,200,200)
	details.TextWrapped = true
	details.Text = ""
	details.TextXAlignment = Enum.TextXAlignment.Left
	детали.Родитель = рамка

	возвращать графический интерфейс, заголовок, подробности
конец

локальная функция showIntroBanner(duration)
	длительность = длительность или 1,6
	pcall(function() game:GetService("CoreGui"):FindFirstChild("Grab_Intro"):Destroy() end)
	pcall(function() playerGui:FindFirstChild("Grab_Intro"):Destroy() end)

	local gui = Instance.new("ScreenGui")
	gui.Name = "Grab_Intro"
	gui.ResetOnSpawn = false

	local frame = Instance.new("Frame")
	frame.Size = UDim2.new(0, 420, 0, 44)
	frame.Position = UDim2.new(0.5, -210, 0, -80)
	frame.AnchorPoint = Vector2.new(0.5, 0)
	frame.BackgroundColor3 = Color3.fromRGB(0,0,0)
	frame.BorderSizePixel = 0
	frame.Parent = gui

	локальная метка = Instance.new("TextLabel")
	label.Name = "IntroLabel"
	label.Size = UDim2.new(1, -64, 1, 0)
	label.Position = UDim2.new(0, 8, 0, 0)
	label.BackgroundTransparency = 1
	label.Font = Enum.Font.Code
	label.TextSize = 18
	label.TextColor3 = Color3.fromRGB(255 255 255)
	label.TextXAlignment = Enum.TextXAlignment.Left
	label.Parent = frame

	local spinLabel = Instance.new("TextLabel")
	spinLabel.Name = "IntroSpin"
	spinLabel.Size = UDim2.new(0, 40, 1, 0)
	spinLabel.Position = UDim2.new(1, -44, 0, 0)
	spinLabel.BackgroundTransparency = 1
	spinLabel.Font = Enum.Font.Code
	spinLabel.TextSize = 18
	spinLabel.TextColor3 = Color3.fromRGB(255 255 255)
	spinLabel.Text = "|"
	spinLabel.Parent = frame

	safeParentGui(gui)

	TweenService:Create(frame, TweenInfo.new(0.45, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
		Position = UDim2.new(0.5, -210, 0, 12)
	}):Играть()


	локальный запуск = true
	spawn(function()
		локальный идентификатор = 1
		во время бега
			spinLabel.Text = spinnerChars[idx]
			idx = idx % #spinnerChars + 1
			task.wait(0.09)
		конец
	конец)

	task.delay(duration, function()
		running = false
		local targetPos = UDim2.new(1, -110, 1, -44)
		local targetSize = UDim2.new(0, 180, 0, 30)
		TweenService:Create(frame, TweenInfo.new(0.45, Enum.EasingStyle.Quad, Enum.EasingDirection.InOut), {
			Позиция = targetPos,
			Размер = целевойРазмер
		}):Играть()
		TweenService:Create(label, TweenInfo.new(0.45, Enum.EasingStyle.Quad, Enum.EasingDirection.InOut), {
			Размер текста = 14
		}):Играть()
		TweenService:Create(spinLabel, TweenInfo.new(0.45, Enum.EasingStyle.Quad, Enum.EasingDirection.InOut), {
			Position = UDim2.new(1, -34, 0, 0)
		}):Играть()
	конец)
конец

локальная функция runPreloadsAndShowLoading()
	pcall(function() game:GetService("CoreGui"):FindFirstChild("Grab_LoadingGui"):Destroy() end)
	pcall(function() playerGui:FindFirstChild("Grab_LoadingGui"):Destroy() end)

	local loadingGui, loadingTitle, loadingDetails = makeLoadingGui()
	safeParentGui(loadingGui)

	local spinnerRunning = true
	spawn(function()
		локальный идентификатор = 1
		пока вращающийся диск выполняет
			loadingTitle.Text = "Загрузка инструмента Grab... " .. spinnerChars[idx]
			idx = idx % #spinnerChars + 1
			task.wait(0.12)
		конец
	конец)

	локальные шаги = {
		"Загрузка частиц",
		"Загрузка графического интерфейса пользователя",
		"Анимация загрузки",
		"Звук загрузки",
		"Загрузка запаса крови",
		«Завершаем...»
	}
	for _, step in ipairs(steps) do
		loadingDetails.Text = ""
		animateWords(loadingDetails, step, 0.10)
		если шаг == "Звук загрузки", то
			local s = Instance.new("Sound")
			s.SoundId = NECK_SNAP_ID
			s.Parent = loadingGui
			Debris:AddItem(s, 1.0)
			task.wait(0.18)
		elseif step == "Загрузка ресурса крови" then
			preloadBlood()
			task.wait(0.25)
		еще
			task.wait(0.16)
		конец
	конец

	spinnerRunning = false
	loadingTitle.Text = "Готово."
	loadingDetails.Text = ""
	task.wait(0.45)
	if loadingGui and loadingGui.Parent then loadingGui:Destroy() end
конец

локальная функция showKillNotification(killerName, targetName, duration)
	длительность = длительность или 2,2
	local gui = Instance.new("ScreenGui")
	gui.Name = "Grab_KillNotify"
	gui.ResetOnSpawn = false
	safeParentGui(gui)

	local frame = Instance.new("Frame")
	frame.Name = "KillFrame"
	frame.Size = UDim2.new(0, 260, 0, 28)
	frame.Position = UDim2.new(1, 10, 0.5, -14)
	frame.AnchorPoint = Vector2.new(1, 0.5)
	frame.BackgroundColor3 = Color3.fromRGB(0,0,0)
	frame.BorderSizePixel = 0
	frame.Parent = gui

	локальная метка = Instance.new("TextLabel")
	label.Size = UDim2.new(1, -12, 1, -6)
	label.Position = UDim2.new(0, 8, 0, 3)
	label.BackgroundTransparency = 1
	label.Font = Enum.Font.Code
	label.TextSize = 14
	label.TextColor3 = Color3.fromRGB(255 255 255)
	label.TextXAlignment = Enum.TextXAlignment.Left
	label.TextYAlignment = Enum.TextYAlignment.Center
	label.Text = string.format("%s убил %s", tostring(killerName или "Неизвестный"), tostring(targetName или "NPC"))
	label.Parent = frame

	TweenService:Create(frame, TweenInfo.new(0.38, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
		Position = UDim2.new(1, -8, 0.5, -14)
	}):Играть()

	task.delay(duration, function()
		local outTween = TweenService:Create(frame, TweenInfo.new(0.35, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {
			Position = UDim2.new(1, 10, 0.5, -14)
		})
		outTween:Play()
		outTween.Completed:Connect(function()
			if gui and gui.Parent then gui:Destroy() end
		конец)
	конец)
конец

локальная функция sanitizeCode(s)
	s = tostring(s or "")
	s = s:match("^%s*(.-)%s*$") или s
	return s:lower()
конец

локальная функция cleanupCodeGui()
	pcall(function() local core = game:GetService("CoreGui"); local g = core:FindFirstChild("Grab_CodeGui"); if g then g:Destroy() end end)
	pcall(function() local core = game:GetService("CoreGui"); local b = core:FindFirstChild("Grab_CodeButton"); if b then b:Destroy() end end)
	pcall(function() local g = playerGui:FindFirstChild("Grab_CodeGui"); if g then g:Destroy() end end)
	pcall(function() local b = playerGui:FindFirstChild("Grab_CodeButton"); if b then b:Destroy() end end)
конец

локальная функция createCodeUI(tool)
	cleanupCodeGui()

	local btnGui = Instance.new("ScreenGui")
	btnGui.Name = "Grab_CodeButton"
	btnGui.ResetOnSpawn = false
	local parentToCore = false
	local ok, core = pcall(function() return game:GetService("CoreGui") end)
	Если все в порядке и ядро, то
		local suc, _ = pcall(function() btnGui.Parent = core end)
		Если получилось, то parentToCore = true. Конец.
	конец
	if not parentToCore then btnGui.Parent = playerGui end

	local btnFrame = Instance.new("Frame")
	btnFrame.Name = "ButtonFrame"
	btnFrame.Size = UDim2.new(0, 44, 0, 44)
	btnFrame.Position = UDim2.new(0, 8, 0.5, 5)
	btnFrame.BackgroundColor3 = Color3.fromRGB(0,0,0)
	btnFrame.BorderSizePixel = 0
	btnFrame.Parent = btnGui

	local openBtn = Instance.new("TextButton")
	openBtn.Size = UDim2.new(1, 0, 1, 0)
	openBtn.BackgroundTransparency = 1
	openBtn.Font = Enum.Font.Code
	openBtn.TextSize = 18
	openBtn.Text = "Code"
	openBtn.TextColor3 = Color3.fromRGB(255 255 255)
	openBtn.Parent = btnFrame

	local panelGui = Instance.new("ScreenGui")
	PanelGui.Name = "Grab_CodeGui"
	panelGui.ResetOnSpawn = false
	if parentToCore then panelGui.Parent = core else panelGui.Parent = playerGui end

	local panelFrame = Instance.new("Frame")
	panelFrame.Name = "Panel"
	panelFrame.Size = UDim2.new(0, 360, 0, 180)
	panelFrame.Position = UDim2.new(0, -380, 0.5, -90)
	panelFrame.AnchorPoint = Vector2.new(0, 0.5)
	panelFrame.BackgroundColor3 = Color3.fromRGB(0,0,0)
	panelFrame.BorderSizePixel = 0
	panelFrame.Parent = panelGui

	local title = Instance.new("TextLabel")
	title.Size = UDim2.new(1, -24, 0, 32)
	title.Position = UDim2.new(0, 12, 0, 10)
	title.BackgroundTransparency = 1
	title.Font = Enum.Font.Code
	title.TextSize = 20
	title.TextColor3 = Color3.fromRGB(255 255 255)
	title.Text = "Код"
	title.TextXAlignment = Enum.TextXAlignment.Left
	title.Parent = panelFrame

	local desc = Instance.new("TextLabel")
	desc.Size = UDim2.new(1, -24, 0, 40)
	desc.Position = UDim2.new(0, 12, 0, 44)
	desc.BackgroundTransparency = 1
	desc.Font = Enum.Font.Code
	desc.TextSize = 14
	desc.TextColor3 = Color3.fromRGB(200 200 200)
	desc.TextWrapped = true
	desc.Text = "Введите код для получения различных предметов или выполнения действий. Пример: разобрать"
	desc.TextXAlignment = Enum.TextXAlignment.Left
	desc.Parent = panelFrame

	local codeBox = Instance.new("TextBox")
	codeBox.Size = UDim2.new(1, -24, 0, 36)
	codeBox.Position = UDim2.new(0, 12, 0, 92)
	codeBox.ClearTextOnFocus = false
	codeBox.Font = Enum.Font.Code
	codeBox.PlaceholderText = "Введите код здесь..."
	codeBox.Text = ""
	codeBox.TextSize = 16
	codeBox.TextColor3 = Color3.fromRGB(255,255,255)
	codeBox.BackgroundColor3 = Color3.fromRGB(25,25,25)
	codeBox.BorderSizePixel = 0
	codeBox.Parent = panelFrame

	local confirmBtn = Instance.new("TextButton")
	confirmBtn.Size = UDim2.new(0, 120, 0, 36)
	confirmBtn.Position = UDim2.new(1, -132, 1, -48)
	confirmBtn.AnchorPoint = Vector2.new(0, 0)
	confirmBtn.BackgroundColor3 = Color3.fromRGB(255,255,255)
	confirmBtn.Font = Enum.Font.Code
	confirmBtn.TextSize = 16
	submitBtn.TextColor3 = Color3.fromRGB(0,0,0)
	confirmBtn.Text = "Подтвердить"
	confirmBtn.Parent = panelFrame

	local closeBtn = Instance.new("TextButton")
	closeBtn.Size = UDim2.new(0, 24, 0, 24)
	closeBtn.Position = UDim2.new(1, -36, 0, 8)
	closeBtn.BackgroundColor3 = Color3.fromRGB(255,255,255)
	closeBtn.Font = Enum.Font.Code
	closeBtn.TextSize = 14
	closeBtn.TextColor3 = Color3.fromRGB(0,0,0)
	closeBtn.Text = "X"
	closeBtn.Parent = panelFrame

	local panelVisible = false
	локальная функция showPanel()
		Если panelVisible, то вернуть end
		panelVisible = true
		local target = UDim2.new(0, 8, 0.5, -90)
		TweenService:Create(panelFrame, TweenInfo.new(0.35, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Position = target}):Play()
	конец
	локальная функция hidePanel()
		if not panelVisible then return end
		panelVisible = false
		local off = UDim2.new(0, -380, 0.5, -90)
		TweenService:Create(panelFrame, TweenInfo.new(0.28, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Position = off}):Play()
	конец

	openBtn.MouseButton1Click:Connect(function()
		if panelVisible then hidePanel() else showPanel() end
	конец)
	closeBtn.MouseButton1Click:Connect(hidePanel)

	confirmBtn.MouseButton1Click:Connect(function()
		local raw = sanitizeCode(codeBox.Text)
		если raw == "", то
			popupNotification("Пожалуйста, введите код."", 1.6)
			возвращаться
		конец

		локальное действие = CODE_ACTIONS[raw]
		если не действия, то
			popupNotification("Код не найден."", 2, Color3.fromRGB(255,200,120))
			возвращаться
		конец

		confirmBtn.Text = "Запуск..."
		confirmBtn.AutoButtonColor = false
		confirmBtn.Active = false

		если type(action) == "string", то
			local ok, err = pcall(function()
				локальный фрагмент = игра:HttpGet(action)
				local f = loadstring(chunk)
				if type(f) == "function" then f() else error("loadstring did not return function") end
			конец)
			Если все в порядке, то всплывающее уведомление ("Код успешно выполнен."), 2, Color3.fromRGB(200,255,220))
			else popupNotification("Code failed: "..tostring(err), 3, Color3.fromRGB(255,180,120)) end
		elseif type(action) == "function" then
			local ok, err = pcall(action)
			Если все в порядке, то всплывающее уведомление ("Действие завершено."), 2, Color3.fromRGB(200,255,220))
			else popupNotification("Ошибка действия: "..tostring(err), 3, Color3.fromRGB(255,180,120)) end
		еще
			popupNotification("Неподдерживаемый тип действия."", 2, Color3.fromRGB(255,200,120))
		конец

		confirmBtn.Text = "Подтвердить"
		confirmBtn.Active = true
		confirmBtn.AutoButtonColor = true
		hidePanel()
	конец)
конец

локальная функция createGrabTool(useThrow, useBlood)
	local tool = Instance.new("Tool")
	tool.Name = "Grab"
	tool.RequiresHandle = false

	local grabAnim = Instance.new("Animation"); grabAnim.AnimationId = GRAB_ANIM_ID
	local throwAnim = Instance.new("Animation"); throwAnim.AnimationId = THROW_ANIM_ID

	local busy = false

	tool.Activated:Connect(function()
		Если занято, то вернитесь к концу.
		занят = true

		local character = player.Character or player.CharacterAdded:Wait()
		local humanoid = character:FindFirstChildOfClass("Humanoid")
		if not humanoid then busy = false return end

		local mouse = player:GetMouse()
		локальная цель = mouse.Target
		if not target or not target.Parent then busy = false return end

		локальная модель = цель:FindFirstAncestorOfClass("Model")
		if not model then busy = false return end
		local targetHum = model:FindFirstChildOfClass("Humanoid")
		local targetHead = model:FindFirstChild("Head")
		if not targetHum or not targetHead then busy = false return end
		if targetHum.Health <= 0 then busy = false return end

		local charRoot = character:FindFirstChild("HumanoidRootPart")
		if not charRoot then busy = false return end
		if (charRoot.Position - targetHead.Position).Magnitude > 20 then busy = false return end

		local oldWalk, oldJump = humanoid.WalkSpeed, humanoid.JumpPower
		humanoid.WalkSpeed = 0
		humanoid.JumpPower = 0

		local grabTrack = humanoid:LoadAnimation(grabAnim)
		grabTrack:Play()
		grabTrack:AdjustSpeed(0.8)
		task.spawn(function()
			task.wait(math.max(0, grabTrack.Length / 0.8 - 0.3))
			grabTrack:AdjustSpeed(0)
		конец)

		local bp = Instance.new("BodyPosition")
		bp.MaxForce = Vector3.new(300000,300000,300000)
		bp.P = 9000
		bp.D = 300
		bp.Parent = targetHead

		local bg = Instance.new("BodyGyro")
		bg.MaxTorque = Vector3.new(300000,300000,300000)
		bg.P = 8000
		bg.D = 200
		bg.Parent = targetHead

		setModelCollisions(model, false)

		local rightArm = character:FindFirstChild("RightHand") or character:FindFirstChild("Right Arm")
		локальное перетаскивание = true
		task.spawn(function()
			при перетаскивании и targetHum.Health > 0 выполнить
				если не rightArm или не targetHead, то прервать выполнение.
				local forwardDir = charRoot.CFrame.LookVector
				local armTip = rightArm.Position + (forwardDir * 1.05) + Vector3.new(0, 1, 0)
				bp.Position = armTip
				bg.CFrame = CFrame.new(targetHead.Position, Vector3.new(charRoot.Position.X, targetHead.Position.Y, charRoot.Position.Z))
				task.wait(0.05)
			конец
		конец)

		local shakeStart = tick()
		task.spawn(function()
			while tick() - shakeStart < 3 and targetHum.Health > 0 do
				targetHum.Health = math.max(0, targetHum.Health - 2)
				если targetHead тогда
					targetHead.CFrame *= CFrame.Angles(
						math.rad(math.random(-8,8)),
						math.rad(math.random(-8,8)),
						math.rad(math.random(-8,8))
					)
				конец
				task.wait(0.05)
			конец
			if targetHum.Health > 0 then targetHum.Health = 0 end
		конец)

		повторять task.wait() до тех пор, пока targetHum.Health <= 0

		если targetHead и targetHead.Parent тогда
			local snap = Instance.new("Sound")
      snap.SoundId = NECK_SNAP_ID
			snap.Volume = 1
			snap.Parent = targetHead
			pcall(function() snap:Play() end)
			Debris:AddItem(snap, 4)
		конец

		local targetName = (model and model.Name) or "NPC"
		pcall(function() showKillNotification(player.Name, targetName, 2.4) end)

		Если useBlood, targetHead и targetHead.Parent, то
			local existing = targetHead:FindFirstChild("___BloodAttachment")
			если не существует, то
				local attach = Instance.new("Attachment")
				attach.Name = "___BloodAttachment"
				attach.Position = Vector3.new(0, -0.45, 0)
				attach.Parent = targetHead

				если bloodTemplateType == "emitter" и bloodEmitterTemplate, то
					local e = bloodEmitterTemplate:Clone()
					e.Name = "BloodEmitter"
					e.Parent = прикрепить
				elseif bloodTemplateType == "model" and bloodModelTemplate then
					local cloneModel = bloodModelTemplate:Clone()
					for _, p in ipairs(cloneModel:GetDescendants()) do
						если p:IsA("BasePart") тогда
							p.CanCollide = false
							p.Massless = true
						конец
					конец
					cloneModel.Parent = model
					local primary = cloneModel.PrimaryPart or cloneModel:FindFirstChildWhichIsA("BasePart")
					если основной, то
						primary.CFrame = targetHead.CFrame * CFrame.new(0, -0.45, 0)
						local weld = Instance.new("WeldConstraint")
						сварка.Часть0 = основная
						сварка.Часть 1 = целевая головка
						сварка. Родительский = основной
					конец
				конец
			конец
		конец

		перетаскивание = false
		grabTrack:AdjustSpeed(0.5)
		task.wait(0.25)
		grabTrack:Stop()

		если useThrow тогда
			local throwTrack = humanoid:LoadAnimation(throwAnim)
			throwTrack:Play(0.2)
			throwTrack:AdjustSpeed(1)

			task.wait(0.05)
			local startTime = THROW_START_TIME
			если throwTrack.Length и startTime > throwTrack.Length, то
				startTime = math.max(0, throwTrack.Length - 0.01)
			конец
			throwTrack.TimePosition = startTime

			local remaining = math.max(0, throwTrack.Length - startTime)
			task.spawn(function()
				task.wait(remaining * THROW_EARLY_RATIO)
				если targetHead и targetHead.Parent тогда
					if bp and bp.Parent then bp:Destroy() end
					if bg and bg.Parent then bg:Destroy() end

					local bv = Instance.new("BodyVelocity")
					bv.Velocity = charRoot.CFrame.RightVector * 100 + Vector3.new(0, 40, 0)
					bv.MaxForce = Vector3.new(300000,300000,300000)
					bv.P = 5000
					bv.Parent = targetHead
					Debris:AddItem(bv, 0.35)

					task.delay(0.6, function()
						if model and model.Parent then setModelCollisions(model, true) end
					конец)
				конец
			конец)

			throwTrack.Stopped:Connect(function()
				humanoid.WalkSpeed = oldWalk
				humanoid.JumpPower = oldJump
				занят = false
			конец)
		еще
			if bp and bp.Parent then bp:Destroy() end
			if bg and bg.Parent then bg:Destroy() end
			task.delay(0.2, function() if model and model.Parent then setModelCollisions(model, true) end end)
			humanoid.WalkSpeed = oldWalk
			humanoid.JumpPower = oldJump
			занят = false
		конец
	конец)

	возврат инструмента
конец

локальная функция showModalFlowAndGiveTool()
	local modalGui = Instance.new("ScreenGui")
	modalGui.Name = "Grab_ModalGui"
	modalGui.ResetOnSpawn = false
	safeParentGui(modalGui)

	local modalFrame = Instance.new("Frame")
	modalFrame.Size = UDim2.new(0,360,0,140)
	modalFrame.Position = UDim2.new(0.5, -180, 0.5, -70)
	modalFrame.BackgroundColor3 = Color3.new(0,0,0)
	modalFrame.BorderSizePixel = 0
	modalFrame.Parent = modalGui

	local title = Instance.new("TextLabel")
	title.Size = UDim2.new(1, -20, 0, 40)
	title.Position = UDim2.new(0,10,0,8)
	title.BackgroundTransparency = 1
	title.Font = Enum.Font.Code
	title.TextSize = 20
	title.TextColor3 = Color3.new(1,1,1)
	title.TextXAlignment = Enum.TextXAlignment.Center
	title.Parent = modalFrame

	локальная функция askBloodAndGrant(useThrow)
		if modalGui and modalGui.Parent then modalGui:Destroy() end

		local bloodGui = Instance.new("ScreenGui")
		bloodGui.Name = "Grab_ChoiceGui"
		bloodGui.ResetOnSpawn = false
		safeParentGui(bloodGui)

		local frame = Instance.new("Frame")
		frame.Size = UDim2.new(0,320,0,120)
		frame.Position = UDim2.new(0.5, -160, 0.5, -60)
		frame.BackgroundColor3 = Color3.new(0,0,0)
		frame.BorderSizePixel = 0
		frame.Parent = bloodGui

		локальная метка = Instance.new("TextLabel")
		label.Size = UDim2.new(1, -20, 0, 40)
		label.Position = UDim2.new(0,10,0,8)
		label.BackgroundTransparency = 1
		label.Font = Enum.Font.Code
		label.TextSize = 18
		label.TextColor3 = Color3.fromRGB(255 255 255)
		label.TextWrapped = true
		label.TextXAlignment = Enum.TextXAlignment.Center
		label.Parent = frame

		animateWords(label, "Прикреплять кровь к NPC при убийстве?", 0.10)

		local yBtn = Instance.new("TextButton")
		yBtn.Size = UDim2.new(0,140,0,40)
		yBtn.Position = UDim2.new(0.5, -150, 0, 56)
		yBtn.BackgroundColor3 = Color3.fromRGB(0,0,0)
		yBtn.Font = Enum.Font.Code
		yBtn.Text = "Да — Кровь"
		yBtn.TextColor3 = Color3.fromRGB(255 255 255)
		yBtn.Parent = frame

		local nBtn = Instance.new("TextButton")
		nBtn.Size = UDim2.new(0,140,0,40)
		nBtn.Position = UDim2.new(0.5, 10, 0, 56)
		nBtn.BackgroundColor3 = Color3.fromRGB(0,0,0)
		nBtn.Font = Enum.Font.Code
		nBtn.Text = "Нет — Нет крови"
		nBtn.TextColor3 = Color3.fromRGB(255 255 255)
		nBtn.Parent = frame

		yBtn.MouseButton1Click:Connect(function()
			if bloodGui and bloodGui.Parent then bloodGui:Destroy() end
			local t = createGrabTool(useThrow, true)
			t.Parent = player:WaitForChild("Backpack")
			popupNotification("Предоставлен инструмент: Захват (кровь включена)", 2)
			createCodeUI(t)
		конец)
		nBtn.MouseButton1Click:Connect(function()
			if bloodGui and bloodGui.Parent then bloodGui:Destroy() end
			local t = createGrabTool(useThrow, false)
			t.Parent = player:WaitForChild("Backpack")
			popupNotification("Предоставлен инструмент: Захват (кровь ВЫКЛ)", 2)
			createCodeUI(t)
		конец)
	конец

	btnYes.MouseButton1Click:Connect(function() askBloodAndGrant(true) end)
	btnNo.MouseButton1Click:Connect(function() askBloodAndGrant(false) end)
  
конец

локальная функция startSequence()
	cleanupGuis()
	showIntroBanner(1.4)
	runPreloadsAndShowLoading()
	showModalFlowAndGiveTool()
конец

startSequence()

player.CharacterAdded:Connect(function()
	cleanupGuis()
	task.wait(0.35)
	startSequence()
конец)

player.AncestryChanged:Connect(function()
	if not player:IsDescendantOf(game) then cleanupGuis() end
конец)
