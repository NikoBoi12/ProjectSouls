```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local StateRemoveFolder = ReplicatedStorage.Remotes.StateRemotes
local KnockdownRemoteTimer = StateRemoveFolder.Knockdown
local StunnedRemoteTimer = StateRemoveFolder.Stunned

local Storage = ReplicatedStorage.Storage
local ProjectileStorage = Storage.Projectiles

local BonesConfig = {
	["Client Config"] = {
		["Cooldown"] = 60/60,
		["Start Up"] = 30/60,
		["Speed"] = 400,
		["Mana Cost"] = 20,
		["InAction Timer"] = 2,
		
		--["Visual Remote"] = ReplicatedStorage.Remotes.AbilityVisualRemote.Ability1Visual,
		
		["Number Of Projectiles"] = 26,
		["Delay Per Bone Spawn"] = 3/60,
		["Max Time"] = 5,
		["Despawn Time"] = 1.5,
		
		["Tween Speed"] = 20,
		
		["Action Taken Timer"] = 2.5,
		
		["Models"] = {
			["Projectile"] = ProjectileStorage.Cross.CrossBone
		},
	},
	["Server Configs"] = {
		{
			["Damage"] = 1,
			["Knockback"] = nil,
			["States"] = {
				[StunnedRemoteTimer] = 3, -- Duration
			},
		},
	},
}

return BonesConfig
```
