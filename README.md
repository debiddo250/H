local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local VirtualInputManager = game:GetService("VirtualInputManager")

local AutoFarm = false
local AutoRaid = false
local AutoSkill = false
local SafeFarm = true
local AutoFarmTarget = nil
local ESPEnabled = true
local Interface = Instance.new("ScreenGui", game.CoreGui)
local MainFrame = Instance.new("Frame", Interface)
local AutoFarmBtn = Instance.new("TextButton", MainFrame)
local AutoRaidBtn = Instance.new("TextButton", MainFrame)
local AutoSkillBtn = Instance.new("TextButton", MainFrame)

MainFrame.Size = UDim2.new(0, 200, 0, 150)
MainFrame.Position = UDim2.new(0, 20, 0, 20)
MainFrame.BackgroundColor3 = Color3.fromRGB(30,30,30)

local function Toggle(button, var)
    var = not var
    button.Text = button.Name .. ": " .. tostring(var)
    return var
end

AutoFarmBtn.Name = "AutoFarm"
AutoFarmBtn.Size = UDim2.new(0, 180, 0, 30)
AutoFarmBtn.Position = UDim2.new(0, 10, 0, 10)
AutoFarmBtn.Text = "AutoFarm: false"
AutoFarmBtn.MouseButton1Click:Connect(function()
    AutoFarm = Toggle(AutoFarmBtn, AutoFarm)
end)

AutoRaidBtn.Name = "AutoRaid"
AutoRaidBtn.Size = UDim2.new(0, 180, 0, 30)
AutoRaidBtn.Position = UDim2.new(0, 10, 0, 50)
AutoRaidBtn.Text = "AutoRaid: false"
AutoRaidBtn.MouseButton1Click:Connect(function()
    AutoRaid = Toggle(AutoRaidBtn, AutoRaid)
end)

AutoSkillBtn.Name = "AutoSkill"
AutoSkillBtn.Size = UDim2.new(0, 180, 0, 30)
AutoSkillBtn.Position = UDim2.new(0, 10, 0, 90)
AutoSkillBtn.Text = "AutoSkill: false"
AutoSkillBtn.MouseButton1Click:Connect(function()
    AutoSkill = Toggle(AutoSkillBtn, AutoSkill)
end)

local function AutoUseSkill()
    for _, key in ipairs({"Z", "X", "C", "V", "F"}) do
        VirtualInputManager:SendKeyEvent(true, Enum.KeyCode[key], false, game)
        task.wait(0.1)
        VirtualInputManager:SendKeyEvent(false, Enum.KeyCode[key], false, game)
    end
end

-- AutoFarm loop
spawn(function()
    while task.wait(0.1) do
        if AutoFarm and AutoFarmTarget and AutoFarmTarget:FindFirstChild("HumanoidRootPart") and AutoFarmTarget:FindFirstChild("Humanoid") and AutoFarmTarget.Humanoid.Health > 0 then
            pcall(function()
                local distance = SafeFarm and 20 or 3
                LocalPlayer.Character.HumanoidRootPart.CFrame = AutoFarmTarget.HumanoidRootPart.CFrame * CFrame.new(0, distance, 0)
                LocalPlayer.Character:FindFirstChildOfClass("Humanoid"):ChangeState(3)
                for _,tool in pairs(LocalPlayer.Backpack:GetChildren()) do
                    if tool:IsA("Tool") then
                        tool.Parent = LocalPlayer.Character
                        tool:Activate()
                    end
                end
                if AutoSkill then
                    AutoUseSkill()
                end
            end)
        end
    end
end)

-- AutoRaid loop
spawn(function()
    while task.wait(5) do
        if AutoRaid then
            pcall(function()
                local raidRoom = workspace:FindFirstChild("RaidEntrance")
                if raidRoom then
                    LocalPlayer.Character.HumanoidRootPart.CFrame = raidRoom.CFrame
                    if ReplicatedStorage:FindFirstChild("Remotes") and ReplicatedStorage.Remotes:FindFirstChild("EnterRaid") then
                        ReplicatedStorage.Remotes.EnterRaid:FireServer()
                    end
                end
            end)
        end
    end
end)

-- Detect Raça
local function DetectRace()
    local race = "Desconhecida"
    pcall(function()
        race = LocalPlayer.Data.Race.Value
    end)
    print("Sua Raça: "..race)
end
DetectRace()

-- ESP (básico, mostra inimigos)
RunService.RenderStepped:Connect(function()
    if ESPEnabled then
        for _,npc in pairs(workspace:FindFirstChild("Enemies") and workspace.Enemies:GetChildren() or {}) do
            if npc:FindFirstChild("HumanoidRootPart") and npc:FindFirstChild("Humanoid") and npc.Humanoid.Health > 0 then
                AutoFarmTarget = npc
                break
            end
        end
    end
end)

print("SCRIPT COMPLETO BLOX FRUITS — Safe Farm + AutoSkill + Interface — Carregado com sucesso!")
