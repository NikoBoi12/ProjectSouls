2023-6-16 This is my Projectile module and OOP creation it is used to manage all of my projectiles in the game I learned a lot with this learning to modualize code and creation of a modulalized scripting projects.

```lua
local Players = game:GetService("Players")

local Utility = require(game.ReplicatedStorage.Modules.NikoModules.Utility)


local Projectile = {}

local Manager = {}

Projectile.New = function(Speed, MaxTime, MainPart, Params, StartUpDelay)
	local self = Utility.Factory(Projectile)
	
	self.Completed = Instance.new("BindableEvent")
	self.MovementEvent = Instance.new("BindableEvent")
	self.MainPart = MainPart
	self.Speed = Speed -- Studs per second
	self.TotalTime = 0
	self.MaxTime = MaxTime
	self.StartUpDelay = StartUpDelay or nil
	self.Params = RaycastParams.new()
	
	self.Params.FilterDescendantsInstances = Params
	self.Params.FilterType = Enum.RaycastFilterType.Exclude
	
	self:StartMovment()

	return self
end


Projectile.StartMovment = function(self)
	table.insert(Manager, self)
end

Projectile.Update = function(self, DeltaTime)
	self.TotalTime = math.clamp(self.TotalTime + DeltaTime,0,self.MaxTime)
	
	if self.StartUpDelay then
		if self.TotalTime >= self.StartUpDelay then
			self.StartUpDelay = nil
			self.TotalTime = 0
			self.MovementEvent:Fire()
		end
		return
	end
	
	self:CastRay(DeltaTime)
	self:Movment(DeltaTime)
	
	if self.TotalTime == self.MaxTime then
		self:DespawnProjectile()
	end
end

Projectile.CastRay = function(self, DeltaTime)
	local RaycastResult = workspace:Raycast(self.MainPart.Position, self.MainPart.CFrame.LookVector * self.Speed * DeltaTime, self.Params)
	
	if RaycastResult then
		--self:Movment(DeltaTime)
		self:Result(RaycastResult)
	end
end

Projectile.Movment = function(self, DeltaTime)
	self.MainPart.CFrame = self.MainPart.CFrame + self.MainPart.CFrame.LookVector * self.Speed * DeltaTime
end

Projectile.Result = function(self, RaycastResult)
	local Find = table.find(Manager, self)
	table.remove(Manager, Find)
	self.Completed:Fire(RaycastResult)
end

Projectile.DespawnProjectile = function(self)
	local Find = table.find(Manager, self)
	table.remove(Manager, Find)
	self.MainPart.Parent:Destroy()
	self.Completed:Destroy()
end


task.defer(function()
	while true do
		local DeltaTime = task.wait()
		for i, Projectile in Manager do
			Projectile:Update(DeltaTime)
		end
	end
end)


return Projectile
```
