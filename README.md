local Players    = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS        = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui   = LocalPlayer:WaitForChild("PlayerGui")

pcall(function()
    for _, name in ipairs({"Editor", "PosTracker"}) do
        local old = PlayerGui:FindFirstChild(name)
        if old then old:Destroy() end
    end
end)

local function copyToClipboard(text)
    text = tostring(text)
    if setclipboard then setclipboard(text); return true
    elseif toclipboard then toclipboard(text); return true end
    return false
end

local W, H_COLLAPSED, H_EXPANDED = 170, 62, 236

-- ═══════════════ Palette ═══════════════
local C_BG      = Color3.fromRGB(30, 30, 36)
local C_HEADER  = Color3.fromRGB(44, 44, 52)
local C_BTN     = Color3.fromRGB(58, 58, 68)
local C_BTN_HOV = Color3.fromRGB(74, 74, 86)
local C_TEXT    = Color3.fromRGB(230, 230, 236)
local C_DIM     = Color3.fromRGB(150, 150, 160)
local C_ACCENT  = Color3.fromRGB(100, 200, 150)
local C_DANGER  = Color3.fromRGB(190, 70, 70)
local C_KILL    = Color3.fromRGB(140, 45, 45)
local C_KILL_HOV= Color3.fromRGB(170, 60, 60)

local function corner(obj, r)
    local c = Instance.new("UICorner", obj)
    c.CornerRadius = UDim.new(0, r or 4)
    return c
end

local function makeButton(parent, text, size, pos, color, textSize, hoverColor)
    local b = Instance.new("TextButton")
    b.Size        = size
    b.Position    = pos
    b.BackgroundColor3 = color or C_BTN
    b.BorderSizePixel  = 0
    b.Text        = text
    b.TextColor3  = C_TEXT
    b.Font        = Enum.Font.GothamMedium
    b.TextSize    = textSize or 10
    b.AutoButtonColor = false
    b.Parent      = parent
    corner(b, 4)

    b.MouseEnter:Connect(function() b.BackgroundColor3 = hoverColor or C_BTN_HOV end)
    b.MouseLeave:Connect(function() b.BackgroundColor3 = color or C_BTN end)
    return b
end

-- ═══════════════ ScreenGui ═══════════════
local gui = Instance.new("ScreenGui")
gui.Name = "Editor"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.DisplayOrder = 999
gui.Parent = PlayerGui

-- ═══════════════ Toggle button (nút nổi) ═══════════════
local TOGGLE_SIZE = 30

local toggle = Instance.new("TextButton")
toggle.Size = UDim2.fromOffset(TOGGLE_SIZE, TOGGLE_SIZE)
toggle.Position = UDim2.new(0, 20, 0.5, -TOGGLE_SIZE/2)
toggle.BackgroundColor3 = C_HEADER
toggle.BorderSizePixel = 0
toggle.Text = "E"
toggle.TextColor3 = C_TEXT
toggle.Font = Enum.Font.GothamBold
toggle.TextSize = 13
toggle.AutoButtonColor = false
toggle.Parent = gui
corner(toggle, 6)

-- ═══════════════ Main frame ═══════════════
local main = Instance.new("Frame")
main.Size = UDim2.fromOffset(W, H_COLLAPSED)
main.Position = UDim2.new(0.5, -W/2, 0.5, -H_COLLAPSED/2)
main.BackgroundColor3 = C_BG
main.BorderSizePixel = 0
main.Parent = gui
corner(main, 6)

-- Header
local header = Instance.new("Frame")
header.Size = UDim2.new(1, 0, 0, 22)
header.BackgroundColor3 = C_HEADER
header.BorderSizePixel = 0
header.Parent = main
corner(header, 6)
for _, x in ipairs({0, 1}) do
    local p = Instance.new("Frame")
    p.Size = UDim2.fromOffset(6, 6)
    p.Position = UDim2.new(x, (x == 0) and 0 or -6, 1, -6)
    p.BackgroundColor3 = C_HEADER
    p.BorderSizePixel = 0
    p.ZIndex = 2
    p.Parent = header
end

-- Title (thu hẹp để chừa chỗ cho 3 nút)
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -120, 1, 0)
title.Position = UDim2.fromOffset(8, 0)
title.BackgroundTransparency = 1
title.Text = "Editor"
title.TextColor3 = C_TEXT
title.Font = Enum.Font.GothamBold
title.TextSize = 11
title.TextXAlignment = Enum.TextXAlignment.Left
title.ZIndex = 3
title.Parent = header

-- ── Header buttons ──────────────────────────────
-- Từ mép phải tính vào (W = 170):
--   ▾  : 1, -108  → chiếm pixel  62..78
--   Xóa: 1,  -84  → chiếm pixel  86..138  (cách × 10px)
--   ×  : 1,  -22  → chiếm pixel 148..164
local toggleBtn = makeButton(header, "▾", UDim2.fromOffset(16, 14), UDim2.new(1, -108, 0, 4), C_BTN, 10)
toggleBtn.ZIndex = 3

local killBtn = makeButton(header, "Xóa tất cả", UDim2.fromOffset(52, 14), UDim2.new(1, -84, 0, 4), C_KILL, 8, C_KILL_HOV)
killBtn.ZIndex = 3

local closeBtn = makeButton(header, "×", UDim2.fromOffset(16, 14), UDim2.new(1, -22, 0, 4), C_DANGER, 11)
closeBtn.ZIndex = 3

-- Pos display
local posDisplay = Instance.new("TextLabel")
posDisplay.Size = UDim2.new(1, -8, 0, 18)
posDisplay.Position = UDim2.fromOffset(4, 24)
posDisplay.BackgroundColor3 = C_HEADER
posDisplay.Text = "X:—  Y:—  Z:—"
posDisplay.TextColor3 = C_ACCENT
posDisplay.Font = Enum.Font.Code
posDisplay.TextSize = 10
posDisplay.Parent = main
corner(posDisplay, 4)

-- Buttons row
local recordBtn  = makeButton(main, "Save", UDim2.new(0.4,  -5, 0, 16), UDim2.fromOffset(4, 44),  C_BTN, 9)
local copyCurBtn = makeButton(main, "Copy", UDim2.new(0.28, -3, 0, 16), UDim2.new(0.4, 3, 0, 44),  C_BTN, 9)
local shopBtn    = makeButton(main, "Shop", UDim2.new(0.32, -8, 0, 16), UDim2.new(0.68, 4, 0, 44), C_BTN, 9)

-- List title + clear
local listTitle = Instance.new("TextLabel")
listTitle.Size = UDim2.new(1, -44, 0, 12)
listTitle.Position = UDim2.fromOffset(6, 62)
listTitle.BackgroundTransparency = 1
listTitle.Text = "Saved (0)"
listTitle.TextColor3 = C_DIM
listTitle.Font = Enum.Font.GothamMedium
listTitle.TextSize = 9
listTitle.TextXAlignment = Enum.TextXAlignment.Left
listTitle.Visible = false
listTitle.Parent = main

local clearBtn = makeButton(main, "Clear", UDim2.fromOffset(34, 12), UDim2.new(1, -40, 0, 62), C_DANGER, 8)
clearBtn.Visible = false

-- List
local list = Instance.new("ScrollingFrame")
list.Size = UDim2.new(1, -8, 0, H_EXPANDED - 80)
list.Position = UDim2.fromOffset(4, 76)
list.BackgroundColor3 = Color3.fromRGB(22, 22, 28)
list.BorderSizePixel = 0
list.ScrollBarThickness = 3
list.AutomaticCanvasSize = Enum.AutomaticSize.Y
list.ScrollingDirection = Enum.ScrollingDirection.Y
list.Visible = false
list.Parent = main
corner(list, 4)

local layout = Instance.new("UIListLayout")
layout.Padding = UDim.new(0, 2)
layout.SortOrder = Enum.SortOrder.LayoutOrder
layout.Parent = list

-- ═══════════════ Drag helpers ═══════════════
local function makeDraggable(frame, dragHandle)
    dragHandle = dragHandle or frame
    local dragging, dragStart, startPos

    dragHandle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = frame.Position
        end
    end)

    local c1 = UIS.InputChanged:Connect(function(input)
        if not dragging then return end
        if input.UserInputType ~= Enum.UserInputType.MouseMovement
        and input.UserInputType ~= Enum.UserInputType.Touch then return end
        local d = input.Position - dragStart
        frame.Position = UDim2.new(
            startPos.X.Scale, startPos.X.Offset + d.X,
            startPos.Y.Scale, startPos.Y.Offset + d.Y
        )
    end)

    local c2 = UIS.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)
    return c1, c2
end

-- ═══════════════ Expand / Collapse ═══════════════
local expanded = false
local function setExpanded(v)
    expanded = v
    toggleBtn.Text = v and "▴" or "▾"
    listTitle.Visible = v
    clearBtn.Visible = v
    list.Visible = v
    main.Size = UDim2.fromOffset(W, v and H_EXPANDED or H_COLLAPSED)

    local cam = workspace.CurrentCamera
    if cam then
        local vp = cam.ViewportSize
        local h = v and H_EXPANDED or H_COLLAPSED
        if main.Position.Y.Offset + h > vp.Y then
            main.Position = UDim2.new(
                main.Position.X.Scale, main.Position.X.Offset,
                main.Position.Y.Scale, math.max(0, vp.Y - h - 4)
            )
        end
    end
end
toggleBtn.MouseButton1Click:Connect(function() setExpanded(not expanded) end)

-- ═══════════════ Toggle button (click vs drag) ═══════════════
local toggleStartPos, toggleStartInput, toggleMoved

toggle.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then
        toggleStartPos   = toggle.Position
        toggleStartInput = input.Position
        toggleMoved      = false
    end
end)

local conns = {}
local function track(c) table.insert(conns, c); return c end
local function cleanup()
    for _, c in ipairs(conns) do
        if typeof(c) == "RBXScriptConnection" and c.Connected then c:Disconnect() end
    end
    table.clear(conns)
end

track(UIS.InputChanged:Connect(function(input)
    if not toggleStartPos then return end
    if input.UserInputType ~= Enum.UserInputType.MouseMovement
    and input.UserInputType ~= Enum.UserInputType.Touch then return end
    local d = input.Position - toggleStartInput
    if math.abs(d.X) + math.abs(d.Y) > 4 then toggleMoved = true end
    if toggleMoved then
        toggle.Position = UDim2.new(
            toggleStartPos.X.Scale, toggleStartPos.X.Offset + d.X,
            toggleStartPos.Y.Scale, toggleStartPos.Y.Offset + d.Y
        )
    end
end))

track(UIS.InputEnded:Connect(function(input)
    if input.UserInputType ~= Enum.UserInputType.MouseButton1
    and input.UserInputType ~= Enum.UserInputType.Touch then return end
    if not toggleStartPos then return end
    local wasClick = not toggleMoved
    toggleStartPos, toggleStartInput, toggleMoved = nil, nil, nil
    if wasClick then
        main.Visible = not main.Visible
    end
end))

local dm1, dm2 = makeDraggable(main, header)
track(dm1); track(dm2)

-- ═══════════════ Logic ═══════════════
local lastPos = nil
local records = {}
local mode    = "records"

local function getPos()
    local char = LocalPlayer.Character
    if char then
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if hrp then return hrp.Position end
    end
end

local acc = 0
track(RunService.Heartbeat:Connect(function(dt)
    acc += dt
    if acc < 0.05 then return end
    acc = 0
    local pos = getPos()
    if pos then
        lastPos = pos
        posDisplay.Text = string.format("X:%.1f  Y:%.1f  Z:%.1f", pos.X, pos.Y, pos.Z)
    else
        lastPos = nil
        posDisplay.Text = "X:—  Y:—  Z:—"
    end
end))

local flashTimers = {}
local function flash(btn, good, dur)
    if flashTimers[btn] then task.cancel(flashTimers[btn]) end
    local base = btn.BackgroundColor3
    btn.BackgroundColor3 = good and Color3.fromRGB(60, 150, 100) or Color3.fromRGB(150, 90, 60)
    flashTimers[btn] = task.delay(dur or 0.5, function()
        flashTimers[btn] = nil
        if btn and btn.Parent then btn.BackgroundColor3 = base end
    end)
end

local function clearList()
    for _, c in ipairs(list:GetChildren()) do
        if c:IsA("Frame") then c:Destroy() end
    end
end

local function makeRow(labelText, pos, nameColor)
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1, 0, 0, 20)
    row.BackgroundColor3 = Color3.fromRGB(42, 42, 52)
    row.BorderSizePixel = 0
    row.Parent = list
    corner(row, 3)

    local posLbl = Instance.new("TextLabel")
    posLbl.Size = UDim2.new(1, -72, 1, 0)
    posLbl.Position = UDim2.fromOffset(6, 0)
    posLbl.BackgroundTransparency = 1
    posLbl.Text = labelText
    posLbl.TextColor3 = nameColor or C_ACCENT
    posLbl.Font = Enum.Font.Code
    posLbl.TextSize = 9
    posLbl.TextXAlignment = Enum.TextXAlignment.Left
    posLbl.TextTruncate = Enum.TextTruncate.AtEnd
    posLbl.Parent = row

    local tpBtn = makeButton(row, "TP", UDim2.fromOffset(28, 16), UDim2.new(1, -68, 0, 2), C_BTN, 8)
    local cpBtn = makeButton(row, "Copy", UDim2.fromOffset(34, 16), UDim2.new(1, -38, 0, 2), C_BTN, 8)

    tpBtn.MouseButton1Click:Connect(function()
        local char = LocalPlayer.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            char:PivotTo(CFrame.new(pos + Vector3.new(0, 3, 0)))
            char.HumanoidRootPart.AssemblyLinearVelocity = Vector3.new(0,0,0)
            flash(tpBtn, true, 0.4)
        end
    end)

    cpBtn.MouseButton1Click:Connect(function()
        local s = string.format("%.2f, %.2f, %.2f", pos.X, pos.Y, pos.Z)
        flash(cpBtn, copyToClipboard(s), 0.5)
    end)
end

-- Records
local function updateCount()
    if mode == "records" then
        listTitle.Text = string.format("Saved (%d)", #records)
    end
end

local function renderRecords()
    clearList()
    for _, pos in ipairs(records) do
        makeRow(string.format("%.1f, %.1f, %.1f", pos.X, pos.Y, pos.Z), pos)
    end
    updateCount()
end

-- Shops
local SHOP_KEYWORDS = {"shop","store","vendor","merchant","seller","market","buy","npc","trade"}

local function isShopName(name)
    local lname = string.lower(name)
    for _, kw in ipairs(SHOP_KEYWORDS) do
        if string.find(lname, kw, 1, true) then return true end
    end
    return false
end

local function getObjPos(obj)
    if obj:IsA("BasePart") then return obj.Position end
    if obj:IsA("Model") then
        local ok, cf = pcall(function() return obj:GetPivot() end)
        if ok and cf then return cf.Position end
        local pp = obj.PrimaryPart
        if pp then return pp.Position end
    end
    return nil
end

local function findShops()
    local found, seen = {}, {}
    local myPos = lastPos or Vector3.zero
    local function add(obj, displayName)
        local pos = getObjPos(obj)
        if not pos then return end
        local key = math.floor(pos.X/3)..","..math.floor(pos.Y/3)..","..math.floor(pos.Z/3)
        if seen[key] then return end
        seen[key] = true
        table.insert(found, { name = displayName or obj.Name, pos = pos, dist = (pos - myPos).Magnitude })
    end
    local ok, descendants = pcall(function() return workspace:GetDescendants() end)
    if not ok then return found end
    for _, obj in ipairs(descendants) do
        if obj:IsA("Model") or obj:IsA("BasePart") then
            if isShopName(obj.Name) then add(obj, obj.Name) end
        elseif obj:IsA("ProximityPrompt") then
            local parent = obj.Parent
            if parent and (parent:IsA("BasePart") or parent:IsA("Model")) then
                local name = (obj.ActionText ~= "" and obj.ActionText) or parent.Name
                if isShopName(name) or isShopName(parent.Name) then add(parent, name) end
            end
        end
    end
    table.sort(found, function(a, b) return a.dist < b.dist end)
    return found
end

local function renderShops()
    clearList()
    local shops = findShops()
    for _, s in ipairs(shops) do
        makeRow(string.format("[%d] %s", math.floor(s.dist), s.name), s.pos, Color3.fromRGB(220, 190, 130))
    end
    listTitle.Text = string.format("Shops (%d)", #shops)
    return #shops
end

-- Mode switch
shopBtn.MouseButton1Click:Connect(function()
    if mode == "records" then
        mode = "shops"
        if not expanded then setExpanded(true) end
        shopBtn.Text = "Saved"
        clearBtn.Text = "Rescan"
        renderShops()
    else
        mode = "records"
        shopBtn.Text = "Shop"
        clearBtn.Text = "Clear"
        renderRecords()
    end
end)

-- Actions
copyCurBtn.MouseButton1Click:Connect(function()
    if lastPos then
        local s = string.format("%.2f, %.2f, %.2f", lastPos.X, lastPos.Y, lastPos.Z)
        flash(copyCurBtn, copyToClipboard(s), 0.6)
    end
end)

recordBtn.MouseButton1Click:Connect(function()
    if lastPos then
        table.insert(records, lastPos)
        if mode == "records" then renderRecords() else updateCount() end
        flash(recordBtn, true, 0.5)
    end
end)

clearBtn.MouseButton1Click:Connect(function()
    if mode == "shops" then
        renderShops()
        flash(clearBtn, true, 0.4)
    else
        records = {}
        renderRecords()
        flash(clearBtn, true, 0.5)
    end
end)

-- ═══════════════ × CHỈ ẨN MENU ═══════════════
closeBtn.MouseButton1Click:Connect(function()
    main.Visible = false
end)

-- ═══════════════ "Xóa tất cả" — XOÁ TOÀN BỘ ═══════════════
killBtn.MouseButton1Click:Connect(function()
    cleanup()
    gui:Destroy()
end)

gui.Destroying:Connect(cleanup)

print("[Editor] loaded — × ẩn menu, 'Xóa tất cả' xoá toàn bộ")
