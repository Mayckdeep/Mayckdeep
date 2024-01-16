local keepTarget = {}

keepTarget.keyCancel = "Escape"

keepTarget.actualTarget = function()
    for key, func in pairs(g_game) do
        if key:lower():find('getatt') then
            local status, result = pcall(function()
               return func():getId()
            end)
            if status then
                local creature = getCreatureById(result)
                if creature then
                    return creature
                end
            end
        end
    end
end

keepTarget.macro = macro(1, "Attack Target",function()
    if modules.corelib.g_keyboard.isKeyPressed(keepTarget.keyCancel) then
        keepTarget.storageId = nil
        return g_game.cancelAttack()
    end
    local target = keepTarget.actualTarget()
    if target then
        if keepTarget.storageId ~= target:getId() then
            keepTarget.storageId = target:getId()
        end
        return
    else
        if keepTarget.storageId then
            local findCreature = getCreatureById(keepTarget.storageId)
            if findCreature then
                local timeWait = math.random(250, 500)
                schedule(timeWait, function()
                local creaturePos = findCreature:getPosition()
                    if creaturePos and pos().z == creaturePos.z then
                        modules.game_interface.processMouseAction(nil, 2, pos(), nil, findCreature, findCreature)
                    end
                end)
                return delay(timeWait + 250)
            end
        end
    end
end)
