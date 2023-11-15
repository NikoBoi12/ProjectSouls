```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

local Physics = require(ReplicatedStorage.Modules.NikoModules.PlayerPhysicsNew)
local Utility = require(ReplicatedStorage.Modules.NikoModules.Utility)
local VFX = require(ReplicatedStorage.Modules.NikoModules.VFX)
local StateManager = require(ReplicatedStorage.Modules.NikoModules.StateManager)
local PlayerData = require(ReplicatedStorage.PlayerData)

local StateRemoveFolder = ReplicatedStorage.Remotes.StateRemotes
local KnockdownRemoteTimer = StateRemoveFolder.Knockdown
local StunnedRemoteTimer = StateRemoveFolder.Stunned
local StartPhysics = ReplicatedStorage.Remotes.PlayerForce
local MeleeHit = ReplicatedStorage.Remotes.MeleeComboHit
local MeleeHitVFX = ReplicatedStorage.Remotes.VFXRemotes.MeleeHitVFX
local AnimationRemote = ReplicatedStorage.Remotes.Animations


local function Melee(Player, Enemy, Combo)
	local EnemyHumanoid = Enemy:FindFirstChild("Humanoid")
	local EnemyHRP = Enemy:FindFirstChild("HumanoidRootPart")
	local EnemyPlayer = Players:GetPlayerFromCharacter(Enemy)
	local Character = Player.Character
	local Humanoid = Character:FindFirstChild("Humanoid")
	local HRP = Character:FindFirstChild("HumanoidRootPart")
	
	local Data = Utility.GetPlayerData(Character)
	local EnemyData = Utility.GetPlayerData(Enemy)
	
	local CharacterConfig = require(ReplicatedStorage.Modules.Characters[Data.Character])
	
	local CharacterConfig = CharacterConfig[Data.Phase]["Config"]
	local AnimationConfig = CharacterConfig["Animations"]
	local Config = CharacterConfig["Melee"][Combo]
	
	if not EnemyPlayer then
		EnemyHRP:SetNetworkOwner(Player)
		StateManager.Ownership(EnemyHumanoid, 3, EnemyData)
	end

	if EnemyData.Blocking == true then
		if EnemyData.PerfectBlocking == true then
			
			local Velocity = Physics.Velocity(HRP)
			print("Checking Stun")
			if Player then
				StartPhysics:FireClient(Player, Velocity, HRP, "PerfectBlock")
				StunnedRemoteTimer:FireClient(Player, 1.5)
				if EnemyPlayer then
					StunnedRemoteTimer:FireClient(Player, 80/60)
				else
					StateManager.Stunned(Humanoid, 80/60, Data)
				end
				print("Target is perfect blocking")
				return
			end
			Physics.StartPhysics(Velocity, HRP, "PerfectBlock")
			StateManager.Stunned(Humanoid, 1.5, Data)
			StateManager.Stunned(EnemyHumanoid, .5, EnemyData)
		end
		if Combo == #CharacterConfig["Melee"] then
			local Velocity = Physics.Velocity(EnemyHRP)
			if Player then
				print("Running")
				StunnedRemoteTimer:FireClient(Player, 10/60)
			else
				StateManager.Stunned(Humanoid, 10/60, Data)
			end
			print("Guard Broken")
			StateManager.Stunned(EnemyHumanoid, 30/60, EnemyData)
			Physics.StartPhysics(Velocity, HRP, "GuardBroken")
			return
		end
		--Animation Code
		print("Target is blocking")
		return
	end
	
	MeleeHitVFX:FireAllClients(Enemy)

	EnemyHumanoid:TakeDamage(Config["Damage"]*EnemyData.DamageMultiplier) --Change Later to PlayerData Health
	
	if Config["Knockback"] then
		local Velocity = Physics.Velocity(EnemyHRP)
		
		if EnemyPlayer then
			AnimationRemote:FireClient(EnemyPlayer, EnemyHumanoid, AnimationConfig["Knockback Types"]["Knockback1"])
			StartPhysics:FireClient(EnemyPlayer, Velocity, EnemyHRP, "Knockback")
			KnockdownRemoteTimer:FireClient(EnemyPlayer, 2)
			return
		end
		
		Utility.Animation(EnemyHumanoid, AnimationConfig["Knockback Types"]["Knockback1"])
		StateManager.Knockdown(EnemyHumanoid, 3, EnemyData)
		Physics.StartPhysics(Velocity, HRP, "Knockback")
	else
		local Velocity = Physics.Velocity(EnemyHRP)

		
		if EnemyPlayer then
			AnimationRemote:FireClient(EnemyPlayer, EnemyHumanoid, AnimationConfig["Stun Types"]["Hit Stun"..Combo])
			StartPhysics:FireClient(EnemyPlayer, Velocity, HRP, "Stunned")
			StunnedRemoteTimer:FireClient(EnemyPlayer, 2)
			return
		end
		
		Utility.Animation(EnemyHumanoid, AnimationConfig["Stun Types"]["Hit Stun"..Combo]) -- Players animations
		StateManager.Stunned(EnemyHumanoid, .5, EnemyData)
		Physics.StartPhysics(Velocity, HRP, "Stunned")
	end
end



local function HitBox(Player, Enemies, Combo)
	local Character = Player.Character
	local HRP = Character:FindFirstChild("HumanoidRootPart")
	local Humanoid = Character:FindFirstChild("Humanoid")
	local Data = Utility.GetPlayerData(Character)
	
	if Data.Stunned == true or Data.KnockedDown == true or Data.Blocking == true or Data.ActiveAbility == true then
		print(Data.Stunned, Data.KnockedDown, Data.Blocking, Data.ActiveAbility)
		return
	end
	
	local PlayerForwardPhysics = Physics.Velocity(HRP)

	for i, Enemy in Enemies do
		local EnemyData = Utility.GetPlayerData(Enemy.Parent)
		local EnemyHRP = Enemy.Parent:FindFirstChild("HumanoidRootPart")
		
		if EnemyData == nil then
			warn("Not a valid target!")
			continue
		end
		
		if (EnemyHRP.Position - HRP.Position).Magnitude >= 10 or EnemyData.IFrames == true or EnemyData.KnockedDown == true then
			print(EnemyData.IFrames, EnemyData.KnockedDown)
			continue
		end
		
		if EnemyData.Blocking == true then
			PlayerForwardPhysics = nil
		end
		
		Melee(Player, Enemy.Parent, Combo)
		
	end
	if PlayerForwardPhysics ~= nil then
		StartPhysics:FireClient(Player, PlayerForwardPhysics, HRP, "Attacking")
	end
end

MeleeHit.OnServerEvent:Connect(HitBox)
```
