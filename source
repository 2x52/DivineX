if not game:IsLoaded() then game.Loaded:Wait() end
if game.PlaceId ~= 4490140733 then return end

local Library = require(game:GetService("ReplicatedStorage"):WaitForChild("Framework", 10):WaitForChild("Library", 10));
assert(Library, "Oopps! Library has not been loaded. Maybe try re-joining?") 
while not Library.Loaded do wait() end

print("Library has been loaded!")

-- ANTI-AFK
if getconnections then
	for i,v in next, getconnections(game.Players.LocalPlayer.Idled) do
		v:Disable()
	end
end

local Players = game:GetService("Players")
local Player = Players.LocalPlayer

getgenv().SecureMode = true
local Rayfield = loadstring(game:HttpGet('https://raw.githubusercontent.com/rafacasari/Rayfield/main/source'))()
assert(Rayfield, "Oopps! Rayfield has not been loaded. Maybe try re-joining?") 

function GetPath(...)
    local path = {...}
    local oldPath = Library
	if path and #path > 0 then
		for _,v in ipairs(path) do
			oldPath = oldPath[v]
		end
	end
    return oldPath
end 

local Food = GetPath("Food")
local Entity = GetPath("Entity")
local Customer = GetPath("Customer")
local Waiter = GetPath("Waiter")
local Appliance = GetPath("Appliance")
local Bakery = GetPath("Bakery")

local Original_RandomFoodChoice = Food.RandomFoodChoice
local GoldFood = false
Food.RandomFoodChoice = function(customerOwnerUID, customerOwnerID, isRichCustomer, isPirateCustomer, isNearTree)
    if GoldFood then
		local spoof = Food.new("45", customerOwnerUID, customerOwnerID, true, true)
		spoof.IsGold = true
		return spoof
	end
	
	return Original_RandomFoodChoice(customerOwnerUID, customerOwnerID, isRichCustomer, isPirateCustomer, isNearTree)
end

local Original_DropPresent = Customer.DropPresent
local AutoGift = false

local Original_CheckForFoodDelivery = Waiter.CheckForFoodDelivery
Waiter.CheckForFoodDelivery = function(waiter)
	if not GoldFood then 
		return Original_CheckForFoodDelivery(waiter)
	end
	
	local myFloor = waiter:GetMyFloor()
	local readyStands = myFloor:GatherOrderStandsWithDeliveryReady()
	if #readyStands == 0 then		
		local indices = Library.Functions.RandomIndices(Library.Variables.MyBakery.floors)
		for _, index in ipairs(indices) do
			local floor = Library.Variables.MyBakery.floors[index]
			if floor ~= myFloor and not floor:HasAtLeastOneIdleStateOfClass("Waiter") then
				readyStands = floor:GatherOrderStandsWithDeliveryReady()
				if #readyStands > 0 then break end
			end		
		end
		
		if #readyStands == 0 then
			return false
		end
	end
	
	local orderStand = readyStands[math.random(#readyStands)]
	if not orderStand then
		return false
	end
	
	orderStand.stateData.foodReadyTargetCount = orderStand.stateData.foodReadyTargetCount + 1
	waiter.state = "WalkingToPickupFood"
	waiter:WalkToNewFloor(orderStand:GetMyFloor(), function()
		if orderStand.isDeleted then
			waiter.state = "Idle"
			return
		end
		
		waiter:WalkToPoint(orderStand.xVoxel, orderStand.yVoxel, orderStand.zVoxel, function()
			if orderStand.isDeleted then
				waiter.state = "Idle"
				return
			end
			
			orderStand.stateData.foodReadyTargetCount = orderStand.stateData.foodReadyTargetCount - 1
			if #orderStand.stateData.foodReadyList == 0 then
				waiter.state = "Idle"
				return
			end
			
			local selectedFoodOrder = orderStand.stateData.foodReadyList[1]
			selectedFoodOrder.isGold = true
			
			table.remove(orderStand.stateData.foodReadyList, 1)

			selectedFoodOrder:DestroyPopupListItemUI()
			local customerOfOrder = waiter:EntityTable()[selectedFoodOrder.customerOwnerUID]
			if not customerOfOrder then
				Library.Print("CRITICAL: customer owner of food not found", true)
				waiter.state = "Idle"
				return false
			end
			waiter:FaceEntity(orderStand)
			waiter:HoldFood(selectedFoodOrder.ID, selectedFoodOrder.isGold)
			waiter.state = "WalkingToDeliverFood"
			if not customerOfOrder.isDeleted then
				waiter:WalkToNewFloor(customerOfOrder:GetMyFloor(), function()
					waiter:WalkToPoint(customerOfOrder.xVoxel, customerOfOrder.yVoxel, customerOfOrder.zVoxel, function()
						waiter:DropFood()
						if customerOfOrder.isDeleted then
							Library.Print("CRITICAL: walked to customer, but they were forced to leave.  aborting", true)
							waiter.state = "Idle"
							return
						end
						customerOfOrder:ChangeToEatingState()
						waiter:FaceEntity(customerOfOrder)
						Library.Network.Fire("AwardWaiterExperienceForDeliveringOrderWithVerification", waiter.UID)
						waiter.state = "Idle"
					end)
				end)
				return
			end
			waiter.state = "Idle"
			waiter.stateData.heldDish = waiter.stateData.heldDish:Destroy()
		end)
	end)
	
	return true
end

Customer.DropPresent = function(gift) 
	if AutoGift then
		local character = Player.Character or Player.CharacterAdded:Wait()
		local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
		
		local UID = Library.Network.Invoke("Santa_RequestPresentUID", gift.UID)
		Library.Network.Fire("Santa_PickUpGift", UID, humanoidRootPart.Position + Vector3.new(1,0,0))
	else 
		Original_DropPresent(gift)
	end
end



local Window = Rayfield:CreateWindow({
   Name = "My Restaurant!",
   LoadingTitle = "My Restaurant!",
   LoadingSubtitle = "by MilkUp Community",
   ConfigurationSaving = {
      Enabled = true,
      FolderName = nil,
      FileName = "MyRestaurant"
   }
})

local FarmTab = Window:CreateTab("Farm")
local SettingsSection = FarmTab:CreateSection("Farm Options")

local Original_ChangeToWaitForOrderState = Customer.ChangeToWaitForOrderState
local FastOrder = false
Customer.ChangeToWaitForOrderState = function(customer)
	if not FastOrder then 
		Original_ChangeToWaitForOrderState(customer) 
		return
	end

	if customer.state ~= "WalkingToSeat" then return end
	
	local seatLeaf = customer:EntityTable()[customer.stateData.seatUID]
	local tableLeaf = customer:EntityTable()[customer.stateData.tableUID]
			
	if seatLeaf.isDeleted or tableLeaf.isDeleted then
		customer:ForcedToLeave()
		return
	end
	
	customer:SetCustomerState("ThinkingAboutOrder")
	customer:SitInSeat(seatLeaf).Completed:Connect(function()
	
		customer.humanoid:SetStateEnabled(Enum.HumanoidStateType.Seated, true)
		customer.xVoxel = seatLeaf.xVoxel
		customer.zVoxel = seatLeaf.zVoxel
		
		coroutine.wrap(function()
			wait(0.05)
			customer:ReadMenu()
			wait(0.1)
			
			if customer.isDeleted or customer.state ~= "ThinkingAboutOrder" then return end
			
			customer:StopReadingMenu()
			customer:SetCustomerState("DecidedOnOrder")
			
			local myGroup = {customer}
			for _, partner in ipairs(customer.stateData.queueGroup) do
				if not partner.isDeleted then
					table.insert(myGroup, partner)
				end
			end
			local foundUndecidedMember = false
			for _, groupMember in ipairs(myGroup) do
				if groupMember.state ~= "DecidedOnOrder" then
					foundUndecidedMember = true
					break
				end
			end
			
			if not foundUndecidedMember then
				for _, groupMember in ipairs(myGroup) do
					groupMember:ReadyToOrder()
				end
			end
		end)()
	end)
end

local FastOrderToggle = FarmTab:CreateToggle({
   Name = "Fast Order",
   CurrentValue = false,
   Flag = "FastOrder",
   Callback = function(Value)
		FastOrder = Value
   end
})

local GoldFoodToggle = FarmTab:CreateToggle({
   Name = "Gold Food",
   CurrentValue = false,
   Flag = "GoldFood",
   Callback = function(Value)
		GoldFood = Value
   end
})

local AutoGiftToggle = FarmTab:CreateToggle({
   Name = "Collect Gifts",
   CurrentValue = false,
   Flag = "AutoGift",
   Callback = function(Value)
		AutoGift = Value
   end
})

local SettingsSection = FarmTab:CreateSection("NPCs Options")
-- FAST NPCS
local Original_WalkThroughWaypoints = Entity.WalkThroughWaypoints
local FastNPC = false
local NPCSpeed = 100
Entity.WalkThroughWaypoints = function(entity, voxelpoints, waypoints, undefined1, undefined2)
	if entity:BelongsToMyBakery() then
		if FastNPC and entity.humanoid then 
			entity.humanoid.WalkSpeed = NPCSpeed
		elseif not FastNPC and entity.humanoid and entity.data and entity.data.walkSpeed then
			entity.humanoid.WalkSpeed = entity.data.walkSpeed
		end
	end
	
	Original_WalkThroughWaypoints(entity, voxelpoints, waypoints, undefined1, undefined2)
end

local FastNPCToggle = FarmTab:CreateToggle({
   Name = "Change NPC Walkspeed",
   CurrentValue = false,
   Flag = "FastNPC",
   Callback = function(Value)
		FastNPC = Value
   end
})

local NPCSpeedSlider = FarmTab:CreateSlider({
   Name = "NPC Walkspeed",
   Range = {16, 300},
   Increment = 1,
   Suffix = "Walkspeed",
   CurrentValue = 100,
   Flag = "NPCSpeed",
   Callback = function(Value)
		NPCSpeed = Value
   end,
})

local TeleportTab = Window:CreateTab("Teleport")
local StoreTeleportsSection = TeleportTab:CreateSection("Store")
-- Store Teleports
function TeleportToPosition(position)

	local character = Player.Character or Player.CharacterAdded:Wait()
	local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
	if humanoidRootPart then
		humanoidRootPart.CFrame = position
	end
	
end

local StoreTeleports = {}
function CreateTeleport(teleportName, position) 
	local newButton = TeleportTab:CreateButton({
	   Name = teleportName,
	   Callback = function()
			TeleportToPosition(position)
	   end
	})
	
	newButton.UI.ElementIndicator.Text = "teleport"
	table.insert(StoreTeleports, newButton)
end

CreateTeleport("Global Market", CFrame.new(Vector3.new(-400, 230, 1086)))
CreateTeleport("Appliances", CFrame.new(Vector3.new(-326, 230, 1130)))
CreateTeleport("Furniture", CFrame.new(Vector3.new(-474, 230, 1130)))
CreateTeleport("Floor and Light", CFrame.new(Vector3.new(-492, 255, 1175)))
CreateTeleport("Restaurant Themes", CFrame.new(Vector3.new(-310, 255, 1175)))

local PlayerTeleportsSection = TeleportTab:CreateSection("Player Restaurant")

local OwnBaseTeleport = TeleportTab:CreateButton({
	Name = Player.Name,
	Callback = function() 
	
		local character = Player.Character or Player.CharacterAdded:Wait()
		local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
		if not humanoidRootPart then return end
		local MyBakery = Library.Variables.MyBakery
		local VoxelX, VoxelY, VoxelZ = Bakery.GetCustomerStartVoxel(MyBakery, 1, 1)
		local QueueX, QueueY, QueueZ = Bakery.GetCustomerQueueVoxel(MyBakery, -5, 1)
		local position = MyBakery.floors[1]:WorldPositionFromVoxel(VoxelX, VoxelY, VoxelZ)
		local lookAt = MyBakery.floors[1]:WorldPositionFromVoxel(QueueX, QueueY, QueueZ)
		
		print(MyBakery.baseOrientation)
		--humanoidRootPart.CFrame = CFrame.new((CFrame.new(position + Vector3.new(0, 2, 0)) * CFrame.Angles(0, MyBakery.baseAngle, 0) * CFrame.new(2, 0, 0)).p, (CFrame.new(lookAt) * CFrame.Angles(0, MyBakery.baseAngle, 0) * CFrame.new(2, 0, 0)).p)
		humanoidRootPart.CFrame = CFrame.new(position + Vector3.new(0, 2, 0)) * CFrame.Angles(0, MyBakery.baseAngle, 0) * CFrame.new(2, 0, -10)
		humanoidRootPart.CFrame = CFrame.Angles(0, math.rad(180), 0)
		--CFrame.new((CFrame.new(v233 + Vector3.new(0, 2, 0)) * CFrame.Angles(0, p80.baseAngle, 0) * CFrame.new(2, 0, 0)).p, (CFrame.new(v234) * CFrame.Angles(0, p80.baseAngle, 0) * CFrame.new(2, 0, 0)).p)
	end
})
OwnBaseTeleport.UI.ElementIndicator.Text = "teleport"

local PlayerTeleports = {}
function AddTeleportToPlayerBakery(player) 
	if not player then return end
	if PlayerTeleports[player] then
		RemoveTeleportToPlayerBakery()
	end
	
	PlayerTeleports[player] = TeleportTab:CreateButton({
	   Name = player.Name,
	   Callback = function()
			local playerBakery = Bakery.GetBakeryByOwner(player)
			--Bakery.TeleportToFloor(playerBakery, 1)
			local character = Player.Character or Player.CharacterAdded:Wait()
			local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
			if not humanoidRootPart then return end
			
			local VoxelX, VoxelY, VoxelZ = Bakery.GetCustomerStartVoxel(playerBakery, 1, 1)
			local position = playerBakery.floors[1]:WorldPositionFromVoxel(VoxelX, VoxelY, VoxelZ)
			
			local rotateAngle = 2
			if playerBakery.baseOrientation < 0 then 
				rotateAngle = -2 
			end
			
			humanoidRootPart.CFrame = CFrame.new(position + Vector3.new(0, 2, 0)) * CFrame.Angles(0, playerBakery.baseAngle, 0) * CFrame.new(rotateAngle, 0, -10)
			humanoidRootPart.CFrame *= CFrame.Angles(0, math.rad(180), 0)
	   end
	})
	
	PlayerTeleports[player].UI.ElementIndicator.Text = "teleport"
end


function RemoveTeleportToPlayerBakery(player)
	if PlayerTeleports[player] then
		PlayerTeleports[player]:DestroyMe()
		PlayerTeleports[player] = nil
	end
end

for _, player in pairs(Players:GetPlayers()) do 
	if player ~= Player then
		AddTeleportToPlayerBakery(player)
	end
end

Players.PlayerAdded:Connect(function(player)
	if player ~= Player then
		AddTeleportToPlayerBakery(player)
	end
end)

Players.PlayerRemoving:Connect(function(player) 
	RemoveTeleportToPlayerBakery(player)
end)
