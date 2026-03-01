local LP = game:GetService("Players").LocalPlayer
local HttpService = game:GetService("HttpService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local FileName = "Navigator_Fixed_Final.json"

local function save(d) pcall(function() writefile(FileName, HttpService:JSONEncode(d)) end) end
local function load() return isfile(FileName) and HttpService:JSONDecode(readfile(FileName)) or {} end
local locations = load()

local beacons = {}
local moveMode = "TP" 
local isMoving = false

-- ПЛАВНЫЙ ДРАГ (ЗАМЕНА УСТАРЕВШЕМУ .DRAGGABLE)
local function makeDraggable(obj)
    local dragging, dragInput, dragStart, startPos
    obj.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true; dragStart = input.Position; startPos = obj.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then dragging = false end
            end)
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local delta = input.Position - dragStart
            obj.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
end

-- ПЕРЕМЕЩЕНИЕ
local function flyTo(targetPos)
    local char = LP.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if not hrp or isMoving then return end

    if moveMode == "TP" then
        hrp.CFrame = CFrame.new(targetPos + Vector3.new(0, 3, 0))
    else
        isMoving = true
        local dist = (hrp.Position - targetPos).Magnitude
        local speed = 150 
        
        local bv = Instance.new("BodyVelocity", hrp)
        bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
        bv.Velocity = Vector3.zero
        
        local bg = Instance.new("BodyGyro", hrp)
        bg.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
        bg.CFrame = hrp.CFrame

        local tween = TweenService:Create(hrp, TweenInfo.new(dist/speed, Enum.EasingStyle.Linear), {CFrame = CFrame.new(targetPos)})
        tween:Play()
        tween.Completed:Connect(function()
            bv:Destroy(); bg:Destroy()
            isMoving = false
        end)
    end
end

-- МАЯКИ (СТОЛБЫ)
local function createVisuals(name, coordsStr)
    if beacons[name] then beacons[name]:Destroy() end
    local t = {} for s in coordsStr:gmatch("[^,]+") do table.insert(t, tonumber(s)) end
    if #t < 3 then return end
    local pos = Vector3.new(t[1], t[2], t[3])

    local folder = Instance.new("Model", workspace)
    folder.Name = "Beacon_" .. name

    local attTop = Instance.new("Attachment", workspace.Terrain)
    attTop.WorldPosition = pos + Vector3.new(0, 5000, 0)
    local attBottom = Instance.new("Attachment", workspace.Terrain)
    attBottom.WorldPosition = pos + Vector3.new(0, -5000, 0)

    local beam = Instance.new("Beam", folder)
    beam.Attachment0 = attTop; beam.Attachment1 = attBottom
    beam.Width0 = 3; beam.Width1 = 3
    beam.Color = ColorSequence.new(Color3.fromRGB(0, 255, 100))
    beam.Transparency = NumberSequence.new(0.5); beam.LightEmission = 1; beam.FaceCamera = true

    local anchor = Instance.new("Part", folder)
    anchor.Anchored = true; anchor.CanCollide = false; anchor.Transparency = 1; anchor.Position = pos

    local bgui = Instance.new("BillboardGui", anchor)
    bgui.Size = UDim2.new(0, 200, 0, 50); bgui.AlwaysOnTop = true

    local label = Instance.new("TextLabel", bgui)
    label.Size = UDim2.new(1, 0, 1, 0); label.BackgroundTransparency = 1
    label.TextColor3 = Color3.fromRGB(255, 255, 255); label.TextSize = 16; label.Font = Enum.Font.SourceSansBold

    task.spawn(function()
        while folder.Parent do
            if LP.Character and LP.Character:FindFirstChild("HumanoidRootPart") then
                local d = math.floor((LP.Character.HumanoidRootPart.Position - pos).Magnitude)
                label.Text = name .. "\n" .. d .. "m"
            end
            task.wait(0.2)
        end
        attTop:Destroy(); attBottom:Destroy()
    end)
    beacons[name] = folder
end

local function refreshBeacons()
    for _, v in pairs(beacons) do v:Destroy() end
    beacons = {}
    for n, c in pairs(locations) do createVisuals(n, c) end
end

-- UI
local Screen = Instance.new("ScreenGui", game:GetService("CoreGui"))
Screen.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

-- КНОПКА-ВИДЖЕТ (КАК НА КАРТИНКЕ)
local function createQuickButton(name, targetPos)
    local QuickBtn = Instance.new("Frame", Screen)
    QuickBtn.Size = UDim2.new(0, 120, 0, 60)
    QuickBtn.Position = UDim2.new(0.8, 0, 0.2, 0)
    QuickBtn.BackgroundColor3 = Color3.fromRGB(180, 180, 180)
    QuickBtn.BorderSizePixel = 2; QuickBtn.ZIndex = 100
    makeDraggable(QuickBtn)

    local RedPart = Instance.new("TextButton", QuickBtn)
    RedPart.Size = UDim2.new(0.7, -4, 1, -8); RedPart.Position = UDim2.new(0, 4, 0, 4)
    RedPart.BackgroundColor3 = Color3.fromRGB(255, 0, 0); RedPart.Text = "TP"
    RedPart.TextColor3 = Color3.fromRGB(180, 160, 255); RedPart.Font = Enum.Font.GothamBlack; RedPart.TextSize = 35
    RedPart.BorderSizePixel = 3; RedPart.BorderColor3 = Color3.fromRGB(150, 0, 0); RedPart.ZIndex = 101

    local Icon = Instance.new("TextLabel", QuickBtn)
    Icon.Size = UDim2.new(0.3, 0, 1, 0); Icon.Position = UDim2.new(0.7, 0, 0, 0)
    Icon.BackgroundTransparency = 1; Icon.Text = "⇆"; Icon.TextColor3 = Color3.new(0,0,0)
    Icon.TextSize = 25; Icon.ZIndex = 101

    local Close = Instance.new("TextButton", QuickBtn)
    Close.Size = UDim2.new(0, 16, 0, 16); Close.Position = UDim2.new(1, -16, 0, 0)
    Close.BackgroundColor3 = Color3.fromRGB(200, 50, 50); Close.Text = "x"; Close.TextColor3 = Color3.new(1,1,1); Close.ZIndex = 105

    RedPart.MouseButton1Click:Connect(function() flyTo(targetPos) end)
    Close.MouseButton1Click:Connect(function() QuickBtn:Destroy() end)
end

-- ОСНОВНОЕ ОКНО
local ToggleBtn = Instance.new("TextButton", Screen)
ToggleBtn.Size = UDim2.new(0, 50, 0, 30); ToggleBtn.Position = UDim2.new(0, 10, 0.5, 0)
ToggleBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60); ToggleBtn.Text = "tp"
ToggleBtn.TextColor3 = Color3.fromRGB(100, 255, 100); ToggleBtn.TextSize = 20; ToggleBtn.BorderSizePixel = 2
makeDraggable(ToggleBtn)

local MainFrame = Instance.new("Frame", Screen)
MainFrame.Size = UDim2.new(0, 450, 0, 320); MainFrame.Position = UDim2.new(0.5, -225, 0.5, -160)
MainFrame.BackgroundColor3 = Color3.fromRGB(180, 180, 180); MainFrame.BorderSizePixel = 4; MainFrame.ZIndex = 10
makeDraggable(MainFrame)

ToggleBtn.MouseButton1Click:Connect(function() MainFrame.Visible = not MainFrame.Visible end)

-- ВКЛАДКИ
local TabBar = Instance.new("Frame", MainFrame)
TabBar.Size = UDim2.new(1, 0, 0, 40); TabBar.BackgroundColor3 = Color3.fromRGB(60, 60, 60)

local btnCords = Instance.new("TextButton", TabBar)
btnCords.Size = UDim2.new(0.5, 0, 1, 0); btnCords.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
btnCords.Text = "cords"; btnCords.TextColor3 = Color3.fromRGB(100, 255, 100); btnCords.TextSize = 22

local btnSaved = Instance.new("TextButton", TabBar)
btnSaved.Size = UDim2.new(0.5, 0, 1, 0); btnSaved.Position = UDim2.new(0.5, 0, 0, 0)
btnSaved.BackgroundColor3 = Color3.fromRGB(80, 80, 80); btnSaved.Text = "saved"
btnSaved.TextColor3 = Color3.fromRGB(100, 255, 100); btnSaved.TextSize = 22

local PageCords = Instance.new("Frame", MainFrame)
PageCords.Size = UDim2.new(1, 0, 1, -40); PageCords.Position = UDim2.new(0, 0, 0, 40); PageCords.BackgroundTransparency = 1

local PageSaved = Instance.new("Frame", MainFrame)
PageSaved.Size = UDim2.new(1, 0, 1, -40); PageSaved.Position = UDim2.new(0, 0, 0, 40); PageSaved.BackgroundTransparency = 1; PageSaved.Visible = false

btnCords.MouseButton1Click:Connect(function() PageCords.Visible = true; PageSaved.Visible = false end)
btnSaved.MouseButton1Click:Connect(function() PageCords.Visible = false; PageSaved.Visible = true end)

--- [ CORDS PAGE ] ---
local PosShow = Instance.new("TextLabel", PageCords)
PosShow.Size = UDim2.new(1, 0, 0, 30); PosShow.BackgroundTransparency = 1
PosShow.TextColor3 = Color3.fromRGB(40, 40, 40); PosShow.TextSize = 16

task.spawn(function()
    while true do
        if LP.Character and LP.Character:FindFirstChild("HumanoidRootPart") then
            local p = LP.Character.HumanoidRootPart.Position
            PosShow.Text = string.format("Current: %.0f, %.0f, %.0f", p.X, p.Y, p.Z)
        end
        task.wait(0.2)
    end
end)

local InputBox = Instance.new("TextBox", PageCords)
InputBox.Size = UDim2.new(0.7, 0, 0, 45); InputBox.Position = UDim2.new(0.05, 0, 0.15, 0)
InputBox.BackgroundColor3 = Color3.fromRGB(70, 70, 70); InputBox.TextColor3 = Color3.fromRGB(100, 255, 100)
InputBox.PlaceholderText = "Input X, Y, Z"; InputBox.TextSize = 20

local btnTP = Instance.new("TextButton", PageCords)
btnTP.Size = UDim2.new(0.2, 0, 0, 45); btnTP.Position = UDim2.new(0.76, 0, 0.15, 0)
btnTP.BackgroundColor3 = Color3.fromRGB(200, 50, 50); btnTP.Text = "MOVE"; btnTP.TextColor3 = Color3.new(1,1,1); btnTP.TextSize = 20

local bSave = Instance.new("TextButton", PageCords)
bSave.Size = UDim2.new(0.18, 0, 0, 35); bSave.Position = UDim2.new(0.05, 0, 0.4, 0)
bSave.BackgroundColor3 = Color3.fromRGB(100, 180, 100); bSave.Text = "save"

local bClean = Instance.new("TextButton", PageCords)
bClean.Size = UDim2.new(0.18, 0, 0, 35); bClean.Position = UDim2.new(0.25, 0, 0.4, 0)
bClean.BackgroundColor3 = Color3.fromRGB(200, 180, 50); bClean.Text = "clean"

local bMode = Instance.new("TextButton", PageCords)
bMode.Size = UDim2.new(0.22, 0, 0, 35); bMode.Position = UDim2.new(0.45, 0, 0.4, 0)
bMode.BackgroundColor3 = Color3.fromRGB(80, 120, 200); bMode.Text = "Mode: TP"

local bThis = Instance.new("TextButton", PageCords)
bThis.Size = UDim2.new(0.22, 0, 0, 35); bThis.Position = UDim2.new(0.7, 0, 0.4, 0)
bThis.BackgroundColor3 = Color3.fromRGB(200, 100, 100); bThis.Text = "this pos"

bMode.MouseButton1Click:Connect(function()
    moveMode = (moveMode == "TP") and "Fly" or "TP"
    bMode.Text = "Mode: " .. moveMode
    bMode.BackgroundColor3 = (moveMode == "TP") and Color3.fromRGB(80, 120, 200) or Color3.fromRGB(200, 150, 50)
end)

--- [ SAVED PAGE ] ---
local Scroll = Instance.new("ScrollingFrame", PageSaved)
Scroll.Size = UDim2.new(1, -20, 1, -20); Scroll.Position = UDim2.new(0, 10, 0, 10)
Scroll.BackgroundTransparency = 1; Scroll.BorderSizePixel = 0

local Layout = Instance.new("UIListLayout", Scroll)
Layout.Padding = UDim.new(0, 5)

local function updateSavedUI()
    for _, child in pairs(Scroll:GetChildren()) do if child:IsA("Frame") then child:Destroy() end end
    for name, coords in pairs(locations) do
        local Frame = Instance.new("Frame", Scroll)
        Frame.Size = UDim2.new(1, -10, 0, 60); Frame.BackgroundColor3 = Color3.fromRGB(120, 120, 120)
        
        local editName = Instance.new("TextBox", Frame)
        editName.Size = UDim2.new(0.5, 0, 0.5, 0); editName.Position = UDim2.new(0.02, 0, 0, 0)
        editName.Text = name; editName.TextColor3 = Color3.new(1,1,1); editName.BackgroundTransparency = 1; editName.TextXAlignment = 0

        local info = Instance.new("TextLabel", Frame)
        info.Size = UDim2.new(0.5, 0, 0.5, 0); info.Position = UDim2.new(0.02, 0, 0.5, 0)
        info.Text = coords; info.TextColor3 = Color3.fromRGB(255, 80, 80); info.BackgroundTransparency = 1; info.TextXAlignment = 0; info.TextSize = 10
        
        -- КНОПКА ПЛЮС ДЛЯ ВИДЖЕТА
        local addW = Instance.new("TextButton", Frame)
        addW.Size = UDim2.new(0, 25, 0, 25); addW.Position = UDim2.new(0.55, 0, 0.3, 0)
        addW.BackgroundColor3 = Color3.fromRGB(80, 80, 80); addW.Text = "+"; addW.TextColor3 = Color3.new(1,1,1)

        local tpBtn = Instance.new("TextButton", Frame)
        tpBtn.Size = UDim2.new(0.3, 0, 0.4, 0); tpBtn.Position = UDim2.new(0.68, 0, 0.08, 0)
        tpBtn.BackgroundColor3 = Color3.fromRGB(0, 180, 0); tpBtn.Text = "go"
        
        local delBtn = Instance.new("TextButton", Frame)
        delBtn.Size = UDim2.new(0.3, 0, 0.4, 0); delBtn.Position = UDim2.new(0.68, 0, 0.52, 0)
        delBtn.BackgroundColor3 = Color3.fromRGB(180, 0, 0); delBtn.Text = "delete"

        local function parse(c)
            local t = {} for s in c:gmatch("[^,]+") do table.insert(t, tonumber(s)) end
            return Vector3.new(t[1], t[2], t[3])
        end

        addW.MouseButton1Click:Connect(function() createQuickButton(name, parse(coords)) end)
        tpBtn.MouseButton1Click:Connect(function() flyTo(parse(coords)) end)
        delBtn.MouseButton1Click:Connect(function() locations[name] = nil; save(locations); updateSavedUI(); refreshBeacons() end)
    end
    Scroll.CanvasSize = UDim2.new(0, 0, 0, Layout.AbsoluteContentSize.Y + 20)
end

bThis.MouseButton1Click:Connect(function()
    local r = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
    if r then
        locations["Point_" .. os.date("%H%M%S")] = math.floor(r.Position.X)..","..math.floor(r.Position.Y)..","..math.floor(r.Position.Z)
        save(locations); updateSavedUI(); refreshBeacons()
    end
end)

bSave.MouseButton1Click:Connect(function()
    if InputBox.Text ~= "" then
        locations["Manual_" .. os.date("%H%M%S")] = InputBox.Text
        save(locations); updateSavedUI(); refreshBeacons()
    end
end)

bClean.MouseButton1Click:Connect(function() InputBox.Text = "" end)
btnTP.MouseButton1Click:Connect(function()
    local t = {} for s in InputBox.Text:gmatch("[^,;%s]+") do table.insert(t, tonumber(s)) end
    if #t >= 3 then flyTo(Vector3.new(t[1], t[2], t[3])) end
end)

updateSavedUI()
refreshBeacons()
