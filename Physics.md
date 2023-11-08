2023-5-12

This is still a WIP as I would like to eventually impliment my own physics instead of rely on the engine but this taught me a lot about managing all player knockback within a single module

```lua
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local KnockbackConfig = require(ReplicatedStorage.Modules.Configs.KnockbackConfig)
local StateManager = require(ReplicatedStorage.Modules.NikoModules.StateManager)
local PlayerData = require(ReplicatedStorage.PlayerData)

local StartPhysics = ReplicatedStorage.Remotes.PlayerForce

local Physics = {}

Physics.Velocity = function(TargetHRP)

	for i, Object in TargetHRP:GetChildren() do
		if Object.ClassName == "LinearVelocity" or Object.Name == "LinearAttachment" then
			Object:Destroy()
		end
	end

	local Knockback = Instance.new("LinearVelocity")
	local Attachment = Instance.new("Attachment")

	Attachment.Name = "LinearAttachment"
	Attachment.Parent = TargetHRP

	Knockback.Attachment0 = Attachment
	Knockback.Parent = TargetHRP

	return Knockback
end

Physics.StartPhysics = function(LinearVelocity, HRP, Type, Ignore)
	local Config = KnockbackConfig[Type]
	LinearVelocity.MaxForce = Config["MaxForce"]
	LinearVelocity.VectorVelocity = HRP.CFrame[Config["DirectionType"]] * Config["Direction"] * Config["Force"]
	
	if not Ignore then
		task.defer(function() task.wait(Config["Time"]) LinearVelocity:Destroy() end)
	else
		task.wait(Config.Time) LinearVelocity:Destroy() 
	end
end

if RunService:IsClient() then
	StartPhysics.OnClientEvent:Connect(Physics.StartPhysics)
end

return Physics
```
