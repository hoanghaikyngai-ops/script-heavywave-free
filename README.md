-- [[ 🍯 HOANGHAI HUB - HEAVYWEIGHT FISHING ]] --

-- Load thư viện giao diện chính Fluent UI
local v0, v1 = pcall(function() 
    return loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))(); 
end)

if not v0 or not v1 then 
    warn("Failed to load Fluent")
    return 
end 

-- Load các Addon (SaveManager và InterfaceManager)
local v2 = nil
local v3 = nil

pcall(function() 
    local v63 = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/SaveManager.lua"))
    if v63 then 
        v2 = v63() 
    end 
end)

pcall(function() 
    local v64 = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/InterfaceManager.lua"))
    if v64 then 
        v3 = v64() 
    end 
end)

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local VirtualInputManager = game:GetService("VirtualInputManager")

local LocalPlayer = Players.LocalPlayer
local MainGui = LocalPlayer.PlayerGui:WaitForChild("MainGui")

-- Thiết lập các biến cấu hình toàn cục (Global Variables) cho Hub
getgenv().HOANGHAI_Anchor = false
getgenv().HOANGHAI_AutoCast = false
getgenv().HOANGHAI_AutoSell = false
getgenv().HOANGHAI_SellDelay = 2 -- Đã giải mã từ biểu thức 1070 - (68 + 997)
getgenv().HOANGHAI_AutoSkill = false
getgenv().HOANGHAI_Skills = { Z = true, X = true, C = true, V = true }

-- Bảng ngôn ngữ hỗ trợ (Thái, Anh, Việt)
local Translations = {
    TH = {
        win_title = "🍯 HOANGHAI Hub",
        win_sub = "[🌴UPD] Heavyweight Fishing",
        tab_fishing = "ตกปลา",
        tab_sell = "ขายปลา",
        tab_tp = "เทเลพอร์ต",
        tab_settings = "ตั้งค่า",
        anchor = "ล็อกแถบ (Anchor)",
        autocast = "เหวี่ยงเบ็ดอัตโนมัติ",
        autoskill = "ออโต้สกิล",
        skill_pick = "เลือกสกิลที่จะกด",
        autosell = "ขายปลาอัตโนมัติ",
        selldelay = "หน่วงเวลาขาย (วินาที)",
        island = "เกาะ",
        loaded = "โหลดสำเร็จ!",
        tp_ok = "เทเลพอร์ตแล้ว!",
        tp_fail = "เทเลพอร์ตไม่ได้",
        no_char = "ไม่พบตัวละคร"
    },
    EN = {
        win_title = "🍯 HOANGHAI Hub",
        win_sub = "[🌴UPD] Heavyweight Fishing",
        tab_fishing = "Fishing",
        tab_sell = "Auto Sell",
        tab_tp = "Teleport",
        tab_settings = "Settings",
        anchor = "Anchor (Lock Bar)",
        autocast = "Auto Cast",
        autoskill = "Auto Skill",
        skill_pick = "Select Skills to Press",
        autosell = "Auto Sell Fish",
        selldelay = "Sell Delay (Seconds)",
        island = "Island",
        loaded = "Loaded successfully!",
        tp_ok = "Teleported!",
        tp_fail = "Teleport failed",
        no_char = "Character not found"
    },
    VN = {
        win_title = "🍯 HOANGHAI Hub",
        win_sub = "Fishing Edition",
        tab_fishing = "Cau ca",
        tab_sell = "Tu ban ca",
        tab_tp = "Dich chuyen",
        tab_settings = "Cai dat",
        anchor = "Khoa thanh (Anchor)",
        autocast = "Tu dong nem cau",
        autoskill = "Tu dong skill",
        skill_pick = "Chon skill can bam",
        autosell = "Tu dong ban ca",
        selldelay = "Delay ban (giay)",
        island = "Dao",
        loaded = "Tai thanh cong!",
        tp_ok = "Da dich chuyen!",
        tp_fail = "Dich chuyen that bai",
        no_char = "Khong tim thay nhan vat"
    }
}

local selectedLang = nil
local langChosenEvent = Instance.new("BindableEvent")

-- Tạo giao diện chọn ngôn ngữ lúc đầu (Language Picker UI)
local LangGui = Instance.new("ScreenGui")
LangGui.Name = "HOANGHAILangPicker"
LangGui.ResetOnSpawn = false
LangGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
LangGui.Parent = LocalPlayer.PlayerGui

local LangFrame = Instance.new("Frame")
LangFrame.Size = UDim2.new(0, 300, 0, 85) -- Đã giải mã kích thước
LangFrame.Position = UDim2.new(0.5, -150, 0.4, -42)
LangFrame.BackgroundColor3 = Color3.fromRGB(18, 20, 25)
LangFrame.BorderSizePixel = 0
LangFrame.Parent = LangGui
Instance.new("UICorner", LangFrame).CornerRadius = UDim.new(0, 12)

local LangTitle = Instance.new("TextLabel")
LangTitle.Size = UDim2.new(1, 0, 0, 48)
LangTitle.BackgroundTransparency = 1
LangTitle.TextColor3 = Color3.new(1, 1, 1)
LangTitle.TextScaled = true
LangTitle.Font = Enum.Font.GothamBold
LangTitle.Text = "🍯 HOANGHAI Hub — Select Language"
LangTitle.Parent = LangFrame

local LangButtonContainer = Instance.new("Frame")
LangButtonContainer.Size = UDim2.new(1, -20, 0, 85)
LangButtonContainer.Position = UDim2.new(0, 10, 0, 58)
LangButtonContainer.BackgroundTransparency = 0
LangButtonContainer.Parent = LangFrame

local UIList = Instance.new("UIListLayout")
UIList.FillDirection = Enum.FillDirection.Horizontal
UIList.HorizontalAlignment = Enum.HorizontalAlignment.Center
UIList.VerticalAlignment = Enum.VerticalAlignment.Center
UIList.Padding = UDim.new(0, 12)
UIList.Parent = LangButtonContainer

-- Thêm các nút chọn ngôn ngữ (Thai, English, Vietnamese)
local languages = {
    {label = "🇹🇭 Thai", key = "TH", color = Color3.fromRGB(0, 120, 200)},
    {label = "🇬🇧 English", key = "EN", color = Color3.fromRGB(0, 160, 80)},
    {label = "🇻🇳 Viet", key = "VN", color = Color3.fromRGB(200, 85, 58)}
}

for _, langInfo in ipairs(languages) do
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 80, 0, 38)
    btn.BackgroundColor3 = langInfo.color
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.TextScaled = true
    btn.Font = Enum.Font.GothamSemibold
    btn.Text = langInfo.label
    btn.Parent = LangButtonContainer
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)
    
    btn.MouseButton1Click:Connect(function()
        selectedLang = langInfo.key
        langChosenEvent:Fire()
    end)
end

-- Chờ người dùng bấm chọn ngôn ngữ rồi mới xóa bảng chọn đi
langChosenEvent.Event:Wait()
LangGui:Destroy()
langChosenEvent:Destroy()

-- Lấy từ điển ngôn ngữ tương ứng
local T = Translations[selectedLang]

-- Khởi tạo cửa sổ chính của Fluent UI
local Window = v1:CreateWindow({
    Title = T.win_title,
    SubTitle = T.win_sub,
    TabWidth = 180,
    Size = UDim2.fromOffset(560, 420),
    Acrylic = true,
    Theme = "Dark",
    MinimizeKey = Enum.KeyCode.RightControl
})

-- Tạo các Tab chức năng
local Tabs = {
    Fishing = Window:AddTab({ Title = T.tab_fishing, Icon = "fish" }),
    Sell = Window:AddTab({ Title = T.tab_sell, Icon = "shopping-cart" }),
    Teleport = Window:AddTab({ Title = T.tab_tp, Icon = "map-pin" }),
    Settings = Window:AddTab({ Title = T.tab_settings, Icon = "settings" })
}

-- Tab Câu Cá (Fishing)
Tabs.Fishing:AddToggle("HOANGHAI_Anchor", {
    Title = T.anchor,
    Default = false,
    Callback = function(value)
        getgenv().HOANGHAI_Anchor = value
    end
})

Tabs.Fishing:AddToggle("HOANGHAI_AutoCast", {
    Title = T.autocast,
    Default = false,
    Callback = function(value)
        getgenv().HOANGHAI_AutoCast = value
    end
})

Tabs.Fishing:AddToggle("HOANGHAI_AutoSkill", {
    Title = T.autoskill,
    Default = false,
    Callback = function(value)
        getgenv().HOANGHAI_AutoSkill = value
    end
})

Tabs.Fishing:AddDropdown("HOANGHAI_SkillPick", {
    Title = T.skill_pick,
    Values = {"Z", "X", "C", "V"},
    Multi = true,
    Default = {"Z", "X", "C", "V"},
    Callback = function(selectedSkills)
        getgenv().HOANGHAI_Skills = { Z = false, X = false, C = false, V = false }
        for skillName, isEnabled in pairs(selectedSkills) do
            if getgenv().HOANGHAI_Skills[skillName] ~= nil then
                getgenv().HOANGHAI_Skills[skillName] = isEnabled
            end
        end
    end
})

-- Tab Bán Cá (Auto Sell)
Tabs.Sell:AddToggle("HOANGHAI_AutoSell", {
    Title = T.autosell,
    Default = false,
    Callback = function(value)
        getgenv().HOANGHAI_AutoSell = value
    end
})

Tabs.Sell:AddSlider("HOANGHAI_SellDelay", {
    Title = T.selldelay,
    Default = 5,
    Min = 1,
    Max = 60,
    Rounding = 1,
    Callback = function(value)
        getgenv().HOANGHAI_SellDelay = value
    end
})

-- Hàm xử lý dịch chuyển đến vị trí (Teleport)
local function TeleportTo(targetObj)
    local char = LocalPlayer.Character
    local rootPart = char and char:FindFirstChild("HumanoidRootPart")
    
    if not rootPart then
        v1:Notify({ Title = "❌", Content = T.no_char,Duration = 3 })
        return
    end
    
    local success, err = pcall(function()
        local part
        if targetObj:IsA("BasePart") then
            part = targetObj
        else
            part = targetObj:FindFirstChildWhichIsA("BasePart", true)
        end
        
        if part then
            rootPart.CFrame = part.CFrame + Vector3.new(0, 25, 0)
        else
            error("no BasePart")
        end
    end)
    
    if success then
        v1:Notify({ Title = "✅", Content = T.tp_ok, Duration = 2 })
    else
        v1:Notify({ Title = "❌ " .. T.tp_fail, Content = tostring(err), Duration = 4 })
    end
end

-- Danh sách các đảo để dịch chuyển (Teleport Destinations)
local islands = {
    {n = 1, fn = function() return workspace.Map["Map 1"].House1; end},
    {n = 2, fn = function() return workspace.Map["Map 2"].House2; end},
    {n = 3, fn = function() return workspace.Map["Map 3"]:GetChildren()[11]; end},
    {n = 4, fn = function() return workspace.Map["Map 4"].Model; end},
    {n = 5, fn = function() return workspace.Map["Map 5"].Bait; end},
    {n = 6, fn = function() return workspace.Map["Map 6"].Folder:GetChildren()[33]; end},
    {n = 7, fn = function() return workspace.Map["Map 7"].New:GetChildren()[72].Grass; end},
    {n = 8, fn = function() return workspace.Map["Map 8"]:GetChildren()[3]; end},
    {n = 9, fn = function() return workspace.Map["Map 9"].Folder:GetChildren()[33]; end}
}

for _, islandData in ipairs(islands) do
    Tabs.Teleport:AddButton({
        Title = "🏝 " .. T.island .. " " .. islandData.n,
        Callback = function()
            pcall(function()
                TeleportTo(islandData.fn())
            end)
        end
    })
end

-- Cấu hình lưu trữ Config và Giao diện trong Tab Settings
if v2 then
    v2:SetLibrary(v1)
    v2:IgnoreThemeSettings()
    v2:SetIgnoreIndexes({})
    v2:SetFolder("HOANGHAIHub/Fishing")
    v2:BuildConfigSection(Tabs.Settings)
end

if v3 then
    v3:SetLibrary(v1)
    v3:SetFolder("HOANGHAIHub")
    v3:BuildInterfaceSection(Tabs.Settings)
end

-- [[ TÍNH NĂNG TỰ ĐỘNG (LOOPS) ]] --

-- 1. Tự động quăng cần câu (Auto Cast)
task.spawn(function()
    while task.wait(1) do
        if getgenv().HOANGHAI_AutoCast then
            pcall(function()
                local char = LocalPlayer.Character
                if char and not char:GetAttribute("Fishing") and not MainGui.Fishing.Visible then
                    ReplicatedStorage.Events.Fishing:FireServer()
                end
            end)
        end
    end
end)

-- 2. Tự động bán cá (Auto Sell)
task.spawn(function()
    while true do
        task.wait(getgenv().HOANGHAI_SellDelay)
        if getgenv().HOANGHAI_AutoSell then
            pcall(function()
                ReplicatedStorage.Events.SellFish:FireServer("All")
            end)
        end
    end
end)

-- 3. Tự động bấm kỹ năng (Auto Skill Z, X, C, V)
task.spawn(function()
    local keyCodes = { Z = Enum.KeyCode.Z, X = Enum.KeyCode.X, C = Enum.KeyCode.C, V = Enum.KeyCode.V }
    while task.wait(0.5) do
        if getgenv().HOANGHAI_AutoSkill then
            for skillName, keyCode in pairs(keyCodes) do
                if getgenv().HOANGHAI_Skills[skillName] then
                    VirtualInputManager:SendKeyEvent(true, keyCode, false, game)
                    task.wait(0.1)
                    VirtualInputManager:SendKeyEvent(false, keyCode, false, game)
                end
            end
        end
    end
end)

-- 4. Khóa thanh câu cá (Anchor / Lock Bar)
RunService.RenderStepped:Connect(function()
    if getgenv().HOANGHAI_Anchor then
        pcall(function()
            local fishingGui = MainGui.Fishing
            if fishingGui.Visible then
                local bar = fishingGui.BarFrame.Bar
                bar.Position = UDim2.new(0.5, 0, bar.Position.Y.Scale, 0)
                ReplicatedStorage.Fishing:FireServer("1")
            end
        end)
    end
end)

-- Hoàn tất tải script
Window:SelectTab(1)
v1:Notify({
    Title = "🍯 HOANGHAI Hub",
    Content = T.loaded,
    Duration = 5
})

if v2 then
    v2:LoadAutoloadConfig()
end
