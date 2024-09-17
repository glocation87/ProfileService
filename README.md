# ProfileService
Original - https://github.com/MadStudioRoblox/ProfileService

# What's Different? 
-  Event Listener for Session Table

# Code
```lua
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
```
