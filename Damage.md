A very simple but efffective script it handles multi hits states and knockback all in a few lines of code using modualized code to keep the clone nice and readable

```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

local Physics = require(ReplicatedStorage.Modules.NikoModules.PlayerPhysicsNew)
local Utility = require(ReplicatedStorage.Modules.NikoModules.Utility)
local VFX = require(ReplicatedStorage.Modules.NikoModules.VFX)
local StateManager = require(ReplicatedStorage.Modules.NikoModules.StateManager)
local PlayerData = require(ReplicatedStorage.PlayerData)

local AnimationRemote = ReplicatedStorage.Remotes.Animations
local EnemyHitRemote = ReplicatedStorage.Remotes.AbilityRemotes.EnemyHit

local function TargetHit(Player, Target, Ability, HitNum)
	local Data = PlayerData.GetData(Player)
	local Character = Player.Character
	local Humanoid = Character.Humanoid
	local HRP = Character.HumanoidRootPart
	
	local EnemyPlayer = Players:GetPlayerFromCharacter(Target)
	local EnemyData = Utility.GetPlayerData(Target)
	local EnemyHumanoid = Target.Humanoid
	local EnemyHRP = Target.HumanoidRootPart
	
	local AbilityConfig = require(ReplicatedStorage.Modules.Characters[Data.Character][Ability].Config)
	local AbilityClientConfig = AbilityConfig["Client Config"]
	local Config = AbilityConfig["Server Configs"][HitNum]
	
	if not EnemyData then
		return
	end

	if not EnemyPlayer then
		EnemyHRP:SetNetworkOwner(Player)
		StateManager.Ownership(EnemyHumanoid, 3, EnemyData)
	end
	
	EnemyHumanoid:TakeDamage(Config["Damage"])
	
	for State, Duration in Config["States"] do
		if EnemyPlayer then
			State:FireClient(EnemyPlayer, Duration)
		else
			StateManager[State.Name](EnemyHumanoid, Duration, EnemyData)
		end
	end
	
	if Config["Knockback"] then
		local Velocity = Physics.Velocity(EnemyHRP)
		Physics.StartPhysics(Velocity, HRP, Config["Knockback"])
	end

end

EnemyHitRemote.OnServerEvent:Connect(TargetHit)
```
