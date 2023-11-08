2023-6-9

This is one of the many abilities I used for my game mobile support is pending this is used for the creation attack and movment of the projectiles aswell as the VFX This taught me a lot on code readability and organization of projects.

```lua
local BonesAbility = {}

if game:GetService("RunService"):IsServer() then
	return BonesAbility
end

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local Utility = require(ReplicatedStorage.Modules.NikoModules.Utility)
local VFX = require(ReplicatedStorage.Modules.NikoModules.VFX)
local PlayerData = require(ReplicatedStorage.PlayerData)
local StateManager = require(ReplicatedStorage.Modules.NikoModules.StateManager)
local AbilityConfig = require(script.Config)
local ProjectileClass = require(ReplicatedStorage.Modules.Classes.Projectile)

local EnemyHitRemote = ReplicatedStorage.Remotes.AbilityRemotes.EnemyHit
local VisualRemote = ReplicatedStorage.Remotes.AbilityVisualRemote.Ability1Visual
local Sync = ReplicatedStorage.Remotes.AbilityRemotes.SyncAbility1

local LocalPlayer = Players.LocalPlayer
local Character = LocalPlayer.Character
local Humanoid = Character:FindFirstChild("Humanoid")
local HRP = Character:FindFirstChild("HumanoidRootPart")

local Config = AbilityConfig["Client Config"]
local ProjectileModels = Config["Models"]
local ServerAbilityConfig = AbilityConfig["Server Configs"]

local Mouse = LocalPlayer:GetMouse()
local Data = PlayerData.GetData(LocalPlayer)

local ActiveCooldown = false

local Params = {}

BonesAbility.Use = function()
	if Data.Stunned == true or Data.KnockedDown == true or Data.InAction == true or ActiveCooldown == true or Data.Mana < Config["Mana Cost"] then
		return
	end
	
	ActiveCooldown = true
	
	Data.InAction = true
	Data.Mana -= Config["Mana Cost"]
	StateManager.ActionTaken(LocalPlayer, Humanoid, Config["Action Taken Timer"], Data)
	BonesAbility.Start()
	
	task.defer(function()
		task.wait(Config["InAction Timer"])
		Data.InAction = false
	end)
	
	task.wait(Config["Cooldown"])
	ActiveCooldown = false
end


BonesAbility.Start = function()
	local Max = 2147483647
	local SeedNum = math.random(Max)
	
	local Seed = Random.new(SeedNum)
	

	for i=1, Config["Number Of Projectiles"], 1 do
		local Bone = ProjectileModels["Projectile"]:Clone()
		local MainPart = Bone.Main
		
		local TargetPos = Utility.FindTarget(Data, Mouse)

		BonesAbility.SpawnAttack(Bone, Seed)
		BonesAbility.TweenEffect(MainPart, Seed)
		BonesAbility.StartAttack(MainPart)
	end
end

Sync.OnClientEvent:Connect(BonesAbility.Start)


BonesAbility.SpawnAttack = function(Bone, Seed)
	local MainPart = Bone.Main
	
	task.wait(Config["Delay Per Bone Spawn"])
	
	MainPart.CFrame = HRP.CFrame * CFrame.new(Seed:NextNumber(-5,-1), 0, Seed:NextNumber(3, 6))
	Bone.Parent = workspace
end

BonesAbility.TweenEffect = function(MainPart, Seed)
	local RandomDirection = Seed:NextUnitVector() * Seed:NextNumber(3,7)
	
	local NewCFrame = CFrame.lookAt(HRP.Position+RandomDirection + Vector3.new(Seed:NextNumber(0,2), 10, Seed:NextNumber(0,2)), Utility.FindTarget(Data, Mouse))
	
	
	
	local Magnitude = (NewCFrame.Position - MainPart.Position).Magnitude
	
	local Time = Magnitude/Config["Tween Speed"]

	if Time > Config["Start Up"] then
		Time = Config["Start Up"]
	end

	local Goal = {}
	Goal.CFrame = NewCFrame
	local TestTween = TweenService:Create(MainPart, TweenInfo.new(Time), Goal)
	TestTween:Play()
end

BonesAbility.StartAttack = function(MainPart)

	local Projectile = ProjectileClass.New(Config["Speed"], Config["Max Time"], MainPart, Character:GetDescendants(), Config["Start Up"])
	
	Projectile.MovementEvent.Event:Once(function()
		local NewPosition = CFrame.lookAt(MainPart.Position, Utility.FindTarget(Data, Mouse))
	end)
	
	Projectile.Completed.Event:Once(function(RayResult)
		Projectile = nil
		
		BonesAbility.TargetHit(RayResult, MainPart)
		
		task.wait(Config["Despawn Time"])
		MainPart.Parent:Destroy()
	end)
end

BonesAbility.TargetHit = function(RayResult, MainPart)
	local Target = RayResult.Instance
	
	if not Target.Parent:FindFirstChild("Humanoid") then
		return
	end
	
	
	
	EnemyHitRemote:FireServer(Target.Parent, script.Name, 1)
	
	local EnemyHRP = Target.Parent:FindFirstChild("HumanoidRootPart")
	--local Vectors = MainPart.CFrame - EnemyHRP.Position
	local Orientation = MainPart.CFrame:ToOrientation()
	
	MainPart.Anchored = false
	
	task.wait()

	local Weld = Instance.new("WeldConstraint")

	Weld.Part0 = MainPart
	Weld.Part1 = Target

	Weld.Parent = MainPart
end





return BonesAbility
```
