Unfinished decided not to finish this until more of the game is completed

```lua
local TweenService = game:GetService("TweenService")
local UserInput = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

local PlayerData = require(ReplicatedStorage.PlayerData)
local EasingStyle = require(ReplicatedStorage.Modules.NikoModules.EasingStyle)
local TweenGoals = require(ReplicatedStorage.Modules.NikoModules.TweenGoals)
local Utility = require(ReplicatedStorage.Modules.NikoModules.Utility)
local VFX = require(ReplicatedStorage.Modules.NikoModules.VFX)
local PlayerData = require(ReplicatedStorage.PlayerData)

local CharacterChosenRemote = ReplicatedStorage.Remotes.CharacterChosen

local LocalPlayer = Players.LocalPlayer
local Character = LocalPlayer.Character
local Humanoid = Character:FindFirstChild("Humanoid")
local HRP = Character:FindFirstChild("HumanoidRootPart")

local GuiStorage = ReplicatedStorage.Storage.GuiStorage
local PlayerGui = LocalPlayer.PlayerGui
local CombatGui = PlayerGui.CombatUI

local Data = PlayerData.GetData(LocalPlayer)

local Ability = {}

local function UIUpdate(Config, i)
	local x = 1
	for i, Ability in CombatGui.Abilities:GetChildren() do
		if Ability:IsA("Frame") then
			Ability:Destroy()
		end
	end
	
	for Index, Ability in Config["Abilities"][i] do
		local Frame = GuiStorage.Ability:Clone()
		
		Frame.Number.Text = x
		Frame.Parent = CombatGui.Abilities
		x += 1
	end
end


local function Abilities()
	local i = 1
	while Data.Character == nil do
		task.wait(1)
		warn("Yielding for selection!")
		i += 1
		if i == 5 then
			warn("Something went wrong!")
			return
		end
	end
	
	local MeleeClass = require(ReplicatedStorage.Modules.Classes.Melee:FindFirstChild(Data.Character))
	local CharacterConfig = require(ReplicatedStorage.Modules.Characters[Data.Character])[Data.Phase]
	local Config = CharacterConfig["Config"]
	
	local Animations = Config["Animations"]
	
	for i, Index in Animations do
		if typeof(Index) == "table" then
			for i, Animations in Index do
				Utility.AnimationLoad(Humanoid, Animations)
			end
			continue
		end
		Utility.AnimationLoad(Humanoid, Index)
	end
	
	UIUpdate(CharacterConfig, 1)
	
	if Config["Theme"] then
		Config["Theme"]:Play()
	end
	
	local Melee = MeleeClass.New()
	
	UserInput.InputBegan:Connect(function(Input, GameExecution)
		if GameExecution or Data.Character == nil then
			return
		end
		
		if Input.UserInputType == Enum.UserInputType.MouseButton1 then
			Melee:Swing()
		end
		
		local AbilityType = CharacterConfig["Abilities"]
		local InputData = AbilityType[i][Input.KeyCode]
		
		if InputData then
			InputData.Use()
		end
		
		if Input.KeyCode == Enum.KeyCode.E then
			i += 1
			if i > #AbilityType then
				i = 1
			end
			UIUpdate(CharacterConfig, i)
		elseif Input.KeyCode == Enum.KeyCode.Q then
			i -= 1
			if i < 1 then
				i = #AbilityType
			end
			UIUpdate(CharacterConfig, i)
		end
	end)
end


local function Menu(Player)
	local CharacterTemp = "Cross"
	--[[
		MENU CODE HERE
	]]

	--EVEN CONNECTION HERE
	
	
	
	CharacterChosenRemote:FireServer(CharacterTemp)
end

Menu()

local function Chosen()
	Abilities()
end

CharacterChosenRemote.OnClientEvent:Connect(Chosen)
```
