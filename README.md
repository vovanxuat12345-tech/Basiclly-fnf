-- [[ MADE BY CHIẾN ĐO ]]
-- Script Autoplayer Basically FNF - RenderStepped Ultimate Speed Version
-- Scans per frame cycle (FPS) - Completely counters high-tempo songs

local RunService = game:GetService("RunService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- ULTRA SPEED CONFIGURATION
local AutoplayerEnabled = true
local HitThreshold = 35 -- Wide scan radius to catch ultra-fast falling notes

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

-- Array to store RenderStepped connections for easy management/disconnection
local Connections = {}

local function startUltraSpeedThread(arrowName, keyCode)
    -- Disconnect old connection if exists to avoid overlapping threads causing lag
    if Connections[arrowName] then
        Connections[arrowName]:Disconnect()
    end
    
    -- Uses RenderStepped running directly on Roblox's frame rate processing clock
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
                        
                        -- High speed logic: process immediately when note hits the threshold edge
                        if diff <= HitThreshold then
                            note:SetAttribute("Processed", true)
                            
                            -- Force hit: releases old key and presses new key with 0ms delay
                            VirtualInputManager:SendKeyEvent(false, keyCode, false, game)
                            VirtualInputManager:SendKeyEvent(true, keyCode, false, game)
                        end
                    end
                end
                
            end
        end
    end)
end

-- ACTIVATE FRAME-BY-FRAME SCANNING FOR 4 BUTTONS
startUltraSpeedThread("Arrow1", KeyMapping["Arrow1"])
startUltraSpeedThread("Arrow2", KeyMapping["Arrow2"])
startUltraSpeedThread("Arrow3", KeyMapping["Arrow3"])
startUltraSpeedThread("Arrow4", KeyMapping["Arrow4"])


-- [[ MENU INTERFACE - BLACK & RED THEME ]]
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
SwitchSideBtn.Text = "Playing: Left Side"
SwitchSideBtn.TextColor3 = Color3.fromRGB(240, 240, 240)

SwitchSideBtn.MouseButton1Click:Connect(function()
    if currentSide == "KeySync1" then
        currentSide = "KeySync2"
        SwitchSideBtn.Text = "Playing: Right Side"
    else
        currentSide = "KeySync1"
        SwitchSideBtn.Text = "Playing: Left Side"
    end
end)
