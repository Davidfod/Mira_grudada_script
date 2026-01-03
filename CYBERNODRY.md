local Camera = workspace.CurrentCamera
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Holding = false

_G.AimbotEnabled = true
_G.TeamCheck = false
_G.AimPart = "Head" 
_G.Sensitivity = 1 

local function GetClosestPlayer()
    local MaximumDistance = math.huge
    local Target = nil
    for _, v in pairs(Players:GetPlayers()) do
        if v ~= LocalPlayer and v.Character then
            local char = v.Character
            local targetPart = char:FindFirstChild(_G.AimPart) or char:FindFirstChild("HumanoidRootPart")
            local hum = char:FindFirstChildOfClass("Humanoid")
            if targetPart and hum and hum.Health > 0 then
                if not _G.TeamCheck or v.Team ~= LocalPlayer.Team then
                    local ScreenPoint, OnScreen = Camera:WorldToScreenPoint(targetPart.Position)
                    if OnScreen then
                        local MousePos = UserInputService:GetMouseLocation()
                        local Distance = (Vector2.new(ScreenPoint.X, ScreenPoint.Y) - MousePos).Magnitude
                        if Distance < MaximumDistance then
                            Target = v
                            MaximumDistance = Distance
                        end
                    end
                end
            end
        end
    end
    return Target
end

if UserInputService.TouchEnabled then
    local ScreenGui = Instance.new("ScreenGui")
    local MobileButton = Instance.new("TextButton")
    local UICorner = Instance.new("UICorner")

    ScreenGui.Parent = game:GetService("CoreGui")
    MobileButton.Name = "MobileAimbot"
    MobileButton.Parent = ScreenGui
    MobileButton.Size = UDim2.new(0, 75, 0, 75)
    MobileButton.Position = UDim2.new(0.75, 0, 0.4, 0)
    MobileButton.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    MobileButton.BackgroundTransparency = 0.3
    MobileButton.Text = "AIM"
    MobileButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    MobileButton.Font = Enum.Font.GothamBold
    MobileButton.TextSize = 20
    MobileButton.Active = true
    MobileButton.Draggable = true

    UICorner.CornerRadius = UDim.new(1, 0)
    UICorner.Parent = MobileButton

    MobileButton.MouseButton1Down:Connect(function() 
        Holding = true 
        MobileButton.BackgroundColor3 = Color3.fromRGB(0, 255, 100)
    end)
    MobileButton.MouseButton1Up:Connect(function() 
        Holding = false 
        MobileButton.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    end)
end

UserInputService.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton2 then
        Holding = true
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton2 then
        Holding = false
    end
end)

RunService.RenderStepped:Connect(function()
    if Holding and _G.AimbotEnabled then
        local Target = GetClosestPlayer()
        if Target and Target.Character then
            local Part = Target.Character:FindFirstChild(_G.AimPart) or Target.Character:FindFirstChild("HumanoidRootPart")
            if Part then
                local LookCFrame = CFrame.lookAt(Camera.CFrame.Position, Part.Position)
                Camera.CFrame = Camera.CFrame:Lerp(LookCFrame, _G.Sensitivity)
            end
        end
    end
end)
