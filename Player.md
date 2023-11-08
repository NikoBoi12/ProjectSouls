This could very well be one of my most complex and longest scripts I learned a lot about applying mathmatics to code managing UI aswell as logical programming and trying to compact a lot of infortmation into something readable

```lua
local TweenService = game:GetService("TweenService")
local UserInput = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

local Utility = require(ReplicatedStorage.Modules.NikoModules.Utility)
local PlayerPhysics = require(ReplicatedStorage.Modules.NikoModules.PlayerPhysicsNew)
local KnockbackConfig = require(ReplicatedStorage.Modules.Configs.KnockbackConfig)
local PlayerData = require(ReplicatedStorage.PlayerData)
local StateManager = require(ReplicatedStorage.Modules.NikoModules.StateManager)

local LocalPlayer = Players.LocalPlayer
local Character = LocalPlayer.Character
local Humanoid = Character:FindFirstChild("Humanoid")
local HRP = Character:FindFirstChild("HumanoidRootPart")

local Camera = workspace.Camera

local SFX = ReplicatedStorage.Storage.Audio.SFX
local PlayerSFX = SFX.PlayerSFX

local Highlight = ReplicatedStorage.Storage.Misc.Lockon

local Connection = nil

local LockedOnTarget = nil
local Sprinting = false
local IsDashing = false
local DashSprint = false
local TeleportBool = false

local DashingIFrames = 2/60
local DashingActionTimer = 2.5
local TeleportingActionTimer = 10/60
local TeleportCooldown = 2

local StaminaCost = 20

local SinValue = 0

local StaminaConfig = {
	StatLoss = 20, -- Stat gain per second
}

local Data = PlayerData.GetData(LocalPlayer)

local DashConfig = {
	["W"] = {
		LastPressed = os.clock(),
		Name = "ForwardDash"
	},
	["A"] = {
		LastPressed = os.clock(),
		Name = "LeftDash"
	},
	["S"] = {
		LastPressed = os.clock(),
		Name = "BackDash"
	},
	["D"] = {
		LastPressed = os.clock(),
		Name = "RightDash"
	},
}

local function PlayerBase(Input, GameInput)
	if GameInput or Data.Character == "None" then
		return
	end
	
	local CharacterConfig = require(ReplicatedStorage.Modules.Characters[Data.Character])
	local CharacterConfig = CharacterConfig[Data.Phase]["Config"]
	local AnimationConfig = CharacterConfig["Animations"]
	
	if Input.UserInputType == Enum.UserInputType.MouseButton3 or Input.KeyCode == Enum.KeyCode.L then -- lock on
		local Mouse = Utility.GetMousePosition()
		if Mouse then
			local EnemyHumanoid = Mouse.Instance.Parent:FindFirstChild("Humanoid") or Mouse.Instance.Parent.Parent:FindFirstChild("Humanoid")
			if EnemyHumanoid and EnemyHumanoid ~= Humanoid then
				if not LockedOnTarget then
					local EnemyHRP = EnemyHumanoid.Parent:FindFirstChild("HumanoidRootPart")
					
					Data.Target = EnemyHRP
					LockedOnTarget = EnemyHRP
					Highlight.Parent = LockedOnTarget.Parent
					PlayerSFX.LockOnSFX.LockOnBattle:Play()
					
					local EnemyHumanoid = EnemyHRP.Parent:FindFirstChild("Humanoid")
					
					Connection = EnemyHumanoid.Died:Once(function()
						Data.Target = nil
						LockedOnTarget = nil
						Highlight.Parent = nil
						Humanoid.AutoRotate = true
						Camera.CameraType = Enum.CameraType.Custom
					end)
				
					while LockedOnTarget ~= nil do
						local WalkSpeed = Humanoid.WalkSpeed
						local DeltaTime = task.wait()
						if LockedOnTarget ~= nil then
							Camera.CameraType = Enum.CameraType.Scriptable -- Reset this back to custom when lock on is finished
							local Origin = HRP.CFrame * Vector3.new(5,4,10) -- Sets origin of camera + some offset to make it to the side
							
							local Goal = CFrame.lookAt(Origin, LockedOnTarget.Position)
							local MoveDirection = Humanoid.MoveDirection
							
							SinValue += 1
							Camera.CFrame = Camera.CFrame:Lerp(
								Goal * 
								CFrame.Angles(math.sin(SinValue/(800/WalkSpeed))/(1120/WalkSpeed),0,0) * -- Rotates camera up and down (Could make it stronger based on speed)
								CFrame.Angles(0,0,-math.rad(MoveDirection.X*1.5)), -- Rotates camera left->right based on the movedirection
								.1
							)
							
							Humanoid.AutoRotate = false
							HRP.CFrame = CFrame.lookAt(HRP.Position, Vector3.new(EnemyHRP.Position.X, HRP.Position.Y, EnemyHRP.Position.Z))
						end
					end
				end
			end
		end
		if LockedOnTarget ~= nil then
			PlayerSFX.LockOnSFX.Flee:Play()
			Data.Target = nil
			LockedOnTarget = nil
			Highlight.Parent = nil
			Humanoid.AutoRotate = true
			Camera.CameraType = Enum.CameraType.Custom
			Connection = nil
		end
	end
	
	-- InputBegan
	local InputData = DashConfig[Input.KeyCode.Name] --Dashing
	if InputData then
		if os.clock() - InputData.LastPressed <= 0.2 and IsDashing == false then
			if Data.Stamina < 20 or Humanoid.WalkSpeed < 16 then
				print(Humanoid.WalkSpeed)
				print("Out of Stamina")
				return
			end
			IsDashing = true
			DashSprint = true
			local KnockbackConfig = KnockbackConfig[InputData["Name"]]
			local DashDirection = InputData.Direction
			
			Utility.Animation(Humanoid, AnimationConfig["Dash"][InputData["Name"]])
			PlayerSFX.PlayerMovmentSFX.Dash:Play()
			
			Data.Stamina -= 20
			
			local Velocity = PlayerPhysics.Velocity(HRP)
			StateManager.InAction(Humanoid, 1.5, Data)
			StateManager.ActionTaken(LocalPlayer, Humanoid, DashingActionTimer, Data)
			StateManager.IFrames(Humanoid, 2/60, Data)
			PlayerPhysics.StartPhysics(Velocity, HRP, InputData["Name"], LocalPlayer)
			task.wait(KnockbackConfig["Time"])
			Humanoid.WalkSpeed = 34
			Sprinting = true
			task.defer(function()
				while true do
					local DeltaTime = task.wait()
					if Humanoid.MoveDirection == Vector3.new(0,0,0) or Data.Stamina <= 1 then
						Humanoid.WalkSpeed = 16
						DashSprint = false
						Sprinting = false
						break
					end
				end
			end)
			task.wait(KnockbackConfig["Cooldown"])
			IsDashing = false
		end
		InputData.LastPressed = os.clock()
	end
	
	if Input.KeyCode == Enum.KeyCode.R then -- Teleport
		local ManaCost = 10
		
		if TeleportBool == true or Data.Stunned == true or Data.KnockedDown == true or Data.Blocking == true or Data.ActiveAbility == true or Data.IsAttacking == true or Data.Mana < ManaCost then
			return
		end
		TeleportBool = true
		Data.Mana -= ManaCost
		StateManager.InAction(Humanoid, 20/60, Data)
		StateManager.ActionTaken(LocalPlayer, Humanoid, TeleportingActionTimer, Data)
		local HRPPos = HRP.Position
		if LockedOnTarget then
			local TargetPos = LockedOnTarget.Position
			local TeleportMagnitude = (Vector3.new(HRPPos.X, 0, HRPPos.Z) - Vector3.new(TargetPos.X, 0, TargetPos.Z)).Magnitude
			local TeleportPosition = CFrame.lookAt(HRP.Position, LockedOnTarget.Position) + HRP.CFrame.LookVector*(TeleportMagnitude-3.5)
			if TeleportMagnitude >= 40 then
				TeleportPosition = CFrame.lookAt(HRP.Position, LockedOnTarget.Position) + HRP.CFrame.LookVector*(40-3.5)
			end
			
			HRP.CFrame = TeleportPosition
		else
			local MouseHit = Utility.GetMousePosition()
			if MouseHit then
				local MousePos = MouseHit.Position
				
				HRP.CFrame = CFrame.lookAt(HRPPos, Vector3.new(MousePos.X, HRPPos.Y, MousePos.Z))
				
				local TeleportMagnitude = (Vector3.new(HRPPos.X, 0, HRPPos.Z) - Vector3.new(MousePos.X, 0, MousePos.Z)).Magnitude
				local TeleportPostion = HRP.CFrame.LookVector*(TeleportMagnitude)
				if TeleportMagnitude >= 36.5 then
					TeleportPostion = HRP.CFrame.LookVector*(36.5)
				end
				HRP.CFrame = HRP.CFrame + TeleportPostion
			end
		end
		task.wait(TeleportCooldown)
		TeleportBool = false
	end
end

UserInput.InputBegan:Connect(PlayerBase)

task.defer(function()
	while true do
		local DeltaTime = task.wait()
		if Humanoid.WalkSpeed >= 16 then
			local Acceleration = DeltaTime * 100
		
			local SpeedChange = 0
			if UserInput:IsKeyDown(Enum.KeyCode.LeftShift) and Data.Stamina >= 1 and Humanoid.MoveDirection ~= Vector3.new(0,0,0) or Sprinting == true then
				Data.Stamina = math.clamp(Data.Stamina - StaminaConfig["StatLoss"]*DeltaTime,0,200)
				StateManager.ActionTaken(LocalPlayer, Humanoid, 2, Data)
				SpeedChange = 0.2 * Acceleration
			else
				SpeedChange = -0.2 * Acceleration
				if DashSprint == true then
					continue
				end
				Sprinting = false
			end
			if Humanoid.MoveDirection == Vector3.new(0,0,0) then
				Humanoid.WalkSpeed = 16
			end
			Humanoid.WalkSpeed = math.clamp(Humanoid.WalkSpeed+SpeedChange,16,34)
		end
	end
end)```
