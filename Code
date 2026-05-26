local Enemy = {}
Enemy.__index = Enemy

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local Ragdoll = require(game.ServerScriptService.Ragdoll)

local ATTACK_COOLDOWN_TIME = 1
local ATTACK_RANGE = 13
local UPWARD_FORCE = 75
local ATTACK_FORCE = 100
local RAGDOLL_TIME = 2.5
local PATROL_INTERVAL = 3
local AI_TICKRATE = 0.1

function Enemy.new(char, WalkSpeed)
	local self = setmetatable({}, Enemy)

	self.char = char
	self.humanoid = char:WaitForChild("Humanoid")
	self.rootPart = char:WaitForChild("HumanoidRootPart")
	self.PatrolPart = self.char.Parent

	self.attackCooldown = false
	self.moving = false
	self.patrolling = false
	self.aiTimer = 0

	self.humanoid.WalkSpeed = WalkSpeed
	self.humanoid.AutoRotate = true
	self.rootPart.Anchored = false
	self.rootPart:SetNetworkOwner(nil)

	if not self.char:FindFirstChild("AttackAnimation") then
		local anim = Instance.new("Animation")
		anim.Name = "AttackAnimation"
		anim.Parent = self.char
		anim.AnimationId = "rbxassetid://129285406557419"
	end

	self.attackAnim = self.humanoid:LoadAnimation(self.char.AttackAnimation)

	self:Start()


	return self
end

function Enemy:IsInsidePatrol(pos)
	local part = self.PatrolPart
	if not part then return false end

	local size = part.Size / 2
	local center = part.Position

	return
		pos.X > center.X - size.X and
		pos.X < center.X + size.X and
		pos.Z > center.Z - size.Z and
		pos.Z < center.Z + size.Z
end

function Enemy:Patrol()
	if self.patrolling then return end
	self.patrolling = true

	task.spawn(function()
		while self.patrolling do
			task.wait(PATROL_INTERVAL)

			if self.moving then continue end

			local part = self.PatrolPart
			if not part then return end

			local size = part.Size
			local center = part.Position

			local randomX = math.random() * size.X - size.X / 2
			local randomZ = math.random() * size.Z - size.Z / 2

			local target = Vector3.new(
				center.X + randomX,
				center.Y + size.Y / 2 + 2,
				center.Z + randomZ
			)

			self.humanoid:MoveTo(target)
		end
	end)
end

function Enemy:IsPlayerOnAllowedPart(playerChar)
	local root = playerChar:FindFirstChild("HumanoidRootPart")
	if not root then return false end

	local rayParams = RaycastParams.new()
	rayParams.FilterDescendantsInstances = {playerChar}
	rayParams.FilterType = Enum.RaycastFilterType.Exclude

	local result = workspace:Raycast(root.Position, Vector3.new(0,-10,0), rayParams)

	return result and result.Instance == self.char.Parent
end

function Enemy:DetectPlayer()
	local nearest = nil
	local shortest = math.huge

	for _, player in ipairs(Players:GetPlayers()) do
		local character = player.Character

		if character and character:FindFirstChild("HumanoidRootPart") then
			if character.Ragdoll.Value then continue end

			local dist = (character.HumanoidRootPart.Position - self.rootPart.Position).Magnitude

			if dist < shortest then
				shortest = dist
				nearest = character
			end
		end
	end

	return nearest
end

function Enemy:Attack(playerChar)
	if self.attackCooldown then return end
	if not playerChar then return end

	local targetRoot = playerChar:FindFirstChild("HumanoidRootPart")
	if not targetRoot then return end

	local distance = (self.rootPart.Position - targetRoot.Position).Magnitude
	if distance > ATTACK_RANGE then return end

	self.attackCooldown = true

	task.delay(ATTACK_COOLDOWN_TIME,function()
		self.attackCooldown = false
	end)

	self.attackAnim:Play()

	task.wait(0.3)

	local direction = self.rootPart.CFrame.LookVector 

	local bodyvel = Instance.new("BodyVelocity")
	bodyvel.MaxForce = Vector3.new(math.huge,math.huge,math.huge)
	bodyvel.Velocity = direction * ATTACK_FORCE + Vector3.new(0,UPWARD_FORCE,0)
	bodyvel.Parent = targetRoot

	game.Debris:AddItem(bodyvel,0.15)

	Ragdoll.Start(playerChar)
	task.wait(RAGDOLL_TIME)
	Ragdoll.Stop(playerChar)
end

function Enemy:Start()

	self:Patrol()

	RunService.Heartbeat:Connect(function(dt)

		if self.humanoid.Health <= 0 then return end

		self.aiTimer += dt
		if self.aiTimer < AI_TICKRATE then return end
		self.aiTimer = 0

		local playerChar = self:DetectPlayer()

		if playerChar and self:IsPlayerOnAllowedPart(playerChar) then

			local targetRoot = playerChar:FindFirstChild("HumanoidRootPart")

			if targetRoot then

				local offset = targetRoot.Position - self.rootPart.Position
				local distance = offset.Magnitude

				if distance > ATTACK_RANGE then

					local direction = offset.Unit
					local nextPos = self.rootPart.Position + direction * 3

					if self:IsInsidePatrol(nextPos) then
						self.humanoid:Move(direction)
					else
						self.humanoid:Move(Vector3.zero)
					end

				else
					self.humanoid:Move(Vector3.zero)
					self:Attack(playerChar)
				end

			end

		else
			self.moving = false
			self.humanoid:Move(Vector3.zero)
		end

	end)

end

return Enemy
