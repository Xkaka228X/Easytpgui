local LP = game:GetService("Players").LocalPlayer
local HttpService = game:GetService("HttpService")
local TweenService = game:GetService("TweenService")
local FileName = "Navigator_Fixed_Final.json"

local function save(d) writefile(FileName, HttpService:JSONEncode(d)) end
local function load() return isfile(FileName) and HttpService:JSONDecode(readfile(FileName)) or {} end
local locations = load()

-- UI ОСНОВА
local Screen = Instance.new("ScreenGui", game:GetService("CoreGui"))

-- КНОПКА SHOW/HIDE
local ToggleBtn = Instance.new("TextButton", Screen)
ToggleBtn.Size = UDim2.new(0, 50, 0, 30)
ToggleBtn.Position = UDim2.new(0, 10, 0.5, 0)
ToggleBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
ToggleBtn.Text = "tp"
ToggleBtn.TextColor3 = Color3.fromRGB(100, 255, 100)
ToggleBtn.Font = Enum.Font.SourceSansBold
ToggleBtn.TextSize = 20
ToggleBtn.BorderSizePixel = 2

local MainFrame = Instance.new("Frame", Screen)
MainFrame.Size = UDim2.new(0, 450, 0, 320)
MainFrame.Position = UDim2.new(0.5, -225, 0.5, -160)
MainFrame.BackgroundColor3 = Color3.fromRGB(180, 180, 180)
MainFrame.BorderSizePixel = 4
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Visible = true

-- Логика кнопки TP (Show/Hide)
ToggleBtn.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
end)

-- ВКЛАДКИ
local TabBar = Instance.new("Frame", MainFrame)
TabBar.Size = UDim2.new(1, 0, 0, 40)
TabBar.BackgroundColor3 = Color3.fromRGB(60, 60, 60)

local function createTabBtn(name, pos)
    local btn = Instance.new("TextButton", TabBar)
    btn.Size = UDim2.new(0.5, 0, 1, 0)
    btn.Position = UDim2.new(pos * 0.5, 0, 0, 0)
    btn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
    btn.Text = name
    btn.TextColor3 = Color3.fromRGB(100, 255, 100)
    btn.Font = Enum.Font.SourceSansBold
    btn.TextSize = 22
    return btn
end

local btnCords = createTabBtn("cords", 0)
local btnSaved = createTabBtn("saved", 1)

-- СТРАНИЦЫ
local PageCords = Instance.new("Frame", MainFrame)
PageCords.Size = UDim2.new(1, 0, 1, -40)
PageCords.Position = UDim2.new(0, 0, 0, 40)
PageCords.BackgroundTransparency = 1

local PageSaved = Instance.new("Frame", MainFrame)
PageSaved.Size = UDim2.new(1, 0, 1, -40)
PageSaved.Position = UDim2.new(0, 0, 0, 40)
PageSaved.BackgroundTransparency = 1
PageSaved.Visible = false

btnCords.MouseButton1Click:Connect(function()
    PageCords.Visible = true
    PageSaved.Visible = false
end)

btnSaved.MouseButton1Click:Connect(function()
    PageCords.Visible = false
    PageSaved.Visible = true
end)

--- [ КОНТЕНТ CORDS ] ---
local InputBox = Instance.new("TextBox", PageCords)
InputBox.Size = UDim2.new(0.7, 0, 0, 45)
InputBox.Position = UDim2.new(0.05, 0, 0.1, 0)
InputBox.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
InputBox.TextColor3 = Color3.fromRGB(100, 255, 100)
InputBox.PlaceholderText = "~;~;~"
InputBox.Text = ""
InputBox.TextSize = 20

local btnTP = Instance.new("TextButton", PageCords)
btnTP.Size = UDim2.new(0.2, 0, 0, 45)
btnTP.Position = UDim2.new(0.76, 0, 0.1, 0)
btnTP.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
btnTP.Text = "TP"
btnTP.TextColor3 = Color3.fromRGB(150, 150, 250)
btnTP.TextSize = 25

local function createSmallBtn(text, color, x, size)
    local b = Instance.new("TextButton", PageCords)
    b.Size = UDim2.new(size, 0, 0, 35)
    b.Position = UDim2.new(x, 0, 0.35, 0)
    b.BackgroundColor3 = color
    b.Text = text
    b.TextColor3 = Color3.fromRGB(255, 255, 255)
    b.TextSize = 14
    b.Font = Enum.Font.SourceSansBold
    return b
end

local bSave = createSmallBtn("save", Color3.fromRGB(100, 180, 100), 0.05, 0.18)
local bClean = createSmallBtn("clean", Color3.fromRGB(200, 180, 50), 0.25, 0.18)
local bFly = createSmallBtn("Fly Mode", Color3.fromRGB(80, 150, 200), 0.45, 0.22)
local bThis = createSmallBtn("this pos", Color3.fromRGB(200, 100, 100), 0.7, 0.22)

--- [ ЛОГИКА SAVED ] ---
local Scroll = Instance.new("ScrollingFrame", PageSaved)
Scroll.Size = UDim2.new(1, -20, 1, -20)
Scroll.Position = UDim2.new(0, 10, 0, 10)
Scroll.BackgroundTransparency = 1
Scroll.BorderSizePixel = 0
Scroll.CanvasSize = UDim2.new(0, 0, 0, 0)

local Layout = Instance.new("UIListLayout", Scroll)
Layout.Padding = UDim.new(0, 5)

local function updateSavedUI()
    for _, child in pairs(Scroll:GetChildren()) do if child:IsA("Frame") then child:Destroy() end end
    for name, coords in pairs(locations) do
        local Frame = Instance.new("Frame", Scroll)
        Frame.Size = UDim2.new(1, -10, 0, 60)
        Frame.BackgroundColor3 = Color3.fromRGB(120, 120, 120)
        
        local info = Instance.new("TextLabel", Frame)
        info.Size = UDim2.new(0.6, 0, 1, 0)
        info.Position = UDim2.new(0.02, 0, 0, 0)
        info.Text = name .. "\n" .. coords
        info.TextColor3 = Color3.fromRGB(255, 80, 80)
        info.TextXAlignment = Enum.TextXAlignment.Left
        info.BackgroundTransparency = 1
        info.TextSize = 14
        
        local tpBtn = Instance.new("TextButton", Frame)
        tpBtn.Size = UDim2.new(0.3, 0, 0.4, 0)
        tpBtn.Position = UDim2.new(0.65, 0, 0.08, 0)
        tpBtn.BackgroundColor3 = Color3.fromRGB(0, 180, 0)
        tpBtn.Text = "teleport"
        
        local delBtn = Instance.new("TextButton", Frame)
        delBtn.Size = UDim2.new(0.3, 0, 0.4, 0)
        delBtn.Position = UDim2.new(0.65, 0, 0.52, 0)
        delBtn.BackgroundColor3 = Color3.fromRGB(180, 0, 0)
        delBtn.Text = "delete"

        tpBtn.MouseButton1Click:Connect(function()
            local t = {} for s in coords:gmatch("[^,]+") do table.insert(t, tonumber(s)) end
            LP.Character.HumanoidRootPart.CFrame = CFrame.new(t[1], t[2], t[3])
        end)
        
        delBtn.MouseButton1Click:Connect(function()
            locations[name] = nil
            save(locations)
            updateSavedUI()
        end)
    end
    Scroll.CanvasSize = UDim2.new(0, 0, 0, Layout.AbsoluteContentSize.Y + 20)
end

bThis.MouseButton1Click:Connect(function()
    local p = LP.Character.HumanoidRootPart.Position
    local uniqueID = "Pos_" .. os.date("%H%M%S") .. "_" .. math.random(10,99)
    locations[uniqueID] = math.floor(p.X)..","..math.floor(p.Y)..","..math.floor(p.Z)
    save(locations)
    updateSavedUI()
end)

bSave.MouseButton1Click:Connect(function()
    if InputBox.Text ~= "" then
        local uniqueID = "Manual_" .. os.date("%H%M%S")
        locations[uniqueID] = InputBox.Text
        save(locations)
        updateSavedUI()
    end
end)

bClean.MouseButton1Click:Connect(function() InputBox.Text = "" end)

btnTP.MouseButton1Click:Connect(function()
    local t = {} for s in InputBox.Text:gmatch("[^,;%s]+") do table.insert(t, tonumber(s)) end
    if #t >= 3 then LP.Character.HumanoidRootPart.CFrame = CFrame.new(t[1], t[2], t[3]) end
end)

updateSavedUI()
