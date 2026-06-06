-- [[ MADE BY CHIẾN ĐO ]]
-- Script Autoplayer Basically FNF - RenderStepped Ultimate Speed Version
-- Quét theo chu kỳ khung hình (FPS) - Khắc chế hoàn toàn bài hát nhịp độ cao

local RunService = game:GetService("RunService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- CẤU HÌNH SIÊU TỐC ĐỘ
local AutoplayerEnabled = true
local HitThreshold = 35 -- Vùng quét rộng để bắt kịp các nốt rơi siêu nhanh

local KeyMapping = {
    ["Arrow1"] = Enum.KeyCode.A,
    ["Arrow2"] = Enum.KeyCode.S,
    ["Arrow3"] = Enum.KeyCode.W,
    ["Arrow4"] = Enum.KeyCode.D
}

local currentSide = "KeySync1"
local function getMyKeySync()
    local MatchFrame = LocalPlayer:WaitForChild("PlayerGui"):WaitForChild("Main"):WaitForChild("MatchFrame")
    return MatchFrame:FindFirstChild(currentSide)
end

print("Script loaded by CHIẾN ĐO - RenderStepped Mode Activated")

-- Mảng lưu trữ các kết nối RenderStepped để dễ quản lý hoặc hủy khi cần
local Connections = {}

local function startUltraSpeedThread(arrowName, keyCode)
    -- Hủy kết nối cũ nếu có để tránh trùng lặp luồng gây lag
    if Connections[arrowName] then
        Connections[arrowName]:Disconnect()
    end
    
    -- Sử dụng RenderStepped chạy trực tiếp trên xung nhịp xử lý khung hình của Roblox
    Connections[arrowName] = RunService.RenderStepped:Connect(function()
        if not AutoplayerEnabled then return end
        
        local MyKeySync = getMyKeySync()
        if MyKeySync then
            local arrowBase = MyKeySync:FindFirstChild(arrowName)
            if arrowBase and arrowBase:FindFirstChild("Notes") then
                
                local currentNotes = arrowBase.Notes:GetChildren()
                
                for _, note in ipairs(currentNotes) do
                    if (note:IsA("ImageLabel") or note:IsA("Frame")) and not note:GetAttribute("Processed") then
                        local noteY = note.AbsolutePosition.Y
                        local baseY = arrowBase.AbsolutePosition.Y
                        local diff = math.abs(noteY - baseY)
                        
                        -- Vì tốc độ cực cao, nốt vừa chạm rìa hồng tâm (diff <= HitThreshold) là xử lý ngay
                        if diff <= HitThreshold then
                            note:SetAttribute("Processed", true)
                            
                            -- Ép nhịp: Nhả phím cũ gối đầu phím mới không trễ 1 mili-giây
                            VirtualInputManager:SendKeyEvent(false, keyCode, false, game)
                            VirtualInputManager:SendKeyEvent(true, keyCode, false, game)
                        end
                    end
                end
                
            end
        end
    end)
end

-- KÍCH HOẠT QUÉT THEO KHUNG HÌNH CHO 4 NÚT
startUltraSpeedThread("Arrow1", KeyMapping["Arrow1"])
startUltraSpeedThread("Arrow2", KeyMapping["Arrow2"])
startUltraSpeedThread("Arrow3", KeyMapping["Arrow3"])
startUltraSpeedThread("Arrow4", KeyMapping["Arrow4"])


-- [[ GIAO DIỆN MENU CHUẨN GU ĐEN ĐỎ ]]
local ScreenGui = Instance.new("ScreenGui")
local MainFrame = Instance.new("Frame")
local Title = Instance.new("TextLabel")
local ToggleBtn = Instance.new("TextButton")
local SwitchSideBtn = Instance.new("TextButton")

ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
ScreenGui.ResetOnSpawn = false

MainFrame.Name = "ChienDoRenderMenu"
MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
MainFrame.Position = UDim2.new(0.05, 0, 0.35, 0)
MainFrame.Size = UDim2.new(0, 160, 0, 130)
MainFrame.Active = true
MainFrame.Draggable = true

Title.Parent = MainFrame
Title.BackgroundTransparency = 1
Title.Size = UDim2.new(1, 0, 0.3, 0)
Title.Text = "MADE BY CHIẾN ĐO"
Title.TextColor3 = Color3.fromRGB(255, 40, 40)
Title.TextSize = 13
Title.Font = Enum.Font.SourceSansBold

ToggleBtn.Parent = MainFrame
ToggleBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
ToggleBtn.Position = UDim2.new(0.1, 0, 0.35, 0)
ToggleBtn.Size = UDim2.new(0.8, 0, 0.25, 0)
ToggleBtn.Text = "Autoplayer: ON"
ToggleBtn.TextColor3 = Color3.fromRGB(0, 255, 100)

ToggleBtn.MouseButton1Click:Connect(function()
    AutoplayerEnabled = not AutoplayerEnabled
    if AutoplayerEnabled then
        ToggleBtn.Text = "Autoplayer: ON"
        ToggleBtn.TextColor3 = Color3.fromRGB(0, 255, 100)
    else
        ToggleBtn.Text = "Autoplayer: OFF"
        ToggleBtn.TextColor3 = Color3.fromRGB(255, 40, 40)
        for _, keyCode in pairs(KeyMapping) do
            VirtualInputManager:SendKeyEvent(false, keyCode, false, game)
        end
    end
end)

SwitchSideBtn.Parent = MainFrame
SwitchSideBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
SwitchSideBtn.Position = UDim2.new(0.1, 0, 0.65, 0)
SwitchSideBtn.Size = UDim2.new(0.8, 0, 0.25, 0)
SwitchSideBtn.Text = "Đang chơi: Bên Trái"
SwitchSideBtn.TextColor3 = Color3.fromRGB(240, 240, 240)

SwitchSideBtn.MouseButton1Click:Connect(function()
    if currentSide == "KeySync1" then
        currentSide = "KeySync2"
        SwitchSideBtn.Text = "Đang chơi: Bên Phải"
    else
        currentSide = "KeySync1"
        SwitchSideBtn.Text = "Đang chơi: Bên Trái"
    end
end)
