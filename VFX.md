Very bad at coding VFX but heres the code
```lua
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local DebrisService = game:GetService("Debris")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local EasingModule = require(ReplicatedStorage.Modules.NikoModules.EasingStyle)

local MeleeHitVFX = ReplicatedStorage.Remotes.VFXRemotes.MeleeHitVFX
local ExplosionVFX = ReplicatedStorage.Remotes.VFXRemotes.ExplosionVFX
local SwingVFX = ReplicatedStorage.Remotes.VFXRemotes.SwingVFX

local TestVFX = ReplicatedStorage.Storage.VFX
local HitVFX = TestVFX.Test

local VFXModule = {}

VFXModule.Explosion = function(Object)
	local Effect = workspace.Boom:Clone()
	
	Effect.Parent = workspace.ActiveVFX
	
	Effect:PivotTo(Object.Main.CFrame * CFrame.Angles(0, math.rad(90), 0))
	
	
		
end

local x = .75

VFXModule.MeleeHit = function(Enemy)
	local HRP = Enemy:FindFirstChild("HumanoidRootPart")

	local Seed = Random.new()
	for i = 1, 20 do
		local RandomSize = math.random(2,5)
		
		local VFX = HitVFX:Clone()
		
		local RandomDirection = Seed:NextUnitVector() * 3
		VFX.CFrame = CFrame.lookAt(HRP.Position+RandomDirection,HRP.Position + RandomDirection * 6)
		VFX.Parent = workspace
		
		local HitGoal = {}
		HitGoal.Transparency = 1
		HitGoal.Size = Vector3.new(0.05, 0.05, RandomSize)
		HitGoal.CFrame = VFX.CFrame 
		
		local HitTween = TweenService:Create(VFX, TweenInfo.new(x, EasingModule.Quint), HitGoal) 
		
		DebrisService:AddItem(VFX, x)
		
		HitTween:Play()
	end
end


VFXModule.BlasterFadeOut = function(CrossBlaster, DelayTimer)
	local TotalTime = 0 - DelayTimer
	
	while true do
		local DeltaTime = task.wait()
		local DelayedDelta = DeltaTime/DelayTimer
		for i, Part in CrossBlaster:GetChildren() do
			if Part:IsA("MeshPart") then
				Part.Transparency = math.min(Part.Transparency + DelayedDelta, 1)
				for i, Part in Part:GetChildren() do
					if Part:IsA("MeshPart") or Part:IsA("UnionOperation") then
						Part.Transparency = math.min(Part.Transparency + DelayedDelta, 1)
					end
				end
			end
		end
		TotalTime += DeltaTime
		if TotalTime >= DelayTimer then
			break
		end
	end
end


VFXModule.Blaster = function(Blaster, TargetPos, MaxRange, MaxTime, Event)
	local VFX = ReplicatedStorage.Storage.VFX
	local StartOutline = VFX.Outline:Clone()
	local VFXModel = VFX.BlasterVFX:Clone()
	
	local StartBlast = StartOutline.BlastStart

	local OutlineVFX = VFXModel.Outline
	local BlastVFX = VFXModel.Blaster

	local BlastSize = BlastVFX.Size
	local BlastOutlineSize = OutlineVFX.Size
	local StartOutlineSize = StartOutline.Size
	local StartBlastSize = StartBlast.Size
	
	local Magnitude = (TargetPos - Blaster.Main.Position).Magnitude + 8
	
	if Magnitude > MaxRange then
		Magnitude = MaxRange
	end
	
	local BlastGoal = Vector3.new(Magnitude - 1, 8, 8)
	local OutlineGoal = Vector3.new(Magnitude, 9, 9)
	
	local TotalTime = 0
	
	StartOutline.CFrame = Blaster.Main.CFrame
	StartBlast.CFrame = Blaster.Main.CFrame
	StartOutline.Parent = workspace.TestVFX
	BlastVFX.CFrame = Blaster.Main.CFrame * CFrame.Angles(0, math.rad(-90), 0)
	OutlineVFX.CFrame = Blaster.Main.CFrame * CFrame.Angles(0, math.rad(-90), 0)
	VFXModel.Parent = workspace.TestVFX
	
	
	for i = 0,1,0.03 do
		local DeltaTime = task.wait()
		
		local BlastLerp = BlastSize:Lerp(BlastGoal,i)
		local OutLineLerp = BlastOutlineSize:Lerp(OutlineGoal,i)

		BlastVFX.CFrame *= CFrame.new(-(BlastLerp.X - BlastVFX.Size.X) / 2, 0, 0)
		BlastVFX.Size = BlastLerp

		OutlineVFX.CFrame *= CFrame.new(-(OutLineLerp.X - OutlineVFX.Size.X) / 2, 0, 0)
		OutlineVFX.Size = OutLineLerp
		
		TotalTime += DeltaTime
	end
	
	local TotalTime = 0 + TotalTime
	local Goal = Vector3.new(1.5, 1.5, 1.5)
	
	while true do
		
		BlastSize = BlastVFX.Size
		BlastOutlineSize = OutlineVFX.Size
		
		StartOutlineSize = StartOutline.Size
		StartBlastSize = StartBlast.Size
		
		for i = 0,1,0.03 do
			local DelaTime = task.wait()

			local BlastLerp = BlastSize:Lerp(BlastSize + Goal,i)
			local OutLineLerp = BlastOutlineSize:Lerp(BlastOutlineSize + Goal ,i)
			local BallBlastLerp = StartBlastSize:Lerp(StartBlastSize + Goal,i)
			local BallOutlineLerp = StartOutlineSize:Lerp(StartOutlineSize + Goal ,i)


			BlastVFX.Size = BlastLerp
			OutlineVFX.Size = OutLineLerp

			StartBlast.Size = BallBlastLerp
			StartOutline.Size = BallOutlineLerp

			TotalTime = math.min(TotalTime + DelaTime, MaxTime)

			if TotalTime == MaxTime then
				break
			end 
		end
		
		if TotalTime == MaxTime then
			Event:Fire()
			break
		end 
		
		if Vector3.new(math.abs(Goal.X), math.abs(Goal.Y), math.abs(Goal.Z)) == Goal then
			Goal = Goal - (Goal * 2)
			continue
		end
		Goal = Goal - (Goal * 2)
	end
	
	BlastSize = BlastVFX.Size
	BlastOutlineSize = OutlineVFX.Size

	StartOutlineSize = StartOutline.Size
	StartBlastSize = StartBlast.Size

	local Goal1 = Vector3.new(BlastSize.X, 2 - 1, 2 - 1)
	local Goal2 = Vector3.new(BlastOutlineSize.X, 2 , 2)

	local Goal3 = Vector3.new(0,0,0)

	for i = 0,1,.02 do
		local BlastLerp = BlastSize:Lerp(Goal1,i)
		local OutLineLerp = BlastOutlineSize:Lerp(Goal2,i)

		local BallBlastLerp = StartBlastSize:Lerp(Goal3,i)
		local BallOutlineLerp = StartOutlineSize:Lerp(Goal3 ,i)


		BlastVFX.Size = BlastLerp
		StartBlast.Size = BallBlastLerp

		OutlineVFX.Size = OutLineLerp
		StartOutline.Size = BallOutlineLerp

		BlastVFX.Transparency = math.min(BlastVFX.Transparency + .02, 1)
		OutlineVFX.Transparency = math.min(OutlineVFX.Transparency + .02, 1)

		StartBlast.Transparency = math.min(StartBlast.Transparency + .02, 1)
		StartOutline.Transparency = math.min(StartOutline.Transparency + .02, 1)

		task.wait()
	end
end




if RunService:IsClient() then
	ExplosionVFX.OnClientEvent:Connect(VFXModule.Explosion)
	MeleeHitVFX.OnClientEvent:Connect(VFXModule.MeleeHit)
end


return VFXModule
```
