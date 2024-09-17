# ProfileService
Original - https://github.com/MadStudioRoblox/ProfileService

# What's Different? 
-  Event Listener for Session Table

# Code
```lua
-- Fires MadworkSignal instance every time a value is changed
local function MonitorTable(tbl, path, signal)
	local newTable = {}
    local metatable = {
		__index = tbl,

        __newindex = function(_table, key, value)
			local oldValue = _table[key]
			path = path .. "." .. tostring(key) .. " = " .. tostring(value)

			if oldValue ~= value then
				tbl[key] = value
				signal:Fire(tbl, key, value, path)
			end
			
            if type(value) == "table" then
                value = MonitorTable(value, path .. "." .. key)
            end
        end,
		
    }
    for key, value in pairs(tbl) do
        if type(value) == "table" then
            value = MonitorTable(value, path .. "." .. key)
        end
    end

    return setmetatable(newTable, metatable)
end

-- Profile Class can now assign event listener for changes in Profile.Data
function Profile:ListenToDataChange(listener) --> [ScriptConnection] (place_id / nil, game_job_id / nil)
	if type(listener) ~= "function" then
		error("[ProfileService]: Only a function can be set as listener in Profile:ListenToDataChange()")
	end
	if self._view_mode == true then
		return {Disconnect = function() end}
	end

	if self:IsActive() == false then
		return {Disconnect = function() end}
	else
		print("Initialized Data Change Event")
		self.Data = MonitorTable(self.Data, self.UserIds[1], self._data_change_listeners)
		return self._data_change_listeners:Connect(listener)
	end
end

```

# Example Usage

```lua
function ExampleService:InitializeDataListener(player) -- Should only be called once per session
    local profile = self:GetProfileRef(player)
    self.LiveProfiles[player].Events["DataChange"] = profile:ListenToDataChange(function(...) -- 
        print(...)
    end)
end

function DataService:LoadPlayerProfile(player) 
    local ProfileStore = self.Stores["PlayerData"]
    local profile = ProfileStore:LoadProfileAsync(DATA_PREFIX .. player.UserId)
    if (profile) then
        profile:AddUserId(player.UserId)
        profile:Reconcile()
        profile:ListenToRelease(function()
            self.LiveProfiles[player] = nil
            player:Kick()
        end)
        if (player:IsDescendantOf(Players)) then
            self.LiveProfiles[player] = {
                Object = profile,
                Events = {}
            }
            --Initialize PlayerData
            self:InitializeDataListener(player)
        else
            profile:Release()
        end
    else
        player:Kick() 
    end
    return {Completed = function(self, callback) -- Return callback event
        if (profile) then
            callback(profile)
        else
            error "Profile not loaded, returned nil"
        end
    end}
end
```
