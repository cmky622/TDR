-- Rayfield GUI 高亮控制脚本
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- 创建窗口
local Window = Rayfield:CreateWindow({
    Name = "TDR透视",
    Icon = 0,
    LoadingTitle = "加载中...",
    LoadingSubtitle = "高亮功能初始化",
    Theme = "Default",
    ToggleUIKeybind = "K",
    ConfigurationSaving = {
        Enabled = false,
    }
})

-- 创建主标签页
local MainTab = Window:CreateTab("高亮控制", nil)
local MainSection = MainTab:CreateSection("高亮设置")

-- 高亮系统变量
local highlightSystem = {
    Enabled = false,
    TargetName = "你的组件名称",
    HighlightColor = Color3.new(1, 0, 0),
    ProcessedObjects = {}
}

-- 应用高亮效果到目标对象
local function applyHighlight(target)
    if highlightSystem.ProcessedObjects[target] then
        return
    end
    
    highlightSystem.ProcessedObjects[target] = true
    
    local existingHighlight = target:FindFirstChildOfClass("Highlight")
    if existingHighlight then
        existingHighlight:Destroy()
    end
    
    local highlight = Instance.new("Highlight")
    highlight.Name = "DynamicHighlight"
    highlight.FillColor = highlightSystem.HighlightColor
    highlight.OutlineColor = highlightSystem.HighlightColor
    highlight.FillTransparency = 0.5
    highlight.OutlineTransparency = 0
    highlight.Parent = target
    
    target.Destroying:Connect(function()
        highlightSystem.ProcessedObjects[target] = nil
    end)
end

-- 检查对象是否符合高亮条件
local function shouldHighlight(object)
    if string.find(object.Name:lower(), highlightSystem.TargetName:lower()) then
        if object:IsA("Model") or object:IsA("BasePart") then
            return true
        end
    end
    return false
end

-- 初始扫描
local function initialScan()
    for _, child in ipairs(workspace:GetChildren()) do
        if shouldHighlight(child) then
            applyHighlight(child)
        end
        
        if child:IsA("Model") then
            for _, descendant in ipairs(child:GetDescendants()) do
                if shouldHighlight(descendant) then
                    applyHighlight(descendant)
                end
            end
        end
    end
end

-- 设置监听器
local function setupListeners()
    -- 监听新对象的添加
    workspace.ChildAdded:Connect(function(child)
        wait(0.1)
        
        if shouldHighlight(child) then
            applyHighlight(child)
        end
        
        if child:IsA("Model") then
            for _, descendant in ipairs(child:GetDescendants()) do
                if shouldHighlight(descendant) then
                    applyHighlight(descendant)
                end
            end
            
            child.DescendantAdded:Connect(function(descendant)
                if shouldHighlight(descendant) then
                    applyHighlight(descendant)
                end
            end)
        end
    end)
end

-- 清除所有高亮效果
local function clearAllHighlights()
    for object, _ in pairs(highlightSystem.ProcessedObjects) do
        local highlight = object:FindFirstChildOfClass("Highlight")
        if highlight then
            highlight:Destroy()
        end
    end
    highlightSystem.ProcessedObjects = {}
end

-- 启动高亮系统
local function startHighlightSystem()
    if highlightSystem.Enabled then return end
    
    highlightSystem.Enabled = true
    initialScan()
    setupListeners()
    Rayfield:Notify({
        Title = "高亮系统",
        Content = "高亮功能已开启",
        Duration = 3,
    })
end

-- 停止高亮系统
local function stopHighlightSystem()
    if not highlightSystem.Enabled then return end
    
    highlightSystem.Enabled = false
    clearAllHighlights()
    Rayfield:Notify({
        Title = "高亮系统",
        Content = "高亮功能已关闭",
        Duration = 3,
    })
end

-- 创建开关控件
local Toggle = MainTab:CreateToggle({
    Name = "高亮功能开关",
    CurrentValue = false,
    Flag = "HighlightToggle",
    Callback = function(Value)
        if Value then
            startHighlightSystem()
        else
            stopHighlightSystem()
        end
    end,
})

-- 创建目标名称输入框
local Input = MainTab:CreateInput({
    Name = "目标名称",
    PlaceholderText = "输入要高亮的组件名称",
    RemoveTextAfterFocusLost = false,
    Callback = function(Text)
        highlightSystem.TargetName = Text
        if highlightSystem.Enabled then
            -- 重新扫描
            clearAllHighlights()
            initialScan()
        end
    end,
})

-- 创建颜色选择器
local ColorPicker = MainTab:CreateColorPicker({
    Name = "高亮颜色",
    Color = Color3.new(1, 0, 0),
    Flag = "HighlightColor",
    Callback = function(Color)
        highlightSystem.HighlightColor = Color
        if highlightSystem.Enabled then
            -- 更新所有已高亮对象的颜色
            for object, _ in pairs(highlightSystem.ProcessedObjects) do
                local highlight = object:FindFirstChildOfClass("Highlight")
                if highlight then
                    highlight.FillColor = Color
                    highlight.OutlineColor = Color
                end
            end
        end
    end
})

-- 创建按钮部分
local ButtonSection = MainTab:CreateSection("控制按钮")

-- 立即扫描按钮
local ScanButton = MainTab:CreateButton({
    Name = "立即扫描",
    Callback = function()
        if highlightSystem.Enabled then
            clearAllHighlights()
            initialScan()
            Rayfield:Notify({
                Title = "高亮系统",
                Content = "已完成重新扫描",
                Duration = 3,
            })
        else
            Rayfield:Notify({
                Title = "高亮系统",
                Content = "请先开启高亮功能",
                Duration = 3,
            })
        end
    end,
})

-- 清除所有高亮按钮
local ClearButton = MainTab:CreateButton({
    Name = "清除所有高亮",
    Callback = function()
        clearAllHighlights()
        Rayfield:Notify({
            Title = "高亮系统",
            Content = "已清除所有高亮效果",
            Duration = 3,
        })
    end,
})

-- 信息显示部分
local InfoSection = MainTab:CreateSection("系统信息")

-- 状态显示标签
local StatusLabel = MainTab:CreateLabel("当前状态: 高亮功能未开启")
local ObjectsLabel = MainTab:CreateLabel("已高亮对象: 0")

-- 更新状态显示的函数
local function updateStatusDisplay()
    StatusLabel:Set("当前状态: " .. (highlightSystem.Enabled and "高亮功能已开启" or "高亮功能未开启"))
    
    local count = 0
    for _ in pairs(highlightSystem.ProcessedObjects) do
        count = count + 1
    end
    ObjectsLabel:Set("已高亮对象: " .. count)
end

-- 定期更新状态显示
spawn(function()
    while true do
        updateStatusDisplay()
        wait(1)
    end
end)

Rayfield:Notify({
    Title = "高亮控制系统",
    Content = "系统加载完成！使用K键切换界面",
    Duration = 6,
})
