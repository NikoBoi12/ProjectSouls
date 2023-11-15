```lua
local TweenService = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

local PlayerData = require(ReplicatedStorage.PlayerData)
local NetSync = require(ReplicatedStorage.NetSync)
local TimerClass = require(ReplicatedStorage.Modules.Classes.Timer)

local BlockingRemote = ReplicatedStorage.Remotes.Blocking
local PerfectBlockingRemote = ReplicatedStorage.Remotes.PerfectBlocking

local PerfectBlockDuration = 6/60
local PerfectBlockCooldown = 2

local DamageReduction = .50

local PlayerInfo = {} --[[
Player = {
	["Debounce"] = bool,
	["Connection"] = Connection
}
]]
local PerfectBlock = {} --[[
Player = bool
]]

local function Blocked(Player, Blocking)
	local Data = nil
	local Character = Player.Character
	if Player then
		Data = PlayerData.GetData(Player)
	else
		warn("Not a valid Player!")
		return
	end
	if Data.Blocking == false then
		Data.Blocking = true
		Data.DamageMultiplier -= DamageReduction

		--Temp Code to be replaced by animation on the client
		
		local Timer = nil
		
		if not PerfectBlock[Player] then
			PerfectBlock[Player] = nil
		end
		
		if not PerfectBlock[Player] then
			print("Searching")
			Data.PerfectBlocking = true
			Timer = TimerClass.new(PerfectBlockDuration, true)
			
			PerfectBlock[Player] = Timer

			PerfectBlock[Player].Completed.Event:Once(function()
				Data.PerfectBlocking = false
				task.wait(PerfectBlockCooldown)
				PerfectBlockingRemote:FireClient(Player)
				PerfectBlock[Player] = nil
			end)
		end
	else 
		if PlayerInfo[Player] then
			if PlayerInfo[Player]["Bool"] == true then
				return
			end
		end
		
		if PlayerInfo[Player] == nil then
			PlayerInfo[Player] = {}
			PlayerInfo[Player]["Bool"] = true
			PlayerInfo[Player]["Connection"] = Player.Destroying:Once(function()
			
				PlayerInfo[Player] = nil
			end)
		end
		print(PlayerInfo[Player])
		print(Data.PerfectBlocking)
		while Data.PerfectBlocking == true do
			local DeltaTime = task.wait()
			--Yielding
		end
		
		Data.Blocking = false
		
		Data.DamageMultiplier += DamageReduction
		
		PlayerInfo[Player] = nil
		print("Unblocking")
		BlockingRemote:FireClient(Player)
		
	end
	
end

BlockingRemote.OnServerEvent:Connect(Blocked)
```
