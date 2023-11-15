```lua
local Storage = game.ReplicatedStorage.Storage

local Cross = {
	["Health"] = 100,
	["Attack"] = 65,
	["Defense"] = 220,
	["Accessories"] = {
		["Weapon"] = Storage.Weapons.XKnife
	},
	["Animations"] = {
		["Stun Types"] = {
			["Hit Stun1"] = 13657886268,
			["Hit Stun2"] = 13657889570,
			["Hit Stun3"] = 13657891689,
			["Hit Stun4"] = 13657894885
		},
		["Knockback Types"] = {
			["Knockback1"] = 13655161509,
		},
		["Melee"] = {
			12663044922,
			12663089665,
			12663070770,
			13658117154,
			13658071220,
		},
		["Dash"] = {
			["ForwardDash"] = 14111811347,
			["LeftDash"] = 14111806873,
			["BackDash"] = 14111792984,
			["RightDash"] = 14111802595,
		},
	},
	["Melee"] = {
		{
			["Damage"] = 3,
			["Swing Speed"] = .25,
			["Stun Duration"] = .15,
			["Range"] = Vector3.new(5, 5, 6),
			["Knockback"] = nil
		},
		{
			["Damage"] = 3,
			["Swing Speed"] = .25,
			["Stun Duration"] = .15,
			["Range"] = Vector3.new(5, 5, 6),
			["Knockback"] = nil
		},
		{
			["Damage"] = 3,
			["Swing Speed"] = .25,
			["Stun Duration"] = .15,
			["Range"] = Vector3.new(5, 5, 6),
			["Knockback"] = nil
		},
		{
			["Damage"] = 3,
			["Swing Speed"] = .25,
			["Stun Duration"] = .15,
			["Range"] = Vector3.new(5, 5, 6),
			["Knockback"] = nil
		},
		{
			["Damage"] = 5,
			["Swing Speed"] = .25,
			["Stun Duration"] = .15,
			["Range"] = Vector3.new(5, 5, 6),
			["Knockback"] = {
				["Can Knockback"] = true,
				["Knockback Distance"] = 50,
				["Speed"] = 3,
			},
		},
	},
	["Theme"] = nil
}

return Cross
```
