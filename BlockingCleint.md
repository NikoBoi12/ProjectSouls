```lua
local UserInput = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

local PlayerData = require(ReplicatedStorage.PlayerData)
local EasingStyle = require(ReplicatedStorage.Modules.NikoModules.EasingStyle)
local TweenGoals = require(ReplicatedStorage.Modules.NikoModules.TweenGoals)
local Utility = require(ReplicatedStorage.Modules.NikoModules.Utility)
local VFX = require(ReplicatedStorage.Modules.NikoModules.VFX)
local PlayerData = require(ReplicatedStorage.PlayerData)
local TimerClass = require(ReplicatedStorage.Modules.Classes.Timer)

local BlockingRemote = ReplicatedStorage.Remotes.Blocking
local PerfectBlockingRemote = ReplicatedStorage.Remotes.PerfectBlocking

local LocalPlayer = Players.LocalPlayer
local Character = LocalPlayer.Character
local Humanoid = Character:FindFirstChild("Humanoid")
local HRP = Character:FindFirstChild("HumanoidRootPart")

local Data = PlayerData.GetData(LocalPlayer)

local Highlighter = ReplicatedStorage.Storage.Misc.Blocking

local Blocking = false

local Time = os.clock()

local CurrentBlock = nil


local HighlighterProp = {
	["PerfectBlock"] = {
		["Name"] = "PerfectBlock",
		["FillColor"] = Color3.fromRGB(255, 255, 255),
		["OutlineColor"] = Color3.fromRGB(255, 255, 255),
		["FillTransparency"] = 0.2,
		["OutlineTransparency"] = 1
	},
	["Blocking"] = {
		["Name"] = "Blocking",
		["FillColor"] = Color3.fromRGB(255, 0 ,0),
		["OutlineColor"] = Color3.fromRGB(255, 0 ,0),
		["FillTransparency"] = 0.5,
		["OutlineTransparency"] = 1
	},
	
}


local function Block(Input, GameService)
	if GameService then
		return
	end
	
	if Input.KeyCode == Enum.KeyCode.F and Blocking == false and Humanoid.WalkSpeed >= 16 then
		print("Blocking")
		
		Blocking = true
		BlockingRemote:FireServer()
		
		local Hightlighted = Highlighter:Clone()
		for Property, Value in HighlighterProp["Blocking"] do
			Hightlighted[Property] = Value
		end
		Hightlighted.Parent = Character
		CurrentBlock = Hightlighted
	end
end

UserInput.InputBegan:Connect(Block)


local function UnBlock(Input, GameService)
	if GameService then
		return
	end
	
	if Input.KeyCode == Enum.KeyCode.F and Blocking == true then
		BlockingRemote:FireServer()
	end
end

UserInput.InputEnded:Connect(UnBlock)


local function Unblocked()
	CurrentBlock:Destroy()
	CurrentBlock = nil
	Blocking =  false
end

BlockingRemote.OnClientEvent:Connect(Unblocked)


local function Flicker(Player)
	local Hightlighted = Highlighter:Clone()
	Hightlighted.Enabled = false
	for Property, Value in HighlighterProp["PerfectBlock"] do
		Hightlighted[Property] = Value
	end
	Hightlighted.Parent = Character
	
	local x = 0
	
	while true do
		if x == 4 then
			Hightlighted:Destroy()
			break
		end
		if Hightlighted.Enabled == true then
			Hightlighted.Enabled = false
		else
			Hightlighted.Enabled = true
		end
		
		x += 1
		task.wait(.05)
		end
end

PerfectBlockingRemote.OnClientEvent:Connect(Flicker)

```
