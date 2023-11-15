```lua
local Blaster = {}

if game:GetService("RunService"):IsServer() then
	return Blaster
end

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local Utility = require(ReplicatedStorage.Modules.NikoModules.Utility)
local VFX = require(ReplicatedStorage.Modules.NikoModules.VFX)
local PlayerData = require(ReplicatedStorage.PlayerData)
local StateManager = require(ReplicatedStorage.Modules.NikoModules.StateManager)
local AbilityConfig = require(script.Config)
local Physics = require(ReplicatedStorage.Modules.NikoModules.PlayerPhysicsNew)

local EnemyHitRemote = ReplicatedStorage.Remotes.AbilityRemotes.EnemyHit

local LocalPlayer = Players.LocalPlayer
local Character = LocalPlayer.Character
local Humanoid = Character:FindFirstChild("Humanoid")
local HRP = Character:FindFirstChild("HumanoidRootPart")

local Config = AbilityConfig["Client Config"]
local ConfigModels = Config["Models"]
local ServerAbilityConfig = AbilityConfig["Server Configs"]

local Mouse = LocalPlayer:GetMouse()
local Data = PlayerData.GetData(LocalPlayer)

local ActiveCooldown = false

local Bindable = Instance.new("BindableEvent")

Blaster.Use = function()
	if Data.Stunned == true or Data.KnockedDown == true or Data.InAction == true or ActiveCooldown == true  or Data.Mana < Config["Mana Cost"] then
		return
	end
	
	ActiveCooldown = true
	
	Data.InAction = true
	Data.Mana -= Config["Mana Cost"]
	StateManager.ActionTaken(LocalPlayer, Humanoid, Config["Action Taken Timer"], Data)
	Humanoid.WalkSpeed = 0
	Humanoid.JumpHeight = 0
	
	Blaster.PlayerJump()
	
	task.wait(Config["Cooldown"])
	ActiveCooldown = false
end


Blaster.PlayerJump = function()
	local Velocity = Physics.Velocity(HRP)

	Physics.StartPhysics(Velocity, HRP, "JumpBack", true)
	
	Blaster.SpawnBlaster()
end


Blaster.SpawnBlaster = function()
	local Target = Utility.FindTarget(Data, Mouse)
	
	local CrossBlaster = Config["Models"]["Blaster"]:Clone()

	CrossBlaster:PivotTo(HRP.CFrame + Vector3.new(0, 10, 0))
	CrossBlaster.Parent = workspace
	
	Blaster.Aim(CrossBlaster, Target)
	Blaster.Animations(CrossBlaster, Target)
end


Blaster.Aim = function(CrossBlaster, Target)
	CrossBlaster:PivotTo(CFrame.lookAt(CrossBlaster:GetPivot().Position, Target))
end


Blaster.Animations = function(CrossBlaster, Target)
	local AnimationController = CrossBlaster.AnimationController
	local Animator = AnimationController.Animator
	local Animations = AnimationController.Animations
	
	local Tracks = {}
	
	for i, Animations in Animations:GetChildren() do
		local AnimationTrack = Animator:LoadAnimation(Animations)
		
		Tracks[Animations.Name] = AnimationTrack
	end
	
	for i, Particle in CrossBlaster.Main.Attachment:GetChildren() do
		Particle.Enabled = true
	end
	
	Tracks["BlasterFire"]:Play()
	Tracks["BlasterFire"].Stopped:Wait()
	Tracks["BlasterIdle"].Looped = true
	Tracks["BlasterIdle"]:Play()
	
	task.defer(function()
		VFX.Blaster(CrossBlaster, Target, Config["MaxRange"], Config["Max Time"], Bindable)
	end)
	
	Bindable.Event:Connect(function()
		Tracks["BlasterIdle"].Looped = false
		Tracks["BlasterIdle"].Stopped:Wait()
		Tracks["BlasterFinished"]:Play()
		for i, Particle in CrossBlaster.Main.Attachment:GetChildren() do
			Particle.Enabled = false
		end
		Blaster.BlastFinished(CrossBlaster)
	end)
	
	Blaster.FireBlaster(CrossBlaster)
end


Blaster.FireBlaster = function(CrossBlaster)
	local HitBox = Config["HitBox"]:Clone()
	
	HitBox.Size = Vector3.new(Config["MaxRange"], HitBox.Size.Y, HitBox.Size.Z)
	HitBox.CFrame = CrossBlaster.Main.CFrame * CFrame.Angles(0, math.rad(90), 0) + CrossBlaster.Main.CFrame.LookVector * Vector3.new(Config["MaxRange"]/2, Config["MaxRange"]/2, Config["MaxRange"]/2)
	HitBox.Parent = workspace
	 
	for i, Configuration in ServerAbilityConfig do
		local Enemies = Utility.PartsInHitBox(HitBox, Humanoid)
		for _, Enemy in Enemies do
			EnemyHitRemote:FireServer(Enemy.Parent, script.Name, i)
		end
		task.wait(Config["Yield Hit"])
	end
	HitBox:Destroy()
end


Blaster.BlastFinished = function(CrossBlaster)
	local Emitter = CrossBlaster.Head.DustingParticle
	
	Data.InAction = false
	Emitter.Enabled = true
	
	Humanoid.WalkSpeed = 16
	Humanoid.JumpHeight = 7.2
	
	VFX.BlasterFadeOut(CrossBlaster, 1)
	
	Emitter.Enabled = false
	task.wait(Emitter.Lifetime.Max)
	CrossBlaster:Destroy()
end


return Blaster
```
