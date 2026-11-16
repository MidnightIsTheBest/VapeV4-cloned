-- Requiem: Auto Block
-- Works like Scaffold, but builds a block cage around nearby players
-- instead of under your own feet. Uses the same helpers Scaffold uses
-- (getPlacedBlock, roundPos, getWool, bedwars.placeBlock).
run(function()
	local AutoBlock
	local Range
	local MaxTargets
	local PerTick
	local Head
	local Top
	local Under
	local LimitItem
	local Mouse
	local SkipSelf

	local adjacent = {}
	for x = -3, 3, 3 do
		for y = -3, 3, 3 do
			for z = -3, 3, 3 do
				local vec = Vector3.new(x, y, z)
				if vec ~= Vector3.zero then
					table.insert(adjacent, vec)
				end
			end
		end
	end

	local function checkAdjacent(pos)
		for _, v in adjacent do
			if getPlacedBlock(pos + v) then
				return true
			end
		end
		return false
	end

	local function getBlock()
		if store.hand.toolType == 'block' then
			return store.hand.tool.Name, store.hand.amount
		elseif not LimitItem.Enabled then
			local wool, amount = getWool()
			if wool then
				return wool, amount
			end
			for _, item in store.inventory.inventory.items do
				if bedwars.ItemMeta[item.itemType].block then
					return item.itemType, item.amount
				end
			end
		end
		return nil, 0
	end

	-- cells around a target, in build order (bottom first so every block
	-- has something to attach to)
	local sides = {Vector3.new(3, 0, 0), Vector3.new(-3, 0, 0), Vector3.new(0, 0, 3), Vector3.new(0, 0, -3)}
	local function getCells(root, hip)
		local cells = {}
		-- same math Scaffold uses for the block under your feet
		local ground = roundPos(root.Position - Vector3.new(0, hip + 1.5, 0))
		local feet = ground + Vector3.new(0, 3, 0)

		if Under.Enabled then
			table.insert(cells, ground)
		end
		for _, s in sides do
			table.insert(cells, feet + s)
		end
		if Head.Enabled then
			for _, s in sides do
				table.insert(cells, feet + Vector3.new(0, 3, 0) + s)
			end
		end
		if Top.Enabled then
			table.insert(cells, feet + Vector3.new(0, Head.Enabled and 6 or 3, 0))
		end
		return cells
	end

	AutoBlock = vape.Categories.Utility:CreateModule({
		Name = 'Auto Block',
		Function = function(callback)
			if callback then
				repeat
					if entitylib.isAlive then
						local wool = getBlock()

						if Mouse.Enabled and not inputService:IsMouseButtonPressed(0) then
							wool = nil
						end

						if wool then
							local selfroot = entitylib.character.RootPart
							local selfhip = entitylib.character.HipHeight
							local selfground = roundPos(selfroot.Position - Vector3.new(0, selfhip + 1.5, 0))
							local selffeet = selfground + Vector3.new(0, 3, 0)
							local placed = 0

							local plrs = entitylib.AllPosition({
								Range = Range.Value,
								Part = 'RootPart',
								Players = true,
								NPCs = false,
								Limit = MaxTargets.Value,
								Sort = sortmethods.Distance
							})

							for _, plr in plrs do
								if placed >= PerTick.Value then break end
								for _, cell in getCells(plr.RootPart, plr.HipHeight or 2) do
									if placed >= PerTick.Value then break end

									-- don't wall yourself in when the target is right next to you
									if SkipSelf.Enabled and ((cell - selffeet).Magnitude < 1 or (cell - (selffeet + Vector3.new(0, 3, 0))).Magnitude < 1) then
										continue
									end

									if not getPlacedBlock(cell) and checkAdjacent(cell) then
										task.spawn(bedwars.placeBlock, cell, wool, false)
										placed += 1
									end
								end
							end
						end
					end

					task.wait(0.03)
				until not AutoBlock.Enabled
			end
		end,
		Tooltip = 'Scaffolds a block cage around nearby players.'
	})
	Range = AutoBlock:CreateSlider({
		Name = 'Range',
		Min = 1,
		Max = 20,
		Default = 12,
		Suffix = function(val)
			return val == 1 and 'stud' or 'studs'
		end
	})
	MaxTargets = AutoBlock:CreateSlider({
		Name = 'Max targets',
		Min = 1,
		Max = 5,
		Default = 2
	})
	PerTick = AutoBlock:CreateSlider({
		Name = 'Blocks per tick',
		Min = 1,
		Max = 6,
		Default = 3
	})
	Head = AutoBlock:CreateToggle({
		Name = 'Head level',
		Default = true,
		Tooltip = 'Also wall the second block (head height)'
	})
	Top = AutoBlock:CreateToggle({
		Name = 'Top',
		Default = true,
		Tooltip = 'Cover the block above their head'
	})
	Under = AutoBlock:CreateToggle({
		Name = 'Under',
		Tooltip = 'Fill the block under their feet'
	})
	SkipSelf = AutoBlock:CreateToggle({
		Name = 'Skip own cell',
		Default = true,
		Tooltip = 'Never place a block inside your own spot'
	})
	LimitItem = AutoBlock:CreateToggle({Name = 'Limit to items'})
	Mouse = AutoBlock:CreateToggle({Name = 'Require mouse down'})
end)
