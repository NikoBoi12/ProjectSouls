
This is a OOP formatted class which can be used to inherit certain features of the timer to a seperate timer class this is used for my game as a state timer which manages about 16 different ingame states.

```lua
local Utility = require(game.ReplicatedStorage.Modules.NikoModules.Utility)

local Timer = {}

local Timers = {}

Timer.new = function(Duration, Start, Name)
	local self = Utility.Factory(Timer)
	self.Name = Name
	self.Completed = Instance.new("BindableEvent")
	self.Current = 0
	self.Duration = Duration
	self.IsRunning = false
	
	if Start then
		self:Start()
	end
	
	return self
end

Timer.Update = function(self, DeltaTime)
	print(self.Current)
	self.Current += DeltaTime
	if self.Current >= self.Duration then
		local CurrentTimer = table.find(Timers, self)
		self.IsRunning = false
		table.remove(Timers, CurrentTimer)
		self.Completed:Fire()
	end
end

Timer.Start = function(self)
	if self.IsRunning == true then
		return
	end
	
	self.IsRunning = true
	table.insert(Timers,self)
end


Timer.Pause = function(self)
	self.IsRunning = false
	local CurrentTimer = table.find(Timers, self)
	table.remove(Timers, CurrentTimer)
end


Timer.Restart = function(self)
	self.Current = 0
end

task.defer(function()
	while true do
		local DeltaTime = task.wait()
		for i,Timer in Timers do
			Timer:Update(DeltaTime)
		end
	end
end)

return Timer
```
