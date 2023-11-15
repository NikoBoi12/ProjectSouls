```lua
local Cross = {
	["Phase1"] = {
		["Config"] = require(script.Phase1Config),
		["Abilities"] = {
			{	
				[Enum.KeyCode.One] = require(script.Ability1),
				[Enum.KeyCode.Two] = require(script.Ability2),
				[Enum.KeyCode.Three] = nil,
				[Enum.KeyCode.Four] = nil,
			},
			
			{
				[Enum.KeyCode.One] = require(script.AltAbility1),
				[Enum.KeyCode.Two] = nil,
				[Enum.KeyCode.Three] = nil,
				[Enum.KeyCode.Four] = nil,
			}
		},
	},
	["Phase2"] = {
		["Config"] = require(script.Phase2Config),
		["Abilities"] = {
			{
				
			},
			
			{
				
			},
		},
	}
}

return Cross
```
