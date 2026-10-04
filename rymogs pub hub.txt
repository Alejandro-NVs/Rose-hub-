local _RP, _RW = print, warn
_G.RyPrivStealWhy = "boot"
_G.RyPrivStealLog = {}
_G.RyPrivStealSay = function(msg)
    -- Fast path for an identical repeat, which is the common case: the steal
    -- tick reports the same reason on every frame for as long as nothing
    -- changes ("steal mode OFF", "no target", "cooldown"). Bumping the counter
    -- is all that is needed, so skip the tostring, the os.clock, the log walk
    -- and the debug branch. Only a message that actually changed does the
    -- full path below.
    if msg == _G.RyPrivStealWhy then
        local L0 = _G.RyPrivStealLog
        local last0 = L0[#L0]
        if last0 then last0.n = last0.n + 1 end
        return
    end
    msg = tostring(msg)
    _G.RyPrivStealWhy = msg
    local L = _G.RyPrivStealLog
    local now = os.clock()
    local last = L[#L]
    if last and last.msg == msg then
        last.n = last.n + 1
        last.t = now
    else
        L[#L + 1] = { msg = msg, t = now, t0 = now, n = 1 }
        while #L > 8 do table.remove(L, 1) end
    end
    if _G.RyPrivStealDebug then
        if now - (_G._mkLastSay or 0) >= (tonumber(_G.RyPrivStealDebugGap) or 0.5) then
            _G._mkLastSay = now
            pcall(_RP, "[steal] " .. msg)
        end
    end
end

_G.RyPrivScanProfile = function(mode)
    if mode == "fast" then
        _G.RyPrivScanMode     = "fast"
        _G.RyPrivPanelScanGap = tonumber(_G.RyPrivFastPanelGap)    or 0.12
        _G.RyPrivRepickGap    = tonumber(_G.RyPrivFastRepickGap)   or 0.10
        _G.RyPrivInRangeGap   = tonumber(_G.RyPrivFastInRangeGap)  or 0.10
        _G.RyPrivWatchdogTick = tonumber(_G.RyPrivFastWatchdogTick) or 0.15
        _G.RyPrivSyncScanGap  = tonumber(_G.RyPrivFastSyncGap)     or 0.05
        _G.RyPrivAutoTPPoll   = tonumber(_G.RyPrivFastTPPoll)      or 0.03
    else
        _G.RyPrivScanMode     = "normal"
        _G.RyPrivPanelScanGap = tonumber(_G.RyPrivNormPanelGap)    or 0.6
        _G.RyPrivRepickGap    = tonumber(_G.RyPrivNormRepickGap)   or 0.5
        _G.RyPrivInRangeGap   = tonumber(_G.RyPrivNormInRangeGap)  or 0.3
        _G.RyPrivWatchdogTick = tonumber(_G.RyPrivNormWatchdogTick) or 0.3
        _G.RyPrivSyncScanGap  = tonumber(_G.RyPrivNormSyncGap)     or 0.25
        _G.RyPrivAutoTPPoll   = tonumber(_G.RyPrivNormTPPoll)      or 0.05
    end
    if _G.RyPrivStealSay then _G.RyPrivStealSay("scan profile -> " .. tostring(_G.RyPrivScanMode)) end
end
_G.RyPrivScanProfile("fast")
task.delay(tonumber(_G.RyPrivFastWindow) or 60, function()
    if _G.RyPrivScanMode == "fast" then _G.RyPrivScanProfile("normal") end
end)
local print = function() end
local warn  = function() end
_G.RyPrivInvisAuto = false
_G.RyPrivAutoKickOnSteal = false
_G.RyPrivAutoBuy = false
if _G.RyPrivStealMode == nil then _G.RyPrivStealMode = "priority" end
-- Teleport movement mode: "normal" is the velocity flight this hub has always
-- used, "fling" is goonpriv's CFrame-tween drag. Guarded so the loader wins.
if _G.RyPrivTPMode == nil then _G.RyPrivTPMode = "normal" end
-- Animations were always suppressed before this was a setting, so the default
-- keeps that rather than changing how the hub behaves for existing users.
if _G.RyPrivDisableAnims == nil then _G.RyPrivDisableAnims = true end
-- ESP defaults. Every one is `== nil` guarded so the settings loader, which
-- runs after this, keeps whatever was saved.
if _G.RyPlayerESP       == nil then _G.RyPlayerESP       = false end
if _G.ESP_Stealer       == nil then _G.ESP_Stealer       = false end
if _G.ESP_BaseOwner     == nil then _G.ESP_BaseOwner     = false end
if _G.ESP_Brainrot      == nil then _G.ESP_Brainrot      = false end
if _G.ESP_Trap          == nil then _G.ESP_Trap          = false end
-- Turret ESP has no working backend, so the flag stays off permanently and the
-- hazard pass skips it. There is deliberately no switch for it.
_G.ESP_Turret = false
if _G.ESP_MaxDistance   == nil then _G.ESP_MaxDistance   = 700 end
if _G.ESP_BrainrotTop   == nil then _G.ESP_BrainrotTop   = 5 end
if _G.ESP_BoxSize       == nil then _G.ESP_BoxSize       = 4.5 end
if _G.ESP_BoxTransp     == nil then _G.ESP_BoxTransp     = 0.5 end
-- INSTANT STEAL: the prompt-burst engine ported from the pub method. Off by
-- default; Base Camp turns it on for you and puts it back off when camp ends.
if _G.RyPrivInstantSteal == nil then _G.RyPrivInstantSteal = false end
if _G.RyPrivAutoTP == nil then _G.RyPrivAutoTP = true end
-- How see-through every panel in the hub is. The Utility panel's Settings
-- popout drives this, and each panel reads it on its repaint tick.
if _G.RyPrivAlpha == nil then _G.RyPrivAlpha = 0.12 end
-- Anti-ragdoll recovery; the Invis panel's AUTO RECOVER row.
if _G.RyPrivAntiRagdoll == nil then _G.RyPrivAntiRagdoll = true end

-- TP safety: prefer a slightly slower, more stable route over hard misses/deaths.
if _G.RyPrivTPSafeMode == nil then _G.RyPrivTPSafeMode = true end
if _G.RyPrivTPSafeSpeed == nil then _G.RyPrivTPSafeSpeed = 300 end
if _G.RyPrivTPSafeVerticalSpeed == nil then _G.RyPrivTPSafeVerticalSpeed = 110 end
if _G.RyPrivTPReachTolerance == nil then _G.RyPrivTPReachTolerance = 4.5 end
if _G.RyPrivTPRetryCount == nil then _G.RyPrivTPRetryCount = 2 end
if _G.RyPrivTPRetryDelay == nil then _G.RyPrivTPRetryDelay = 0.08 end
if _G.RyPrivVoidRecover == nil then _G.RyPrivVoidRecover = true end

if not game:IsLoaded() then game.Loaded:Wait() end

--[[ Net env-spoof: DISABLED ----------------------------------------------------
     This used to require ReplicatedStorage.Packages.Net and then write into the
     game module's own environment:
         env.getfenv                  = <our function>
         setreadonly(env.debug, false)
         env.debug.getmemorycategory  = function() return "--" end
     re-applied 80x over 8 seconds.

     That is the loudest signal in this script, and it is self-defeating. The
     Net wrapper calls debug.getmemorycategory() to identify its caller. Legit
     game code returns a real script category; this returned the literal "--",
     which is not a valid category -- so anything validating the returned value
     against a known set sees "--" and flags immediately. Leaving env.debug
     writable via setreadonly is independently checkable (table.isfrozen).

     RyPriv doesn't need it: the grapple now resolves the UseItem
     RemoteEvent structurally and calls FireServer on the raw Instance, which
     never enters the Net wrapper.

     Set _G.RyPrivNetSpoofEnable = true to restore the old behaviour.
--------------------------------------------------------------------------- ]]
do
    local RS = game:GetService("ReplicatedStorage")
    local _spoofOk = false
    local function _applySpoof()
        if _G.RyPrivNetSpoofEnable ~= true then return false end
        local ok = pcall(function()
            local Net = require(RS:WaitForChild("Packages"):WaitForChild("Net"))
            local og = getfenv
            if type(og) ~= "function" then return end
            for _, fname in ipairs({ "RemoteEvent", "RemoteFunction", "UnreliableRemoteEvent" }) do
                local f = rawget(Net, fname)
                if type(f) == "function" then
                    local env = og(f)
                    if type(env) == "table" then
                        env.getfenv = function() return og(f) end
                        if type(env.debug) == "table" then
                            if setreadonly then pcall(setreadonly, env.debug, false) end
                            env.debug.getmemorycategory = function() return "--" end
                        end
                    end
                end
            end
            _spoofOk = true
        end)
        return ok and _spoofOk
    end
    _G.RyPrivNetSpoof = _applySpoof
    _G.RyPrivNetSpoofReady = function() return _spoofOk end
    if _G.RyPrivNetSpoofEnable == true then
        _applySpoof()
        task.spawn(function()
            for _ = 1, 80 do
                if _applySpoof() then end
                task.wait(0.1)
            end
        end)
    end
end

pcall(function() if setfpscap then setfpscap(9999) end end)


task.spawn(function()
    local Workspace = game:GetService("Workspace")
    local LocalPlayer = game:GetService("Players").LocalPlayer
    if not Workspace.StreamingEnabled then return end
    local plots
    local t0 = os.clock()
    repeat plots = Workspace:FindFirstChild("Plots"); if not plots then task.wait(0.1) end
    until plots or (os.clock() - t0) > 25
    if not plots then return end
    local function plotPos(plot)
        local ok, pv = pcall(function() return plot:GetPivot().Position end)
        if ok and pv and pv.Magnitude > 1 then return pv end
        if plot.PrimaryPart then return plot.PrimaryPart.Position end
        local bp = plot:FindFirstChildWhichIsA("BasePart", true)
        return bp and bp.Position or nil
    end
    local pending = 0
    for _, plot in ipairs(plots:GetChildren()) do
        local pos = plotPos(plot)
        if pos then
            pending = pending + 1
            task.spawn(function()
                pcall(function() LocalPlayer:RequestStreamAroundAsync(pos) end)
                pending = pending - 1
            end)
        end
    end
    local sw = os.clock()
    while pending > 0 and os.clock() - sw < 10 do task.wait(0.05) end
end)

_G.RyPrivBootDelay = tonumber(_G.RyPrivBootDelay) or 6
if _G.RyPrivWaitForTools == nil then _G.RyPrivWaitForTools = true end
_G.RyPrivToolWait = tonumber(_G.RyPrivToolWait) or 35
do
    local _bootT0 = os.clock()
    local _BOOT_TOOLS = {
        "Flying Carpet", "Waverider", "Santa's Sleigh", "Witch's Broom", "Cupid's Wings",
        "Grapple Hook", "Grappling Hook", "Grapple", "Hook", "Web Slinger", "Grapple Gun", "GrappleHook",
    }
    local function _hasTool(n)
        local plr = game:GetService("Players").LocalPlayer
        if not plr then return false end
        local char = plr.Character
        local bp = plr:FindFirstChild("Backpack")
        local t = (char and char:FindFirstChild(n)) or (bp and bp:FindFirstChild(n))
        return t ~= nil and t:IsA("Tool")
    end
    local function _toolsReady()
        if type(_G.RyPrivCarpetTool) == "string" and _G.RyPrivCarpetTool ~= "" and _hasTool(_G.RyPrivCarpetTool) then return true end
        for _, n in ipairs(_BOOT_TOOLS) do
            if _hasTool(n) then return true end
        end
        return false
    end
    _G.RyPrivToolsReady = _toolsReady
    _G.RyPrivBootWait = function()
        local d = tonumber(_G.RyPrivBootDelay) or 8
        while os.clock() - _bootT0 < d do task.wait(0.25) end
        if _G.RyPrivWaitForTools == false then return end
        local cap = tonumber(_G.RyPrivToolWait) or 30
        local t0 = os.clock()
        while os.clock() - t0 < cap do
            local ok, ready = pcall(_toolsReady)
            if ok and ready then
                task.wait(0.35)
                return
            end
            task.wait(0.2)
        end
        warn("[Rymogs priv] tools never loaded within " .. cap .. "s, continuing anyway")
    end

    -- ── UI GATE ──────────────────────────────────────────────────────────
    -- Panels do not need a finished world or a loaded backpack to be drawn --
    -- only the engines do. Several windows were gated on RyPrivBootWait, so
    -- they sat invisible for the whole 6s boot delay plus however long the
    -- tool wait took, which is why parts of the hub turned up seconds after
    -- the rest. All a panel actually needs is somewhere to parent itself.
    _G.RyPrivUIWait = function()
        local plr = game:GetService("Players").LocalPlayer
        if not plr then return end
        if plr:FindFirstChildOfClass("PlayerGui") then return end
        local t0 = os.clock()
        while not plr:FindFirstChildOfClass("PlayerGui") and os.clock() - t0 < 10 do
            task.wait(0.05)
        end
    end
end

_G.RyPrivPriVersion = _G.RyPrivPriVersion or 0
if type(_G.RyPrivPriorityDefault) ~= "table" then _G.RyPrivPriorityDefault = {} end
if type(_G.SHARED_PRIORITY_ITEMS) ~= "table" then _G.SHARED_PRIORITY_ITEMS = {} end

if LPH_OBFUSCATED == nil then
    local env = getfenv()
    env["LPH_NO_" .. "VIRTUALIZE"] = function(...) return ... end
    env["LPH_JIT_" .. "MAX"]       = function(...) return ... end
end

do
    local _HS = game:GetService("HttpService")
    local _TS = game:GetService("TeleportService")
    local fileData, tpData
    if readfile then
        -- A pcall here stops a malformed file crashing the load, but it also
        -- meant every setting silently reverted to defaults and the next save
        -- overwrote the file, so whatever was in it was gone for good. Keep a
        -- copy instead: the individual reads below are each type-guarded, so a
        -- single bad VALUE already cannot take the others down with it -- this
        -- covers the file being unparseable as a whole.
        local raw
        pcall(function() raw = readfile("SideTP.json") end)
        if type(raw) == "string" and #raw > 0 then
            local ok, decoded = pcall(function() return _HS:JSONDecode(raw) end)
            if ok and type(decoded) == "table" then
                fileData = decoded
            elseif writefile then
                pcall(function() writefile("SideTP.bad.json", raw) end)
            end
        end
    end
    pcall(function()
        local td = _TS:GetLocalPlayerTeleportData()
        if td and td.SideTP then tpData = td.SideTP end
    end)
    local merged = {}
    if type(tpData) == "table" then for k, v in pairs(tpData) do merged[k] = v end end
    if type(fileData) == "table" then for k, v in pairs(fileData) do merged[k] = v end end

    if type(merged.tpDelay) == "number" then _G._stp_tpDelay = merged.tpDelay end
    if type(merged.tpVelocity) == "number" then _G.TPVelocity = math.clamp(merged.tpVelocity, 200, 750) end
    if type(merged.climbSpeed) == "number" then _G.RyPrivClimb = math.clamp(merged.climbSpeed, 100, 250) end
    if type(merged.cframeSpeed) == "number" then _G.RyPrivCFrameSpeed = math.clamp(merged.cframeSpeed, 100, 900) end
    -- 5..64 is what the walkspeed engine clamps to and what the Invis panel's
    -- slider offers. The old 32 ceiling was a limit of the CFrame-stepping
    -- engine, not of the game; the velocity engine backs itself off instead.
    if type(merged.walkSpeed) == "number" then _G.RyPrivWalkSpeed = math.clamp(merged.walkSpeed, 5, 64) end
    if type(merged.carpetTool) == "string" then _G.RyPrivCarpetTool = merged.carpetTool end
    if type(merged.landingDelay) == "number" then _G.LandingDelay = math.clamp(merged.landingDelay, 0.05, 0.75) end
    if type(merged.closeSpeed) == "number" then _G.RyPrivCloseSpeed = math.clamp(merged.closeSpeed, 20, 400) end
    if type(merged.tpSafeMode) == "boolean" then _G.RyPrivTPSafeMode = merged.tpSafeMode end
    if type(merged.tpSafeSpeed) == "number" then _G.RyPrivTPSafeSpeed = math.clamp(merged.tpSafeSpeed, 180, 400) end
    if type(merged.tpSafeVerticalSpeed) == "number" then _G.RyPrivTPSafeVerticalSpeed = math.clamp(merged.tpSafeVerticalSpeed, 60, 160) end
    if type(merged.tpRetryCount) == "number" then _G.RyPrivTPRetryCount = math.clamp(merged.tpRetryCount, 0, 4) end
    if type(merged.tpKey) == "string" then _G._stp_tpKeyName = merged.tpKey end
    if type(merged.nearestKey) == "string" then _G.RyPrivNearestKey = merged.nearestKey end
    if type(merged.prioritySoundID) == "string" then _G.RyPrivPrioritySoundID = merged.prioritySoundID end
    -- This forced "priority" on every load, so the Steal Nearest / Steal
    -- Priority switches were saved and then thrown away at startup. Honour
    -- what was saved; fall back to priority only when there is nothing.
    if merged.stealMode == "nearest" or merged.stealMode == "priority" then
        _G.RyPrivStealMode = merged.stealMode
    elseif _G.RyPrivStealMode == nil then
        _G.RyPrivStealMode = "priority"
    end
    if type(merged.disableAnims) == "boolean" then _G.RyPrivDisableAnims = merged.disableAnims end
    if merged.tpMode == "fling" or merged.tpMode == "normal" then
        _G.RyPrivTPMode = merged.tpMode
    end
    if type(merged.instantSteal) == "boolean" then _G.RyPrivInstantSteal = merged.instantSteal end
    if type(merged.instantKey)   == "string"  then _G.RyPrivInstantKeyName = merged.instantKey end
    _G._stealUserOff = false
    if type(merged.priorityList) == "table" then
        local clean = {}
        for _, v in ipairs(merged.priorityList) do
            if type(v) == "string" and v ~= "" then clean[#clean + 1] = v end
        end
        if #clean > 0 then
            local L = _G.SHARED_PRIORITY_ITEMS
            table.clear(L)
            for i = 1, #clean do L[i] = clean[i] end
            _G.RyPrivPriVersion = _G.RyPrivPriVersion + 1
        end
    end
    if type(merged.priorityDefault) == "table" then
        local d = {}
        for _, v in ipairs(merged.priorityDefault) do
            if type(v) == "string" and v ~= "" then d[#d + 1] = v end
        end
        if #d > 0 then _G.RyPrivPriorityDefault = d end
    end
    if type(merged.invisAuto) == "boolean" then _G.RyPrivInvisAuto = merged.invisAuto end
    if type(merged.invisDepth) == "number" then _G.RyPrivInvisDepth = math.clamp(merged.invisDepth, 0, 15) end
    if type(merged.dropView) == "boolean" then _G.RyPrivDropView = merged.dropView end
    if type(merged.invisAngle) == "number" then _G.RyPrivInvisAngle = math.clamp(merged.invisAngle, 0, 360) end
    if type(merged.autoTp) == "boolean" then _G.RyPrivAutoTP = merged.autoTp end
    if type(merged.autoBuy) == "boolean" then _G.RyPrivAutoBuy = merged.autoBuy end
    if type(merged.autoBuyRange) == "number" then _G.RyPrivAutoBuyRange = math.clamp(merged.autoBuyRange, 5, 40) end
    if type(merged.autoBuyHover) == "number" then _G.RyPrivAutoBuyHover = math.clamp(merged.autoBuyHover, 0, 20) end
    if type(merged.tgtX) == "number" then _G._ryPriv_tgtX = merged.tgtX end
    if type(merged.tgtY) == "number" then _G._ryPriv_tgtY = merged.tgtY end
    if type(merged.tgtW) == "number" then _G._ryPriv_tgtW = merged.tgtW end
    if type(merged.adW)  == "number" then _G._ryPriv_adW  = merged.adW  end
    if type(merged.adH)  == "number" then _G._ryPriv_adH  = merged.adH  end
    if type(merged.autoKickOnSteal) == "boolean" then _G.RyPrivAutoKickOnSteal     = merged.autoKickOnSteal end
    if type(merged.resetKey)        == "string"  then _G.RyPrivResetKeyName        = merged.resetKey end
    if type(merged.cloneKey)        == "string"  then _G.RyPrivCloneKeyName        = merged.cloneKey end
    if type(merged.carpetSpeedKey)  == "string"  then _G.RyPrivCarpetSpeedKeyName  = merged.carpetSpeedKey end
    if type(merged.kickKey)         == "string"  then _G.RyPrivKickKeyName         = merged.kickKey end
    if type(merged.stopTpKey)       == "string"  then _G.RyPrivStopTPKeyName       = merged.stopTpKey end
    if type(merged.dropKey)         == "string"  then _G.RyPrivDropKeyName         = merged.dropKey end
    if type(merged.goSpeed)      == "number"  then _G.RyPrivGoSpeed            = math.clamp(merged.goSpeed, 80, 600) end
    if type(merged.kickToPS)     == "boolean" then _G.RyPrivKickToPS           = merged.kickToPS end
    if type(merged.psLink)       == "string"  then _G.RyPrivPrivateServerLink  = merged.psLink end
    if type(merged.priAlert)     == "boolean" then _G.RyPrivPriAlert           = merged.priAlert end
    if type(merged.alertSound)   == "string"  then _G.RyPrivAlertSound         = merged.alertSound end
    if type(merged.alertMinGen)  == "number"  then _G.RyPrivAlertMinGen        = merged.alertMinGen end
    if type(merged.walkSpeedOn)  == "boolean" then _G.RyPrivWalkSpeedOn        = merged.walkSpeedOn end
    if type(merged.xray)         == "boolean" then _G.RyPrivXray               = merged.xray end
    if type(merged.espPlayer)    == "boolean" then _G.RyPlayerESP      = merged.espPlayer end
    if type(merged.espStealer)   == "boolean" then _G.ESP_Stealer      = merged.espStealer end
    if type(merged.espBaseOwner) == "boolean" then _G.ESP_BaseOwner    = merged.espBaseOwner end
    if type(merged.espBrainrot)  == "boolean" then _G.ESP_Brainrot     = merged.espBrainrot end
    if type(merged.espBox)       == "boolean" then _G.ESP_BrainrotBox  = merged.espBox end
    if type(merged.espTrap)      == "boolean" then _G.ESP_Trap         = merged.espTrap end
    if type(merged.espDist)      == "number"  then _G.ESP_MaxDistance  = math.clamp(merged.espDist, 100, 2000) end
    if type(merged.espTop)       == "number"  then _G.ESP_BrainrotTop  = math.clamp(merged.espTop, 1, 15) end
    if type(merged.espBoxSize)   == "number"  then _G.ESP_BoxSize      = math.clamp(merged.espBoxSize, 2, 10) end
    if type(merged.espBoxTransp) == "number"  then _G.ESP_BoxTransp    = math.clamp(merged.espBoxTransp, 0, 0.9) end
    if type(merged.antiFlash)    == "boolean" then _G.RyPrivAntiFlash          = merged.antiFlash end
    if type(merged.antiRagdoll)  == "boolean" then _G.RyPrivAntiRagdoll        = merged.antiRagdoll end
    if type(merged.uiAlpha)      == "number"  then _G.RyPrivAlpha              = math.clamp(merged.uiAlpha, 0, 0.9) end
    if type(merged.faceAway)        == "boolean" then _G.RyPrivFaceAway        = merged.faceAway end
    if type(merged.faceAwayNearest) == "boolean" then _G.RyPrivFaceAwayNearest = merged.faceAwayNearest end
    if type(merged.faceAwayDelay)   == "number"  then _G.RyPrivFaceAwayDelay   = merged.faceAwayDelay end
    if type(merged.antiBee)      == "boolean" then _G.RyPrivAntiBee            = merged.antiBee end
    if type(merged.infJump)      == "boolean" then _G.RyPrivInfJump            = merged.infJump end
    if type(merged.antiDie)      == "boolean" then _G.AntiDieDisabled          = not merged.antiDie end
    if type(merged.carpetSpeedValue) == "number" then _G.RyPrivCarpetSpeedValue = merged.carpetSpeedValue end
    if type(merged.sbX) == "number" then _G._ryPriv_sbX = merged.sbX end
    if type(merged.sbY) == "number" then _G._ryPriv_sbY = merged.sbY end
    if type(merged.hdrX) == "number" then _G._ryPriv_hdrX = merged.hdrX end
    if type(merged.hdrY) == "number" then _G._ryPriv_hdrY = merged.hdrY end
    if type(merged.utX) == "number" then _G._ryPriv_utX = merged.utX end
    if type(merged.utY) == "number" then _G._ryPriv_utY = merged.utY end
    if type(merged.ivX) == "number" then _G._ryPriv_ivX = merged.ivX end
    if type(merged.mbX) == "number" then _G._ryPriv_mbX = merged.mbX end
    if type(merged.mbY) == "number" then _G._ryPriv_mbY = merged.mbY end
    if type(merged.ivY) == "number" then _G._ryPriv_ivY = merged.ivY end
    if type(merged.utilKey) == "string" then _G.RyPrivUtilKeyName       = merged.utilKey end
    if type(merged.invKey)  == "string" then _G.RyPrivInvisPanelKeyName = merged.invKey  end
    if type(merged.adminKey) == "string" then _G.RyPrivAdminKeyName = merged.adminKey end
    if type(merged.adX) == "number" then _G._ryPriv_adX = merged.adX end
    if type(merged.adY) == "number" then _G._ryPriv_adY = merged.adY end
    if type(merged.apX) == "number" then _G._ryPriv_apX = merged.apX end
    if type(merged.apY) == "number" then _G._ryPriv_apY = merged.apY end
    if type(merged.exX) == "number" then _G._ryPriv_exX = merged.exX end
    if type(merged.exY) == "number" then _G._ryPriv_exY = merged.exY end
    if type(merged.autoTpKey)  == "string" then _G.RyPrivAutoTPKeyName      = merged.autoTpKey  end
    if type(merged.autoBuyKey) == "string" then _G.RyPrivAutoBuyKeyName     = merged.autoBuyKey end
    if type(merged.cloneNowKey)== "string" then _G.RyPrivCloneNowKey        = merged.cloneNowKey end
    if type(merged.carpetKey)  == "string" then _G.RyPrivCarpetSpeedKeyName = merged.carpetKey  end
    if type(merged.uiHidden)   == "table"  then _G.RyPrivUIHidden           = merged.uiHidden   end

    -- This used to force autoTp on and rewrite the file with autoTp = true, so
    -- the AUTO TP switch came back on every launch no matter what was saved.
    -- merged.autoTp above already applied the saved value; only fill in a
    -- default when the file had nothing to say.
    if _G.RyPrivAutoTP == nil then _G.RyPrivAutoTP = true end
end

local Players    = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS        = game:GetService("UserInputService")
local RS         = game:GetService("ReplicatedStorage")

local LP = Players.LocalPlayer

-- ════════════════════════════════════════════════════════════════════════
-- PANEL POSITION + DRAG  (one implementation, shared by every panel)
-- ════════════════════════════════════════════════════════════════════════
-- There used to be five separate drag blocks plus one shared helper, each a
-- copy of the same logic carrying the same faults:
--   * a permanent UIS.InputChanged connection per panel, never disconnected,
--     so rebuilding a panel stacked another one and they fought each other
--   * a fresh InputObject.Changed connection on every mouse press
--   * release was only noticed through that per-press connection, so letting
--     go outside the panel could leave a drag running
--   * no clamping, so a panel saved on a wide monitor came back off-screen
--   * one of them (the target panel) never called RyPrivSaveSettings at all,
--     so its position was written to _G and then lost on reload
--
-- Positions keep the _G names the settings file already uses (_ryPriv_utX,
-- _ryPriv_adY and so on) so configs saved by earlier versions still load.
--
--   RyPrivApplyPos(frame, keyX, keyY, fallback, opts)  restore, clamped
--   RyPrivPanelDrag(frame, handle, keyX, keyY, opts)   drag, save on release
--
-- opts.keepScale  the frame is positioned partly by scale (the steal bar sits
--                 at 0.5 scale and only its offset is saved); the scale
--                 components are preserved and the clamp works on the
--                 resulting absolute position
-- opts.pinAnchor  a frame that starts centred on the screen; the first drag
--                 converts it to a plain top-left position so it stops
--                 tracking the centre
do
    -- The one drag in flight, if any. Everything below reads this rather than
    -- each panel owning its own input plumbing.
    local active = nil

    local function viewport()
        local cam = workspace.CurrentCamera
        local vp = cam and cam.ViewportSize
        if vp and vp.X > 1 and vp.Y > 1 then return vp end
        return nil
    end

    -- Keep a restored position reachable. A panel saved at x=1800 on a wide
    -- monitor must not come back entirely outside a 1280-wide one, and a
    -- negative y must not put the drag handle above the top edge.
    local function clampAbs(x, y, frame)
        local vp = viewport()
        if not vp then return x, y end
        local sz = frame.AbsoluteSize
        local w = (sz and sz.X > 0) and sz.X or 200
        local h = (sz and sz.Y > 0) and sz.Y or 40
        local keep = 48            -- how much of the panel must stay on screen
        x = math.clamp(x, -(w - math.min(w, keep)), vp.X - math.min(w, keep))
        y = math.clamp(y, 0, math.max(0, vp.Y - math.min(h, keep)))
        return math.floor(x + 0.5), math.floor(y + 0.5)
    end

    -- Restore a saved position, or leave the fallback in place when there is
    -- nothing saved. Returns true when a saved position was applied, which the
    -- status HUD uses to decide whether it is still centre-anchored.
    _G.RyPrivApplyPos = function(frame, keyX, keyY, fallback, opts)
        if not frame then return false end
        opts = opts or {}
        local sx, sy = tonumber(_G[keyX]), tonumber(_G[keyY])
        if sx == nil or sy == nil then
            if fallback then frame.Position = fallback end
            return false
        end
        if opts.pinAnchor then frame.AnchorPoint = Vector2.new(0, 0) end

        local function place()
            local p = frame.Position
            local scx = opts.keepScale and p.X.Scale or 0
            local scy = opts.keepScale and p.Y.Scale or 0
            local vp = viewport()
            local offX = vp and scx * vp.X or 0
            local offY = vp and scy * vp.Y or 0
            -- clamp where the frame actually lands, then back the offset out
            local ax, ay = clampAbs(sx + offX, sy + offY, frame)
            frame.Position = UDim2.new(scx, math.floor(ax - offX + 0.5),
                                       scy, math.floor(ay - offY + 0.5))
        end
        place()
        -- AbsoluteSize is zero until the frame has been rendered once, so the
        -- first clamp is working from a guess. Re-clamp once it is real.
        task.defer(function() if frame.Parent then pcall(place) end end)
        return true
    end

    -- Weak keys: when a panel is destroyed its handle goes with it and the
    -- entry disappears rather than pinning the instance in memory.
    local regs = setmetatable({}, { __mode = "k" })

    _G.RyPrivPanelDrag = function(frame, handle, keyX, keyY, opts)
        if not (frame and handle and keyX and keyY) then return end
        opts = opts or {}

        -- Rebuilding a panel would otherwise leave the previous handlers
        -- attached to the old handle and add a second set to the new one.
        local prev = regs[handle]
        if prev then
            for _, c in ipairs(prev) do pcall(function() c:Disconnect() end) end
        end
        local conns = {}
        regs[handle] = conns

        conns[#conns + 1] = handle.InputBegan:Connect(function(i)
            if i.UserInputType ~= Enum.UserInputType.MouseButton1
                and i.UserInputType ~= Enum.UserInputType.Touch then return end
            if opts.pinAnchor and frame.AnchorPoint.X ~= 0 then
                -- first drag pins it, so it stops tracking the screen centre
                frame.Position = UDim2.fromOffset(frame.AbsolutePosition.X,
                                                  frame.AbsolutePosition.Y)
                frame.AnchorPoint = Vector2.new(0, 0)
            end
            -- base is captured at the press, so the panel does not jump to the
            -- cursor: every move is an offset from where it already was
            active = {
                frame = frame, handle = handle, opts = opts,
                keyX = keyX, keyY = keyY,
                from = i.Position, base = frame.Position, tracked = nil,
            }
        end)

        conns[#conns + 1] = handle.InputChanged:Connect(function(i)
            if i.UserInputType ~= Enum.UserInputType.MouseMovement
                and i.UserInputType ~= Enum.UserInputType.Touch then return end
            if active and active.handle == handle then active.tracked = i end
        end)

        return conns
    end

    -- One mover for every panel, instead of one connection per panel.
    UIS.InputChanged:Connect(function(i)
        local a = active
        if not a or i ~= a.tracked then return end
        if not (a.frame and a.frame.Parent) then active = nil return end
        local d = i.Position - a.from
        local b = a.base
        if a.opts.keepScale then
            a.frame.Position = UDim2.new(b.X.Scale, b.X.Offset + d.X,
                                         b.Y.Scale, b.Y.Offset + d.Y)
        else
            a.frame.Position = UDim2.fromOffset(b.X.Offset + d.X, b.Y.Offset + d.Y)
        end
    end)

    -- Release is handled here rather than through a per-press connection on the
    -- InputObject, so letting go anywhere -- including off the panel or off the
    -- window -- ends the drag and commits the position exactly once.
    UIS.InputEnded:Connect(function(i)
        local a = active
        if not a then return end
        if i.UserInputType ~= Enum.UserInputType.MouseButton1
            and i.UserInputType ~= Enum.UserInputType.Touch then return end
        active = nil
        if not (a.frame and a.frame.Parent) then return end
        local p = a.frame.Position
        _G[a.keyX] = p.X.Offset
        _G[a.keyY] = p.Y.Offset
        pcall(_G.RyPrivSaveSettings)
    end)

    -- Panels restore themselves at build time; this re-clamps everything that
    -- is already on screen when the window is resized, so nothing is stranded
    -- outside the new viewport.
    do
        local cam = workspace.CurrentCamera
        if cam then
            cam:GetPropertyChangedSignal("ViewportSize"):Connect(function()
                for _, e in ipairs(_G.RyPrivPanels or {}) do
                    if e.frame and e.frame.Parent then
                        pcall(_G.RyPrivApplyPos, e.frame, e.keyX, e.keyY, nil, e.opts)
                    end
                end
            end)
        end
    end
end

-- The same treatment for resize grips. Both grips in the hub had their own
-- copy of this, one of which never saved its result anywhere.
--
--   RyPrivPanelResize(frame, grip, keyW, keyH, minW, minH, maxW, maxH)
--
-- keyH may be nil for a panel whose height is driven by its contents; only the
-- width is then tracked. The size is committed once, on release.
do
    local rs = nil
    _G.RyPrivPanelResize = function(frame, grip, keyW, keyH, minW, minH, maxW, maxH)
        if not (frame and grip and keyW) then return end
        minW, minH = minW or 200, minH or 120
        maxW, maxH = maxW or 2000, maxH or 2000
        grip.InputBegan:Connect(function(i)
            if i.UserInputType ~= Enum.UserInputType.MouseButton1
                and i.UserInputType ~= Enum.UserInputType.Touch then return end
            rs = {
                frame = frame, keyW = keyW, keyH = keyH,
                from = i.Position, base = frame.AbsoluteSize,
                minW = minW, minH = minH, maxW = maxW, maxH = maxH,
            }
        end)
    end

    UIS.InputChanged:Connect(function(i)
        local a = rs
        if not a then return end
        if i.UserInputType ~= Enum.UserInputType.MouseMovement
            and i.UserInputType ~= Enum.UserInputType.Touch then return end
        if not (a.frame and a.frame.Parent) then rs = nil return end
        local d = i.Position - a.from
        local w = math.clamp(a.base.X + d.X, a.minW, a.maxW)
        local h = a.keyH and math.clamp(a.base.Y + d.Y, a.minH, a.maxH)
            or a.frame.Size.Y.Offset
        a.frame.Size = UDim2.fromOffset(math.floor(w), math.floor(h))
    end)

    -- Released anywhere, including off the panel: commit once.
    UIS.InputEnded:Connect(function(i)
        local a = rs
        if not a then return end
        if i.UserInputType ~= Enum.UserInputType.MouseButton1
            and i.UserInputType ~= Enum.UserInputType.Touch then return end
        rs = nil
        if not (a.frame and a.frame.Parent) then return end
        _G[a.keyW] = a.frame.Size.X.Offset
        if a.keyH then _G[a.keyH] = a.frame.Size.Y.Offset end
        pcall(_G.RyPrivSaveSettings)
    end)

    -- Restore a saved size. Kept separate from the position restore because a
    -- panel can have one without the other.
    _G.RyPrivApplySize = function(frame, keyW, keyH, minW, minH, maxW, maxH)
        if not frame then return end
        local w = tonumber(_G[keyW])
        local h = keyH and tonumber(_G[keyH]) or nil
        if w == nil and h == nil then return end
        local cur = frame.Size
        frame.Size = UDim2.fromOffset(
            w and math.clamp(w, minW or 200, maxW or 2000) or cur.X.Offset,
            h and math.clamp(h, minH or 120, maxH or 2000) or cur.Y.Offset)
    end
end

-- Every panel that persists a position registers here, so the viewport-resize
-- handler above can find them all without each panel wiring up its own.
_G.RyPrivPanels = _G.RyPrivPanels or {}
_G.RyPrivRegisterPanel = function(frame, keyX, keyY, opts)
    for _, e in ipairs(_G.RyPrivPanels) do
        if e.keyX == keyX then e.frame, e.opts = frame, opts return end
    end
    _G.RyPrivPanels[#_G.RyPrivPanels + 1] =
        { frame = frame, keyX = keyX, keyY = keyY, opts = opts }
end

if _G.RyPrivNoZeroVel == nil then _G.RyPrivNoZeroVel = false end
function _vzOK()
    if _G.RyPrivNoZeroVel == true then return false end
    if _G.RyPrivZeroWhileStealing ~= true and LP:GetAttribute("Stealing") == true then return false end
    return true
end
function _vzL(p)
    if p and _vzOK() then
        p.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
    end
end
function _vzA(p)
    if p and _vzOK() then
        p.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
    end
end

_BLOCKING_MACHINE_TYPES = {
    Fuse     = true,
    Duel     = true,
    Trade    = true,
    Crafting = true,
}
function _RyPrivIsFusing(animalData)
    if type(animalData) ~= "table" then return false end
    local m = animalData.Machine
    if type(m) ~= "table" then return false end
    return _BLOCKING_MACHINE_TYPES[m.Type] == true
end

do
local _netFolder = RS:WaitForChild("Packages"):WaitForChild("Net")
local _net, _spoofed
-- Requiring the Net module is now opt-in. Without it _get() returns nil and
-- callers fall back to structural resolution / the UI path.
local function _getNet()
    if _G.RyPrivNetSpoofEnable ~= true then return nil end
    if _net then return _net end
    local ok, m = pcall(require, _netFolder)
    if ok and type(m) == "table" then _net = m end
    return _net
end
local function _spoofFn(Net, fname)
    local f = rawget(Net, fname)
    if type(f) ~= "function" then return end
    local og = getfenv
    if type(og) ~= "function" then return end
    local env = og(f)
    if type(env) ~= "table" then return end
    env.getfenv = function() return og(f) end
    if type(env.debug) == "table" then
        if setreadonly then pcall(setreadonly, env.debug, false) end
        env.debug.getmemorycategory = function() return "--" end
    end
end
local function _spoof()
    if _G.RyPrivNetSpoofEnable ~= true then return false end
    if _G.RyPrivNetSpoof then return _G.RyPrivNetSpoof() end
    if _spoofed then return true end
    local Net = _getNet()
    if not Net then return false end
    pcall(function()
        _spoofFn(Net, "RemoteEvent")
        _spoofFn(Net, "RemoteFunction")
        _spoofFn(Net, "UnreliableRemoteEvent")
    end)
    _spoofed = true
    return true
end
local _cache = {}
local function _get(name, kind)
    kind = (kind == "RemoteFunction" and "RemoteFunction")
        or (kind == "UnreliableRemoteEvent" and "UnreliableRemoteEvent")
        or "RemoteEvent"
    if type(name) ~= "string" or name == "" then return nil end
    local logical = name:match("^R[EF]/(.+)$") or name:match("^URE/(.+)$") or name
    local ck = kind .. "|" .. logical
    local hit = _cache[ck]
    if hit and hit.Parent then return hit end
    _cache[ck] = nil
    local Net = _getNet()
    if not Net then return nil end
    _spoof()
    local ok, inst = pcall(function()
        if kind == "RemoteFunction" then return Net:RemoteFunction(logical) end
        if kind == "UnreliableRemoteEvent" then return Net:UnreliableRemoteEvent(logical) end
        return Net:RemoteEvent(logical)
    end)
    if ok and typeof(inst) == "Instance" then
        _cache[ck] = inst
        return inst
    end
    return nil
end
_G.RyPrivNet = {
    RemoteEvent = function(_, name) return _get(name, "RemoteEvent") end,
    RemoteFunction = function(_, name) return _get(name, "RemoteFunction") end,
    UnreliableRemoteEvent = function(_, name) return _get(name, "UnreliableRemoteEvent") end,
}
_G.RyPrivGetRemote = _get
_G.Resolve = _get
_G.__secureGetRemote = function(method, name) return _get(name, method) end
do
    local _dummy = Instance.new("RemoteEvent")
    local _rawFire = (clonefunction and clonefunction(_dummy.FireServer)) or _dummy.FireServer
    _G.RawFire = function(name, ...)
        local r = _get(name)
        if not r then return false end
        _rawFire(r, ...)
        return true
    end
end
end

do
local _xchan
local _deepScans, _lastDeep = 0, 0
local RS_SYNC = game:GetService("ReplicatedStorage")
local _mask_sc
local function _getMask()
    if _mask_sc and _mask_sc.Parent then return _mask_sc end
    local c = RS_SYNC:FindFirstChild("Controllers")
    _mask_sc = c and c:FindFirstChild("PlotController")
    return _mask_sc
end
local secure_call = _G.secure_call
if type(secure_call) ~= "function" then
    local LAYERS = 2
    local TEMPLATE do
        local lines = {
            "return function(__renv, __func, ...)",
            "\tsetfenv(0, __renv)",
            ("\tlocal function l%d(...) return __func(...) end"):format(LAYERS),
        }
        for c = LAYERS - 1, 1, -1 do
            lines[#lines + 1] = ("\tlocal function l%d(...) return l%d(...) end"):format(c, c + 1)
        end
        lines[#lines + 1] = "\treturn l1(...)"
        lines[#lines + 1] = "end"
        TEMPLATE = table.concat(lines, string.char(10))
    end
    local _gti = getthreadidentity or get_thread_identity or getidentity
    local _sti = setthreadidentity or set_thread_identity or setidentity
    local _sentinel
    secure_call = function(func, mask, ...)
        if type(func) ~= "function" or typeof(mask) ~= "Instance" then return nil end
        if not (_gti and _sti and getrenv and loadstring and setfenv) then return nil end
        local renv = getrenv()
        if not _sentinel then
            local ok, env = pcall(getsenv, mask)
            if not ok or type(env) ~= "table" then
                env = setmetatable({ script = mask, _G = {}, shared = {} }, { __index = renv, __newindex = renv })
            end
            local loader = loadstring(TEMPLATE, "=" .. mask:GetFullName())
            if not loader then return nil end
            setfenv(loader, env)
            _sentinel = loader()
        end
        local level = _gti()
        local changed = {}
        local okU, snap = pcall(debug.getupvalues, func)
        if okU and type(snap) == "table" then
            for i, v in pairs(snap) do
                if typeof(v) == "Instance" and v:IsA("LuaSourceContainer") then
                    if pcall(debug.setupvalue, func, i, mask) then changed[i] = v end
                end
            end
        end
        _sti(2)
        local co, args = coroutine.create(_sentinel), table.pack(renv, func, ...)
        local response, err
        while true do
            local r = table.pack(coroutine.resume(co, table.unpack(args, 1, args.n)))
            if not r[1] then err = r[2] break end
            if coroutine.status(co) == "dead" then response = table.pack(table.unpack(r, 2, r.n)) break end
            args = table.pack(coroutine.yield(table.unpack(r, 2, r.n)))
        end
        _sti(level)
        for i, orig in pairs(changed) do pcall(debug.setupvalue, func, i, orig) end
        if err or not response then return nil end
        return table.unpack(response, 1, response.n)
    end
    _G.secure_call = secure_call
end
local _syncMod_sc, _nextTry_sc = nil, 0
local function _getSyncMod()
    if _syncMod_sc then return _syncMod_sc end
    local p = RS_SYNC:FindFirstChild("Packages")
    local m = p and p:FindFirstChild("Synchronizer")
    if not m then return nil end
    local ok, mod = pcall(require, m)
    if ok and type(mod) == "table" then _syncMod_sc = mod end
    return _syncMod_sc
end
_G.__secureChans = function()
    if os.clock() < _nextTry_sc then return _xchan end
    local sync = _getSyncMod()
    if not sync then _nextTry_sc = os.clock() + 0.1; return _xchan end
    if type(sync.GetAllChannels) ~= "function" then _nextTry_sc = os.clock() + 0.1; return _xchan end
    local mask = _getMask()
    if not mask then _nextTry_sc = os.clock() + 0.1; return _xchan end
    local ok, reg = pcall(secure_call, sync.GetAllChannels, mask)
    if ok and type(reg) == "table" then
        _xchan = reg; _nextTry_sc = os.clock() + 1.5
        _G.RyPrivSyncDiag = "GetAllChannels (secure_call) - undetect/instant"
    else
        _nextTry_sc = os.clock() + 0.1
    end
    return _xchan
end

local function _channelCount(t)
    if type(t) ~= "table" then return 0 end
    local ok, hits = pcall(function()
        local h, n = 0, 0
        for _, v in next, t do
            n = n + 1
            if type(v) == "table" and type(rawget(v, "CacheTable")) == "table" then
                h = h + 1
            end
            
            if n >= 50 then break end
        end
        return h
    end)
    return (ok and hits) or 0
end


local function _probe(mod)
    local gu = (debug and debug.getupvalue) or getupvalue
    if type(gu) ~= "function" then return nil, 0, nil end
    local best, bestN, where = nil, 0, nil
    local function consider(t, tag)
        local n = _channelCount(t)
        if n > bestN then best, bestN, where = t, n, tag end
    end
    local cands = {
        { mod.Get, 10, 4, "Get/10/4" },
        { mod.GetAllChannels, 10, 1, "GetAll/10/1" },
        { mod.Wait, 10, 1, "Wait/10/1" },
    }
    for _, c in ipairs(cands) do
        if type(c[1]) == "function" then
            local o, u = pcall(gu, c[1], c[2])
            if o and type(u) == "table" then
                consider(rawget(u, c[3]), c[4])
                if bestN > 0 then return best, bestN, where end
            end
        end
    end
    for _, fn in ipairs({ mod.Get, mod.GetAllChannels, mod.Wait, mod.WaitAndCall, mod.GetTableFromChannel }) do
        if type(fn) == "function" then
            for i = 8, 12 do
                local o, u = pcall(gu, fn, i)
                if o and type(u) == "table" then
                    consider(u, "up" .. i)
                    for j = 1, 4 do consider(rawget(u, j), "up" .. i .. "/" .. j) end
                    if bestN > 0 then return best, bestN, where end
                end
            end
        end
    end
    local ok, up = pcall(gu, mod.Get, 9)
    if ok and type(up) == "table" then
        consider(rawget(up, 3), "Get/9/3")
        if bestN > 0 then return best, bestN, where end
        consider(up, "Get/9")
        if bestN > 0 then return best, bestN, where end
    end
    return best, bestN, where
end

local function _deepScan(mod)
    local gu = (debug and debug.getupvalue) or getupvalue
    if type(gu) ~= "function" then return nil, 0, nil end
    local best, bestN, where = nil, 0, nil
    local function consider(t, tag)
        local n = _channelCount(t)
        if n > bestN then best, bestN, where = t, n, tag end
    end
    for fname, fn in next, mod do
        if type(fn) == "function" then
            for i = 1, 24 do
                local o, u = pcall(gu, fn, i)
                if not o then break end
                if type(u) == "table" then
                    consider(u, tostring(fname) .. "/" .. i)
                    for j = 1, 4 do
                        consider(rawget(u, j), tostring(fname) .. "/" .. i .. "/" .. j)
                    end
                end
            end
        end
    end
    return best, bestN, where
end

local _gcFound, _gcFoundN = nil, 0

local function _gcScanOnce()
    if type(getgc) ~= "function" then return end
    local plots = workspace:FindFirstChild("Plots")
    if not plots then return end
    local live, nLive = {}, 0
    for _, p in ipairs(plots:GetChildren()) do live[p.Name] = true; nLive = nLive + 1 end
    if nLive == 0 then return end

    local best, bestN = nil, 0
    pcall(function()
        local gc = getgc(true)
        for i = 1, #gc do
            local t = gc[i]
            if type(t) == "table" then
                pcall(function()
                    
                    local pk = 0
                    for k in next, t do
                        if type(k) == "string" and live[k] then pk = pk + 1; if pk >= 2 then break end end
                    end
                    if pk < 2 then return end
                    local hits, seen = 0, 0
                    for k, v in next, t do
                        seen = seen + 1
                        if type(k) == "string" and live[k]
                            and type(v) == "table" and type(rawget(v, "CacheTable")) == "table" then
                            hits = hits + 1
                        end
                        if seen >= 64 then break end
                    end
                    if hits > bestN and hits >= 2 then best, bestN = t, hits end
                end)
                if bestN >= nLive then break end
            end
        end
    end)
    if best and bestN > _gcFoundN then _gcFound, _gcFoundN = best, bestN end
end

local _gcLastScan = 0
local _gcRescans = 0
local function _gcChans()
    if _gcFound then
        local _pl = workspace:FindFirstChild("Plots")
        local _nLive = _pl and #_pl:GetChildren() or 0
        if _G.RyPrivSyncRescan == true and _gcFoundN < _nLive
            and _gcRescans < (tonumber(_G.RyPrivSyncMaxRescan) or 20)
            and (os.clock() - _gcLastScan) >= (tonumber(_G.RyPrivSyncRescanGap) or 3) then
            _gcLastScan = os.clock()
            _gcRescans = _gcRescans + 1
            _gcScanOnce()
        end
        return _gcFound, _gcFoundN, "getgc/plot-key+CacheTable"
    end
    if os.clock() - _gcLastScan < (tonumber(_G.RyPrivSyncScanGap) or 0.2) then return nil, 0, nil end
    _gcLastScan = os.clock()
    _gcScanOnce()
    if _gcFound then return _gcFound, _gcFoundN, "getgc/plot-key+CacheTable" end
    return nil, 0, nil
end

local function _apiChans(mod)
    if type(mod) ~= "table" then return nil, 0, nil end
    local fn = rawget(mod, "GetAllChannels")
    if type(fn) ~= "function" then return nil, 0, nil end
    local function _accept(reg, tag)
        if type(reg) ~= "table" then return nil, 0, nil end
        local n = _channelCount(reg)
        if n > 0 then return reg, n, tag end
        return nil, 0, nil
    end
    local mask = _getMask()
    if _G.RyPrivAllowSecureSyncCall and mask and type(secure_call) == "function" then
        for _, form in ipairs({ "self", "plain" }) do
            local ok, reg = pcall(function()
                if form == "self" then return secure_call(fn, mask, mod) end
                return secure_call(fn, mask)
            end)
            if ok then
                local r, n, t = _accept(reg, "GetAllChannels/secure_call(" .. form .. ")")
                if r then return r, n, t end
            end
        end
    end
    if _G.RyPrivAllowRawSyncCall then
        for _, form in ipairs({ "self", "plain" }) do
            local ok, reg = pcall(function()
                if form == "self" then return fn(mod) end
                return fn()
            end)
            if ok then
                local r, n, t = _accept(reg, "GetAllChannels/RAW(" .. form .. ")")
                if r then return r, n, t end
            end
        end
    end
    return nil, 0, nil
end

local _xchan2, _nextTry2, _attempts2 = nil, 0, 0
local _xchan2Until = 0
local function _chans()
    if _xchan2 then
        local now = os.clock()
        if now < _xchan2Until then return _xchan2 end
        if _channelCount(_xchan2) > 0 then _xchan2Until = now + 2 return _xchan2 end
    end
    if os.clock() < _nextTry2 then return _xchan2 end
    _attempts2 = _attempts2 + 1

    local best, n, where = _gcChans()

    if (not best or n == 0) and _attempts2 <= 400
        and (_G.RyPrivAllowUpvalueProbe or _G.RyPrivAllowSecureSyncCall or _G.RyPrivAllowRawSyncCall) then
        local mod = _getSyncMod()
        if mod then
            if _G.RyPrivAllowUpvalueProbe then
                best, n, where = _probe(mod)
                if (not best or n == 0) and os.clock() - _lastDeep > 1 then
                    _lastDeep = os.clock()
                    _deepScans = _deepScans + 1
                    best, n, where = _deepScan(mod)
                end
            end
            if not best or n == 0 then best, n, where = _apiChans(mod) end
        end
    end
    if best and n > 0 then
        _xchan2 = best
        _G.RyPrivSyncDiag = string.format("rawget/CacheTable scoring via %s - %d channels", tostring(where), n)
        return _xchan2
    end
    _nextTry2 = os.clock() + 0.1
    return _xchan2
end
_G.__secureChans = _chans

_G.RyPrivSyncAll=function()return _chans()end
_G.RyPrivSyncGet=function(idx)
local t=_chans()
if not t or idx==nil then return nil end
local ok,cd=pcall(rawget,t,idx)
if ok and type(cd)=="table" then return cd end
local ok2,cd2=pcall(function() return t[idx] end)
if ok2 and type(cd2)=="table" then return cd2 end
return nil
end
_G.sProp=function(ch,key)
if type(ch)~="table" or key==nil then return nil end
local ct=rawget(ch,"CacheTable")
if type(ct)~="table" then
local okC,c2=pcall(function() return ch.CacheTable end)
if okC and type(c2)=="table" then ct=c2 end
end
if type(ct)~="table" then return nil end
local v=rawget(ct,key)
if v~=nil then return v end
local okV,v2=pcall(function() return ct[key] end)
if okV then return v2 end
return nil
end
_G._ryPrivRawCT=function(plotName)
local c=_G.RyPrivSyncGet(plotName)
if not c then return nil end
return rawget(c,"CacheTable")
end
local _AD,_MD,_TD
local function _data()
if _AD then return true end
local ok=pcall(function()
local d=game:GetService("ReplicatedStorage"):WaitForChild("Datas")
_AD=require(d:WaitForChild("Animals"))
_MD=require(d:WaitForChild("Mutations"))
_TD=require(d:WaitForChild("Traits"))
end)
return ok and _AD~=nil
end
_G._ryPrivGen=function(index,mutation,traits)
if not _data() then return 0 end
local info=_AD[index]
if not info or not info.Generation then return 0 end
local mult=1
if mutation and mutation~="None" and mutation~="" then
local m=_MD[mutation]
if m and m.Modifier then mult=mult+m.Modifier end
end
if type(traits)=="table" then
for _,tr in ipairs(traits)do
local t=_TD[tr]
if t and t.MultiplierModifier then mult=mult+t.MultiplierModifier end
end
end
return info.Generation*mult
end
_G._ryPrivAnimShim=setmetatable({GetGeneration=function(_,index,mutation,traits)return _G._ryPrivGen(index,mutation,traits)end},{
__index=function(_,k)
local ok,real=pcall(function()return require(game:GetService("ReplicatedStorage"):WaitForChild("Shared"):WaitForChild("Animals"))end)
if ok and type(real)=="table" then return rawget(real,k) end
return nil
end})
_G.RyPriv_GetPlotChannel=function(plotName)return _G.RyPrivSyncGet(plotName)end
_G.RyPriv_GetAllPlots=function()return _G.RyPrivSyncAll() or {} end
_G.RyPriv_GetPlotAnimalList=function(plotName)
local ct=_G._ryPrivRawCT(plotName)
local al=ct and ct.AnimalList
return type(al)=="table" and al or nil
end
end

_G.RyPrivSyncAll  = _G.RyPrivSyncAll
_G.RyPrivSyncGet  = _G.RyPrivSyncGet
_G.RyPrivRawCT    = _G._ryPrivRawCT
_G.RyPrivGen      = _G._ryPrivGen
_G.RyPrivAnimShim = _G._ryPrivAnimShim
_G.stealthGet    = function(n) return _G.RyPrivSyncGet(n) end
_G.SyncInt       = {_cache={},_data=nil}

task.spawn(function() for _=1,150 do if _G.RyPrivSyncAll() then break end task.wait(0.04) end end)

_G.RyPrivGetSyncData = _G.RyPrivGetSyncData or function(plot)
    local plotName = type(plot) == "string" and plot or (plot and plot.Name)
    if not plotName then return nil end
    local Pkgs = game:GetService("ReplicatedStorage"):FindFirstChild("Packages")
    local Sync = Pkgs and Pkgs:FindFirstChild("Synchronizer")
    if not Sync then return nil end
    local okMod, mod = pcall(require, Sync)
    if not okMod or type(mod) ~= "table" then return nil end

    local okT, data = pcall(function() return _G.RyPrivRawCT(plotName) end)
    if okT and type(data) == "table" then return data end

    local okC, ch = pcall(function() return _G.RyPrivSyncGet(plotName) end)
    if okC and ch then
        local synth = { __channel = ch }
        pcall(function() local ct = rawget(ch, "CacheTable"); if type(ct)=="table" then synth.AnimalList = ct.AnimalList; synth.Owner = ct.Owner end end)
        return synth
    end
    return nil
end


_mkNextModTry = 0
function loadModules()
    if AnimalsData then return true end
    if os.clock() < _mkNextModTry then return false end
    _mkNextModTry = os.clock() + (tonumber(_G.RyPrivModRetry) or 3)
    pcall(function()
        local Datas = RS:FindFirstChild("Datas") or RS:WaitForChild("Datas", 5)
        if Datas then
            local a = Datas:FindFirstChild("Animals") or Datas:WaitForChild("Animals", 5)
            if a then AnimalsData = require(a) end
        end
    end)
    AnimalsShared = _G.RyPrivAnimShim
    if not NumberUtils then
        pcall(function()
            local Utils = RS:FindFirstChild("Utils")
            local n = Utils and Utils:FindFirstChild("NumberUtils")
            if n then NumberUtils = require(n) end
        end)
    end
    return AnimalsData ~= nil
end

function loadNet() return false end

function getRemote(method, name)
    return _G.__secureGetRemote(method, name)
end
_G.RyPrivGetRemote = getRemote

--[[ Grapple: rymogs method ---------------------------------------------------
     The remote is gone. Every earlier version of this fired a UseItem
     RemoteEvent -- first by asking a spoofed Packages.Net for it, later by
     resolving it from the Net folder's instance layout. Both break the moment
     the game reorders or rehashes Net, and both leave a remote call with no
     matching client-side action.

     The rymogs method drives the tool instead: fake the aim on PlayerMouse,
     clear the Grapple entry out of Packages.Debounce, then
     firesignal(tool.Activated) and keep the aim pinned for one ping window.
     Nothing to resolve, nothing to go stale.

     _grappleHookWarm / _grappleHookReady are the pre-fire half of it: firing
     into a tool whose Activated has no listeners yet is a silent no-op, so the
     hook state is warmed in the background and checked before every shot.

     Quiet mode still applies: mute the hook's Sounds/Beams, drop the
     server-side FlightPower, and hold velocity down so the pull does not fight
     the TP.
--------------------------------------------------------------------------- ]]

do
local _WSg = game:GetService("Workspace")

local _gMuteCache = setmetatable({}, { __mode = "k" })
local function _gCollectMutes(tool)
    local e = _gMuteCache[tool]
    if e and (os.clock() - e.t) < 8 and tool.Parent then return e end
    local snd, bm = {}, {}
    pcall(function()
        local scanned = 0
        for _, s in ipairs(tool:GetDescendants()) do
            scanned = scanned + 1
            if scanned > 500 then break end
            if s:IsA("Sound") then snd[#snd + 1] = s
            elseif s:IsA("Beam") then bm[#bm + 1] = s end
        end
    end)
    e = { t = os.clock(), snd = snd, bm = bm }
    _gMuteCache[tool] = e
    return e
end

local function _gMuteAndStopPull(tool, hrp)
    if tool and tool.Parent then
        local e = _gCollectMutes(tool)
        for _, s in ipairs(e.snd) do
            if s.Parent then pcall(function() s.Volume = 0; s:Stop() end) end
        end
        for _, b in ipairs(e.bm) do
            if b.Parent then pcall(function() b.Enabled = false; b.Attachment0 = nil end) end
        end
    end
    if hrp and hrp.Parent then
        local fp = hrp:FindFirstChild("FlightPower")
        if fp then pcall(function() fp:Destroy() end) end
        -- never fight velMoveThrough / cframeStepThrough while a TP is running
        if not isTeleporting and not _G.RyPrivTPBusy then
            _vzL(hrp)
            _vzA(hrp)
        end
    end
end

-- one hold loop at a time; re-firing the hook just extends the existing one so
-- the 0.3s in-TP fire cadence cannot stack Heartbeat connections
local _gHoldUntil, _gHoldConn = 0, nil
local function _gHoldNoPull(tool, hrp, seconds)
    _gHoldUntil = os.clock() + seconds
    if _gHoldConn then return end
    local last = 0
    _gHoldConn = RunService.Heartbeat:Connect(function()
        local now = os.clock()
        if now - last >= 0.03 then
            last = now
            _gMuteAndStopPull(tool, hrp)
        end
        if now > _gHoldUntil then
            if _gHoldConn then _gHoldConn:Disconnect(); _gHoldConn = nil end
            _gMuteAndStopPull(tool, hrp)
        end
    end)
end

function _grappleToolReady(tool, waitS)
    if not tool or not tool.Parent then return false end
    local t0 = os.clock()
    while true do
        local ok, en = pcall(function() return tool.Enabled end)
        if not ok or en ~= false then return true end
        if os.clock() - t0 > (tonumber(waitS) or 0) then return false end
        RunService.Heartbeat:Wait()
        if not tool.Parent then return false end
    end
end
_G.RyPrivToolReady = _grappleToolReady

-- ============================================================
-- GRAPPLE PRE-HOOK
--
-- firesignal(tool.Activated) does nothing until the game's own LocalScript has
-- connected to that signal, which happens a few frames after the tool first
-- exists. Waiting for it on the firing path costs up to a second on the first
-- shot after a spawn, so the waiting is done in the background instead and the
-- shot reads a cached answer.
--
-- Nothing here equips or fires anything: a Tool's LocalScript runs while the
-- tool sits in the Backpack, so the connection is observable without touching
-- it. Cache is per character -- a respawn makes the old closures stale -- and
-- expires on RyPrivGrappleHookTTL regardless.
-- ============================================================
_gHookWarm = _gHookWarm or setmetatable({}, { __mode = "k" })
_gHookGen  = _gHookGen or 0

function _grappleHookWarm(tool)
    if not tool then return false end
    local e = _gHookWarm[tool]
    if not e or e.gen ~= _gHookGen or e.ok ~= true then return false end
    return (os.clock() - e.at) <= (tonumber(_G.RyPrivGrappleHookTTL) or 20)
end

function _grappleHookMark(tool, ok)
    if not tool then return end
    _gHookWarm[tool] = { gen = _gHookGen, at = os.clock(), ok = ok and true or false }
end

function _grappleHookReady(tool, cap)
    if _G.RyPrivGrappleHookReady == false or not tool then return true end
    if _grappleHookWarm(tool) then
        _G.RyPrivGrappleHookTook = 0
        return true
    end
    cap = tonumber(cap) or tonumber(_G.RyPrivGrappleHookMax) or 1.5
    if type(getconnections) ~= "function" then
        task.wait(math.min(0.35, cap))
        return true
    end
    local t0 = os.clock()
    while os.clock() - t0 < cap do
        local ok, conns = pcall(getconnections, tool.Activated)
        if ok and conns and #conns > 0 then
            _G.RyPrivGrappleHookTook = os.clock() - t0
            _grappleHookMark(tool, true)
            return true
        end
        task.wait(0.05)
    end
    _G.RyPrivGrappleHookTook = -1
    return false
end
_G.RyPrivGrappleHookReadyWait = _grappleHookReady

-- How long the game's answer to a shot can take: round trip plus a margin.
function _grapplePingWindow()
    local rtt = 0
    pcall(function()
        local v = game:GetService("Stats").Network.ServerStatsItem["Data Ping"]:GetValue()
        if type(v) == "number" then rtt = math.max(rtt, v) end
    end)
    pcall(function()
        local v = LP:GetNetworkPing() * 2000
        if type(v) == "number" then rtt = math.max(rtt, v) end
    end)
    return math.clamp((rtt / 1000) * (tonumber(_G.RyPrivGrapplePingMult) or 1.25) + 0.05,
        0.05, tonumber(_G.RyPrivGrappleHoldMax) or 0.6)
end
_G.RyPrivGrapplePingWindow = _grapplePingWindow

-- One throttled channel for "the grapple did not fire, and here is why". The
-- file shadows `warn` with a no-op, so this goes through the captured one.
-- _G.RyPrivGrappleWarn = false silences it.
function _grappleSay(why)
    _G.RyPrivGrappleWhy = why
    if _G.RyPrivGrappleWarn == false or type(_RW) ~= "function" then return end
    local now = os.clock()
    if now - (tonumber(_G.__ryGrappleWarnAt) or 0) > 3 then
        _G.__ryGrappleWarnAt = now
        _RW("[Rymogs priv] grapple did not fire: " .. tostring(why))
    end
end

function _grappleDo(tool, pos, target, char)
    if not tool or not pos then _grappleSay("no tool or no position") return false end
    if typeof(pos) == "Instance" then pos = pos.Position end
    if typeof(pos) == "CFrame"   then pos = pos.Position end
    if typeof(pos) ~= "Vector3" then
        _grappleSay("bad target position")
        return false
    end
    char = char or LP.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if not hrp then
        _grappleSay("no hrp")
        return false
    end
    -- FIRESIGNAL, not the UseItem remote.
    --
    -- This was r:FireServer(amt, pos) on a resolved UseItem remote. That path
    -- is gone: it depends on finding the right remote by index or structure,
    -- and it breaks whenever the game reorders Packages.Net. The rymogs method
    -- goes through the tool itself -- fake the aim on PlayerMouse, clear the
    -- Debounce, then firesignal(tool.Activated) -- so there is no remote to
    -- resolve and nothing to go stale.
    --
    -- The rymogs original fires with no readiness gate at all. This had one --
    -- "if getconnections(tool.Activated) is empty, return false" -- and that is
    -- a condition that can stop a shot from ever being taken: if that call is
    -- restricted, or the game's listener lives somewhere it cannot see, the
    -- grapple simply never fired and the only symptom was silence.
    --
    -- Give the listeners a short moment to turn up, then fire either way.
    -- hooked == false only when we actually looked and saw none, which is the
    -- one case where firesignal really is a no-op and Activate() is worth a go.
    local hooked = _grappleHookReady(tool, tonumber(_G.RyPrivGrappleHookMax) or 0.4)

    local quiet = (_G.RyPrivGrappleQuiet ~= false)
    if quiet then _gMuteAndStopPull(tool, hrp) end

    local function _killFlightPower()
        task.defer(function()
            if hrp and hrp.Parent then
                local fp = hrp:FindFirstChild("FlightPower")
                if fp then pcall(function() fp:Destroy() end) end
            end
        end)
    end

    local function _settle(ok, why)
        if ok then _G.RyPrivGrappleWhy = why else _grappleSay(why) end
        if quiet then
            _gMuteAndStopPull(tool, hrp)
            _gHoldNoPull(tool, hrp, tonumber(_G.RyPrivGrappleQuietFor) or 0.6)
        end
        if ok then _G.RyPrivLastGrappleAt = os.clock() end
        return ok
    end

    if typeof(firesignal) ~= "function" then
        return _settle(false, "no firesignal on this executor")
    end

    -- PlayerMouse is what the game reads to decide where the hook goes.
    local okPm, pm = pcall(require, RS.Packages.PlayerMouse)
    if not okPm or type(pm) ~= "table" then
        return _settle(false, "PlayerMouse unavailable")
    end
    pcall(function()
        pm.Hit    = CFrame.new(pos)
        pm.Target = workspace.Terrain
    end)

    pcall(function()
        local Deb = require(RS.Packages.Debounce)
        if type(Deb) == "function" and debug and debug.getupvalues then
            for _, uv in ipairs(debug.getupvalues(Deb)) do
                if type(uv) == "table" then
                    for k in pairs(uv) do
                        if type(k) == "string" and (k:find("rapple") or k:find("Grapple")) then
                            uv[k] = nil
                        end
                    end
                end
            end
        end
    end)

    local fired = pcall(firesignal, tool.Activated)
    -- Nothing was listening, so that firesignal went nowhere. Tool:Activate()
    -- is the client API for the same thing; take that shot rather than none.
    if not hooked then
        local okAct = pcall(function() tool:Activate() end)
        fired = fired or okAct
    end
    -- Keep the aim pinned until the game's answer can be back: its own mouse
    -- code rewrites PlayerMouse.Hit every frame, and at high ping the tool
    -- would otherwise read the real mouse instead of the target.
    if fired then
        local pinUntil = os.clock() + _grapplePingWindow()
        local function pin()
            pcall(function()
                pm.Hit    = CFrame.new(pos)
                pm.Target = workspace.Terrain
            end)
        end
        local pinConn
        pinConn = RunService.Heartbeat:Connect(function()
            if os.clock() > pinUntil then
                if pinConn then pinConn:Disconnect() end
                return
            end
            pin()
        end)
        local bindName = "RyPrivGrapplePin" .. tostring(math.random(1, 1000000000))
        pcall(function()
            RunService:BindToRenderStep(bindName, Enum.RenderPriority.Last.Value + 1, function()
                if os.clock() > pinUntil then
                    pcall(function() RunService:UnbindFromRenderStep(bindName) end)
                    return
                end
                pin()
            end)
        end)
    end
    _killFlightPower()
    return _settle(fired == true, fired and "ok(firesignal)" or "firesignal threw")
end
_G.RyPrivGrappleDo = _grappleDo

local _GFAN  = { 9, 10, 11, 12, 13, 14, 16, 18, 21, 24, 28, 32, 36, 40, 45 }
local _GFAN2 = { 11, 14, 18, 24, 32, 42, 48 }
local _GCOMPASS = {
    Vector3.new( 1, 0,  0), Vector3.new(-1, 0,  0),
    Vector3.new( 0, 0,  1), Vector3.new( 0, 0, -1),
    Vector3.new( 0.7, 0,  0.7), Vector3.new( 0.7, 0, -0.7),
    Vector3.new(-0.7, 0,  0.7), Vector3.new(-0.7, 0, -0.7),
}

-- _fireGrapple()            -> pick a target in front / around and hook it
-- _fireGrapple(vec3)        -> aim toward that point, still snapped to real geometry
-- _fireGrapple(vec3, true)  -> hook exactly that point (must be 10-50 studs out)
function _fireGrapple(value, exact)
    local char = LP.Character
    local hrp  = char and char:FindFirstChild("HumanoidRootPart")
    if not hrp then return false end
    if LP:GetAttribute("Stealing") and _G.RyPrivNoGrappleWhileCarrying == true then
        _G.RyPrivGrappleWhy = "refused: carrying a brainrot"
        return false
    end

    local tool = char:FindFirstChild("Grapple Hook") or (findGrapple())
    if tool and tool.Parent ~= char then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not hum then return false end
        pcall(function() hum:EquipTool(tool) end)
        local t0 = os.clock()
        repeat
            RunService.Heartbeat:Wait()
            char = LP.Character
            tool = char and (char:FindFirstChild("Grapple Hook") or (findGrapple()))
        until (tool and tool.Parent == char) or os.clock() - t0 > 0.5
        hrp = char and char:FindFirstChild("HumanoidRootPart")
    end
    if not tool or not hrp then return false end

    local par = RaycastParams.new()
    par.FilterType = Enum.RaycastFilterType.Exclude
    par.FilterDescendantsInstances = { char }

    if exact and typeof(value) == "Vector3" then
        local m = (value - hrp.Position).Magnitude
        if m < 10 or m > 50 then return false end
        local target
        local hit = _WSg:Raycast(hrp.Position, value - hrp.Position, par)
        if hit then target = hit.Instance end
        if not target then
            local okb, parts = pcall(function() return _WSg:GetPartBoundsInRadius(value, 6) end)
            if okb and parts then
                for _, p in ipairs(parts) do
                    if p:IsA("BasePart") and not p:IsDescendantOf(char) then target = p; break end
                end
            end
        end
        if not target then return false end
        return _grappleDo(tool, value, target, char)
    end

    local dest
    if typeof(value) == "Vector3" then dest = value
    elseif typeof(value) == "Instance" and value:IsA("BasePart") then dest = value.Position end

    local dir
    if dest then
        local d = dest - hrp.Position
        dir = Vector3.new(d.X, 0, d.Z)
    end
    if not dir or dir.Magnitude < 1 then
        local lv = hrp.CFrame.LookVector
        dir = Vector3.new(lv.X, 0, lv.Z)
    end
    if dir.Magnitude < 0.01 then return false end
    dir = dir.Unit

    local pos, target, bestM
    for _, d in ipairs(_GFAN) do
        local origin = hrp.Position + dir * d + Vector3.new(0, 3, 0)
        local hit = _WSg:Raycast(origin, Vector3.new(0, -60, 0), par)
        if hit then
            local m = (hit.Position - hrp.Position).Magnitude
            if m >= 10 and m <= 50 and (not bestM or m < bestM) then
                pos, target, bestM = hit.Position, hit.Instance, m
            end
        end
    end
    if not pos then
        for _, d in ipairs(_GFAN2) do
            local hit = _WSg:Raycast(hrp.Position, dir * d, par)
            if hit then
                local m = (hit.Position - hrp.Position).Magnitude
                if m >= 10 and m <= 50 and (not bestM or m < bestM) then
                    pos, target, bestM = hit.Position, hit.Instance, m
                end
            end
        end
    end
    if not pos then
        for _, cd in ipairs(_GCOMPASS) do
            for _, d in ipairs(_GFAN) do
                local origin = hrp.Position + cd * d + Vector3.new(0, 3, 0)
                local hit = _WSg:Raycast(origin, Vector3.new(0, -70, 0), par)
                if hit then
                    local m = (hit.Position - hrp.Position).Magnitude
                    if m >= 10 and m <= 50 and (not bestM or m < bestM) then
                        pos, target, bestM = hit.Position, hit.Instance, m
                    end
                end
            end
        end
    end
    if not pos or not target then
        _grappleSay("no anchor within 10-50 studs")
        return false
    end
    return _grappleDo(tool, pos, target, char)
end
_G.RyPrivFireGrappleBoth = _fireGrapple

function fireGrapple(value, exact)
    local char = LP.Character
    if not char then return false end
    return _fireGrapple(value, exact)
end
_G.RyPrivFireGrapple = fireGrapple
-- Ready means "there is a hook and its Activated has listeners", which is what
-- the firesignal path actually needs. It used to also require a resolved
-- UseItem remote, and that remote no longer exists. Non-blocking, and it does
-- not touch the hook diagnostics -- it only reports.
_G.RyPrivGrappleReady = function()
    local tool = findGrapple()
    if not tool then return false end
    if _grappleHookWarm(tool) then return true end
    if type(getconnections) ~= "function" then return true end
    local ok, conns = pcall(getconnections, tool.Activated)
    return ok and conns ~= nil and #conns > 0
end
end
-- === end grapple ===========================================================

CARPET_SPEED = 280
INBASE_SPEED = 450
SKY_CLONE_WAIT = 0.35
CARPET_NAMES = { "Flying Carpet", "Waverider", "Santa's Sleigh", "Witch's Broom", "Cupid's Wings" }
function findTool(name)
    local char = LP.Character
    local bp = LP:FindFirstChild("Backpack")
    return (char and char:FindFirstChild(name)) or (bp and bp:FindFirstChild(name))
end
GRAPPLE_NAMES = { "Grapple Hook", "Grappling Hook", "Grapple", "Hook", "Web Slinger", "Grapple Gun", "GrappleHook" }
function findGrapple()
    for _, n in ipairs(GRAPPLE_NAMES) do
        local t = findTool(n)
        if t and t:IsA("Tool") then return t, n end
    end
    return nil
end

-- ── GRAPPLE PRE-HOOK WARMER ───────────────────────────────────────────────
-- Does, in the background, the waiting that _grappleHookReady would otherwise
-- do on the firing path, so the first shot after a spawn does not pay for it.
-- Purely observational: it reads getconnections on a tool sitting in the
-- Backpack and writes nothing to the game.
-- _G.RyPrivGrapplePreHook = false turns it off.
task.spawn(function()
    if _G.RyPrivGrapplePreHook == false then return end
    if type(getconnections) ~= "function" then
        _G.RyPrivGrapplePreHookTook = -1
        return
    end
    local function warm()
        local t0  = os.clock()
        local cap = tonumber(_G.RyPrivGrapplePreHookMax) or 25
        while os.clock() - t0 < cap do
            if _G.RyPrivGrapplePreHook == false then return end
            local tool = findGrapple()
            if tool then
                local ok, conns = pcall(getconnections, tool.Activated)
                if ok and conns and #conns > 0 then
                    _grappleHookMark(tool, true)
                    _G.RyPrivGrapplePreHookTook = os.clock() - t0
                    return
                end
            end
            task.wait(tonumber(_G.RyPrivGrapplePreHookPoll) or 0.1)
        end
        _G.RyPrivGrapplePreHookTook = -1
    end
    -- Connected before the first warm(), which can sit in its poll loop for up
    -- to 25s -- a respawn during that window would otherwise go unnoticed.
    LP.CharacterAdded:Connect(function()
        -- the old tool's connections died with the old character
        _gHookGen = _gHookGen + 1
        task.wait(tonumber(_G.RyPrivGrapplePreHookRespawnGap) or 0.3)
        warm()
    end)
    warm()
end)

function listTools()
    local out, char, bp = {}, LP.Character, LP:FindFirstChild("Backpack")
    if char then for _, t in ipairs(char:GetChildren()) do if t:IsA("Tool") then out[#out + 1] = t.Name end end end
    if bp then for _, t in ipairs(bp:GetChildren()) do if t:IsA("Tool") then out[#out + 1] = t.Name end end end
    return table.concat(out, ", ")
end
_lastCarpetName = nil
function equipCarpet()
    local char = LP.Character
    if not char then return nil end
    if _lastCarpetName then
        local t = char:FindFirstChild(_lastCarpetName)
        if t and t.Parent == char then return _lastCarpetName end
    end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not hum then return nil end
    for _, n in ipairs(CARPET_NAMES) do
        local t = findTool(n)
        if t and t:IsA("Tool") then
            if t.Parent ~= char then pcall(function() hum:EquipTool(t) end) end
            _lastCarpetName = n
            return n
        end
    end
    return nil
end
function setCarpetTool(name)
    if type(name) ~= "string" or name == "" then return end
    _G.RyPrivCarpetTool = name
    for i = #CARPET_NAMES, 1, -1 do
        if CARPET_NAMES[i] == name then table.remove(CARPET_NAMES, i) end
    end
    table.insert(CARPET_NAMES, 1, name)
end
_G.RyPrivSetCarpetTool = setCarpetTool
if type(_G.RyPrivCarpetTool) == "string" and _G.RyPrivCarpetTool ~= "" then
    setCarpetTool(_G.RyPrivCarpetTool)
end
_carpetEngaging = false
-- Set to the character we last confirmed has no Grapple Hook, so repeat
-- engages on that life skip the search instead of re-waiting for it.
_noHookFor = nil
function carpetEngage(force)
    if not force then
        local c = LP.Character
        if c then
            for _, n in ipairs(CARPET_NAMES) do
                local t = c:FindFirstChild(n)
                if t and t:IsA("Tool") then
                    _G.TPEngage = "carpet=" .. tostring(n)
                    return n
                end
            end
        end
    end
    if _carpetEngaging then
        local _tw = os.clock()
        repeat RunService.Heartbeat:Wait() until (not _carpetEngaging) or os.clock() - _tw > 6
        local c = LP.Character
        if c then
            for _, n in ipairs(CARPET_NAMES) do
                local t = c:FindFirstChild(n)
                if t and t:IsA("Tool") then return n end
            end
        end
    end
    _carpetEngaging = true

    -- Finding the hook used to be a flat 5s wait, then equipping it another
    -- flat 1.5s -- both burned in full on every engage for anyone who does not
    -- own a Grapple Hook, which is 6.5s of dead time before the carpet even
    -- starts. Wait only while the backpack could still be filling, remember a
    -- miss for this character, and skip the whole grapple leg when there is
    -- nothing to grapple with.
    local hookCap = tonumber(_G.RyPrivGrappleFindMax) or 5
    pcall(function()
        if type(_G.RyPrivToolsReady) == "function" and _G.RyPrivToolsReady() then
            hookCap = tonumber(_G.RyPrivGrappleFindReady) or 0.4
        end
    end)
    if _noHookFor ~= nil and _noHookFor == LP.Character then hookCap = 0 end

    local _t0 = os.clock()
    while not findTool("Grapple Hook") and os.clock() - _t0 < hookCap do
        RunService.Heartbeat:Wait()
    end
    local char = LP.Character
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if not char or not hum then _carpetEngaging = false; return nil end

    local hookTool = findTool("Grapple Hook")
    if not hookTool then
        _noHookFor = char
    else
        _noHookFor = nil
        if not char:FindFirstChild("Grapple Hook") then
            pcall(function() hum:EquipTool(hookTool) end)
        end
        local _te = os.clock()
        while not (LP.Character and LP.Character:FindFirstChild("Grapple Hook"))
            and os.clock() - _te < 1.5 do
            local c = LP.Character
            local h2 = c and c:FindFirstChildOfClass("Humanoid")
            local g = findTool("Grapple Hook")
            if g and h2 then pcall(function() h2:EquipTool(g) end) end
            RunService.Heartbeat:Wait()
        end
        if LP.Character and LP.Character:FindFirstChild("Grapple Hook") then
            _fireGrapple()
        end
        task.wait(0.05)
        local h = LP.Character and LP.Character:FindFirstChildOfClass("Humanoid")
        if h then pcall(function() h:UnequipTools() end) end
        task.wait(0.05)
    end

    local cn
    local _tc = os.clock()
    repeat
        cn = equipCarpet()
        local c = LP.Character
        if cn and c and c:FindFirstChild(cn) then break end
        RunService.Heartbeat:Wait()
    until os.clock() - _tc > 0.5
    _G.TPEngage = "carpet=" .. tostring(cn)
    _carpetEngaging = false
    return cn
end

_G.RyPrivEquipCarpet = equipCarpet
_G.RyPrivCarpetEngaging = function() return _carpetEngaging end

if _G.RyPrivKeepCarpet == nil then _G.RyPrivKeepCarpet = true end
LP.CharacterAdded:Connect(function(c)
    if _G.RyPrivKeepCarpet == false then return end
    task.spawn(function()
        c:WaitForChild("Humanoid", 10)
        task.wait(tonumber(_G.RyPrivCarpetRespawnDelay) or 0.6)
        if _G.RyPrivKeepCarpet == false then return end
        if _carpetEngaging or LP.Character ~= c then return end
        if LP:GetAttribute("Stealing") == true then return end
        for _, n in ipairs(CARPET_NAMES) do
            if c:FindFirstChild(n) then return end
        end
        pcall(equipCarpet)
    end)
end)

PET_PRIORITY_TIERS = {
    [1] = { pets = {"Headless Horseman"}, threshold = 0 },
    [2] = { pets = {"Signore Carapace"}, threshold = 0 },
    [3] = { pets = {"John Pork"}, threshold = 0 },
    [4] = { pets = {"Strawberry Elephant"}, threshold = 0 },
    [5] = { pets = {"Arcadragon"}, threshold = 5e9 },
    [6] = { pets = {"Elefanto Frigo"}, threshold = 10e9 },
    [7] = { pets = {"Meowl"}, threshold = 5e9 },
    [8] = { pets = {"Skibidi Toilet"}, threshold = 5e9 },
    [9] = { pets = {"Love Love Bear"}, threshold = 0 },
    [10] = { pets = {"Antonio"}, threshold = 0 },
    [11] = { pets = {"Pancake and Syrup"}, threshold = 0 },
    [12] = { pets = {"Griffin"}, threshold = 0 },
    [13] = { pets = {"Globa Steppa","La Supreme Combinasion","Fishino Clownino","Dragon Gingerini","Tirilikalika Tirilikalako"}, threshold = 5e9 },
    [14] = { pets = {"Ginger Gerat","Pet"}, threshold = 10e9 },
    [15] = { pets = {"Hydra Bunny","Digi Narwhal","Kalika Bros"}, threshold = 3e9 },
    [16] = { pets = {"Hydra Dragon Cannelloni","Dragon Cannelloni","Bunny and Eggy"}, threshold = 3e9 },
    [17] = { pets = {"Ketupat Bros","Rosey and Teddy","La Casa Boo","Fragola la la"}, threshold = 3e9 },
    [18] = { pets = {"Fragola La La La","Cerberus","Guest 666","Los Hackers"}, threshold = 1e9 },
    [19] = { pets = {"Garama and Madunung","Spooky and Pumpky","Reinito Sleighito","Burguro And Fryuro","Cooki and Milki","Fragrama and Chocrama","La Food Combinasion","Los Amigos","Foxini Lanternini","Capitano Moby","Fortunu and Cashuru","Los Sekolahs","Celestial Pegasus"}, threshold = 750e6 },
    [20] = { pets = {"La Secret Combinasion","Sammyni Fattini","Cloverat Clapat","Popcuru and Fizzuru"}, threshold = 1e9 },
}

TIER_LOOKUP = {}
for tier, data in pairs(PET_PRIORITY_TIERS) do
    for _, name in ipairs(data.pets) do TIER_LOOKUP[name] = tier end
end

LOCKED_TIERS = { [1]=true, [2]=true, [3]=true, [4]=true }

DIRECT_THRESHOLDS = {
    [3] = { [4] = 10e9 },
    [4] = {},
    [5] = { [6] = math.huge },
    [6] = { [9] = math.huge, [10] = math.huge, [12] = 15e9 },
    [10] = { [12] = 20e9 },
    [11] = { [12] = 10e9 },
}

MUTATION_PRIORITY = {
    ["Galaxy"]=1,["Candy"]=1,["Yin Yang"]=1,["YinYang"]=1,["Divine"]=1,
    ["Cursed"]=1,["Lava"]=1,["Radioactive"]=1,["Cyber"]=1,["Rainbow"]=1,["Bloodrot"]=2,
}

MUTATED_BEATS_GRIFFIN = {
    ["Fishino Clownino"]=true,["Globa Steppa"]=true,
    ["La Supreme Combinasion"]=true,["Tirilikalika Tirilikalako"]=true,
}

function getMutPrio(m)
    if not m or m == "" or m == "None" then return 0 end
    if MUTATION_PRIORITY[m] then return MUTATION_PRIORITY[m] end
    local n = tostring(m):lower():gsub("[%s%-_]","")
    if n == "bloodrot" then return 2 end
    if n == "yinyang" or n == "galaxy" or n == "candy" or n == "divine"
        or n == "cursed" or n == "lava" or n == "radioactive" or n == "cyber"
        or n == "rainbow" then return 1 end
    return 0
end

function getCumThreshold(hi, lo)
    if DIRECT_THRESHOLDS[hi] and DIRECT_THRESHOLDS[hi][lo] then return DIRECT_THRESHOLDS[hi][lo] end
    if LOCKED_TIERS[hi] then return math.huge end
    local total = 0
    for t = hi + 1, lo do
        local td = PET_PRIORITY_TIERS[t]
        if td and td.threshold > 0 then total = total + td.threshold end
    end
    return total
end

function _normName(s)
    return tostring(s):lower():gsub("[%s%-_'%.]", "")
end
_priCacheVer, _priCache = -1, {}
function _priLookup()
    local ver = _G.RyPrivPriVersion or 0
    if _priCacheVer ~= ver then
        table.clear(_priCache)
        local plist = _G.SHARED_PRIORITY_ITEMS
        if type(plist) == "table" then
            for i = #plist, 1, -1 do _priCache[_normName(plist[i])] = i end
        end
        _priCacheVer = ver
    end
    return _priCache
end
_G.RyPrivPriLookup = _priLookup
function _priIndexOf(name)
    if not name then return nil end
    return _priLookup()[_normName(name)]
end

function petOutranks(aName, bName, aMut, bMut, aMPS, bMPS)
    local iA = _priIndexOf(aName)
    local iB = _priIndexOf(bName)
    if iA ~= nil and iB ~= nil then return iA < iB end
    if (iA ~= nil) ~= (iB ~= nil) then return iA ~= nil end
    return (aMPS or 0) > (bMPS or 0)
end

function getPlotChannel(plotName)
    local channel
    pcall(function() channel = _G.RyPrivSyncGet(plotName) end)
    return channel
end

function channelGet(channel, key)
    if not channel then return nil end
    local v
    pcall(function()
        local ct = rawget(channel, "CacheTable")
        if type(ct) == "table" then v = ct[key] end
    end)
    if v ~= nil then return v end
    pcall(function()
        local d = rawget(channel, "Data")
        if type(d) == "table" then v = d[key] end
    end)
    if v ~= nil then return v end
    pcall(function() v = rawget(channel, key) end)
    return v
end

function isMyPlot(channel)
    if not channel then return false end
    local owner = channelGet(channel, "Owner")
    if not owner then return false end
    local result = false
    pcall(function()
        if typeof(owner) == "Instance" and owner:IsA("Player") then
            result = owner.UserId == LP.UserId
        elseif type(owner) == "table" and owner.UserId then
            result = owner.UserId == LP.UserId
        elseif typeof(owner) == "Instance" then
            result = owner == LP
        elseif type(owner) == "string" then
            
            result = owner:lower() == LP.Name:lower()
                or owner:lower() == (LP.DisplayName or LP.Name):lower()
        end
    end)
    return result
end

function ownerInGame(channel)
    if not channel then return false end
    local owner = channelGet(channel, "Owner")
    if not owner then return false end
    local inGame = false
    pcall(function()
        if typeof(owner) == "Instance" and owner:IsA("Player") then
            inGame = Players:FindFirstChild(owner.Name) ~= nil
        elseif type(owner) == "number" then
            inGame = Players:GetPlayerByUserId(owner) ~= nil
        elseif type(owner) == "table" and owner.Name then
            inGame = Players:FindFirstChild(tostring(owner.Name)) ~= nil
        elseif typeof(owner) == "Instance" and owner.Name then
            inGame = Players:FindFirstChild(owner.Name) ~= nil
        elseif type(owner) == "string" then
            
            if Players:FindFirstChild(owner) then
                inGame = true
            else
                local lo = owner:lower()
                for _, pl in ipairs(Players:GetPlayers()) do
                    if pl.Name:lower() == lo or (pl.DisplayName or ""):lower() == lo then inGame = true break end
                end
            end
        end
    end)
    return inGame
end

_petModelCache = setmetatable({}, { __mode = "v" })
_genCache = {}
-- read the brainrot's own overhead "$3.5M/s" label when the data lookup comes up empty
local _MPS_SUFFIX = { K = 1e3, M = 1e6, B = 1e9, T = 1e12, Q = 1e15 }
function _worldMPS(plot, slot)
    local podiums = plot and plot:FindFirstChild("AnimalPodiums")
    local podium = podiums and podiums:FindFirstChild(tostring(slot))
    if not podium then return nil end
    local best = nil
    for _, d in ipairs(podium:GetDescendants()) do
        if d:IsA("TextLabel") or d:IsA("TextButton") then
            local txt = d.Text
            if type(txt) == "string" and txt ~= "" then
                local num, suf = txt:match("%$%s*([%d%.]+)%s*([KMBTQ]?)%s*/s")
                if num then
                    local v = tonumber(num)
                    if v then
                        v = v * (_MPS_SUFFIX[suf] or 1)
                        if not best or v > best then best = v end
                    end
                end
            end
        end
    end
    return best
end
function getPetPosition(plot, slot, strict)
    local podiums = plot:FindFirstChild("AnimalPodiums")
    if not podiums then return nil end
    local podium = podiums:FindFirstChild(tostring(slot))
    if not podium then return nil end

    
    local key = plot.Name .. "\0" .. tostring(slot)
    local cached = _petModelCache[key]
    if cached and cached.Parent and cached:IsDescendantOf(podium) then
        local ok, cf = pcall(function() return cached:GetBoundingBox() end)
        if ok then return cf.Position end
    end
    _petModelCache[key] = nil

    for _, desc in ipairs(podium:GetDescendants()) do
        if desc:IsA("Model") and desc.Name ~= "Claim" and desc.Name ~= "Base" and desc.Name ~= "Decorations" then
            if desc:FindFirstChildWhichIsA("MeshPart", true) then
                local ok, cf = pcall(function() return desc:GetBoundingBox() end)
                if ok then
                    _petModelCache[key] = desc
                    return cf.Position
                end
            end
        end
    end

    if strict then return nil end
    local ok, cf = pcall(function() return podium:GetPivot() end)
    if ok then return cf.Position end
    return podium.Position
end

function scanAllPets()
    local pets = {}
    if not loadModules() then return pets end

    local Plots = workspace:FindFirstChild("Plots")
    if not Plots then return pets end

    local _mkSeen, _mkUse, _mkTotal = 0, 0, 0
    local _mkTags = {}
    for _, plot in ipairs(Plots:GetChildren()) do
        _mkTotal = _mkTotal + 1
        local channel = getPlotChannel(plot.Name)
        if not channel then _mkTags[#_mkTags + 1] = "nochan" continue end
        _mkSeen = _mkSeen + 1
        if isMyPlot(channel) then _G.RyPrivMyPlot = plot.Name _mkTags[#_mkTags + 1] = "MINE" continue end
        if _G.RyPrivMyPlot == plot.Name then _mkTags[#_mkTags + 1] = "MINE" continue end
        if not _G.RyPrivMyPlot and channelGet(channel, "Owner") == nil then _mkTags[#_mkTags + 1] = "noowner?" continue end
        if _G.RyPrivRequireOwner ~= false and not ownerInGame(channel) then _mkTags[#_mkTags + 1] = "left" continue end
        _mkUse = _mkUse + 1

        local animalList = channelGet(channel, "AnimalList")
        if not animalList then _mkTags[#_mkTags + 1] = "nolist" continue end
        local _mkPlotN = 0

        for slot, animalData in pairs(animalList) do
            if type(animalData) ~= "table" then continue end
            local animalName = animalData.Index
            if not animalName then continue end
            local animalInfo = AnimalsData and AnimalsData[animalName]
            if not animalInfo and _G.RyPrivRequireKnown == true then continue end
            if _RyPrivIsFusing(animalData) then continue end

            local mutation = animalData.Mutation or "None"

            local genValue
            local _gk = plot.Name .. "\0" .. tostring(slot)
            local _gc = _genCache[_gk]
            if _gc and _gc.n == animalName and _gc.m == animalData.Mutation and _gc.t == animalData.Traits then
                genValue = _gc.g
            else
                genValue = 0
                pcall(function()
                    genValue = AnimalsShared:GetGeneration(animalName, animalData.Mutation, animalData.Traits, nil)
                end)
                if genValue and genValue > 0 then
                    _genCache[_gk] = { n = animalName, m = animalData.Mutation, t = animalData.Traits, g = genValue }
                end
            end
            if (not genValue or genValue <= 0) and _G.RyPrivWorldMPS ~= false then
                local wm = _worldMPS(plot, slot)
                if wm and wm > 0 then genValue = wm end
            end

            local displayName = (animalInfo and animalInfo.DisplayName) or animalName

            local pos = getPetPosition(plot, slot, (_G.RyPrivRequireOwner == false) or (_G.RyPrivRequireModel == true))

            if pos then
                if genValue >= (tonumber(_G.RyPrivMinMPS) or 0) then
                    table.insert(pets, {
                        name = displayName,
                        index = animalName,
                        mps = genValue,
                        mutation = mutation,
                        traits = type(animalData.Traits) == "table" and animalData.Traits or {},
                        position = pos,
                        plot = plot.Name,
                        slot = tostring(slot),
                    })
                    _mkPlotN = _mkPlotN + 1
                end
            end
        end
        _mkTags[#_mkTags + 1] = tostring(_mkPlotN)
    end

    _G.RyPrivScanInfo = string.format("plots %d scanned of %d  pets %d  [%s]  %s", _mkUse, _mkTotal, #pets, table.concat(_mkTags, " "), tostring(_G.RyPrivScanMode))
    local _priLk = _priLookup()
    for _, p in ipairs(pets) do
        p._pri = _priLk[_normName(p.name)] or (p.index and _priLk[_normName(p.index)]) or nil
    end

    local mode = _G.RyPrivStealMode
    if mode == "highest" then
        table.sort(pets, function(a, b) return (a.mps or 0) > (b.mps or 0) end)
        return pets
    end

    table.sort(pets, function(a, b)
        local ia, ib = a._pri, b._pri
        if ia and ib then
            if ia ~= ib then return ia < ib end
            return (a.mps or 0) > (b.mps or 0)
        end
        if (ia ~= nil) ~= (ib ~= nil) then return ia ~= nil end
        return (a.mps or 0) > (b.mps or 0)
    end)

    return pets
end
_G.RyPrivScanAllPets = scanAllPets  

function scanForTP()
    local full = scanAllPets()
    if type(full) ~= "table" then full = {} end
    if _G.RyPrivScanTiered and _G.RyPrivUseTiered ~= false then
        local ok, tiered = pcall(_G.RyPrivScanTiered)
        if ok and type(tiered) == "table" and #tiered > 0 then
            local seen = {}
            for _, p in ipairs(full) do
                if p and p.plot and p.slot ~= nil then
                    seen[tostring(p.plot) .. "_" .. tostring(p.slot)] = true
                end
            end
            for _, p in ipairs(tiered) do
                if p and p.plot and p.slot ~= nil then
                    local uid = tostring(p.plot) .. "_" .. tostring(p.slot)
                    if not seen[uid] then
                        seen[uid] = true
                        full[#full + 1] = p
                    end
                end
            end
        end
    end
    return full
end

function _petUid(p)
    if not p then return nil end
    return tostring(p.plot) .. "_" .. tostring(p.slot)
end
function _pickPetOnFoot(pets, myPos)
    if not pets or #pets == 0 then return nil end
    local best
    if _G.RyPrivStealMode == "nearest" and myPos then
        local bestD = math.huge
        for _, p in ipairs(pets) do
            if not p.conveyor and p.position then
                local d = (p.position - myPos).Magnitude
                if d < bestD then bestD = d; best = p end
            end
        end
    else
        for _, p in ipairs(pets) do
            if not p.conveyor then best = p; break end
        end
    end
    return best or pets[1]
end
function _findTPSyncedPet(pets)
    local uid = _G.RyPrivStealTargetUID
    if type(uid) ~= "string" or uid == "" then return nil end
    for _, p in ipairs(pets) do
        if _petUid(p) == uid then return p end
    end
    return nil
end
function _clearTPSync()
    _G.RyPrivTPSyncActive = false
    _G.RyPrivStealTargetUID = nil
    _G.RyPrivStealTarget = nil
end
function _armTPSync(pet)
    if not pet then return end
    _G.RyPrivStealTargetUID = _petUid(pet)
    _G.RyPrivStealTarget = pet
    _G.RyPrivTPSyncActive = true
    local gen = (_G._RyPrivTPSyncGen or 0) + 1
    _G._RyPrivTPSyncGen = gen
    task.delay(12, function()
        if _G._RyPrivTPSyncGen == gen then _clearTPSync() end
    end)
end
_G.RyPrivClearTPSync = _clearTPSync
function _findStealTarget(pets)
    if not _G.RyPrivTPSyncActive then return nil end
    return _findTPSyncedPet(pets)
end
function _publishStealTarget(pet)
    if not pet then return end
    _G.RyPrivStealTargetUID = _petUid(pet)
    _G.RyPrivStealTarget = pet
end

MK_UPPER = {
    B = {{coord=Vector3.new(-487.921448,16.850713,-75.768013),facing="NORTH"},{coord=Vector3.new(-332.379730,16.850722,-75.762100),facing="NORTH"},{coord=Vector3.new(-487.134918,16.850713,-18.094154),facing="SOUTH"},{coord=Vector3.new(-316.300171,16.850713,-17.845898),facing="SOUTH"}},
    C = {{coord=Vector3.new(-330.765381,16.850713,31.424425),facing="NORTH"},{coord=Vector3.new(-502.989349,16.850713,31.172430),facing="NORTH"},{coord=Vector3.new(-489.077087,16.850713,89.010147),facing="SOUTH"},{coord=Vector3.new(-330.908936,16.850713,88.930145),facing="SOUTH"}},
    D = {{coord=Vector3.new(-331.264893,16.850713,138.209167),facing="NORTH"},{coord=Vector3.new(-487.935181,16.850713,138.026321),facing="NORTH"},{coord=Vector3.new(-487.774933,16.850713,195.882538),facing="SOUTH"},{coord=Vector3.new(-330.799133,16.850575,196.022354),facing="SOUTH"}},
}
MK_LOWER = {
    B = {{coord=Vector3.new(-335.725586,-3.048217,-74.984589),facing="NORTH"},{coord=Vector3.new(-503.214233,-3.048217,-75.043137),facing="NORTH"},{coord=Vector3.new(-483.619385,-3.718430,-18.844337),facing="SOUTH"},{coord=Vector3.new(-316.147095,-3.048218,-18.818844),facing="SOUTH"}},
    C = {{coord=Vector3.new(-335.985413,-3.048218,32.051426),facing="NORTH"},{coord=Vector3.new(-503.277008,-3.048217,31.956175),facing="NORTH"},{coord=Vector3.new(-483.749390,-3.048218,88.147003),facing="SOUTH"},{coord=Vector3.new(-315.793823,-3.048217,88.163979),facing="SOUTH"}},
    D = {{coord=Vector3.new(-335.476654,-3.048218,139.001083),facing="NORTH"},{coord=Vector3.new(-503.710083,-3.048218,138.989883),facing="NORTH"},{coord=Vector3.new(-315.654938,-3.048218,195.302444),facing="SOUTH"},{coord=Vector3.new(-483.859253,-3.048218,195.269043),facing="SOUTH"}},
}
UPPER_Y_THRESHOLD = 7
TALL_PETS = { ["La Secret Combinasion"]=true, ["La Jolly Grande"]=true }
TALL_OFFSET = 3

BASES_LOW = {
    [1] = Vector3.new(-476.52, -2, 220.94090270996094),
    [2] = Vector3.new(-476.52, -2, 113.77315521240234),
    [3] = Vector3.new(-476.52, -2, 6.178487777709961),
    [4] = Vector3.new(-476.52, -2, -101.07275390625),
    [5] = Vector3.new(-342.66, -2, 221.44737243652344),
    [6] = Vector3.new(-342.66, -2, 113.41409301757812),
    [7] = Vector3.new(-342.66, -2, 6.249461650848389),
    [8] = Vector3.new(-342.66, -2, -99.73458862304688),
}
BASES_HIGH = {
    [1] = Vector3.new(-479.51, 18, 220.94090270996094),
    [2] = Vector3.new(-479.51, 18, 113.77315521240234),
    [3] = Vector3.new(-479.51, 18, 6.178487777709961),
    [4] = Vector3.new(-479.51, 18, -101.07275390625),
    [5] = Vector3.new(-339.48, 18, 221.44737243652344),
    [6] = Vector3.new(-339.48, 18, 113.41409301757812),
    [7] = Vector3.new(-339.48, 18, 6.249461650848389),
    [8] = Vector3.new(-339.48, 18, -99.73458862304688),
}
FRONT_Y_LOW   = -3.048217
FRONT_Y_HIGH  = 16.850713
COLUMN_SPLIT_X = -410
FRONT_Z_CLAMP  = 18
SIDE_NEAR_Z    = 45

-- Base-approach tables (used by the front/side picker in doVelocityTP).
-- Front spots come from BASES_LOW / BASES_HIGH above via buildFrontCandidate.
BASE_FACING = { "WEST", "WEST", "WEST", "WEST", "EAST", "EAST", "EAST", "EAST" }
SIDE_TP_HIGH = {
    {plot = 4, coord = Vector3.new(-503.277008, 15.5, -75.768013), facing = "NORTH"},
    {plot = 3, coord = Vector3.new(-503.277008, 15.5, -18.094154), facing = "SOUTH"},
    {plot = 3, coord = Vector3.new(-503.277008, 15.5, 31.172430), facing = "NORTH"},
    {plot = 2, coord = Vector3.new(-503.277008, 15.5, 89.010147), facing = "SOUTH"},
    {plot = 2, coord = Vector3.new(-503.277008, 15.5, 138.026321), facing = "NORTH"},
    {plot = 1, coord = Vector3.new(-503.277008, 15.5, 195.882538), facing = "SOUTH"},
    {plot = 8, coord = Vector3.new(-316.147095, 15.5, -75.762100), facing = "NORTH"},
    {plot = 7, coord = Vector3.new(-316.147095, 15.5, -17.845898), facing = "SOUTH"},
    {plot = 7, coord = Vector3.new(-316.147095, 15.5, 31.424425), facing = "NORTH"},
    {plot = 6, coord = Vector3.new(-316.147095, 15.5, 88.930145), facing = "SOUTH"},
    {plot = 6, coord = Vector3.new(-316.147095, 15.5, 138.209167), facing = "NORTH"},
    {plot = 5, coord = Vector3.new(-316.147095, 15.5, 196.022354), facing = "SOUTH"},
}
SIDE_TP_LOW = {
    {plot = 4, coord = Vector3.new(-503.277008, -5, -74.984589), facing = "NORTH"},
    {plot = 3, coord = Vector3.new(-503.277008, -5, -18.844337), facing = "SOUTH"},
    {plot = 3, coord = Vector3.new(-503.277008, -5, 31.956175), facing = "NORTH"},
    {plot = 2, coord = Vector3.new(-503.277008, -5, 88.147003), facing = "SOUTH"},
    {plot = 2, coord = Vector3.new(-503.277008, -5, 138.989883), facing = "NORTH"},
    {plot = 1, coord = Vector3.new(-503.277008, -5, 195.269043), facing = "SOUTH"},
    {plot = 8, coord = Vector3.new(-316.147095, -5, -75.043137), facing = "NORTH"},
    {plot = 7, coord = Vector3.new(-316.147095, -5, -18.818844), facing = "SOUTH"},
    {plot = 7, coord = Vector3.new(-316.147095, -5, 32.051426), facing = "NORTH"},
    {plot = 6, coord = Vector3.new(-316.147095, -5, 88.163979), facing = "SOUTH"},
    {plot = 6, coord = Vector3.new(-316.147095, -5, 139.001083), facing = "NORTH"},
    {plot = 5, coord = Vector3.new(-316.147095, -5, 195.302444), facing = "SOUTH"},
}
SIDE_X_RANGE = 20

function getClosestBaseIdx(pos)
    local closest, dist = 1, math.huge
    for i = 1, 8 do
        local b = BASES_LOW[i]
        local d = (pos.X - b.X)^2 + (pos.Z - b.Z)^2
        if d < dist then dist = d; closest = i end
    end
    return closest
end

function buildFrontCandidate(idx, isUpper, playerZ)
    local base = isUpper and BASES_HIGH[idx] or BASES_LOW[idx]
    local frontY = isUpper and FRONT_Y_HIGH or FRONT_Y_LOW
    local frontZ = math.clamp(playerZ - base.Z, -FRONT_Z_CLAMP, FRONT_Z_CLAMP) + base.Z
    local coord = Vector3.new(base.X, frontY, frontZ)
    local faceDir = (idx <= 4) and Vector3.new(-1, 0, 0) or Vector3.new(1, 0, 0)
    return coord, faceDir
end

function plotSides(coordTable, idx)
    local base = BASES_LOW[idx]
    local isWest = idx <= 4
    local out = {}
    for _, coords in pairs(coordTable) do
        for _, data in ipairs(coords) do
            if ((data.coord.X < COLUMN_SPLIT_X) == isWest)
               and math.abs(data.coord.Z - base.Z) < SIDE_NEAR_Z then
                out[#out + 1] = data
            end
        end
    end
    return out
end

function _floor1LaserSolid(plotName)
    local solid = false
    pcall(function()
        local Plots = workspace:FindFirstChild("Plots")
        local plot = Plots and Plots:FindFirstChild(plotName)
        if not plot then return end
        for _, d in ipairs(plot:GetDescendants()) do
            if d:IsA("BasePart") and (d.Name == "LaserHitbox" or d.Name == "Laser")
                and d.CanCollide and d.Position.Y <= 9 then
                solid = true
                break
            end
        end
    end)
    return solid
end

function isPlotUnlocked(plotName)
    local ok, res = pcall(function()
        local channel = getPlotChannel(plotName)
        if not channel then return false end
        if channelGet(channel, "BlockEndTimeFirstFloor") ~= nil then return false end
        return not _floor1LaserSolid(plotName)
    end)
    return ok and (res == true)
end

function findClosest(petPos, coordTable)
    local best, bestKey, bestDist = nil, nil, math.huge
    for skyKey, coords in pairs(coordTable) do
        for _, data in ipairs(coords) do
            local c = data.coord
            local d = math.sqrt((petPos.X - c.X)^2 + (petPos.Z - c.Z)^2)
            if d < bestDist then bestDist = d; best = data; bestKey = skyKey end
        end
    end
    return best, bestKey
end

do
_vizParts = {}
_vizGen = 0

function _vizEnsure()
    if _vizFolder and _vizFolder.Parent then return end
    _vizFolder = Instance.new("Folder")
    -- was "RyPrivPathViz": Workspace is readable by the game's own LocalScripts,
    -- so a self-identifying name is a free detection. Cleanup is by reference.
    _vizFolder.Name = "Effects"
    _vizFolder.Parent = workspace
    _vizAnchor = Instance.new("Part")
    _vizAnchor.Name = "Anchor"
    _vizAnchor.Anchored = true; _vizAnchor.CanCollide = false; _vizAnchor.CanQuery = false
    _vizAnchor.CanTouch = false; _vizAnchor.Transparency = 1; _vizAnchor.Size = Vector3.one
    _vizAnchor.CFrame = CFrame.new()
    _vizAnchor.Parent = _vizFolder
end
clearViz = function()
    if _vizFolder then pcall(function() _vizFolder:Destroy() end) end
    _vizFolder, _vizAnchor = nil, nil
    table.clear(_vizParts)
end
local function _neon(cf, size, color, ball, transp)
    _vizEnsure()
    local p = Instance.new("Part")
    p.Anchored = true; p.CanCollide = false; p.CanQuery = false; p.CanTouch = false; p.CastShadow = false
    p.Material = Enum.Material.Neon; p.Color = color
    p.Transparency = transp or 0
    if ball then p.Shape = Enum.PartType.Ball end
    p.Size = size; p.CFrame = cf; p.Parent = _vizFolder
end
function vizLine(a, b, color)
    local d = b - a
    if d.Magnitude < 0.05 then return end
    _neon(CFrame.lookAt((a + b) * 0.5, b), Vector3.new(0.25, 0.25, d.Magnitude), color, false, 0)
end
function vizDot(pos, color, sz)
    _neon(CFrame.new(pos), Vector3.new(sz, sz, sz), color, true, 0)
end
vizPath = function(fromPos, waypoints)
    if _G.RyPrivShowPath ~= true then return end
    if #waypoints == 0 then return end
    local COL = Color3.fromRGB(255, 195, 45)
    local prev = fromPos
    for _, wp in ipairs(waypoints) do
        vizLine(prev, wp, COL)
        prev = wp
    end
    vizDot(waypoints[#waypoints], COL, 1.6)
end
end

MK_SPEED = 125
MK_ARRIVE = 3
-- Was computed from "does getconnections exist", which is not the same
-- question as "did we actually disable the watcher" -- it read true while
-- strip() was returning on its first line, so _climbCap handed out the
-- confident 200 the whole time on the strength of protection that was not
-- there. It now starts false and is set by strip() only once it has really
-- disabled something. RyPrivAssumeStripped = true forces the old behaviour.
_STRIP_OK = false
function _climbCap()
    local v = math.clamp(tonumber(_G.RyPrivClimb) or 200, 100, 250)
    if not (_STRIP_OK or _G.RyPrivAssumeStripped == true) then v = 55 end
    return v
end

function vZero(hrp)
    if hrp then _vzL(hrp); _vzA(hrp) end
end


function _setFlightVel(hrp, vel)
    local char = hrp and hrp.Parent
    local part = (char and (char:FindFirstChild("UpperTorso") or char:FindFirstChild("Torso"))) or hrp
    if part then part.AssemblyLinearVelocity = vel end

    -- FLING MODE.
    --
    -- This is the part that actually makes goonpriv's teleport look like it is
    -- throwing you around, and it is not in the path at all -- the route it
    -- flies is the same one. Every frame the root's ROTATION is overwritten
    -- with a continuous yaw spin, a continuous barrel roll and a pitch
    -- oscillation, so the body tumbles while the velocity carries it. That is
    -- the ragdolling, moving-weird look.
    --
    -- Only the rotation is replaced: CFrame.new(hrp.Position) keeps the
    -- position exactly as the physics left it, so the route, the arrival and
    -- the lagback handling are all unaffected.
    if _G.RyPrivTPMode == "fling" and typeof(vel) == "Vector3" and vel.Magnitude > 5 then
        pcall(function()
            local t = os.clock()
            local amp = math.clamp(tonumber(_G.RyPrivFlingAmp) or 1, 0, 3)
            local spinYaw   = (t * 18.0) % (math.pi * 2)
            local spinRoll  = (t * 16.0) % (math.pi * 2)
            local flipPitch = math.sin(t * 20.0) * math.rad(45) * amp
            hrp.CFrame = CFrame.new(hrp.Position)
                * CFrame.Angles(0, spinYaw, 0)
                * CFrame.Angles(flipPitch, 0, spinRoll)
        end)
    end
end

-- ── FLIGHT ─────────────────────────────────────────────────────────────────
--   * lagback detection: if we moved BACKWARDS against our push, the server
--     rejected the step -> speed backs off (x0.6, floor 35%) and the stall
--     counter resets instead of aborting the run
--   * per-frame step cap (MAX_STEP studs): an FPS dip can't ask for a
--     125-stud jump the server will snap back
--   * corner smoothing: steer toward the next node while closing on this one
--   * unstuck ladder: clear movers -> hop 6 up -> only then give up
--   * finish only snaps if we're actually within 8 studs of the end
--   * time budget that accounts for step cap and lagbacks
-- Also:
--   * carpet re-equip, jump-each-step on long routes, zero-velocity at nodes
--   * climb / descend caps -- but on the VERTICAL component only, so a
--     rising leg no longer drags the horizontal speed down with it
--   * _G.RyPrivTPStop, path viz, _setFlightVel (drives the torso)
-- TP Safe Mode no longer means "cap at 300". On = safety (lagback
-- backoff + step cap 16). Off = raw speed (step cap 24, no backoff).
MAX_STEP_SAFE, MAX_STEP_RAW = 16, 24
LAGBACK_STUDS = 6
SMOOTH_RADIUS = 14

function velMoveThrough(hrp, waypoints, speedOverride, allowJump, quickStart)
    if not hrp or not hrp.Parent or #waypoints == 0 then return end
    local _runSpeed = speedOverride or (_G.TPVelocity and math.clamp(_G.TPVelocity, 200, 750)) or CARPET_SPEED
    local _safe = _G.RyPrivTPSafeMode ~= false
    local _maxStep = tonumber(_G.RyPrivMaxStep) or (_safe and MAX_STEP_SAFE or MAX_STEP_RAW)
    vizPath(hrp.Position, waypoints)
    local wpIdx = 1
    local done = false
    local conn
    local function finish()
        if done then return end
        done = true
        if hrp and hrp.Parent then
            _vzL(hrp)
            _vzA(hrp)
            -- only close the last gap; a long snap is exactly what gets rejected
            if (hrp.Position - waypoints[#waypoints]).Magnitude <= 8 then
                local _, y = hrp.CFrame:ToEulerAnglesYXZ()
                hrp.CFrame = CFrame.new(waypoints[#waypoints]) * CFrame.Angles(0, y, 0)
            end
        end
        if conn then conn:Disconnect() end
    end
    local lastDist, stall, unstuckTried = math.huge, 0, 0
    local speedScale, lagbacks, lagPenalty = 1, 0, 0
    local lastPos, lastDir = hrp.Position, nil
    local _lastJump = 0

    local _routeLen = 0
    do
        local _p = hrp.Position
        for _, wp in ipairs(waypoints) do
            _routeLen = _routeLen + (_p - wp).Magnitude
            _p = wp
        end
    end
    local _mayJump = _routeLen >= (tonumber(_G.RyPrivJumpMinDist) or 100)
    local _ = quickStart

    local function _jump()
        local _jh = hrp.Parent and hrp.Parent:FindFirstChildOfClass("Humanoid")
        if _jh then
            _lastJump = os.clock()
            -- pcall the method directly instead of wrapping each call in its
            -- own closure: this fires several times a second for the whole
            -- flight and allocated two closures every time.
            if pcall(_jh.ChangeState, _jh, Enum.HumanoidStateType.Jumping) then
                _jh.Jump = true
            end
        end
    end
    -- equipCarpet was called on every single frame of the flight. Its fast
    -- path still does a FindFirstChild, and on a miss it walks every carpet
    -- name against both the character and the backpack -- about eleven
    -- lookups a frame. The carpet only needs re-equipping if something took
    -- it away, so a quarter-second cadence is imperceptible in flight and
    -- turns ~60 checks a second into four.
    local _carpetAt = 0

    conn = RunService.Heartbeat:Connect(LPH_NO_VIRTUALIZE(function(dt)
        if not hrp or not hrp.Parent or done then
            if conn then conn:Disconnect() end
            done = true
            return
        end
        if _G.RyPrivTPStop then finish() return end
        do
            local _cn = os.clock()
            if _cn - _carpetAt >= (tonumber(_G.RyPrivCarpetCheckGap) or 0.25) then
                _carpetAt = _cn
                equipCarpet()
            end
        end
        if _mayJump and _G.RyPrivJumpEachStep ~= false
            and os.clock() - _lastJump >= (tonumber(_G.RyPrivJumpGap) or 0.2) then
            _jump()
        end

        -- lagback: moved against our own push = server pulled us back
        local nowPos = hrp.Position
        if _safe and lastDir then
            if (nowPos - lastPos):Dot(lastDir) < -LAGBACK_STUDS then
                lagbacks = lagbacks + 1
                lagPenalty = lagPenalty + 0.6
                speedScale = math.max(0.35, speedScale * 0.6)
                stall, lastDist, unstuckTried = 0, math.huge, 0
            end
        end
        lastPos = nowPos

        local target = waypoints[wpIdx]
        local diff = target - nowPos
        local mag = diff.Magnitude
        local frameDt = (dt and dt > 0) and dt or (1 / 60)
        -- mid-route nodes: arrive radius grows with speed so fast legs don't
        -- orbit a node; last node: tight, finish() closes the rest
        local _arr = (wpIdx == #waypoints) and 0.6
            or math.max(MK_ARRIVE, _runSpeed * speedScale * frameDt * 1.25)
        if mag < _arr then
            wpIdx = wpIdx + 1
            if wpIdx > #waypoints then finish() return end
            lastDist, stall = math.huge, 0
            if _G.RyPrivZeroEachStep ~= false and _vzOK() then
                pcall(function()
                    local _v = hrp.AssemblyLinearVelocity
                    local _keepY = (_G.RyPrivZeroStepKeepY == false) and 0 or math.max(_v.Y, 0)
                    hrp.AssemblyLinearVelocity = Vector3.new(0, _keepY, 0)
                    hrp.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
                end)
            end
            if _mayJump and _G.RyPrivJumpEachStep ~= false then _jump() end
            target = waypoints[wpIdx]
            diff = target - hrp.Position
            mag = diff.Magnitude
        end

        if mag > lastDist - 0.05 then stall = stall + 1 else stall = 0 end
        lastDist = mag
        if stall >= (tonumber(_G.RyPrivStallFrames) or 18) then
            if unstuckTried == 0 then
                unstuckTried = 1
                stall, lastDist = 0, math.huge
                pcall(function()
                    hrp.Anchored = false
                    local h2 = hrp.Parent and hrp.Parent:FindFirstChildOfClass("Humanoid")
                    if h2 then h2.Sit = false; h2.PlatformStand = false end
                    for _, c in ipairs(hrp:GetChildren()) do
                        if c:IsA("LinearVelocity") or c:IsA("BodyVelocity") or c:IsA("BodyGyro")
                            or c:IsA("BodyPosition") or c:IsA("AngularVelocity") or c:IsA("AlignOrientation")
                            or c:IsA("AlignPosition") or c:IsA("BodyAngularVelocity") then
                            c:Destroy()
                        end
                    end
                end)
            elseif unstuckTried == 1 then
                -- most snags are a lip or a doorframe: hop it
                unstuckTried = 2
                stall, lastDist = 0, math.huge
                pcall(function()
                    hrp.CFrame = hrp.CFrame + Vector3.new(0, 6, 0)
                    hrp.AssemblyLinearVelocity = Vector3.new(0, 12, 0)
                end)
            else
                finish() return
            end
        end

        if mag >= 0.1 then
            local dir = diff.Unit
            if (allowJump or diff.Y > 10) and diff.Y > 5 and wpIdx < #waypoints then
                local hum = hrp.Parent and hrp.Parent:FindFirstChildOfClass("Humanoid")
                if hum then
                    local st = hum:GetState()
                    if st ~= Enum.HumanoidStateType.Jumping and st ~= Enum.HumanoidStateType.Freefall then
                        _jump()
                    end
                end
            end
            -- ease into the next leg (corner smoothing)
            local nxt = waypoints[wpIdx + 1]
            if nxt and mag < SMOOTH_RADIUS then
                local d2 = nxt - hrp.Position
                if d2.Magnitude > 0.05 then
                    local blend = 1 - (mag / SMOOTH_RADIUS)
                    local mixed = dir:Lerp(d2.Unit, blend * 0.65)
                    if mixed.Magnitude > 0.05 then dir = mixed.Unit end
                end
            end

            local speed = _runSpeed * speedScale
            -- never overshoot the node this frame, never step more than MAX_STEP
            speed = math.min(speed, mag / frameDt, _maxStep / frameDt)

            -- ry's climb / descend caps. Shallow legs: cap only the vertical
            -- part so horizontal keeps full speed. Steep legs (>30 deg): scale
            -- the whole vector so the path stays straight and can't clip a wall.
            local _vCap = tonumber(_G.RyPrivTPSafeVerticalSpeed) or 110
            local _upCap = math.min(_climbCap(), _vCap)
            local _lim = dir.Y > 0 and _upCap or _vCap
            if math.abs(dir.Y) > 0.5 then
                if math.abs(dir.Y) * speed > _lim then speed = _lim / math.abs(dir.Y) end
            end
            local vx, vy, vz = dir.X * speed, dir.Y * speed, dir.Z * speed

            -- FLING MODE. goonpriv's teleport follows the same route as the
            -- normal one; what makes it drag you around is a helix laid over
            -- the velocity -- a sideways sine drift perpendicular to travel, a
            -- vertical wave, and a pulse on the forward speed. The path is
            -- unchanged, the ride is not.
            --
            -- The drift is scaled off the current speed and capped, so it stays
            -- a wobble on a slow leg instead of throwing the character off the
            -- route, and it is applied before the vertical caps below so a
            -- fling can never climb faster than a normal flight is allowed to.
            if _G.RyPrivTPMode == "fling" then
                local flat = Vector3.new(dir.X, 0, dir.Z)
                if flat.Magnitude > 0.1 then
                    local u = flat.Unit
                    local perp = Vector3.new(-u.Z, 0, u.X)
                    local t = os.clock()
                    local amp = tonumber(_G.RyPrivFlingAmp) or 1
                    local slide = ((math.sin(t * 16) * 0.8) + (math.cos(t * 8) * 0.55))
                        * math.min(125, speed * 0.28) * amp
                    local waveY = ((math.sin(t * 20) * 0.75) + (math.cos(t * 10) * 0.5))
                        * math.min(75, speed * 0.18) * amp
                    local pulse = 1.0 + (math.sin(t * 24) * 0.18 * amp)
                    vx = dir.X * speed * pulse + perp.X * slide
                    vz = dir.Z * speed * pulse + perp.Z * slide
                    vy = vy + waveY
                end
            end

            if vy > _upCap then vy = _upCap elseif vy < -_vCap then vy = -_vCap end

            lastDir = dir
            _setFlightVel(hrp, Vector3.new(vx, vy, vz))
        end
    end))

    local totalDist = 0
    do
        local prev = hrp.Position
        for _, wp in ipairs(waypoints) do
            totalDist = totalDist + (prev - wp).Magnitude
            prev = wp
        end
    end
    -- budget: configured speed, limited by step cap, plus lagback penalty
    local timeout = (totalDist / math.min(_runSpeed, _maxStep * 60)) * 1.8
        + (tonumber(_G.RyPrivFlightGrace) or 2)
    local _t0 = os.clock()
    while not done and (os.clock() - _t0) < (timeout + lagPenalty) do
        RunService.Heartbeat:Wait()
        if not (hrp and hrp.Parent) then break end
    end
    finish()
    vZero(hrp)
    if lagbacks > 0 then
        _G.RyPrivStealSay(string.format("TP leg: %d lagback%s, speed settled at %d%%",
            lagbacks, lagbacks == 1 and "" or "s", math.floor(speedScale * 100 + 0.5)))
    end
end

_OTHER_CLONES = {}
do
    local _MY_CLONE = tostring(LP.UserId) .. "_Clone"
    local _seen = {}
    local function _isOtherClone(inst)
        if not inst then return false end
        local n = inst.Name
        if n == _MY_CLONE then return false end
        if n:match("^%d+_Clone$") then return true end
        if not n:find("lone", 1, true) then return false end
        if not inst:IsA("Model") then return false end
        if inst == LP.Character then return false end
        if not inst:FindFirstChild("HumanoidRootPart") then return false end
        if not inst:FindFirstChildOfClass("Humanoid") then return false end
        for _, pl in ipairs(Players:GetPlayers()) do
            if pl.Character == inst then return false end
        end
        return true
    end
    local function _neutralize(inst)
        if not inst or _seen[inst] then return end
        _seen[inst] = true
        _OTHER_CLONES[#_OTHER_CLONES + 1] = inst
        local function declaw(d)
            if d:IsA("BasePart") and d.CanCollide then pcall(function() d.CanCollide = false end) end
        end
        for _, d in ipairs(inst:GetDescendants()) do declaw(d) end
        inst.DescendantAdded:Connect(declaw)
        inst.Destroying:Connect(function()
            _seen[inst] = nil
            for i = #_OTHER_CLONES, 1, -1 do
                if _OTHER_CLONES[i] == inst then table.remove(_OTHER_CLONES, i); break end
            end
        end)
    end
    local function _scan(inst)
        if _isOtherClone(inst) then _neutralize(inst) end
    end
    for _, c in ipairs(workspace:GetChildren()) do _scan(c) end
    workspace.ChildAdded:Connect(function(c)
        _scan(c)
        task.defer(function() if c and c.Parent == workspace then _scan(c) end end)
    end)
end

do
    local _pSeen = setmetatable({}, { __mode = "k" })
    local function _declaw(d)
        if d:IsA("BasePart") and d.CanCollide then pcall(function() d.CanCollide = false end) end
    end
    local function _declawChar(char)
        if not char then return end
        for _, d in ipairs(char:GetDescendants()) do _declaw(d) end
        if not _pSeen[char] then
            _pSeen[char] = true
            char.DescendantAdded:Connect(_declaw)
        end
    end
    local function _hookPlayer(pl)
        if pl == LP then return end
        if pl.Character then _declawChar(pl.Character) end
        pl.CharacterAdded:Connect(function(c) task.wait(0.15); _declawChar(c) end)
    end
    for _, pl in ipairs(Players:GetPlayers()) do _hookPlayer(pl) end
    Players.PlayerAdded:Connect(_hookPlayer)
    task.spawn(function()
        _G.RyPrivBootWait()
        while true do
            task.wait(3)
            local myHrp = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
            for _, pl in ipairs(Players:GetPlayers()) do
                if pl ~= LP and pl.Character then
                    local h = pl.Character:FindFirstChild("HumanoidRootPart")
                    if not myHrp or (h and (h.Position - myHrp.Position).Magnitude <= 150) then
                        for _, d in ipairs(pl.Character:GetDescendants()) do _declaw(d) end
                        task.wait() 
                    end
                end
            end
        end
    end)
end

local _len
do
local _DIRS = { Vector3.new(1,0,0), Vector3.new(-1,0,0), Vector3.new(0,0,1), Vector3.new(0,0,-1) }
local _STRUCT = { ["structure base home"] = true, ["Wall"] = true, ["Floor"] = true, ["Roof"] = true }
local _SKIP_NAME = { ["DeliveryHitbox"]=true, ["StealHitbox"]=true, ["LaserHitbox"]=true,
    ["AnimalTarget"]=true, ["Multiplier"]=true, ["Laser"]=true, ["Hitbox"]=true,
    ["Spawn"]=true, ["MainRoot"]=true, ["SecondFloor"]=true, ["ThirdFloor"]=true, ["Slope"]=true }
local function _blocks(inst)
    if not inst then return false end
    if _SKIP_NAME[inst.Name] then return false end
    if inst.CanCollide then return true end
    local _par = inst.Parent
    if _par and (_par.Name == "ObstacleVolumes" or _par.Name == "ObstacleVolume") then return false end
    if _STRUCT[inst.Name] then return true end
    local s = inst.Size
    if s and math.max(s.X * s.Y, s.X * s.Z, s.Y * s.Z) > 150 then return true end
    return false
end
local function _blocksWide(inst)
    if not inst then return false end
    if _SKIP_NAME[inst.Name] then return false end
    if inst.CanCollide then return true end
    local _par = inst.Parent
    if _par and (_par.Name == "ObstacleVolumes" or _par.Name == "ObstacleVolume") then return false end
    if _STRUCT[inst.Name] then return true end
    local s = inst.Size
    if s and math.max(s.X * s.Y, s.X * s.Z, s.Y * s.Z) > 30 then return true end
    return false
end
-- ── CAST SETUP CACHE ─────────────────────────────────────────────────────
-- _block and _sweepDir each allocated a fresh RaycastParams and rebuilt the
-- ignore list -- walking every player in the server -- on EVERY call. A single
-- _clearWide does three of those (one _clear plus two sweep directions), and
-- computeRoute runs _clearWide dozens of times per plan, with the string-pull
-- calling it in a nested loop on top. That is hundreds of RaycastParams and
-- hundreds of full player walks to plan one route.
--
-- The ignore list only changes when somebody spawns, leaves, or a clone is
-- made, so build it on a short interval and copy it into a reusable buffer.
-- Neither function yields or re-enters, so a shared buffer each is safe.
local _skipBase, _skipAt = {}, -1
local function _skipList()
    local now = os.clock()
    if now - _skipAt < (tonumber(_G.RyPrivCastSkipTTL) or 0.5) then return _skipBase end
    local t = {}
    for _, pl in ipairs(Players:GetPlayers()) do
        if pl.Character then t[#t + 1] = pl.Character end
    end
    for _, cl in ipairs(_OTHER_CLONES) do t[#t + 1] = cl end
    _skipBase, _skipAt = t, now
    return t
end

local _blockRP = RaycastParams.new()
_blockRP.FilterType = Enum.RaycastFilterType.Exclude
_blockRP.IgnoreWater = true
local _blockSkip = {}

local function _block(origin, target, blockFn)
    blockFn = blockFn or _blocks
    local rp, skip = _blockRP, _blockSkip
    table.clear(skip)
    local base = _skipList()
    table.move(base, 1, #base, 1, skip)
    rp.FilterDescendantsInstances = skip
    local o = origin
    for _ = 1, 16 do
        local d = target - o
        if d.Magnitude < 0.05 then return nil end
        local res = workspace:Raycast(o, d, rp)
        if not res then return nil end
        if blockFn(res.Instance) then return res end
        -- only re-assign the filter when the list actually changed; assigning
        -- FilterDescendantsInstances copies the table every time
        skip[#skip + 1] = res.Instance
        rp.FilterDescendantsInstances = skip
        o = res.Position + d.Unit * 0.3
    end
    return nil
end
local function _clear(a, b) return _block(a, b) == nil end
function _len(pts)
    local s, prev = 0, pts[1]
    for k = 2, #pts do s = s + (pts[k] - prev).Magnitude; prev = pts[k] end
    return s
end
-- An earlier route planner lived here: _pull, _stages, _routeClear, _peakY,
-- _starts and _candidates. None of it was reachable -- computeRoute builds
-- routes from _pullWide, voxelRoute and its own lateral-offset candidates,
-- and _pull was the narrow-clearance ancestor of _pullWide.

local PathfindingService = game:GetService("PathfindingService")
local _CLEARANCE = 16
local function _clearWideRay(a, b)
    return _block(a, b, _blocksWide) == nil
end

local _SWEEP_R = 4
local _ENDPOINT_SLACK = 6
local _canSphere = nil
local function _sweepBlockFn(inst)
    if _G.RyPrivStrictSweep == false then return _blocks(inst) end
    return _blocksWide(inst)
end
local _sweepRP = RaycastParams.new()
_sweepRP.FilterType = Enum.RaycastFilterType.Exclude
_sweepRP.IgnoreWater = true
local _sweepSkip = {}

local function _sweepDir(a, b)
    local rp, skip = _sweepRP, _sweepSkip
    table.clear(skip)
    local base = _skipList()
    table.move(base, 1, #base, 1, skip)
    rp.FilterDescendantsInstances = skip
    local o = a
    for _ = 1, 24 do
        local d = b - o
        if d.Magnitude < 0.05 then return false end
        -- pcall the method directly rather than wrapping it in a closure:
        -- this loop ran up to 24 times per call, twice per _clearWide, and
        -- every one of those iterations allocated a fresh closure.
        local ok, res = pcall(workspace.Spherecast, workspace, o, _SWEEP_R, d, rp)
        if not ok then _canSphere = false; return nil end
        if not res then return false end
        if _sweepBlockFn(res.Instance) then return true end
        skip[#skip + 1] = res.Instance
        rp.FilterDescendantsInstances = skip
        local adv = (res.Distance or 0) - 0.05
        if adv > 0 then o = o + d.Unit * math.min(adv, d.Magnitude) end
    end
    return true
end
local function _sweepBlocked(a, b, slackA, slackB)
    if _canSphere == nil then
        _canSphere = pcall(function()
            workspace:Spherecast(Vector3.new(0, 10000, 0), 1, Vector3.new(0, -1, 0), RaycastParams.new())
        end)
    end
    if not _canSphere then return nil end
    local d = b - a
    local len = d.Magnitude
    if len < 0.1 then return false end
    local u = d / len
    local a2 = a + u * math.min(slackA or _ENDPOINT_SLACK, len * 0.4)
    local b2 = b - u * math.min(slackB or _ENDPOINT_SLACK, len * 0.4)
    local fwd = _sweepDir(a2, b2)
    if fwd == nil then return nil end
    if fwd then return true end
    local rev = _sweepDir(b2, a2)
    if rev == nil then return nil end
    return rev
end

local function _clearWide(a, b, slackA, slackB)
    if not _clear(a, b) then return false end
    local sw = _sweepBlocked(a, b, slackA, slackB)
    if sw ~= nil then return not sw end
    local d = Vector3.new(b.X - a.X, 0, b.Z - a.Z)
    if d.Magnitude < 0.1 then
        local ox = Vector3.new(_CLEARANCE, 0, 0)
        local oz = Vector3.new(0, 0, _CLEARANCE)
        return _clearWideRay(a + ox, b + ox) and _clearWideRay(a - ox, b - ox)
            and _clearWideRay(a + oz, b + oz) and _clearWideRay(a - oz, b - oz)
    end
    local perp = Vector3.new(-d.Z, 0, d.X).Unit * _CLEARANCE
    local up = Vector3.new(0, _CLEARANCE, 0)
    return _clearWideRay(a + perp, b + perp)
        and _clearWideRay(a - perp, b - perp)
        and _clearWideRay(a + up, b + up)
        and _clearWideRay(a - up, b - up)
end

local function _crestClear(a, b)
    if not _clear(a, b) then return false end
    local cl = tonumber(_G.RyPrivCrestClearance) or 6
    local d = Vector3.new(b.X - a.X, 0, b.Z - a.Z)
    if d.Magnitude < 0.1 then
        return true
    end
    local perp = Vector3.new(-d.Z, 0, d.X).Unit * cl
    return _clearWideRay(a + perp, b + perp)
        and _clearWideRay(a - perp, b - perp)
        and _clearWideRay(a + Vector3.new(0, cl, 0), b + Vector3.new(0, cl, 0))
end

local function _pullWide(pts)
    if #pts <= 2 then return pts end
    local out = { pts[1] }
    local i = 1
    local n = #pts
    while i < n do
        local j = n
        while j > i + 1 do
            local a, b = out[#out], pts[j]
            local sA = (i == 1) and _ENDPOINT_SLACK or 0
            local sB = (j == n) and _ENDPOINT_SLACK or 0
            if _clearWide(a, b, sA, sB) then break end
            j = j - 1
        end
        out[#out + 1] = pts[j]
        i = j
    end
    return out
end

local function _pushOffWalls(pts)
    if #pts <= 2 then return pts end
    local MARGIN = 8
    local MAX_PUSH = 12
    local out = { pts[1] }
    for i = 2, #pts - 1 do
        local p = pts[i]
        local shift = Vector3.zero
        for _, dr in ipairs(_DIRS) do
            local res = _block(p, p + dr * MARGIN, _blocks)
            if res then
                local dist = (res.Position - p).Magnitude
                if dist < MARGIN then
                    shift = shift - dr * (MARGIN - dist)
                end
            end
        end
        do
            local resUp = _block(p, p + Vector3.new(0, MARGIN, 0), _blocks)
            if resUp then
                local dist = (resUp.Position - p).Magnitude
                if dist < 4 then shift = shift + Vector3.new(0, -(4 - dist), 0) end
            end
        end
        if shift.Magnitude > 0.1 then
            if shift.Magnitude > MAX_PUSH then shift = shift.Unit * MAX_PUSH end
            local moved = p + shift
            if _clear(out[#out], moved) then
                out[#out + 1] = moved
            else
                out[#out + 1] = p
            end
        else
            out[#out + 1] = p
        end
    end
    out[#out + 1] = pts[#pts]
    return out
end


do
local _vxFloor, _vxSqrt = math.floor, math.sqrt
local _vxMin, _vxMax = math.min, math.max
local function _vxAbs(n) return n < 0 and -n or n end

local _vxOverlap = OverlapParams.new()
_vxOverlap.FilterType = Enum.RaycastFilterType.Exclude
_vxOverlap.RespectCanCollide = true

local _vxCast = RaycastParams.new()
_vxCast.FilterType = Enum.RaycastFilterType.Exclude
_vxCast.RespectCanCollide = true
_vxCast.IgnoreWater = true

local _vxOrigin
local _vxDimX, _vxDimY, _vxDimZ = 0, 0, 0
local _vxSz, _vxInflate = 4, 2.5
local _vxSolid = {}
-- Identifies the grid the solidity cache belongs to, so it survives a replan
-- of the same region instead of being thrown away on every call.
local _vxSig, _vxSigAt = nil, 0
local _vxHeight = 5

local function _vxWorld(sz, x, y, z)
    local h = sz * 0.5
    return Vector3.new(_vxOrigin.X + x * sz + h, _vxOrigin.Y + y * sz + h, _vxOrigin.Z + z * sz + h)
end
local function _vxKey(x, y, z) return x + y * 1024 + z * 1048576 end

local function _vxIsSolid(x, y, z)
    if x < 0 or y < 0 or z < 0 or x >= _vxDimX or y >= _vxDimY or z >= _vxDimZ then return true end
    local k = _vxKey(x, y, z)
    local c = _vxSolid[k]
    if c ~= nil then return c end
    local h = _vxSz * 0.5
    local cx = _vxOrigin.X + x * _vxSz + h
    local cy = _vxOrigin.Y + y * _vxSz + h
    local cz = _vxOrigin.Z + z * _vxSz + h
    local sxz = _vxSz + _vxInflate
    local vy = _vxHeight > _vxSz and _vxHeight or _vxSz
    local vcy = cy - h + vy * 0.5
    local parts = workspace:GetPartBoundsInBox(CFrame.new(cx, vcy, cz), Vector3.new(sxz, vy, sxz), _vxOverlap)
    local solid = #parts > 0
    _vxSolid[k] = solid
    return solid
end

local function _vxSegClear(from, to, radius, height, sample)
    local dir = to - from
    local mag = dir.Magnitude
    if mag < 0.05 then return true end
    if workspace:Raycast(from, dir, _vxCast) then return false end
    local r = radius > 1 and radius or 1
    if workspace:Blockcast(CFrame.new(from), Vector3.new(r * 2, height, r * 2), dir, _vxCast) ~= nil then return false end
    -- Step by the probe's own width instead of one stud.
    --
    -- This sampled every stud along the segment, so a 300-stud leg fired 299
    -- GetPartBoundsInBox calls -- and _vxSimplify runs it over and over while
    -- string-pulling the A* result. The probe box is r*2 wide, so stepping by
    -- exactly r*2 leaves the boxes touching end to end: identical coverage,
    -- a fraction of the queries. The cap bounds the very longest legs.
    local stepLen = r * 2
    if stepLen < 1 then stepLen = 1 end
    local n = _vxFloor(mag / stepLen)
    local maxN = tonumber(_G.RyPrivPathSampleMax) or 64
    if n > maxN then n = maxN end
    if sample ~= false and n >= 2 then
        local step = dir / n
        local torso = Vector3.new(r * 2, 3, r * 2)
        for i = 1, n - 1 do
            local pt = from + step * i
            if #workspace:GetPartBoundsInBox(CFrame.new(pt), torso, _vxOverlap) > 0 then return false end
        end
    end
    return true
end

local _vxNeigh = {}
do
    for dx = -1, 1 do
        for dy = -1, 1 do
            for dz = -1, 1 do
                if dx ~= 0 or dy ~= 0 or dz ~= 0 then
                    local nz = (dx ~= 0 and 1 or 0) + (dy ~= 0 and 1 or 0) + (dz ~= 0 and 1 or 0)
                    local kd = dx + dy * 1024 + dz * 1048576
                    _vxNeigh[#_vxNeigh + 1] = { dx, dy, dz, _vxSqrt(dx * dx + dy * dy + dz * dz), nz, kd }
                end
            end
        end
    end
end

local function _vxNoCorner(cx, cy, cz, off)
    if off[5] < 2 then return true end
    if off[1] ~= 0 and _vxIsSolid(cx + off[1], cy, cz) then return false end
    if off[2] ~= 0 and _vxIsSolid(cx, cy + off[2], cz) then return false end
    if off[3] ~= 0 and _vxIsSolid(cx, cy, cz + off[3]) then return false end
    return true
end

local function _vxSnapGoal(goalPos, x, y, z)
    if not _vxIsSolid(x, y, z) then return x, y, z end
    for r = 1, 16 do
        for dx = -r, r do
            for dy = -r, r do
                for dz = -r, r do
                    if _vxMax(_vxAbs(dx), _vxAbs(dy), _vxAbs(dz)) == r then
                        local nx, ny, nz = x + dx, y + dy, z + dz
                        if not _vxIsSolid(nx, ny, nz) and (_vxWorld(_vxSz, nx, ny, nz) - goalPos).Magnitude <= 8 then
                            return nx, ny, nz
                        end
                    end
                end
            end
        end
    end
    return x, y, z
end
local function _vxSnapStart(pos, x, y, z)
    if not _vxIsSolid(x, y, z) then return x, y, z end
    for r = 1, 16 do
        for dx = -r, r do
            for dy = -r, r do
                for dz = -r, r do
                    if _vxMax(_vxAbs(dx), _vxAbs(dy), _vxAbs(dz)) == r then
                        local nx, ny, nz = x + dx, y + dy, z + dz
                        if not _vxIsSolid(nx, ny, nz) and not workspace:Raycast(pos, _vxWorld(_vxSz, nx, ny, nz) - pos, _vxCast) then
                            return nx, ny, nz
                        end
                    end
                end
            end
        end
    end
    return x, y, z
end

local function _vxPush(h, f, key)
    local i = #h + 1
    h[i] = { f, key }
    while i > 1 do
        local p = _vxFloor(i * 0.5)
        if h[p][1] <= h[i][1] then break end
        h[p], h[i] = h[i], h[p]
        i = p
    end
end
local function _vxPop(h)
    local n = #h
    if n == 0 then return nil end
    local top = h[1]
    h[1] = h[n]
    h[n] = nil
    n -= 1
    local i = 1
    while true do
        local l, r, s = i + i, i + i + 1, i
        if l <= n and h[l][1] < h[s][1] then s = l end
        if r <= n and h[r][1] < h[s][1] then s = r end
        if s == i then break end
        h[i], h[s] = h[s], h[i]
        i = s
    end
    return top[2]
end

local _vxHeurW = 2
local function _vxAStar(sz, startCell, goalCell, startPos, goalPos)
    local sx, sy, sz2 = _vxSnapStart(startPos, startCell.x, startCell.y, startCell.z)
    local gx, gy, gz = _vxSnapGoal(goalPos, goalCell.x, goalCell.y, goalCell.z)
    local goalKey = _vxKey(gx, gy, gz)
    local startKey = _vxKey(sx, sy, sz2)

    local nodes = { [startKey] = { x = sx, y = sy, z = sz2, g = 0, parent = nil } }
    local closed = {}
    local heap = {}
    _vxPush(heap, 0, startKey)

    local function Heur(x, y, z)
        local ax, ay, az = x - gx, y - gy, z - gz
        return _vxSqrt(ax * ax + ay * ay + az * az)
    end

    -- Budget. The old ceiling was 300000 pops against a grid capped at 200000
    -- cells, so it was never reached: a plan that could not find the goal
    -- expanded the entire reachable grid, doing a GetPartBoundsInBox on first
    -- touch of every cell, and stalled the frame for seconds. computeRoute
    -- has cheaper fallbacks below this, so give up early and let them run.
    --
    -- The wall-clock deadline is the real guarantee: cold cells cost a spatial
    -- query each, so a pop count alone cannot bound the time.
    local pops = 0
    local _budget   = tonumber(_G.RyPrivPathMaxPops) or 12000
    local _deadline = os.clock() + (tonumber(_G.RyPrivPathMaxTime) or 0.05)
    while #heap > 0 do
        local curKey = _vxPop(heap)
        if closed[curKey] then continue end
        closed[curKey] = true
        pops += 1
        if pops > _budget then break end
        if pops % 256 == 0 and os.clock() > _deadline then break end

        local cur = nodes[curKey]
        if curKey == goalKey then
            local path = {}
            local n = cur
            while n do
                path[#path + 1] = _vxWorld(sz, n.x, n.y, n.z)
                n = n.parent and nodes[n.parent]
            end
            local rev = {}
            for i = #path, 1, -1 do rev[#rev + 1] = path[i] end
            return rev
        end

        local cx, cy, cz = cur.x, cur.y, cur.z
        local cg = cur.g
        for _, off in _vxNeigh do
            local nk = curKey + off[6]
            if closed[nk] then continue end
            local nx, ny, nz = cx + off[1], cy + off[2], cz + off[3]
            if _vxIsSolid(nx, ny, nz) then continue end
            if not _vxNoCorner(cx, cy, cz, off) then continue end
            local tg = cg + off[4]
            local ex = nodes[nk]
            if not ex or tg < ex.g then
                if ex then
                    ex.g, ex.parent, ex.x, ex.y, ex.z = tg, curKey, nx, ny, nz
                else
                    nodes[nk] = { x = nx, y = ny, z = nz, g = tg, parent = curKey }
                end
                _vxPush(heap, tg + _vxHeurW * Heur(nx, ny, nz), nk)
            end
        end
    end
    return nil
end

local function _vxSimplify(path, radius, height)
    if not path or #path < 3 then return path end
    local out = { path[1] }
    local anchor = 1
    local i = 2
    while i <= #path do
        if not _vxSegClear(path[anchor], path[i + 1] or path[i], radius, height, false) then
            out[#out + 1] = path[i]
            anchor = i
        end
        i += 1
    end
    out[#out + 1] = path[#path]
    return out
end

voxelRoute = function(fromPos, toPos)
    local char = LP.Character
    local _flt = char and { char } or {}
    for _, cl in ipairs(_OTHER_CLONES) do _flt[#_flt + 1] = cl end
    _vxOverlap.FilterDescendantsInstances = _flt
    _vxCast.FilterDescendantsInstances = _flt

    local sz      = tonumber(_G.RyPrivPathCell)   or 4
    local inflate = tonumber(_G.RyPrivPathRadius) or 2.5
    local height  = tonumber(_G.RyPrivPathHeight) or 5
    local pad     = tonumber(_G.RyPrivPathPad)    or 40

    _vxSz, _vxInflate, _vxHeight = sz, inflate, height

    local mn = Vector3.new(_vxMin(fromPos.X, toPos.X), _vxMin(fromPos.Y, toPos.Y), _vxMin(fromPos.Z, toPos.Z)) - Vector3.new(pad, pad, pad)
    local mx = Vector3.new(_vxMax(fromPos.X, toPos.X), _vxMax(fromPos.Y, toPos.Y), _vxMax(fromPos.Z, toPos.Z)) + Vector3.new(pad, pad, pad)
    _vxOrigin = mn
    local size = mx - mn
    _vxDimX = _vxFloor(size.X / sz) + 1
    _vxDimY = _vxFloor(size.Y / sz) + 1
    _vxDimZ = _vxFloor(size.Z / sz) + 1
    if _vxDimX * _vxDimY * _vxDimZ > 200000 then return nil end

    -- Keep the solidity cache while the grid is the same region. It used to be
    -- wiped on every call, so a replan over the same ground -- which is exactly
    -- what a TP retry or a re-pick does -- re-ran a spatial query for every
    -- cell it touched. Cell keys are relative to the origin, so the cache is
    -- only valid for an identical grid; the TTL covers the world moving
    -- underneath it (doors, podiums, other players).
    do
        local sig = string.format("%d:%d:%d:%d:%d:%d:%d",
            _vxFloor(mn.X), _vxFloor(mn.Y), _vxFloor(mn.Z),
            _vxDimX, _vxDimY, _vxDimZ, sz)
        local now = os.clock()
        if sig ~= _vxSig or (now - _vxSigAt) > (tonumber(_G.RyPrivPathCellTTL) or 3) then
            table.clear(_vxSolid)
            _vxSig, _vxSigAt = sig, now
        end
    end

    local startCell = {
        x = _vxFloor((fromPos.X - _vxOrigin.X) / sz),
        y = _vxFloor((fromPos.Y - _vxOrigin.Y) / sz),
        z = _vxFloor((fromPos.Z - _vxOrigin.Z) / sz),
    }
    local goalCell = {
        x = _vxFloor((toPos.X - _vxOrigin.X) / sz),
        y = _vxFloor((toPos.Y - _vxOrigin.Y) / sz),
        z = _vxFloor((toPos.Z - _vxOrigin.Z) / sz),
    }

    local path = _vxAStar(sz, startCell, goalCell, fromPos, toPos)
    if not path then return nil end
    path = _vxSimplify(path, inflate, height)
    if not path or #path == 0 then return nil end

    local route = {}
    for idx = 2, #path do route[#route + 1] = path[idx] end
    if #route == 0 or (route[#route] - toPos).Magnitude > 0.5 then
        route[#route + 1] = toPos
    end
    return route
end

end

_G.RyPrivVoxelRoute = voxelRoute

local _MAP_CENTER = { minX = -458, maxX = -362, minZ = -40, maxZ = 185 }
local _BYPASS_Z_NORTH, _BYPASS_Z_SOUTH = 205, -95
local _BYPASS_X_WEST,  _BYPASS_X_EAST  = -525, -295

local function _inCenterZone(x, z)
    return x >= _MAP_CENTER.minX and x <= _MAP_CENTER.maxX
       and z >= _MAP_CENTER.minZ and z <= _MAP_CENTER.maxZ
end

local function _segmentCrossesCenter(a, b)
    if _inCenterZone(a.X, a.Z) or _inCenterZone(b.X, b.Z) then return true end
    for i = 1, 10 do
        local t = i / 11
        if _inCenterZone(a.X + (b.X - a.X) * t, a.Z + (b.Z - a.Z) * t) then return true end
    end
    return false
end

local function _findBestCenterDetour(fromPos, toPos, y)
    local candidates = {
        { Vector3.new(fromPos.X, y, _BYPASS_Z_NORTH), Vector3.new(toPos.X, y, _BYPASS_Z_NORTH) },
        { Vector3.new(fromPos.X, y, _BYPASS_Z_SOUTH), Vector3.new(toPos.X, y, _BYPASS_Z_SOUTH) },
        { Vector3.new(_BYPASS_X_WEST, y, fromPos.Z), Vector3.new(_BYPASS_X_WEST, y, toPos.Z) },
        { Vector3.new(_BYPASS_X_EAST, y, fromPos.Z), Vector3.new(_BYPASS_X_EAST, y, toPos.Z) },
    }
    local best, bestLen = nil, math.huge
    for _, pair in ipairs(candidates) do
        local w1, w2 = pair[1], pair[2]
        if _clearWide(fromPos, w1) and _clearWide(w1, w2) and _clearWide(w2, toPos) then
            local len = (fromPos - w1).Magnitude + (w1 - w2).Magnitude + (w2 - toPos).Magnitude
            if len < bestLen then bestLen = len; best = { w1, w2 } end
        end
    end
    return best
end
_G.RyPrivSegmentCrossesCenter = _segmentCrossesCenter
_G.RyPrivFindCenterDetour     = _findBestCenterDetour

local function _rowBoxX() return tonumber(_G.RyPrivRowBoxX) or 26 end
local function _rowBoxZ() return tonumber(_G.RyPrivRowBoxZ) or 30 end
local function _rowLane() return tonumber(_G.RyPrivRowLane) or 30 end

local function _nearestBase(p)
    local bi, bd = nil, math.huge
    for i = 1, 8 do
        local b = BASES_LOW[i]
        local d = (p.X - b.X) ^ 2 + (p.Z - b.Z) ^ 2
        if d < bd then bd = d; bi = i end
    end
    if bd > 70 * 70 then return nil end
    return bi
end

local function _segmentHitsOtherBase(a, b, ignA, ignB)
    local hx, hz = _rowBoxX(), _rowBoxZ()
    for i = 0, 24 do
        local t = i / 24
        local px = a.X + (b.X - a.X) * t
        local pz = a.Z + (b.Z - a.Z) * t
        for k = 1, 8 do
            if k ~= ignA and k ~= ignB then
                local bs = BASES_LOW[k]
                if math.abs(px - bs.X) <= hx and math.abs(pz - bs.Z) <= hz then
                    return true
                end
            end
        end
    end
    return false
end

local function _findRowDetour(fromPos, toPos, y)
    local iFrom, iTo = _nearestBase(fromPos), _nearestBase(toPos)
    if not _segmentHitsOtherBase(fromPos, toPos, iFrom, iTo) then return nil end

    local colX = BASES_LOW[iTo or 1].X
    local off  = _rowLane()
    local lanes = {}
    local outer = (colX < COLUMN_SPLIT_X) and (colX - off) or (colX + off)
    lanes[#lanes + 1] = outer
    local inner = (colX < COLUMN_SPLIT_X) and (colX + off) or (colX - off)
    if not _inCenterZone(inner, (fromPos.Z + toPos.Z) * 0.5) then
        lanes[#lanes + 1] = inner
    end

    local best, bestLen = nil, math.huge
    for _, laneX in ipairs(lanes) do
        local w1 = Vector3.new(laneX, y, fromPos.Z)
        local w2 = Vector3.new(laneX, y, toPos.Z)
        if _clearWide(fromPos, w1) and _clearWide(w1, w2) and _clearWide(w2, toPos)
           and not _segmentHitsOtherBase(w1, w2, iFrom, iTo) then
            local len = (fromPos - w1).Magnitude + (w1 - w2).Magnitude + (w2 - toPos).Magnitude
            if len < bestLen then bestLen = len; best = { w1, w2 } end
        end
    end
    return best
end
_G.RyPrivFindRowDetour = _findRowDetour

local _hopRP = RaycastParams.new()
_hopRP.FilterType = Enum.RaycastFilterType.Exclude

local function _partTopY(inst)
    local ok, t = pcall(function()
        local cf, sz = inst.CFrame, inst.Size
        local half = (math.abs(cf.RightVector.Y) * sz.X
            + math.abs(cf.UpVector.Y) * sz.Y
            + math.abs(cf.LookVector.Y) * sz.Z) / 2
        return cf.Position.Y + half
    end)
    if ok and t then return t end
    return inst.Position.Y + (inst.Size.Y / 2)
end

local function _hopBlockerTop(a, b)
    local ignore = { LP.Character }
    for _, cl in ipairs(_OTHER_CLONES) do ignore[#ignore + 1] = cl end
    local top = nil
    for _ = 1, 10 do
        _hopRP.FilterDescendantsInstances = ignore
        local r = workspace:Raycast(a, b - a, _hopRP)
        if not r then break end
        if _blocks(r.Instance) then
            local t = _partTopY(r.Instance)
            if not top or t > top then top = t end
        end
        ignore[#ignore + 1] = r.Instance
    end
    return top
end

local function _hopRoute(fromPos, toPos)
    local flat = Vector3.new(toPos.X - fromPos.X, 0, toPos.Z - fromPos.Z)
    local dist = flat.Magnitude
    if dist < 8 then return nil end
    local dir = flat.Unit
    local step = tonumber(_G.RyPrivHopStep) or 8
    local clear = tonumber(_G.RyPrivHopClear) or 4
    local maxUp = tonumber(_G.RyPrivHopMaxUp) or 22
    local groundY = fromPos.Y
    local route = {}
    local inHop, hopY, hopStart = false, nil, nil
    local i = step
    while i <= dist do
        local a = fromPos + dir * (i - step)
        local b = fromPos + dir * math.min(i, dist)
        local ga = Vector3.new(a.X, groundY, a.Z)
        local gb = Vector3.new(b.X, groundY, b.Z)
        if not _clear(ga, gb) then
            local top = _hopBlockerTop(ga, gb) or (groundY + 6)
            local want = top + clear
            if want - groundY > maxUp then return nil end
            if not inHop then
                inHop, hopY, hopStart = true, want, ga
                route[#route + 1] = Vector3.new(a.X, want, a.Z)
            elseif want > hopY then
                hopY = want
                route[#route + 1] = Vector3.new(a.X, want, a.Z)
            end
        elseif inHop then
            inHop = false
            route[#route + 1] = Vector3.new(b.X, hopY, b.Z)
            route[#route + 1] = gb
        end
        i = i + step
    end
    if inHop then
        route[#route + 1] = Vector3.new(toPos.X, hopY, toPos.Z)
    end
    route[#route + 1] = toPos
    return route
end
_G.RyPrivHopRoute = _hopRoute

-- ── ROUTE SMOOTHING ──────────────────────────────────────────────────────
-- velMoveThrough flies straight at each waypoint, so every hard corner in a
-- route is a stop-and-turn: that is what the flight reads as jerky. Chaikin
-- corner cutting replaces a corner with two points a quarter of the way down
-- each adjacent leg, turning it into an arc.
--
-- A cut is only kept if the shortcut it creates passes the same clearance
-- test the route was built with. A corner that cannot be cut safely is left
-- sharp rather than forced, so smoothing can never route you into geometry.
function _smoothRoute(pts, passes)
    if not pts or #pts < 3 then return pts end
    if _G.RyPrivPathSmooth == false then return pts end
    -- Cost scales with corner count, so leave long routes alone: those come
    -- from the voxel planner and are already string-pulled.
    if #pts > (tonumber(_G.RyPrivPathSmoothMax) or 8) then return pts end
    passes = math.max(math.floor(passes or tonumber(_G.RyPrivPathSmoothPasses) or 1), 1)
    for _ = 1, passes do
        if #pts < 3 then break end
        local out = { pts[1] }
        for i = 2, #pts - 1 do
            local a, b, c = pts[i - 1], pts[i], pts[i + 1]
            local p = b + (a - b) * 0.25
            local q = b + (c - b) * 0.25
            if (p - q).Magnitude > 0.5
                and _clearWide(out[#out], p)
                and _clearWide(p, q)
                and _clearWide(q, c) then
                out[#out + 1] = p
                out[#out + 1] = q
            else
                out[#out + 1] = b
            end
        end
        out[#out + 1] = pts[#pts]
        pts = out
    end
    return pts
end

-- ── ROUTE CACHE ──────────────────────────────────────────────────────────
-- The same trip gets planned repeatedly -- the steal loop re-picks a target,
-- the TP retries, an arrival re-routes -- and each plan can cost hundreds of
-- casts. Endpoints are bucketed to 4 studs so small drift reuses the answer,
-- and entries expire quickly because the world moves: other players, doors,
-- podiums. A stale route is worse than a slow one, so the TTL is short.
local _routeCache, _routeCacheN = {}, 0
local function _routeKey(a, b)
    local function q(n) return math.floor(n / 4) end
    return string.format("%d,%d,%d>%d,%d,%d",
        q(a.X), q(a.Y), q(a.Z), q(b.X), q(b.Y), q(b.Z))
end
local function _routeCached(key)
    local e = _routeCache[key]
    if not e then return nil end
    if os.clock() - e.at > (tonumber(_G.RyPrivRouteTTL) or 2) then
        _routeCache[key] = nil
        _routeCacheN = _routeCacheN - 1
        return nil
    end
    return e.route
end
local function _routeStore(key, route)
    if not route or #route == 0 then return route end
    if _routeCacheN > 64 then _routeCache, _routeCacheN = {}, 0 end
    if _routeCache[key] == nil then _routeCacheN = _routeCacheN + 1 end
    _routeCache[key] = { at = os.clock(), route = route }
    return route
end
_G.RyPrivClearRouteCache = function() _routeCache, _routeCacheN = {}, 0 end
local function _routeDrop(key)
    if _routeCache[key] ~= nil then
        _routeCache[key] = nil
        _routeCacheN = _routeCacheN - 1
    end
end

function computeRoute(fromPos, toPos, facingDir, maxLift, preferCrest)
    local _ = maxLift

    local _ck = _routeKey(fromPos, toPos)
    do
        local hit = _routeCached(_ck)
        -- A cache entry is keyed on 4-stud buckets and lives for RyPrivRouteTTL
        -- seconds, so the world can move under it -- a plot door closing, a
        -- part streaming in. Re-check the leg we are about to fly before
        -- trusting the rest of it. Recomputing costs a few milliseconds;
        -- flying into a wall costs the whole run.
        if hit and #hit > 0 then
            if _clearWide(fromPos, hit[1], _ENDPOINT_SLACK, 0) then return hit end
            _routeDrop(_ck)
        end
    end

    if _G.RyPrivHopFirst == true and not _clearWide(fromPos, toPos) then
        local hop = _hopRoute(fromPos, toPos)
        if hop and #hop > 0 then return _routeStore(_ck, hop) end
    end

    if _G.RyPrivCrestFirst == true and not _clearWide(fromPos, toPos) then
        local baseY = math.max(fromPos.Y, toPos.Y, 26)
        local lifts = { tonumber(_G.RyPrivCrestLift) or 12, 20, 30, 44, 60 }
        for _, lift in ipairs(lifts) do
            local cruiseY = baseY + lift
            local up   = Vector3.new(fromPos.X, cruiseY, fromPos.Z)
            local over = Vector3.new(toPos.X,   cruiseY, toPos.Z)
            local crest = { fromPos, up, over, toPos }
            local ok = true
            for i = 1, #crest - 1 do
                local a, b = crest[i], crest[i + 1]
                if (a - b).Magnitude > 0.5 then
                    if not _crestClear(a, b) then ok = false; break end
                end
            end
            if ok then return _routeStore(_ck, crest) end
        end
    end

    local centerPatch = nil
    if _segmentCrossesCenter(fromPos, toPos) and not _clearWide(fromPos, toPos) then
        centerPatch = _findBestCenterDetour(fromPos, toPos, fromPos.Y)
        if centerPatch and #centerPatch > 0 then
            fromPos = centerPatch[#centerPatch]
        end
    end
    local rowPatch = _findRowDetour(fromPos, toPos, fromPos.Y)
    if rowPatch and #rowPatch > 0 then
        fromPos = rowPatch[#rowPatch]
    end

    local function _withPatch(route)
        if (not centerPatch or #centerPatch == 0)
           and (not rowPatch or #rowPatch == 0) then return route end
        local merged = {}
        if centerPatch then for _, p in ipairs(centerPatch) do merged[#merged + 1] = p end end
        if rowPatch    then for _, p in ipairs(rowPatch)    do merged[#merged + 1] = p end end
        for _, p in ipairs(route) do merged[#merged + 1] = p end
        -- The detour patches were concatenated raw, so every join between a
        -- patch and the route it was prepended to stayed as a dog-leg -- and
        -- because _withPatch runs AFTER _smoothRoute, nothing ever straightened
        -- them. Pull the merged list: _pullWide only drops a waypoint when the
        -- two either side of it are line-of-sight clear by _clearWide, so this
        -- cannot make a route less safe, only shorter and straighter.
        return _pullWide(merged)
    end

    if _clearWide(fromPos, toPos) then return _withPatch({ toPos }) end

    if preferCrest then
        local cruiseY = math.max(fromPos.Y, toPos.Y, 26) + 12
        local up   = Vector3.new(fromPos.X, cruiseY, fromPos.Z)
        local over = Vector3.new(toPos.X,   cruiseY, toPos.Z)
        local crest = { fromPos, up, over, toPos }
        local ok = true
        for i = 1, #crest - 1 do
            local a, b = crest[i], crest[i + 1]
            if (a - b).Magnitude > 0.5 then
                local sA = (i == 1) and _ENDPOINT_SLACK or 0
                local sB = (i == #crest - 1) and _ENDPOINT_SLACK or 0
                if not _clearWide(a, b, sA, sB) then ok = false; break end
            end
        end
        if ok then return _withPatch(crest) end
    end

    do
        local vr = voxelRoute(fromPos, toPos)
        if vr and #vr > 0 then return _withPatch(vr) end
    end

    local entry = facingDir and (toPos - facingDir * 14) or toPos

    local best, bestLen = nil, math.huge
    local function consider(pts)
        if not pts or #pts < 2 then return end
        local n = #pts
        for i = 1, n - 1 do
            local a, b = pts[i], pts[i + 1]
            if (a - b).Magnitude > 0.5 then
                local sA = (i == 1) and _ENDPOINT_SLACK or 0
                local sB = (i == n - 1) and _ENDPOINT_SLACK or 0
                if not _clearWide(a, b, sA, sB) then return end
            end
        end
        local pulled = _pullWide(pts)
        local L = _len(pulled)
        if L < bestLen then best, bestLen = pulled, L end
    end

    do
        local dirF = Vector3.new(entry.X - fromPos.X, 0, entry.Z - fromPos.Z)
        if dirF.Magnitude > 0.1 then
            dirF = dirF.Unit
            local perp = Vector3.new(-dirF.Z, 0, dirF.X)
            local midBase = (fromPos + entry) * 0.5
            -- Offsets are ordered by magnitude and a smaller sideways
            -- offset always yields a shorter route, so the first one that
            -- validates is the one this loop would have picked anyway. It
            -- used to validate all twenty every time -- each a full
            -- per-segment clearance sweep -- then choose the shortest.
            for _, off in ipairs({ 14, -14, 24, -24, 38, -38, 56, -56, 76, -76 }) do
                consider({ fromPos, midBase + perp * off, entry })
                if best then break end
                consider({ fromPos, fromPos + perp * off, entry + perp * off, entry })
                if best then break end
            end
        end
    end

    local navRaw
    if not best then
        local groundTo = Vector3.new(entry.X, fromPos.Y, entry.Z)
        local path = PathfindingService:CreatePath({
            AgentRadius = 16, AgentHeight = 5, AgentCanJump = true, AgentJumpHeight = 10, AgentMaxSlope = 89,
        })
        local FLOAT = 5
        local nav = { fromPos }
        local ok = pcall(function()
            path:ComputeAsync(Vector3.new(fromPos.X, fromPos.Y, fromPos.Z), groundTo)
        end)
        if ok and path.Status == Enum.PathStatus.Success then
            local last = fromPos
            for _, wp in ipairs(path:GetWaypoints()) do
                if (wp.Position - last).Magnitude >= 8 then
                    nav[#nav + 1] = wp.Position + Vector3.new(0, FLOAT, 0)
                    last = wp.Position
                end
            end
        end
        nav[#nav + 1] = entry + Vector3.new(0, FLOAT, 0)
        nav = _pushOffWalls(nav)
        navRaw = nav
        consider(nav)
    end

    local route = best
    if not route and _clear(fromPos, toPos) then route = { toPos } end
    if not route and navRaw then route = _pullWide(navRaw) end
    if not route then route = { toPos } end
    if (route[#route] - toPos).Magnitude > 0.5 then
        route[#route + 1] = toPos
    end
    route = _smoothRoute(route)
    return _routeStore(_ck, _withPatch(route))
end
end

function equipTool(name)
    local char = LP.Character
    if not char or char:FindFirstChild(name) then return char ~= nil end
    local bp = LP:FindFirstChild("Backpack")
    if not bp then return false end
    local tool = bp:FindFirstChild(name)
    if tool and tool:IsA("Tool") then tool.Parent = char; return true end
    return false
end

function unequipAll()
    local char, bp = LP.Character, LP.Backpack
    if not char or not bp then return end
    for _, t in pairs(char:GetChildren()) do
        if t:IsA("Tool") then t.Parent = bp end
    end
end

function doClone()
    local char = LP.Character or LP.CharacterAdded:Wait()
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if not char or not hum then return false end

    local cloner = (LP:FindFirstChild("Backpack") and LP.Backpack:FindFirstChild("Quantum Cloner"))
                or char:FindFirstChild("Quantum Cloner")
    if not cloner then return false end

    if cloner.Parent ~= char then
        pcall(function() hum:EquipTool(cloner) end)
        task.wait()
    end

    pcall(function() hum:UnequipTools() end)
    task.wait()
    if cloner.Parent ~= char then
        pcall(function() hum:EquipTool(cloner) end)
        task.wait()
    end

    local pg = LP:FindFirstChild("PlayerGui")
    local tf = pg and pg:FindFirstChild("ToolsFrames")
    local qc = tf and tf:FindFirstChild("QuantumCloner")
    local tb = qc and qc:FindFirstChild("TeleportToClone")

    _G.isCloning = true
    pcall(function() cloner:Activate() end)
    task.wait(0.05)

    local fired = false
    if tb and type(firesignal) == "function" then
        pcall(function() tb.Visible = true end)
        pcall(function() firesignal(tb.MouseButton1Click) end)
        pcall(function() firesignal(tb.MouseButton1Up) end)
        pcall(function() firesignal(tb.Activated) end)
        fired = true
    else
        local useItem = getRemote("RemoteEvent", "UseItem")
        local onTel   = getRemote("RemoteEvent", "QuantumCloner/OnTeleport")
        if useItem and onTel then
            pcall(function() useItem:FireServer() end)
            task.wait(0.05)
            pcall(function() onTel:FireServer() end)
            fired = true
        end
    end

    task.delay(0.55, function() _G.isCloning = false end)
    return fired
end

_TweenTS = game:GetService("TweenService")
function ryPrivTween(rootPart, hum, targetPos, lookDir)
    if not rootPart or not rootPart.Parent then return end
    local STEP = 20
    local speed = (_G.TPTravelSpeed or 100)
    local hasLook = lookDir ~= nil and lookDir.Magnitude > 0.001
    local prevAnchored = rootPart.Anchored
    _vzL(rootPart)
    _vzA(rootPart)
    pcall(function() rootPart.Anchored = true end)
    local deadline = os.clock() + 12
    while rootPart and rootPart.Parent and os.clock() < deadline do
        local pos = rootPart.Position
        local toTarget = targetPos - pos
        local d = toTarget.Magnitude
        if d < 0.5 then break end
        local stepDist = math.min(STEP, d)
        local stepGoal = pos + toTarget.Unit * stepDist
        local stepCF
        if hasLook then
            stepCF = CFrame.lookAt(stepGoal, stepGoal + lookDir)
        else
            stepCF = (rootPart.CFrame - rootPart.CFrame.Position) + stepGoal
        end
        local dur = math.clamp(stepDist / speed, 0.02, 1)
        local tw = _TweenTS:Create(rootPart, TweenInfo.new(dur, Enum.EasingStyle.Linear), { CFrame = stepCF })
        tw:Play()
        tw.Completed:Wait()
    end
    pcall(function() rootPart.Anchored = prevAnchored end)
    if rootPart and rootPart.Parent then _vzL(rootPart) end
end

function _makeOneWay(plat)
    if not plat then return end
    local rsConn
    local lastY = nil
    rsConn = RunService.Stepped:Connect(function()
        if not plat or not plat.Parent then
            if rsConn then rsConn:Disconnect() end
            return
        end
        local char = LP.Character
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        if hrp then
            local currentY = hrp.Position.Y
            if not lastY then lastY = currentY end
            local deltaY = currentY - lastY
            local isMovingUp = (hrp.AssemblyLinearVelocity.Y > 1) or (deltaY > 0.01 and deltaY < 5)
            if isMovingUp then
                plat.CanCollide = false
            else
                plat.CanCollide = (currentY > plat.Position.Y + 0.1)
            end
            lastY = currentY
        end
    end)
end

function goToBrainrot(petPos, slot)
    if not petPos then return end
    local char, hrp, hum
    local _t0 = os.clock()
    repeat
        char = LP.Character
        hrp = char and char:FindFirstChild("HumanoidRootPart")
        hum = char and char:FindFirstChildOfClass("Humanoid")
        if hrp and hum then break end
        RunService.Heartbeat:Wait()
    until os.clock() - _t0 > 3
    if not hrp or not hum then return end
    pcall(function() hrp.Anchored = false end)
    local _equipped = false
    do
        local _e0 = os.clock()
        repeat
            char = LP.Character
            for _, _cn in ipairs(CARPET_NAMES) do
                if char and char:FindFirstChild(_cn) then _equipped = true; break end
            end
            if _equipped then break end
            equipCarpet()
            RunService.Heartbeat:Wait()
        until _equipped or os.clock() - _e0 > (tonumber(_G.RyPrivGoCarpetWait) or 1.5)
        if _equipped then task.wait(tonumber(_G.RyPrivGoCarpetSettle) or 0.03) end
    end
    char = LP.Character
    hrp = char and char:FindFirstChild("HumanoidRootPart")
    hum = char and char:FindFirstChildOfClass("Humanoid")
    if not hrp then return end
    pcall(function() hrp.Anchored = false end)
    local _plotRad = (petPos.Y <= 8.9) and 26 or 25
    do
        local _t0b = os.clock()
        repeat
            local p = hrp.Position
            local inRad = false
            local plotsFolder = workspace:FindFirstChild("Plots")
            if plotsFolder then
                for _, plot in ipairs(plotsFolder:GetChildren()) do
                    pcall(function()
                        local pp = plot:GetPivot().Position
                        if math.abs(p.X - pp.X) < _plotRad and math.abs(p.Z - pp.Z) < _plotRad then inRad = true end
                    end)
                    if inRad then break end
                end
            end
            if inRad then break end
            RunService.Heartbeat:Wait()
        until os.clock() - _t0b > (tonumber(_G.RyPrivGoPlotWait) or 0.4)
    end
    local h = petPos.Y
    pcall(function()
        hrp.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
        hrp.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
    end)
    local _wentUnder = false
    local _slot = tonumber(slot)
    local _under = tonumber(_G.RyPrivUnderOffset) or 6
    local targetY = hrp.Position.Y
    if (_slot and _slot >= 19) or (not _slot and h > 23.15) then
        targetY = h - (tonumber(_G.RyPrivUnderOffset3) or 4)
        _wentUnder = true
    elseif (_slot and _slot >= 11) or (not _slot and h >= 11 and h <= 23.15) then
        targetY = h - _under
        _wentUnder = true
    elseif (_slot and _slot >= 1) or (not _slot and h >= -6.9 and h <= 8.9) then
        targetY = tonumber(_G.RyPrivFloor1Y) or -4
        if _G.RyPrivFloor1Platform ~= false then _wentUnder = true end
    else
        targetY = h - _under
        _wentUnder = true
    end
    local _to = Vector3.new(petPos.X, targetY, petPos.Z)
    if hrp and hrp.Parent then
        -- Keep whatever direction we were already facing after cloning in;
        -- only move the position. (CFrame.new(_to) alone would reset the
        -- rotation and snap us to face the plot/brainrot.)
        do
            local _s1 = tonumber(_G.RyPrivGoGlideSpeed1) or 150
            local _t1 = tonumber(_G.RyPrivGoGlideTime1) or 0.03
            local _s2 = tonumber(_G.RyPrivGoGlideSpeed2) or 400
            local _cap = tonumber(_G.RyPrivGoGlideMax) or 2.5
            local _g0 = os.clock()
            while os.clock() - _g0 < _cap do
                if not (hrp and hrp.Parent) then break end
                if LP:GetAttribute("Stealing") or _G.RyPrivTPStop then break end
                local d = _to - hrp.Position
                if d.Magnitude <= 3 then break end
                equipCarpet()
                if _G.RyPrivGoJump == true then
                    local _gh = hrp.Parent and hrp.Parent:FindFirstChildOfClass("Humanoid")
                    if _gh then
                        pcall(function() _gh:ChangeState(Enum.HumanoidStateType.Jumping) end)
                        pcall(function() _gh.Jump = true end)
                    end
                end
                if _G.RyPrivGoZeroEachFrame ~= false then
                    pcall(function()
                        hrp.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
                        hrp.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
                    end)
                end
                local _v = d.Unit * (((os.clock() - _g0) < _t1) and _s1 or _s2)
                if _G.RyPrivGoNoRise ~= false and hrp.Position.Y >= _to.Y - 0.5 and _v.Y > 0 then
                    _v = Vector3.new(_v.X, 0, _v.Z)
                end
                _setFlightVel(hrp, _v)
                RunService.Heartbeat:Wait()
            end
        end
        if _G.RyPrivGoJump == true then
            local _gh = hrp.Parent and hrp.Parent:FindFirstChildOfClass("Humanoid")
            if _gh then
                pcall(function() _gh:ChangeState(Enum.HumanoidStateType.Jumping) end)
                pcall(function() _gh.Jump = true end)
            end
        end
        local _snapCF = CFrame.new(_to) * (hrp.CFrame - hrp.CFrame.Position)
        pcall(function()
            hrp.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
            hrp.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
            hrp.CFrame = _snapCF
        end)
        -- Hold it for a few frames: a single write can be undone by the
        -- humanoid's own physics step or a server correction landing in the
        -- same tick, which would drop us back where we came from.
        for _ = 1, (tonumber(_G.RyPrivSnapHoldFrames) or 8) do
            RunService.Heartbeat:Wait()
            if not (hrp and hrp.Parent) then break end
            if (hrp.Position - _to).Magnitude > 4 then
                pcall(function()
                    hrp.CFrame = _snapCF
                end)
                if _G.RyPrivGoJump == true then
                    local _gh = hrp.Parent and hrp.Parent:FindFirstChildOfClass("Humanoid")
                    if _gh then
                        pcall(function() _gh:ChangeState(Enum.HumanoidStateType.Jumping) end)
                        pcall(function() _gh.Jump = true end)
                    end
                end
            end
        end
    end
    if _wentUnder and _G.RyPrivPlatform ~= false then
        local _platPos = (hrp and hrp.Parent and hrp.Position) or _to
        local _feetY = _platPos.Y - 3
        local _sz = tonumber(_G.RyPrivPlatSize) or 10
        -- was named "ANGVELSTempPlatform" and found again by that name; both the
        -- name and the name-lookup are visible to the game. Track it by reference.
        local _old = _G.__RyPrivPlat
        if _old and _old.Parent then pcall(function() _old:Destroy() end) end
        local _plat = Instance.new("Part")
        _G.__RyPrivPlat = _plat
        _plat.Size = Vector3.new(_sz, 1, _sz)
        _plat.Position = Vector3.new(petPos.X, _feetY - (tonumber(_G.RyPrivPlatDrop) or 1.5), petPos.Z)
        _plat.Anchored = true; _plat.CanCollide = false; pcall(_makeOneWay, _plat); _plat.Transparency = 1
        _plat.Material = Enum.Material.SmoothPlastic; _plat.Parent = workspace
        task.spawn(function()
            local _s = tick()
            while tick() - _s < (tonumber(_G.RyPrivPlatLife) or 20) do
                if LP:GetAttribute("Stealing") then break end
                task.wait(0.1)
            end
            if _plat and _plat.Parent then _plat:Destroy() end
        end)
    end
end

_Stats = game:GetService("Stats")
function _pingMs()
    local ok, p = pcall(function() return LP:GetNetworkPing() * 1000 end)
    if ok and type(p) == "number" and p > 0 then return p end
    local ok2, p2 = pcall(function()
        return _Stats.Network.ServerStatsItem["Data Ping"]:GetValue()
    end)
    if ok2 and type(p2) == "number" and p2 > 0 then return p2 end
    return 0
end
_G.RyPrivPingMs = _pingMs
function _pingAdjustSpeed(spd)
    local thresh = tonumber(_G.RyPrivPingThresh) or 170
    local capped = tonumber(_G.RyPrivHighPingSpeed) or 400
    if _pingMs() >= thresh and spd > capped then return capped end
    return spd
end

function _inVoid(hrp)
    if not hrp or not hrp.Parent then return true end
    local voidY = tonumber(_G.RyPrivVoidY) or -50
    return hrp.Position.Y < voidY
end
function _waitOutOfVoid(timeout)
    local t0 = os.clock()
    local good = 0
    while os.clock() - t0 < (timeout or 12) do
        if _G.RyPrivTPStop then return false end
        local char = LP.Character
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        if hrp and hrp.Parent and not _inVoid(hrp) and math.abs(hrp.AssemblyLinearVelocity.Y) < 12 then
            good += 1
            if good >= 4 then return true end
        else
            good = 0
        end
        RunService.Heartbeat:Wait()
    end
    return false
end

do
    local lastSafe = nil
    local recovering = false
    RunService.Heartbeat:Connect(LPH_NO_VIRTUALIZE(function()
        if _G.RyPrivVoidRecover == false then return end
        local char = LP.Character
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        if not hrp or not hrp.Parent then return end
        local voidY = tonumber(_G.RyPrivVoidY) or -50
        if hrp.Position.Y >= voidY then
            if math.abs(hrp.AssemblyLinearVelocity.Y) < 40 then
                lastSafe = hrp.Position
            end
            return
        end
        if recovering or not lastSafe then return end
        recovering = true

        pcall(function()
            _vzL(hrp)
            _vzA(hrp)
            hrp.CFrame = CFrame.new(lastSafe + Vector3.new(0, 5, 0))
        end)
        task.spawn(function()
            local delay = tonumber(_G.RyPrivVoidRecoverDelay) or 0
            if delay > 0 then task.wait(delay) end
            local vy = tonumber(_G.RyPrivVoidY) or -50
            local tries = 0
            while tries < 80 do
                local c = LP.Character
                local h = c and c:FindFirstChild("HumanoidRootPart")
                if not h or not h.Parent then break end
                if h.Position.Y >= vy and math.abs(h.AssemblyLinearVelocity.Y) < 18 then
                    break
                end
                if lastSafe then
                    pcall(function()
                        _vzL(h)
                        _vzA(h)
                        h.CFrame = CFrame.new(lastSafe + Vector3.new(0, 5, 0))
                    end)
                end
                tries = tries + 1
                RunService.Heartbeat:Wait()
            end
            recovering = false
        end)
    end))
end

isTeleporting = false

task.spawn(function()
    local _since = nil
    while true do
        task.wait(1)
        if isTeleporting then
            _since = _since or os.clock()
            if os.clock() - _since > (tonumber(_G.RyPrivTPWatchdog) or 25) then
                isTeleporting = false
                _G.RyPrivStealHold = false
                _since = nil
                -- through the captured warn: the file shadows `warn` with a
                -- no-op, so this message was never actually reaching anyone
                if _G.RyPrivTPDebug ~= false and type(_RW) == "function" then
                    _RW("[Rymogs priv] watchdog: isTeleporting was stuck -> cleared")
                end
            end
        else
            _since = nil
        end
    end
end)

_G.RyPrivDoClone = doClone
_G.RyPrivIsTeleporting = function() return isTeleporting end

function cframeStepThrough(hrp, waypoints, stepSize)
    if not hrp or not hrp.Parent or #waypoints == 0 then return end
    stepSize = stepSize or 24
    local lockY = hrp.Position.Y
    vizPath(hrp.Position, waypoints)
    local wpIdx = 1
    local deadline = os.clock() + 10
    local _lastFire = 0
    while hrp and hrp.Parent and os.clock() < deadline do
        if _G.RyPrivTPStop then break end
        do
            local char = hrp.Parent
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            if hum and char and not char:FindFirstChild("Grapple Hook") then
                local g = findGrapple()
                if g then pcall(function() hum:EquipTool(g) end) end
            end
            if os.clock() - _lastFire > 0.3 then
                _lastFire = os.clock()
                if char and char:FindFirstChild("Grapple Hook") then
                    -- the grapple fire can yield (cooldown gate / equip wait);
                    -- spawn it so the 24-stud step cadence stays on Heartbeat
                    task.spawn(_fireGrapple)
                end
            end
        end
        local target = waypoints[wpIdx]
        local flat = Vector3.new(target.X - hrp.Position.X, 0, target.Z - hrp.Position.Z)
        local mag = flat.Magnitude
        if mag < 2 then
            wpIdx = wpIdx + 1
            if wpIdx > #waypoints then break end
            RunService.Heartbeat:Wait()
        else
            local hop = math.min(stepSize, mag)
            local nextPos = hrp.Position + flat.Unit * hop
            local _, y = hrp.CFrame:ToEulerAnglesYXZ()
            hrp.CFrame = CFrame.new(Vector3.new(nextPos.X, lockY, nextPos.Z)) * CFrame.Angles(0, y, 0)
            _vzL(hrp)
            _vzA(hrp)
            task.wait(hop / math.clamp(tonumber(_G.RyPrivCFrameSpeed) or 450, 60, 900))
        end
    end
    do
        local hum = hrp and hrp.Parent and hrp.Parent:FindFirstChildOfClass("Humanoid")
        if hum then pcall(function() hum:UnequipTools() end) end
        task.wait(0.05)
        equipCarpet()
    end
    if hrp and hrp.Parent then
        _vzL(hrp)
        _vzA(hrp)
    end
end

function _isStraightRoute(fromPos, route)
    if not route or #route <= 1 then return true end
    local turns, lastDir, prev = 0, nil, fromPos
    for _, wp in ipairs(route) do
        local seg = wp - prev
        if seg.Magnitude > 1 then
            local dir = seg.Unit
            if lastDir and dir:Dot(lastDir) < 0.94 then turns = turns + 1 end
            lastDir = dir
        end
        prev = wp
    end
    return turns <= (tonumber(_G.RyPrivStraightMaxTurns) or 1)
end

function doVelocityTP(forceGrapple, overridePet)
    if isTeleporting then return end
    if LP:GetAttribute("Stealing") == true then return end
    isTeleporting = true
    if _G.RyPrivDisarmSteal then _G.RyPrivDisarmSteal() end
    _G.RyPrivTPStop = false
    clearViz()
    if not NetModule then pcall(loadNet) end

    local char = LP.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if not hrp or not hum then isTeleporting = false; return end

    pcall(function()
        hrp.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
        hrp.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
    end)

    if _inVoid(hrp) or hrp.AssemblyLinearVelocity.Y < -40 then
        _waitOutOfVoid(12)
        if _G.RyPrivTPStop then isTeleporting = false; return end
        char = LP.Character
        hrp = char and char:FindFirstChild("HumanoidRootPart")
        hum = char and char:FindFirstChildOfClass("Humanoid")
        if not hrp or not hum then isTeleporting = false; return end
    end

    -- overridePet: a fixed destination (Base Camp's empty slot). Skips the
    -- scan and the pick and runs the rest of the pipeline on it as-is.
    local allPets = overridePet and { overridePet } or scanForTP()
    if #allPets == 0 then
        local _t0 = os.clock()
        while #allPets == 0 and os.clock() - _t0 < (tonumber(_G.RyPrivTPScanWait) or 10) do
            task.wait(0.05)
            allPets = scanForTP()
        end
    end
    if #allPets == 0 then isTeleporting = false; return end

    if not overridePet and not (type(_G.RyPrivStealTargetUID) == "string" and _G.RyPrivStealTargetUID ~= "") then
        local pool, seen = {}, {}
        local function _absorb(list)
            if type(list) ~= "table" then return end
            for _, p in ipairs(list) do
                if not p.conveyor and p.plot and p.slot ~= nil then
                    local uid = tostring(p.plot) .. "_" .. tostring(p.slot)
                    local ex = seen[uid]
                    if not ex then
                        seen[uid] = p
                        pool[#pool + 1] = p
                    elseif (p.mps or 0) > (ex.mps or 0) then
                        for i = 1, #pool do
                            if pool[i] == ex then pool[i] = p break end
                        end
                        seen[uid] = p
                    end
                end
            end
        end
        _absorb(allPets)
        for _ = 1, (tonumber(_G.RyPrivTPScans) or 2) do
            task.wait(tonumber(_G.RyPrivTPScanGap) or 0.05)
            local _ok, _more = pcall(scanForTP)
            if _ok then _absorb(_more) end
        end
        if #pool > 0 then allPets = pool end
    end

    local pet
    if overridePet then
        pet = overridePet
    elseif type(_G.RyPrivStealTargetUID) == "string" and _G.RyPrivStealTargetUID ~= "" then
        pet = _findStealTarget(allPets)
        if not pet then isTeleporting = false; return end
    else
        local prio, best
        for _, p in ipairs(allPets) do
            if not p.conveyor then
                if p._pri and (not prio or p._pri < prio._pri) then prio = p end
                if not best or (p.mps or 0) > (best.mps or 0) then best = p end
            end
        end
        pet = prio or best or allPets[1]
    end
    local petPos = pet.position
    local petName = pet.name
    -- Past this line a target is chosen and the run is really happening. The
    -- callers use this stamp to tell "teleported" from "bailed out before it
    -- started", which is the difference between retrying and not.
    _G.RyPrivTPStartedAt = os.clock()
    if _G.RyPrivArmSteal and not overridePet then pcall(_G.RyPrivArmSteal, pet) end
    _G.RyPrivStealSay(string.format("TP start -> %s [%s/%s] mps=%s pri=%s of %d", tostring(pet and pet.name), tostring(pet and pet.plot), tostring(pet and pet.slot), tostring(pet and pet.mps), tostring(pet and pet._pri), #allPets))

    _G.RyPrivStealHold = true

    pcall(fireGrapple)

    local adjY = petPos.Y
    if TALL_PETS[petName] then adjY = petPos.Y - TALL_OFFSET end
    local coordTable = adjY > 23.15 and MK_UPPER or MK_LOWER

    if petPos.Y <= 8.9 and isPlotUnlocked(pet.plot) then
        carpetEngage(forceGrapple)
        vZero(hrp)
        local _to = Vector3.new(petPos.X, -4, petPos.Z)
        local route = computeRoute(hrp.Position, _to, nil)
        if not route or #route == 0 then route = { _to } end
        local _straight = _isStraightRoute(hrp.Position, route)
        local _obSpeed = math.clamp(tonumber(_G.TPVelocity) or 400, 200, 750)
        if _straight then
            _obSpeed = math.clamp(tonumber(_G.RyPrivStraightSpeed) or 300, 100, 500)
        end
        do
            local _len, _prev = 0, hrp.Position
            for _, wp in ipairs(route) do _len = _len + (wp - _prev).Magnitude; _prev = wp end
            if _len < 100 then
                _obSpeed = math.clamp(tonumber(_G.RyPrivCloseSpeed) or 80, 20, 400)
            end
        end
        _obSpeed = _pingAdjustSpeed(_obSpeed)
        velMoveThrough(hrp, route, _obSpeed, true, true)
        if hrp and hrp.Parent then
            _vzL(hrp)
            _vzA(hrp)
        end
        _G.RyPrivStealHold = false
        isTeleporting = false
        if _G.RyPrivTPStop then return end
        return
    end

    local closestData, skyKey = findClosest(petPos, coordTable)
    if not closestData or not skyKey then _G.RyPrivStealHold = false; isTeleporting = false; return end

    local destPos = closestData.coord

    carpetEngage(forceGrapple)
    vZero(hrp)

    local facingDir = closestData.facing == "NORTH" and Vector3.new(0, 0, -1) or Vector3.new(0, 0, 1)

    local _frontApproach = false
    do
        -- Base approach:
        --   * floor: only pets above Y 25 (3rd floor) route on the upper level;
        --     2nd floor routes low unless _G.RyPrivSecondFloorRoute == "High"
        --   * front = the base's entrance line at YOUR Z, clamped +-18
        --   * side  = the nearest side platform belonging to THAT plot, with
        --     its X pulled toward you by up to 20
        --   * whichever is closer to you (flat XZ distance) wins.
        -- _G.RyPrivApproachMode = "Front" / "Side" forces one ("Both" default).
        local isThirdFloor = petPos.Y > 25
        local isSecondFloor = (not isThirdFloor) and petPos.Y > 10
        local isUpper = isThirdFloor or (isSecondFloor and _G.RyPrivSecondFloorRoute == "High")
        local idx = getClosestBaseIdx(petPos)

        local frontCoord, frontFace = buildFrontCandidate(idx, isUpper, hrp.Position.Z)

        local sideTable = isUpper and SIDE_TP_HIGH or SIDE_TP_LOW
        local closestSide, closestSideDist = nil, math.huge
        for _, data in ipairs(sideTable) do
            if data.plot == idx then
                local c = data.coord
                local d = math.sqrt((hrp.Position.X - c.X) ^ 2 + (hrp.Position.Z - c.Z) ^ 2)
                if d < closestSideDist then closestSideDist = d; closestSide = data end
            end
        end
        local sidePos = nil
        if closestSide then
            local c = closestSide.coord
            local dx = math.clamp(hrp.Position.X - c.X, -SIDE_X_RANGE, SIDE_X_RANGE)
            sidePos = Vector3.new(c.X + dx, c.Y, c.Z)
        end

        local function _flatDist(a, b) return math.sqrt((a.X - b.X) ^ 2 + (a.Z - b.Z) ^ 2) end
        local distToFront = _flatDist(hrp.Position, frontCoord)
        local distToSide = sidePos and _flatDist(hrp.Position, sidePos) or math.huge

        local _am = _G.RyPrivApproachMode or "Both"
        local pickedFront
        if _am == "Front" then
            pickedFront = true
        elseif _am == "Side" then
            pickedFront = (sidePos == nil)
        else
            pickedFront = (distToFront <= distToSide)
        end

        local target = pickedFront and frontCoord or sidePos
        if pickedFront then
            facingDir = frontFace
        else
            facingDir = (closestSide.facing == "NORTH") and Vector3.new(0, 0, -1) or Vector3.new(0, 0, 1)
        end
        -- fly a little above upper-level platforms
        if target.Y > 0 then target = Vector3.new(target.X, target.Y + 3.1, target.Z) end
        destPos = target
        _frontApproach = pickedFront
        _G.RyPrivLastApproach = string.format("%s base=%d %s front=%.0f side=%.0f",
            pickedFront and "FRONT" or "SIDE", idx, isUpper and "upper" or "lower", distToFront, distToSide)
        if _G.RyPrivTPDebug ~= false then
            warn(string.format(
                "[Rymogs priv] PICK baseIdx=%d %s floor=%s | pet=(%.0f,%.0f,%.0f) plot=%s | dest=(%.0f,%.0f,%.0f) | me=(%.0f,%.0f,%.0f) | front=%.0f side=%.0f",
                idx, pickedFront and "FRONT" or "SIDE", isUpper and "upper" or "lower",
                petPos.X, petPos.Y, petPos.Z, tostring(pet.plot),
                destPos.X, destPos.Y, destPos.Z,
                hrp.Position.X, hrp.Position.Y, hrp.Position.Z,
                distToFront, distToSide))
        end
    end

    if facingDir and facingDir.Magnitude > 0.1 then
        local axis = facingDir.Unit
        local toPlayer = hrp.Position - destPos
        local sign = (axis:Dot(toPlayer) >= 0) and 1 or -1
        destPos = destPos + axis * sign * (tonumber(_G.RyPrivCloneBackoff) or 0.5)
    end

    local _route = computeRoute(hrp.Position, destPos, facingDir, nil, true)

    local ASCEND_STEP = 10
    local _stepped = {}
    do
        local prev = hrp.Position
        for _, wp in ipairs(_route) do
            local dy = wp.Y - prev.Y
            if dy > ASCEND_STEP * 1.5 then
                local n = math.ceil(dy / ASCEND_STEP)
                for s = 1, n - 1 do
                    local t = s / n
                    _stepped[#_stepped + 1] = Vector3.new(
                        prev.X + (wp.X - prev.X) * t,
                        prev.Y + dy * t,
                        prev.Z + (wp.Z - prev.Z) * t
                    )
                end
            end
            _stepped[#_stepped + 1] = wp
            prev = wp
        end
    end
    local _mainSpeed = math.clamp(tonumber(_G.TPVelocity) or 400, 200, 750)
    do
        local _len, _prev = 0, hrp.Position
        for _, wp in ipairs(_route) do _len = _len + (wp - _prev).Magnitude; _prev = wp end
        if _len < 100 then
            _mainSpeed = math.clamp(tonumber(_G.RyPrivCloseSpeed) or 80, 20, 400)
        end
    end
    _mainSpeed = _pingAdjustSpeed(_mainSpeed)
    velMoveThrough(hrp, _stepped, _mainSpeed, true, true)
    if _G.RyPrivTPStop then
        if hrp and hrp.Parent then vZero(hrp) end
        _G.RyPrivStealHold = false
        isTeleporting = false
        return
    end

    do
        local above = destPos + Vector3.new(0, 16, 0)
        local _t0 = os.clock()
        while os.clock() - _t0 < 1.5 do
            if not hrp or not hrp.Parent then break end
            if LP:GetAttribute("Stealing") or _G.RyPrivTPStop then break end
            equipCarpet()
            local d = above - hrp.Position
            local flat = Vector3.new(d.X, 0, d.Z).Magnitude
            if flat <= 3 and hrp.Position.Y >= destPos.Y then break end
            if d.Y > 3 then
                local _hum = hrp.Parent and hrp.Parent:FindFirstChildOfClass("Humanoid")
                if _hum then
                    local st = _hum:GetState()
                    if st ~= Enum.HumanoidStateType.Jumping and st ~= Enum.HumanoidStateType.Freefall then
                        pcall(function() _hum:ChangeState(Enum.HumanoidStateType.Jumping) end)
                        pcall(function() _hum.Jump = true end)
                    end
                end
            end
            _setFlightVel(hrp, d.Unit * math.min(math.max(d.Magnitude * 8, 55), 320))
            _vzA(hrp)
            RunService.Heartbeat:Wait()
        end
    end

    do
        local _runCap = _frontApproach and (tonumber(_G.RyPrivFrontRunIn) or 120) or 320
        local _t0 = os.clock()
        local _bestMag, _bestT = math.huge, os.clock()
        while os.clock() - _t0 < 4 do
            if not hrp or not hrp.Parent then break end
            if LP:GetAttribute("Stealing") or _G.RyPrivTPStop then break end
            equipCarpet()
            local diff = destPos - hrp.Position
            local mag = diff.Magnitude
            if mag <= 3 then break end
            if mag < _bestMag - 0.5 then _bestMag = mag; _bestT = os.clock()
            elseif os.clock() - _bestT > 0.6 then break end
            if diff.Y > 3 then
                local _hum = hrp.Parent and hrp.Parent:FindFirstChildOfClass("Humanoid")
                if _hum then
                    local st = _hum:GetState()
                    if st ~= Enum.HumanoidStateType.Jumping and st ~= Enum.HumanoidStateType.Freefall then
                        pcall(function() _hum:ChangeState(Enum.HumanoidStateType.Jumping) end)
                        pcall(function() _hum.Jump = true end)
                    end
                end
            end
            _setFlightVel(hrp, diff.Unit * math.min(math.max(mag * 8, 55), _runCap))
            _vzA(hrp)
            RunService.Heartbeat:Wait()
        end
    end

    if hrp and hrp.Parent and not _G.RyPrivTPStop then
        local _tol = tonumber(_G.RyPrivTPReachTolerance) or 4.5
        local _retryMax = math.clamp(tonumber(_G.RyPrivTPRetryCount) or 2, 0, 4)
        for _retry = 1, _retryMax do
            if not (hrp and hrp.Parent) or _G.RyPrivTPStop then break end
            local _flatOff = Vector3.new(destPos.X - hrp.Position.X, 0, destPos.Z - hrp.Position.Z).Magnitude
            local _yOff = math.abs(destPos.Y - hrp.Position.Y)
            if _flatOff <= _tol and _yOff <= 6 then break end
            if _G.RyPrivTPDebug ~= false then
                warn(string.format("[Rymogs priv] REACH retry %d/%d: flat=%.1f y=%.1f -> fresh route", _retry, _retryMax, _flatOff, _yOff))
            end
            task.wait(tonumber(_G.RyPrivTPRetryDelay) or 0.08)
            local _fresh = computeRoute(hrp.Position, destPos, facingDir, nil, true)
            if not _fresh or #_fresh == 0 then _fresh = { destPos } end
            local _freshStepped = {}
            local _prev = hrp.Position
            for _, _wp in ipairs(_fresh) do
                local _dy = _wp.Y - _prev.Y
                if _dy > 15 then
                    local _n = math.ceil(_dy / 10)
                    for _s = 1, _n - 1 do
                        local _t = _s / _n
                        _freshStepped[#_freshStepped + 1] = Vector3.new(
                            _prev.X + (_wp.X - _prev.X) * _t,
                            _prev.Y + _dy * _t,
                            _prev.Z + (_wp.Z - _prev.Z) * _t)
                    end
                end
                _freshStepped[#_freshStepped + 1] = _wp
                _prev = _wp
            end
            local _retrySpeed = _mainSpeed
            velMoveThrough(hrp, _freshStepped, _retrySpeed, true, true)
            if hrp and hrp.Parent then
                _vzL(hrp); _vzA(hrp)
            end
        end
    end

    if hrp and hrp.Parent then
        hrp.CFrame = CFrame.new(hrp.Position, hrp.Position + facingDir)
    end
    vZero(hrp)

    local syncFrames = (_G.RyPrivTPSafeMode ~= false) and 8 or 5
    local syncConn
    syncConn = RunService.Heartbeat:Connect(LPH_NO_VIRTUALIZE(function()
        if not hrp or not hrp.Parent then syncConn:Disconnect(); return end
        syncFrames = syncFrames - 1
        hrp.CFrame = CFrame.new(destPos, destPos + facingDir)
        _vzL(hrp)
        _vzA(hrp)
        if syncFrames <= 0 then syncConn:Disconnect() end
    end))

    for _ = 1, 20 do
        task.wait(0.05)
        if hum.FloorMaterial ~= Enum.Material.Air then break end
    end

    do
        local stable, _st0 = 0, os.clock()
        while os.clock() - _st0 < 3 do
            if _G.RyPrivTPStop then break end
            local _hrp = char and char:FindFirstChild("HumanoidRootPart")
            if not _hrp or not _hrp.Parent then break end
            local flat = (Vector3.new(_hrp.Position.X, 0, _hrp.Position.Z) - Vector3.new(destPos.X, 0, destPos.Z)).Magnitude
            if flat <= 3.5 and math.abs(_hrp.Position.Y - destPos.Y) <= 4 then
                stable = stable + 1
                if stable >= 4 then break end
            else
                stable = 0
                pcall(function() _hrp.CFrame = CFrame.new(destPos, destPos + facingDir) end)
                _vzL(_hrp)
                _vzA(_hrp)
            end
            RunService.Heartbeat:Wait()
        end
    end
    local _ahrp = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
    local _clonePos = (_ahrp and _ahrp.Parent and _ahrp.Position) or destPos

    local _clonePlat = Instance.new("Part")
    -- was "RyPrivHubClonePlatform"; an unnamed Part is indistinguishable from
    -- the thousands the game already has in Workspace.
    _clonePlat.Size = Vector3.new(12, 1, 12)
    _clonePlat.Position = Vector3.new(_clonePos.X, _clonePos.Y - 3, _clonePos.Z)
    _clonePlat.Anchored = true
    _clonePlat.CanCollide = true
    _clonePlat.Transparency = 1
    _clonePlat.Material = Enum.Material.SmoothPlastic
    _clonePlat.Parent = workspace

    if _ahrp and _ahrp.Parent then
        _vzL(_ahrp)
        _vzA(_ahrp)
    end

    local _preClonePos, _preCloneChar
    do
        _preCloneChar = LP.Character
        local _h = _preCloneChar and _preCloneChar:FindFirstChild("HumanoidRootPart")
        _preClonePos = _h and _h.Position or destPos
    end
    local _charAdded = false
    local _caConn = LP.CharacterAdded:Connect(function() _charAdded = true end)

    _G.RyPrivStealHold = false

    task.wait(tonumber(_G.LandingDelay) or tonumber(_G.TPCloneDelay) or 0.35)

    if facingDir and facingDir.Magnitude > 0.1 then
        local _pinHum = LP.Character and LP.Character:FindFirstChildOfClass("Humanoid")
        if _pinHum then pcall(function() _pinHum.AutoRotate = false end) end
        for _ = 1, 4 do
            local _h = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
            if not _h or not _h.Parent then break end
            pcall(function()
                _h.CFrame = CFrame.new(_h.Position, _h.Position + facingDir)
                _vzL(_h)
                _vzA(_h)
            end)
            RunService.Heartbeat:Wait()
        end
    end

    doClone()
    if _clonePlat then
        local _plat = _clonePlat
        _clonePlat = nil
        task.delay(1.5, function() pcall(function() _plat:Destroy() end) end)
    end
    do
        local _t0 = os.clock()
        repeat
            if _charAdded then break end
            if LP.Character ~= _preCloneChar then break end
            local _h = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
            if _h then
                local _dx = _h.Position.X - _preClonePos.X
                local _dz = _h.Position.Z - _preClonePos.Z
                if (_dx * _dx + _dz * _dz) > 1 then break end
            end
            RunService.Heartbeat:Wait()
        until os.clock() - _t0 > (tonumber(_G.RyPrivCloneSettle) or 0.5)
    end
    if _caConn then _caConn:Disconnect() end

    pcall(function()
        local _rh = LP.Character and LP.Character:FindFirstChildOfClass("Humanoid")
        if _rh then _rh.AutoRotate = true end
    end)

    goToBrainrot(petPos, pet and pet.slot)
    isTeleporting = false
    if _G.RyPrivTPStop then return end
    if _G.RyPrivArmSteal and not overridePet then pcall(_G.RyPrivArmSteal, pet) end
    _G.RyPrivStealSay("TP arrive -> " .. tostring(pet and pet.name) .. " [" .. tostring(pet and pet.plot) .. "/" .. tostring(pet and pet.slot) .. "]")
end

_manualTPBusy = false
function manualFullTP()
    if _manualTPBusy or isTeleporting then return end
    if LP:GetAttribute("Stealing") == true then return end
    _manualTPBusy = true
    if _G.RyPrivScanProfile and _G.RyPrivScanMode == "fast" then _G.RyPrivScanProfile("normal") end
    local okAll = pcall(function()
        local _t0 = os.clock()
        local _lastN, _lastTop = -1, nil
        repeat
            local ok, pets = pcall(scanForTP)
            if ok and pets and #pets > 0 then
                local top = tostring(pets[1].plot) .. "_" .. tostring(pets[1].slot)
                if #pets == _lastN and top == _lastTop then break end
                _lastN, _lastTop = #pets, top
                task.wait(tonumber(_G.RyPrivScanSettle) or 0.06)
            else
                task.wait(0.05)
            end
        until os.clock() - _t0 > (tonumber(_G.RyPrivTPMaxWait) or 4)
        -- Same retry as the join path: only counts as done once a run really
        -- starts, so a bail-out before target selection tries again.
        local tries = math.max(1, math.floor(tonumber(_G.RyPrivManualTPTries) or 2))
        for attempt = 1, tries do
            local before = tonumber(_G.RyPrivTPStartedAt) or 0
            doVelocityTP(true)
            if (tonumber(_G.RyPrivTPStartedAt) or 0) > before then break end
            if attempt < tries then task.wait(tonumber(_G.RyPrivManualTPRetryGap) or 0.8) end
        end
    end)
    _manualTPBusy = false
    return okAll
end
_G.RyPrivStartSideTP = manualFullTP

if type(_G.SHARED_PRIORITY_ITEMS) ~= "table" or #_G.SHARED_PRIORITY_ITEMS == 0 then
_G.SHARED_PRIORITY_ITEMS = {
    "Headless Horseman","Strawberry Elephant","Signore Carapace","John Pork","Meowl",
    "Elefanto Frigo","Arcadragon","Skibidi Toilet","Griffin","Antonio",
    "Dragon Aquanini","Dragon Gingerini","Love Love Bear","Kalika Bros","Moby Bros",
    "Grabatron","Jelly Moby","La Supreme Combinasion","Ginger Gerat","Digi Narwhal",
    "Hydra Dragon Cannelloni","Hydra Bunny","Bunny and Eggy","Kraken","Fishino Clownino",
    "Tirilikalika Tirilikalako","Pancake and Syrup","Dragon Cannelloni","Sammyni Cakini","Ketupat Bros",
    "Bumbatron","Venuspino","Dug dug dug","La Casa Boo","Rico Dinero",
    "Foxini Lanternini","Duggy Bros","Rosey and Teddy","Globa Steppa","Los Hackers",
    "Cerberus","Fragrama and Chocrama","Cooki and Milki","La Secret Combinasion","Burguro and Fryuro",
    "Capitano Moby","Spooky and Pumpky","Garama and Madundung","Popcuru and Fizzuru","Pizza and Ranch",
    "Reinito Sleighito","Tenini Ballini","Fragola La La La","Ketchuru and Musturu","Tralaledon",
    "Tictac Sahur","Ketupat Kepat","Tang Tang Keletang","Orcaledon","La Ginger Sekolah",
    "Los Spaghettis","Lavadorito Spinito","Swaggy Bros","La Taco Combinasion","Los Primos",
    "Los Chillis","Chillin Chili","Tuff Toucan","W or L","Chipso and Queso",
    "Guest 666","Money Money Reindeer","Quackini Snackini","Los Sekolahs","Los Tacoritas",
    "Los Amigos","Fortunu and Cashuru","Jolly Jolly Sahur","Boppin Bunny","Gym Bros",
    "Los Cupids","Festive 67","Celularcini Viciosini","Cloverat Clapat","La Food Combinasion",
    "Hopilikalika Hopilikalako","Celestial Pegasus","Sammyni Fattini","Money Money Bros","La Spooky Grande",
    "Cash or Card","Swag Soda","Los Planitos","Lovin Rose","Tacorita Bicicleta",
    "Los Jolly Combinasionas","La Romantic Grande","La Easter Grande","Los Hotspotsitos","Rosetti Tualetti",
    "Los Bros","Gobblino Uniciclino","Chicleteira Cupideira","La Extinct Grande","Las Sis",
    "Nacho Spyder","Gold Gold Gold","Los Mariachis","Snailo Clovero","La Jolly Grande",
    "Los Candies","Churrito Bunnito","Bananito","Eviledon","Los 67",
    "Los Sweethearts","Noo my Heart","La Lucky Grande","Ventoliero Pavonero","Baskito",
    "Chimnino","Los Puggies","Camera Ramena","Los 25","Spinny Hammy",
    "Money Money Puggy","Cigno Fulgoro","Los Spooky Combinasionas","Chicleteira Noelteira","Mariachi Corazoni",
    "Tacorillo Crocodillo","Noo my Gold","Los Mobilis","Mieteteira Bicicleteira","DJ Panda",
    "Los Combinasionas","Nuclearo Dinossauro","Bacuru and Egguru","Spaghetti Tualetti","La Grande Combinasion",
    "Esok Sekolah",
}
end

-- CLONE NOW and MANUAL TP. Both were hardcoded (V, and T via _stp_tpKeyName
-- with no way to see or change it), and MANUAL TP's default T collided with
-- the AUTO TP toggle key further down -- one press did both. They are ordinary
-- rebindable keys now and the Settings KEYBINDS tab lists them.
_G.RyPrivCloneNowKey = _G.RyPrivCloneNowKey or "V"
if type(_G._stp_tpKeyName) ~= "string" or _G._stp_tpKeyName == "" then
    _G._stp_tpKeyName = "T"
end
UIS.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.UserInputType ~= Enum.UserInputType.Keyboard then return end
    if _G.RyPrivBindListening then return end
    local kn = input.KeyCode.Name
    if type(_G.RyPrivCloneNowKey) == "string" and kn == _G.RyPrivCloneNowKey then
        task.spawn(function() pcall(doClone) end)
    end
    if type(_G._stp_tpKeyName) == "string" and kn == _G._stp_tpKeyName then
        task.spawn(function() pcall(manualFullTP) end)
    end
end)

task.spawn(function() pcall(loadModules) pcall(loadNet) end)

_G.RyPrivChannelsReady = false

task.spawn(function()
    local char = LP.Character or LP.CharacterAdded:Wait()

    local hrpReady, humReady = false, false
    task.spawn(function() char:WaitForChild("HumanoidRootPart", 20); hrpReady = true end)
    task.spawn(function() char:WaitForChild("Humanoid", 20); humReady = true end)
    local _tw0 = os.clock()
    while (not hrpReady or not humReady) and os.clock() - _tw0 < 20 do
        RunService.Heartbeat:Wait()
    end
    pcall(loadModules); pcall(loadNet)
    if _G.RyPrivAutoTP == false then return end

    if _G.RyPrivWaitForTools ~= false and type(_G.RyPrivToolsReady) == "function" then
        local _tt0 = os.clock()
        local _tcap = tonumber(_G.RyPrivToolWait) or 30
        while os.clock() - _tt0 < _tcap do
            local ok, ready = pcall(_G.RyPrivToolsReady)
            if ok and ready then break end
            if _G.RyPrivAutoTP == false then return end
            task.wait(0.2)
        end
    end
    pcall(carpetEngage)
    do
        local _d = tonumber(_G._stp_tpDelay) or tonumber(_G.TPDelay) or 0
        if _d > 0 then task.wait(_d) end
    end

    local _w0 = os.clock()
    local _wMax = tonumber(_G.RyPrivAutoTPWait) or 20
    while os.clock() - _w0 < _wMax do
        if _G.RyPrivAutoTP == false then return end
        if LP:GetAttribute("Stealing") == true then return end
        local ok, pets = pcall(scanForTP)
        if ok and pets and #pets > 0 then break end
        task.wait(tonumber(_G.RyPrivAutoTPPoll) or 0.05)
    end

    -- This was one shot: if doVelocityTP bailed before picking a target -- no
    -- pets yet, character not ready, still in the void -- nothing ever tried
    -- again and the join simply never teleported. Retry a few times, and stop
    -- as soon as a run actually starts (RyPrivTPStartedAt moves).
    local tries = math.max(1, math.floor(tonumber(_G.RyPrivAutoTPTries) or 3))
    for attempt = 1, tries do
        if _G.RyPrivAutoTP == false then break end
        if LP:GetAttribute("Stealing") == true then break end
        local before = tonumber(_G.RyPrivTPStartedAt) or 0
        pcall(doVelocityTP)
        if (tonumber(_G.RyPrivTPStartedAt) or 0) > before then break end
        if attempt < tries then
            task.wait(tonumber(_G.RyPrivAutoTPRetryGap) or 1.5)
        end
    end
    if _G.RyPrivScanProfile then _G.RyPrivScanProfile("normal") end
end)

do
    local Players = game:GetService("Players")
    local RunService = game:GetService("RunService")
    local LP = Players.LocalPlayer
    local PG = LP:WaitForChild("PlayerGui")

    local function diag()
        pcall(loadModules)
        local Plots = workspace:FindFirstChild("Plots")
        local nPlots = Plots and #Plots:GetChildren() or 0
        local nCh, nOwn, nAl = 0,0,0
        if Plots then
            for _, plot in ipairs(Plots:GetChildren()) do
                local ch = getPlotChannel(plot.Name)
                if ch then
                    nCh = nCh + 1
                    if ownerInGame(ch) then nOwn = nOwn + 1 end
                    if channelGet(ch, "AnimalList") then nAl = nAl + 1 end
                end
            end
        end
        local ok, pets = pcall(scanAllPets)
        local nPets = (ok and pets and #pets) or 0
        local msg = string.format("plots=%d ch=%d owner=%d animals=%d PETS=%d", nPlots, nCh, nOwn, nAl, nPets)
        print("[TP TEST] " .. msg)
        if nPets > 0 and pets[1] then
            print("[TP TEST] top=", pets[1].name, pets[1].plot, pets[1].slot)
        end
        return nPets
    end
    -- The Settings panel's MANUAL TP button runs this first so it can say
    -- "no brainrots" instead of silently doing nothing.
    _G.RyPrivTPDiag = diag

    local function findStealPrompt(pet)
        if not pet then return nil end
        if pet.plot and pet.slot then
            local plots = workspace:FindFirstChild("Plots")
            local plot = plots and plots:FindFirstChild(pet.plot)
            local podiums = plot and plot:FindFirstChild("AnimalPodiums")
            local podium = podiums and podiums:FindFirstChild(tostring(pet.slot))
            if podium then
                local base = podium:FindFirstChild("Base")
                local spawn = base and base:FindFirstChild("Spawn")
                local attach = spawn and spawn:FindFirstChild("PromptAttachment")
                if attach then
                    for _, p in ipairs(attach:GetChildren()) do
                        if p:IsA("ProximityPrompt") then return p end
                    end
                end
                for _, d in ipairs(podium:GetDescendants()) do
                    if d:IsA("ProximityPrompt") then return d end
                end
            end
        end
        return nil
    end

    local InternalStealCache = {}
    local STEAL_HOLD_DURATION = 1.3
    local STEAL_PROXIMITY = tonumber(_G.RyPrivStealProximity) or 28
    local _stealHoldStart, _stealHoldActive = 0, false
    local _stealTarget, _stealArmedAt = nil, 0
    local _lastTargetPick, _lastPickUid, _currentTargetName = 0, nil, nil
    local _stealLastScan, _autoLastScan = 0, 0
    local _inRangeLast = 0
    local _armCheckAt = 0
    local _petsCache, _petsCacheAt = nil, 0

    -- one scan shared by the picker, the in-range check and the watchdog
    local function _scanCached(now, maxAge)
        if _petsCache and (now - _petsCacheAt) <= (maxAge or 0.15) then return _petsCache end
        local ok, pets = pcall(scanAllPets)
        if ok and type(pets) == "table" then
            _petsCache, _petsCacheAt = pets, now
            return pets
        end
        return nil
    end

    -- is the brainrot still on that podium, per the game's own data
    local function _petListed(plot, slot)
        local ch = getPlotChannel(plot)
        local al = ch and channelGet(ch, "AnimalList")
        if type(al) ~= "table" then return true, false end
        local e = al[slot]
        if e == nil then e = al[tonumber(slot) or -1] end
        if e == nil then e = al[tostring(slot)] end
        return e ~= nil, true
    end

    local function _prox()
        return tonumber(_G.RyPrivStealProximity) or STEAL_PROXIMITY
    end
    local function _holdDur()
        return tonumber(_G.RyPrivStealHoldDuration) or STEAL_HOLD_DURATION
    end

    -- after each attempt: did the brainrot actually leave that podium?
    -- one in-range miss flips the method, so the next grab already uses the other one
    local _cooldownUntil = 0
    local function _verifyFire(pet, path, judge)
        task.delay(tonumber(_G.RyPrivVerifyDelay) or 0.5, function()
            if not (pet and pet.plot and pet.slot ~= nil) then return end
            local listed, known = true, false
            pcall(function() listed, known = _petListed(pet.plot, pet.slot) end)
            if not known then return end
            if not listed then
                _G.RyPrivStealFails = 0
                _cooldownUntil = 0
                _G.RyPrivStealSay("steal landed (" .. tostring(path) .. ")")
                return
            end
            if not judge then _cooldownUntil = 0 return end
            if _G.RyPrivAutoSwapMethod ~= true then return end
            _G.RyPrivStealFails = (_G.RyPrivStealFails or 0) + 1
            if _G.RyPrivStealFails >= (tonumber(_G.RyPrivFailsBeforeSwap) or 1) then
                _G.RyPrivStealFails = 0
                if path == "remote" then
                    _G.RyPrivRemoteStealOn = false
                    _G.RyPrivStealSay("remote missed -> prompt mode")
                else
                    _G.RyPrivRemoteStealOn = true
                    _G.RyPrivStealSay("prompt missed -> remote mode")
                end
            end
        end)
    end
    local function _holdBusy()
        if _stealHoldActive
            and (tick() - _stealHoldStart) < (_holdDur() + (tonumber(_G.RyPrivStealRetryGap) or 0.15)) then
            return true
        end
        return tick() < _cooldownUntil
    end

    local _armedPet, _armLostSince = nil, 0
    local function _armLivePos(p)
        if not (p and p.plot and p.slot ~= nil) then return nil end
        local plots = workspace:FindFirstChild("Plots")
        local plot = plots and plots:FindFirstChild(p.plot)
        if not plot then return nil end
        local ok, pos = pcall(getPetPosition, plot, p.slot)
        if ok then return pos end
        return nil
    end
    _G.RyPrivArmSteal = function(pet)
        if type(pet) ~= "table" or type(pet.plot) ~= "string" or pet.slot == nil then return end
        _armedPet = {
            name = pet.name, index = pet.index, mps = pet.mps, _pri = pet._pri,
            plot = pet.plot, slot = tostring(pet.slot), position = pet.position,
        }
        _armLostSince = 0
        _stealTarget = _armedPet
        _stealArmedAt = os.clock()
        if type(pet.name) == "string" and pet.name ~= "" then _currentTargetName = pet.name end
    end
    _G.RyPrivDisarmSteal = function()
        _armedPet, _armLostSince = nil, 0
    end
    _G.RyPrivArmedName = function() return _armedPet and _armedPet.name or nil end

    -- Grey with light blue accents (was near-black with green).
    local UIC = {
        bg    = Color3.fromRGB(30, 32, 36),
        bg2   = Color3.fromRGB(40, 43, 48),
        card  = Color3.fromRGB(52, 56, 62),
        track = Color3.fromRGB(64, 69, 77),
        line  = Color3.fromRGB(92, 99, 110),
        acc   = Color3.fromRGB(96, 178, 235),
        acc2  = Color3.fromRGB(155, 212, 248),
        txt   = Color3.fromRGB(240, 243, 247),
        dim   = Color3.fromRGB(158, 166, 178),
    }
    -- One shared see-through amount for panels and controls, so the whole look
    -- moves together. _G.RyPrivAlpha overrides it at runtime.
    local function UIAlpha() return math.clamp(tonumber(_G.RyPrivAlpha) or 0.12, 0, 0.9) end
    local UIFB = Enum.Font.GothamBold

    local function mk(class, parent, props)
        local o = Instance.new(class)
        for k, v in pairs(props or {}) do o[k] = v end
        o.Parent = parent
        return o
    end
    local function corner(o, r) mk("UICorner", o, { CornerRadius = UDim.new(0, 0) }) return o end
    local function stroke(o, col) mk("UIStroke", o, { Color = col or UIC.line, Thickness = 1 }) return o end

    pcall(function()
        local _oldDbg = LP:FindFirstChild("PlayerGui")
        _oldDbg = _oldDbg and _oldDbg:FindFirstChild("RyPrivStealDbg")
        if _oldDbg then _oldDbg:Destroy() end
    end)

    -- This bar was the one piece of UI parented straight into PlayerGui under a
    -- self-identifying name, so the game could both see it and name it. Prefer
    -- the executor's hidden container; fall back to PlayerGui under a native-
    -- looking name, and track it by reference rather than re-finding by name.
    -- ── STEAL BAR ─────────────────────────────────────────────────────────
    -- One wide 440x22 slab near the top of the screen instead of the small
    -- labelled 260x34 bar: the fill IS the bar, there is no separate title or
    -- percentage row, and the whole thing is draggable. A "ragdolled" label
    -- takes over the bar and turns it red while the character is down.
    local stealBarSg, stealBarFill, stealBarTitle, stealBarPct, stealBarLbl
    local stealBarGrad, stealBarRagLbl
    local function ensureStealBar()
        if stealBarSg and stealBarSg.Parent then return end
        local host = (gethui and gethui()) or PG
        stealBarSg = mk("ScreenGui", host, {
            Name = "hudProgressLayer", ResetOnSpawn = false,
            IgnoreGuiInset = true, DisplayOrder = 120,
        })
        local wrap = mk("Frame", stealBarSg, {
            Name = "Wrap", AnchorPoint = Vector2.new(0, 0),
            Position = UDim2.new(
                0.5, tonumber(_G._ryPriv_sbX) or -220,
                0,   tonumber(_G._ryPriv_sbY) or 68
            ),
            Size = UDim2.fromOffset(440, 22),
            BackgroundTransparency = 1, Active = true,
        })
        -- Kept nil on purpose: the callers below write to them only through
        -- guarded helpers, and rymogs' bar carries no text of its own.
        stealBarTitle = nil
        stealBarPct   = nil
        -- The track is the visible slab and fills the wrap exactly. Drag is
        -- bound to it, not to the transparent wrap, because the track sits on
        -- top and would otherwise swallow the press.
        local track = corner(mk("TextButton", wrap, {
            Position = UDim2.new(0, 0, 0, 0), Size = UDim2.new(1, 0, 1, 0),
            BackgroundColor3 = UIC.track, BackgroundTransparency = 0.3 + UIAlpha() * 0.5,
            BorderSizePixel = 0, Text = "", AutoButtonColor = false, Active = true,
        }), 5)
        stroke(track, UIC.line)
        -- keepScale: the bar is centred on 0.5 scale and only its offset is
        -- saved, so the scale components have to survive a drag and a restore
        _G.RyPrivPanelDrag(wrap, track, "_ryPriv_sbX", "_ryPriv_sbY", { keepScale = true })
        _G.RyPrivApplyPos(wrap, "_ryPriv_sbX", "_ryPriv_sbY", nil, { keepScale = true })
        _G.RyPrivRegisterPanel(wrap, "_ryPriv_sbX", "_ryPriv_sbY", { keepScale = true })
        stealBarFill = corner(mk("Frame", track, {
            Size = UDim2.new(0, 0, 1, 0), BackgroundColor3 = UIC.acc, BorderSizePixel = 0,
        }), 5)
        stealBarGrad = mk("UIGradient", stealBarFill, {
            Color = ColorSequence.new(UIC.acc2, UIC.acc),
        })
        -- Sits over the fill and reads "INSTANT 47%" while instant steal is
        -- holding. Hidden the rest of the time, so the bar is unchanged for
        -- ordinary steals.
        stealBarLbl = mk("TextLabel", track, {
            Name = "ModeLabel", Size = UDim2.new(1, -20, 1, 0),
            Position = UDim2.fromOffset(12, 0), BackgroundTransparency = 1,
            Text = "", Font = UIFB, TextSize = 11,
            TextColor3 = Color3.fromRGB(255, 255, 255),
            TextXAlignment = Enum.TextXAlignment.Left,
            TextStrokeTransparency = 0.4,
            TextStrokeColor3 = Color3.fromRGB(0, 0, 0),
            ZIndex = 6, Visible = false,
        })
        stealBarRagLbl = mk("TextLabel", track, {
            Name = "RagLabel", Size = UDim2.new(1, 0, 1, 0),
            BackgroundTransparency = 1, Text = "ragdolled", Font = UIFB, TextSize = 12,
            TextColor3 = Color3.fromRGB(255, 255, 255),
            TextXAlignment = Enum.TextXAlignment.Center,
            TextStrokeTransparency = 0.4, TextStrokeColor3 = Color3.new(0, 0, 0),
            ZIndex = 5, Visible = false,
        })
        stealBarSg.Enabled = not (_G.RyPrivUIHidden and _G.RyPrivUIHidden.stealbar == true)
    end

    local _sbRag = nil
    local function setStealBarRagdolled(on)
        on = on and true or false
        if _sbRag == on then return end
        _sbRag = on
        pcall(function()
            if not stealBarFill then return end
            if on then
                if stealBarGrad then stealBarGrad.Enabled = false end
                stealBarFill.BackgroundColor3 = Color3.fromRGB(200, 40, 45)
            else
                if stealBarGrad then stealBarGrad.Enabled = true end
                stealBarFill.BackgroundColor3 = UIC.acc
            end
            if stealBarRagLbl then stealBarRagLbl.Visible = on end
        end)
    end
    local function showStealBar(_name, pct)
        pcall(function()
            ensureStealBar()
            pct = math.clamp(tonumber(pct) or 0, 0, 1)
            stealBarSg.Enabled = not (_G.RyPrivUIHidden and _G.RyPrivUIHidden.stealbar == true)
            stealBarFill.Size = UDim2.new(pct, 0, 1, 0)
        end)
    end
    local function setStealBarPct(pct)
        if not stealBarSg or not stealBarSg.Enabled then return end
        pcall(function()
            pct = math.clamp(tonumber(pct) or 0, 0, 1)
            stealBarFill.Size = UDim2.new(pct, 0, 1, 0)
        end)
    end
    local function hideStealBar()
        pcall(function()
            ensureStealBar()
            stealBarSg.Enabled = not (_G.RyPrivUIHidden and _G.RyPrivUIHidden.stealbar == true)
            stealBarFill.Size = UDim2.new(0, 0, 1, 0)
            setStealBarRagdolled(false)
        end)
    end
    task.defer(hideStealBar)

    -- Reads the fill width rather than running its own timer, so the number on
    -- the bar is the hold that is actually in progress.
    task.spawn(function()
        while true do
            task.wait(0.05)
            pcall(function()
                if not (stealBarLbl and stealBarFill) then return end
                local on = _G.RyPrivInstantSteal == true
                    and stealBarSg and stealBarSg.Enabled
                if on then
                    -- pinned full: there is no hold to track in instant mode,
                    -- so a progress number would be reporting nothing
                    stealBarFill.Size = UDim2.new(1, 0, 1, 0)
                    stealBarLbl.Text = "INSTANT  100%"
                    stealBarLbl.Visible = true
                elseif stealBarLbl.Visible then
                    stealBarLbl.Visible = false
                end
            end)
        end
    end)

    -- Independent of the steal loop, so the bar reads "ragdolled" whether or not
    -- auto steal happens to be running.
    task.spawn(function()
        while true do
            task.wait(0.15)
            local down = false
            pcall(function()
                local e = tonumber(LP:GetAttribute("RagdollEndTime"))
                down = e ~= nil and (e - workspace:GetServerTimeNow()) > 0
            end)
            setStealBarRagdolled(down)
        end
    end)

    local function buildStealCallbacks(prompt)
        if InternalStealCache[prompt] then return end
        if not prompt or not prompt.Parent then return end
        local data = { holdCallbacks = {}, triggerCallbacks = {}, holdEndCallbacks = {}, ready = true }
        local function grab(sig, into)
            local ok, conns = pcall(getconnections, sig)
            if ok and type(conns) == "table" then
                for _, c in ipairs(conns) do
                    if type(c.Function) == "function" then table.insert(into, c.Function) end
                end
            end
        end
        grab(prompt.PromptButtonHoldBegan, data.holdCallbacks)
        grab(prompt.Triggered, data.triggerCallbacks)
        grab(prompt.PromptButtonHoldEnded, data.holdEndCallbacks)
        if #data.holdCallbacks > 0 or #data.triggerCallbacks > 0 or #data.holdEndCallbacks > 0 then
            InternalStealCache[prompt] = data
        end
    end

    local _rawFireServer = Instance.new("RemoteEvent").FireServer
    local _reBegin, _reCommit = nil, nil
    local _reState, _reRetryAt = {}, {}
    local function _re(cache, hash)
        if cache and cache.Parent then return cache end
        local st = _reState[hash]
        if typeof(st) == "Instance" then
            if st.Parent then return st end
            _reState[hash] = nil
            st = nil
        end
        if st == "pending" then return nil end
        if st == "failed" and os.clock() < (_reRetryAt[hash] or 0) then return nil end
        _reState[hash] = "pending"
        task.spawn(function()
            local got
            pcall(function()
                if _G.RyPrivGetRemote then got = _G.RyPrivGetRemote("RemoteEvent", hash) end
            end)
            if typeof(got) == "Instance" then
                _reState[hash] = got
                _G.RyPrivStealSay("remote resolved " .. tostring(hash):sub(1, 8))
            else
                _reState[hash] = "failed"
                _reRetryAt[hash] = os.clock() + (tonumber(_G.RyPrivRemoteRetry) or 5)
                _G.RyPrivStealSay("remote resolve FAILED " .. tostring(hash):sub(1, 8))
            end
        end)
        task.delay(tonumber(_G.RyPrivRemoteResolveWait) or 3, function()
            if _reState[hash] == "pending" then
                _reState[hash] = "failed"
                _reRetryAt[hash] = os.clock() + (tonumber(_G.RyPrivRemoteRetry) or 5)
                _G.RyPrivStealSay("remote resolve TIMEOUT " .. tostring(hash):sub(1, 8))
            end
        end)
        return nil
    end

    local function _stealBegin()
        _reBegin = _re(_reBegin, "f40f7d9e-2f0d-4167-b250-899273f46874")
        if not _reBegin then return false end
        local t = workspace:GetServerTimeNow() + 124
        pcall(_rawFireServer, _reBegin, t, "68c86eb7-eb7e-4b4d-96ae-cf7cd847c5b0")
        pcall(function() _rawFireServer(_reBegin, t, "07b9cc25-2a1f-4a26-a0ec-f2fab578d8bd") end)
        return true
    end

    local function _stealCommit(plot, slot)
        _reCommit = _re(_reCommit, "3ba148c9-7ed6-4675-93f8-9f7c356a2c54")
        if not _reCommit then return false end
        if type(plot) ~= "string" or slot == nil then return false end
        local sl = tonumber(slot) or slot
        local t = workspace:GetServerTimeNow() + 31
        pcall(_rawFireServer, _reCommit, t, "cda5c764-d4e3-45c4-94e4-53a538347590", plot, sl)
        pcall(function()
            _rawFireServer(_reCommit, t, "8c852fbf-d542-4ef4-aa28-612e24db8d4a", plot, sl)
        end)
        return true
    end
    _G.RyPrivStealBegin, _G.RyPrivStealCommit = _stealBegin, _stealCommit

    -- fire the same two remotes (4) does; have them resolved before the first grab
    local _RE_BEGIN  = "f40f7d9e-2f0d-4167-b250-899273f46874"
    local _RE_COMMIT = "3ba148c9-7ed6-4675-93f8-9f7c356a2c54"
    local function _remotesReady()
        return typeof(_reState[_RE_BEGIN]) == "Instance"
            and typeof(_reState[_RE_COMMIT]) == "Instance"
    end
    task.spawn(function()
        for _ = 1, 60 do
            if _remotesReady() then _G.RyPrivStealSay("steal remotes ready") return end
            _re(nil, _RE_BEGIN)
            _re(nil, _RE_COMMIT)
            task.wait(0.25)
        end
    end)

    local function remoteStealAsync(pet, judge)
        if not (pet and type(pet.plot) == "string" and pet.slot ~= nil) then return false end
        if _holdBusy() then
            return false
        end
        if not _stealBegin() then return false end
        _stealHoldStart = tick()
        _stealHoldActive = true
        showStealBar(pet.name, 0)
        task.spawn(function()
            local hold = tonumber(_G.RyPrivStealHoldDuration) or STEAL_HOLD_DURATION
            while true do
                local el = tick() - _stealHoldStart
                if el >= hold then break end
                setStealBarPct(el / hold)
                RunService.Heartbeat:Wait()
            end
            setStealBarPct(1)
            _stealCommit(pet.plot, pet.slot)
            _verifyFire(pet, "remote", judge)
            _stealHoldActive = false
            task.wait(0.2)
            hideStealBar()
        end)
        task.delay(STEAL_HOLD_DURATION + 0.6, hideStealBar)
        return true
    end

    -- key identifies the pet this hold was started for. The engine this
    -- follows checks the target is still the same one before it fires the
    -- trigger, and abandons the hold if it is not -- otherwise a repick
    -- mid-hold completes the steal on whatever the prompt now belongs to.
    local function executeStealAsync(prompt, petName, restoreMax, key)
        if _holdBusy() then
            return false
        end
        local data = InternalStealCache[prompt]
        if not data or not data.ready then return false end
        data.ready = false
        _stealHoldStart = tick()
        _stealHoldActive = true
        pcall(function() prompt.MaxActivationDistance = math.huge end)
        showStealBar(petName, 0)
        task.spawn(function()
            for _, fn in ipairs(data.holdCallbacks) do task.spawn(fn) end
            local hold = tonumber(_G.RyPrivStealHoldDuration) or STEAL_HOLD_DURATION
            -- Normally the hold is stretched to at least the prompt's own
            -- HoldDuration. Instant steal wants a flat wait instead: the engine
            -- it follows holds for a fixed 1.2s and triggers, regardless of
            -- what the prompt declares.
            if _G.RyPrivFlatHold ~= true then
                pcall(function()
                    local hd = prompt.HoldDuration
                    if type(hd) == "number" and hd > 0 then
                        hold = math.max(hold, hd + 0.05)
                    end
                end)
            end
            while true do
                local el = tick() - _stealHoldStart
                if el >= hold then break end
                setStealBarPct(el / hold)
                RunService.Heartbeat:Wait()
            end
            setStealBarPct(1)
            local moved = false
            if key then
                local t = _stealTarget
                local nowKey = t and (tostring(t.plot) .. "/" .. tostring(t.slot)) or nil
                moved = (nowKey ~= nil and nowKey ~= key)
            end
            if prompt and prompt.Parent and not moved then
                for _, fn in ipairs(data.triggerCallbacks) do task.spawn(fn) end
            elseif moved then
                _G.RyPrivStealSay("hold abandoned: target changed")
            end
            for _, fn in ipairs(data.holdEndCallbacks) do task.spawn(fn) end
            _stealHoldActive = false
            if restoreMax ~= nil then
                pcall(function()
                    if prompt and prompt.Parent then prompt.MaxActivationDistance = restoreMax end
                end)
            end
            task.wait(0.05)
            data.ready = true
            task.wait(0.2)
            hideStealBar()
        end)
        task.delay(STEAL_HOLD_DURATION + 0.6, hideStealBar)
        return true
    end

    local function timeUntilCanSteal()
        if LP:GetAttribute("Stealing") or LP:GetAttribute("IsTrading")
            or LP:GetAttribute("IsDuelSelecting") or LP:GetAttribute("Web") then
            return -1
        end
        local ragdoll = LP:GetAttribute("RagdollEndTime")
        if ragdoll then
            local r = ragdoll - workspace:GetServerTimeNow()
            if r > 0 then return r end
        end
        return 0
    end


    local stealOn = (_G.RyPrivStealMode ~= nil)
    local _emptyScans = 0
    local function _fireSteal(pet)
        if not pet then return false end
        if _holdBusy() then _G.RyPrivStealSay("hold already running") return false end
        local _fhrp = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
        local _fd = (_fhrp and pet.position) and (pet.position - _fhrp.Position).Magnitude or nil
        local _judge = (_fd ~= nil) and (_fd <= _prox() + 2) or false
        local _cd = _holdDur() + (tonumber(_G.RyPrivVerifyDelay) or 0.5) + 0.15
        _G.RyPrivStealSay("gates ok -> firing on " .. tostring(pet.name))
        if _G.RyPrivRemoteStealOn ~= false then
            if remoteStealAsync(pet, _judge) then
                _G.RyPrivLastFire = os.clock()
                _cooldownUntil = tick() + _cd
                _G.RyPrivStealSay("FIRE remote -> " .. tostring(pet.name))
                return true
            end
            _G.RyPrivStealSay("remote not ready -> prompt path")
        end
        local prompt = findStealPrompt(pet)
        if not prompt or not prompt.Parent then
            _G.RyPrivStealSay("no prompt at " .. tostring(pet.plot) .. "/" .. tostring(pet.slot))
            return false
        end
        local oldMax
        pcall(function() oldMax = prompt.MaxActivationDistance end)
        pcall(function() prompt.MaxActivationDistance = math.huge end)
        buildStealCallbacks(prompt)
        if InternalStealCache[prompt] then
            if executeStealAsync(prompt, pet.name, oldMax,
                    tostring(pet.plot) .. "/" .. tostring(pet.slot)) then
                _G.RyPrivLastFire = os.clock()
                _cooldownUntil = tick() + _cd
                _verifyFire(pet, "prompt", _judge)
                _G.RyPrivStealSay("FIRE prompt-callback -> " .. tostring(pet.name))
                return true
            end
            _G.RyPrivStealSay("prompt-callback not ready")
            if oldMax ~= nil then pcall(function() prompt.MaxActivationDistance = oldMax end) end
            return false
        elseif fireproximityprompt then
            pcall(function() prompt.MaxActivationDistance = math.huge end)
            showStealBar(pet.name, 1)
            pcall(function() fireproximityprompt(prompt) end)
            _G.RyPrivLastFire = os.clock()
            _cooldownUntil = tick() + (tonumber(_G.RyPrivVerifyDelay) or 0.5) + 0.5
            _verifyFire(pet, "prompt", _judge)
            _G.RyPrivStealSay("FIRE proximityprompt -> " .. tostring(pet.name))
            task.delay(0.4, hideStealBar)
            pcall(function() if oldMax ~= nil then prompt.MaxActivationDistance = oldMax end end)
            return true
        end
        _G.RyPrivStealSay("no steal method available")
        return false
    end
    _G.RyPrivFireSteal = _fireSteal
    -- Tracks the last reported on/off so the idle branch can log the change
    -- rather than the state, see below.
    local _stealWasOn = nil
    local function _stealTick()
        stealOn = (_G.RyPrivStealMode ~= nil)
        if not stealOn then
            -- Say it once on the transition, not once per frame. This is the
            -- branch that runs every frame whenever auto steal is simply off,
            -- and RyPrivStealSay is not free: a tostring, a global write, an
            -- os.clock and a walk of the log table, 60 times a second to
            -- report a state that has not changed. The counter moved below
            -- the early-out for the same reason.
            if _stealWasOn ~= false then
                _stealWasOn = false
                _G.RyPrivStealSay("steal mode OFF")
            end
            return
        end
        _stealWasOn = true
        _G.RyPrivTickN = (_G.RyPrivTickN or 0) + 1
        local now = os.clock()
        local _lockUid = _G.RyPrivStealTargetUID
        local _lockChanged = (type(_lockUid) == "string" and _lockUid ~= "" and _lockUid ~= _lastPickUid)
        local _needPick = (not _stealTarget) or _lockChanged or (now - _lastTargetPick) >= (tonumber(_G.RyPrivRepickGap) or 0.35)
        if _armedPet then
            local _lp = _armLivePos(_armedPet)
            if _lp then
                _armedPet.position = _lp
                _armLostSince = 0
            elseif _armLostSince == 0 then
                _armLostSince = now
            elseif now - _armLostSince > (tonumber(_G.RyPrivArmLostGrace) or 1.5) then
                _armedPet, _armLostSince = nil, 0
            end
            if _armedPet and (now - _armCheckAt) >= (tonumber(_G.RyPrivArmCheckGap) or 0.5) then
                _armCheckAt = now
                local okL, listed = pcall(_petListed, _armedPet.plot, _armedPet.slot)
                if okL and not listed then
                    _G.RyPrivStealSay("armed pet gone -> repicking")
                    _armedPet, _armLostSince = nil, 0
                end
            end
            if _armedPet then
                _stealTarget = _armedPet
                if type(_armedPet.name) == "string" then _currentTargetName = _armedPet.name end
                _needPick = false
            end
        end
        local _scanGap = (_emptyScans >= 3) and (tonumber(_G.RyPrivStealIdleGap) or 0.3) or 0.01
        if not LP:GetAttribute("Stealing") and _needPick and (now - _autoLastScan) >= _scanGap then
            _autoLastScan = now
            _lastTargetPick = now
            local char = LP.Character
            local hrp = char and char:FindFirstChild("HumanoidRootPart")
            if hrp then
                local pets = _scanCached(now, 0.05)
                local ok = pets ~= nil
                if ok and pets and #pets > 0 then _emptyScans = 0 else _emptyScans = _emptyScans + 1 end
                if ok and pets and #pets > 0 then
                    local best
                    if type(_lockUid) == "string" and _lockUid ~= "" then
                        for _, p in ipairs(pets) do
                            if not p.conveyor and _petUid(p) == _lockUid then best = p break end
                        end
                    end
                    if not best and _G.RyPrivStealMode == "nearest" then
                        local myPos = hrp.Position
                        local bestD = math.huge
                        for _, p in ipairs(pets) do
                            if not p.conveyor and p.position then
                                local d = (p.position - myPos).Magnitude
                                if d < bestD then bestD = d; best = p end
                            end
                        end
                    elseif not best then
                        for _, p in ipairs(pets) do if not p.conveyor then best = p; break end end
                    end
                    best = best or pets[1]
                    if best then
                        _stealTarget = best
                        _stealArmedAt = now
                        _lastPickUid = _lockUid
                        if type(best.name) == "string" and best.name ~= "" then
                            _currentTargetName = best.name
                        end
                    end
                end
            end
        end
        if _G.RyPrivStealInRangeFirst ~= false
            and not (_armedPet and _G.RyPrivTPTargetWins ~= false)
            and not (type(_lockUid) == "string" and _lockUid ~= "") then
            local _c2 = LP.Character
            local _h2 = _c2 and _c2:FindFirstChild("HumanoidRootPart")
            if _h2 then
                local _cur = _stealTarget
                local _curD = (_cur and _cur.position) and (_cur.position - _h2.Position).Magnitude or math.huge
                if _curD > _prox() and (now - _inRangeLast) >= (tonumber(_G.RyPrivInRangeGap) or 0.25) then
                    _inRangeLast = now
                    local _pets2 = _scanCached(now, 0.15)
                    local _ok2 = _pets2 ~= nil
                    if _ok2 and type(_pets2) == "table" then
                        local _near, _nd = nil, math.huge
                        local _mp = _h2.Position
                        for _, p in ipairs(_pets2) do
                            if not p.conveyor and p.position then
                                local d = (p.position - _mp).Magnitude
                                if d < _nd then _nd, _near = d, p end
                            end
                        end
                        if _near and _nd <= _prox() then
                            _stealTarget = _near
                            _armedPet, _armLostSince = nil, 0
                            _lastTargetPick = now
                            if type(_near.name) == "string" then _currentTargetName = _near.name end
                            _G.RyPrivStealSay(string.format("switch to in-range %s d=%.0f", tostring(_near.name), _nd))
                        end
                    end
                end
            end
        end
        local pet = _stealTarget
        if not pet then _G.RyPrivStealSay("no target") return end

        if now - _stealLastScan < 0.033 then _G.RyPrivThrN = (_G.RyPrivThrN or 0) + 1 return end
        _stealLastScan = now
        local t = timeUntilCanSteal()
        if t == -1 then
            if LP:GetAttribute("Stealing") then _stealTarget = nil; _armedPet, _armLostSince = nil, 0 end
            _G.RyPrivStealSay("busy: stealing/trading/web")
            return
        end
        if t > 0 and t > _holdDur() then _G.RyPrivStealSay(string.format("cooldown %.1fs", t)) return end
        local char = LP.Character
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        if not hrp then _G.RyPrivStealSay("no character") return end

        if pet.position then
            local toPet = pet.position - hrp.Position
            local dist = toPet.Magnitude
            _G.RyPrivStealTgt = string.format("%s [%s/%s] d=%.0f", tostring(pet.name), tostring(pet.plot), tostring(pet.slot), dist)
            local hold = tonumber(_G.RyPrivStealHoldDuration) or STEAL_HOLD_DURATION

            local lead = tonumber(_G.RyPrivStealLead) or 0.27

            local closing = dist > 0.001 and (hrp.AssemblyLinearVelocity:Dot(toPet) / dist) or 0

            local _nearBase = false
            do
                local _bi = getClosestBaseIdx(pet.position)
                local _b = _bi and BASES_LOW[_bi]
                if _b then
                    local _bx, _bz = hrp.Position.X - _b.X, hrp.Position.Z - _b.Z
                    if math.sqrt(_bx * _bx + _bz * _bz) <= (tonumber(_G.RyPrivCloseBaseRange) or 100) then
                        _nearBase = true
                    end
                end
            end

            if _nearBase then
                if dist > (tonumber(_G.RyPrivNearBaseStealStart) or 55) then _G.RyPrivStealSay(string.format("nearbase far d=%.0f", dist)) return end
            elseif closing > (tonumber(_G.RyPrivFastApproach) or 150) then

                if dist > (tonumber(_G.RyPrivStealLeadMax) or 223) then _G.RyPrivStealSay(string.format("lead far d=%.0f", dist)) return end
                if (dist / closing) > hold * lead then _G.RyPrivStealSay(string.format("lead wait d=%.0f c=%.0f", dist, closing)) return end
            elseif closing > 5 then

                if dist > (tonumber(_G.RyPrivCloseProximity) or 24) then _G.RyPrivStealSay(string.format("moving far d=%.0f c=%.0f", dist, closing)) return end
            else
                
                if dist > _prox() then _G.RyPrivStealSay(string.format("far d=%.0f", dist)) return end
            end
        end
        -- INSTANT STEAL OWNS THE FIRING.
        --
        -- In instant mode the engine this follows cancels its hold, pins the
        -- bar full and returns before it ever reaches the held steal -- the
        -- prompt burst is the only thing that takes anything. Letting this run
        -- as well is what made it behave like ordinary auto steal with an
        -- INSTANT label on the bar: the target path was still picking one
        -- brainrot, walking to it and holding, which is the thing instant mode
        -- exists to skip.
        if _G.RyPrivInstantSteal == true then return end
        _fireSteal(pet)
    end
    RunService.Heartbeat:Connect(function()
        local _ok, _err = pcall(_stealTick)
        if not _ok then _G.RyPrivStealSay("ERROR " .. tostring(_err)) end
    end)

    -- ══════════════════════════════════════════════════════════════════════
    -- INSTANT STEAL
    -- ══════════════════════════════════════════════════════════════════════
    -- Keeps a registry of every ProximityPrompt under Plots/*/AnimalPodiums and
    -- fires the ones that pass the gate below. A prompt is tried the moment it
    -- is discovered and again the moment its Enabled flips true, so a brainrot
    -- that lands while you are stood in a base is taken on that frame.
    --
    -- THE GATE IS THE WHOLE FEATURE, and getting it wrong is what made every
    -- previous attempt behave like ordinary priority steal:
    --
    --   * normally a prompt only fires if it belongs to the ONE pet the hub has
    --     selected. That restriction is what makes it "priority steal".
    --   * in instant mode that restriction is dropped entirely and replaced
    --     with "is this prompt in the same plot I am standing in". Everything
    --     in the base becomes fair game the instant it is stealable.
    --
    -- Instant mode is only live when RyPrivInstantSteal AND nearest mode are
    -- both on -- that is the condition the gate reads. Turning instant steal on
    -- therefore switches the hub to nearest, because on priority the gate never
    -- engages and you get the normal selected-pet behaviour instead.
    do
        -- Plot strips, as measured for this map. Deriving these from model
        -- bounding boxes was tried and is not reliable: a plot's decorations
        -- overhang into its neighbour, so the boxes overlap and the "which plot
        -- am I in" answer comes out arbitrary.
        local PLOTS = {
            { min = Vector3.new(-337.448303, -3.898971, -122.397758), max = Vector3.new(-328.004578, -3.898971, 242.625626) },
            { min = Vector3.new(-327.257660, -3.899109, -122.228622), max = Vector3.new(-320.600891, -3.899109, 242.612259) },
            { min = Vector3.new(-319.783386, -3.898970, -122.227089), max = Vector3.new(-312.908325, -3.898970, 242.585617) },
            { min = Vector3.new(-312.445648, -3.899108, -122.389832), max = Vector3.new(-305.489899, -3.899108, 242.456818) },
            { min = Vector3.new(-305.037048, -3.898970, -122.230743), max = Vector3.new(-293.957489, -3.898970, 242.606873) },
            { min = Vector3.new(-491.448608, -3.898972, -122.253258), max = Vector3.new(-481.811737, -3.898972, 242.615005) },
            { min = Vector3.new(-498.971069, -3.898970, -122.382767), max = Vector3.new(-491.748840, -3.898970, 242.612061) },
            { min = Vector3.new(-506.436737, -3.898972, -122.411476), max = Vector3.new(-499.318542, -3.898972, 242.615982) },
            { min = Vector3.new(-513.783569, -3.898972, -122.223297), max = Vector3.new(-506.801849, -3.898972, 242.627090) },
            { min = Vector3.new(-525.236938, -3.898972, -122.409813), max = Vector3.new(-514.265015, -3.898972, 242.608932) },
        }
        local RADIUS = 12

        local REG, NEXT_OK, DISCOVERED = {}, {}, {}
        local ARMED, ARMING = false, false
        local SNAP = {}
        local _daConn = nil
        local _ownPlotName, _ownPlotAt = nil, -1

        -- which strip contains this XZ, or nil for the walkways between them
        local function plotIndex(pos)
            for i = 1, #PLOTS do
                local b = PLOTS[i]
                if pos.X >= math.min(b.min.X, b.max.X) and pos.X <= math.max(b.min.X, b.max.X)
                    and pos.Z >= math.min(b.min.Z, b.max.Z) and pos.Z <= math.max(b.min.Z, b.max.Z) then
                    return i
                end
            end
            return nil
        end

        local function promptPos(p)
            local parent = p.Parent
            if not parent then return nil end
            if parent:IsA("Attachment") and parent.Parent then parent = parent.Parent end
            if parent:IsA("BasePart") then return parent.Position end
            if parent:IsA("Model") then
                local ok, cf = pcall(parent.GetPivot, parent)
                if ok and cf then return cf.Position end
            end
            return nil
        end

        -- never fire on our own base
        local function ownPlot()
            local now = os.clock()
            if now - _ownPlotAt < 5 then return _ownPlotName end
            _ownPlotAt, _ownPlotName = now, nil
            local plots = workspace:FindFirstChild("Plots")
            if not plots then return nil end
            local me = LP.Name:lower()
            local dn = (LP.DisplayName or ""):lower()
            for _, plot in ipairs(plots:GetChildren()) do
                local sign = plot:FindFirstChild("PlotSign")
                local gui  = sign and sign:FindFirstChildWhichIsA("SurfaceGui", true)
                local lbl  = gui and gui:FindFirstChildWhichIsA("TextLabel", true)
                local txt  = lbl and type(lbl.Text) == "string" and lbl.Text:lower() or nil
                if txt and txt ~= "" and (txt:find(me, 1, true) or (dn ~= "" and txt:find(dn, 1, true))) then
                    _ownPlotName = plot.Name
                    break
                end
            end
            return _ownPlotName
        end

        -- Which plot are we standing in, asked of the game rather than of a
        -- coordinate table: the plot with the nearest podium, provided it is
        -- close enough to actually be that base.
        local _nearPlot, _nearAt = nil, -1
        local function plotHere(myPos)
            local now = os.clock()
            if now - _nearAt < 0.5 then return _nearPlot end
            _nearAt, _nearPlot = now, nil
            local plots = workspace:FindFirstChild("Plots")
            if not plots then return nil end
            local bestD = tonumber(_G.RyPrivInstantPlotRange) or 60
            for _, plot in ipairs(plots:GetChildren()) do
                local pods = plot:FindFirstChild("AnimalPodiums")
                if pods then
                    for _, slot in ipairs(pods:GetChildren()) do
                        local base = slot:FindFirstChild("Base")
                        local spawn = base and base:FindFirstChild("Spawn")
                        if spawn and spawn:IsA("BasePart") then
                            local d = (spawn.Position - myPos).Magnitude
                            if d < bestD then bestD, _nearPlot = d, plot.Name end
                        end
                    end
                end
            end
            return _nearPlot
        end

        local function plotNameOf(p)
            local pods = p:FindFirstAncestor("AnimalPodiums")
            local plot = pods and pods.Parent
            local plots = workspace:FindFirstChild("Plots")
            if plot and plots and plot.Parent == plots then return plot.Name end
            return nil
        end

        -- The gate. myPos is the character's position.
        local function passes(p, myPos)
            if not (p and p.Parent and p.Enabled) then return false end
            local pos = promptPos(p)
            if not pos then return false end

            local pn = plotNameOf(p)
            if pn and pn == ownPlot() then return false end

            -- Instant mode drops the selected-pet restriction entirely and
            -- replaces it with "is this in the plot I am standing in". That
            -- swap is the whole feature: without it every prompt is rejected
            -- and the ordinary target path is left doing all the work, which
            -- reads as normal auto steal with an INSTANT label on the bar.
            --
            -- The strip table answers first because it is cheap. When it
            -- places neither of us -- a different map revision, a podium
            -- sitting outside its rectangle -- fall back to asking the game.
            local mine, theirs = plotIndex(myPos), plotIndex(pos)
            if mine and theirs then
                if mine ~= theirs then return false end
            else
                local here = plotHere(myPos)
                if not here then return false end
                if pn and pn ~= here then return false end
                if not pn then return false end
            end

            local mad = p.MaxActivationDistance
            local base = tonumber(_G.RyPrivInstantRadius) or RADIUS
            local r = (type(mad) == "number" and mad > 0) and math.min(base, mad) or base
            return (pos - myPos).Magnitude <= r
        end

        -- arg2 = how many times, arg3 = per-prompt cooldown before the next one
        local function fire(p, times, gap)
            if not (p and p.Parent and p.Enabled) then return end
            local now = os.clock()
            if (NEXT_OK[p] or 0) > now then return end
            NEXT_OK[p] = now + (gap or 0)
            for _ = 1, times do
                local ok = pcall(fireproximityprompt, p, 0)
                if not ok then pcall(fireproximityprompt, p) end
                if not (p.Parent and p.Enabled) then break end
            end
            _G.RyPrivLastFire = now
        end

        local function live()
            return _G.RyPrivInstantSteal == true
        end

        local function tryNow(p, times, gap)
            if not (ARMED and live()) then return end
            local c = LP.Character
            local hrp = c and c:FindFirstChild("HumanoidRootPart")
            if not hrp then return end
            if passes(p, hrp.Position) then fire(p, times, gap) end
        end

        local function register(p)
            if REG[p] then return end
            REG[p] = true
            pcall(function()
                p:GetPropertyChangedSignal("Enabled"):Connect(function()
                    -- the edge: 25 shots the moment it becomes stealable
                    if p.Enabled then tryNow(p, 25, 0.08) end
                end)
            end)
            pcall(function()
                p.AncestryChanged:Connect(function()
                    if not p:IsDescendantOf(workspace) then
                        REG[p], NEXT_OK[p], DISCOVERED[p] = nil, nil, nil
                    end
                end)
            end)
            if not DISCOVERED[p] then
                DISCOVERED[p] = true
                task.defer(tryNow, p, 25, 0.08)
            end
        end

        local function scan()
            local plots = workspace:FindFirstChild("Plots")
            if not plots then return end
            for _, plot in ipairs(plots:GetChildren()) do
                local pods = plot:FindFirstChild("AnimalPodiums")
                if pods then
                    for _, d in ipairs(pods:GetDescendants()) do
                        if d:IsA("ProximityPrompt") then register(d) end
                    end
                end
            end
        end

        -- workspace.DescendantAdded fires for everything the game streams in, so
        -- it is only connected while instant steal is actually on
        local function hookAdded()
            if _daConn then return end
            _daConn = workspace.DescendantAdded:Connect(function(d)
                if d:IsA("ProximityPrompt") and d:FindFirstAncestor("AnimalPodiums") then
                    register(d)
                end
            end)
        end
        local function unhookAdded()
            if _daConn then pcall(function() _daConn:Disconnect() end) _daConn = nil end
        end

        -- Waits for the game to finish loading before it is allowed to fire.
        -- Bursting into a half-streamed Plots tree aims at podiums that have not
        -- settled yet.
        local function arm()
            if ARMING then return end
            ARMING = true
            task.spawn(function()
                if not game:IsLoaded() then game.Loaded:Wait() end
                task.wait(tonumber(_G.RyPrivInstantArmDelay) or 0.5)
                ARMING = false
                if live() then ARMED = true scan() end
            end)
        end

        -- the sustain pass: 3 shots per prompt, 0.12s apart, ten times a second
        task.spawn(function()
            local rescanAt = 0
            while true do
                if live() then
                    task.wait(tonumber(_G.RyPrivInstantPollGap) or 0.1)
                else
                    task.wait(0.35)
                end
                if live() and ARMED then
                    local c = LP.Character
                    local hrp = c and c:FindFirstChild("HumanoidRootPart")
                    if hrp then
                        local now = os.clock()
                        if now > rescanAt then
                            rescanAt = now + 2
                            scan()
                        end
                        local myPos = hrp.Position
                        table.clear(SNAP)
                        for p in pairs(REG) do SNAP[#SNAP + 1] = p end
                        local hits = 0
                        for _, p in ipairs(SNAP) do
                            if passes(p, myPos) then
                                hits = hits + 1
                                fire(p, 3, 0.12)
                            end
                        end
                        -- One line a second while it is finding nothing, so
                        -- "still using normal auto steal" comes with a reason.
                        if hits == 0 and now - (_G._ryInstSaidAt or 0) > 1 then
                            _G._ryInstSaidAt = now
                            pcall(_RP, string.format(
                                "[instant] %d prompts known, 0 in range | plot=%s strip=%s r=%s",
                                #SNAP, tostring(plotHere(myPos)),
                                tostring(plotIndex(myPos)),
                                tostring(tonumber(_G.RyPrivInstantRadius) or RADIUS)))
                        end
                    end
                end
            end
        end)

        -- THE RECYCLE, AND WHY IT MATTERS MORE THAN IT LOOKS.
        --
        -- Every 1.5s the flag is genuinely dropped for 0.05s and put straight
        -- back. That does three things, and only the first is obvious:
        --
        --   1. clears the per-prompt cooldowns and re-walks the registry, so
        --      prompts the server swapped out are picked up again
        --   2. the bar visibly blinks off and back, which is the flicker you
        --      see while it is running
        --   3. and the one that actually matters: for those 0.05s the target
        --      path is no longer standing down, so the ordinary held steal gets
        --      a window to fire. That is why it holds and redoes it -- the
        --      burst runs continuously and a held steal is let through on every
        --      cycle.
        --
        -- Clearing the arm flags without touching RyPrivInstantSteal, which is
        -- what this used to do, keeps the target path suppressed the whole time
        -- and the hold never happens at all.
        task.spawn(function()
            while true do
                task.wait(tonumber(_G.RyPrivInstantCycle) or 1.5)
                if live() and _G.RyPrivInstantRecycle ~= false then
                    ARMED, ARMING = false, false
                    table.clear(NEXT_OK)
                    -- _fireSteal parks _cooldownUntil roughly a hold-plus-verify
                    -- ahead, which is longer than this cycle -- so without
                    -- clearing it the next blink is blocked by _holdBusy before
                    -- it can start anything and only every other one lands.
                    _cooldownUntil = 0
                    _G.RyPrivInstantSteal = false
                    if _G.RyPrivRepaintInstant then pcall(_G.RyPrivRepaintInstant) end
                    task.wait(tonumber(_G.RyPrivInstantBlink) or 0.05)
                    _G.RyPrivInstantSteal = true
                    if _G.RyPrivRepaintInstant then pcall(_G.RyPrivRepaintInstant) end
                    arm()
                end
            end
        end)

        LP.CharacterAdded:Connect(function()
            if live() then
                ARMED, ARMING = false, false
                table.clear(NEXT_OK)
                arm()
            end
        end)

        _G.RyPrivSetInstantSteal = function(on, quiet)
            on = (on == true)
            if (_G.RyPrivInstantSteal == true) == on then
                if _G.RyPrivRepaintInstant then pcall(_G.RyPrivRepaintInstant) end
                return on
            end
            _G.RyPrivInstantSteal = on
            if on then
                -- The gate only opens when nearest mode is on as well. On
                -- priority the selected-pet restriction stays in force and this
                -- behaves like ordinary priority steal -- which is exactly what
                -- it looked like before. Remember what it was so switching off
                -- puts it back.
                _G._ryInstPrevMode = _G.RyPrivStealMode
                if _G.RyPrivStealMode ~= "nearest" then
                    if _G.RyPrivSetStealMode then
                        pcall(_G.RyPrivSetStealMode, "nearest")
                    else
                        _G.RyPrivStealMode = "nearest"
                    end
                end

                -- The steal that fires in the recycle's blink window is the
                -- held one: fire the prompt's PromptButtonHoldBegan callbacks,
                -- wait a flat 1.2s, fire its Triggered callbacks. _fireSteal
                -- would otherwise reach for the steal remotes first, which is a
                -- different mechanism with different timing and no hold to see.
                -- Both settings are put back when instant steal goes off.
                _G._ryInstPrevRemote = _G.RyPrivRemoteStealOn
                _G._ryInstPrevHold   = _G.RyPrivStealHoldDuration
                _G.RyPrivRemoteStealOn     = false
                _G.RyPrivStealHoldDuration = tonumber(_G.RyPrivInstantHoldSecs) or 1.2
                _G.RyPrivFlatHold          = true

                hookAdded()
                arm()
            else
                ARMED, ARMING = false, false
                table.clear(NEXT_OK)
                unhookAdded()
                _G.RyPrivFlatHold = nil
                if _G._ryInstPrevRemote ~= nil or _G._ryInstPrevHold ~= nil then
                    _G.RyPrivRemoteStealOn     = _G._ryInstPrevRemote
                    _G.RyPrivStealHoldDuration = _G._ryInstPrevHold
                    _G._ryInstPrevRemote, _G._ryInstPrevHold = nil, nil
                end
                local prev = _G._ryInstPrevMode
                _G._ryInstPrevMode = nil
                if prev and prev ~= "nearest" and _G.RyPrivStealMode == "nearest" then
                    if _G.RyPrivSetStealMode then
                        pcall(_G.RyPrivSetStealMode, prev)
                    else
                        _G.RyPrivStealMode = prev
                    end
                end
            end
            if _G.RyPrivRepaintInstant then pcall(_G.RyPrivRepaintInstant) end
            -- Base Camp passes quiet, so its override never overwrites the panel
            if not quiet then pcall(_G.RyPrivSaveSettings) end
            _G.RyPrivStealSay("instant steal " .. (on and "ON" or "off"))
            return on
        end
        -- One call, everything that decides whether the burst runs. After
        -- several rounds of fixing one gate at a time and still landing on the
        -- old behaviour, this is here so the next answer comes from the game
        -- rather than from me guessing which gate it is.
        _G.RyPrivInstantDiag = function()
            local c = LP.Character
            local hrp = c and c:FindFirstChild("HumanoidRootPart")
            local known, inRange, nearest = 0, 0, math.huge
            for _ in pairs(REG) do known = known + 1 end
            if hrp then
                local myPos = hrp.Position
                for p in pairs(REG) do
                    if passes(p, myPos) then
                        inRange = inRange + 1
                        local pp = promptPos(p)
                        if pp then
                            local d = (pp - myPos).Magnitude
                            if d < nearest then nearest = d end
                        end
                    end
                end
            end
            local msg = table.concat({
                "instant=" .. tostring(_G.RyPrivInstantSteal == true),
                "armed=" .. tostring(ARMED),
                "mode=" .. tostring(_G.RyPrivStealMode),
                "prompts=" .. known,
                "inRange=" .. inRange,
                "nearest=" .. (nearest == math.huge and "-" or string.format("%.0f", nearest)),
                "myPlot=" .. tostring(hrp and plotHere(hrp.Position)),
                "strip=" .. tostring(hrp and plotIndex(hrp.Position)),
                "ownPlot=" .. tostring(ownPlot()),
                "radius=" .. tostring(tonumber(_G.RyPrivInstantRadius) or RADIUS),
                "fpp=" .. tostring(fireproximityprompt ~= nil),
                "autoTP=" .. tostring(_G.RyPrivAutoTP == true),
                "remote=" .. tostring(_G.RyPrivRemoteStealOn),
                "hold=" .. tostring(_G.RyPrivStealHoldDuration),
            }, " ")
            pcall(_RP, "[instant] " .. msg)
            return msg
        end

        _G.RyPrivToggleInstantSteal = function()
            return _G.RyPrivSetInstantSteal(_G.RyPrivInstantSteal ~= true)
        end

        -- Restored on from the settings file.
        --
        -- This used to just arm the registry, which meant everything else
        -- RyPrivSetInstantSteal does was skipped on a restore: nearest mode was
        -- not forced, the remote was left in play and the hold stayed at its
        -- normal duration. The result was the ordinary remote steal running
        -- underneath on every recycle blink -- the old behaviour, with an
        -- INSTANT label on the bar and no way to tell from the outside.
        --
        -- Clearing the flag first is deliberate: the setter early-returns when
        -- the state already matches, which is exactly how this was missed.
        if _G.RyPrivInstantSteal == true then
            _G.RyPrivInstantSteal = false
            _G.RyPrivSetInstantSteal(true, true)
        end
    end

    -- ======================================================================

    task.spawn(function()
        while true do
            task.wait(tonumber(_G.RyPrivWatchdogTick) or 0.2)
            if _G.RyPrivStealMode ~= nil and _G.RyPrivStealWatchdog ~= false
                -- The watchdog fires _fireSteal on its own 0.2s timer, so it
                -- bypasses the stand-down in _stealTick entirely. Leaving it
                -- running while instant steal is on means the old target-driven
                -- steal keeps happening underneath and nothing looks like it
                -- changed. It was taken out of the stand-down on the reasoning
                -- that the burst takes no hold -- but the hold was never the
                -- point: the watchdog IS the steal instant mode replaces.
                and _G.RyPrivInstantSteal ~= true
                and not LP:GetAttribute("Stealing")
                and not _holdBusy()
                and (os.clock() - (_G.RyPrivLastFire or 0)) > (tonumber(_G.RyPrivWatchdogGap) or 1.5) then
                local c = LP.Character
                local h = c and c:FindFirstChild("HumanoidRootPart")
                if h then
                    local pets = _scanCached(os.clock(), 0.15)
                    local ok = pets ~= nil
                    if ok and type(pets) == "table" then
                        local near, nd = nil, math.huge
                        local mp = h.Position
                        for _, p in ipairs(pets) do
                            if not p.conveyor and p.position then
                                local d = (p.position - mp).Magnitude
                                if d < nd then nd, near = d, p end
                            end
                        end
                        if near and nd <= _prox() then
                            _G.RyPrivStealSay(string.format("WATCHDOG in range %s d=%.0f", tostring(near.name), nd))
                            pcall(_fireSteal, near)
                        end
                    end
                end
            end
        end
    end)

    do
        -- ── ANIMATE REPAIR ───────────────────────────────────────────────
        -- The game's own Animate script throws on startup when any of its
        -- Animation entries has an empty AnimationId:
        --
        --   LoadAnimation requires the asset id to not be empty
        --   Workspace.<name>.Animate, Line 198 - configureAnimationSet
        --
        -- Normally that never bites, because Animate only runs once, at spawn,
        -- and by then the entry has been filled in. But re-parenting a
        -- character restarts every LocalScript inside it, and this hub does
        -- exactly that -- the Invis system moves the character out to a holder
        -- model and back. On that second run Animate hits the empty entry,
        -- dies, never binds to the humanoid, and the character is left with no
        -- animations at all until it respawns.
        --
        -- That is the "stuck in the disable-animations state" everyone blames
        -- on the toggle. Filling the empty entries in makes the script
        -- survivable, so a restart is harmless.
        local function repairAnimate(char)
            local a = char and char:FindFirstChild("Animate")
            if not a then return 0 end
            local fixed = 0
            for _, set in ipairs(a:GetChildren()) do
                -- a valid id from a sibling in the same set is the safest
                -- filler: the set stays populated and the state still animates
                local donor
                for _, d in ipairs(set:GetChildren()) do
                    if d:IsA("Animation") and d.AnimationId ~= nil and d.AnimationId ~= "" then
                        donor = d.AnimationId
                        break
                    end
                end
                for _, d in ipairs(set:GetChildren()) do
                    if d:IsA("Animation") and (d.AnimationId == nil or d.AnimationId == "") then
                        if donor then
                            pcall(function() d.AnimationId = donor end)
                        else
                            -- nothing to copy: drop it so the set is simply
                            -- shorter rather than fatal
                            pcall(function() d:Destroy() end)
                        end
                        fixed = fixed + 1
                    end
                end
            end
            return fixed
        end
        _G.RyPrivRepairAnimate = repairAnimate

        -- Run it before anything can restart the script, and again on every
        -- respawn, since the fresh character carries the same empty entry.
        if LP.Character then pcall(repairAnimate, LP.Character) end
        LP.CharacterAdded:Connect(function(c)
            task.defer(function() pcall(repairAnimate, c) end)
        end)

        -- Suppression stops the tracks the Animate script starts; it does NOT
        -- disable the script.
        --
        -- Disabling it is more thorough, and it is what this did before, but in
        -- this game it is a one-way door: restarting Animate throws
        -- "LoadAnimation requires the asset id to not be empty" out of
        -- configureAnimationSet, because one of the game's own animation
        -- entries has no id. The script dies on the way back up, never rebinds,
        -- and the character stays frozen until it respawns -- which is exactly
        -- what turning the toggle off used to do.
        --
        -- Leaving the script running and killing its tracks is fully
        -- reversible: switching off simply stops killing them, and the next
        -- state change animates normally.
        local function applyUnwalkAlways(char)
            if not char then return end
            if _G.RyPrivDisableAnims == false then return end
            local hum = char:FindFirstChildOfClass("Humanoid")
            local animator = hum and hum:FindFirstChildOfClass("Animator")
            if not animator then return end
            local ok, tracks = pcall(function() return animator:GetPlayingAnimationTracks() end)
            if ok and tracks then
                for _, t in ipairs(tracks) do pcall(function() t:Stop(0) end) end
            end
        end
        local function hook(char)
            task.spawn(function()
                char:WaitForChild("Humanoid", 10); task.wait(0.05)
                for i = 1, 8 do
                    if LP.Character ~= char then break end
                    applyUnwalkAlways(char); task.wait(0.25)
                end
            end)
        end
        if LP.Character then hook(LP.Character) end
        LP.CharacterAdded:Connect(hook)
        -- Turning it back on has to undo what was done: re-enable the Animate
        -- script and kick the humanoid out of whatever state it was frozen in,
        -- or the character stays rigid until it respawns.
        _G.RyPrivSetDisableAnims = function(on)
            on = on and true or false
            _G.RyPrivDisableAnims = on
            local c = LP.Character
            if on then
                if c then pcall(applyUnwalkAlways, c) end
            else
                -- Nothing to undo any more: the tracks simply stop being
                -- stopped. The only repair needed is for a character left with
                -- Animate switched off by an older build of this hub.
                pcall(function()
                    local a = c and c:FindFirstChild("Animate")
                    if a and a.Disabled then a.Disabled = false end
                    local hum = c and c:FindFirstChildOfClass("Humanoid")
                    if hum then hum:ChangeState(Enum.HumanoidStateType.Running) end
                end)
            end
            pcall(_G.RyPrivSaveSettings)
        end

        -- 1.5s was fine while the Animate script was disabled outright, since
        -- nothing could start a track in between. Now that the script keeps
        -- running, tracks have to be caught as they start, so this runs often
        -- enough that nothing visibly plays -- and not at all while the toggle
        -- is off.
        local _unwalkLast = 0
        RunService.Heartbeat:Connect(function()
            if _G.RyPrivDisableAnims == false then return end
            local now = os.clock()
            if now - _unwalkLast < (tonumber(_G.RyPrivAnimStopGap) or 0.05) then return end
            _unwalkLast = now
            if LP.Character then applyUnwalkAlways(LP.Character) end
        end)
    end

    local HS = game:GetService("HttpService")
    local UIS = game:GetService("UserInputService")
    local CFG_FILE = "SideTP.json"

    local function loadCfgTable()
        local t = {}
        if readfile then
            pcall(function()
                local raw = readfile(CFG_FILE)
                if type(raw) == "string" and #raw > 0 then
                    local ok, d = pcall(HS.JSONDecode, HS, raw)
                    if ok and type(d) == "table" then t = d end
                end
            end)
        end
        return t
    end

    local function saveTpSettings()
        if not writefile then return end
        local t = loadCfgTable()
        t.tpVelocity = tonumber(_G.TPVelocity) or 400
        t.climbSpeed = tonumber(_G.RyPrivClimb) or 160
        t.goSpeed = tonumber(_G.RyPrivGoSpeed) or 200
        t.cframeSpeed = tonumber(_G.RyPrivCFrameSpeed) or 450
        t.walkSpeed = tonumber(_G.RyPrivWalkSpeed) or 27
        t.landingDelay = tonumber(_G.LandingDelay) or 0.35
        t.closeSpeed = tonumber(_G.RyPrivCloseSpeed) or 80
        t.tpSafeMode = _G.RyPrivTPSafeMode ~= false
        t.tpSafeSpeed = tonumber(_G.RyPrivTPSafeSpeed) or 300
        t.tpSafeVerticalSpeed = tonumber(_G.RyPrivTPSafeVerticalSpeed) or 110
        t.tpRetryCount = tonumber(_G.RyPrivTPRetryCount) or 2
        t.autoTp = _G.RyPrivAutoTP ~= false
        t.autoKickOnSteal = _G.RyPrivAutoKickOnSteal == true
        t.kickToPS = _G.RyPrivKickToPS == true
        t.psLink = tostring(_G.RyPrivPrivateServerLink or "")
        t.priAlert = _G.RyPrivPriAlert == true
        t.alertSound = tostring(_G.RyPrivAlertSound or "111786441593851")
        t.alertMinGen = tonumber(_G.RyPrivAlertMinGen) or 80e6
        t.walkSpeedOn = _G.RyPrivWalkSpeedOn ~= false
        t.xray = _G.RyPrivXray ~= false
        t.espPlayer    = _G.RyPlayerESP   == true
        t.espStealer   = _G.ESP_Stealer   == true
        t.espBaseOwner = _G.ESP_BaseOwner == true
        t.espBrainrot  = _G.ESP_Brainrot  == true
        t.espBox       = _G.ESP_BrainrotBox == true
        t.espTrap      = _G.ESP_Trap      == true
        t.espDist      = tonumber(_G.ESP_MaxDistance) or 700
        t.espTop       = tonumber(_G.ESP_BrainrotTop) or 5
        t.espBoxSize   = tonumber(_G.ESP_BoxSize) or 4.5
        t.espBoxTransp = tonumber(_G.ESP_BoxTransp) or 0.5
        t.invisAuto = _G.RyPrivInvisAuto == true
        t.invisDepth = tonumber(_G.RyPrivInvisDepth) or 4.2
        t.dropView = _G.RyPrivDropView == true
        t.invisAngle = tonumber(_G.RyPrivInvisAngle) or 180
        t.autoSteal = stealOn
        t.stealMode = _G.RyPrivStealMode
        t.tpMode = (_G.RyPrivTPMode == "fling") and "fling" or "normal"
        t.disableAnims = _G.RyPrivDisableAnims ~= false
        t.instantSteal = _G.RyPrivInstantSteal == true
        t.priorityList = _G.SHARED_PRIORITY_ITEMS
        -- panelX / panelY / panelPos used to live here. They were written and
        -- read back faithfully and then never applied to anything -- leftovers
        -- from a panel that no longer exists.
        t.tgtX = tonumber(_G._ryPriv_tgtX); t.tgtY = tonumber(_G._ryPriv_tgtY)
        t.tgtW = tonumber(_G._ryPriv_tgtW)
        t.adW = tonumber(_G._ryPriv_adW); t.adH = tonumber(_G._ryPriv_adH)
        
        if type(_G.RyPrivCloneKeyName)  == "string" then t.cloneKey  = _G.RyPrivCloneKeyName  else t.cloneKey  = nil end
        if type(_G.RyPrivKickKeyName)   == "string" then t.kickKey   = _G.RyPrivKickKeyName   else t.kickKey   = nil end
        if type(_G.RyPrivStopTPKeyName) == "string" then t.stopTpKey = _G.RyPrivStopTPKeyName else t.stopTpKey = nil end
        if type(_G.RyPrivNearestKey)    == "string" then t.nearestKey= _G.RyPrivNearestKey    else t.nearestKey= nil end
        if type(_G.RyPrivInstantKeyName)== "string" then t.instantKey= _G.RyPrivInstantKeyName else t.instantKey= nil end
        if type(_G.RyPrivDropKeyName)   == "string" then t.dropKey   = _G.RyPrivDropKeyName   else t.dropKey   = nil end
        if type(_G.RyPrivResetKeyName)  == "string" then t.resetKey  = _G.RyPrivResetKeyName  else t.resetKey  = nil end
        t.antiFlash = _G.RyPrivAntiFlash ~= false
        t.antiRagdoll = _G.RyPrivAntiRagdoll ~= false
        t.uiAlpha = tonumber(_G.RyPrivAlpha) or 0.12
        t.faceAway = _G.RyPrivFaceAway == true
        t.faceAwayNearest = _G.RyPrivFaceAwayNearest == true
        t.faceAwayDelay = tonumber(_G.RyPrivFaceAwayDelay) or 2
        t.antiBee   = _G.RyPrivAntiBee ~= false
        t.infJump   = _G.RyPrivInfJump ~= false
        t.antiDie   = _G.AntiDieDisabled ~= true
        t.carpetSpeedValue = tonumber(_G.RyPrivCarpetSpeedValue) or 140
        t.autoBuy = _G.RyPrivAutoBuy == true
        t.autoBuyRange = tonumber(_G.RyPrivAutoBuyRange) or 17
        t.autoBuyHover = tonumber(_G.RyPrivAutoBuyHover) or 9
        t.sbX = tonumber(_G._ryPriv_sbX); t.sbY = tonumber(_G._ryPriv_sbY)
        t.hdrX = tonumber(_G._ryPriv_hdrX); t.hdrY = tonumber(_G._ryPriv_hdrY)
        t.utX = tonumber(_G._ryPriv_utX); t.utY = tonumber(_G._ryPriv_utY)
        t.ivX = tonumber(_G._ryPriv_ivX); t.ivY = tonumber(_G._ryPriv_ivY)
        t.mbX = tonumber(_G._ryPriv_mbX); t.mbY = tonumber(_G._ryPriv_mbY)
        if type(_G.RyPrivUtilKeyName)       == "string" then t.utilKey    = _G.RyPrivUtilKeyName       else t.utilKey    = nil end
        if type(_G.RyPrivInvisPanelKeyName) == "string" then t.invKey     = _G.RyPrivInvisPanelKeyName else t.invKey     = nil end
        if type(_G.RyPrivAdminKeyName)      == "string" then t.adminKey   = _G.RyPrivAdminKeyName      else t.adminKey   = nil end
        t.adX = tonumber(_G._ryPriv_adX); t.adY = tonumber(_G._ryPriv_adY)
        t.apX = tonumber(_G._ryPriv_apX); t.apY = tonumber(_G._ryPriv_apY)
        if type(_G.RyPrivAutoTPKeyName)     == "string" then t.autoTpKey  = _G.RyPrivAutoTPKeyName     else t.autoTpKey  = nil end
        if type(_G.RyPrivAutoBuyKeyName)    == "string" then t.autoBuyKey = _G.RyPrivAutoBuyKeyName    else t.autoBuyKey = nil end
        if type(_G.RyPrivCloneNowKey)       == "string" then t.cloneNowKey= _G.RyPrivCloneNowKey       else t.cloneNowKey= nil end
        if type(_G.RyPrivCarpetSpeedKeyName)== "string" then t.carpetKey  = _G.RyPrivCarpetSpeedKeyName else t.carpetKey = nil end
        if type(_G._stp_tpKeyName)          == "string" then t.tpKey      = _G._stp_tpKeyName          else t.tpKey      = nil end
        -- Which hub layers the Settings UI tab has switched off.
        t.uiHidden = (type(_G.RyPrivUIHidden) == "table") and _G.RyPrivUIHidden or nil
        t.exX = tonumber(_G._ryPriv_exX); t.exY = tonumber(_G._ryPriv_exY)
        pcall(function() writefile(CFG_FILE, HS:JSONEncode(t)) end)
    end
    _G.RyPrivSaveSettings = saveTpSettings  

    if _G.TPVelocity == nil then _G.TPVelocity = 400 end
    if _G.RyPrivClimb == nil then _G.RyPrivClimb = 160 end
    if _G.RyPrivGoSpeed == nil then _G.RyPrivGoSpeed = 200 end
    if _G.RyPrivCFrameSpeed == nil then _G.RyPrivCFrameSpeed = 450 end
    if _G.RyPrivWalkSpeed == nil then _G.RyPrivWalkSpeed = 27 end
    if _G.LandingDelay == nil then _G.LandingDelay = 0.35 end
    if _G.RyPrivCloseSpeed == nil then _G.RyPrivCloseSpeed = 80 end
    if _G.RyPrivInvisDepth == nil then _G.RyPrivInvisDepth = 4.2 end
    if _G.RyPrivInvisAngle == nil then _G.RyPrivInvisAngle = 180 end

    local guiParent = (gethui and gethui()) or game:GetService("CoreGui") or PG
    for _, par in ipairs({ guiParent, PG }) do
        pcall(function()
            local old = par:FindFirstChild("uiMainLayer")
            if old then old:Destroy() end
        end)
    end

    -- ── CTRL OPENS SETTINGS ──────────────────────────────────────────────
    -- uiMainLayer used to be the features card's own ScreenGui, and Ctrl
    -- toggled it. With that card gone the key briefly became a master
    -- show/hide over every panel, which is not what it is for: per-layer
    -- show/hide is a job for the Settings panel's UI tab. Ctrl opens Settings.
    UIS.InputBegan:Connect(function(input, gameProcessed)
        if gameProcessed then return end
        if input.KeyCode ~= Enum.KeyCode.LeftControl
            and input.KeyCode ~= Enum.KeyCode.RightControl then return end
        if _G.RyPrivBindListening then return end
        if _G.RyPrivToggleSettings then pcall(_G.RyPrivToggleSettings) end
    end)

    do
        -- The header lives in its OWN ScreenGui, outside the layers LeftControl
        -- hides, so it stays on screen the whole time.
        --
        -- No background either: no Frame fill, no gradient, no stroke -- just the
        -- text over the game, with a shadow so it stays readable on light ground.
        local hdrSg = mk("ScreenGui", nil, {
            Name = "uiHeaderLayer", ResetOnSpawn = false, IgnoreGuiInset = true,
            DisplayOrder = 1000000, ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
        })
        pcall(function() hdrSg.Parent = guiParent end)
        if not hdrSg.Parent then hdrSg.Parent = PG end
        hdrSg.Enabled = true

        -- ── STATUS HUD ───────────────────────────────────────────────────
        -- No panel behind it, by request: the brand on one line and FPS / ping
        -- on the line under it, floating straight on the game. With nothing
        -- behind the text it has to carry its own contrast, so every label
        -- gets a heavy black stroke rather than the faint shadow the first
        -- version used -- that is what made it unreadable over bright ground.
        local GOOD = Color3.fromRGB(120, 205, 150)
        local WARN = Color3.fromRGB(232, 185, 95)
        local BAD  = Color3.fromRGB(226, 96, 104)

        local hdr = mk("Frame", hdrSg, {
            Name = "Bar",
            -- centred until the first drag pins it; RyPrivApplyPos below
            -- replaces both of these when there is a saved position
            AnchorPoint = Vector2.new(0.5, 0),
            Position = UDim2.new(0.5, 0, 0, 8),
            Size = UDim2.fromOffset(220, 40),
            BackgroundTransparency = 1, BorderSizePixel = 0,
            Active = true, ZIndex = 50,
        })
        _G.RyPrivApplyPos(hdr, "_ryPriv_hdrX", "_ryPriv_hdrY", nil, { pinAnchor = true })

        local function shout(props)
            props.BackgroundTransparency = 1
            props.TextStrokeTransparency = 0.15
            props.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
            props.ZIndex = 51
            return mk("TextLabel", hdr, props)
        end

        shout({
            Position = UDim2.fromOffset(0, 0), Size = UDim2.new(1, 0, 0, 20),
            Text = "leaked at discord.gg/Z7vPrxecnJ", Font = UIFB, TextSize = 15,
            TextColor3 = UIC.txt, TextXAlignment = Enum.TextXAlignment.Center,
        })

        -- Second line: the two readings side by side, each coloured by itself.
        local fpsLbl = shout({
            Position = UDim2.fromOffset(0, 20), Size = UDim2.new(0.5, -5, 0, 16),
            Text = "FPS --", Font = UIFB, TextSize = 12,
            TextColor3 = UIC.dim, TextXAlignment = Enum.TextXAlignment.Right,
        })
        local pingLbl = shout({
            Position = UDim2.new(0.5, 5, 0, 20), Size = UDim2.new(0.5, -5, 0, 16),
            Text = "PING --", Font = UIFB, TextSize = 12,
            TextColor3 = UIC.dim, TextXAlignment = Enum.TextXAlignment.Left,
        })

        -- A Frame with BackgroundTransparency 1 still receives input, so the
        -- invisible body of the bar is what you grab to move it. pinAnchor:
        -- until it has been dragged once it stays centred on the screen.
        _G.RyPrivPanelDrag(hdr, hdr, "_ryPriv_hdrX", "_ryPriv_hdrY", { pinAnchor = true })
        _G.RyPrivRegisterPanel(hdr, "_ryPriv_hdrX", "_ryPriv_hdrY", { pinAnchor = true })

        local frames, since, pingItem = 0, 0, nil
        RunService.RenderStepped:Connect(function(dt)
            frames, since = frames + 1, since + dt
            if since < 0.5 then return end

            local fps = math.floor(frames / since + 0.5)
            frames, since = 0, 0
            fpsLbl.Text = "FPS " .. fps
            fpsLbl.TextColor3 = (fps >= 100 and GOOD) or (fps >= 45 and WARN) or BAD

            if not pingItem then
                pcall(function()
                    pingItem = game:GetService("Stats").Network.ServerStatsItem["Data Ping"]
                end)
            end
            local ok, v = pcall(function() return pingItem:GetValue() end)
            if ok and v then
                local ms = math.floor(v + 0.5)
                pingLbl.Text = "PING " .. ms
                pingLbl.TextColor3 = (ms <= 120 and GOOD) or (ms <= 260 and WARN) or BAD
            else
                pingLbl.Text = "PING --"
                pingLbl.TextColor3 = UIC.dim
            end
        end)
    end

    -- ── OLD FEATURES PANEL REMOVED ───────────────────────────────────
    -- The Home / Tuning / Priority card that used to be built here is gone;
    -- the Utility and Invis Steal panels replace it.
    --
    -- Its flags and repaint hooks stay, because the rest of the file still
    -- reads them (RyPrivAutoTP in 11 places, RyPrivTPSafeMode in 8,
    -- RyPrivPriVersion in 5). The repaints become no-ops until the new
    -- Utility UI registers its own.
    if _G.RyPrivAutoTP     == nil then _G.RyPrivAutoTP     = true  end
    if _G.RyPrivTPSafeMode == nil then _G.RyPrivTPSafeMode = false end
    if _G.RyPrivPriVersion == nil then _G.RyPrivPriVersion = 0     end
    if type(_G.RyPrivRepaintAutoTP)  ~= "function" then _G.RyPrivRepaintAutoTP  = function() end end
    if type(_G.RyPrivRepaintTPSafe)  ~= "function" then _G.RyPrivRepaintTPSafe  = function() end end
    task.delay(1.5, diag)
end

do
    local RunService = game:GetService("RunService")
    local Lighting   = game:GetService("Lighting")
    local RS         = game:GetService("ReplicatedStorage")
    local LP         = game:GetService("Players").LocalPlayer

    local RAG_STATES = {
        [Enum.HumanoidStateType.Physics]     = true,
        [Enum.HumanoidStateType.Ragdoll]     = true,
        [Enum.HumanoidStateType.FallingDown] = true,
        [Enum.HumanoidStateType.GettingUp]   = true,
    }
    local KILL = {
        BallSocketConstraint = true, NoCollisionConstraint = true, HingeConstraint = true,
        BodyVelocity = true, BodyPosition = true, BodyGyro = true,
    }

    local conns, char, hum, hrp, anim, lastVel = {}, nil, nil, nil, nil, Vector3.zero

    -- _G.RyPrivAntiRagdoll is the Invis panel's AUTO RECOVER switch. Gating it
    -- here disables recovery AND the cleanup/velocity clamp in one place, since
    -- every one of them is behind this predicate.
    local function ragdolled()
        if _G.RyPrivAntiRagdoll == false then return false end
        return hum ~= nil and RAG_STATES[hum:GetState()] == true
    end

    local function cleanup()
        if not char then return end
        pcall(function()
            for _, o in ipairs(char:GetDescendants()) do
                if KILL[o.ClassName] then o:Destroy()
                elseif o:IsA("Motor6D") then o.Enabled = true
                elseif o:IsA("Attachment") and (o.Name == "A" or o.Name == "B") then o:Destroy() end
            end
        end)
        if anim then
            for _, t in pairs(anim:GetPlayingAnimationTracks()) do
                local n = t.Animation and t.Animation.Name:lower() or ""
                if n:find("rag") or n:find("fall") or n:find("hurt") or n:find("down") then t:Stop(0) end
            end
        end
    end

    local function recover()
        hum:ChangeState(Enum.HumanoidStateType.Running)
        cleanup()
        pcall(function() workspace.CurrentCamera.CameraSubject = hum end)
        pcall(function()
            require(LP:WaitForChild("PlayerScripts"):WaitForChild("PlayerModule", 10)):GetControls():Enable()
        end)
    end

    local function bind(c)
        for _, v in pairs(conns) do pcall(function() v:Disconnect() end) end
        conns = {}
        char = c
        hum  = c:WaitForChild("Humanoid", 10)
        hrp  = c:WaitForChild("HumanoidRootPart", 10)
        anim = hum and hum:WaitForChild("Animator", 10)
        lastVel = Vector3.zero
        if not hum then return end

        conns[#conns + 1] = hum.StateChanged:Connect(function()
            if ragdolled() then recover() end
        end)
        conns[#conns + 1] = c.DescendantAdded:Connect(function()
            if ragdolled() then cleanup() end
        end)
        local f = 0
        conns[#conns + 1] = RunService.Heartbeat:Connect(function()
            f = f + 1
            if f < 6 then return end
            f = 0
            if not (ragdolled() and hrp) then return end
            cleanup()
            local v = hrp.AssemblyLinearVelocity
            if (v - lastVel).Magnitude > 40 and v.Magnitude > 25 then
                hrp.AssemblyLinearVelocity = v.Unit * math.min(v.Magnitude, 15)
            end
            lastVel = v
        end)
    end

    LP.CharacterAdded:Connect(function(c) pcall(bind, c) end)
    if LP.Character then pcall(bind, LP.Character) end

    -- The anti-flash / anti-bee subsystem that lived here was a duplicate
    -- of the one further down: same BAD set, same nuke, same muteBuzz, a
    -- second Lighting.DescendantAdded hook, a second full Lighting sweep
    -- on boot and a second per-frame Heartbeat reading the camera. That
    -- copy also ran ungated, where the surviving one respects ANTI BEE.
end

;(function()
    -- Grey shell, light blue accents, see-through surfaces.
    --
    -- `card` was missing while five places referenced C.card -- the rows, the
    -- count chip, the mode chip, the locked state and mouse-leave. A nil value
    -- in a Lua table constructor means the key is never created at all, so
    -- BackgroundColor3 was simply never assigned and every new row came up in
    -- Roblox's default 163,162,165 -- the white rows, which "fixed" themselves
    -- on hover because MouseEnter assigns a colour that does exist.
    local C = {
        bg   = Color3.fromRGB(30, 32, 36),
        bg2  = Color3.fromRGB(40, 43, 48),
        card = Color3.fromRGB(52, 56, 62),
        row  = Color3.fromRGB(52, 56, 62),
        rowH = Color3.fromRGB(64, 69, 77),
        line = Color3.fromRGB(92, 99, 110),
        acc  = Color3.fromRGB(96, 178, 235),
        acc2 = Color3.fromRGB(155, 212, 248),
        txt  = Color3.fromRGB(240, 243, 247),
        dim  = Color3.fromRGB(158, 166, 178),
        -- Both of these were read but never defined. A key missing from a Lua
        -- table constructor is never created, so the read is nil and the
        -- property is silently left at its default rather than erroring.
        -- track: the KEYBINDS key buttons had no BackgroundColor3 assigned at
        --   all, and every action button's hover tween got an empty goal table.
        -- bad: the KICK button passes it as its accent, so the one destructive
        --   button in FEATURES rendered in the ordinary blue.
        track = Color3.fromRGB(64, 69, 77),
        bad   = Color3.fromRGB(216, 84, 92),
    }
    local function A() return math.clamp(tonumber(_G.RyPrivAlpha) or 0.12, 0, 0.9) end
    local FB, FR = Enum.Font.GothamBold, Enum.Font.Gotham
    local TS = game:GetService("TweenService")
    local EASE = TweenInfo.new(0.2, Enum.EasingStyle.Quart, Enum.EasingDirection.Out)
    local function tw(o, p) TS:Create(o, EASE, p):Play() end

    local function mk(cls, parent, props)
        local o = Instance.new(cls)
        for k, v in pairs(props or {}) do o[k] = v end
        o.Parent = parent
        return o
    end
    local host = (gethui and gethui()) or game:GetService("CoreGui")
        or LP:WaitForChild("PlayerGui")
    pcall(function()
        local old = host:FindFirstChild("uiListLayer")
        if old then old:Destroy() end
    end)

    local sg = mk("ScreenGui", nil, {
        Name = "uiListLayer", ResetOnSpawn = false,
        IgnoreGuiInset = true, DisplayOrder = 999998,
        ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
    })
    pcall(function() sg.Parent = host end)
    if not sg.Parent then sg.Parent = LP:WaitForChild("PlayerGui") end

    -- ── TARGET PANEL ─────────────────────────────────────────────────────
    -- Square throughout, like the rest of the hub: hard-edged shell, hard-edged
    -- rows, hard-edged badges. Structure comes from the blocks themselves and
    -- the gaps between them, so there are no dividing rules to clutter it.
    -- It reports the steal mode; the Utility panel sets it.
    local GOOD = Color3.fromRGB(120, 205, 150)
    -- One place the panel's corner style is decided. Hard square read as
    -- unfinished at this size; a small radius keeps the crisp look without
    -- the cut-out corners.
    local function pill(o, r) mk("UICorner", o, { CornerRadius = UDim.new(0, r or 6) }) return o end

    -- Height is driven by how many targets there are (see fitHeight below), so
    -- one brainrot gets a short panel instead of a tall box with a void in it.
    -- Only the width is yours to drag.
    --
    -- The body is nearly opaque on purpose. It used to sit at the shared UI
    -- alpha, which meant it took on whatever was behind it -- over grass the
    -- whole panel went green and the rows washed out. The shared dial still
    -- moves it, just over a much narrower range.
    local ROW_H, ROW_GAP = 32, 4
    local HEAD_H, FOOT_H = 42, 30
    local function bodyAlpha() return math.clamp(A() * 0.45, 0, 0.35) end

    local root = mk("Frame", sg, {
        Name = "Root", Active = true, BackgroundColor3 = C.bg,
        BackgroundTransparency = bodyAlpha(), BorderSizePixel = 0,
        ClipsDescendants = true,
        Size = UDim2.fromOffset(tonumber(_G._ryPriv_tgtW) or 278, 200),
        Position = UDim2.fromOffset(tonumber(_G._ryPriv_tgtX) or 0,
            tonumber(_G._ryPriv_tgtY) or 184),
    })
    mk("UICorner", root, { CornerRadius = UDim.new(0, 8) })
    mk("UIStroke", root, { Color = C.line, Thickness = 1, Transparency = 0.35 })

    -- Depth: a faint accent wash at the top of the panel fading out. A flat
    -- single-tone slab is what made this read as a plain box.
    do
        local sheen = mk("Frame", root, {
            Name = "Sheen", Size = UDim2.new(1, 0, 0, 90),
            BackgroundColor3 = C.acc, BorderSizePixel = 0, ZIndex = 0,
        })
        mk("UIGradient", sheen, {
            Rotation = 90,
            Transparency = NumberSequence.new({
                NumberSequenceKeypoint.new(0, 0.9),
                NumberSequenceKeypoint.new(1, 1),
            }),
        })
    end

    -- Header: accent spine, title, count; mode chip on the right, and a rule
    -- separating it from the list. The old one had a small floating dot and no
    -- divider, so the header and the rows ran together.
    local head = mk("TextButton", root, {
        Name = "Head", Size = UDim2.new(1, 0, 0, 42),
        BackgroundTransparency = 1, BorderSizePixel = 0,
        Text = "", AutoButtonColor = false, Active = true,
    })
    pill(mk("Frame", head, {
        AnchorPoint = Vector2.new(0, 0.5), Position = UDim2.new(0, 12, 0.5, 0),
        Size = UDim2.fromOffset(3, 15), BackgroundColor3 = C.acc,
        BorderSizePixel = 0, ZIndex = 2,
    }), 2)
    mk("TextLabel", head, {
        Position = UDim2.fromOffset(22, 0), Size = UDim2.fromOffset(66, 42),
        BackgroundTransparency = 1, Text = "TARGETS", Font = FB, TextSize = 12,
        TextColor3 = C.txt, TextXAlignment = Enum.TextXAlignment.Left, ZIndex = 2,
    })
    local countChip = pill(mk("Frame", head, {
        AnchorPoint = Vector2.new(0, 0.5), Position = UDim2.new(0, 86, 0.5, 0),
        Size = UDim2.fromOffset(24, 18), BackgroundColor3 = C.card,
        BackgroundTransparency = 0.25, BorderSizePixel = 0, ZIndex = 2,
    }), 4)
    local count = mk("TextLabel", countChip, {
        Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1,
        Text = "0", Font = FB, TextSize = 10, TextColor3 = C.dim,
        TextXAlignment = Enum.TextXAlignment.Center, ZIndex = 3,
    })
    -- hairline under the header
    mk("Frame", root, {
        Position = UDim2.fromOffset(0, 41), Size = UDim2.new(1, 0, 0, 1),
        BackgroundColor3 = C.line, BackgroundTransparency = 0.55,
        BorderSizePixel = 0, ZIndex = 2,
    })
    local modeChip = pill(mk("Frame", head, {
        AnchorPoint = Vector2.new(1, 0.5), Position = UDim2.new(1, -12, 0.5, 0),
        Size = UDim2.fromOffset(78, 20), BackgroundColor3 = C.acc,
        BackgroundTransparency = 0.75, BorderSizePixel = 0, ZIndex = 2,
    }), 4)
    local modeStroke = mk("UIStroke", modeChip, {
        Color = C.acc, Thickness = 1, Transparency = 0.4,
    })
    local modeLbl = mk("TextLabel", modeChip, {
        Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1,
        Text = "PRIORITY", Font = FB, TextSize = 9, TextColor3 = C.acc2,
        TextXAlignment = Enum.TextXAlignment.Center, ZIndex = 3,
    })

    -- Live mode read-out.
    task.spawn(function()
        local last = "\0"
        while sg.Parent do
            local m = _G.RyPrivStealMode
            local txt = (m == "nearest" and "NEAREST")
                or (m == "priority" and "PRIORITY") or "AUTO OFF"
            if txt ~= last then
                last = txt
                modeLbl.Text = txt
                modeLbl.TextColor3 = (m == nil) and C.dim or C.acc2
                modeChip.BackgroundColor3 = (m == nil) and C.card or C.acc
                modeChip.BackgroundTransparency = (m == nil) and 0.35 or 0.75
                modeStroke.Color = (m == nil) and C.line or C.acc
            end
            task.wait(0.25)
        end
    end)

    local list = mk("ScrollingFrame", root, {
        Position = UDim2.fromOffset(9, HEAD_H), Size = UDim2.new(1, -18, 0, 0),
        BackgroundTransparency = 1, BorderSizePixel = 0, ScrollBarThickness = 2,
        ScrollBarImageColor3 = C.line, ScrollBarImageTransparency = 0.5,
        CanvasSize = UDim2.new(),
        AutomaticCanvasSize = Enum.AutomaticSize.Y,
    })
    mk("UIListLayout", list, {
        Padding = UDim.new(0, ROW_GAP), SortOrder = Enum.SortOrder.LayoutOrder,
    })

    -- Empty state. Without it the panel just reads as a blank box while the
    -- scanner is still coming up, which looks like a dead UI.
    local emptyLbl = mk("TextLabel", root, {
        Name = "Empty", Position = UDim2.fromOffset(0, HEAD_H),
        Size = UDim2.new(1, 0, 0, 44), BackgroundTransparency = 1,
        Font = FR, TextSize = 11, TextColor3 = C.dim, TextWrapped = true,
        Text = "scanning for brainrots...",
        TextXAlignment = Enum.TextXAlignment.Center,
        TextYAlignment = Enum.TextYAlignment.Center, ZIndex = 2,
    })

    -- Footer: a hairline and the two totals, sitting on the panel itself.
    -- The separate filled bar it used to be just added another floating block.
    local foot = mk("Frame", root, {
        AnchorPoint = Vector2.new(0, 1), Position = UDim2.new(0, 0, 1, 0),
        Size = UDim2.new(1, 0, 0, FOOT_H), BackgroundTransparency = 1,
        BorderSizePixel = 0,
    })
    mk("Frame", foot, {
        Position = UDim2.fromOffset(9, 0), Size = UDim2.new(1, -18, 0, 1),
        BackgroundColor3 = C.line, BackgroundTransparency = 0.6, BorderSizePixel = 0,
    })
    local footL = mk("TextLabel", foot, {
        Position = UDim2.fromOffset(11, 0), Size = UDim2.new(0.5, -11, 1, 0),
        BackgroundTransparency = 1, Text = "0 targets", Font = FR, TextSize = 10,
        TextColor3 = C.dim, TextXAlignment = Enum.TextXAlignment.Left,
    })
    local footR = mk("TextLabel", foot, {
        AnchorPoint = Vector2.new(1, 0), Position = UDim2.new(1, -24, 0, 0),
        Size = UDim2.new(0.5, -24, 1, 0), BackgroundTransparency = 1,
        Text = "$0/s", Font = FB, TextSize = 11, TextColor3 = C.acc2,
        TextXAlignment = Enum.TextXAlignment.Right,
    })

    -- Grow the panel to fit what is in it, capped so a busy server scrolls
    -- instead of filling the screen.
    local function fitHeight(n)
        local maxRows = math.max(1, math.floor(tonumber(_G.RyPrivTargetMaxRows) or 8))
        local shown = math.clamp(n, 0, maxRows)
        local bodyH = (shown > 0)
            and (shown * ROW_H + (shown - 1) * ROW_GAP)
            or 44
        list.Size = UDim2.new(1, -18, 0, (shown > 0) and bodyH or 0)
        root.Size = UDim2.fromOffset(root.Size.X.Offset, HEAD_H + bodyH + 6 + FOOT_H)
    end
    fitHeight(0)

    -- This panel wrote its position to _G on release but never called
    -- RyPrivSaveSettings, and tgtX/tgtY were in neither the save table nor the
    -- loader -- so it was the one panel that could never persist at all.
    _G.RyPrivPanelDrag(root, head, "_ryPriv_tgtX", "_ryPriv_tgtY")
    _G.RyPrivApplyPos(root, "_ryPriv_tgtX", "_ryPriv_tgtY", nil)
    _G.RyPrivRegisterPanel(root, "_ryPriv_tgtX", "_ryPriv_tgtY")

    -- Width grip on the right edge. Height is not draggable any more: the
    -- panel sizes itself to its rows, and a hand-set height is exactly what
    -- produced the tall empty box.
    local resize = mk("TextButton", root, {
        Name = "ResizeHandle", AnchorPoint = Vector2.new(1, 0.5),
        Position = UDim2.new(1, -2, 0.5, 0), Size = UDim2.fromOffset(10, 40),
        BackgroundTransparency = 1, BorderSizePixel = 0, Text = "",
        AutoButtonColor = false, Active = true, ZIndex = 20,
    })
    mk("Frame", resize, {
        AnchorPoint = Vector2.new(0.5, 0.5), Position = UDim2.fromScale(0.5, 0.5),
        Size = UDim2.fromOffset(2, 22), BackgroundColor3 = C.dim,
        BackgroundTransparency = 0.55, BorderSizePixel = 0, ZIndex = 21,
    })

    -- Width only: the height is driven by fitHeight() from the row count, and
    -- a hand-set height is exactly what produced the tall empty box.
    _G.RyPrivPanelResize(root, resize, "_ryPriv_tgtW", nil, 230, 0, 520, 0)

    local function short(n)
        n = tonumber(n) or 0
        if n >= 1e9 then return string.format("%.1fB", n / 1e9) end
        if n >= 1e6 then return string.format("%.1fM", n / 1e6) end
        if n >= 1e3 then return string.format("%.1fK", n / 1e3) end
        return string.format("%d", n)
    end

    local _CTP_TOL_X, _CTP_TOL_Z = 62, 58
    local function _ctpFloorFromY(y)
        if y >= -6.9 and y <= 8.9 then return 1 end
        if y >= 11 and y <= 23.15 then return 2 end
        if y > 23.15 then return 3 end
        return nil
    end
    local function _ctpFloorFromSlot(slot, py)
        local s = tonumber(slot)
        if s then
            if s >= 19 then return 3 end
            if s >= 11 then return 2 end
            if s >= 1 then return 1 end
        end
        if py then return _ctpFloorFromY(py) end
        return nil
    end
    local function _ctpSameBase(petPos, myPos)
        local pIdx = getClosestBaseIdx(petPos)
        local mIdx = getClosestBaseIdx(myPos)
        if pIdx ~= mIdx then return false end
        local b = BASES_LOW[mIdx]
        if not b then return false end
        return math.abs(myPos.X - b.X) <= _CTP_TOL_X
           and math.abs(myPos.Z - b.Z) <= _CTP_TOL_Z
    end
    local function _ctpAllowed(pet)
        if not pet or not pet.position then return false end
        local char = LP.Character
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        if not hrp then return false end
        local myPos = hrp.Position
        if not _ctpSameBase(pet.position, myPos) then return false end
        local myFloor = _ctpFloorFromY(myPos.Y)
        if not myFloor then return false end
        local tgtFloor = _ctpFloorFromSlot(pet.slot, pet.position.Y)
        if not tgtFloor then return false end
        local diff = tgtFloor - myFloor
        local up = tonumber(_G.RyPrivClickTPMaxUp) or 3
        local down = tonumber(_G.RyPrivClickTPMaxDown) or 3
        return diff <= up and diff >= -down
    end

    local rows = {}

    -- Resolve icon IDs from the game's Mutation/Trait data.
    local _iconDataCache = {}
    local function _iconId(v)
        if v == nil then return nil end
        if type(v) == "number" then
            return "rbxassetid://" .. tostring(math.floor(v))
        end
        if type(v) == "string" and v ~= "" then
            if v:match("^rbxassetid://") or v:match("^rbxasset://") or v:match("^https?://") then
                return v
            end
            local n = tonumber(v)
            if n then return "rbxassetid://" .. tostring(math.floor(n)) end
        end
        return nil
    end

    local function _dataModule(name)
        if _iconDataCache[name] ~= nil then
            return _iconDataCache[name] or nil
        end
        local ok, result = pcall(function()
            local d = RS:FindFirstChild("Datas")
            local m = d and d:FindFirstChild(name)
            return m and require(m)
        end)
        if ok and type(result) == "table" then
            _iconDataCache[name] = result
            return result
        end
        _iconDataCache[name] = false
        return nil
    end

    local function _findIcon(data)
        if type(data) ~= "table" then return nil end
        for _, key in ipairs({
            "Icon", "IconId", "IconID", "Image", "ImageId", "ImageID",
            "Texture", "TextureId", "TextureID", "Thumbnail", "ThumbnailId",
        }) do
            local id = _iconId(data[key])
            if id then return id end
        end
        return nil
    end

    local function _mutationIcon(name)
        if not name or name == "" or name == "None" then return nil end
        local md = _dataModule("Mutations")
        return md and _findIcon(md[name]) or nil
    end

    local function _traitIcon(name)
        if not name or name == "" then return nil end
        local td = _dataModule("Traits")
        return td and _findIcon(td[name]) or nil
    end

    local function _clearIcons(holder)
        for _, child in ipairs(holder:GetChildren()) do
            if child:IsA("ImageLabel") then child:Destroy() end
        end
    end

    local function _addIcon(holder, image, order)
        if not image then return end
        mk("ImageLabel", holder, {
            Size = UDim2.fromOffset(13, 13),
            BackgroundTransparency = 1,
            BorderSizePixel = 0,
            Image = image,
            ScaleType = Enum.ScaleType.Fit,
            LayoutOrder = order or 1,
        })
    end

    -- ── ROW ──────────────────────────────────────────────────────────────
    -- A square block: rank badge on the left, the name, the mutation/trait
    -- icons, and the rate on the right coloured by tier against the best on
    -- screen -- green top third, blue middle, dim bottom -- the same way the
    -- HUD colours FPS and ping. The locked target fills solid accent and flips
    -- its text dark.
    -- Columns are pinned to the right edge so the name takes what is left and
    -- nothing collides when the panel is resized.
    -- ROW_H / ROW_GAP come from the panel above, so the height the rows take
    -- and the height the panel grows to can never drift apart.
    local function rowFor(uid)
        local r = rows[uid]
        if r and r.card.Parent then return r end

        local card = pill(mk("Frame", list, {
            Size = UDim2.new(1, 0, 0, ROW_H), BackgroundColor3 = C.card,
            BackgroundTransparency = 0.12, BorderSizePixel = 0, Active = true,
            ClipsDescendants = true,
        }))
        local stroke = mk("UIStroke", card, {
            Color = C.line, Thickness = 1, Transparency = 0.7,
        })

        -- Locked spine down the left edge, hidden until this row is the target.
        local lockBar = mk("Frame", card, {
            Name = "LockBar", Size = UDim2.new(0, 3, 1, 0),
            BackgroundColor3 = C.acc, BackgroundTransparency = 1,
            BorderSizePixel = 0, ZIndex = 4,
        })

        -- Value bar along the bottom edge: how this brainrot's rate compares to
        -- the best one on screen. The list is ranked, but a number alone does
        -- not show *by how much* -- this makes the spread readable at a glance
        -- without adding a second line of text to every row.
        mk("Frame", card, {
            AnchorPoint = Vector2.new(0, 1), Position = UDim2.new(0, 0, 1, 0),
            Size = UDim2.new(1, 0, 0, 2), BackgroundColor3 = C.line,
            BackgroundTransparency = 0.82, BorderSizePixel = 0, ZIndex = 2,
        })
        local vbar = mk("Frame", card, {
            Name = "ValueBar", AnchorPoint = Vector2.new(0, 1),
            Position = UDim2.new(0, 0, 1, 0), Size = UDim2.new(0, 0, 0, 2),
            BackgroundColor3 = C.acc, BackgroundTransparency = 0.15,
            BorderSizePixel = 0, ZIndex = 3,
        })

        -- Rank badge. Doubles as the locked marker.
        local bg = pill(mk("Frame", card, {
            Name = "RankDot", AnchorPoint = Vector2.new(0, 0.5),
            Position = UDim2.new(0, 8, 0.5, 0), Size = UDim2.fromOffset(20, 20),
            BackgroundColor3 = C.bg2, BackgroundTransparency = 0.1,
            BorderSizePixel = 0, ZIndex = 2,
        }), 4)
        local idx = mk("TextLabel", bg, {
            Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1,
            Text = "1", Font = FB, TextSize = 10, TextColor3 = C.dim,
            TextXAlignment = Enum.TextXAlignment.Center,
            TextYAlignment = Enum.TextYAlignment.Center, ZIndex = 3,
        })

        local nm = mk("TextLabel", card, {
            Position = UDim2.fromOffset(32, 0), Size = UDim2.new(1, -166, 0, ROW_H),
            BackgroundTransparency = 1, Font = FB, TextSize = 11, TextColor3 = C.txt,
            TextXAlignment = Enum.TextXAlignment.Left,
            TextTruncate = Enum.TextTruncate.AtEnd, Text = "", ZIndex = 2,
        })

        local iconHolder = mk("Frame", card, {
            AnchorPoint = Vector2.new(1, 0.5), Position = UDim2.new(1, -94, 0.5, 0),
            Size = UDim2.fromOffset(36, 16),
            BackgroundTransparency = 1, ZIndex = 2,
        })
        mk("UIListLayout", iconHolder, {
            FillDirection = Enum.FillDirection.Horizontal,
            HorizontalAlignment = Enum.HorizontalAlignment.Left,
            VerticalAlignment = Enum.VerticalAlignment.Center,
            Padding = UDim.new(0, 2),
            SortOrder = Enum.SortOrder.LayoutOrder,
        })

        local sub = mk("TextLabel", card, {
            AnchorPoint = Vector2.new(1, 0), Position = UDim2.new(1, -10, 0, 0),
            Size = UDim2.fromOffset(80, ROW_H), BackgroundTransparency = 1,
            Font = FB, TextSize = 12, TextColor3 = C.acc2,
            TextXAlignment = Enum.TextXAlignment.Right, Text = "", ZIndex = 2,
        })

        local hit = mk("TextButton", card, {
            Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1, Text = "",
            AutoButtonColor = false, Active = true, ZIndex = 5,
        })

        r = {
            card = card, bg = bg, idx = idx, name = nm, stroke = stroke,
            sub = sub, icons = iconHolder, hit = hit, lastLocked = false,
            vbar = vbar, lockBar = lockBar,
        }
        rows[uid] = r
        return r
    end

    task.spawn(function()
        local _pd = tonumber(_G.RyPrivPanelStartDelay) or 0
        if _pd > 0 then task.wait(_pd) end
        local _filled = false
        local _panelT0 = os.clock()
        while sg.Parent do
            local _pw = os.clock()
            while _filled
                and (_G.RyPrivStealHold or (_G.RyPrivIsTeleporting and _G.RyPrivIsTeleporting()))
                and os.clock() - _pw < (tonumber(_G.RyPrivPanelPauseMax) or 3) do
                task.wait(0.2)
            end
            local ok, pets = pcall(scanAllPets)
            if ok and type(pets) == "table" then
                local seen = {}
                -- Best rate on screen drives every row's value bar, and the
                -- running total feeds the footer.
                local _best, _total = 0, 0
                for _, p in ipairs(pets) do
                    if not p.conveyor then
                        local v = tonumber(p.mps) or 0
                        _total = _total + v
                        if v > _best then _best = v end
                    end
                end
                for i, p in ipairs(pets) do
                    if not p.conveyor then
                        local uid = _petUid(p)
                        seen[uid] = true
                        local r = rowFor(uid)
                        r.pet = p
                        local locked = (_G.RyPrivStealTargetUID == uid)
                        -- Guarded writes. Assigning a property the value it
                        -- already holds still fires its changed signal, and
                        -- for LayoutOrder that means re-sorting the whole
                        -- list -- every row, every refresh, usually for no
                        -- change at all.
                        if r.card.LayoutOrder ~= i then r.card.LayoutOrder = i end
                        local _ixt = tostring(i)
                        if r.idx and r.idx.Text ~= _ixt then r.idx.Text = _ixt end
                        local _nmt = tostring(p.name or "?")
                        if r.name.Text ~= _nmt then r.name.Text = _nmt end
                        local _subt = "$" .. short(p.mps or 0) .. "/s"
                        if r.sub.Text ~= _subt then r.sub.Text = _subt end
                        -- Rate colour by tier against the best on screen. The
                        -- locked row is solid accent, so its text goes dark
                        -- instead of taking a tier colour.
                        local frac = (_best > 0) and ((tonumber(p.mps) or 0) / _best) or 0
                        local vcol = locked and C.bg
                            or (frac >= 0.66 and GOOD)
                            or (frac >= 0.33 and C.acc2) or C.dim
                        if r.sub.TextColor3 ~= vcol then r.sub.TextColor3 = vcol end

                        -- Value bar: same tier colour as the rate, so the bar
                        -- and the number agree. Guarded, like the text writes.
                        if r.vbar then
                            local _w = UDim2.new(math.clamp(frac, 0, 1), 0, 0, 2)
                            if r.vbar.Size ~= _w then r.vbar.Size = _w end
                            local _bc = locked and C.acc2 or vcol
                            if r.vbar.BackgroundColor3 ~= _bc then
                                r.vbar.BackgroundColor3 = _bc
                            end
                        end

                        -- Mutation + Traits icons.
                        --
                        -- These were torn down and rebuilt on every refresh
                        -- for every row -- with eight rows that is dozens of
                        -- Instance destroys and creates a second, each one
                        -- re-triggering the layout, to redraw icons that
                        -- essentially never change. Rebuild only when the
                        -- mutation/trait set actually differs.
                        local _sig = tostring(p.mutation or "-")
                        if type(p.traits) == "table" then
                            for _, tr in ipairs(p.traits) do
                                _sig = _sig .. "|" .. tostring(tr)
                            end
                        end
                        if _sig ~= r.iconSig then
                            r.iconSig = _sig
                            _clearIcons(r.icons)
                            local _io = 0
                            local _mi = _mutationIcon(p.mutation)
                            if _mi then
                                _io = _io + 1
                                _addIcon(r.icons, _mi, _io)
                            end
                            if type(p.traits) == "table" then
                                for _, tr in ipairs(p.traits) do
                                    local _ti = _traitIcon(tr)
                                    if _ti then
                                        _io = _io + 1
                                        _addIcon(r.icons, _ti, _io)
                                    end
                                end
                            end
                        end
                        if locked ~= r.lastLocked then
                            r.lastLocked = locked
                            -- Locked: the whole row fills accent, its badge
                            -- goes dark, and the text on it flips to dark.
                            tw(r.bg, {
                                BackgroundColor3 = locked and C.bg or C.bg2,
                                BackgroundTransparency = locked and 0.15 or 0.1,
                            })
                            tw(r.card, {
                                BackgroundColor3 = locked and C.acc or C.card,
                                BackgroundTransparency = locked
                                    and 0.04 or 0.12,
                            })
                            if r.stroke then
                                r.stroke.Color = locked and C.acc2 or C.line
                                r.stroke.Transparency = locked and 0.25 or 0.7
                            end
                            if r.idx then r.idx.TextColor3 = locked and C.acc2 or C.dim end
                            if r.name then r.name.TextColor3 = locked and C.bg or C.txt end
                            if r.lockBar then
                                tw(r.lockBar, { BackgroundTransparency = locked and 0 or 1 })
                            end
                        end
                        if not r.wired then
                            r.wired = true
                            local myUid = uid
                            local _crow = r
                            r.hit.MouseButton1Click:Connect(function()
                                if _G.RyPrivStealTargetUID == myUid then
                                    _G.RyPrivStealTargetUID = nil
                                    _G.RyPrivStealTarget = nil
                                else
                                    _G.RyPrivStealTargetUID = myUid
                                    local pet = _crow.pet
                                    do
                                        local okS, fresh = pcall(scanAllPets)
                                        if okS and type(fresh) == "table" then
                                            for _, fp in ipairs(fresh) do
                                                if _petUid(fp) == myUid then pet = fp break end
                                            end
                                        end
                                    end
                                    if pet and pet.position and not isTeleporting
                                        and not _G.RyPrivClickTPBusy
                                        and _ctpAllowed(pet) then
                                        _G.RyPrivClickTPBusy = true
                                        isTeleporting = true
                                        if _G.RyPrivArmSteal then pcall(_G.RyPrivArmSteal, pet) end
                                        task.spawn(function()
                                            pcall(function()
                                                goToBrainrot(pet.position, pet.slot)
                                            end)
                                            isTeleporting = false
                                            _G.RyPrivClickTPBusy = false
                                        end)
                                    end
                                end
                            end)
                            r.hit.MouseEnter:Connect(function()
                                if _G.RyPrivStealTargetUID ~= myUid then
                                    tw(r.card, {
                                        BackgroundColor3 = C.rowH,
                                        BackgroundTransparency = 0.04,
                                    })
                                    if r.stroke then r.stroke.Transparency = 0.45 end
                                end
                            end)
                            r.hit.MouseLeave:Connect(function()
                                if _G.RyPrivStealTargetUID ~= myUid then
                                    tw(r.card, {
                                        BackgroundColor3 = C.card,
                                        BackgroundTransparency = 0.12,
                                    })
                                    if r.stroke then r.stroke.Transparency = 0.7 end
                                end
                            end)
                        end
                    end
                end
                local n = 0
                for uid, r in pairs(rows) do
                    if seen[uid] then n = n + 1
                    else r.card:Destroy() rows[uid] = nil end
                end
                count.Text = tostring(n)
                footL.Text = (n == 1) and "1 target" or (tostring(n) .. " targets")
                footR.Text = "$" .. short(_total) .. "/s"
                emptyLbl.Visible = (n == 0)
                if n == 0 then
                    emptyLbl.Text = _filled and "no brainrots in range"
                        or "scanning for brainrots..."
                end
                fitHeight(n)
                root.BackgroundTransparency = bodyAlpha()
                if n > 0 then _filled = true end
            end
            if not _filled and (os.clock() - _panelT0) < (tonumber(_G.RyPrivPanelRush) or 10) then
                task.wait(tonumber(_G.RyPrivPanelRushGap) or 0.05)
            else
                task.wait(tonumber(_G.RyPrivPanelScanGap) or 0.4)
            end
        end
    end)

    -- Show/hide for this panel is the Settings UI tab's job now; a second API
    -- writing sg.Enabled would just fight the loop that re-applies it.
end)()

-- ============================================================
-- extra
-- ============================================================
-- The flat task.wait(3) that used to sit here is gone. It dates from when
-- this half was a separate script that had to let the teleport half publish
-- its globals first. Both halves are one chunk now, everything this half
-- reads (BootWait, SaveSettings, EquipCarpet, CarpetEngaging, ScanAllPets,
-- DoClone, IsTeleporting) is already published above, and the guard block
-- immediately below polls for them anyway and installs no-op fallbacks.
--
-- It sat in the MAIN chunk, so it did not delay one feature -- it delayed
-- every single thing defined after this line by three seconds, which is why
-- the target panel and header appeared at once and the Settings, Utility,
-- Invis, Admin and Auto-AP windows turned up later.

-- Players, RunService, UIS, RS and LP are already file-scope locals from the
-- top of this file; re-declaring them here only shadowed them with identical
-- values and cost five more registers. Lighting and TeleportService are not
-- declared anywhere above, so they stay.
local Lighting        = game:GetService("Lighting")
local TeleportService = game:GetService("TeleportService")

do
    local t0 = os.clock()
    while type(_G.RyPrivBootWait) ~= "function" and os.clock() - t0 < 20 do task.wait(0.1) end
end
if type(_G.RyPrivBootWait)      ~= "function" then _G.RyPrivBootWait = function() end end
if type(_G.RyPrivCarpetEngaging) ~= "function" then _G.RyPrivCarpetEngaging = function() return false end end
if type(_G.RyPrivSaveSettings)   ~= "function" then _G.RyPrivSaveSettings = function() end end

local function equipCarpet() if _G.RyPrivEquipCarpet then return _G.RyPrivEquipCarpet() end end
local function scanAllPets() if _G.RyPrivScanAllPets then return _G.RyPrivScanAllPets() end return {} end

if _G.RyPrivAntiFlash        == nil then _G.RyPrivAntiFlash        = true end
if _G.RyPrivAntiBee          == nil then _G.RyPrivAntiBee          = true end
if _G.RyPrivInfJump          == nil then _G.RyPrivInfJump          = true end
if _G.RyPrivInvisDepth       == nil then _G.RyPrivInvisDepth       = 4.2  end
if _G.RyPrivDropView         == nil then _G.RyPrivDropView         = false end
if _G.RyPrivInvisAngle       == nil then _G.RyPrivInvisAngle       = 180  end
if _G.RyPrivAlertMinGen      == nil then _G.RyPrivAlertMinGen      = 80e6 end
if _G.RyPrivCarpetSpeedValue == nil then _G.RyPrivCarpetSpeedValue = 140  end
if _G.RyPrivFOVEnabled       == nil then _G.RyPrivFOVEnabled       = false end

-- FOV enforcement: fully independent of every other feature toggle (it was
-- previously piggybacking on the Anti Bee heartbeat loop, so if that was
-- off, the custom FOV never applied at all and the camera just sat at the
-- game's own default -- which happened to be 70, giving the impression it
-- was "resetting" when really it just never got set in the first place).
task.spawn(function()
    while true do
        if _G.RyPrivFOVEnabled then
            local cam = workspace.CurrentCamera
            local target = tonumber(_G.RyPrivFOV) or 70
            if cam and math.abs(cam.FieldOfView - target) > 0.1 then
                cam.FieldOfView = target
            end
        end
        task.wait(0.2)
    end
end)

;(function()
    local FX = { ParticleEmitter = true, Beam = true, Trail = true, Fire = true,
        Smoke = true, Sparkles = true, Explosion = true, PointLight = true,
        SpotLight = true, SurfaceLight = true }

    local function kill(item)
        if pcall(function() item:Destroy() end) then return end
        pcall(function()
            for _, d in ipairs(item:GetDescendants()) do
                if d:IsA("BasePart") then
                    d.Transparency = 1
                    d.CanCollide = false
                    d.CanQuery = false
                    d.CastShadow = false
                    d.LocalTransparencyModifier = 1
                elseif FX[d.ClassName] then
                    d.Enabled = false
                elseif d:IsA("Sound") then
                    d.Volume = 0
                    d:Stop()
                end
            end
        end)
    end

    local function strip(char)
        if not char or _G.RyPrivAntiFlash == false then return end
        for _, item in ipairs(char:GetChildren()) do
            if item:IsA("Accessory") then kill(item) end
        end
    end

    local hooked = setmetatable({}, { __mode = "k" })
    local function hookChar(char)
        if not char or hooked[char] then return end
        hooked[char] = true
        strip(char)
        char.ChildAdded:Connect(function(c)
            if c:IsA("Accessory") and _G.RyPrivAntiFlash ~= false then
                task.defer(kill, c)
            end
        end)
    end
    local function hookPlayer(p)
        if p.Character then task.spawn(hookChar, p.Character) end
        p.CharacterAdded:Connect(hookChar)
    end
    for _, p in ipairs(Players:GetPlayers()) do pcall(hookPlayer, p) end
    Players.PlayerAdded:Connect(hookPlayer)

    workspace.DescendantAdded:Connect(function(d)
        if _G.RyPrivAntiFlash == false then return end
        if d.ClassName == "Accessory" then
            local par = d.Parent
            if par and par:FindFirstChildOfClass("Humanoid") then task.defer(kill, d) end
        end
    end)

    task.spawn(function()
        while true do
            if _G.RyPrivAntiFlash ~= false then
                for _, p in ipairs(Players:GetPlayers()) do
                    if p.Character then strip(p.Character) end
                end
                for _, m in ipairs(workspace:GetChildren()) do
                    if m:IsA("Model") and m:FindFirstChildOfClass("Humanoid") then strip(m) end
                end
            end
            task.wait(6) 
        end
    end)
end)()

;(function()
    local TeleportService = game:GetService("TeleportService")
    if _G.RyPrivCleanErrStop then pcall(_G.RyPrivCleanErrStop) end

    local _ref = cloneref or function(x) return x end
    local GS = _ref(game:GetService("GuiService"))
    local running, conns = true, {}

    task.spawn(function()
        while running do
            pcall(function() GS:ClearError() end)
            task.wait(0.1)
        end
    end)

    conns[#conns + 1] = TeleportService.TeleportInitFailed:Connect(function(plr, result, message, placeId)
        if plr ~= LP then return end
        local name = "Unknown"
        pcall(function() name = result and result.Name or tostring(result) end)
        _G.RyPrivCleanErrLast = {
            result = name, message = tostring(message or ""),
            placeId = placeId, at = os.clock(),
        }
    end)

    _G.RyPrivCleanErrStop = function()
        running = false
        for _, c in ipairs(conns) do pcall(function() c:Disconnect() end) end
        conns = {}
        _G.RyPrivCleanErrStop = nil
    end
end)()

;(function()
    local held = false
    local function hop()
        local c = LP.Character
        local hrp = c and c:FindFirstChild("HumanoidRootPart")
        local hum = c and c:FindFirstChildOfClass("Humanoid")
        if not hrp or not hum or hum.Health <= 0 then return end
        hrp.Velocity = Vector3.new(hrp.Velocity.X, hum.JumpPower or 50, hrp.Velocity.Z)
    end
    local function on() return _G.RyPrivInfJump ~= false end
    UIS.JumpRequest:Connect(function() if on() then hop() end end)
    UIS.InputBegan:Connect(function(i, g)
        if not g and i.KeyCode == Enum.KeyCode.Space then held = true end
    end)
    UIS.InputEnded:Connect(function(i)
        if i.KeyCode == Enum.KeyCode.Space then held = false end
    end)
    RunService.Heartbeat:Connect(function() if held and on() then hop() end end)
end)()




if _G.AntiDieDisabled == nil then _G.AntiDieDisabled = false end
task.spawn(function()
local __P = game:GetService("Players")
while not __P.LocalPlayer do task.wait() end
local okAD, errAD = pcall(function()
    local Players    = game:GetService("Players")
    local RunService = game:GetService("RunService")
    local LP         = Players.LocalPlayer

    local _conn, _diedConn, _hbConn
    local function _harden(hum)
        pcall(function() hum.BreakJointsOnDeath = false end)
        pcall(function() hum.RequiresNeck = false end)
        pcall(function() hum:SetStateEnabled(Enum.HumanoidStateType.Dead, false) end)
        pcall(function() hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false) end)
        pcall(function() hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false) end)
        pcall(function() hum:SetStateEnabled(Enum.HumanoidStateType.Physics, false) end)
    end
    local function _revive(hum)
        pcall(function() hum.Health = hum.MaxHealth end)
        pcall(function() hum:ChangeState(Enum.HumanoidStateType.Running) end)
    end
    local function _bind()
        local char = LP.Character
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        if not hum then return end
        _harden(hum)
        if _conn then pcall(function() _conn:Disconnect() end) end
        if _diedConn then pcall(function() _diedConn:Disconnect() end) end
        if _hbConn then pcall(function() _hbConn:Disconnect() end) end
        _conn = hum:GetPropertyChangedSignal("Health"):Connect(function()
            if _G.AntiDieDisabled then return end
            if hum.Health <= 0 then _revive(hum) end
        end)
        _diedConn = hum.Died:Connect(function()
            if _G.AntiDieDisabled then return end
            _revive(hum)
        end)
        local _lastHarden = 0
        _hbConn = RunService.Heartbeat:Connect(function()
            if _G.AntiDieDisabled or not hum or not hum.Parent then return end
            local now = os.clock()
            if now - _lastHarden >= 0.5 then _lastHarden = now; _harden(hum) end
            if hum.Health <= 0 then _revive(hum) end
            
            if _G.RyPrivStealHold and hum.Health < hum.MaxHealth then
                pcall(function() hum.Health = hum.MaxHealth end)
            end
            local char = hum.Parent
            local hrp = char and char:FindFirstChild("HumanoidRootPart")
            if char and hrp then
                local state = hum:GetState()
                local _rag = (state == Enum.HumanoidStateType.Physics
                    or state == Enum.HumanoidStateType.Ragdoll
                    or state == Enum.HumanoidStateType.FallingDown)
                if not _rag then
                    local _et = tonumber(LP:GetAttribute("RagdollEndTime"))
                    if _et and (_et - workspace:GetServerTimeNow()) > 0 then _rag = true end
                end
                if _rag then
                    pcall(function() LP:SetAttribute("RagdollEndTime", workspace:GetServerTimeNow()) end)
                    pcall(function() hum:ChangeState(Enum.HumanoidStateType.Running) end)
                    if not _G.__ryPrivResetBusy and LP:GetAttribute("Stealing") ~= true then
                        pcall(function() hrp.AssemblyLinearVelocity = Vector3.zero end)
                    end
                    local cam = workspace.CurrentCamera
                    if cam and cam.CameraSubject ~= hum then
                        pcall(function() cam.CameraSubject = hum end)
                    end
                    for _, obj in ipairs(char:GetDescendants()) do
                        if obj:IsA("BallSocketConstraint") or (obj.Name and obj.Name:find("RagdollAttachment")) then
                            pcall(function() obj:Destroy() end)
                        end
                    end
                end
            end
            if hum:GetState() == Enum.HumanoidStateType.Dead then
                pcall(function() hum:ChangeState(Enum.HumanoidStateType.Running) end)
            end
        end)
    end
    _bind()
    LP.CharacterAdded:Connect(function(char)
        local hum = char:WaitForChild("Humanoid", 5)
        if hum then _harden(hum) end
        task.wait(0.1)
        _bind()
    end)
end)
if not okAD then warn("[ANTI-DIE] BLOC EN ERREUR: " .. tostring(errAD)) end
end)

;(function()
    local Lighting = game:GetService("Lighting")
    local BAD = { Blue = true, DiscoEffect = true, BeeBlur = true,
        Flashbang = true, ColorCorrection = true }
    local function on() return _G.RyPrivAntiBee ~= false end

    local function nuke(o)
        if on() and o and o.Parent and BAD[o.Name] then pcall(function() o:Destroy() end) end
    end

    local buzz
    local function muteBuzz()
        if not on() then return end
        pcall(function()
            if not (buzz and buzz.Parent) then
                local ctl = RS:FindFirstChild("Controllers")
                local item = ctl and ctl:FindFirstChild("ItemController")
                local bee = item and item:FindFirstChild("BeeLauncherController")
                local s = bee and bee:FindFirstChild("Buzzing")
                if s and s:IsA("Sound") then buzz = s end
            end
            if buzz then
                buzz.Volume = 0
                if buzz.IsPlaying then buzz:Stop() end
            end
        end)
    end

    local guarded = {}
    local function guard(Controls, original)
        if not Controls or guarded[Controls] then return end
        local base = original or Controls.moveFunction
        if not base then return end
        local function safeMove(self, mv, rtc) return base(self, mv, rtc) end
        guarded[Controls] = safeMove
        Controls.moveFunction = safeMove
        RunService.Heartbeat:Connect(function()
            if not on() then return end
            if Controls.moveFunction ~= safeMove then Controls.moveFunction = safeMove end
        end)
    end
    local function protect()
        pcall(function()
            local cc = RS:FindFirstChild("Controllers")
            local mod = cc and cc:FindFirstChild("CharacterController")
            local m = mod and require(mod)
            if type(m) == "table" then guard(m.Controls, m.originalMoveFunction) end
        end)
        pcall(function()
            local ps = LP:WaitForChild("PlayerScripts", 5)
            local pm = ps and ps:FindFirstChild("PlayerModule")
            if pm then guard(require(pm):GetControls()) end
        end)
    end

    task.spawn(function()
        LP:WaitForChild("PlayerScripts", 8)
        _G.RyPrivBootWait()
        Lighting.DescendantAdded:Connect(nuke)
        do local n = 0
            for _, o in ipairs(Lighting:GetDescendants()) do
                n = n + 1; if n % 150 == 0 then task.wait() end
                nuke(o)
            end
        end
        protect()
        local acc = 1
        RunService.Heartbeat:Connect(function(dt)
            if not on() then return end
            local cam = workspace.CurrentCamera
            if cam and math.abs(cam.FieldOfView - 20) < 0.01 then
                cam.FieldOfView = tonumber(_G.RyPrivFOV) or 70
            end
            acc = acc + dt
            if acc < 0.5 then return end
            acc = 0
            muteBuzz()
            
        end)
    end)
    LP.CharacterAdded:Connect(function() task.delay(1, protect) end)
end)()

;(function()
    local active, conns = false, {}
    local realHRP, cloneHRP, track, hip, origT
    local stuck = false
    local start, stop

    -- How far under the pet the real root is pushed. This used to span 0.01 to
    -- 0.20 studs across the entire slider, so even at the old maximum the
    -- character only sank a fifth of a stud and still showed above the pet.
    --
    -- The range now runs to 15 and travels about twelve times as far, so the
    -- top of the slider actually buries it. RyPrivInvisDepthScale moves the
    -- ceiling again if 2.5 studs is still not enough.
    local function depth()
        local d = math.clamp(tonumber(_G.RyPrivInvisDepth) or 4.2, 0, 15)
        local scale = tonumber(_G.RyPrivInvisDepthScale) or 2.5
        return 0.01 + (d / 15) * scale
    end
    local function angle()
        return math.clamp(tonumber(_G.RyPrivInvisAngle) or 0, 0, 360)
    end
    local function add(c) conns[#conns + 1] = c end
    local function rewire(char, from, to)
        for _, v in ipairs(char:GetDescendants()) do
            if v:IsA("Weld") or v:IsA("Motor6D") then
                if v.Part0 == from then v.Part0 = to end
                if v.Part1 == from then v.Part1 = to end
            end
        end
    end

    stop = function()
        active, stuck = false, false
        for _, c in ipairs(conns) do pcall(function() c:Disconnect() end) end
        conns = {}
        if origT then
            for p, v in pairs(origT) do
                if p and p.Parent then pcall(function() p.LocalTransparencyModifier = v end) end
            end
            origT = nil
        end
        if track then
            pcall(function() track:AdjustSpeed(1) track:Stop(0) track:Destroy() end)
            track = nil
        end
        local char = LP.Character
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        if realHRP and realHRP:IsDescendantOf(game) and hum then
            -- Re-parenting restarts every LocalScript in the character, and the
            -- game's Animate script does not survive its own restart unless the
            -- empty animation entry is filled in first.
            if _G.RyPrivRepairAnimate then pcall(_G.RyPrivRepairAnimate, char) end
            local tmp = Instance.new("Model")
            tmp.Parent = game
            char.Parent = tmp
            realHRP.Parent = char
            char.PrimaryPart = realHRP
            char.Parent = workspace
            realHRP.CanCollide = true
            rewire(char, cloneHRP, realHRP)
            if cloneHRP then
                local at = cloneHRP.CFrame
                cloneHRP:Destroy()
                cloneHRP = nil
                realHRP.CFrame = at
            end
            hum.HipHeight = hip or 0
            tmp:Destroy()
        end
        realHRP, cloneHRP = nil, nil
        _G.RyPrivInvisActive = false
    end

    local function animTrick(char, hum)
        if not (char and hum and hum.Health > 0) then return end
        local a = Instance.new("Animation")
        a.AnimationId = "http://www.roblox.com/asset/?id=18537363391"
        local animator = hum:FindFirstChildOfClass("Animator") or Instance.new("Animator", hum)
        track = animator:LoadAnimation(a)
        track.Priority = Enum.AnimationPriority.Action4
        track.Looped = true
        track:Play(0, 1, 0)
        a:Destroy()
        add(track.Stopped:Connect(function()
            if active then animTrick(char, hum) end
        end))
        task.defer(function()
            if not track then return end
            track.TimePosition = 0.7
            task.delay(0.1, function() if track then track:AdjustSpeed(math.huge) end end)
        end)
    end

    start = function()
        local char = LP.Character
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        if not hum or active then return false end

        local folder = workspace:FindFirstChild(LP.Name)
        if folder then
            for _, n in ipairs({ "DoubleRig", "Constraints" }) do
                local x = folder:FindFirstChild(n)
                if x then x:Destroy() end
            end
            add(folder.ChildAdded:Connect(function(c)
                if c.Name == "DoubleRig" or c.Name == "Constraints" then c:Destroy() end
            end))
        end
        for _, s in ipairs({ "Dead", "FallingDown", "Ragdoll" }) do
            hum:SetStateEnabled(Enum.HumanoidStateType[s], false)
        end

        hip = hum.HipHeight
        realHRP = char:FindFirstChild("HumanoidRootPart")
        if not (realHRP and realHRP.Parent) then return false end

        local tmp = Instance.new("Model")
        tmp.Parent = game
        -- same reason as the stop path: make Animate survivable before the
        -- character is moved
        if _G.RyPrivRepairAnimate then pcall(_G.RyPrivRepairAnimate, char) end
        char.Parent = tmp
        cloneHRP = realHRP:Clone()
        cloneHRP.Parent = char
        realHRP.Parent = workspace.CurrentCamera
        cloneHRP.CFrame = realHRP.CFrame
        char.PrimaryPart = cloneHRP
        char.Parent = workspace
        rewire(char, realHRP, cloneHRP)
        tmp:Destroy()

        active = true
        animTrick(char, hum)

        origT = {}
        for _, p in ipairs(char:GetDescendants()) do
            if p:IsA("BasePart") then
                origT[p] = p.LocalTransparencyModifier
                p.LocalTransparencyModifier = 1
            end
        end
        add(char.DescendantAdded:Connect(function(d)
            if not (active and d:IsA("BasePart")) then return end
            if origT and origT[d] == nil then origT[d] = d.LocalTransparencyModifier end
            pcall(function() d.LocalTransparencyModifier = 1 end)
        end))

        add(RunService.Heartbeat:Connect(function()
            if not active or not hum.Parent or _G.__ryPrivResetBusy then return end
            hum.Health = hum.MaxHealth
            hum:SetStateEnabled(Enum.HumanoidStateType.Dead, false)
            local st = hum:GetState()
            if st == Enum.HumanoidStateType.Dead or st == Enum.HumanoidStateType.FallingDown
                or st == Enum.HumanoidStateType.Ragdoll then
                hum:ChangeState(Enum.HumanoidStateType.Running)
            end
        end))

        -- Keep the hidden root perfectly aligned with the visible character.
        -- Do not rotate it upside-down or apply a vertical offset: those were
        -- the main causes of the "head hitting the ground" / abnormal movement.
        add(RunService.PreSimulation:Connect(function()
            if not (active and realHRP and realHRP.Parent and hum.Parent and hum.Health > 0) then
                return
            end

            local base = char.PrimaryPart or char:FindFirstChild("HumanoidRootPart")
            if not (base and base:IsA("BasePart")) then return end

            -- Keep the real/hidden root slightly below the visible body,
            -- matching the original pet-on-ground placement, but do NOT
            -- rotate the character upside-down.
            local groundOffset = hum.HipHeight + (base.Size.Y / 2) - 1 + depth()
            local cf = base.CFrame - Vector3.new(0, groundOffset, 0)
            realHRP.CFrame = cf * CFrame.Angles(math.rad(angle()), 0, 0)
            realHRP.AssemblyLinearVelocity = base.AssemblyLinearVelocity
            realHRP.AssemblyAngularVelocity = base.AssemblyAngularVelocity
            realHRP.CanCollide = false

            if realHRP.Parent ~= workspace.CurrentCamera then
                realHRP.Parent = workspace.CurrentCamera
            end
        end))

        add(LP.CharacterAdded:Connect(function() if active then stop() end end))
        _G.RyPrivInvisActive = true
        return true
    end

    _G.RyPrivInvisStart, _G.RyPrivInvisStop = start, stop
    _G.RyPrivInvisToggle = function()
        if active then stop() else start() end
    end

    local armed, autoOn = false, false
    local function syncAuto()
        local want = (_G.RyPrivInvisAuto == true) and (LP:GetAttribute("Stealing") == true)
        if not want then
            armed = false
            if active and autoOn then autoOn = false pcall(stop) end
            return
        end
        if active or armed then return end
        armed = true
        task.delay(tonumber(_G.RyPrivInvisAutoDelay) or 1.5, function()
            armed = false
            if _G.RyPrivInvisAuto == true and not active
                and LP:GetAttribute("Stealing") == true then
                autoOn = true
                pcall(start)
            end
        end)
    end
    _G.RyPrivInvisSync = syncAuto
    LP:GetAttributeChangedSignal("Stealing"):Connect(syncAuto)
end)()

;(function()
    -- `getinfo` is not a global in Lua, in Roblox, or in any executor -- the
    -- spelling is debug.getinfo. It was never defined in this file, so the
    -- guard below read nil and strip() returned on its first line every single
    -- time it was called. Resolve it properly, preferring a bare global if some
    -- executor does provide one.
    local _getinfo = (type(getinfo) == "function" and getinfo)
        or (type(debug) == "table" and type(debug.getinfo) == "function" and debug.getinfo)
        or nil

    -- The old match was an exact compare against "=ReplicatedFirst.test". A
    -- game update that renames or moves that script defeats an exact compare
    -- silently -- no error, strip just stops matching anything -- so match on a
    -- needle instead. RyPrivACSource overrides it with a string or a list.
    local function _acNeedles()
        local n = _G.RyPrivACSource
        if type(n) == "string" then return { n:lower() } end
        if type(n) == "table" then return n end
        return { "replicatedfirst" }
    end

    -- Every source seen on those signals, so a rename can be read off rather
    -- than guessed at: print(_G.RyPrivACSeen) in the console.
    _G.RyPrivACSeen = _G.RyPrivACSeen or {}
    local _killed = 0
    _G.RyPrivStripKills = function() return _killed end

    local function strip()
        local c = LP.Character
        local h = c and c:FindFirstChild("HumanoidRootPart")
        if not h or not getconnections or not _getinfo then return end
        local needles = _acNeedles()
        for _, s in ipairs({ "CFrame", "Position" }) do
            pcall(function()
                for _, cn in ipairs(getconnections(h:GetPropertyChangedSignal(s))) do
                    if cn.Function and cn.Enabled then
                        local ok, info = pcall(_getinfo, cn.Function)
                        local raw = ok and info and tostring(info.source) or nil
                        if raw then
                            _G.RyPrivACSeen[raw] = (_G.RyPrivACSeen[raw] or 0) + 1
                            local low = raw:lower()
                            for _, needle in ipairs(needles) do
                                if low:find(needle, 1, true) then
                                    if pcall(function() cn:Disable() end) then
                                        _killed = _killed + 1
                                        if not _STRIP_OK then
                                            _STRIP_OK = true
                                            _G.RyPrivStealSay("stripped AC watcher: " .. raw)
                                        end
                                    end
                                    break
                                end
                            end
                        end
                    end
                end
            end)
        end
    end
    _G.RyPrivStripAC = strip
    task.spawn(function() _G.RyPrivBootWait() while true do pcall(strip) task.wait(3) end end) 
    LP.CharacterAdded:Connect(function() task.wait(0.3) pcall(strip) end)
end)()

;(function()
    local wsConn

    -- Requested speed is clamped here, not at 32 like the old CFrame engine.
    -- That ceiling existed because the old mechanism fell apart past it; this
    -- one is limited by what the server tolerates, which the backoff below
    -- discovers on its own.
    local WS_MIN, WS_MAX = 5, 64

    -- How hard the server has been pushing back lately: 1 is the full requested
    -- speed, 0.35 the floor. Same idea as velMoveThrough's lagback handling,
    -- which is why the flight engine can hold 300+ studs/s without falling over.
    local _scale, _lastPos, _lastDir, _lagAt = 1, nil, nil, 0
    -- _trim closes the loop on what the world actually gives us while carrying;
    -- _lastAt is what makes measuring that possible at all.
    local _trim, _lastAt = 1, 0
    -- _wsBase is the game's own WalkSpeed before we ever touched it, so it can
    -- be handed back. _wsSet is the last value we wrote.
    local _wsBase, _wsSet = nil, nil
    local function _wsRestore()
        if _wsBase == nil then return end
        local ch = LP.Character
        local h = ch and ch:FindFirstChildOfClass("Humanoid")
        if h then pcall(function() h.WalkSpeed = _wsBase end) end
        _wsBase, _wsSet = nil, nil
    end
    local function _wsReset()
        _scale, _lastPos, _lastDir, _lagAt = 1, nil, nil, 0
        _trim, _lastAt = 1, 0
        _wsSet = nil
    end
    -- read this to see what it settled on
    _G.RyPrivWalkSpeedScale = function() return _scale end

    _G.RyPrivSetWalkSpeed = function(enabled)
        _G.RyPrivWalkSpeedOn = enabled == true
        if wsConn then pcall(function() wsConn:Disconnect() end) wsConn = nil end
        _wsRestore()
        _wsReset()
        if not _G.RyPrivWalkSpeedOn then return end

        wsConn = RunService.Heartbeat:Connect(function()
            if not _G.RyPrivWalkSpeedOn then return end
            pcall(function()
                -- Unchanged gate: this is the "outrun them while carrying one"
                -- feature. RyPrivWalkSpeedAlways = true lifts it.
                if _G.RyPrivWalkSpeedAlways ~= true
                    and LP:GetAttribute("Stealing") ~= true then
                    _lastPos, _lastDir = nil, nil
                    return
                end

                local char = LP.Character
                local hum = char and char:FindFirstChildOfClass("Humanoid")
                if not char or not hum or hum.Health <= 0 then return end

                -- Invisible creates a visible clone as Character.PrimaryPart while
                -- the real HumanoidRootPart is temporarily moved to the camera.
                -- Always drive the active body root, never a stale HRP reference.
                local root = char.PrimaryPart or char:FindFirstChild("HumanoidRootPart")
                if not root or not root:IsA("BasePart") or not root.Parent then return end

                local move = hum.MoveDirection
                if move.Magnitude <= 0.001 then
                    -- not steering: hand motion, and the humanoid's own
                    -- WalkSpeed, entirely back
                    _lastPos, _lastDir = nil, nil
                    _wsRestore()
                    return
                end
                local dir = Vector3.new(move.X, 0, move.Z)
                if dir.Magnitude <= 0.001 then return end
                dir = dir.Unit

                local now = os.clock()
                local pos = root.Position

                -- Lagback: did we travel AGAINST our own push since the last
                -- frame? Only the server rewinding us does that, so ease off.
                -- What the world actually gave us since the last frame. This is
                -- the number the server sees, and it is not the number we asked
                -- for: the humanoid's own mover keeps pushing alongside the
                -- velocity we set, so measured runs above want.
                local measured = 0
                if _lastPos and _lastDir then
                    local moved = pos - _lastPos
                    moved = Vector3.new(moved.X, 0, moved.Z)
                    local gap = now - _lastAt
                    if gap > 0.0001 then measured = moved.Magnitude / gap end
                    if moved:Dot(_lastDir) < -(tonumber(_G.RyPrivWalkLagStuds) or 0.75) then
                        _scale = math.max(_scale * 0.7, 0.35)
                        _lagAt = now
                    end
                end
                -- nothing has pushed back for a while: creep back up
                if _scale < 1 and now - _lagAt > (tonumber(_G.RyPrivWalkRecoverGap) or 1.5) then
                    _scale = math.min(_scale * 1.05, 1)
                end

                local want = math.clamp(tonumber(_G.RyPrivWalkSpeed) or 29, WS_MIN, WS_MAX) * _scale

                -- CARRY CAP.
                --
                -- Going too fast while carrying does not slow you down, it
                -- costs you the brainrot: three logged runs had Stealing,
                -- StealingPlayer and StealingIndex cleared together mid-run,
                -- with no lagback and no ragdoll, every one of them asking
                -- 26-28 against the ~23 the server tolerates. Nothing about the
                -- movement looked wrong in those logs because nothing about the
                -- movement WAS wrong -- it was confiscation, not correction.
                --
                -- The cap is enforced on measured speed rather than the request
                -- because the open loop overshoots: the last run asked 28.0 and
                -- measured 31.7, so capping the request alone would still have
                -- gone over. _trim pulls the request down until what the world
                -- gives back is under the cap, then eases off again.
                --
                -- RyPrivCarryCap = math.huge disables this and hands the
                -- brainrot back to the server.
                -- Make the replicated WalkSpeed agree with what we are actually
                -- doing. This hub has never written this property -- it only
                -- read it -- so every run so far had the humanoid claiming one
                -- speed while the character moved at another, which is the
                -- inconsistency a carry check looks for. A small pad over the
                -- target keeps the claim comfortably above the motion.
                if _G.RyPrivMatchWalkSpeed ~= false then
                    local cur = tonumber(hum.WalkSpeed) or 0
                    if _wsBase == nil then _wsBase = cur end
                    local target = want + (tonumber(_G.RyPrivWalkSpeedPad) or 2)
                    if math.abs(cur - target) > 0.5 then
                        pcall(function() hum.WalkSpeed = target end)
                        _wsSet = target
                    end
                end

                if LP:GetAttribute("Stealing") == true then
                    -- 23 is the speed a carry is known to survive; above it the
                    -- server confiscates the brainrot rather than slowing you
                    -- down, which is why nothing about the movement looks wrong
                    -- when it happens. Raise RyPrivCarryCap to trade brainrots
                    -- for speed.
                    local cap = tonumber(_G.RyPrivCarryCap) or 23
                    if measured > cap then
                        _trim = math.max(_trim * 0.90, 0.40)
                    elseif measured > 0 and measured < cap - 2 and _trim < 1 then
                        _trim = math.min(_trim * 1.02, 1)
                    end
                    if want > cap then want = cap end
                    want = want * _trim
                elseif _trim ~= 1 then
                    _trim = 1
                end

                -- Horizontal only. Leaving Y as the engine just computed it
                -- keeps gravity, falling and jumping behaving normally --
                -- overwriting it is what makes velocity speed hacks float.
                local v = root.AssemblyLinearVelocity
                root.AssemblyLinearVelocity = Vector3.new(dir.X * want, v.Y, dir.Z * want)

                _lastPos, _lastDir, _lastAt = pos, dir, now

                -- Keep the real root with the visible one during Invisible so a
                -- carried pet does not stay behind at the old position.
                if _G.RyPrivInvisActive == true then
                    local real = char:FindFirstChild("HumanoidRootPart")
                    if real and real ~= root and real:IsA("BasePart") and real.Parent then
                        real.CFrame = root.CFrame
                        real.AssemblyLinearVelocity = root.AssemblyLinearVelocity
                    end
                end
            end)
        end)
    end

    -- a fresh character has none of the old one's backoff history
    LP.CharacterAdded:Connect(function() _wsBase, _wsSet = nil, nil _wsReset() end)

    if _G.RyPrivWalkSpeedOn == nil then _G.RyPrivWalkSpeedOn = true end
    if _G.RyPrivWalkSpeedOn == true then
        task.defer(function() pcall(_G.RyPrivSetWalkSpeed, true) end)
    end
end)()

;(function()
    local conn
    _G.RyPrivSetCarpetSpeed = function(enabled)
        _G.RyPrivCarpetSpeed = enabled and true or false
        if conn then conn:Disconnect() conn = nil end
        if not _G.RyPrivCarpetSpeed then return end
        task.spawn(function() pcall(equipCarpet) end)
        conn = RunService.Heartbeat:Connect(function()
            if LP:GetAttribute("Stealing") == true then
                _G.RyPrivSetCarpetSpeed(false)
                return
            end
            local c = LP.Character
            local hum = c and c:FindFirstChildOfClass("Humanoid")
            local part = c and (c:FindFirstChild("UpperTorso")
                or c:FindFirstChild("Torso")
                or c:FindFirstChild("HumanoidRootPart"))
            if not hum or not part then return end
            if not _G.RyPrivCarpetEngaging() then pcall(equipCarpet) end
            local spd = math.clamp(tonumber(_G.RyPrivCarpetSpeedValue) or 140, 20, 400)
            local md, keepY = hum.MoveDirection, part.Velocity.Y
            if md.Magnitude > 0 then
                part.Velocity = Vector3.new(md.X * spd, keepY, md.Z * spd)
            else
                part.Velocity = Vector3.new(0, keepY, 0)
            end
        end)
    end

    UIS.InputBegan:Connect(function(i, g)
        if g or i.UserInputType ~= Enum.UserInputType.Keyboard then return end
        if i.KeyCode.Name ~= (_G.RyPrivCarpetSpeedKeyName or "Q") then return end
        if LP:GetAttribute("Stealing") == true then return end
        local on = not (_G.RyPrivCarpetSpeed == true)
        _G.RyPrivSetCarpetSpeed(on)
        if on then task.spawn(function() pcall(equipCarpet) end) end
    end)
end)()

;(function()
    local FOLDERS = { "Base", "PlotSign", "FriendPanel", "Cash", "Laser",
        "Decorations", "Skin", "Unlock", "Purchases" }
    local orig = setmetatable({}, { __mode = "k" })
    local conns, gen = {}, 0

    local function paint(o, a)
        if not o:IsA("BasePart") then return end
        if orig[o] == nil then orig[o] = (o.Transparency == a) and 0 or o.Transparency end
        local base = orig[o]
        if base >= 1 then return end
        local want = base + (1 - base) * a
        if math.abs(o.Transparency - want) > 0.01 then o.Transparency = want end
    end

    local function calm()
        while _G.RyPrivStealHold do task.wait(0.15) end
    end

    local function track(root, a, id)
        if not root or id ~= gen then return end
        paint(root, a)
        local n = 0
        for _, d in ipairs(root:GetDescendants()) do
            if id ~= gen then return end
            paint(d, a)
            n = n + 1
            if n % 250 == 0 then task.wait() end
        end
        conns[#conns + 1] = root.DescendantAdded:Connect(function(d)
            if id == gen then paint(d, a) end
        end)
    end

    local function doPlot(plot, a, id)
        if not plot or id ~= gen then return end
        for _, fname in ipairs(FOLDERS) do
            if id ~= gen then return end
            track(plot:FindFirstChild(fname), a, id)
        end
        if id ~= gen then return end
        conns[#conns + 1] = plot.ChildAdded:Connect(function(c)
            if id ~= gen then return end
            for _, fname in ipairs(FOLDERS) do
                if c.Name == fname then track(c, a, id) break end
            end
        end)
        local pods = plot:FindFirstChild("AnimalPodiums")
        if not pods then return end
        local function pod(pd)
            for _, c in ipairs(pd:GetChildren()) do
                if c.Name == "Claim" then track(c, a, id)
                elseif c.Name == "Base" then track(c:FindFirstChild("Decorations"), a, id) end
            end
        end
        for _, pd in ipairs(pods:GetChildren()) do pod(pd) end
        conns[#conns + 1] = pods.ChildAdded:Connect(function(pd)
            if id ~= gen then return end
            task.wait(0.1)
            if id == gen then pod(pd) end
        end)
    end

    local function stop()
        for _, c in ipairs(conns) do pcall(function() c:Disconnect() end) end
        conns, gen = {}, gen + 1
    end

    _G.RyPrivEnableXray = function()
        stop()
        local id = gen
        local a = math.clamp(tonumber(_G.RyPrivXrayAlpha) or 0.9, 0, 1)
        task.spawn(function()
            while id == gen and not workspace:FindFirstChild("Plots") do task.wait(0.5) end
            local plots = workspace:FindFirstChild("Plots")
            if id ~= gen or not plots then return end
            calm()
            for _, p in ipairs(plots:GetChildren()) do
                if id ~= gen then return end
                pcall(doPlot, p, a, id)
                task.wait()
                calm()
            end
            conns[#conns + 1] = plots.ChildAdded:Connect(function(p)
                if id ~= gen then return end
                task.wait(0.2)
                pcall(doPlot, p, a, id)
            end)
        end)
    end

    _G.RyPrivDisableXray = function()
        stop()
        local snap = orig
        orig = setmetatable({}, { __mode = "k" })
        for o, t in pairs(snap) do
            pcall(function() if o:IsA("BasePart") then o.Transparency = t end end)
        end
    end

    _G.RyPrivToggleXray = function()
        _G.RyPrivXray = not (_G.RyPrivXray ~= false)
        if _G.RyPrivXray then _G.RyPrivEnableXray() else _G.RyPrivDisableXray() end
        return _G.RyPrivXray
    end

    task.spawn(function()
        _G.RyPrivBootWait()
        task.wait(tonumber(_G.RyPrivXrayDelay) or 1)
        if _G.RyPrivXray ~= false then _G.RyPrivEnableXray() end
    end)
end)()

;(function()
    local function psCode(link)
        link = tostring(link or ""):match("^%s*(.-)%s*$")
        if link == "" then return nil end
        if not link:find("://", 1, true) then return link end
        return link:match("[?&]privateServerLinkCode=([^&]+)")
            or link:match("[?&]linkCode=([^&]+)")
            or link:match("[?&]code=([^&]+)")
    end

    local function kickOut()
        if _G.RyPrivKickToPS == true then
            local code = psCode(_G.RyPrivPrivateServerLink)
            if code and code ~= "" then
                local ok = pcall(function()
                    game:GetService("ExperienceService"):LaunchExperience({
                        placeId = tonumber(_G.RyPrivPrivateServerPlaceId) or game.PlaceId,
                        linkCode = code,
                    })
                end)
                if ok then return end
            end
        end
        if pcall(function() game:Shutdown() end) then return end
        pcall(function() LP:Kick("") end)
    end
    _G.RyPrivKickOut = kickOut

    task.spawn(function()
        local PG2 = LP:FindFirstChildOfClass("PlayerGui") or LP:WaitForChild("PlayerGui", 10)
        if not PG2 then return end
        local hooked = setmetatable({}, { __mode = "k" })
        local function hit(t)
            return type(t) == "string" and t:lower():find("you stole", 1, true) ~= nil
        end
        local function isText(o)
            return o:IsA("TextLabel") or o:IsA("TextButton") or o:IsA("TextBox")
        end
        local function watch(o)
            if hooked[o] then return end
            hooked[o] = true
            if _G.RyPrivAutoKickOnSteal == true and hit(o.Text) then kickOut() return end
            o:GetPropertyChangedSignal("Text"):Connect(function()
                if _G.RyPrivAutoKickOnSteal == true and hit(o.Text) then kickOut() end
            end)
        end
        local function root(g)
            g.DescendantAdded:Connect(function(d) if isText(d) then watch(d) end end)
            local n = 0
            for _, d in ipairs(g:GetDescendants()) do
                n = n + 1
                if n % 200 == 0 then task.wait() end
                if isText(d) then watch(d) end
            end
        end

        while _G.RyPrivAutoKickOnSteal ~= true do task.wait(1) end
        PG2.ChildAdded:Connect(root)
        for _, g in ipairs(PG2:GetChildren()) do root(g) end
    end)
end)()

;(function()
    local seen, snd, lastAt = {}, nil, 0

    _G.RyPrivAlertCheck = function(pets)
        if _G.RyPrivPriAlert ~= true then
            if next(seen) then seen = {} end
            return
        end
        local minGen = tonumber(_G.RyPrivAlertMinGen) or 80e6
        local live, fresh = {}, false
        for _, p in ipairs(pets) do
            if p._pri or (tonumber(p.mps) or 0) >= minGen then
                local k = tostring(p.plot) .. "_" .. tostring(p.slot)
                live[k] = true
                if not seen[k] then fresh = true end
            end
        end
        seen = live
        if not fresh or os.clock() - lastAt < 3 then return end
        lastAt = os.clock()
        pcall(function()
            local id = tostring(_G.RyPrivAlertSound or "111786441593851"):match("%d+")
            if not id then return end
            if not (snd and snd.Parent) then
                snd = Instance.new("Sound")
                snd.Name = "RyPrivAlert"
                snd.Parent = game:GetService("SoundService")
            end
            snd.SoundId = "rbxassetid://" .. id
            snd.Volume = math.clamp(tonumber(_G.RyPrivAlertVolume) or 1, 0, 10)
            snd:Play()
        end)
    end

    task.spawn(function()
        while true do
            task.wait(0.5)
            if _G.RyPrivPriAlert == true then
                local ok, pets = pcall(scanAllPets)
                if ok and type(pets) == "table" then pcall(_G.RyPrivAlertCheck, pets) end
            end
        end
    end)
end)()



pcall(function() game:GetService("Players").RespawnTime = 0 end)  

do
    local RunService = game:GetService("RunService")
    local Players    = game:GetService("Players")
    local Workspace  = game:GetService("Workspace")
    local _wfConns, _wfActive = {}, false
    local function stopWalkFling()
        _wfActive = false
        for _, c in ipairs(_wfConns) do
            if typeof(c) == "RBXScriptConnection" then pcall(function() c:Disconnect() end) end
        end
        _wfConns = {}
    end
    local function startWalkFling()
        _wfActive = true
        local ch = LP.Character
        if not ch then return end
        local rr = ch:FindFirstChild("HumanoidRootPart")
        for _, o in pairs(Workspace.CurrentCamera:GetChildren()) do
            if o.Name == "HumanoidRootPart" then rr = o break end
        end
        if not rr then return end
        table.insert(_wfConns, RunService.Stepped:Connect(function()
            if not _wfActive then return end
            for _, p in ipairs(Players:GetPlayers()) do
                if p ~= LP and p.Character then
                    for _, pt in ipairs(p.Character:GetChildren()) do
                        if pt:IsA("BasePart") then pt.CanCollide = false end
                    end
                end
            end
        end))
        local co = coroutine.create(function()
            if _G.invisibleStealEnabled then rr.CFrame = rr.CFrame * CFrame.new(0, 3, 0) end
            while _wfActive do
                RunService.Heartbeat:Wait()
                if not rr or not rr.Parent then break end
                local v = rr.Velocity
                rr.Velocity = v * 10000 + Vector3.new(0, 10000, 0)
                RunService.RenderStepped:Wait()
                if rr then rr.Velocity = v end
                RunService.Stepped:Wait()
                if rr then rr.Velocity = v + Vector3.new(0, 0.1, 0) end
            end
        end)
        coroutine.resume(co)
        table.insert(_wfConns, co)
    end
    _G.DropBrainrot = function()
        if _wfActive then return end
        startWalkFling()
        task.delay(0.4, stopWalkFling)
    end
    _G.stopWalkFling = stopWalkFling
end


local CARPET_NAMES = { "Flying Carpet", "Waverider", "Santa's Sleigh", "Witch's Broom", "Cupid's Wings" }
_G.__ryPrivResetBusy = false
_G.RyPrivResetCooldown = tonumber(_G.RyPrivResetCooldown) or 2.5
_G.RyPrivResetFlingTime = tonumber(_G.RyPrivResetFlingTime) or 5
_G.RyPrivInstaReset = function()
    local _now = os.clock()
    if _G.__ryPrivResetBusy
        and (_now - (tonumber(_G.__ryPrivResetAt) or 0)) < (tonumber(_G.RyPrivResetCooldown) or 2.5) then
        return
    end
    _G.__ryPrivResetBusy = true
    _G.__ryPrivResetAt = _now
    task.spawn(function()
        local RunService = game:GetService("RunService")
        local _prevAntiDie = _G.AntiDieDisabled
        local _origChar = LP.Character
        _G.AntiDieDisabled = true
        -- Everything that fights a death has to let go first. Mid-steal that is
        -- the steal hold (keeps health topped up), a running TP (keeps setting
        -- velocity) and invis steal (parks the real root in the camera and pins
        -- Health to MaxHealth every frame -- the old reset could never win that).
        _G.RyPrivStealHold = false
        _G.RyPrivTPStop = true
        if _G.RyPrivDisarmSteal then pcall(_G.RyPrivDisarmSteal) end
        if _G.RyPrivInvisStop then pcall(_G.RyPrivInvisStop) end

        local _restored, _holding = false, true
        local function _restore()
            if _restored then return end
            _restored = true
            _holding = false
            _G.AntiDieDisabled = _prevAntiDie
            _G.RyPrivTPStop = false
            _G.__ryPrivResetBusy = false
        end
        local _conn
        _conn = LP.CharacterAdded:Connect(function(newChar)
            if _conn then _conn:Disconnect(); _conn = nil end
            task.defer(function()
                pcall(function() newChar:WaitForChild("Humanoid", 12) end)
                RunService.Heartbeat:Wait()
                _restore()
            end)
        end)
        task.delay(8, function() if _conn then _conn:Disconnect(); _conn = nil end _restore() end)

        local char = _origChar
        if not char then _restore() return end
        local _isCarpet = {}
        for _, n in ipairs(CARPET_NAMES) do _isCarpet[n] = true end
        pcall(equipCarpet)
        task.spawn(function()
            while _holding and LP.Character == _origChar do
                pcall(equipCarpet)
                RunService.Heartbeat:Wait()
            end
        end)
        local bp = LP:FindFirstChild("Backpack")
        if bp then
            for _, ch in ipairs(char:GetChildren()) do
                if ch:IsA("Tool") and not _isCarpet[ch.Name] then pcall(function() ch.Parent = bp end) end
            end
        end

        -- anti-die / invis turned the Dead state off; death is impossible until
        -- it is back on
        local function _unharden(h)
            pcall(function() h:SetStateEnabled(Enum.HumanoidStateType.Dead, true) end)
            pcall(function() h.RequiresNeck = true end)
            pcall(function() h.BreakJointsOnDeath = true end)
        end
        local function _root()
            local r = char:FindFirstChild("HumanoidRootPart")
            if r then return r end
            local cam = workspace.CurrentCamera
            return cam and cam:FindFirstChild("HumanoidRootPart")
        end
        local _voidY = (tonumber(workspace.FallenPartsDestroyHeight) or -500) - 100
        local _t0 = os.clock()
        while os.clock() - _t0 < (tonumber(_G.RyPrivResetFlingTime) or 5) do
            if LP.Character ~= _origChar or not char.Parent then break end
            local h = char:FindFirstChildOfClass("Humanoid")
            if not h then break end
            _unharden(h)
            -- 1) client-owned humanoid: health 0 + Dead state replicates
            pcall(function() h.Health = 0 end)
            pcall(function() h:ChangeState(Enum.HumanoidStateType.Dead) end)
            -- 2) no neck = dead (RequiresNeck is back on)
            pcall(function()
                for _, d in ipairs(char:GetDescendants()) do
                    if d:IsA("Motor6D") and d.Name == "Neck" then d:Destroy() end
                end
            end)
            -- 3) drop the whole assembly under the kill plane; anything under
            --    FallenPartsDestroyHeight gets deleted and the character dies.
            --    Strip movers first so the steal carry / TP can't hold it up.
            local r = _root()
            if r then
                pcall(function()
                    for _, part in ipairs(char:GetDescendants()) do
                        if part:IsA("BodyPosition") or part:IsA("BodyVelocity") or part:IsA("BodyGyro")
                            or part:IsA("AlignPosition") or part:IsA("LinearVelocity")
                            or part:IsA("AlignOrientation") or part:IsA("VectorForce") then
                            part:Destroy()
                        elseif part:IsA("BasePart") and part.Anchored then
                            part.Anchored = false
                        end
                    end
                end)
                pcall(function()
                    r.CFrame = CFrame.new(r.Position.X, _voidY, r.Position.Z)
                    r.AssemblyLinearVelocity = Vector3.new(0, -5000, 0)
                end)
            end
            RunService.Heartbeat:Wait()
        end
    end)
end

task.spawn(function()
    local _fired = false
    while true do
        if _G.RyPrivAutoClone == true then
            if not _fired
                and LP:GetAttribute("Stealing") ~= true
                and not (_G.RyPrivIsTeleporting and _G.RyPrivIsTeleporting()) then
                _fired = true
                if _G.RyPrivDoClone then pcall(_G.RyPrivDoClone) end
                if _G.RyPrivAutoCloneOneShot ~= false then
                    _G.RyPrivAutoClone = false
                    if _G.RyPrivRepaintAutoClone then pcall(_G.RyPrivRepaintAutoClone) end
                end
            end
        else
            _fired = false
        end
        task.wait(0.15)
    end
end)

-- Was "T", which is MANUAL TP's key -- one press ran a teleport and flipped
-- the auto-TP flag at the same time.
_G.RyPrivAutoTPKeyName = _G.RyPrivAutoTPKeyName or "G"
_G.RyPrivAutoBuyKeyName = _G.RyPrivAutoBuyKeyName or "Y"
_G.RyPrivUtilKeyName = _G.RyPrivUtilKeyName or "U"
_G.RyPrivInvisPanelKeyName = _G.RyPrivInvisPanelKeyName or "J"
_G.RyPrivCarpetSpeedKeyName = _G.RyPrivCarpetSpeedKeyName or "Q"
_G.RyPrivAdminKeyName = _G.RyPrivAdminKeyName or "K"
-- INSTANT STEAL. H is clear of every other default in this hub (T G Y U J Q K
-- V) and rebindable from Settings -> KEYBINDS like the rest of them.
_G.RyPrivInstantKeyName = _G.RyPrivInstantKeyName or "H"

local function _kb(input, name)
    return type(name) == "string" and name ~= "" and input.KeyCode.Name == name
end
UIS.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.UserInputType ~= Enum.UserInputType.Keyboard then return end
    if _G.RyPrivBindListening then return end   

    if _kb(input, _G.RyPrivCloneKeyName) then
        _G.RyPrivAutoClone = not (_G.RyPrivAutoClone == true)
    end
    
    if _kb(input, _G.RyPrivKickKeyName) then
        task.spawn(function()
            _G.RyPrivKickToPS = true
            if _G.RyPrivKickOut then pcall(_G.RyPrivKickOut) end
        end)
    end
    
    if _kb(input, _G.RyPrivStopTPKeyName) then
        _G.RyPrivTPStop = true
        _G.RyPrivAutoTP = false
        if _G.RyPrivRepaintAutoTP then pcall(_G.RyPrivRepaintAutoTP) end
        local ch = LP.Character
        local hrp = ch and ch:FindFirstChild("HumanoidRootPart")
        if hrp then pcall(function() hrp.AssemblyLinearVelocity = Vector3.zero end) end
    end
    
    if _kb(input, _G.RyPrivNearestKey) then
        _G.RyPrivStealMode = (_G.RyPrivStealMode == "nearest") and "priority" or "nearest"
    end

    -- INSTANT STEAL
    if _kb(input, _G.RyPrivInstantKeyName) then
        if _G.RyPrivToggleInstantSteal then pcall(_G.RyPrivToggleInstantSteal) end
    end
    
    if _kb(input, _G.RyPrivDropKeyName) then
        task.spawn(function() if _G.DropBrainrot then pcall(_G.DropBrainrot) end end)
    end
    
    if _kb(input, _G.RyPrivResetKeyName) then
        task.spawn(function()
            if _G.RyPrivInstaReset then
                pcall(_G.RyPrivInstaReset)
            end
        end)
    end

    -- AUTO TELEPORT keybind
    if _kb(input, _G.RyPrivAutoTPKeyName) then
        _G.RyPrivAutoTP = not (_G.RyPrivAutoTP == true)
        if _G.RyPrivRepaintAutoTP then pcall(_G.RyPrivRepaintAutoTP) end
        pcall(_G.RyPrivSaveSettings)
    end

    -- AUTO BUY keybind
    if _kb(input, _G.RyPrivAutoBuyKeyName) then
        _G.RyPrivAutoBuy = not (_G.RyPrivAutoBuy == true)
        if _G.RyPrivRepaintAutoBuy then pcall(_G.RyPrivRepaintAutoBuy) end
        pcall(_G.RyPrivSaveSettings)
    end

    -- UTILITY / INVIS panel show-hide
    if _kb(input, _G.RyPrivUtilKeyName) then
        if _G.RyPrivToggleUtility then pcall(_G.RyPrivToggleUtility) end
    end
    if _kb(input, _G.RyPrivInvisPanelKeyName) then
        if _G.RyPrivToggleInvisPanel then pcall(_G.RyPrivToggleInvisPanel) end
    end
    if _kb(input, _G.RyPrivAdminKeyName) then
        if _G.RyPrivToggleAdmin then pcall(_G.RyPrivToggleAdmin) end
    end
end)


if _G.RyPrivAutoBuy == nil then _G.RyPrivAutoBuy = false end
if _G.RyPrivAutoBuyRange == nil then _G.RyPrivAutoBuyRange = 17 end
if _G.RyPrivAutoBuyHover == nil then _G.RyPrivAutoBuyHover = 9 end
_G.RyPrivRepaintAutoBuy = _G.RyPrivRepaintAutoBuy or function() end


do
    local Workspace = game:GetService("Workspace")
    local lockedPrompt, lockedPart, lockedModel
    local bodyPos, purchaseRemote

    local function resolvePurchaseRemote()
        if purchaseRemote and purchaseRemote.Parent then return purchaseRemote end
        pcall(function()
            local net = RS:FindFirstChild("Packages") and RS.Packages:FindFirstChild("Net")
            if not net then return end
            for _, v in ipairs(net:GetChildren()) do
                local nl = (v.Name or ""):lower()
                for _, kw in ipairs({ "buy", "purchase", "animal", "shop", "acquire", "conveyor" }) do
                    if nl:find(kw) then purchaseRemote = v; return end
                end
            end
        end)
        return purchaseRemote
    end

    local function firePurchase(prompt)
        if not (prompt and prompt.Parent and prompt.Enabled) then return false end
        local fired = false
        pcall(function() prompt.HoldDuration = 0 end)
        if fireproximityprompt then
            for _ = 1, 2 do
                local ok = pcall(function() fireproximityprompt(prompt) end)
                if ok then fired = true end
                task.wait()
                if not (prompt.Parent and prompt.Enabled) then break end
            end
        end
        -- Only use a discovered purchase remote when the game exposes one;
        -- do not assume an unrelated RemoteEvent's argument shape.
        local r = resolvePurchaseRemote()
        if r and prompt.Parent then
            pcall(function()
                if r:IsA("RemoteFunction") then
                    local ok = pcall(function() r:InvokeServer(prompt.Parent) end)
                    fired = fired or ok
                elseif r:IsA("RemoteEvent") then
                    local ok = pcall(function() r:FireServer(prompt.Parent) end)
                    fired = fired or ok
                end
            end)
        end
        return fired
    end

    local function partAlive()
        return lockedPart and lockedPart.Parent and lockedModel and lockedModel.Parent
    end
    local function promptAlive()
        return lockedPrompt and lockedPrompt.Parent and lockedPrompt.Enabled
    end

    local function hoverPart(char)
        return char:FindFirstChild("UpperTorso") or char:FindFirstChild("Torso") or char:FindFirstChild("HumanoidRootPart")
    end
    local function ensureBodyPos(part)
        if bodyPos and bodyPos.Parent == part then return bodyPos end
        if bodyPos then bodyPos:Destroy() end
        local bp = Instance.new("BodyPosition")
        bp.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
        bp.P = 20000
        bp.D = 1200
        bp.Position = part.Position
        bp.Parent = part
        bodyPos = bp
        return bp
    end
    local function destroyBodyPos()
        if bodyPos then pcall(function() bodyPos:Destroy() end); bodyPos = nil end
    end

    local _promptCache, _promptCacheAt = nil, 0
    local function allPrompts()
        if _promptCache and os.clock() - _promptCacheAt < 5 then return _promptCache end
        local out, n = {}, 0
        for _, obj in ipairs(Workspace:GetDescendants()) do
            n = n + 1
            if n % 3000 == 0 then task.wait() end
            if obj:IsA("ProximityPrompt") then out[#out + 1] = obj end
        end
        _promptCache, _promptCacheAt = out, os.clock()
        return out
    end
    Workspace.DescendantAdded:Connect(function(d)
        if _G.RyPrivAutoBuy ~= true or not _promptCache then return end
        if d:IsA("ProximityPrompt") then _promptCache[#_promptCache + 1] = d end
    end)

    local function scanConveyor()
        local results = {}
        for _, obj in ipairs(allPrompts()) do
            if obj.Parent and obj.Enabled then
                local tx = (obj.ActionText or ""):lower()
                if tx:find("purchase") or tx:find("comprar") or tx:find("buy") then
                    local part = obj.Parent
                    local realPart = (part and part:IsA("Attachment") and part.Parent) or part
                    if realPart and realPart:IsA("BasePart") then
                        local model, cur = nil, realPart
                        for _ = 1, 8 do
                            if cur and cur:IsA("Model") then model = cur; break end
                            cur = cur and cur.Parent
                        end
                        results[#results + 1] = { prompt = obj, part = realPart, model = model }
                    end
                end
            end
        end
        return results
    end

    local _abRag = {
        [Enum.HumanoidStateType.Physics] = true,
        [Enum.HumanoidStateType.Ragdoll] = true,
        [Enum.HumanoidStateType.FallingDown] = true,
    }
    local _abSweep, _abHarden = 0, 0
    RunService.Heartbeat:Connect(function()
        if _G.RyPrivAutoBuy ~= true or not partAlive() then destroyBodyPos(); return end
        local char = LP.Character
        local part = char and hoverPart(char)
        if not part then destroyBodyPos(); return end
        local hum = char:FindFirstChildOfClass("Humanoid")
        local ragged = false
        if hum then
            local now = os.clock()
            if now - _abHarden > 0.5 and not _G.__ryPrivResetBusy then
                _abHarden = now
                pcall(function() hum:SetStateEnabled(Enum.HumanoidStateType.Physics, false) end)
                pcall(function() hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false) end)
                pcall(function() hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false) end)
                pcall(function() hum.BreakJointsOnDeath = false end)
            end
            ragged = _abRag[hum:GetState()] == true
            local et = tonumber(LP:GetAttribute("RagdollEndTime"))
            if et and (et - workspace:GetServerTimeNow()) > 0 then ragged = true end
            if ragged then
                pcall(function() LP:SetAttribute("RagdollEndTime", workspace:GetServerTimeNow()) end)
                pcall(function() hum:ChangeState(Enum.HumanoidStateType.GettingUp) end)
                pcall(function() hum:ChangeState(Enum.HumanoidStateType.Running) end)
                local cam = workspace.CurrentCamera
                if cam and cam.CameraSubject ~= hum then
                    pcall(function() cam.CameraSubject = hum end)
                end
                if now - _abSweep > 0.25 then
                    _abSweep = now
                    for _, o in ipairs(char:GetDescendants()) do
                        if o:IsA("BallSocketConstraint") or o.Name == "RagdollAttachment" then
                            pcall(function() o:Destroy() end)
                        end
                    end
                end
            end
        end
        local hover = tonumber(_G.RyPrivAutoBuyHover) or 9
        if _G.__ryPrivResetBusy then return end
        local goal = lockedPart.Position + Vector3.new(0, hover, 0)
        if ragged then
            pcall(function() part.AssemblyAngularVelocity = Vector3.zero end)
        end
        local bp = ensureBodyPos(part)
        bp.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
        bp.P = 20000
        bp.D = 1000
        bp.Position = goal
        if (part.Position - goal).Magnitude > (tonumber(_G.RyPrivAutoBuySnap) or 60) then
            pcall(function()
                part.CFrame = CFrame.new(goal)
                part.AssemblyLinearVelocity = Vector3.zero
                part.AssemblyAngularVelocity = Vector3.zero
            end)
        end
    end)

    RunService.Heartbeat:Connect(function()
        if _G.RyPrivAutoBuy ~= true or not partAlive() or not promptAlive() then return end
        firePurchase(lockedPrompt)
    end)

    task.spawn(function()
        while true do
            task.wait(0.1)
            if _G.RyPrivAutoBuy ~= true then
                lockedPrompt, lockedPart, lockedModel = nil, nil, nil
                destroyBodyPos()
            elseif lockedPart or lockedModel then
                if not partAlive() then lockedPrompt, lockedPart, lockedModel = nil, nil, nil end
            else
                local ok, found = pcall(scanConveyor)
                local list = (ok and found) or {}
                local char = LP.Character
                local hrp = char and char:FindFirstChild("HumanoidRootPart")
                if hrp then
                    local best, bestDist = nil, math.huge
                    local radius = tonumber(_G.RyPrivAutoBuyRange) or 17
                    for _, e in ipairs(list) do
                        if e.prompt and e.prompt.Parent and e.prompt.Enabled and e.part and e.part.Parent then
                            local d = (hrp.Position - e.part.Position).Magnitude
                            if d <= radius and d < bestDist then bestDist = d; best = e end
                        end
                    end
                    if best then
                        lockedPrompt, lockedPart, lockedModel = best.prompt, best.part, best.model or best.part.Parent
                        pcall(function() best.prompt.HoldDuration = 0 end)
                        resolvePurchaseRemote()
                        firePurchase(best.prompt)
                    end
                end
            end
        end
    end)
end

if _G.RyPrivFaceAway == nil then _G.RyPrivFaceAway = false end
if _G.RyPrivFaceAwayNearest == nil then _G.RyPrivFaceAwayNearest = false end

do
    local Workspace = game:GetService("Workspace")
    local function getRoot(pl)
        local c = pl and pl.Character
        return c and c:FindFirstChild("HumanoidRootPart")
    end
    local function findNearest(myRoot)
        local best, bestD = nil, math.huge
        for _, pl in ipairs(Players:GetPlayers()) do
            if pl ~= LP then
                local r = getRoot(pl)
                if r then
                    local d = (r.Position - myRoot.Position).Magnitude
                    if d < bestD then best, bestD = pl, d end
                end
            end
        end
        return best
    end
    local function getPlotAtPosition(pos)
        local plots = Workspace:FindFirstChild("Plots")
        if not plots then return nil end
        local best, bestD = nil, math.huge
        for _, plot in ipairs(plots:GetChildren()) do
            local pp
            if plot:IsA("Model") then
                pp = (plot.PrimaryPart and plot.PrimaryPart.Position) or plot:GetPivot().Position
            else
                pp = plot.Position
            end
            if pp then
                local dx, dz = pos.X - pp.X, pos.Z - pp.Z
                local d = math.sqrt(dx * dx + dz * dz)
                if d < bestD then bestD, best = d, plot end
            end
        end
        return (best and bestD < 72) and best or nil
    end
    local function getPlotOwner(plot)
        if not plot then return nil end
        local sign = plot:FindFirstChild("PlotSign")
        local lbl = sign
            and sign:FindFirstChild("SurfaceGui")
            and sign.SurfaceGui:FindFirstChild("Frame")
            and sign.SurfaceGui.Frame:FindFirstChild("TextLabel")
        if lbl then
            local nick = (lbl.Text and lbl.Text:match("^(.-)'")) or lbl.Text
            if nick and nick ~= "" then
                for _, pl in ipairs(Players:GetPlayers()) do
                    if pl.DisplayName == nick or pl.Name == nick then return pl end
                end
            end
        end
        return nil
    end
    local _ownerCache, _ownerAt = nil, 0
    local function resolveTarget(myRoot)
        if _G.RyPrivFaceAwayNearest == true then
            return findNearest(myRoot)
        end
        if os.clock() - _ownerAt > 0.5 then
            _ownerAt = os.clock()
            local owner = getPlotOwner(getPlotAtPosition(myRoot.Position))
            _ownerCache = (owner ~= LP) and owner or nil
        end
        return _ownerCache
    end
    local _faceWasOn, _stealSince = false, nil
    local function faceActive()
        if _G.RyPrivFaceAway ~= true then
            _stealSince = nil
            return false
        end
        if LP:GetAttribute("Stealing") ~= true then
            _stealSince = nil
            return false
        end
        if not _stealSince then
            _stealSince = os.clock()
            return false
        end
        return (os.clock() - _stealSince) >= (tonumber(_G.RyPrivFaceAwayDelay) or 2)
    end
    local function faceAwayStep()
        local char = LP.Character
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        if not faceActive() then
            if _faceWasOn then
                _faceWasOn = false
                if hum then pcall(function() hum.AutoRotate = true end) end
            end
            return
        end
        _faceWasOn = true
        local myRoot = getRoot(LP)
        if not myRoot then return end
        if hum and hum.AutoRotate then pcall(function() hum.AutoRotate = false end) end
        local tRoot = getRoot(resolveTarget(myRoot))
        if not tRoot then return end
        local flat = Vector3.new(
            tRoot.Position.X - myRoot.Position.X,
            0,
            tRoot.Position.Z - myRoot.Position.Z
        )
        if flat.Magnitude < 0.05 then return end
        myRoot.CFrame = CFrame.lookAt(myRoot.Position, myRoot.Position + Vector3.new(-flat.Z, 0, flat.X).Unit)
        local av = myRoot.AssemblyAngularVelocity
        if av.Y ~= 0 then
            myRoot.AssemblyAngularVelocity = Vector3.new(av.X, 0, av.Z)
        end
    end
    RunService.RenderStepped:Connect(faceAwayStep)
    RunService.Heartbeat:Connect(faceAwayStep)
end

-- ============================================================
-- SETTINGS  (replaces the old RYMOGS PRIV EXTRAS panel)
-- ============================================================
-- One panel, three tabs, opened with Ctrl.
--
--   UI        show/hide each of the hub's layers, plus the transparency dial
--   KEYBINDS  every configurable key, including the ones that used to be
--             hardcoded (manual TP, clone now, carpet speed)
--   FEATURES  the feature list plus the TP controls: MANUAL TP, AUTO TP,
--             TP SAFE MODE and the movement sliders
-- ============================================================
do
    local TweenService = game:GetService("TweenService")
    local C = {
        bg    = Color3.fromRGB(30, 32, 36),
        bg2   = Color3.fromRGB(40, 43, 48),
        card  = Color3.fromRGB(52, 56, 62),
        track = Color3.fromRGB(64, 69, 77),
        line  = Color3.fromRGB(92, 99, 110),
        acc   = Color3.fromRGB(96, 178, 235),
        acc2  = Color3.fromRGB(155, 212, 248),
        txt   = Color3.fromRGB(240, 243, 247),
        dim   = Color3.fromRGB(158, 166, 178),
        bad   = Color3.fromRGB(216, 84, 92),
    }
    local FB, FR = Enum.Font.GothamBold, Enum.Font.Gotham
    local EASE = TweenInfo.new(0.16, Enum.EasingStyle.Quart, Enum.EasingDirection.Out)
    local function A() return math.clamp(tonumber(_G.RyPrivAlpha) or 0.12, 0, 0.9) end
    local function tw(o, p) TweenService:Create(o, EASE, p):Play() end
    local function mk(cls, parent, props)
        local o = Instance.new(cls)
        for k, v in pairs(props or {}) do o[k] = v end
        o.Parent = parent
        return o
    end
    local function corner(o, r) mk("UICorner", o, { CornerRadius = UDim.new(0, 0) }) return o end

    local _alphaPaint, _repaints = {}, {}
    local _ord = 0
    local function nextOrd() _ord = _ord + 1 return _ord end
    local function cardAlpha(o)
        local function p() o.BackgroundTransparency = 0.18 + A() * 0.42 end
        p()
        _alphaPaint[#_alphaPaint + 1] = p
        return o
    end

    local host = (gethui and gethui()) or game:GetService("CoreGui")
    pcall(function()
        local old = host:FindFirstChild("RyPrivSettingsUI")
        if old then old:Destroy() end
    end)
    local sg = mk("ScreenGui", nil, {
        Name = "RyPrivSettingsUI", ResetOnSpawn = false, IgnoreGuiInset = true,
        DisplayOrder = 999997, ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
    })
    pcall(function() sg.Parent = host end)
    if not sg.Parent then sg.Parent = LP:WaitForChild("PlayerGui") end

    -- ── SHELL ────────────────────────────────────────────────────────────
    local panel = corner(mk("Frame", sg, {
        Name = "Root", Active = true, BackgroundColor3 = C.bg,
        BackgroundTransparency = A(), BorderSizePixel = 0,
        Size = UDim2.fromOffset(436, 452), Visible = false,
        Position = UDim2.fromOffset(tonumber(_G._ryPriv_exX) or 300,
            tonumber(_G._ryPriv_exY) or 180),
    }), 9)
    mk("UIStroke", panel, { Thickness = 1, Transparency = 0.35, Color = C.line })
    _alphaPaint[#_alphaPaint + 1] = function() panel.BackgroundTransparency = A() end

    local head = mk("TextButton", panel, {
        Name = "Head", Size = UDim2.new(1, 0, 0, 38), BackgroundTransparency = 1,
        Text = "", AutoButtonColor = false, Active = true,
    })
    corner(mk("Frame", head, {
        Size = UDim2.fromOffset(3, 16), Position = UDim2.new(0, 12, 0.5, -8),
        BackgroundColor3 = C.acc, BorderSizePixel = 0,
    }), 2)
    mk("TextLabel", head, {
        Size = UDim2.new(1, -70, 1, 0), Position = UDim2.new(0, 22, 0, 0),
        BackgroundTransparency = 1, Text = "SETTINGS", Font = FB, TextSize = 13,
        TextColor3 = C.txt, TextXAlignment = Enum.TextXAlignment.Left,
    })
    local closeBtn = corner(mk("TextButton", head, {
        Size = UDim2.fromOffset(22, 22), Position = UDim2.new(1, -32, 0.5, -11),
        BackgroundColor3 = C.card, BackgroundTransparency = 0.3, Text = "x",
        Font = FB, TextSize = 12, TextColor3 = C.dim, AutoButtonColor = false,
    }), 5)
    closeBtn.MouseButton1Click:Connect(function() panel.Visible = false end)

    _G.RyPrivPanelDrag(panel, head, "_ryPriv_exX", "_ryPriv_exY")
    _G.RyPrivApplyPos(panel, "_ryPriv_exX", "_ryPriv_exY", nil)
    _G.RyPrivRegisterPanel(panel, "_ryPriv_exX", "_ryPriv_exY")

    -- ── TABS (left rail) ─────────────────────────────────────────────────
    -- Six tabs do not fit across the top of a 300px panel, and a row of them
    -- shrinks to unreadable labels. A rail down the left side gives each one a
    -- full-width label and room to grow.
    local RAIL_W = 104
    local rail = mk("Frame", panel, {
        Name = "Rail",
        Position = UDim2.fromOffset(10, 44), Size = UDim2.new(0, RAIL_W, 1, -54),
        BackgroundTransparency = 1,
    })
    mk("UIListLayout", rail, {
        FillDirection = Enum.FillDirection.Vertical, Padding = UDim.new(0, 4),
        SortOrder = Enum.SortOrder.LayoutOrder,
    })

    local function mkList()
        local l = mk("ScrollingFrame", panel, {
            Position = UDim2.fromOffset(10 + RAIL_W + 10, 44),
            Size = UDim2.new(1, -(RAIL_W + 34), 1, -54),
            BackgroundTransparency = 1, BorderSizePixel = 0, Visible = false,
            ScrollBarThickness = 2, ScrollBarImageColor3 = C.line,
            ScrollBarImageTransparency = 0.4,
            CanvasSize = UDim2.new(), AutomaticCanvasSize = Enum.AutomaticSize.Y,
        })
        mk("UIListLayout", l, { Padding = UDim.new(0, 5), SortOrder = Enum.SortOrder.LayoutOrder })
        mk("UIPadding", l, { PaddingBottom = UDim.new(0, 10), PaddingRight = UDim.new(0, 6) })
        return l
    end

    local visList  = mkList()
    local movList  = mkList()
    local tpList   = mkList()
    local admList  = mkList()
    local keyList  = mkList()
    local uiList   = mkList()

    local tabs = {}
    local function showTab(which)
        for _, t in ipairs(tabs) do
            local on = (t.list == which)
            t.list.Visible = on
            tw(t.btn, { BackgroundColor3 = on and C.acc or C.card })
            t.btn.TextColor3 = on and C.bg or C.dim
            t.mark.BackgroundTransparency = on and 0 or 1
        end
        _G._ryPriv_exTab = which and tostring(which.Name) or nil
    end
    local function mkTab(label, list, order)
        list.Name = "Tab" .. label
        local b = corner(mk("TextButton", rail, {
            Size = UDim2.new(1, 0, 0, 30), BackgroundColor3 = C.card,
            Text = label, Font = FB, TextSize = 10, TextColor3 = C.dim,
            AutoButtonColor = false, BorderSizePixel = 0, LayoutOrder = order,
            TextXAlignment = Enum.TextXAlignment.Left,
        }), 5)
        mk("UIPadding", b, { PaddingLeft = UDim.new(0, 12) })
        -- accent tick on the selected tab, so the rail reads at a glance
        local mark = corner(mk("Frame", b, {
            Size = UDim2.fromOffset(3, 14), Position = UDim2.new(0, 3, 0.5, -7),
            BackgroundColor3 = C.bg, BorderSizePixel = 0, BackgroundTransparency = 1,
        }), 2)
        tabs[#tabs + 1] = { btn = b, list = list, mark = mark }
        b.MouseButton1Click:Connect(function() showTab(list) end)
        return b
    end
    mkTab("VISUALS",  visList, 1)
    mkTab("MOVEMENT", movList, 2)
    mkTab("TELEPORT", tpList,  3)
    mkTab("ADMIN",    admList, 4)
    mkTab("KEYBINDS", keyList, 5)
    mkTab("UI",       uiList,  6)

    -- ── ROW HELPERS ──────────────────────────────────────────────────────
    local function section(parent, text)
        mk("TextLabel", parent, {
            Size = UDim2.new(1, 0, 0, 16), BackgroundTransparency = 1,
            Font = FB, TextSize = 9, TextColor3 = C.dim, LayoutOrder = nextOrd(),
            TextXAlignment = Enum.TextXAlignment.Left, Text = text:upper(),
        })
    end

    local function toggle(parent, label, get, set)
        local card = cardAlpha(corner(mk("Frame", parent, {
            Size = UDim2.new(1, 0, 0, 30), BackgroundColor3 = C.card,
            BorderSizePixel = 0, LayoutOrder = nextOrd(),
        }), 6))
        local st = mk("UIStroke", card, { Color = C.line, Thickness = 1, Transparency = 0.55 })
        mk("TextLabel", card, {
            Size = UDim2.new(1, -52, 1, 0), Position = UDim2.fromOffset(10, 0),
            BackgroundTransparency = 1, Text = label, Font = FB, TextSize = 11,
            TextColor3 = C.txt, TextXAlignment = Enum.TextXAlignment.Left,
            TextTruncate = Enum.TextTruncate.AtEnd,
        })
        local pill = corner(mk("Frame", card, {
            Size = UDim2.fromOffset(28, 14), Position = UDim2.new(1, -40, 0.5, -7),
            BackgroundColor3 = C.line, BorderSizePixel = 0,
        }), 7)
        local knob = corner(mk("Frame", pill, {
            Size = UDim2.fromOffset(10, 10), Position = UDim2.fromOffset(2, 2),
            BackgroundColor3 = Color3.fromRGB(226, 231, 237), BorderSizePixel = 0,
        }), 5)
        local hit = mk("TextButton", card, {
            Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1, Text = "",
            AutoButtonColor = false, Active = true,
        })
        local last
        local function paint()
            local v = get() and true or false
            if v == last then return end
            last = v
            pill.BackgroundColor3 = v and C.acc or C.line
            knob.Position = v and UDim2.fromOffset(16, 2) or UDim2.fromOffset(2, 2)
            st.Color = v and C.acc or C.line
            st.Transparency = v and 0.3 or 0.55
        end
        hit.MouseButton1Click:Connect(function()
            pcall(set)
            last = nil
            paint()
            pcall(_G.RyPrivSaveSettings)
        end)
        paint()
        _repaints[#_repaints + 1] = paint
        return paint
    end

    local function slider(parent, label, minV, maxV, get, set, step)
        step = step or 1
        local dec = (step < 0.1 and 2) or (step < 1 and 1) or 0
        local card = cardAlpha(corner(mk("Frame", parent, {
            Size = UDim2.new(1, 0, 0, 42), BackgroundColor3 = C.card,
            BorderSizePixel = 0, LayoutOrder = nextOrd(),
        }), 6))
        mk("UIStroke", card, { Color = C.line, Thickness = 1, Transparency = 0.6 })
        mk("TextLabel", card, {
            Size = UDim2.new(0.66, 0, 0, 14), Position = UDim2.fromOffset(10, 5),
            BackgroundTransparency = 1, Text = label, Font = FR, TextSize = 10,
            TextColor3 = C.dim, TextXAlignment = Enum.TextXAlignment.Left,
            TextTruncate = Enum.TextTruncate.AtEnd,
        })
        local valLbl = mk("TextLabel", card, {
            AnchorPoint = Vector2.new(1, 0), Position = UDim2.new(1, -10, 0, 5),
            Size = UDim2.fromOffset(60, 14), BackgroundTransparency = 1,
            Font = FB, TextSize = 10, TextColor3 = C.acc2,
            TextXAlignment = Enum.TextXAlignment.Right, Text = "",
        })
        local trk = corner(mk("TextButton", card, {
            Size = UDim2.new(1, -20, 0, 5), Position = UDim2.fromOffset(10, 26),
            BackgroundColor3 = C.line, BackgroundTransparency = 0.4,
            Text = "", AutoButtonColor = false, BorderSizePixel = 0,
        }), 3)
        local fill = corner(mk("Frame", trk, {
            Size = UDim2.new(0, 0, 1, 0), BackgroundColor3 = C.acc, BorderSizePixel = 0,
        }), 3)
        local function refresh()
            local v = tonumber(get()) or minV
            fill.Size = UDim2.new(
                math.clamp((v - minV) / math.max(maxV - minV, 0.001), 0, 1), 0, 1, 0)
            valLbl.Text = string.format("%." .. dec .. "f", v)
        end
        local sliding = false
        local function applyAt(x)
            local pct = math.clamp((x - trk.AbsolutePosition.X)
                / math.max(trk.AbsoluteSize.X, 1), 0, 1)
            local raw = minV + (maxV - minV) * pct
            set(math.clamp(math.floor(raw / step + 0.5) * step, minV, maxV))
            refresh()
        end
        trk.InputBegan:Connect(function(i)
            if i.UserInputType == Enum.UserInputType.MouseButton1
                or i.UserInputType == Enum.UserInputType.Touch then
                sliding = true
                applyAt(i.Position.X)
            end
        end)
        UIS.InputChanged:Connect(function(i)
            if not sliding then return end
            if i.UserInputType == Enum.UserInputType.MouseMovement
                or i.UserInputType == Enum.UserInputType.Touch then
                applyAt(i.Position.X)
            end
        end)
        UIS.InputEnded:Connect(function(i)
            if i.UserInputType == Enum.UserInputType.MouseButton1
                or i.UserInputType == Enum.UserInputType.Touch then
                if sliding then sliding = false; pcall(_G.RyPrivSaveSettings) end
            end
        end)
        refresh()
        _repaints[#_repaints + 1] = refresh
        return refresh
    end

    local function keybind(parent, label, getName, setName)
        local card = cardAlpha(corner(mk("Frame", parent, {
            Size = UDim2.new(1, 0, 0, 32), BackgroundColor3 = C.card,
            BorderSizePixel = 0, LayoutOrder = nextOrd(),
        }), 6))
        mk("UIStroke", card, { Color = C.line, Thickness = 1, Transparency = 0.6 })
        mk("TextLabel", card, {
            Size = UDim2.new(1, -100, 1, 0), Position = UDim2.fromOffset(10, 0),
            BackgroundTransparency = 1, Text = label, Font = FR, TextSize = 11,
            TextColor3 = C.txt, TextXAlignment = Enum.TextXAlignment.Left,
            TextTruncate = Enum.TextTruncate.AtEnd,
        })
        local btn = corner(mk("TextButton", card, {
            Size = UDim2.fromOffset(80, 22), Position = UDim2.new(1, -90, 0.5, -11),
            BackgroundColor3 = C.track, Text = "", Font = FB, TextSize = 11,
            TextColor3 = C.txt, AutoButtonColor = false, Active = true,
        }), 5)
        local listening, conn = false, nil
        local function repaint()
            if listening then btn.Text = "..." return end
            local k = getName()
            btn.Text = (type(k) == "string" and k ~= "") and k or "NONE"
        end
        local function stop()
            listening = false
            _G.RyPrivBindListening = false
            if conn then conn:Disconnect() conn = nil end
            tw(btn, { BackgroundColor3 = C.track })
            repaint()
        end
        btn.MouseButton1Click:Connect(function()
            if listening then stop() return end
            listening = true
            _G.RyPrivBindListening = true
            tw(btn, { BackgroundColor3 = C.acc })
            repaint()
            conn = UIS.InputBegan:Connect(function(inp)
                if inp.UserInputType ~= Enum.UserInputType.Keyboard then return end
                local kn = inp.KeyCode.Name
                if kn == "Escape" or kn == "Backspace" or kn == "Delete" then
                    setName(nil)
                else
                    setName(kn)
                end
                stop()
                pcall(_G.RyPrivSaveSettings)
            end)
        end)
        -- right click clears
        btn.MouseButton2Click:Connect(function()
            if listening then stop() return end
            setName(nil)
            repaint()
            pcall(_G.RyPrivSaveSettings)
        end)
        repaint()
        _repaints[#_repaints + 1] = repaint
    end

    local function input(parent, label, placeholder, get, set)
        local card = cardAlpha(corner(mk("Frame", parent, {
            Size = UDim2.new(1, 0, 0, 46), BackgroundColor3 = C.card,
            BorderSizePixel = 0, LayoutOrder = nextOrd(),
        }), 6))
        mk("UIStroke", card, { Color = C.line, Thickness = 1, Transparency = 0.6 })
        mk("TextLabel", card, {
            Size = UDim2.new(1, -20, 0, 12), Position = UDim2.fromOffset(10, 5),
            BackgroundTransparency = 1, Text = label, Font = FR, TextSize = 9,
            TextColor3 = C.dim, TextXAlignment = Enum.TextXAlignment.Left,
        })
        local box = corner(mk("TextBox", card, {
            Size = UDim2.new(1, -20, 0, 20), Position = UDim2.fromOffset(10, 20),
            BackgroundColor3 = C.bg2, BackgroundTransparency = 0.15,
            BorderSizePixel = 0, Font = FR, TextSize = 10, TextColor3 = C.txt,
            TextXAlignment = Enum.TextXAlignment.Left, ClearTextOnFocus = false,
            PlaceholderText = placeholder, PlaceholderColor3 = C.dim,
            Text = tostring(get() or ""), TextTruncate = Enum.TextTruncate.AtEnd,
        }), 5)
        mk("UIPadding", box, { PaddingLeft = UDim.new(0, 6), PaddingRight = UDim.new(0, 6) })
        box.FocusLost:Connect(function()
            set(box.Text)
            box.Text = tostring(get() or "")
            pcall(_G.RyPrivSaveSettings)
        end)
    end


    -- ══ TAB: UI ══════════════════════════════════════════════════════════
    -- One switch per hub layer. State lives in _G.RyPrivUIHidden so it rides
    -- along in the settings file, and a slow loop re-applies it because some
    -- layers are built later than this panel and one is rebuilt on demand.
    if type(_G.RyPrivUIHidden) ~= "table" then _G.RyPrivUIHidden = {} end
    local UI_LAYERS = {
        { key = "targets",  label = "Target Panel", gui = "uiListLayer" },
        { key = "header",   label = "Header Bar",   gui = "uiHeaderLayer" },
        { key = "stealbar", label = "Steal Bar",    gui = "hudProgressLayer" },
        { key = "utility",  label = "Utility Panel", gui = "RyPrivUtilityUI" },
        { key = "invis",    label = "Invis Steal",  gui = "RyPrivInvisUI" },
        { key = "autoap",   label = "Auto AP Panel", gui = "RyPrivAutoAPUI" },
        { key = "admin",    label = "Admin Panel",   gui = "RyPrivAdminUI" },
    }
    local function layerGui(name)
        for _, par in ipairs({ host, LP:FindFirstChild("PlayerGui") }) do
            local g = par and par:FindFirstChild(name)
            if g and g:IsA("ScreenGui") then return g end
        end
        return nil
    end
    local function applyLayers()
        for _, d in ipairs(UI_LAYERS) do
            local g = layerGui(d.gui)
            if g then g.Enabled = not (_G.RyPrivUIHidden[d.key] == true) end
        end
    end

    section(uiList, "Show / hide")
    for _, d in ipairs(UI_LAYERS) do
        local key = d.key
        toggle(uiList, d.label,
            function() return _G.RyPrivUIHidden[key] ~= true end,
            function()
                _G.RyPrivUIHidden[key] = not (_G.RyPrivUIHidden[key] == true)
                applyLayers()
            end)
    end

    section(uiList, "Appearance")
    slider(uiList, "UI Transparency", 0, 0.9,
        function() return tonumber(_G.RyPrivAlpha) or 0.12 end,
        function(v)
            _G.RyPrivAlpha = v
            for _, f in ipairs(_alphaPaint) do pcall(f) end
        end, 0.01)

    task.spawn(function()
        while sg.Parent do
            pcall(applyLayers)
            task.wait(1)
        end
    end)

    -- ══ TAB: KEYBINDS ════════════════════════════════════════════════════
    mk("TextLabel", keyList, {
        Size = UDim2.new(1, 0, 0, 28), BackgroundTransparency = 1, LayoutOrder = nextOrd(),
        Font = FR, TextSize = 9, TextColor3 = C.dim,
        TextXAlignment = Enum.TextXAlignment.Left, TextWrapped = true,
        Text = "Click a key to rebind, right click to clear.  Ctrl opens this panel.",
    })

    section(keyList, "Teleport")
    keybind(keyList, "MANUAL TP",     function() return _G._stp_tpKeyName end,       function(k) _G._stp_tpKeyName = k end)
    keybind(keyList, "AUTO TP",       function() return _G.RyPrivAutoTPKeyName end,  function(k) _G.RyPrivAutoTPKeyName = k end)
    keybind(keyList, "STOP TP",       function() return _G.RyPrivStopTPKeyName end,  function(k) _G.RyPrivStopTPKeyName = k end)

    section(keyList, "Steal")
    keybind(keyList, "PRIORITY / NEAREST", function() return _G.RyPrivNearestKey end,  function(k) _G.RyPrivNearestKey = k end)
    keybind(keyList, "INSTANT STEAL",       function() return _G.RyPrivInstantKeyName end, function(k) _G.RyPrivInstantKeyName = k end)
    keybind(keyList, "DROP BRAINROT",      function() return _G.RyPrivDropKeyName end, function(k) _G.RyPrivDropKeyName = k end)
    keybind(keyList, "AUTO BUY",           function() return _G.RyPrivAutoBuyKeyName end, function(k) _G.RyPrivAutoBuyKeyName = k end)

    section(keyList, "Movement")
    keybind(keyList, "CLONE NOW",     function() return _G.RyPrivCloneNowKey end,    function(k) _G.RyPrivCloneNowKey = k end)
    keybind(keyList, "AUTO CLONE",    function() return _G.RyPrivCloneKeyName end,   function(k) _G.RyPrivCloneKeyName = k end)
    keybind(keyList, "CARPET SPEED",  function() return _G.RyPrivCarpetSpeedKeyName end, function(k) _G.RyPrivCarpetSpeedKeyName = k end)
    keybind(keyList, "INSTA RESET",   function() return _G.RyPrivResetKeyName end,   function(k) _G.RyPrivResetKeyName = k end)

    section(keyList, "Panels & server")
    keybind(keyList, "UTILITY PANEL", function() return _G.RyPrivUtilKeyName end,       function(k) _G.RyPrivUtilKeyName = k end)
    keybind(keyList, "INVIS PANEL",   function() return _G.RyPrivInvisPanelKeyName end, function(k) _G.RyPrivInvisPanelKeyName = k end)
    keybind(keyList, "KICK TO PRIVATE SERVER", function() return _G.RyPrivKickKeyName end, function(k) _G.RyPrivKickKeyName = k end)

    -- ══ TAB CONTENT ══════════════════════════════════════════════════════
    -- What used to be one FEATURES list, split across the new tabs. MANUAL TP
    -- and KICK are gone: both are on keybinds and the Utility panel already,
    -- and a mis-click on either in a settings window is expensive.
    section(tpList, "Movement mode")
    -- Two views of one setting, the same shape as Steal Nearest / Priority:
    -- turning one on turns the other off, and they can never both be lit.
    do
        local nPaint, fPaint
        local function pickTP(m)
            _G.RyPrivTPMode = m
            pcall(_G.RyPrivSaveSettings)
            pcall(nPaint) pcall(fPaint)
        end
        nPaint = toggle(tpList, "NORMAL",
            function() return _G.RyPrivTPMode ~= "fling" end,
            function() pickTP("normal") end)
        fPaint = toggle(tpList, "FLING",
            function() return _G.RyPrivTPMode == "fling" end,
            function() pickTP("fling") end)
    end

    section(tpList, "Teleport")
    _G.RyPrivRepaintAutoTP = toggle(tpList, "AUTO TP",
        function() return _G.RyPrivAutoTP == true end,
        function() _G.RyPrivAutoTP = not (_G.RyPrivAutoTP == true) end)
    _G.RyPrivRepaintTPSafe = toggle(tpList, "TP SAFE MODE",
        function() return _G.RyPrivTPSafeMode ~= false end,
        function() _G.RyPrivTPSafeMode = not (_G.RyPrivTPSafeMode ~= false) end)
    for _, s in ipairs({
        { "TP Velocity",           200,  750, "TPVelocity",       5    },
        { "Rise Speed",            100,  250, "RyPrivClimb",      5    },
        { "Go To Brainrot Speed",   80,  600, "RyPrivGoSpeed",    5    },
        { "Landing Delay",        0.05, 0.75, "LandingDelay",     0.05 },
        { "100 Studs Base Speed",   20,  400, "RyPrivCloseSpeed", 5    },
    }) do
        local g = s[4]
        slider(tpList, s[1], s[2], s[3],
            function() return _G[g] end, function(v) _G[g] = v end, s[5])
    end

    section(movList, "Movement")
    toggle(movList, "DISABLE ANIMATIONS",
        function() return _G.RyPrivDisableAnims ~= false end,
        function()
            if _G.RyPrivSetDisableAnims then
                pcall(_G.RyPrivSetDisableAnims, _G.RyPrivDisableAnims == false)
            end
        end)
    -- Arms a clone for the next safe moment. The loop clears it again after it
    -- fires, which is what RyPrivRepaintAutoClone is for -- it was read there
    -- but nothing ever set it, so the flag had no UI at all.
    _G.RyPrivRepaintAutoClone = toggle(movList, "AUTO CLONE",
        function() return _G.RyPrivAutoClone == true end,
        function() _G.RyPrivAutoClone = not (_G.RyPrivAutoClone == true) end)
    toggle(movList, "CARPET SPEED",
        function() return _G.RyPrivCarpetSpeed == true end,
        function() if _G.RyPrivSetCarpetSpeed then pcall(_G.RyPrivSetCarpetSpeed, not (_G.RyPrivCarpetSpeed == true)) end end)
    slider(movList, "Carpet Speed", 20, 400,
        function() return _G.RyPrivCarpetSpeedValue end,
        function(v) _G.RyPrivCarpetSpeedValue = v end, 5)
    toggle(movList, "INF JUMP",
        function() return _G.RyPrivInfJump ~= false end,
        function() _G.RyPrivInfJump = not (_G.RyPrivInfJump ~= false) end)

    section(movList, "Protection")
    toggle(visList, "ANTI FLASH",
        function() return _G.RyPrivAntiFlash ~= false end,
        function() _G.RyPrivAntiFlash = not (_G.RyPrivAntiFlash ~= false) end)
    toggle(movList, "ANTI BEE",
        function() return _G.RyPrivAntiBee ~= false end,
        function() _G.RyPrivAntiBee = not (_G.RyPrivAntiBee ~= false) end)
    toggle(movList, "ANTI DIE",
        function() return _G.AntiDieDisabled ~= true end,
        function() _G.AntiDieDisabled = (_G.AntiDieDisabled ~= true) end)

    section(visList, "ESP")
    -- Each of these drives a setter from the ported ESP backend. They all share
    -- one budgeted pass and one clear, so switching the last one off tears down
    -- every adornment rather than leaving them hanging in the world.
    --
    -- There is no Turret ESP row: it does not work, so it is not offered.
    for _, e in ipairs({
        { "PLAYER ESP",      "RyPlayerESP",     "setPlayerESP"      },
        { "STEALER ESP",     "ESP_Stealer",     "setStealerESP"     },
        { "BASE OWNER ESP",  "ESP_BaseOwner",   "setBaseOwnerESP"   },
        { "BRAINROT ESP",    "ESP_Brainrot",    "setBrainrotESP"    },
        { "BRAINROT BOXES",  "ESP_BrainrotBox", "setBrainrotBoxESP" },
        { "TRAP ESP",        "ESP_Trap",        "setTrapESP"        },
    }) do
        local flag, setter = e[2], e[3]
        toggle(visList, e[1],
            function() return _G[flag] == true end,
            function()
                local on = not (_G[flag] == true)
                if _G[setter] then
                    pcall(_G[setter], on)
                else
                    -- The switch used to light up here regardless, which made a
                    -- backend that never loaded look identical to one that was
                    -- simply finding nothing. Say so instead.
                    _G[flag] = on
                    pcall(_RP, "[ESP] no backend for " .. tostring(setter)
                        .. " -- the ESP block did not load")
                end
                pcall(_G.RyPrivSaveSettings)
            end)
    end
    for _, s in ipairs({
        { "ESP Distance",     100, 2000, "ESP_MaxDistance", 50   },
        { "Brainrots Shown",    1,   15, "ESP_BrainrotTop",  1   },
        { "Box Size",           2,   10, "ESP_BoxSize",      0.5 },
        { "Box Transparency",   0,  0.9, "ESP_BoxTransp",    0.05},
    }) do
        local g = s[4]
        slider(visList, s[1], s[2], s[3],
            function() return _G[g] end,
            function(v) _G[g] = v pcall(_G.RyPrivSaveSettings) end, s[5])
    end

    section(visList, "Vision")
    toggle(visList, "XRAY BASES",
        function() return _G.RyPrivXray ~= false end,
        function() if _G.RyPrivToggleXray then pcall(_G.RyPrivToggleXray) end end)
    toggle(visList, "FOV",
        function() return _G.RyPrivFOVEnabled == true end,
        function() _G.RyPrivFOVEnabled = not (_G.RyPrivFOVEnabled == true) end)
    slider(visList, "FOV Value", 50, 120,
        function() return tonumber(_G.RyPrivFOV) or 70 end,
        function(v) _G.RyPrivFOV = v end, 1)
    toggle(visList, "FACE AWAY",
        function() return _G.RyPrivFaceAway == true end,
        function() _G.RyPrivFaceAway = not (_G.RyPrivFaceAway == true) end)
    toggle(visList, "FACE AWAY: NEAREST",
        function() return _G.RyPrivFaceAwayNearest == true end,
        function() _G.RyPrivFaceAwayNearest = not (_G.RyPrivFaceAwayNearest == true) end)

    section(movList, "Auto buy")
    _G.RyPrivRepaintAutoBuy = toggle(movList, "AUTO BUY",
        function() return _G.RyPrivAutoBuy == true end,
        function() _G.RyPrivAutoBuy = not (_G.RyPrivAutoBuy == true) end)
    slider(movList, "Auto Buy Range", 5, 40,
        function() return _G.RyPrivAutoBuyRange end,
        function(v) _G.RyPrivAutoBuyRange = v end, 1)
    slider(movList, "Auto Buy Float", 0, 20,
        function() return _G.RyPrivAutoBuyHover end,
        function(v) _G.RyPrivAutoBuyHover = v end, 1)

    section(admList, "Alerts & server")
    toggle(admList, "PRIORITY ALERT",
        function() return _G.RyPrivPriAlert == true end,
        function() _G.RyPrivPriAlert = not (_G.RyPrivPriAlert == true) end)
    slider(admList, "Alert Min M/s", 0, 500,
        function() return (tonumber(_G.RyPrivAlertMinGen) or 80e6) / 1e6 end,
        function(v) _G.RyPrivAlertMinGen = v * 1e6 end, 5)
    input(admList, "ALERT SOUND ID", "111786441593851",
        function() return _G.RyPrivAlertSound end,
        function(v) _G.RyPrivAlertSound = (tostring(v or ""):match("%d+")) or "111786441593851" end)
    toggle(admList, "AUTO KICK ON STEAL",
        function() return _G.RyPrivAutoKickOnSteal == true end,
        function() _G.RyPrivAutoKickOnSteal = not (_G.RyPrivAutoKickOnSteal == true) end)
    toggle(admList, "KICK TO PRIVATE SERVER",
        function() return _G.RyPrivKickToPS == true end,
        function() _G.RyPrivKickToPS = not (_G.RyPrivKickToPS == true) end)
    input(admList, "PRIVATE SERVER LINK", "paste link or share code",
        function() return _G.RyPrivPrivateServerLink end,
        function(v) _G.RyPrivPrivateServerLink = (tostring(v or ""):match("^%s*(.-)%s*$")) end)

    -- ── GO ───────────────────────────────────────────────────────────────
    showTab(visList)
    for _, f in ipairs(_alphaPaint) do pcall(f) end
    applyLayers()

    _G.RyPrivToggleSettings = function() panel.Visible = not panel.Visible end

    task.spawn(function()
        while sg.Parent do
            task.wait(0.3)
            if panel.Visible then
                for _, r in ipairs(_repaints) do pcall(r) end
                for _, r in ipairs(_alphaPaint) do pcall(r) end
            end
        end
    end)
end

-- ============================================================
-- UTILITY PANEL
-- ============================================================
-- Grey shell, light blue accents, see-through surfaces. Every row drives a
-- switch this hub actually implements. INSTANT STEAL drives the edge-burst
-- engine in the steal section; FPS Boost is left out rather than added as a
-- dead toggle.
--
-- STEAL NEAREST / STEAL PRIORITY live here now, not on the target panel. They
-- are two views of one setting, _G.RyPrivStealMode: "nearest", "priority", or
-- nil for auto steal off, so turning one on turns the other off.
-- ============================================================
do
    local UC = {
        bg    = Color3.fromRGB(30, 32, 36),
        bg2   = Color3.fromRGB(40, 43, 48),
        row   = Color3.fromRGB(52, 56, 62),
        rowOn = Color3.fromRGB(58, 78, 96),
        line  = Color3.fromRGB(92, 99, 110),
        acc   = Color3.fromRGB(96, 178, 235),
        acc2  = Color3.fromRGB(155, 212, 248),
        txt   = Color3.fromRGB(240, 243, 247),
        dim   = Color3.fromRGB(158, 166, 178),
        bad   = Color3.fromRGB(216, 84, 92),
    }
    local FB, FR = Enum.Font.GothamBold, Enum.Font.Gotham
    local function A() return math.clamp(tonumber(_G.RyPrivAlpha) or 0.12, 0, 0.9) end
    local function mk(cls, parent, props)
        local o = Instance.new(cls)
        for k, v in pairs(props or {}) do o[k] = v end
        o.Parent = parent
        return o
    end
    local function round(o, r) mk("UICorner", o, { CornerRadius = UDim.new(0, 0) }) return o end

    -- drag/position now comes from the shared implementation near the top of
    -- the file; this block used to carry its own copy and export it
    local dragBy = _G.RyPrivPanelDrag

    -- One place that owns the steal mode, so the keybind, the utility rows and
    -- the target panel's read-out can never disagree.
    _G.RyPrivSetStealMode = function(m)
        _G.RyPrivStealMode = (m == "nearest" or m == "priority") and m or nil
        pcall(_G.RyPrivSaveSettings)
    end

    task.spawn(function()
        _G.RyPrivUIWait()
        local host = (gethui and gethui()) or game:GetService("CoreGui") or PG
        pcall(function()
            local old = host:FindFirstChild("RyPrivUtilityUI")
            if old then old:Destroy() end
        end)
        local sg = mk("ScreenGui", nil, {
            Name = "RyPrivUtilityUI", ResetOnSpawn = false,
            IgnoreGuiInset = true, DisplayOrder = 999995,
            ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
        })
        pcall(function() sg.Parent = host end)
        if not sg.Parent then sg.Parent = PG end

        local repaints = {}
        local alphaPaint = {}

        local panel = round(mk("Frame", sg, {
            Name = "UtilityPanel", Active = true,
            Size = UDim2.fromOffset(212, 0), AutomaticSize = Enum.AutomaticSize.Y,
            Position = UDim2.fromOffset(tonumber(_G._ryPriv_utX) or 250,
                tonumber(_G._ryPriv_utY) or 300),
            BackgroundColor3 = UC.bg, BorderSizePixel = 0,
        }), 9)
        mk("UIStroke", panel, { Color = UC.line, Thickness = 1, Transparency = 0.35 })
        alphaPaint[#alphaPaint + 1] = function() panel.BackgroundTransparency = A() end

        local head = mk("TextButton", panel, {
            Name = "Head", Size = UDim2.new(1, 0, 0, 34),
            BackgroundTransparency = 1, Text = "", AutoButtonColor = false, Active = true,
        })
        mk("Frame", head, {
            Size = UDim2.fromOffset(3, 14), Position = UDim2.new(0, 10, 0.5, -7),
            BackgroundColor3 = UC.acc, BorderSizePixel = 0,
        })
        mk("TextLabel", head, {
            Size = UDim2.new(1, -60, 1, 0), Position = UDim2.new(0, 19, 0, 0),
            BackgroundTransparency = 1, Text = "UTILITY", Font = FB, TextSize = 12,
            TextColor3 = UC.txt, TextXAlignment = Enum.TextXAlignment.Left,
        })
        local closeBtn = round(mk("TextButton", head, {
            Size = UDim2.fromOffset(20, 20), Position = UDim2.new(1, -28, 0.5, -10),
            BackgroundColor3 = UC.row, BackgroundTransparency = 0.3,
            Text = "x", Font = FB, TextSize = 12, TextColor3 = UC.dim,
            AutoButtonColor = false,
        }), 5)
        closeBtn.MouseButton1Click:Connect(function() panel.Visible = false end)
        dragBy(panel, head, "_ryPriv_utX", "_ryPriv_utY")
        _G.RyPrivApplyPos(panel, "_ryPriv_utX", "_ryPriv_utY", nil)
        _G.RyPrivRegisterPanel(panel, "_ryPriv_utX", "_ryPriv_utY")

        mk("Frame", panel, {
            Size = UDim2.new(1, -20, 0, 1), Position = UDim2.new(0, 10, 0, 34),
            BackgroundColor3 = UC.line, BackgroundTransparency = 0.5, BorderSizePixel = 0,
        })

        local body = mk("Frame", panel, {
            Size = UDim2.new(1, -20, 0, 0), AutomaticSize = Enum.AutomaticSize.Y,
            Position = UDim2.new(0, 10, 0, 40), BackgroundTransparency = 1,
        })
        mk("UIListLayout", body, {
            SortOrder = Enum.SortOrder.LayoutOrder, Padding = UDim.new(0, 4),
        })
        mk("UIPadding", body, { PaddingBottom = UDim.new(0, 10) })

        -- Explicit LayoutOrder: with a LayoutOrder-sorted UIListLayout, rows
        -- that all share the default 0 are free to come out in any order.
        local _ord = 0
        local function ord() _ord = _ord + 1 return _ord end

        local function mkSection(text)
            mk("TextLabel", body, {
                Size = UDim2.new(1, 0, 0, 17), BackgroundTransparency = 1,
                Font = FB, TextSize = 9, TextColor3 = UC.dim, LayoutOrder = ord(),
                TextXAlignment = Enum.TextXAlignment.Left, Text = text:upper(),
            })
        end

        local function mkToggle(label, get, set)
            local row = round(mk("Frame", body, {
                Size = UDim2.new(1, 0, 0, 30), BackgroundColor3 = UC.row,
                BorderSizePixel = 0, LayoutOrder = ord(),
            }), 6)
            local st = mk("UIStroke", row, { Color = UC.line, Thickness = 1, Transparency = 0.5 })
            mk("TextLabel", row, {
                Size = UDim2.new(1, -46, 1, 0), Position = UDim2.new(0, 10, 0, 0),
                BackgroundTransparency = 1, Font = FB, TextSize = 11, TextColor3 = UC.txt,
                TextXAlignment = Enum.TextXAlignment.Left, Text = label,
            })
            local pill = round(mk("Frame", row, {
                Size = UDim2.fromOffset(28, 14), Position = UDim2.new(1, -38, 0.5, -7),
                BackgroundColor3 = UC.line, BorderSizePixel = 0,
            }), 7)
            local knob = round(mk("Frame", pill, {
                Size = UDim2.fromOffset(10, 10), Position = UDim2.fromOffset(2, 2),
                BackgroundColor3 = Color3.fromRGB(226, 231, 237), BorderSizePixel = 0,
            }), 5)
            local click = mk("TextButton", row, {
                Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1, Text = "",
                AutoButtonColor = false, Active = true,
            })
            local function paint()
                local on = get() == true
                pill.BackgroundColor3 = on and UC.acc or UC.line
                knob.Position = on and UDim2.fromOffset(16, 2) or UDim2.fromOffset(2, 2)
                row.BackgroundColor3 = on and UC.rowOn or UC.row
                row.BackgroundTransparency = (on and 0.15 or 0.28) + A() * 0.45
                st.Color = on and UC.acc or UC.line
                st.Transparency = on and 0.3 or 0.55
            end
            click.MouseButton1Click:Connect(function() pcall(set) paint() end)
            paint()
            repaints[#repaints + 1] = paint
            return paint
        end

        local function mkPair()
            local f = mk("Frame", body, {
                Size = UDim2.new(1, 0, 0, 28), BackgroundTransparency = 1,
                LayoutOrder = ord(),
            })
            mk("UIListLayout", f, {
                FillDirection = Enum.FillDirection.Horizontal,
                Padding = UDim.new(0, 4), SortOrder = Enum.SortOrder.LayoutOrder,
            })
            return f
        end
        local function mkHalf(parent, label, order, onClick, danger)
            local b = round(mk("TextButton", parent, {
                Size = UDim2.new(0.5, -2, 1, 0), BackgroundColor3 = UC.row,
                Font = FB, TextSize = 10, LayoutOrder = order,
                TextColor3 = danger and UC.bad or UC.txt, Text = label,
                AutoButtonColor = false, BorderSizePixel = 0,
            }), 6)
            mk("UIStroke", b, {
                Color = danger and UC.bad or UC.line, Thickness = 1, Transparency = 0.55,
            })
            alphaPaint[#alphaPaint + 1] = function() b.BackgroundTransparency = 0.28 + A() * 0.45 end
            if onClick then b.MouseButton1Click:Connect(onClick) end
            return b
        end

        -- ── AUTOMATION ────────────────────────────────────────────────────
        mkSection("Automation")

        local nrPaint, prPaint
        local function pickMode(m)
            if _G.RyPrivStealMode == m then
                _G.RyPrivSetStealMode(nil)
            else
                _G.RyPrivSetStealMode(m)
            end
            pcall(nrPaint) pcall(prPaint)
        end
        nrPaint = mkToggle("Steal Nearest",
            function() return _G.RyPrivStealMode == "nearest" end,
            function() pickMode("nearest") end)
        prPaint = mkToggle("Steal Priority",
            function() return _G.RyPrivStealMode == "priority" end,
            function() pickMode("priority") end)

        -- INSTANT STEAL is a burst engine, not a third mode: nearest/priority
        -- keep picking and steering the target, this takes over the firing while
        -- it is on. Base Camp switches it on for you and back off again, and the
        -- key for it is in Settings -> KEYBINDS -> Steal.
        local instPaint
        instPaint = mkToggle("Instant Steal",
            function() return _G.RyPrivInstantSteal == true end,
            function()
                if _G.RyPrivToggleInstantSteal then pcall(_G.RyPrivToggleInstantSteal) end
            end)
        -- the engine repaints through this on the keybind, on a Base Camp flip
        -- and on each blink of its own 1.5s recycle, which the panel's 0.3s
        -- repaint tick would otherwise nearly always miss
        _G.RyPrivRepaintInstant = instPaint

        -- Auto Teleport, Auto Buy, Auto Kick on Steal, Priority Alert, Walk
        -- Speed, Carpet Speed and Xray live in Settings -> FEATURES. This
        -- panel is the steal controls and the quick actions.

        -- ── ACTIONS ───────────────────────────────────────────────────────
        mkSection("Actions")

        do
            local p1 = mkPair()
            local rjBtn
            rjBtn = mkHalf(p1, "REJOIN", 1, function()
                rjBtn.Text = "REJOIN..."
                task.spawn(function()
                    local TPS = game:GetService("TeleportService")
                    local ok = pcall(function() TPS:TeleportAsync(game.PlaceId, { LP }) end)
                    if not ok then pcall(function() TPS:Teleport(game.PlaceId, LP) end) end
                end)
                task.delay(2, function() if rjBtn.Parent then rjBtn.Text = "REJOIN" end end)
            end)
            mkHalf(p1, "KICK", 2, function()
                task.spawn(function() if _G.RyPrivKickOut then pcall(_G.RyPrivKickOut) end end)
            end, true)

            local p2 = mkPair()
            mkHalf(p2, "RESET", 1, function()
                task.spawn(function() if _G.RyPrivInstaReset then pcall(_G.RyPrivInstaReset) end end)
            end)
            -- JOIN QUEUE, in place of the old AJ saved-script runner. The
            -- engine is at the bottom of this file; this button just drives
            -- RyToggleJoinQueue and reads its status back.
            local jqBtn
            jqBtn = mkHalf(p2, "JQ: OFF", 2, function()
                if _G.RyToggleJoinQueue then pcall(_G.RyToggleJoinQueue) end
            end)
            local function refreshJQ()
                if not jqBtn.Parent then return end
                local on = _G.RyJoinQueueEnabled == true
                local last = on and _G.RyJoinQueueLast or nil
                if type(last) == "string" and last ~= "" then
                    jqBtn.Text = "JQ: " .. last:upper():sub(1, 13)
                else
                    jqBtn.Text = on and "JQ: ON" or "JQ: OFF"
                end
                jqBtn.TextColor3 = on and UC.acc2 or UC.txt
            end
            refreshJQ()
            -- the engine calls this on every status change
            _G.RyRefreshJoinQueueBtn = refreshJQ
            -- and the panel's own 0.3s repaint tick keeps it honest, rather than
            -- the dedicated forever-loop the v5 hub spends a thread on
            repaints[#repaints + 1] = refreshJQ
        end

        -- ── SETTINGS POPOUT ───────────────────────────────────────────────
        local gearBtn = round(mk("TextButton", body, {
            Size = UDim2.new(1, 0, 0, 24), BackgroundColor3 = UC.bg2,
            Font = FB, TextSize = 10, TextColor3 = UC.dim, Text = "Settings",
            AutoButtonColor = false, BorderSizePixel = 0, LayoutOrder = ord(),
        }), 6)
        mk("UIStroke", gearBtn, { Color = UC.line, Thickness = 1, Transparency = 0.55 })
        alphaPaint[#alphaPaint + 1] = function() gearBtn.BackgroundTransparency = 0.25 + A() * 0.45 end

        local pop = round(mk("Frame", sg, {
            Name = "UtilitySettings", Size = UDim2.fromOffset(212, 96),
            Position = UDim2.fromOffset(250, 620), BackgroundColor3 = UC.bg,
            BorderSizePixel = 0, Active = true, Visible = false,
        }), 9)
        mk("UIStroke", pop, { Color = UC.acc, Thickness = 1, Transparency = 0.45 })
        alphaPaint[#alphaPaint + 1] = function() pop.BackgroundTransparency = A() end

        local function popLabel(text, y)
            mk("TextLabel", pop, {
                Size = UDim2.new(1, -16, 0, 14), Position = UDim2.fromOffset(10, y),
                BackgroundTransparency = 1, Font = FB, TextSize = 9, TextColor3 = UC.dim,
                TextXAlignment = Enum.TextXAlignment.Left, Text = text,
            })
        end

        popLabel("PRIVATE SERVER LINK", 10)
        do
            local psBox = round(mk("TextBox", pop, {
                Size = UDim2.fromOffset(160, 24), Position = UDim2.fromOffset(10, 27),
                BackgroundColor3 = UC.bg2, BackgroundTransparency = 0.15,
                TextColor3 = UC.txt, BorderSizePixel = 0, Font = FR, TextSize = 10,
                ClearTextOnFocus = false, TextXAlignment = Enum.TextXAlignment.Left,
                PlaceholderText = "paste link or share code", PlaceholderColor3 = UC.dim,
                Text = tostring(_G.RyPrivPrivateServerLink or ""), TextTruncate = Enum.TextTruncate.AtEnd,
            }), 5)
            mk("UIStroke", psBox, { Color = UC.line, Thickness = 1, Transparency = 0.55 })
            mk("UIPadding", psBox, { PaddingLeft = UDim.new(0, 6) })
            local maskLbl = mk("TextLabel", pop, {
                Size = psBox.Size, Position = psBox.Position, BackgroundTransparency = 1,
                Font = FR, TextSize = 10, TextColor3 = UC.txt,
                TextXAlignment = Enum.TextXAlignment.Left, Visible = false, ZIndex = 2,
            })
            mk("UIPadding", maskLbl, { PaddingLeft = UDim.new(0, 6) })
            local shown = false
            local function renderMask()
                if shown or #psBox.Text == 0 then
                    psBox.TextTransparency = 0
                    maskLbl.Visible = false
                else
                    psBox.TextTransparency = 1
                    maskLbl.Visible = true
                    maskLbl.Text = string.rep("*", math.min(#psBox.Text, 26))
                end
            end
            local eye = round(mk("TextButton", pop, {
                Size = UDim2.fromOffset(28, 24), Position = UDim2.fromOffset(174, 27),
                BackgroundColor3 = UC.row, BackgroundTransparency = 0.25,
                Text = "o", Font = FB, TextSize = 11, TextColor3 = UC.dim,
                AutoButtonColor = false, BorderSizePixel = 0,
            }), 5)
            eye.MouseButton1Click:Connect(function()
                shown = not shown
                eye.TextColor3 = shown and UC.acc2 or UC.dim
                renderMask()
            end)
            psBox:GetPropertyChangedSignal("Text"):Connect(renderMask)
            psBox.FocusLost:Connect(function()
                _G.RyPrivPrivateServerLink = tostring(psBox.Text or ""):match("^%s*(.-)%s*$")
                pcall(_G.RyPrivSaveSettings)
            end)
            renderMask()
        end

        -- Kick-to-private-server switch, and the transparency dial that every
        -- panel in the hub reads through _G.RyPrivAlpha.
        do
            local ktRow = round(mk("TextButton", pop, {
                Size = UDim2.new(1, -20, 0, 26), Position = UDim2.fromOffset(10, 58),
                BackgroundColor3 = UC.row, Text = "", AutoButtonColor = false,
                BorderSizePixel = 0,
            }), 5)
            mk("TextLabel", ktRow, {
                Size = UDim2.new(1, -46, 1, 0), Position = UDim2.new(0, 8, 0, 0),
                BackgroundTransparency = 1, Font = FB, TextSize = 10, TextColor3 = UC.txt,
                TextXAlignment = Enum.TextXAlignment.Left, Text = "Kick to Private Server",
            })
            local dot = round(mk("Frame", ktRow, {
                Size = UDim2.fromOffset(10, 10), Position = UDim2.new(1, -22, 0.5, -5),
                BackgroundColor3 = UC.line, BorderSizePixel = 0,
            }), 5)
            local function ktPaint()
                local on = _G.RyPrivKickToPS == true
                dot.BackgroundColor3 = on and UC.acc or UC.line
                ktRow.BackgroundColor3 = on and UC.rowOn or UC.row
                ktRow.BackgroundTransparency = 0.25 + A() * 0.45
            end
            ktRow.MouseButton1Click:Connect(function()
                _G.RyPrivKickToPS = not (_G.RyPrivKickToPS == true)
                pcall(_G.RyPrivSaveSettings)
                ktPaint()
            end)
            ktPaint()
            repaints[#repaints + 1] = ktPaint
        end

        -- UI TRANSPARENCY moved to the Settings panel's UI tab, which is where
        -- the rest of the per-layer UI controls live.

        gearBtn.MouseButton1Click:Connect(function()
            pop.Visible = not pop.Visible
            if pop.Visible then
                pop.Position = UDim2.fromOffset(
                    panel.AbsolutePosition.X,
                    panel.AbsolutePosition.Y + panel.AbsoluteSize.Y + 6)
            end
        end)

        for _, f in ipairs(alphaPaint) do pcall(f) end

        _G.RyPrivToggleUtility = function()
            panel.Visible = not panel.Visible
            if not panel.Visible then pop.Visible = false end
        end

        task.spawn(function()
            while sg.Parent do
                task.wait(0.3)
                if panel.Visible then
                    for _, f in ipairs(repaints) do pcall(f) end
                    -- picks up the Settings panel's transparency dial
                    for _, f in ipairs(alphaPaint) do pcall(f) end
                end
            end
        end)
    end)
end

-- ============================================================
-- INVIS STEAL PANEL
-- ============================================================
-- Same rows as the rymogs panel, driving this hub's own invis engine
-- (_G.RyPrivInvis*) instead of carrying a second copy of it. AUTO RECOVER is
-- the anti-ragdoll watcher's new _G.RyPrivAntiRagdoll switch.
-- ============================================================
do
    local IC = {
        bg    = Color3.fromRGB(30, 32, 36),
        bg2   = Color3.fromRGB(40, 43, 48),
        row   = Color3.fromRGB(52, 56, 62),
        rowOn = Color3.fromRGB(58, 78, 96),
        line  = Color3.fromRGB(92, 99, 110),
        acc   = Color3.fromRGB(96, 178, 235),
        acc2  = Color3.fromRGB(155, 212, 248),
        txt   = Color3.fromRGB(240, 243, 247),
        dim   = Color3.fromRGB(158, 166, 178),
    }
    local FB, FR = Enum.Font.GothamBold, Enum.Font.Gotham
    local function A() return math.clamp(tonumber(_G.RyPrivAlpha) or 0.12, 0, 0.9) end
    local function mk(cls, parent, props)
        local o = Instance.new(cls)
        for k, v in pairs(props or {}) do o[k] = v end
        o.Parent = parent
        return o
    end
    local function round(o, r) mk("UICorner", o, { CornerRadius = UDim.new(0, 0) }) return o end

    task.spawn(function()
        _G.RyPrivUIWait()
        local host = (gethui and gethui()) or game:GetService("CoreGui") or PG
        pcall(function()
            local old = host:FindFirstChild("RyPrivInvisUI")
            if old then old:Destroy() end
        end)
        local sg = mk("ScreenGui", nil, {
            Name = "RyPrivInvisUI", ResetOnSpawn = false,
            IgnoreGuiInset = true, DisplayOrder = 999994,
            ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
        })
        pcall(function() sg.Parent = host end)
        if not sg.Parent then sg.Parent = PG end

        local ups = {}

        local panel = round(mk("Frame", sg, {
            Name = "InvisPanel", Active = true,
            Size = UDim2.fromOffset(206, 0), AutomaticSize = Enum.AutomaticSize.Y,
            Position = UDim2.fromOffset(tonumber(_G._ryPriv_ivX) or 14,
                tonumber(_G._ryPriv_ivY) or 410),
            BackgroundColor3 = IC.bg, BackgroundTransparency = A(), BorderSizePixel = 0,
        }), 9)
        mk("UIStroke", panel, { Color = IC.line, Thickness = 1, Transparency = 0.35 })

        local head = mk("TextButton", panel, {
            Name = "Head", Size = UDim2.new(1, 0, 0, 34),
            BackgroundTransparency = 1, Text = "", AutoButtonColor = false, Active = true,
        })
        mk("Frame", head, {
            Size = UDim2.fromOffset(3, 14), Position = UDim2.new(0, 10, 0.5, -7),
            BackgroundColor3 = IC.acc, BorderSizePixel = 0,
        })
        mk("TextLabel", head, {
            Size = UDim2.new(1, -60, 1, 0), Position = UDim2.new(0, 19, 0, 0),
            BackgroundTransparency = 1, Text = "INVIS STEAL", Font = FB, TextSize = 12,
            TextColor3 = IC.txt, TextXAlignment = Enum.TextXAlignment.Left,
        })
        local closeBtn = round(mk("TextButton", head, {
            Size = UDim2.fromOffset(20, 20), Position = UDim2.new(1, -28, 0.5, -10),
            BackgroundColor3 = IC.row, BackgroundTransparency = 0.3,
            Text = "x", Font = FB, TextSize = 12, TextColor3 = IC.dim,
            AutoButtonColor = false,
        }), 5)
        closeBtn.MouseButton1Click:Connect(function() panel.Visible = false end)
        _G.RyPrivPanelDrag(panel, head, "_ryPriv_ivX", "_ryPriv_ivY")
        _G.RyPrivApplyPos(panel, "_ryPriv_ivX", "_ryPriv_ivY", nil)
        _G.RyPrivRegisterPanel(panel, "_ryPriv_ivX", "_ryPriv_ivY")

        mk("Frame", panel, {
            Size = UDim2.new(1, -20, 0, 1), Position = UDim2.new(0, 10, 0, 34),
            BackgroundColor3 = IC.line, BackgroundTransparency = 0.5, BorderSizePixel = 0,
        })

        local body = mk("Frame", panel, {
            Size = UDim2.new(1, -20, 0, 0), AutomaticSize = Enum.AutomaticSize.Y,
            Position = UDim2.new(0, 10, 0, 40), BackgroundTransparency = 1,
        })
        mk("UIListLayout", body, {
            SortOrder = Enum.SortOrder.LayoutOrder, Padding = UDim.new(0, 4),
        })
        mk("UIPadding", body, { PaddingBottom = UDim.new(0, 10) })

        local _ord = 0
        local function ord() _ord = _ord + 1 return _ord end

        local function mkBtn(label, get, set)
            local row = round(mk("Frame", body, {
                Size = UDim2.new(1, 0, 0, 30), BackgroundColor3 = IC.row,
                BorderSizePixel = 0, LayoutOrder = ord(),
            }), 6)
            local st = mk("UIStroke", row, { Color = IC.line, Thickness = 1, Transparency = 0.55 })
            local lbl = mk("TextLabel", row, {
                Size = UDim2.new(1, -20, 1, 0), Position = UDim2.new(0, 10, 0, 0),
                BackgroundTransparency = 1, Font = FB, TextSize = 11, TextColor3 = IC.txt,
                TextXAlignment = Enum.TextXAlignment.Left, Text = label,
            })
            local click = mk("TextButton", row, {
                Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1, Text = "",
                AutoButtonColor = false, Active = true,
            })
            local function up()
                local on = get() == true
                lbl.Text = label .. (on and ": ON" or ": OFF")
                row.BackgroundColor3 = on and IC.rowOn or IC.row
                row.BackgroundTransparency = (on and 0.15 or 0.28) + A() * 0.45
                st.Color = on and IC.acc or IC.line
                st.Transparency = on and 0.3 or 0.55
            end
            click.MouseButton1Click:Connect(function() pcall(set) up() end)
            up()
            ups[#ups + 1] = up
            return up
        end

        local function mkSlider(label, minV, maxV, step, get, set)
            local row = round(mk("Frame", body, {
                Size = UDim2.new(1, 0, 0, 42), BackgroundColor3 = IC.row,
                BackgroundTransparency = 0.28 + A() * 0.45, BorderSizePixel = 0,
                LayoutOrder = ord(),
            }), 6)
            mk("UIStroke", row, { Color = IC.line, Thickness = 1, Transparency = 0.6 })
            mk("TextLabel", row, {
                Size = UDim2.new(0.62, 0, 0, 14), Position = UDim2.fromOffset(10, 5),
                BackgroundTransparency = 1, Font = FR, TextSize = 10, TextColor3 = IC.dim,
                TextXAlignment = Enum.TextXAlignment.Left, Text = label,
            })
            local valLbl = mk("TextLabel", row, {
                AnchorPoint = Vector2.new(1, 0), Position = UDim2.new(1, -10, 0, 5),
                Size = UDim2.fromOffset(52, 14), BackgroundTransparency = 1,
                Font = FB, TextSize = 10, TextColor3 = IC.acc2,
                TextXAlignment = Enum.TextXAlignment.Right, Text = "",
            })
            local track = round(mk("TextButton", row, {
                Size = UDim2.new(1, -20, 0, 5), Position = UDim2.fromOffset(10, 26),
                BackgroundColor3 = IC.line, BackgroundTransparency = 0.4,
                Text = "", AutoButtonColor = false, BorderSizePixel = 0,
            }), 3)
            local fill = round(mk("Frame", track, {
                Size = UDim2.new(0, 0, 1, 0), BackgroundColor3 = IC.acc, BorderSizePixel = 0,
            }), 3)
            local function refresh()
                local v = tonumber(get()) or minV
                fill.Size = UDim2.new(
                    math.clamp((v - minV) / math.max(maxV - minV, 0.001), 0, 1), 0, 1, 0)
                valLbl.Text = string.format((step < 1) and "%.1f" or "%.0f", v)
            end
            local sliding = false
            local function applyAt(x)
                local pct = math.clamp((x - track.AbsolutePosition.X)
                    / math.max(track.AbsoluteSize.X, 1), 0, 1)
                local raw = minV + (maxV - minV) * pct
                set(math.clamp(math.floor(raw / step + 0.5) * step, minV, maxV))
                refresh()
            end
            track.InputBegan:Connect(function(i)
                if i.UserInputType == Enum.UserInputType.MouseButton1
                    or i.UserInputType == Enum.UserInputType.Touch then
                    sliding = true
                    applyAt(i.Position.X)
                end
            end)
            UIS.InputChanged:Connect(function(i)
                if not sliding then return end
                if i.UserInputType == Enum.UserInputType.MouseMovement
                    or i.UserInputType == Enum.UserInputType.Touch then
                    applyAt(i.Position.X)
                end
            end)
            UIS.InputEnded:Connect(function(i)
                if i.UserInputType == Enum.UserInputType.MouseButton1
                    or i.UserInputType == Enum.UserInputType.Touch then
                    sliding = false
                    pcall(_G.RyPrivSaveSettings)
                end
            end)
            refresh()
            ups[#ups + 1] = refresh
            return refresh
        end

        local function mkPresets(defs)
            local row = mk("Frame", body, {
                Size = UDim2.new(1, 0, 0, 24), BackgroundTransparency = 1,
                LayoutOrder = ord(),
            })
            mk("UIListLayout", row, {
                FillDirection = Enum.FillDirection.Horizontal,
                Padding = UDim.new(0, 4), SortOrder = Enum.SortOrder.LayoutOrder,
            })
            for n, def in ipairs(defs) do
                local b = round(mk("TextButton", row, {
                    Size = UDim2.new(1 / #defs, -(4 * (#defs - 1) / #defs), 1, 0),
                    BackgroundColor3 = IC.bg2, BackgroundTransparency = 0.25 + A() * 0.45,
                    Font = FB, TextSize = 10, TextColor3 = IC.txt, Text = def[1],
                    AutoButtonColor = false, BorderSizePixel = 0, LayoutOrder = n,
                }), 5)
                mk("UIStroke", b, { Color = IC.line, Thickness = 1, Transparency = 0.6 })
                b.MouseButton1Click:Connect(def[2])
            end
            return row
        end

        mkBtn("INVIS",
            function() return _G.RyPrivInvisActive == true end,
            function() if _G.RyPrivInvisToggle then pcall(_G.RyPrivInvisToggle) end end)
        mkBtn("DROP VIEW",
            function() return _G.RyPrivDropView == true end,
            function() if _G.RyPrivToggleDropView then pcall(_G.RyPrivToggleDropView) end end)
        mkBtn("AUTO INVIS",
            function() return _G.RyPrivInvisAuto == true end,
            function()
                _G.RyPrivInvisAuto = not (_G.RyPrivInvisAuto == true)
                if _G.RyPrivInvisSync then pcall(_G.RyPrivInvisSync) end
                pcall(_G.RyPrivSaveSettings)
            end)
        mkBtn("AUTO RECOVER",
            function() return _G.RyPrivAntiRagdoll ~= false end,
            function()
                _G.RyPrivAntiRagdoll = (_G.RyPrivAntiRagdoll == false)
                pcall(_G.RyPrivSaveSettings)
            end)
        mkBtn("WALKSPEED",
            function() return _G.RyPrivWalkSpeedOn ~= false end,
            function()
                if _G.RyPrivSetWalkSpeed then
                    pcall(_G.RyPrivSetWalkSpeed, _G.RyPrivWalkSpeedOn == false)
                end
                pcall(_G.RyPrivSaveSettings)
            end)

        local refAngle = mkSlider("Rotation", 0, 360, 5,
            function() return _G.RyPrivInvisAngle end,
            function(v) _G.RyPrivInvisAngle = v end)
        mkPresets({
            { "180", function() _G.RyPrivInvisAngle = 180 refAngle() pcall(_G.RyPrivSaveSettings) end },
            { "225", function() _G.RyPrivInvisAngle = 225 refAngle() pcall(_G.RyPrivSaveSettings) end },
        })

        mkSlider("Depth", 0, 15, 0.1,
            function() return _G.RyPrivInvisDepth end,
            function(v) _G.RyPrivInvisDepth = math.clamp(v, 0, 15) end)

        local refWS = mkSlider("WalkSpeed", 5, 64, 1,
            function() return _G.RyPrivWalkSpeed end,
            function(v)
                _G.RyPrivWalkSpeed = v
                _G.RyPrivWalkSpeedOn = true
                if _G.RyPrivSetWalkSpeed then pcall(_G.RyPrivSetWalkSpeed, true) end
            end)
        mkPresets({
            { "24", function()
                _G.RyPrivWalkSpeed = 24
                if _G.RyPrivSetWalkSpeed then pcall(_G.RyPrivSetWalkSpeed, true) end
                refWS() pcall(_G.RyPrivSaveSettings)
            end },
            { "29", function()
                _G.RyPrivWalkSpeed = 29
                if _G.RyPrivSetWalkSpeed then pcall(_G.RyPrivSetWalkSpeed, true) end
                refWS() pcall(_G.RyPrivSaveSettings)
            end },
        })

        _G.RyPrivToggleInvisPanel = function() panel.Visible = not panel.Visible end

        task.spawn(function()
            while sg.Parent do
                task.wait(0.3)
                if panel.Visible then
                    panel.BackgroundTransparency = A()
                    for _, u in ipairs(ups) do pcall(u) end
                end
            end
        end)
    end)
end

-- ============================================================
-- ADMIN PANEL + BASE CAMP
-- ============================================================
-- Wide 640x340 panel: REFRESH, PLAYERS + SETTINGS tabs, one row per player with JAIL / RAG / RKT / BLN /
-- ALL / CAMP, "Stealing: <brainrot>" read-out, cooldown colouring,
-- blacklist, command order + pins, random-command mode, prox circle.
-- Click to AP / Prox AP / Spam Base Owner / Nightvision Owner are switches
-- on their own AUTO AP panel.
-- Commands are fired through the game's own AdminPanel GUI (you need the
-- admin panel in game for them to do anything).
--
-- Anti-AP: skips players detected as Atlas / Kawaifu / Braintopia users
-- from their in-world markers.
--
-- BASE CAMP (the CAMP button): finds the player's plot from its PlotSign,
-- flies you to the first empty podium in it with this hub's TP (route,
-- grapple, carpet, clone), then keeps the autosteal locked on the nearest
-- brainrot in that base, so anything already there -- or anything the owner
-- brings home -- gets taken. Turns itself off if the target leaves.
-- ============================================================
do
    local Players          = game:GetService("Players")
    local RunService       = game:GetService("RunService")
    local UIS              = game:GetService("UserInputService")
    local HttpService      = game:GetService("HttpService")
    local GuiService       = game:GetService("GuiService")
    local TextService      = game:GetService("TextService")
    local LP               = Players.LocalPlayer
    local PlayerGui        = LP:WaitForChild("PlayerGui")
    local VIM              = nil
    pcall(function() VIM = game:GetService("VirtualInputManager") end)

    -- the hub's own look (same palette as Utility / Invis / Settings):
    -- grey shell, light blue accents, square corners, see-through surfaces
    local Theme = {
        Background       = Color3.fromRGB(30, 32, 36),
        Surface          = Color3.fromRGB(40, 43, 48),
        SurfaceHighlight = Color3.fromRGB(52, 56, 62),
        Accent1          = Color3.fromRGB(96, 178, 235),
        Accent2          = Color3.fromRGB(92, 99, 110),
        TextPrimary      = Color3.fromRGB(240, 243, 247),
        TextSecondary    = Color3.fromRGB(158, 166, 178),
        Success          = Color3.fromRGB(96, 178, 235),
        Error            = Color3.fromRGB(216, 84, 92),
        Stroke           = Color3.fromRGB(92, 99, 110),
    }
    local function A() return math.clamp(tonumber(_G.RyPrivAlpha) or 0.12, 0, 0.9) end
    local function OnAccent(c)
        c = c or Theme.Accent1
        local lum = c.R * 0.299 + c.G * 0.587 + c.B * 0.114
        return lum > 0.55 and Color3.fromRGB(15, 15, 18) or Color3.fromRGB(245, 245, 250)
    end
    local IS_MOBILE = UIS.TouchEnabled and not UIS.KeyboardEnabled

    -- ── CONFIG (own file so it never collides with SideTP.json) ──────────
    local CFG_FILE = "RyPrivAdmin.json"
    local Config = {
        Blacklist = {}, AdminAllowedCmds = {}, AdminCmdPinned = {},
        AdminCmdOrder = { "ragdoll", "balloon", "rocket", "jail", "inverse", "jumpscare", "morph", "nightvision", "tiny" },
        AdminRandomCmd = false, ShowProxCircle = false, ProximityRange = 15,
        ClickToAP = false, ClickToAPSingleCommand = false,
        AutoProxAP = false, AutoSpamOwner = false, AutoNightOwner = false,
        AntiApAtlasUsers = true, AntiApKawaifuUsers = true, AntiApBraintopiaUsers = true,
        HideAdminPanel = false,
    }
    pcall(function()
        if not (isfile and readfile and isfile(CFG_FILE)) then return end
        local d = HttpService:JSONDecode(readfile(CFG_FILE))
        if type(d) ~= "table" then return end
        for k, v in pairs(d) do if Config[k] ~= nil and type(v) == type(Config[k]) then Config[k] = v end end
        if #Config.AdminCmdOrder == 0 then
            Config.AdminCmdOrder = { "ragdoll", "balloon", "rocket", "jail", "inverse", "jumpscare", "morph", "nightvision", "tiny" }
        end
    end)
    local function SaveConfig()
        if not writefile then return end
        pcall(function() writefile(CFG_FILE, HttpService:JSONEncode(Config)) end)
    end

    local SharedState = { AdminButtonCache = {}, BalloonedPlayers = {} }
    local ProximityAPActive = false

    -- ── UI helpers ────────────────────────────────────────────────────────
        local function new(class, parent, props)
        local o = Instance.new(class)
        for k, v in pairs(props or {}) do o[k] = v end
        if parent ~= nil then o.Parent = parent end
        return o
    end
    -- blocky, like every other panel in the hub
    local function corner(o) return o end
    local function round(o) return o end
    local function stroke(o, col, th, tr)
        return new("UIStroke", o, { Color = col or Theme.Stroke, Thickness = th or 1, Transparency = tr or 0 })
    end
    local function lit(c) return Color3.new(math.min(c.R * 1.35, 1), math.min(c.G * 1.35, 1), math.min(c.B * 1.35, 1)) end

    local function notify(title, text)
        pcall(function()
            game:GetService("StarterGui"):SetCore("SendNotification", { Title = title, Text = text, Duration = 4 })
        end)
    end

    -- brainrot display names, for "Stealing: X" and the priority sort
    local _dispMap
    local function displayNameOf(indexKey)
        if not indexKey then return nil end
        if not _dispMap then
            _dispMap = {}
            pcall(function()
                local AD = require(game:GetService("ReplicatedStorage"):WaitForChild("Datas"):WaitForChild("Animals"))
                for idx, d in pairs(AD) do
                    if type(d) == "table" and d.DisplayName and d.DisplayName ~= idx then
                        _dispMap[idx] = d.DisplayName
                        _dispMap[tostring(idx):lower()] = d.DisplayName
                    end
                end
            end)
        end
        return _dispMap[indexKey] or _dispMap[tostring(indexKey):lower()]
    end

    local DANGER_TOOLS = {
        ["Boogie Bomb"] = true, ["Medusa's Head"] = true, ["Body Swap Potion"] = true,
        ["Laser Cape"] = true, ["Rainbowrath Sword"] = true, ["Gummy Bear"] = true,
    }

    -- ════════════════════════════════════════════════════════════════════
    -- BASE CAMP  (_G.RyPrivCamp, driving this hub's TP + autosteal)
    -- ════════════════════════════════════════════════════════════════════
    local RC = { active = false, target = nil, plotName = nil, status = "idle" }
    _G.RyPrivCamp = RC

    local function _plotOwnerName(plot)
        local sign = plot and plot:FindFirstChild("PlotSign")
        local gui  = sign and sign:FindFirstChildWhichIsA("SurfaceGui", true)
        local lbl  = gui and gui:FindFirstChildWhichIsA("TextLabel", true)
        if not (lbl and lbl.Text) then return nil end
        local nick = lbl.Text:match("^(.-)'") or lbl.Text
        nick = nick and nick:gsub("%s+$", "")
        if nick == "" then return nil end
        return nick
    end
    local function findPlotFor(plr)
        local plots = workspace:FindFirstChild("Plots")
        if not plots then return nil end
        local n = plr.Name:lower()
        local dn = (plr.DisplayName or ""):lower()
        for _, plot in ipairs(plots:GetChildren()) do
            local owner = _plotOwnerName(plot)
            if owner then
                local ol = owner:lower()
                if ol == n or ol == dn then return plot end
            end
        end
        -- fallback: the plot's synchronizer channel owner
        for _, plot in ipairs(plots:GetChildren()) do
            local ok, o = pcall(function()
                local ch = getPlotChannel(plot.Name)
                return ch and channelGet(ch, "Owner")
            end)
            if ok and o then
                local on = (typeof(o) == "Instance" and o.Name) or (type(o) == "table" and o.Name) or (type(o) == "string" and o)
                if on == plr.Name then return plot end
            end
        end
        return nil
    end
    local function resolvePlayer(q)
        q = tostring(q or ""):lower():gsub("%s+", "")
        if q == "" then return nil end
        local partial
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= LP then
                local n, d = p.Name:lower(), (p.DisplayName or ""):lower()
                if n == q or d == q then return p end
                if not partial and (n:find(q, 1, true) or d:find(q, 1, true)) then partial = p end
            end
        end
        return partial
    end
    -- occupied = the slot's Base.Spawn carries an enabled "Steal" prompt
    local function _slotOccupied(slot)
        local base  = slot:FindFirstChild("Base")
        local spawn = base and base:FindFirstChild("Spawn")
        local att   = spawn and spawn:FindFirstChild("PromptAttachment")
        if att then
            for _, p in ipairs(att:GetChildren()) do
                if p:IsA("ProximityPrompt") and p.Enabled and p.ActionText == "Steal" then return true end
            end
        end
        for _, d in ipairs(slot:GetDescendants()) do
            if d:IsA("ProximityPrompt") and d.Enabled and d.ActionText == "Steal" then return true end
        end
        return false
    end
    -- (slotName, spawnPart, full)
    local function campSlot(plot)
        local pods = plot:FindFirstChild("AnimalPodiums")
        if not pods then return nil, nil, false end
        local slots = {}
        for _, s in ipairs(pods:GetChildren()) do
            local base  = s:FindFirstChild("Base")
            local spawn = base and base:FindFirstChild("Spawn")
            if spawn and spawn:IsA("BasePart") then slots[#slots + 1] = s end
        end
        table.sort(slots, function(a, b)
            local na, nb = tonumber(tostring(a.Name):match("%d+")), tonumber(tostring(b.Name):match("%d+"))
            if na and nb then return na < nb end
            return tostring(a.Name) < tostring(b.Name)
        end)
        local firstName, firstSpawn
        for _, slot in ipairs(slots) do
            local spawn = slot.Base.Spawn
            firstName  = firstName  or slot.Name
            firstSpawn = firstSpawn or spawn
            if not _slotOccupied(slot) then return slot.Name, spawn, false end
        end
        return firstName, firstSpawn, true
    end
    -- nearest stealable brainrot sitting in the camped base
    local function campTarget()
        local ok, pets = pcall(function() return _G.RyPrivScanAllPets and _G.RyPrivScanAllPets() or {} end)
        if not ok or type(pets) ~= "table" then return nil end
        local hrp = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
        local myPos = hrp and hrp.Position
        local best, bestD, firstAny = nil, math.huge, nil
        for _, p in ipairs(pets) do
            if p and p.plot == RC.plotName and not p.conveyor then
                firstAny = firstAny or p
                if myPos and p.position then
                    local d = (p.position - myPos).Magnitude
                    if d < bestD then bestD, best = d, p end
                end
            end
        end
        return best or firstAny
    end
    local _lastSteer = 0
    local function campSteer()
        if not RC.arrived then return end
        if os.clock() - _lastSteer < 0.25 then return end
        _lastSteer = os.clock()
        local tgt = campTarget()
        if tgt then
            local uid = tostring(tgt.plot) .. "_" .. tostring(tgt.slot)
            if RC._mt ~= uid or not (_G.RyPrivArmedName and _G.RyPrivArmedName()) then
                RC._mt = uid
                if _G.RyPrivArmSteal then pcall(_G.RyPrivArmSteal, tgt) end
            end
        elseif RC._mt ~= nil then
            RC._mt = nil
            if _G.RyPrivDisarmSteal then pcall(_G.RyPrivDisarmSteal) end
        end
    end

    RC.start = function(q)
        local plr = resolvePlayer(q)
        if not plr then RC.status = "player not found"; notify("Base Camp", "player not found") return false, RC.status end
        local plot = findPlotFor(plr)
        if not plot then RC.status = "base not found"; notify("Base Camp", plr.Name .. "'s base not found") return false, RC.status end
        if RC.active then RC.stop(true) end
        RC.target   = plr.Name
        RC.plotName = plot.Name
        RC.active   = true
        RC.arrived  = false
        RC._mt      = nil
        _G.RyPrivCampActive = true
        -- camping needs the autosteal running; remember what it was
        RC._prevMode = _G.RyPrivStealMode
        if _G.RyPrivStealMode == nil then _G.RyPrivStealMode = "nearest" end
        -- Camping is the case instant steal exists for: you are parked in the
        -- target's base, so the frame a podium prompt lights up it should be
        -- fired rather than run through the approach gates. Quiet, so the camp's
        -- temporary override is not written into the settings file.
        if _G.RyPrivSetInstantSteal then pcall(_G.RyPrivSetInstantSteal, true, true) end
        local slotName, spawn, full = campSlot(plot)
        if full then
            RC.status = plr.Name .. "'s base is FULL"
            notify("Base Camp", plr.Name .. "'s base is FULL (no open slots)")
        else
            RC.status = "camping " .. plr.Name
            notify("Base Camp", "camping " .. plr.Name)
        end
        if slotName and spawn then
            local pos = spawn.Position + Vector3.new(0, 3, 0)
            -- keep 2nd-floor slots classed as 2nd floor by the TP's floor split
            if spawn.Position.Y > 10 then pos = Vector3.new(pos.X, math.max(pos.Y, 24), pos.Z) end
            local entry = { name = "Camp", index = "Camp", mps = 0, position = pos,
                            plot = plot.Name, slot = tostring(slotName) }
            RC.entry = entry
            task.spawn(function()
                -- wait out a TP already in flight
                local t0 = os.clock()
                while RC.active and isTeleporting and os.clock() - t0 < 8 do task.wait(0.1) end
                if not RC.active then return end
                if type(doVelocityTP) == "function" then
                    pcall(doVelocityTP, false, entry)   -- route + grapple + carpet + clone
                end
                if RC.entry == entry then RC.entry = nil end
                RC.arrived = RC.active
            end)
        else
            RC.arrived = true
        end
        return true, RC.status
    end

    RC.stop = function(silent)
        RC.active  = false
        RC.status  = "stopped"
        RC.entry   = nil
        RC.arrived = false
        RC._mt     = nil
        _G.RyPrivCampActive = false
        -- Instant steal goes off with the camp and the hub parks on priority
        -- steal. That is the resting state asked for, rather than whatever the
        -- mode happened to be when the camp was started.
        if _G.RyPrivSetInstantSteal then pcall(_G.RyPrivSetInstantSteal, false, true) end
        RC._prevMode = nil
        if _G.RyPrivSetStealMode then
            pcall(_G.RyPrivSetStealMode, "priority")
        else
            _G.RyPrivStealMode = "priority"
        end
        if _G.RyPrivDisarmSteal then pcall(_G.RyPrivDisarmSteal) end
        if not silent then notify("Base Camp", "camp off") end
    end

    Players.PlayerRemoving:Connect(function(p)
        if RC.active and RC.target and p.Name == RC.target then
            notify("Base Camp", (RC.target or "target") .. " left the game - camp off")
            RC.stop(true)
        end
    end)
    task.spawn(function()
        while true do
            task.wait(0.1)
            if RC.active then
                if RC.target and not Players:FindFirstChild(RC.target) then
                    notify("Base Camp", (RC.target or "target") .. " left the game - camp off")
                    RC.stop(true)
                else
                    pcall(campSteer)
                end
            end
        end
    end)

    -- ════════════════════════════════════════════════════════════════════
    -- ADMIN COMMANDS
    -- ════════════════════════════════════════════════════════════════════
    task.spawn(function()
        _G.RyPrivBootWait()
        local COOLDOWNS = { rocket = 120, ragdoll = 30, balloon = 30, inverse = 60, nightvision = 60, jail = 60, tiny = 60, jumpscare = 60, morph = 60 }
        local ALL_COMMANDS = { "balloon", "inverse", "jail", "jumpscare", "morph", "nightvision", "ragdoll", "rocket", "tiny" }
        local CMD_KNOWN = {}
        for _, c in ipairs(ALL_COMMANDS) do CMD_KNOWN[c] = true end
        local function _cmdEnabled(c) local a = Config.AdminAllowedCmds return not (a and a[c] == false) end
        local function _mergedOrder()
            local out, seen = {}, {}
            if type(Config.AdminCmdOrder) == "table" then
                for _, c in ipairs(Config.AdminCmdOrder) do
                    if type(c) == "string" and CMD_KNOWN[c] and not seen[c] then seen[c] = true; out[#out + 1] = c end
                end
            end
            for _, c in ipairs(ALL_COMMANDS) do if not seen[c] then seen[c] = true; out[#out + 1] = c end end
            return out
        end
        local function getOrderedCommands()
            local out = {}
            for _, c in ipairs(_mergedOrder()) do if _cmdEnabled(c) then out[#out + 1] = c end end
            return out
        end
        local function buildCmdSequence(randomize)
            local order = getOrderedCommands()
            if not randomize or #order < 2 then return order end
            local pinned = (type(Config.AdminCmdPinned) == "table") and Config.AdminCmdPinned or nil
            local seq, freeIdx, pool = {}, {}, {}
            for i, cmd in ipairs(order) do
                if pinned and pinned[cmd] then seq[i] = cmd
                else seq[i] = false; freeIdx[#freeIdx + 1] = i; pool[#pool + 1] = cmd end
            end
            local rng = Random.new()
            for i = #pool, 2, -1 do local j = rng:NextInteger(1, i); pool[i], pool[j] = pool[j], pool[i] end
            for k, idx in ipairs(freeIdx) do seq[idx] = pool[k] end
            return seq
        end

        -- ── blacklist + anti-AP ─────────────────────────────────────────
        local BlacklistedPlayers = Config.Blacklist
        local function _blIndex(username)
            if not username then return nil end
            local lower = string.lower(tostring(username))
            for i, v in ipairs(BlacklistedPlayers) do if string.lower(tostring(v)) == lower then return i end end
            return nil
        end
        local function isBlacklisted(username) return _blIndex(username) ~= nil end
        local function _detector(gate, probe)
            return function(username)
                if gate() == false then return false end
                if not username then return false end
                local ok, result = pcall(probe, username)
                return ok and result == true
            end
        end
        local isAtlasUser = _detector(function() return Config.AntiApAtlasUsers end, function(username)
            local folder = workspace:FindFirstChild("AtlasESPFolder")
            local pf = folder and folder:FindFirstChild(tostring(username) .. "_ESP")
            local hl = pf and pf:FindFirstChildWhichIsA("Highlight")
            if not hl then return false end
            local c = hl.FillColor
            return math.abs(c.R - 1) < 0.02 and math.abs(c.G - (215 / 255)) < 0.02 and math.abs(c.B - 0) < 0.02
        end)
        local isKawaifuUser = _detector(function() return Config.AntiApKawaifuUsers end, function(username)
            local p = Players:FindFirstChild(username)
            local char = p and p.Character
            return char ~= nil and char:FindFirstChild("KaWaifu_NeonHighlight") ~= nil
        end)
        local _btScan = { host = nil, t = 0 }
        local function _btHueOk(hl)
            local f, o = hl.FillColor, hl.OutlineColor
            return math.abs(f.R - 100 / 255) < 0.02 and math.abs(f.G - 0 / 255) < 0.02 and math.abs(f.B - 220 / 255) < 0.02
               and math.abs(o.R - 170 / 255) < 0.02 and math.abs(o.G - 80 / 255) < 0.02 and math.abs(o.B - 255 / 255) < 0.02
        end
        local isBraintopiaUser = _detector(function() return Config.AntiApBraintopiaUsers end, function(username)
            local p = Players:FindFirstChild(username)
            local char = p and p.Character
            if not char then return false end
            local hl = char:FindFirstChild("BT_ESP", true)
            if hl and hl:IsA("Highlight") and _btHueOk(hl) then return true end
            if char:FindFirstChild("BT_ESP_Label", true) then return true end
            local host = _btScan.host
            if not (host and host.Parent) then
                local now = os.clock()
                if now - _btScan.t < 3 then return false end
                _btScan.t = now
                local cg = game:GetService("CoreGui")
                host = cg:FindFirstChild("BT_ESP") and cg or nil
                if not host then for _, a in ipairs(cg:GetChildren()) do if a:FindFirstChild("BT_ESP") then host = a; break end end end
                _btScan.host = host
                if not host then return false end
            end
            for _, ch in ipairs(host:GetChildren()) do
                if ch.Name == "BT_ESP" and ch:IsA("Highlight") and ch.Adornee == char and _btHueOk(ch) then return true end
            end
            return false
        end)
        local function getAutojoinerName(username)
            return (isAtlasUser(username) and "Atlas") or (isKawaifuUser(username) and "Kawaifu")
                or (isBraintopiaUser(username) and "Braintopia") or nil
        end
        local function canUseAdminAction(targetPlayer)
            if not targetPlayer then return false end
            if isBlacklisted(targetPlayer.Name) then return false end
            return getAutojoinerName(targetPlayer.Name) == nil
        end
        _G.RyPrivIsBlacklisted = isBlacklisted

        -- ── GUI shell ────────────────────────────────────────────────────
        local host = (gethui and gethui()) or game:GetService("CoreGui")
        pcall(function() local old = host:FindFirstChild("RyPrivAdminUI") if old then old:Destroy() end end)
        local adminGui = new("ScreenGui", nil, {
            Name = "RyPrivAdminUI", ResetOnSpawn = false, DisplayOrder = 999994,
            ZIndexBehavior = Enum.ZIndexBehavior.Sibling, Enabled = not Config.HideAdminPanel,
        })
        pcall(function() adminGui.Parent = host end)
        if not adminGui.Parent then adminGui.Parent = PlayerGui end

        local frame = new("Frame", adminGui, {
            Size = UDim2.new(0, 640, 0, 340),
            Position = UDim2.fromOffset(tonumber(_G._ryPriv_adX) or 120, tonumber(_G._ryPriv_adY) or 200),
            BackgroundColor3 = Theme.Background, BackgroundTransparency = A(), BorderSizePixel = 0,
            Active = true,
        })
        local listFrame, layout, blFrame
        stroke(frame, Theme.Stroke, 1, 0.35)
        -- follows the Settings -> UI Transparency dial like the other panels
        task.spawn(function()
            while frame.Parent do
                task.wait(0.3)
                if adminGui.Enabled then frame.BackgroundTransparency = A() end
            end
        end)
        -- header: same shape as the Utility panel's (accent tick, title, x)
        local header = new("TextButton", frame, {
            Size = UDim2.new(1, 0, 0, 34), BackgroundTransparency = 1,
            Text = "", AutoButtonColor = false, Active = true,
        })
        if _G.RyPrivPanelDrag then
            pcall(_G.RyPrivPanelDrag, frame, header, "_ryPriv_adX", "_ryPriv_adY")
            pcall(_G.RyPrivApplyPos, frame, "_ryPriv_adX", "_ryPriv_adY", nil)
            pcall(_G.RyPrivRegisterPanel, frame, "_ryPriv_adX", "_ryPriv_adY")
            -- the grip's saved size, restored the same way
            pcall(_G.RyPrivApplySize, frame, "_ryPriv_adW", "_ryPriv_adH", 420, 220, 1400, 1000)
        end
        new("Frame", header, {
            Size = UDim2.fromOffset(3, 14), Position = UDim2.new(0, 10, 0.5, -7),
            BackgroundColor3 = Theme.Accent1, BorderSizePixel = 0,
        })
        new("TextLabel", header, {
            Size = UDim2.new(1, -140, 1, 0), Position = UDim2.new(0, 19, 0, 0), BackgroundTransparency = 1,
            Text = "ADMIN", Font = Enum.Font.GothamBold, TextSize = 12,
            TextColor3 = Theme.TextPrimary, TextXAlignment = Enum.TextXAlignment.Left,
        })
        new("Frame", frame, {
            Size = UDim2.new(1, -20, 0, 1), Position = UDim2.new(0, 10, 0, 34),
            BackgroundColor3 = Theme.Stroke, BackgroundTransparency = 0.5, BorderSizePixel = 0,
        })
        local refreshBtn = new("TextButton", header, {
            Size = UDim2.fromOffset(70, 20), Position = UDim2.new(1, -104, 0.5, -10),
            BackgroundColor3 = Theme.SurfaceHighlight, BackgroundTransparency = 0.3,
            Text = "REFRESH", Font = Enum.Font.GothamBold, TextSize = 10, TextColor3 = Theme.Accent1,
            AutoButtonColor = false,
        })
        stroke(refreshBtn, Theme.Stroke, 1, 0.55)
        local closeX = new("TextButton", header, {
            Size = UDim2.fromOffset(20, 20), Position = UDim2.new(1, -28, 0.5, -10),
            BackgroundColor3 = Theme.SurfaceHighlight, BackgroundTransparency = 0.3,
            Text = "x", Font = Enum.Font.GothamBold, TextSize = 12, TextColor3 = Theme.TextSecondary,
            AutoButtonColor = false,
        })
        closeX.MouseButton1Click:Connect(function()
            adminGui.Enabled = false
            if type(_G.RyPrivUIHidden) == "table" then _G.RyPrivUIHidden.admin = true end
            pcall(_G.RyPrivSaveSettings)
        end)

        -- resize grip (bottom-right)
        do
            local grip = new("TextButton", frame, {
                Size = UDim2.fromOffset(16, 16), Position = UDim2.new(1, -18, 1, -18),
                BackgroundTransparency = 1, Text = "◢", TextSize = 12, TextColor3 = Theme.Accent2,
                AutoButtonColor = false, ZIndex = 20,
            })
            -- This used to resize the panel and then throw the result away:
            -- there was no key, no save call, and a `local sc = 1` that every
            -- dimension was pointlessly divided by.
            _G.RyPrivPanelResize(frame, grip, "_ryPriv_adW", "_ryPriv_adH", 420, 220, 1400, 1000)
        end


        -- ── fire through the game's real AdminPanel ─────────────────────
        local function fireClick(button)
            if not button then return end
            if firesignal then
                pcall(firesignal, button.MouseButton1Click)
                pcall(firesignal, button.MouseButton1Down)
                pcall(firesignal, button.Activated)
            elseif VIM then
                local x = button.AbsolutePosition.X + (button.AbsoluteSize.X / 2)
                local y = button.AbsolutePosition.Y + (button.AbsoluteSize.Y / 2) + 58
                VIM:SendMouseButtonEvent(x, y, 0, true, game, 0)
                VIM:SendMouseButtonEvent(x, y, 0, false, game, 0)
            end
        end
        local function runAdminCommand(targetPlayer, commandName, selfAction)
            local _isSelf = (selfAction == true) and targetPlayer == LP
            if not _isSelf then
                if targetPlayer and isBlacklisted(targetPlayer.Name) then return false end
                if targetPlayer and getAutojoinerName(targetPlayer.Name) then return false end
            end
            local realAdminGui = PlayerGui:WaitForChild("AdminPanel", 5)
            if not realAdminGui then return false end
            local ok, profilesScroll = pcall(function()
                return realAdminGui:WaitForChild("AdminPanel"):WaitForChild("Profiles"):WaitForChild("ScrollingFrame")
            end)
            if not ok or not profilesScroll then return false end
            local playerBtn = profilesScroll:FindFirstChild(targetPlayer.Name)
            if not playerBtn then return false end
            fireClick(playerBtn)
            task.wait(0.05)
            local ok2, contentScroll = pcall(function()
                return realAdminGui.AdminPanel:WaitForChild("Content"):WaitForChild("ScrollingFrame")
            end)
            if not ok2 or not contentScroll then return false end
            local cmdBtn = contentScroll:FindFirstChild(commandName)
            if not cmdBtn then return false end
            fireClick(cmdBtn)
            return true
        end
        _G.runAdminCommand = runAdminCommand
        local activeCooldowns = {}
        local function isOnCooldown(cmd)
            local t = PlayerGui:FindFirstChild("AdminPanel")
            t = t and t:FindFirstChild("AdminPanel"); t = t and t:FindFirstChild("Content")
            t = t and t:FindFirstChild("ScrollingFrame"); t = t and t:FindFirstChild(cmd)
            t = t and t:FindFirstChild("Timer")
            if t then return t.Visible end
            if not activeCooldowns[cmd] then return false end
            return (tick() - activeCooldowns[cmd]) < (COOLDOWNS[cmd] or 0)
        end
        _G.isOnCooldown = isOnCooldown
        local function getNextAvailableCommand()
            for _, cmd in ipairs(getOrderedCommands()) do if not isOnCooldown(cmd) then return cmd end end
            return nil
        end
        local function setGlobalVisualCooldown(cmd)
            if SharedState.AdminButtonCache[cmd] then
                for _, b in ipairs(SharedState.AdminButtonCache[cmd]) do
                    if b and b.Parent then
                        b.BackgroundColor3 = Theme.Error
                        task.delay(COOLDOWNS[cmd] or 5, function()
                            if b and b.Parent then
                                local hasBallooned = (cmd == "balloon" and next(SharedState.BalloonedPlayers) ~= nil)
                                b.BackgroundColor3 = hasBallooned and Theme.Error or Theme.SurfaceHighlight
                            end
                        end)
                    end
                end
            end
        end
        local function updateBalloonButtons()
            local hasBallooned = next(SharedState.BalloonedPlayers) ~= nil
            for _, b in ipairs(SharedState.AdminButtonCache["balloon"] or {}) do
                if b and b.Parent then b.BackgroundColor3 = hasBallooned and Theme.Error or Theme.SurfaceHighlight end
            end
        end
        local function noteCmdFired(plr, cmd)
            activeCooldowns[cmd] = tick()
            setGlobalVisualCooldown(cmd)
            if cmd == "balloon" then SharedState.BalloonedPlayers[plr.UserId] = true updateBalloonButtons() end
        end
        local function triggerAll(plr)
            if not canUseAdminAction(plr) then return end
            local count = 0
            for _, cmd in ipairs(buildCmdSequence(Config.AdminRandomCmd == true)) do
                if not isOnCooldown(cmd) then
                    task.delay(count * 0.1, function()
                        if runAdminCommand(plr, cmd) then noteCmdFired(plr, cmd) end
                    end)
                    count = count + 1
                end
            end
        end
        _G.triggerAll = triggerAll
        local _cycle = {}
        local function nextCycleCmd()
            for _ = 1, 2 do
                while #_cycle > 0 do local cmd = table.remove(_cycle, 1) if not isOnCooldown(cmd) then return cmd end end
                _cycle = buildCmdSequence(Config.AdminRandomCmd == true)
                if #_cycle == 0 then return nil end
            end
            return nil
        end
        local function fireRandomCmd(plr)
            if not canUseAdminAction(plr) then return end
            local cmd = nextCycleCmd()
            if not cmd then return end
            if runAdminCommand(plr, cmd) then noteCmdFired(plr, cmd) return cmd end
        end
        local function apAct(plr)
            if Config.AdminRandomCmd then fireRandomCmd(plr) else triggerAll(plr) end
        end
        local _spamBusy = false
        local function spamOnce(plr, onDone)
            if _spamBusy then return false end
            if not plr or not plr.Parent then return false end
            if not canUseAdminAction(plr) then return false end
            _spamBusy = true
            task.spawn(function()
                for _, cmd in ipairs(buildCmdSequence(Config.AdminRandomCmd == true)) do
                    if not isOnCooldown(cmd) then
                        local ok, res = pcall(runAdminCommand, plr, cmd)
                        if ok and res then noteCmdFired(plr, cmd) end
                        task.wait(0.15)
                    end
                end
                _spamBusy = false
                if onDone then pcall(onDone) end
            end)
            return true
        end

        -- nearest other base's owner (PlotSign distance), for Spam Base Owner /
        -- Nightvision Owner
        local function nearestBaseOwner()
            local hrp = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
            if not hrp then return nil end
            local nearestPlot, nearestDist = nil, math.huge
            local Plots = workspace:FindFirstChild("Plots")
            if Plots then
                for _, plot in ipairs(Plots:GetChildren()) do
                    local sign = plot:FindFirstChild("PlotSign")
                    if sign then
                        local yourBase = sign:FindFirstChild("YourBase")
                        if not yourBase or not yourBase.Enabled then
                            local signPos = sign:IsA("BasePart") and sign.Position or (sign.PrimaryPart and sign.PrimaryPart.Position)
                            if not signPos then local part = sign:FindFirstChildWhichIsA("BasePart", true) signPos = part and part.Position end
                            if signPos then
                                local dist = (hrp.Position - signPos).Magnitude
                                if dist < nearestDist then nearestDist = dist nearestPlot = plot end
                            end
                        end
                    end
                end
            end
            if not nearestPlot then return nil end
            local targetPlayer
            pcall(function()
                local ch = getPlotChannel(nearestPlot.Name)
                local owner = ch and channelGet(ch, "Owner")
                if typeof(owner) == "Instance" and owner:IsA("Player") then targetPlayer = owner
                elseif type(owner) == "table" and owner.UserId then targetPlayer = Players:GetPlayerByUserId(owner.UserId)
                elseif type(owner) == "string" then targetPlayer = Players:FindFirstChild(owner) end
            end)
            if not targetPlayer then
                local nick = _plotOwnerName(nearestPlot)
                if nick then
                    for _, p in ipairs(Players:GetPlayers()) do
                        if p.DisplayName == nick or p.Name == nick then targetPlayer = p break end
                    end
                end
            end
            if not targetPlayer or targetPlayer == LP then return nil end
            if isBlacklisted(targetPlayer.Name) or getAutojoinerName(targetPlayer.Name) then return nil end
            return targetPlayer
        end
        -- ── AUTO AP (own panel) ─────────────────────────────────────────
        -- Four switches that used to be one-shot buttons on the admin panel:
        --   CLICK TO AP         click a player in the world to fire at them
        --   PROX AP             fire at anyone inside Prox range
        --   SPAM BASE OWNER     keep spamming the owner of the base you're nearest
        --   NIGHTVISION OWNER   keep nightvision on the owner of that base
        -- All four skip blacklisted / Anti-AP players and respect cooldowns.
        ProximityAPActive = Config.AutoProxAP == true
        task.spawn(function()
            while true do
                task.wait(0.3)
                if Config.AutoSpamOwner and not _spamBusy and getNextAvailableCommand() then
                    local t = nearestBaseOwner()
                    if t then spamOnce(t) end
                end
            end
        end)
        local _nightBusy = false
        task.spawn(function()
            while true do
                task.wait(0.3)
                if Config.AutoNightOwner and not _nightBusy and not _spamBusy
                    and not isOnCooldown("nightvision") and _cmdEnabled("nightvision") then
                    local t = nearestBaseOwner()
                    if t and canUseAdminAction(t) then
                        _nightBusy = true
                        local ok, res = pcall(runAdminCommand, t, "nightvision")
                        if ok and res then noteCmdFired(t, "nightvision") end
                        _nightBusy = false
                    end
                end
            end
        end)

        do
            pcall(function() local old = host:FindFirstChild("RyPrivAutoAPUI") if old then old:Destroy() end end)
            local apGui = new("ScreenGui", nil, {
                Name = "RyPrivAutoAPUI", ResetOnSpawn = false, DisplayOrder = 999994,
                ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
            })
            pcall(function() apGui.Parent = host end)
            if not apGui.Parent then apGui.Parent = PlayerGui end
            local ap = new("Frame", apGui, {
                Size = UDim2.fromOffset(212, 0), AutomaticSize = Enum.AutomaticSize.Y,
                Position = UDim2.fromOffset(tonumber(_G._ryPriv_apX) or 470, tonumber(_G._ryPriv_apY) or 300),
                BackgroundColor3 = Theme.Background, BackgroundTransparency = A(),
                BorderSizePixel = 0, Active = true,
            })
            stroke(ap, Theme.Stroke, 1, 0.35)
            local apHead = new("TextButton", ap, {
                Size = UDim2.new(1, 0, 0, 34), BackgroundTransparency = 1,
                Text = "", AutoButtonColor = false, Active = true,
            })
            new("Frame", apHead, {
                Size = UDim2.fromOffset(3, 14), Position = UDim2.new(0, 10, 0.5, -7),
                BackgroundColor3 = Theme.Accent1, BorderSizePixel = 0,
            })
            new("TextLabel", apHead, {
                Size = UDim2.new(1, -60, 1, 0), Position = UDim2.new(0, 19, 0, 0),
                BackgroundTransparency = 1, Text = "AUTO AP", Font = Enum.Font.GothamBold, TextSize = 12,
                TextColor3 = Theme.TextPrimary, TextXAlignment = Enum.TextXAlignment.Left,
            })
            local apClose = new("TextButton", apHead, {
                Size = UDim2.fromOffset(20, 20), Position = UDim2.new(1, -28, 0.5, -10),
                BackgroundColor3 = Theme.SurfaceHighlight, BackgroundTransparency = 0.3,
                Text = "x", Font = Enum.Font.GothamBold, TextSize = 12, TextColor3 = Theme.TextSecondary,
                AutoButtonColor = false,
            })
            apClose.MouseButton1Click:Connect(function()
                apGui.Enabled = false
                if type(_G.RyPrivUIHidden) == "table" then _G.RyPrivUIHidden.autoap = true end
                pcall(_G.RyPrivSaveSettings)
            end)
            pcall(_G.RyPrivPanelDrag, ap, apHead, "_ryPriv_apX", "_ryPriv_apY")
            pcall(_G.RyPrivApplyPos, ap, "_ryPriv_apX", "_ryPriv_apY", nil)
            pcall(_G.RyPrivRegisterPanel, ap, "_ryPriv_apX", "_ryPriv_apY")
            new("Frame", ap, {
                Size = UDim2.new(1, -20, 0, 1), Position = UDim2.new(0, 10, 0, 34),
                BackgroundColor3 = Theme.Stroke, BackgroundTransparency = 0.5, BorderSizePixel = 0,
            })
            local apBody = new("Frame", ap, {
                Size = UDim2.new(1, -20, 0, 0), AutomaticSize = Enum.AutomaticSize.Y,
                Position = UDim2.new(0, 10, 0, 40), BackgroundTransparency = 1,
            })
            new("UIListLayout", apBody, { SortOrder = Enum.SortOrder.LayoutOrder, Padding = UDim.new(0, 4) })
            new("UIPadding", apBody, { PaddingBottom = UDim.new(0, 10) })
            local paints = {}
            local function apToggle(order, label, get, set)
                local row = new("Frame", apBody, {
                    Size = UDim2.new(1, 0, 0, 30), BackgroundColor3 = Theme.SurfaceHighlight,
                    BorderSizePixel = 0, LayoutOrder = order,
                })
                local st = stroke(row, Theme.Stroke, 1, 0.5)
                new("TextLabel", row, {
                    Size = UDim2.new(1, -46, 1, 0), Position = UDim2.new(0, 10, 0, 0),
                    BackgroundTransparency = 1, Font = Enum.Font.GothamBold, TextSize = 11,
                    TextColor3 = Theme.TextPrimary, TextXAlignment = Enum.TextXAlignment.Left, Text = label,
                })
                local pill = new("Frame", row, {
                    Size = UDim2.fromOffset(28, 14), Position = UDim2.new(1, -38, 0.5, -7),
                    BackgroundColor3 = Theme.Stroke, BorderSizePixel = 0,
                })
                local knob = new("Frame", pill, {
                    Size = UDim2.fromOffset(10, 10), Position = UDim2.fromOffset(2, 2),
                    BackgroundColor3 = Color3.fromRGB(226, 231, 237), BorderSizePixel = 0,
                })
                local hit = new("TextButton", row, {
                    Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1, Text = "",
                    AutoButtonColor = false, Active = true,
                })
                local function paint()
                    local on = get() == true
                    pill.BackgroundColor3 = on and Theme.Accent1 or Theme.Stroke
                    knob.Position = on and UDim2.fromOffset(16, 2) or UDim2.fromOffset(2, 2)
                    row.BackgroundColor3 = on and Color3.fromRGB(58, 78, 96) or Theme.SurfaceHighlight
                    row.BackgroundTransparency = (on and 0.15 or 0.28) + A() * 0.45
                    st.Color = on and Theme.Accent1 or Theme.Stroke
                    st.Transparency = on and 0.3 or 0.55
                end
                hit.MouseButton1Click:Connect(function() pcall(set) SaveConfig() paint() end)
                paint()
                paints[#paints + 1] = paint
            end
            apToggle(1, "Click to AP", function() return Config.ClickToAP end, function()
                Config.ClickToAP = not Config.ClickToAP
                if Config.ClickToAP then Config.ClickToAPSingleCommand = true end
            end)
            apToggle(2, "Prox AP", function() return ProximityAPActive end, function()
                ProximityAPActive = not ProximityAPActive
                Config.AutoProxAP = ProximityAPActive
            end)
            apToggle(3, "Spam Base Owner", function() return Config.AutoSpamOwner end, function()
                Config.AutoSpamOwner = not Config.AutoSpamOwner
            end)
            apToggle(4, "Nightvision Owner", function() return Config.AutoNightOwner end, function()
                Config.AutoNightOwner = not Config.AutoNightOwner
            end)
            _G.RyPrivToggleProximityAP = function()
                ProximityAPActive = not ProximityAPActive
                Config.AutoProxAP = ProximityAPActive
                SaveConfig()
                for _, f in ipairs(paints) do pcall(f) end
            end
            if type(_G.RyPrivUIHidden) ~= "table" then _G.RyPrivUIHidden = {} end
            apGui.Enabled = _G.RyPrivUIHidden.autoap ~= true
            task.spawn(function()
                while ap.Parent do
                    task.wait(0.3)
                    if apGui.Enabled then
                        ap.BackgroundTransparency = A()
                        for _, f in ipairs(paints) do pcall(f) end
                    end
                end
            end)
        end

        -- prox circle
        local proxViz, _proxVizR = nil, false
        local PROXVIZ_ROT = CFrame.Angles(0, 0, math.rad(90))
        local PROXVIZ_OFF = Vector3.new(0, -2.5, 0)
        RunService.Heartbeat:Connect(function()
            if Config.ShowProxCircle then
                if not proxViz then
                    proxViz = new("Part", nil, {
                        Name = "Part", Anchored = true, CanCollide = false, CanQuery = false, CanTouch = false,
                        Shape = Enum.PartType.Cylinder, Color = Theme.Accent1, Transparency = 0.6, CastShadow = false,
                    })
                    proxViz.Parent = workspace
                    _proxVizR = false
                end
                local hrp = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
                if hrp then
                    local r = Config.ProximityRange
                    if r ~= _proxVizR then proxViz.Size = Vector3.new(0.5, r * 2, r * 2) _proxVizR = r end
                    proxViz.CFrame = hrp.CFrame * PROXVIZ_ROT + PROXVIZ_OFF
                end
            elseif proxViz then
                proxViz:Destroy(); proxViz = nil
            end
        end)

        -- ── tabs ─────────────────────────────────────────────────────────
        local tabBar = new("Frame", frame, {
            Size = UDim2.new(1, -20, 0, 30), Position = UDim2.new(0, 10, 0, 42),
            BackgroundTransparency = 1, BorderSizePixel = 0,
        })
        new("UIListLayout", tabBar, {
            FillDirection = Enum.FillDirection.Horizontal, Padding = UDim.new(0, 4),
            SortOrder = Enum.SortOrder.LayoutOrder, VerticalAlignment = Enum.VerticalAlignment.Center,
        })
        local function makeTabBtn(label, order)
            local btn = new("TextButton", tabBar, {
                LayoutOrder = order, Size = UDim2.new(0, 122, 0, 28), AutoButtonColor = false,
                Text = label, Font = Enum.Font.GothamBold, TextSize = 13, BorderSizePixel = 0,
            })
            corner(btn, 10)
            local s = new("UIStroke", btn, { Thickness = 1, Transparency = 0.4 })
            return btn, s
        end
        local tabPlayers, tabPlayersStroke = makeTabBtn("PLAYERS", 1)
        local tabSettings, tabSettingsStroke = makeTabBtn("SETTINGS", 2)
        local function setActiveTab(name)
            local isP = (name == "players")
            tabPlayers.BackgroundColor3 = isP and Theme.SurfaceHighlight or Theme.Surface
            tabPlayers.TextColor3 = isP and Theme.Accent1 or Theme.TextSecondary
            tabSettings.BackgroundColor3 = (not isP) and Theme.SurfaceHighlight or Theme.Surface
            tabSettings.TextColor3 = (not isP) and Theme.Accent1 or Theme.TextSecondary
            tabPlayersStroke.Color = isP and Theme.Accent1 or Theme.SurfaceHighlight
            tabPlayersStroke.Transparency = isP and 0.35 or 0.75
            tabSettingsStroke.Color = (not isP) and Theme.Accent1 or Theme.SurfaceHighlight
            tabSettingsStroke.Transparency = (not isP) and 0.35 or 0.75
            listFrame.Visible = isP
            blFrame.Visible = not isP
        end
        tabPlayers.MouseButton1Click:Connect(function() setActiveTab("players") end)
        tabSettings.MouseButton1Click:Connect(function() setActiveTab("settings") end)

        -- ── SETTINGS tab ─────────────────────────────────────────────────
        blFrame = new("ScrollingFrame", frame, {
            Size = UDim2.new(1, -20, 1, -86), Position = UDim2.new(0, 10, 0, 78),
            BackgroundTransparency = 1, BorderSizePixel = 0, ZIndex = 5, Active = true, Visible = false,
            ScrollBarThickness = 3, ScrollBarImageColor3 = Theme.TextSecondary,
            CanvasSize = UDim2.new(0, 0, 0, 0), AutomaticCanvasSize = Enum.AutomaticSize.Y,
        })
        new("UIListLayout", blFrame, { Padding = UDim.new(0, 5), SortOrder = Enum.SortOrder.LayoutOrder })
        new("UIPadding", blFrame, { PaddingRight = UDim.new(0, 6), PaddingBottom = UDim.new(0, 10) })
        local function secHeader(text, order)
            local r = new("Frame", blFrame, { Size = UDim2.new(1, 0, 0, 20), BackgroundTransparency = 1, LayoutOrder = order })
            new("Frame", r, { Size = UDim2.new(1, 0, 0, 1), Position = UDim2.new(0, 0, 1, -1), BackgroundColor3 = Theme.SurfaceHighlight, BorderSizePixel = 0 })
            new("TextLabel", r, {
                Size = UDim2.new(1, 0, 1, 0), BackgroundTransparency = 1, Text = text, Font = Enum.Font.GothamBold,
                TextSize = 12, TextColor3 = Theme.TextSecondary, TextXAlignment = Enum.TextXAlignment.Left,
            })
            return r
        end
        local SW_OFF = Color3.fromRGB(92, 99, 110)
        local function cfgToggle(label, get, set, order)
            local row = new("Frame", blFrame, {
                Size = UDim2.new(1, 0, 0, 34), BackgroundColor3 = Theme.Surface,
                BackgroundTransparency = 0.08, BorderSizePixel = 0, LayoutOrder = order,
            })
            corner(row, 7)
            new("TextLabel", row, {
                Size = UDim2.new(1, -50, 1, 0), Position = UDim2.new(0, 11, 0, 0), BackgroundTransparency = 1,
                Text = label, Font = Enum.Font.Gotham, TextSize = 12, TextColor3 = Theme.TextPrimary,
                TextXAlignment = Enum.TextXAlignment.Left, TextTruncate = Enum.TextTruncate.AtEnd,
            })
            local rst = stroke(row, Theme.Stroke, 1, 0.5)
            local sw = new("Frame", row, { Size = UDim2.fromOffset(28, 14), Position = UDim2.new(1, -38, 0.5, -7), BackgroundColor3 = SW_OFF, BorderSizePixel = 0 })
            local dot = new("Frame", sw, { Size = UDim2.fromOffset(10, 10), Position = UDim2.fromOffset(2, 2), BackgroundColor3 = Color3.fromRGB(226, 231, 237), BorderSizePixel = 0 })
            local function paint()
                local on = get() == true
                sw.BackgroundColor3 = on and Theme.Accent1 or SW_OFF
                dot.Position = on and UDim2.fromOffset(16, 2) or UDim2.fromOffset(2, 2)
                row.BackgroundColor3 = on and Color3.fromRGB(58, 78, 96) or Theme.SurfaceHighlight
                rst.Color = on and Theme.Accent1 or Theme.Stroke
                rst.Transparency = on and 0.3 or 0.55
            end
            local hit = new("TextButton", row, { Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1, Text = "", AutoButtonColor = false })
            hit.MouseButton1Click:Connect(function() set(not (get() == true)) SaveConfig() paint() end)
            paint()
            return row
        end
        secHeader("ADMIN", 1)
        cfgToggle("Random Command", function() return Config.AdminRandomCmd end, function(v) Config.AdminRandomCmd = v end, 2)
        cfgToggle("Click AP: One Command Per Click", function() return Config.ClickToAPSingleCommand end, function(v) Config.ClickToAPSingleCommand = v end, 3)
        cfgToggle("Show Prox Circle", function() return Config.ShowProxCircle end, function(v) Config.ShowProxCircle = v end, 4)
        do  -- Prox Circle Studs slider (5..50)
            local row = new("Frame", blFrame, {
                Size = UDim2.new(1, 0, 0, 44), BackgroundColor3 = Theme.Surface,
                BackgroundTransparency = 0.08, BorderSizePixel = 0, LayoutOrder = 5,
            })
            corner(row, 7)
            new("TextLabel", row, {
                Size = UDim2.new(1, -70, 0, 18), Position = UDim2.new(0, 11, 0, 4), BackgroundTransparency = 1,
                Text = "Prox Circle Studs", Font = Enum.Font.Gotham, TextSize = 12,
                TextColor3 = Theme.TextPrimary, TextXAlignment = Enum.TextXAlignment.Left,
            })
            local val = new("TextLabel", row, {
                Size = UDim2.new(0, 50, 0, 18), Position = UDim2.new(1, -60, 0, 4), BackgroundTransparency = 1,
                Text = tostring(Config.ProximityRange), Font = Enum.Font.GothamBold, TextSize = 12,
                TextColor3 = Theme.Accent1, TextXAlignment = Enum.TextXAlignment.Right,
            })
            local trk = new("TextButton", row, {
                Size = UDim2.new(1, -22, 0, 6), Position = UDim2.new(0, 11, 0, 30), BackgroundColor3 = Theme.SurfaceHighlight,
                Text = "", AutoButtonColor = false, BorderSizePixel = 0,
            })
            round(trk)
            local fill = new("Frame", trk, { Size = UDim2.new(0, 0, 1, 0), BackgroundColor3 = Theme.Accent1, BorderSizePixel = 0 })
            round(fill)
            local function refresh()
                fill.Size = UDim2.new((Config.ProximityRange - 5) / 45, 0, 1, 0)
                val.Text = tostring(Config.ProximityRange)
            end
            local sliding = false
            local function at(x)
                local p = math.clamp((x - trk.AbsolutePosition.X) / math.max(trk.AbsoluteSize.X, 1), 0, 1)
                Config.ProximityRange = math.floor(5 + 45 * p + 0.5)
                refresh()
            end
            trk.InputBegan:Connect(function(i)
                if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
                    sliding = true at(i.Position.X)
                end
            end)
            UIS.InputChanged:Connect(function(i)
                if sliding and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then at(i.Position.X) end
            end)
            UIS.InputEnded:Connect(function(i)
                if sliding and (i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch) then
                    sliding = false SaveConfig()
                end
            end)
            refresh()
        end
        -- Allowed Commands (collapsible)
        do
            local hdr = new("TextButton", blFrame, {
                Size = UDim2.new(1, 0, 0, 30), BackgroundColor3 = Theme.SurfaceHighlight, BorderSizePixel = 0,
                LayoutOrder = 6, AutoButtonColor = false, Font = Enum.Font.GothamBold, TextSize = 12,
                TextColor3 = Theme.Accent1, TextXAlignment = Enum.TextXAlignment.Left, Text = "  ▶  Allowed Commands",
            })
            corner(hdr, 7)
            local rows = {}
            for i, cmd in ipairs(ALL_COMMANDS) do
                local r = cfgToggle(cmd:sub(1, 1):upper() .. cmd:sub(2),
                    function() return not (Config.AdminAllowedCmds and Config.AdminAllowedCmds[cmd] == false) end,
                    function(v) Config.AdminAllowedCmds[cmd] = v end, 7 + i)
                r.Visible = false
                rows[#rows + 1] = r
            end
            local open = false
            hdr.MouseButton1Click:Connect(function()
                open = not open
                for _, r in ipairs(rows) do r.Visible = open end
                hdr.Text = (open and "  ▼  " or "  ▶  ") .. "Allowed Commands"
            end)
        end
        -- Command Order 
        do
            local rowH, hdrH, crn, tSize = 28, 30, 7, 12
            local hdr = new("TextButton", blFrame, {
                Size = UDim2.new(1, 0, 0, hdrH), BackgroundColor3 = Theme.SurfaceHighlight, BorderSizePixel = 0,
                LayoutOrder = 20, AutoButtonColor = false, Font = Enum.Font.GothamBold, TextSize = tSize,
                TextColor3 = Theme.Accent1, TextXAlignment = Enum.TextXAlignment.Left, Text = "  ▶  Command Order",
            })
            corner(hdr, crn)
            local holder = new("Frame", blFrame, {
                BackgroundTransparency = 1, BorderSizePixel = 0, LayoutOrder = 21, Visible = false, Size = UDim2.new(1, 0, 0, 0),
            })
            new("UIListLayout", holder, { Padding = UDim.new(0, 6), SortOrder = Enum.SortOrder.LayoutOrder })
            local open, refresh = false, nil
            refresh = function()
                for _, c in ipairs(holder:GetChildren()) do if not c:IsA("UIListLayout") then c:Destroy() end end
                local order = _mergedOrder()
                Config.AdminCmdOrder = order
                new("TextLabel", holder, {
                    Size = UDim2.new(1, 0, 0, 24), BackgroundTransparency = 1, LayoutOrder = 0,
                    Text = "  Arrows set the fire order. PIN locks a slot in Random mode.",
                    Font = Enum.Font.Gotham, TextSize = 11, TextColor3 = Color3.fromRGB(150, 150, 165),
                    TextXAlignment = Enum.TextXAlignment.Left, TextWrapped = true,
                })
                local bH = rowH - 8
                for i, cmd in ipairs(order) do
                    local enabled = _cmdEnabled(cmd)
                    local pinned = (type(Config.AdminCmdPinned) == "table") and Config.AdminCmdPinned[cmd] == true
                    local r = new("Frame", holder, { Size = UDim2.new(1, 0, 0, rowH), BackgroundColor3 = Theme.Surface, BorderSizePixel = 0, LayoutOrder = i })
                    corner(r, crn)
                    new("TextLabel", r, {
                        Size = UDim2.new(1, -116, 1, 0), Position = UDim2.new(0, 10, 0, 0), BackgroundTransparency = 1,
                        Text = i .. ".  " .. cmd:sub(1, 1):upper() .. cmd:sub(2) .. (enabled and "" or "   (off)"),
                        Font = Enum.Font.GothamBold, TextSize = tSize,
                        TextColor3 = enabled and Theme.TextPrimary or Color3.fromRGB(120, 120, 132),
                        TextXAlignment = Enum.TextXAlignment.Left,
                    })
                    local function move(delta)
                        local j = i + delta
                        if j < 1 or j > #order then return end
                        order[i], order[j] = order[j], order[i]
                        SaveConfig()
                        refresh()
                    end
                    local function mkBtn(txt, xOff, w, on)
                        local b = new("TextButton", r, {
                            Size = UDim2.new(0, w, 0, bH), Position = UDim2.new(1, xOff, 0.5, -bH / 2), AutoButtonColor = false,
                            BorderSizePixel = 0, Text = txt, Font = Enum.Font.GothamBold, TextSize = 11,
                            BackgroundColor3 = on and Theme.Accent1 or Theme.SurfaceHighlight,
                            TextColor3 = on and OnAccent() or Theme.TextPrimary,
                        })
                        corner(b, 5)
                        return b
                    end
                    mkBtn("▲", -92, 22, false).MouseButton1Click:Connect(function() move(-1) end)
                    mkBtn("▼", -66, 22, false).MouseButton1Click:Connect(function() move(1) end)
                    mkBtn("PIN", -40, 34, pinned).MouseButton1Click:Connect(function()
                        if type(Config.AdminCmdPinned) ~= "table" then Config.AdminCmdPinned = {} end
                        Config.AdminCmdPinned[cmd] = (not pinned) or nil
                        SaveConfig()
                        refresh()
                    end)
                end
                holder.Size = UDim2.new(1, 0, 0, 24 + 6 + #order * (rowH + 6))
            end
            hdr.MouseButton1Click:Connect(function()
                open = not open
                if open then refresh() end
                holder.Visible = open
                hdr.Text = (open and "  ▼  " or "  ▶  ") .. "Command Order"
            end)
        end
        secHeader("ANTI-AP", 30)
        cfgToggle("Anti-AP Atlas Users", function() return Config.AntiApAtlasUsers end, function(v) Config.AntiApAtlasUsers = v end, 31)
        cfgToggle("Anti-AP Kawaifu Users", function() return Config.AntiApKawaifuUsers end, function(v) Config.AntiApKawaifuUsers = v end, 32)
        cfgToggle("Anti-AP Braintopia Users", function() return Config.AntiApBraintopiaUsers end, function(v) Config.AntiApBraintopiaUsers = v end, 33)

        secHeader("BLACKLIST", 40)
        local blAddRow = new("Frame", blFrame, { Name = "BLAddRow", LayoutOrder = 41, Size = UDim2.new(1, 0, 0, 28), BackgroundTransparency = 1 })
        local blInput = new("TextBox", blAddRow, {
            Size = UDim2.new(1, -58, 0, 26), ZIndex = 6, BackgroundColor3 = Theme.Surface, BorderSizePixel = 0,
            Text = "", PlaceholderText = "Roblox username...", Font = Enum.Font.Gotham, TextSize = 13,
            TextColor3 = Theme.TextPrimary, PlaceholderColor3 = Theme.TextSecondary, ClearTextOnFocus = false,
        })
        corner(blInput, 6)
        stroke(blInput, Theme.SurfaceHighlight, 1, 0.3)
        new("UIPadding", blInput, { PaddingLeft = UDim.new(0, 8) })
        local blAddBtn = new("TextButton", blAddRow, {
            Size = UDim2.new(0, 50, 0, 26), Position = UDim2.new(1, -50, 0, 0), ZIndex = 6,
            BackgroundColor3 = Theme.Error, Text = "ADD", Font = Enum.Font.GothamBold, TextSize = 13,
            TextColor3 = OnAccent(), AutoButtonColor = false,
        })
        corner(blAddBtn, 6)
        stroke(blAddBtn, Theme.Error, 1, 0.4)
        local rebuildBlList
        rebuildBlList = function()
            for _, c in ipairs(blFrame:GetChildren()) do if c.Name == "BLEntry" then c:Destroy() end end
            local rowBg = Color3.fromRGB(math.floor(Theme.Error.R * 255 * 0.35), math.floor(Theme.Error.G * 255 * 0.35), math.floor(Theme.Error.B * 255 * 0.35))
            for i, name in ipairs(BlacklistedPlayers) do
                local row = new("Frame", blFrame, { Name = "BLEntry", LayoutOrder = 100 + i, Size = UDim2.new(1, 0, 0, 28), BackgroundColor3 = rowBg, BorderSizePixel = 0 })
                corner(row, 6)
                stroke(row, Theme.Error, 1, 0.5)
                new("TextLabel", row, {
                    Size = UDim2.new(1, -36, 1, 0), Position = UDim2.new(0, 10, 0, 0), BackgroundTransparency = 1,
                    Text = name, Font = Enum.Font.GothamBold, TextSize = 13, TextColor3 = Theme.TextPrimary,
                    TextXAlignment = Enum.TextXAlignment.Left,
                })
                local removeBtn = new("TextButton", row, {
                    Size = UDim2.new(0, 26, 0, 20), Position = UDim2.new(1, -30, 0.5, -10),
                    BackgroundColor3 = Theme.Error, Text = "X", Font = Enum.Font.GothamBold, TextSize = 12,
                    TextColor3 = OnAccent(), AutoButtonColor = false,
                })
                corner(removeBtn, 4)
                local capName = name
                removeBtn.MouseButton1Click:Connect(function()
                    for j, n in ipairs(BlacklistedPlayers) do
                        if n:lower() == capName:lower() then table.remove(BlacklistedPlayers, j) break end
                    end
                    SaveConfig()
                    rebuildBlList()
                end)
            end
        end
        local function addBlacklistUser(username)
            username = username:match("^%s*(.-)%s*$")
            if username == "" or _blIndex(username) then return end
            table.insert(BlacklistedPlayers, username)
            SaveConfig()
            rebuildBlList()
            blInput.Text = ""
        end
        blAddBtn.MouseButton1Click:Connect(function() addBlacklistUser(blInput.Text) end)
        blInput.FocusLost:Connect(function(enter) if enter then addBlacklistUser(blInput.Text) end end)
        rebuildBlList()

        -- ── PLAYERS tab ──────────────────────────────────────────────────
        listFrame = new("ScrollingFrame", frame, {
            Size = UDim2.new(1, -20, 1, -86), Position = UDim2.new(0, 10, 0, 78),
            BackgroundTransparency = 1, BorderSizePixel = 0, ScrollBarThickness = 3,
            ScrollBarImageColor3 = Theme.Accent1, ScrollBarImageTransparency = 0.45,
        })
        layout = new("UIListLayout", listFrame, { Padding = UDim.new(0, 6), SortOrder = Enum.SortOrder.LayoutOrder })
        setActiveTab("players")

        local function priorityIndexOf(brainrotIndex)
            local disp = displayNameOf(brainrotIndex) or brainrotIndex
            local list = _G.SHARED_PRIORITY_ITEMS
            if type(list) ~= "table" then return nil end
            for i, pName in ipairs(list) do
                if pName == brainrotIndex or pName == disp then return i end
            end
            return nil
        end
        local function getSortKey(plr)
            if not plr or not plr.Parent then return 3, 9999, "" end
            if not plr:GetAttribute("Stealing") then return 3, 9999, plr.Name or "" end
            local idx = priorityIndexOf(plr:GetAttribute("StealingIndex"))
            if idx then return 1, idx, plr.Name or "" end
            return 2, 9999, plr.Name or ""
        end
        local function sortList()
            if not (listFrame.Visible and adminGui.Enabled and frame.Visible) then return end
            local rows = {}
            for _, child in ipairs(listFrame:GetChildren()) do
                if child:IsA("TextButton") and child.Name ~= "" then
                    local plr = Players:FindFirstChild(child.Name)
                    if plr then
                        local t, p, n = getSortKey(plr)
                        rows[#rows + 1] = { row = child, t = t, p = p, n = n or "" }
                    end
                end
            end
            table.sort(rows, function(a, b)
                if a.t ~= b.t then return a.t < b.t end
                if a.p ~= b.p then return a.p < b.p end
                return a.n < b.n
            end)
            for i, e in ipairs(rows) do if e.row.LayoutOrder ~= i then e.row.LayoutOrder = i end end
        end

        -- click-to-AP: hover highlight + tap the player in the world
        local function _slab(mn, mx, o, d)
            local a, b = (mn - o) / d, (mx - o) / d
            if a > b then a, b = b, a end
            return a, b
        end
        local function rayToCube(ro, rd, cc, cs)
            local h = cs / 2
            local mn, mx = cc - Vector3.new(h, h, h), cc + Vector3.new(h, h, h)
            local dx = rd.X ~= 0 and rd.X or 0.0001
            local dy = rd.Y ~= 0 and rd.Y or 0.0001
            local dz = rd.Z ~= 0 and rd.Z or 0.0001
            local tmin, tmax = _slab(mn.X, mx.X, ro.X, dx)
            local tymin, tymax = _slab(mn.Y, mx.Y, ro.Y, dy)
            if tmin > tymax or tymin > tmax then return false end
            if tymin > tmin then tmin = tymin end
            if tymax < tmax then tmax = tymax end
            local tzmin, tzmax = _slab(mn.Z, mx.Z, ro.Z, dz)
            if tmin > tzmax or tzmin > tmax then return false end
            return true
        end
        local function pickPlayerUnderRay(ray)
            local best, bestD = nil, math.huge
            for _, p in ipairs(Players:GetPlayers()) do
                local h = p ~= LP and p.Character and p.Character:FindFirstChild("HumanoidRootPart")
                if h and rayToCube(ray.Origin, ray.Direction, h.Position, 8) then
                    local d = (ray.Origin - h.Position).Magnitude
                    if d < bestD then bestD, best = d, p end
                end
            end
            return best
        end
        local hlParent = PlayerGui
        pcall(function() hlParent = game:GetService("CoreGui") end)
        local highlight = Instance.new("Highlight")
        highlight.FillColor = Theme.Error; highlight.FillTransparency = 0.3
        highlight.OutlineColor = Theme.Error; highlight.OutlineTransparency = 0
        highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
        pcall(function() highlight.Parent = hlParent end)
        if not highlight.Parent then highlight.Parent = PlayerGui end
        local _hoverT = 0
        RunService.RenderStepped:Connect(function()
            local now = tick()
            if now - _hoverT < 0.05 then return end
            _hoverT = now
            if Config.ClickToAP and UIS.MouseEnabled then
                local cam = workspace.CurrentCamera
                local mp = UIS:GetMouseLocation()
                local bp = pickPlayerUnderRay(cam:ViewportPointToRay(mp.X, mp.Y))
                local na = bp and bp.Character or nil
                if highlight.Adornee ~= na then
                    highlight.Adornee = na
                    local hoveredName = bp and bp.Name or ""
                    local hoverBg = Color3.fromRGB(math.floor(Theme.Error.R * 255 * 0.35), math.floor(Theme.Error.G * 255 * 0.35), math.floor(Theme.Error.B * 255 * 0.35))
                    for _, child in ipairs(listFrame:GetChildren()) do
                        if child:IsA("TextButton") then
                            local st = child:FindFirstChildOfClass("UIStroke")
                            if st then
                                local hot = na and child.Name == hoveredName
                                st.Color = hot and Theme.Error or Theme.SurfaceHighlight
                                st.Transparency = hot and 0 or 0.78
                                child.BackgroundColor3 = hot and hoverBg or Theme.Surface
                            end
                        end
                    end
                end
            else
                highlight.Adornee = nil
            end
        end)
        UIS.InputBegan:Connect(function(inp, g)
            local isTap = inp.UserInputType == Enum.UserInputType.MouseButton1 or inp.UserInputType == Enum.UserInputType.Touch
            if g or not isTap or not Config.ClickToAP then return end
            local cam = workspace.CurrentCamera
            local mp
            if inp.UserInputType == Enum.UserInputType.Touch then
                local ins = GuiService:GetGuiInset()
                mp = Vector2.new(inp.Position.X + ins.X, inp.Position.Y + ins.Y)
            else
                mp = UIS:GetMouseLocation()
            end
            local bp = pickPlayerUnderRay(cam:ViewportPointToRay(mp.X, mp.Y))
            if not bp or isBlacklisted(bp.Name) or getAutojoinerName(bp.Name) then return end
            local nextCmd = getNextAvailableCommand()
            if nextCmd then
                if Config.AdminRandomCmd then fireRandomCmd(bp)
                elseif Config.ClickToAPSingleCommand then
                    if runAdminCommand(bp, nextCmd) then noteCmdFired(bp, nextCmd) end
                else triggerAll(bp) end
            else
                local real = PlayerGui:FindFirstChild("AdminPanel")
                local ok, pbtn = pcall(function()
                    return real.AdminPanel.Profiles.ScrollingFrame:FindFirstChild(bp.Name)
                end)
                if ok and pbtn then fireClick(pbtn) end
            end
        end)
        -- proximity AP
        task.spawn(function()
            while true do
                task.wait(0.2)
                if ProximityAPActive then
                    local mh = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
                    if mh then
                        for _, p in ipairs(Players:GetPlayers()) do
                            local h = p ~= LP and p.Character and p.Character:FindFirstChild("HumanoidRootPart")
                            if h and (h.Position - mh.Position).Magnitude <= Config.ProximityRange then
                                if not (isBlacklisted(p.Name) or getAutojoinerName(p.Name)) and getNextAvailableCommand() then
                                    apAct(p)
                                end
                            end
                        end
                    end
                end
            end
        end)

        -- ── player rows ──────────────────────────────────────────────────
        local removePlayer
        local function _dropRow(row)
            for _, buttons in pairs(SharedState.AdminButtonCache) do
                for i = #buttons, 1, -1 do if buttons[i] and buttons[i]:IsDescendantOf(row) then table.remove(buttons, i) end end
            end
            row:Destroy()
        end
        local ROW_CMDS = { { icon = "JAIL", cmd = "jail" }, { icon = "RAG", cmd = "ragdoll" }, { icon = "RKT", cmd = "rocket" }, { icon = "BLN", cmd = "balloon" } }
        local AJ_BADGE_COLORS = {
            Atlas = Color3.fromRGB(255, 215, 0), Kawaifu = Color3.fromRGB(255, 120, 200), Braintopia = Color3.fromRGB(170, 80, 255),
        }
        local campSyncs = {}
        local function createPlayerRow(plr)
            local row = new("TextButton", listFrame, {
                Name = plr.Name, LayoutOrder = 0, Size = UDim2.new(1, -4, 0, 58), BackgroundColor3 = Theme.Surface,
                BackgroundTransparency = 0.16, BorderSizePixel = 0, AutoButtonColor = false, Text = "",
            })
            corner(row, 14)
            local rowStroke = stroke(row, Theme.SurfaceHighlight, 1, 0.78)
            row.MouseEnter:Connect(function() row.BackgroundTransparency = 0.08; rowStroke.Transparency = 0.35; rowStroke.Color = Theme.Accent1 end)
            row.MouseLeave:Connect(function() row.BackgroundTransparency = 0.16; rowStroke.Transparency = 0.78; rowStroke.Color = Theme.SurfaceHighlight end)
            local dName = new("TextLabel", row, {
                Size = UDim2.new(1, -276, 0, 23), Position = UDim2.new(0, 12, 0, 5), BackgroundTransparency = 1,
                RichText = true, Text = plr.Name, Font = Enum.Font.GothamBold, TextSize = 16, TextColor3 = Theme.TextPrimary,
                TextXAlignment = Enum.TextXAlignment.Left, TextTruncate = Enum.TextTruncate.AtEnd,
            })
            local NAME_MAX, NAME_MIN = 16, 13
            local _fitKey, _lastBadgeAj = nil, false
            local function fitName()
                if not dName.Parent then return end
                local avail = dName.AbsoluteSize.X
                if avail <= 1 then return end
                local key = plr.Name .. "|" .. tostring(math.floor(avail))
                if key == _fitKey then return end
                _fitKey = key
                pcall(function()
                    local w = TextService:GetTextSize(plr.Name, NAME_MAX, dName.Font, Vector2.new(100000, 1000)).X
                    dName.TextSize = (w <= avail) and NAME_MAX or math.clamp(math.floor(NAME_MAX * avail / w), NAME_MIN, NAME_MAX)
                end)
            end
            dName:GetPropertyChangedSignal("AbsoluteSize"):Connect(fitName)
            task.defer(fitName)
            local apBadgeLbl = new("TextLabel", row, {
                Name = "APBadgeLabel", Size = UDim2.new(0, 138, 0, 17), Position = UDim2.new(1, -146, 0, 34),
                BackgroundTransparency = 1, Font = Enum.Font.GothamBold, TextSize = 12,
                TextColor3 = Color3.fromRGB(255, 215, 0), Text = "", TextXAlignment = Enum.TextXAlignment.Right,
            })
            local function refreshBadge()
                local aj = getAutojoinerName(plr.Name)
                if aj == _lastBadgeAj then return end
                _lastBadgeAj = aj
                if aj then
                    apBadgeLbl.Text = "[" .. string.upper(aj) .. " USER]"
                    apBadgeLbl.TextColor3 = AJ_BADGE_COLORS[aj] or Color3.fromRGB(255, 215, 0)
                elseif isBlacklisted(plr.Name) then
                    apBadgeLbl.Text = "[BLACKLISTED]"
                    apBadgeLbl.TextColor3 = Theme.Error
                else
                    apBadgeLbl.Text = ""
                end
            end
            refreshBadge()
            local apToolLbl = new("TextLabel", row, {
                Name = "APToolLabel", Size = UDim2.new(1, -208, 0, 17), Position = UDim2.new(0, 12, 0, 34),
                BackgroundTransparency = 1, Font = Enum.Font.Gotham, TextSize = 13, TextColor3 = Theme.TextSecondary,
                TextXAlignment = Enum.TextXAlignment.Left, TextTruncate = Enum.TextTruncate.AtEnd,
            })
            local function scanHeldTool()
                local ht
                local c = plr.Character
                if c then for _, o in ipairs(c:GetChildren()) do if o:IsA("Tool") then ht = o.Name break end end end
                dName.TextColor3 = (ht and DANGER_TOOLS[ht]) and Theme.TextSecondary or Theme.TextPrimary
                local sIdx = plr:GetAttribute("StealingIndex")
                if plr:GetAttribute("Stealing") or sIdx then
                    local raw = sIdx ~= nil and tostring(sIdx) or nil
                    apToolLbl.Text = "Stealing: " .. (raw and (displayNameOf(raw) or raw) or "Brainrot")
                    apToolLbl.Font = Enum.Font.GothamBold
                    apToolLbl.TextColor3 = Theme.Accent1
                else
                    apToolLbl.Text = ht or ""
                    apToolLbl.Font = Enum.Font.Gotham
                    apToolLbl.TextColor3 = Theme.TextPrimary
                end
            end
            scanHeldTool()
            local btnCont = new("Frame", row, {
                Size = UDim2.new(0, 248, 1, 0), Position = UDim2.new(1, -256, 0, 0), BackgroundTransparency = 1, ZIndex = 10,
            })
            local AB_H, AB_Y = IS_MOBILE and 26 or 24, IS_MOBILE and 4 or 17
            local cmdBtns = {}
            for i, def in ipairs(ROW_CMDS) do
                local cd = isOnCooldown(def.cmd)
                local b = new("TextButton", btnCont, {
                    Size = UDim2.new(0, 38, 0, AB_H), Position = UDim2.new(0, (i - 1) * 42, 0, AB_Y), AutoButtonColor = false,
                    Text = def.icon, TextSize = 12, TextColor3 = cd and OnAccent(Theme.Error) or Theme.TextPrimary,
                    Font = Enum.Font.GothamBold, ZIndex = 11, Active = true,
                    BackgroundColor3 = cd and Theme.Error or Theme.SurfaceHighlight, BackgroundTransparency = 0,
                })
                corner(b, 8)
                local bs = new("UIStroke", b, { Color = cd and Theme.Error or Theme.Accent2, Thickness = 1, Transparency = 0.5 })
                local function busy()
                    return isOnCooldown(def.cmd) or (def.cmd == "balloon" and next(SharedState.BalloonedPlayers) ~= nil)
                end
                b.MouseEnter:Connect(function() if not busy() then b.BackgroundColor3 = lit(Theme.SurfaceHighlight); bs.Transparency = 0.1 end end)
                b.MouseLeave:Connect(function() if not busy() then b.BackgroundColor3 = Theme.SurfaceHighlight; bs.Transparency = 0.5 end end)
                SharedState.AdminButtonCache[def.cmd] = SharedState.AdminButtonCache[def.cmd] or {}
                table.insert(SharedState.AdminButtonCache[def.cmd], b)
                cmdBtns[i] = { b = b, s = bs, icon = def.icon, busy = busy }
                b.MouseButton1Click:Connect(function()
                    if isBlacklisted(plr.Name) or getAutojoinerName(plr.Name) then return end
                    if runAdminCommand(plr, def.cmd) then noteCmdFired(plr, def.cmd) end
                end)
            end
            local allBtn = new("TextButton", btnCont, {
                Size = UDim2.new(0, 38, 0, AB_H), Position = UDim2.new(0, 168, 0, AB_Y), AutoButtonColor = false,
                Text = "ALL", TextSize = 12, Font = Enum.Font.GothamBold, TextColor3 = Theme.Accent1, ZIndex = 11,
                Active = true, BackgroundColor3 = Theme.SurfaceHighlight, BackgroundTransparency = 0,
            })
            corner(allBtn, 8)
            local allStroke = new("UIStroke", allBtn, { Color = Theme.Accent2, Thickness = 1, Transparency = 0.4 })
            local allHover = false
            allBtn.MouseEnter:Connect(function() allHover = true allBtn.BackgroundColor3 = lit(Theme.SurfaceHighlight) allStroke.Transparency = 0.1 end)
            allBtn.MouseLeave:Connect(function() allHover = false allBtn.BackgroundColor3 = Theme.SurfaceHighlight allStroke.Transparency = 0.4 end)
            allBtn.MouseButton1Click:Connect(function()
                if isBlacklisted(plr.Name) or getAutojoinerName(plr.Name) then return end
                for _, cmd in ipairs(getOrderedCommands()) do if not isOnCooldown(cmd) then triggerAll(plr) return end end
            end)
            local campBtn = new("TextButton", btnCont, {
                Size = UDim2.new(0, 38, 0, AB_H), Position = UDim2.new(0, 210, 0, AB_Y), AutoButtonColor = false,
                Text = "CAMP", TextSize = 11, Font = Enum.Font.GothamBold, TextColor3 = Theme.Accent1, ZIndex = 11,
                Active = true, BackgroundColor3 = Theme.SurfaceHighlight, BackgroundTransparency = 0,
            })
            corner(campBtn, 8)
            local campStroke = new("UIStroke", campBtn, { Color = Theme.Accent2, Thickness = 1, Transparency = 0.4 })
            local function campSync()
                local on = RC.active and RC.target == plr.Name
                campBtn.BackgroundColor3 = on and Theme.Accent1 or Theme.SurfaceHighlight
                campBtn.TextColor3 = on and OnAccent() or Theme.Accent1
                campStroke.Transparency = on and 0.1 or 0.4
            end
            campSyncs[row] = campSync
            campBtn.MouseButton1Click:Connect(function()
                if RC.active and RC.target == plr.Name then RC.stop() else RC.start(plr.Name) end
                for _, f in pairs(campSyncs) do pcall(f) end
            end)
            campSync()
            task.spawn(function()
                while row.Parent do
                    task.wait(0.5)
                    if not plr or not plr.Parent or not Players:FindFirstChild(plr.Name) then removePlayer(plr) break end
                    if listFrame.Visible and adminGui.Enabled and frame.Visible then
                        pcall(scanHeldTool)
                        pcall(refreshBadge)
                        pcall(campSync)
                        for _, e in ipairs(cmdBtns) do
                            local hot = e.busy()
                            e.b.BackgroundColor3 = hot and Theme.Error or Theme.SurfaceHighlight
                            e.s.Color = hot and Theme.Error or Theme.Accent2
                            e.s.Transparency = hot and 0.2 or 0.5
                            e.b.TextColor3 = hot and OnAccent(Theme.Error) or Theme.TextPrimary
                        end
                        if not allHover then allBtn.BackgroundColor3 = Theme.SurfaceHighlight allStroke.Color = Theme.Accent2 end
                    end
                end
                campSyncs[row] = nil
            end)
            local rowHl = new("Frame", row, {
                Size = UDim2.new(1, 0, 1, 0), BackgroundColor3 = Theme.Accent1, BackgroundTransparency = 1, BorderSizePixel = 0, ZIndex = 1,
            })
            corner(rowHl, 14)
            row.MouseEnter:Connect(function() rowHl.BackgroundTransparency = 0.88 end)
            row.MouseLeave:Connect(function() rowHl.BackgroundTransparency = 1 end)
            return row
        end
        local playerRows, playerRowsByUserId = {}, {}
        local function addPlayer(plr)
            if plr == LP or playerRowsByUserId[plr.UserId] or playerRows[plr] then return end
            if not Players:FindFirstChild(plr.Name) then return end
            local row = createPlayerRow(plr)
            playerRows[plr] = row
            playerRowsByUserId[plr.UserId] = { player = plr, row = row }
            listFrame.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 30)
            sortList()
        end
        removePlayer = function(plr)
            local userId = plr and plr.UserId or nil
            local entry = userId and playerRowsByUserId[userId] or nil
            local row = entry and entry.row or playerRows[plr]
            if row then
                if row.Parent then _dropRow(row) end
                if plr then playerRows[plr] = nil end
                if userId then
                    playerRowsByUserId[userId] = nil
                    SharedState.BalloonedPlayers[userId] = nil
                end
                listFrame.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 30)
            end
        end
        refreshBtn.MouseButton1Click:Connect(function()
            for _, row in pairs(playerRows) do if row and row.Parent then _dropRow(row) end end
            playerRows, playerRowsByUserId = {}, {}
            SharedState.AdminButtonCache = {}
            SharedState.BalloonedPlayers = {}
            for _, child in ipairs(listFrame:GetChildren()) do if child:IsA("TextButton") then child:Destroy() end end
            task.wait(0.1)
            for _, p in ipairs(Players:GetPlayers()) do if p ~= LP then addPlayer(p) end end
            sortList()
        end)
        Players.PlayerAdded:Connect(function(plr) task.wait(0.1) if plr and plr.Parent then addPlayer(plr) end end)
        Players.PlayerRemoving:Connect(function(plr) removePlayer(plr) end)
        for _, p in ipairs(Players:GetPlayers()) do if p ~= LP then addPlayer(p) end end
        sortList()
        task.spawn(function() while listFrame and listFrame.Parent do task.wait(0.5) pcall(sortList) end end)
        task.spawn(function()
            while true do
                task.wait(1)
                local cur = {}
                for _, p in ipairs(Players:GetPlayers()) do if p ~= LP and p.Parent then cur[p.UserId] = true end end
                for userId, entry in pairs(playerRowsByUserId) do
                    if not cur[userId] or not entry.player or not entry.player.Parent then removePlayer(entry.player) end
                end
                for _, p in ipairs(Players:GetPlayers()) do
                    if p ~= LP and p.Parent and not playerRowsByUserId[p.UserId] then addPlayer(p) end
                end
            end
        end)
        layout.Changed:Connect(function() listFrame.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 30) end)

        -- K (rebindable) and the Settings UI tab both flip the same flag, so
        -- the Settings layer loop never fights the keybind.
        if type(_G.RyPrivUIHidden) ~= "table" then _G.RyPrivUIHidden = {} end
        adminGui.Enabled = _G.RyPrivUIHidden.admin ~= true
        _G.RyPrivToggleAdmin = function()
            if type(_G.RyPrivUIHidden) ~= "table" then _G.RyPrivUIHidden = {} end
            local show = not adminGui.Enabled
            adminGui.Enabled = show
            _G.RyPrivUIHidden.admin = not show
            pcall(_G.RyPrivSaveSettings)
        end
    end)
end
-- ════════════════════════════════════════════════════════════════════════
-- JOIN QUEUE  (ported from v5 rymogs hub, replacing the AJ script runner)
-- ════════════════════════════════════════════════════════════════════════
-- When on, any attempt to join one specific server -- TeleportToPlaceInstance,
-- or TeleportAsync carrying a ServerInstanceId -- is intercepted and retried on
-- a schedule instead of failing once and giving up. That is what makes a full
-- or not-yet-open server joinable: it keeps knocking.
--
-- The hooks go in only when the queue is switched on, so nothing is patched
-- while it is off. ExperienceService:LaunchExperience is preferred when the
-- client exposes it, because it queues server-side; the raw teleport is the
-- fallback, and `bypass` is what stops that fallback re-entering our own hook.
--
-- Globals kept exactly as the v5 hub names them (RyJoinQueueEnabled,
-- RyToggleJoinQueue, RySetJoinQueue, RyJoinQueueLast, RyRefreshJoinQueueBtn),
-- so anything already written against that hub still drives this.
task.spawn(function()
    local ok, err = pcall(function()
        local TeleportService = game:GetService("TeleportService")
        local Players         = game:GetService("Players")
        local hasES, ExperienceService = pcall(function() return game:GetService("ExperienceService") end)
        if not hasES then ExperienceService = nil end

        local JQ = { tries = 5, retry = 3, gap = 3 }
        _G.RyJoinQueue = JQ

        local enabled, hooked, bypass = false, false, false
        local gen, activeJob, lastAt = 0, nil, 0
        local oldNamecall, oldTTPI

        local function setStatus(text, isError)
            _G.RyJoinQueueLast = tostring(text)
            if _G.RyPrivStealSay then pcall(_G.RyPrivStealSay, "joinqueue: " .. tostring(text)) end
            if _G.RyRefreshJoinQueueBtn then pcall(_G.RyRefreshJoinQueueBtn) end
        end

        local function cleanJobId(jobId)
            return (tostring(jobId or ""):gsub("[%s\"'{}%[%]]", ""))
        end

        local function rawTeleport(placeId, jobId)
            bypass = true
            local okT, errT = pcall(function()
                if oldTTPI then
                    oldTTPI(TeleportService, placeId, jobId, Players.LocalPlayer)
                else
                    TeleportService:TeleportToPlaceInstance(placeId, jobId, Players.LocalPlayer)
                end
            end)
            bypass = false
            return okT, errT
        end

        local function attemptOnce(placeId, jobId)
            if ExperienceService then
                return pcall(function()
                    return ExperienceService:LaunchExperience({ placeId = placeId, gameInstanceId = jobId })
                end)
            end
            return rawTeleport(placeId, jobId)
        end

        local function launch(placeId, jobId)
            placeId = tonumber(placeId)
            jobId = cleanJobId(jobId)
            if not placeId or placeId == 0 then placeId = game.PlaceId end
            if jobId == "" then setStatus("no job id given", true) return false end
            if jobId == activeJob then return true end
            activeJob = jobId
            -- gen retires any run still in flight, so switching target servers
            -- does not leave two retry loops racing each other
            gen = gen + 1
            local myGen = gen
            task.spawn(function()
                for attempt = 1, JQ.tries do
                    if gen ~= myGen then return end
                    local cool = JQ.gap - (os.clock() - lastAt)
                    if cool > 0 then task.wait(cool) end
                    if gen ~= myGen then return end
                    lastAt = os.clock()
                    setStatus(string.format("joining %d/%d", attempt, JQ.tries), false)
                    local okA, errA = attemptOnce(placeId, jobId)
                    if okA then setStatus("queued", false) return end
                    setStatus(tostring(errA), true)
                    task.wait(JQ.retry)
                end
                if gen ~= myGen then return end
                setStatus("gave up", true)
                activeJob = nil
            end)
            return true
        end

        TeleportService.TeleportInitFailed:Connect(function(_, result, message)
            if not enabled then return end
            setStatus(tostring(result) .. ": " .. tostring(message), true)
        end)

        local function installHook()
            if hooked then return end
            hooked = true
            if hookfunction and newcclosure then
                pcall(function()
                    oldTTPI = hookfunction(TeleportService.TeleportToPlaceInstance,
                        newcclosure(function(self, placeId, jobId, ...)
                            if enabled and not bypass then launch(placeId, jobId) return end
                            return oldTTPI(self, placeId, jobId, ...)
                        end))
                end)
            end
            if hookmetamethod and getnamecallmethod and newcclosure then
                pcall(function()
                    oldNamecall = hookmetamethod(game, "__namecall", newcclosure(function(self, ...)
                        if enabled and not bypass and self == TeleportService then
                            local m = getnamecallmethod()
                            if m == "TeleportToPlaceInstance" then
                                local a1, a2 = ...
                                launch(a1, a2)
                                return
                            elseif m == "TeleportAsync" then
                                local a1, _, a3 = ...
                                local jobId = a3 and a3.ServerInstanceId
                                if jobId and jobId ~= "" then launch(a1, jobId) return end
                            end
                        end
                        return oldNamecall(self, ...)
                    end))
                end)
            end
        end

        local function setEnabled(state)
            enabled = state and true or false
            _G.RyJoinQueueEnabled = enabled
            if enabled then
                installHook()
                setStatus("on", false)
            else
                gen = gen + 1
                activeJob = nil
                _G.RyJoinQueueLast = nil
                if _G.RyRefreshJoinQueueBtn then pcall(_G.RyRefreshJoinQueueBtn) end
            end
        end
        _G.RySetJoinQueue = setEnabled
        _G.RyToggleJoinQueue = function() setEnabled(not enabled) return enabled end
        _G.RyJoinQueueEnabled = false
    end)
    if not ok and _G.RyPrivStealSay then
        pcall(_G.RyPrivStealSay, "joinqueue block failed: " .. tostring(err))
    end
end)

-- ════════════════════════════════════════════════════════════════════════
-- ESP  (ported from the v5 rymogs hub, drives Settings -> VISUALS)
-- ════════════════════════════════════════════════════════════════════════
-- Player name billboards, the stealer / base-owner / brainrot / trap marks,
-- and the x-ray brainrot boxes. One budgeted pass on a timer does the work and
-- everything shares a single clear, so switching the last flag off tears the
-- whole lot down rather than leaving adornments behind.
--
-- TURRET ESP IS NOT HERE ON PURPOSE. It does not work in v5 either, and a
-- switch that silently does nothing is worse than no switch, so the setter is
-- gone and ESP_Turret stays false -- the hazard pass reads it and skips.
--
-- scanAllPets and isTeleporting are this hub's own; they were checked to exist
-- before porting rather than assumed.

do
    local cachedId, lastT = nil, 0
    -- Which base am I standing in? Base Owner ESP reads this; the main ESP pass
    -- guards the call, so without it that one switch simply resolves no owner.
    function _G.__getCurrentBaseOwnerId()
        local now = os.clock()
        if (now - lastT) < 0.4 then return cachedId end
        lastT, cachedId = now, nil
        local char = LP.Character
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        local plots = workspace:FindFirstChild("Plots")
        if not (hrp and plots) then return nil end
        local best, bestD
        for _, plot in ipairs(plots:GetChildren()) do
            local ok, pos = pcall(function() return plot:GetPivot().Position end)
            if ok and pos then
                local d = (pos - hrp.Position).Magnitude
                if not bestD or d < bestD then best, bestD = plot, d end
            end
        end
        if not best or (bestD or math.huge) > 180 then return nil end
        local okO, owner = pcall(function()
            local ch = getPlotChannel(best.Name)
            return ch and channelGet(ch, "Owner")
        end)
        if not okO or not owner then return nil end
        local nm = (typeof(owner) == "Instance" and owner.Name)
            or (type(owner) == "table" and owner.Name)
            or (type(owner) == "string" and owner) or nil
        if not nm then return nil end
        local p = game:GetService("Players"):FindFirstChild(nm)
        cachedId = p and p.UserId or nil
        return cachedId
    end
end



;(function()
    local Players    = game:GetService("Players")
    local RunService = game:GetService("RunService")
    local LPlr       = Players.LocalPlayer

    local FOLDER_NAME = "RymogsESP"
    local function espFolder()
        local f = workspace:FindFirstChild(FOLDER_NAME)
        if not f then
            f = Instance.new("Folder")
            f.Name = FOLDER_NAME
            f.Parent = workspace
        end
        return f
    end

    for _, k in ipairs({ "ESP_Stealer", "ESP_BaseOwner", "ESP_Brainrot", "ESP_Turret", "ESP_Trap", "ESP_BrainrotBox" }) do
        if _G[k] == nil then _G[k] = false end
    end

    local COL = {
        stealer  = Color3.fromRGB(255,  90,  90),
        owner    = Color3.fromRGB( 90, 170, 255),
        brainrot = Color3.fromRGB(235, 235, 245),
        turret   = Color3.fromRGB(255,  40,  40),
        trap     = Color3.fromRGB(255, 150,  20),
    }

    local marks = {}
    local _espErrSaid = {}
    local _espReport = 0

    local function dropMark(key)
        local m = marks[key]
        if not m then return end
        if m.hl and m.hl.Parent then pcall(function() m.hl:Destroy() end) end
        if m.bb and m.bb.Parent then pcall(function() m.bb:Destroy() end) end
        marks[key] = nil
    end

    local function clearAll()
        for k in pairs(marks) do dropMark(k) end
        local f = workspace:FindFirstChild(FOLDER_NAME)
        if f then pcall(function() f:ClearAllChildren() end) end
    end

    local function ensureMark(key, adornee, colour, text, yOffset)
        if not adornee or not adornee.Parent then dropMark(key) return end
        local m = marks[key]
        if m and m.adornee == adornee and m.bb and m.bb.Parent then
            if m.text ~= text then
                m.text = text
                if m.lbl then m.lbl.Text = text end
            end
            m.seen = true
            return
        end
        dropMark(key)

        local holder = espFolder()
        local hl
        if adornee:IsA("Model") then
            hl = Instance.new("Highlight")
            hl.Adornee = adornee
            hl.FillColor = colour
            hl.FillTransparency = 0.6
            hl.OutlineColor = colour
            hl.OutlineTransparency = 0
            hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
            hl.Parent = holder
        elseif adornee:IsA("BasePart") then
            hl = Instance.new("SelectionBox")
            hl.Adornee = adornee
            hl.Color3 = colour
            hl.LineThickness = 0.06
            hl.SurfaceColor3 = colour
            hl.SurfaceTransparency = 0.75
            hl.Parent = holder
        end

        local bbAd = adornee:IsA("BasePart") and adornee
            or adornee:FindFirstChild("Head")
            or adornee:FindFirstChild("HumanoidRootPart")
            or adornee:FindFirstChildWhichIsA("BasePart", true)
        local bb, lbl
        if bbAd then
            bb = Instance.new("BillboardGui")
            bb.Adornee = bbAd
            bb.Size = UDim2.fromOffset(190, 36)
            bb.StudsOffset = Vector3.new(0, yOffset or 3, 0)
            bb.AlwaysOnTop = true
            bb.MaxDistance = tonumber(_G.ESP_MaxDistance) or 700
            bb.Parent = holder
            lbl = Instance.new("TextLabel", bb)
            lbl.Size = UDim2.fromScale(1, 1)
            lbl.BackgroundTransparency = 1
            lbl.Font = Enum.Font.GothamBold
            lbl.TextSize = 12
            lbl.TextColor3 = colour
            lbl.TextStrokeColor3 = Color3.new(0, 0, 0)
            lbl.TextStrokeTransparency = 0
            lbl.Text = text
        end
        marks[key] = { hl = hl, bb = bb, lbl = lbl, adornee = adornee, text = text, seen = true }
    end

    local sfind = string.find
    local TRAP_WORDS = { "trap", "mine", "hive" }
    local function hazardKind(name)
        local ln = name:lower()
        if sfind(ln, "sentrybullet", 1, true) then return nil end
        if sfind(ln, "tripmine", 1, true) then return "trap" end
        if sfind(ln, "sentry",  1, true) then return "turret" end
        if sfind(ln, "turret",  1, true) then return "turret" end
        for _, w in ipairs(TRAP_WORDS) do
            if sfind(ln, w, 1, true) then return "trap" end
        end
        return nil
    end

    local function passPlayers()
        local sOn = _G.ESP_Stealer   == true
        local oOn = _G.ESP_BaseOwner == true
        if not sOn and not oOn then return end

        local ownerId
        if oOn and _G.__getCurrentBaseOwnerId then
            ownerId = _G.__getCurrentBaseOwnerId()
        end

        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= LPlr then
                local char = p.Character
                if char and char:FindFirstChild("HumanoidRootPart") then
                    local kind, colour, text
                    if sOn and p:GetAttribute("Stealing") == true then
                        kind, colour = "stealer", COL.stealer
                        local pet = p:GetAttribute("StealingPet")
                        text = "STEALER" .. (pet and ("  " .. tostring(pet)) or "")
                    elseif oOn and ownerId and p.UserId == ownerId then
                        kind, colour, text = "owner", COL.owner, "BASE OWNER"
                    end
                    if kind then
                        ensureMark("plr_" .. p.UserId, char, colour, text, 3)
                    end
                end
            end
        end
    end


    -- The brainrot pass used to live here. Brainrot marks are owned by the
    -- dedicated ESP block at the end of this file now.

    local function passHazards()
        local tOn = _G.ESP_Turret == true
        local rOn = _G.ESP_Trap   == true
        if not tOn and not rOn then return end

        local plots = workspace:FindFirstChild("Plots")
        if not plots then return end

        local budget = (tonumber(_G.ESP_BudgetMs) or 3) / 1000
        local t0, n = os.clock(), 0
        for _, d in ipairs(plots:GetDescendants()) do
            n = n + 1
            if n >= 400 or (os.clock() - t0) >= budget then
                RunService.Heartbeat:Wait()
                n, t0 = 0, os.clock()
            end
            if d:IsA("Model") or d:IsA("BasePart") then
                local kind = hazardKind(d.Name)
                if kind and ((kind == "turret" and tOn) or (kind == "trap" and rOn)) then
                    ensureMark(d, d, COL[kind], (kind == "turret") and "TURRET" or "TRAP", 4)
                end
            end
        end
    end

    local function anyOn()
        -- ESP_Brainrot is deliberately not here: the dedicated block below owns
        -- brainrot marks, and leaving it in made this pass clear its own marks
        -- whenever brainrot ESP was the only thing switched on.
        return _G.ESP_Stealer == true or _G.ESP_BaseOwner == true
            or _G.ESP_Turret == true or _G.ESP_Trap == true
    end

    task.spawn(function()
        while true do
            if not anyOn() then
                if next(marks) then clearAll() end
                task.wait(0.5)
            else
                -- First few cycles only: say what the passes actually found,
                -- so "nothing shows" comes with a reason attached.
                _espReport = (_espReport or 0) + 1
                for _, m in pairs(marks) do m.seen = false end
                -- A bare pcall here makes a broken pass look exactly like a
                -- pass that found nothing, which is why "ESP does nothing" had
                -- no error to go with it. Report the first failure from each
                -- pass, once, and then stay quiet.
                -- brainrots are handled by the dedicated ESP block further
                -- down; this loop keeps the stealer/base-owner and hazard marks
                for _, pass in ipairs({
                    { "players", passPlayers },
                    { "hazards", passHazards },
                }) do
                    local okP, errP = pcall(pass[2])
                    if not okP and not _espErrSaid[pass[1]] then
                        _espErrSaid[pass[1]] = true
                        -- _RP is the real print, captured at the top of this
                        -- file before print is nulled. RyPrivStealSay only
                        -- reaches the console when RyPrivStealDebug is set and
                        -- throttles to one line every half second, which would
                        -- swallow these.
                        pcall(_RP, "[ESP] " .. pass[1] .. " pass failed: " .. tostring(errP))
                    end
                end
                for k, m in pairs(marks) do
                    if not m.seen then dropMark(k) end
                end
                if _espReport <= 3 then
                    local nMarks = 0
                    for _ in pairs(marks) do nMarks = nMarks + 1 end
                    local nPets = (type(_petsCache) == "table") and #_petsCache or -1
                    local folder = workspace:FindFirstChild(FOLDER_NAME)
                    pcall(_RP, string.format(
                        "[ESP] pass %d: pets=%d marks=%d folder=%s children=%d"
                        .. " (brainrot=%s stealer=%s owner=%s trap=%s)",
                        _espReport, nPets, nMarks, tostring(folder ~= nil),
                        folder and #folder:GetChildren() or -1,
                        tostring(_G.ESP_Brainrot == true), tostring(_G.ESP_Stealer == true),
                        tostring(_G.ESP_BaseOwner == true), tostring(_G.ESP_Trap == true)))
                end
                task.wait(tonumber(_G.ESP_Interval) or 0.5)
            end
        end
    end)

    Players.PlayerRemoving:Connect(function(p)
        if p then dropMark("plr_" .. p.UserId) end
    end)

    local function mkSetter(flag)
        return function(on)
            _G[flag] = on and true or false
            if not anyOn() then clearAll() end
        end
    end
    _G.setStealerESP   = mkSetter("ESP_Stealer")
    _G.setBaseOwnerESP = mkSetter("ESP_BaseOwner")
    _G.setTrapESP      = mkSetter("ESP_Trap")
    _G.clearAllESP     = clearAll
end)()

if _G.ESP_BrainrotBox == nil then _G.ESP_BrainrotBox = false end
do
    local _xf
    local _boxes = {}
    local function _ensure()
        if _xf and _xf.Parent then return end
        _xf = Instance.new("Folder"); _xf.Name = "Effects"; _xf.Parent = workspace
        table.clear(_boxes)
    end
    local function _clearAll()
        if _xf then pcall(function() _xf:Destroy() end) end
        _xf = nil; table.clear(_boxes)
    end
    _G.setBrainrotBoxESP = function(on)
        _G.ESP_BrainrotBox = on and true or false
        if not on then _clearAll() end
    end
    task.spawn(function()
        if not game:IsLoaded() then game.Loaded:Wait() end
        local lastN = nil
        while true do
            task.wait(tonumber(_G.ESP_BoxGap) or 0.75)
            if _G.ESP_BrainrotBox ~= true then
                if _xf then _clearAll() end
                lastN = nil
            elseif isTeleporting and _xf then
            else
                local ok, pets = pcall(scanAllPets)
                if not ok then
                    if lastN ~= "err" then
                        lastN = "err"
                        warn("[rymogs boxes] pet scan error: " .. tostring(pets))
                    end
                elseif type(pets) == "table" then
                    _ensure()
                    local seen, n = {}, 0
                    local sz = tonumber(_G.ESP_BoxSize) or 4.5
                    local tr = tonumber(_G.ESP_BoxTransp) or 0.5
                    local col = _G.ESP_BoxColor or Color3.fromRGB(255, 255, 255)
                    for _, p in ipairs(pets) do
                        if p and p.position then
                            local uid = tostring(p.plot) .. "_" .. tostring(p.slot)
                            seen[uid] = true
                            n = n + 1
                            local b = _boxes[uid]
                            if not (b and b.anchor and b.anchor.Parent) then
                                local anchor = Instance.new("Part")
                                anchor.Anchored = true; anchor.CanCollide = false
                                anchor.CanQuery = false; anchor.CanTouch = false
                                anchor.Transparency = 1; anchor.Size = Vector3.one
                                anchor.Parent = _xf
                                local a = Instance.new("BoxHandleAdornment")
                                a.Adornee = anchor; a.AlwaysOnTop = true; a.ZIndex = 0
                                pcall(function() a.Shading = Enum.AdornShading.XRayShaded end)
                                a.Parent = anchor
                                b = { anchor = anchor, adorn = a }; _boxes[uid] = b
                            end
                            pcall(function() b.anchor.CFrame = CFrame.new(p.position) end)
                            pcall(function()
                                b.adorn.Size = Vector3.new(sz, sz, sz)
                                b.adorn.Transparency = tr
                                b.adorn.Color3 = col
                            end)
                        end
                    end
                    for uid, b in pairs(_boxes) do
                        if not seen[uid] then
                            pcall(function() b.anchor:Destroy() end); _boxes[uid] = nil
                        end
                    end
                    if n ~= lastN then
                        lastN = n
                        print(string.format("[rymogs boxes] %d boxes", n))
                    end
                end
            end
        end
    end)
end
-- ════════════════════════════════════════════════════════════════════════
-- PLAYER ESP + BRAINROT ESP
-- ════════════════════════════════════════════════════════════════════════
-- Own implementation. The previous one parented each BillboardGui straight
-- into workspace, onto the part it was labelling; that dies whenever the part
-- is streamed out, replaced or re-parented, and this game runs with streaming
-- enabled. Here the GUIs live in one folder we own under the UI host and only
-- their Adornee points at the world, so nothing the game does to a character or
-- a podium can take them with it.
--
-- Both run off one timer. Marks are reused between passes rather than being
-- destroyed and rebuilt, and anything whose target has gone is dropped at the
-- end of the pass.
do
    local Players    = game:GetService("Players")
    local LPlr       = Players.LocalPlayer

    local guiHolder, partHolder

    local function uiHost()
        return (gethui and gethui()) or LPlr:FindFirstChildOfClass("PlayerGui")
    end

    -- the BillboardGuis, kept out of workspace
    local function guis()
        if guiHolder and guiHolder.Parent then return guiHolder end
        guiHolder = Instance.new("Folder")
        guiHolder.Name = "RyPrivESPGui"
        guiHolder.Parent = uiHost()
        return guiHolder
    end

    -- invisible anchors for brainrots, which have a world position but no part
    -- of their own we can safely hold on to
    local function anchors()
        if partHolder and partHolder.Parent then return partHolder end
        partHolder = Instance.new("Folder")
        partHolder.Name = "RyPrivESPAnchors"
        partHolder.Parent = workspace
        return partHolder
    end

    local function mkLabel(text, colour)
        local bb = Instance.new("BillboardGui")
        bb.Name = "mark"
        bb.Size = UDim2.fromOffset(200, 34)
        bb.StudsOffset = Vector3.new(0, 2.6, 0)
        bb.AlwaysOnTop = true
        bb.LightInfluence = 0
        bb.ResetOnSpawn = false
        local lbl = Instance.new("TextLabel")
        lbl.Size = UDim2.fromScale(1, 1)
        lbl.BackgroundTransparency = 1
        lbl.Font = Enum.Font.GothamBold
        lbl.TextSize = 13
        lbl.TextColor3 = colour
        lbl.TextStrokeTransparency = 0.35
        lbl.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
        lbl.TextScaled = false
        lbl.Text = text
        lbl.Parent = bb
        bb.Parent = guis()
        return bb, lbl
    end

    local function maxDist()
        return tonumber(_G.ESP_MaxDistance) or 700
    end

    -- ── PLAYER ESP ───────────────────────────────────────────────────────
    local pMarks = {}                 -- userId -> { bb, lbl, part }

    local function dropPlayer(uid)
        local m = pMarks[uid]
        if not m then return end
        if m.bb then pcall(function() m.bb:Destroy() end) end
        pMarks[uid] = nil
    end
    local function clearPlayers()
        for uid in pairs(pMarks) do dropPlayer(uid) end
    end

    local function playerPass()
        if _G.RyPlayerESP ~= true then
            if next(pMarks) then clearPlayers() end
            return
        end
        local seen = {}
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= LPlr then
                local char = plr.Character
                local part = char and (char:FindFirstChild("Head")
                    or char:FindFirstChild("HumanoidRootPart"))
                if part then
                    local uid = plr.UserId
                    seen[uid] = true
                    local m = pMarks[uid]
                    local text = plr.DisplayName
                    if plr.DisplayName ~= plr.Name then
                        text = text .. "  (@" .. plr.Name .. ")"
                    end
                    if not (m and m.bb and m.bb.Parent) then
                        dropPlayer(uid)
                        local bb, lbl = mkLabel(text, Color3.fromRGB(255, 255, 255))
                        m = { bb = bb, lbl = lbl }
                        pMarks[uid] = m
                    end
                    -- re-point rather than rebuild: the character is replaced on
                    -- every respawn and streaming can swap the part underneath us
                    if m.bb.Adornee ~= part then m.bb.Adornee = part end
                    if m.lbl.Text ~= text then m.lbl.Text = text end
                    m.bb.MaxDistance = maxDist()
                    m.bb.Enabled = true
                end
            end
        end
        for uid in pairs(pMarks) do
            if not seen[uid] then dropPlayer(uid) end
        end
    end

    -- ── BRAINROT ESP ─────────────────────────────────────────────────────
    local bMarks = {}                 -- "plot_slot" -> { bb, lbl, anchor }

    local function dropBrainrot(key)
        local m = bMarks[key]
        if not m then return end
        if m.bb then pcall(function() m.bb:Destroy() end) end
        if m.anchor then pcall(function() m.anchor:Destroy() end) end
        bMarks[key] = nil
    end
    local function clearBrainrots()
        for k in pairs(bMarks) do dropBrainrot(k) end
    end

    local function short(n)
        n = tonumber(n) or 0
        if n >= 1e9 then return string.format("%.1fB", n / 1e9) end
        if n >= 1e6 then return string.format("%.1fM", n / 1e6) end
        if n >= 1e3 then return string.format("%.1fK", n / 1e3) end
        return tostring(math.floor(n))
    end

    local function brainrotPass()
        if _G.ESP_Brainrot ~= true then
            if next(bMarks) then clearBrainrots() end
            return
        end
        local ok, pets = pcall(function()
            return _G.RyPrivScanAllPets and _G.RyPrivScanAllPets() or nil
        end)
        if not ok or type(pets) ~= "table" then return end

        -- highest generation first, so a cap on how many are labelled keeps the
        -- ones worth looking at
        local list = {}
        for _, p in ipairs(pets) do
            if p and p.position and p.plot and p.slot and not p.conveyor then
                list[#list + 1] = p
            end
        end
        table.sort(list, function(a, b)
            return (tonumber(a.mps) or 0) > (tonumber(b.mps) or 0)
        end)

        local cap = math.clamp(tonumber(_G.ESP_BrainrotTop) or 5, 1, 50)
        local seen = {}
        for i = 1, math.min(cap, #list) do
            local p = list[i]
            local key = tostring(p.plot) .. "_" .. tostring(p.slot)
            seen[key] = true
            local text = string.format("%s  [%s/s]", tostring(p.name), short(p.mps))
            local m = bMarks[key]
            if not (m and m.bb and m.bb.Parent and m.anchor and m.anchor.Parent) then
                dropBrainrot(key)
                local a = Instance.new("Part")
                a.Name = "brAnchor"
                a.Anchored = true
                a.CanCollide = false
                a.CanQuery = false
                a.CanTouch = false
                a.Transparency = 1
                a.Size = Vector3.new(0.2, 0.2, 0.2)
                a.CFrame = CFrame.new(p.position)
                a.Parent = anchors()
                local bb, lbl = mkLabel(text, Color3.fromRGB(235, 235, 245))
                bb.Adornee = a
                m = { bb = bb, lbl = lbl, anchor = a }
                bMarks[key] = m
            end
            -- brainrots move along the conveyor and get swapped on the podium,
            -- so the anchor follows rather than the mark being rebuilt
            m.anchor.CFrame = CFrame.new(p.position)
            if m.lbl.Text ~= text then m.lbl.Text = text end
            m.bb.MaxDistance = maxDist()
            m.bb.Enabled = true
        end
        for k in pairs(bMarks) do
            if not seen[k] then dropBrainrot(k) end
        end
    end

    _G.setPlayerESP = function(on)
        _G.RyPlayerESP = on and true or false
        if not _G.RyPlayerESP then clearPlayers() end
    end
    _G.setBrainrotESP = function(on)
        _G.ESP_Brainrot = on and true or false
        if not _G.ESP_Brainrot then clearBrainrots() end
    end

    task.spawn(function()
        while true do
            local okP, errP = pcall(playerPass)
            if not okP and not _G._ryEspSaidP then
                _G._ryEspSaidP = true
                pcall(_RP, "[ESP] player pass: " .. tostring(errP))
            end
            local okB, errB = pcall(brainrotPass)
            if not okB and not _G._ryEspSaidB then
                _G._ryEspSaidB = true
                pcall(_RP, "[ESP] brainrot pass: " .. tostring(errB))
            end
            task.wait(tonumber(_G.ESP_Interval) or 0.5)
        end
    end)

    -- a respawn replaces the whole character, so the old adornees are gone
    LPlr.CharacterAdded:Connect(function() clearPlayers() end)
    Players.PlayerRemoving:Connect(function(p)
        if p then dropPlayer(p.UserId) end
    end)
end
-- ════════════════════════════════════════════════════════════════════════
-- DROP VIEW  (hide the carried brainrot instead of hiding yourself)
-- ════════════════════════════════════════════════════════════════════════
-- The existing invis hides YOU: it clones your root, parents the real one into
-- the camera and sinks it under the pet. That is why you end up standing inside
-- the brainrot, and why movement goes through a stand-in body.
--
-- This does the opposite, and leaves your character completely alone. Normal
-- root, normal movement, normal stealing. The only thing touched is the carried
-- brainrot's own model: its joints to you are cut on the client and it is let
-- go, so it falls away through the map. The server still has you carrying it,
-- so the steal completes and the delivery still counts -- it just is not drawn
-- sitting on top of you any more.
--
-- Everything here is client-side and reversible. Joints are disabled rather
-- than destroyed, and parts remember what they were, so switching off or
-- finishing the carry puts it all back.
do
    local RIG = {
        Head = true, HumanoidRootPart = true, UpperTorso = true, LowerTorso = true,
        Torso = true, LeftHand = true, RightHand = true, LeftFoot = true, RightFoot = true,
        LeftUpperArm = true, RightUpperArm = true, LeftLowerArm = true, RightLowerArm = true,
        LeftUpperLeg = true, RightUpperLeg = true, LeftLowerLeg = true, RightLowerLeg = true,
        ["Left Arm"] = true, ["Right Arm"] = true, ["Left Leg"] = true, ["Right Leg"] = true,
    }

    local held = nil          -- the model we are currently hiding
    local joints = {}         -- joint -> true, disabled by us
    local parts = {}          -- part  -> { anchored, collide, transp }
    local conn = nil

    -- The carried brainrot rides inside the character. It is the one model in
    -- there that is not the rig and has no Humanoid of its own -- that rules
    -- out the invis clone and anything player-shaped.
    local function findCarried(char)
        for _, d in ipairs(char:GetChildren()) do
            if d:IsA("Model")
                and not d:FindFirstChildOfClass("Humanoid")
                and not RIG[d.Name]
                and d:FindFirstChildWhichIsA("BasePart", true) then
                return d
            end
        end
        -- some builds weld the pet in as a loose part rather than a model
        for _, d in ipairs(char:GetChildren()) do
            if d:IsA("BasePart") and not RIG[d.Name] and d.Name ~= "Handle" then
                return d
            end
        end
        return nil
    end

    local function partsOf(inst)
        local out = {}
        if inst:IsA("BasePart") then out[#out + 1] = inst end
        for _, d in ipairs(inst:GetDescendants()) do
            if d:IsA("BasePart") then out[#out + 1] = d end
        end
        return out
    end

    local function release(inst)
        if not inst or not inst.Parent then return end
        -- cut every joint tying it to us. Disabled, not destroyed, so the
        -- server re-replicating them does not leave anything mangled.
        for _, d in ipairs(inst:GetDescendants()) do
            if d:IsA("JointInstance") or d:IsA("WeldConstraint") then
                if d.Enabled ~= false then
                    joints[d] = true
                    pcall(function() d.Enabled = false end)
                end
            end
        end
        local char = LP.Character
        for _, d in ipairs(char and char:GetDescendants() or {}) do
            if (d:IsA("JointInstance") or d:IsA("WeldConstraint")) and not joints[d] then
                local a = d:IsA("WeldConstraint") and d.Part0 or d.Part0
                local b = d:IsA("WeldConstraint") and d.Part1 or d.Part1
                local function mine(p) return p and (p == inst or p:IsDescendantOf(inst)) end
                if mine(a) or mine(b) then
                    joints[d] = true
                    pcall(function() d.Enabled = false end)
                end
            end
        end
        -- and let it go
        for _, p in ipairs(partsOf(inst)) do
            if parts[p] == nil then
                parts[p] = { anchored = p.Anchored, collide = p.CanCollide,
                             transp = p.LocalTransparencyModifier }
            end
            pcall(function()
                p.Anchored = false
                p.CanCollide = false
                p.CanQuery = false
                p.CanTouch = false
                -- falls out of the world rather than sitting in the floor
                p.AssemblyLinearVelocity = Vector3.new(0, -60, 0)
            end)
        end
    end

    local function restore()
        for j in pairs(joints) do
            if j and j.Parent then pcall(function() j.Enabled = true end) end
        end
        table.clear(joints)
        for p, old in pairs(parts) do
            if p and p.Parent then
                pcall(function()
                    p.Anchored = old.anchored
                    p.CanCollide = old.collide
                    p.LocalTransparencyModifier = old.transp or 0
                    p.CanQuery = true
                    p.CanTouch = true
                end)
            end
        end
        table.clear(parts)
        held = nil
    end

    local function tick()
        if _G.RyPrivDropView ~= true or LP:GetAttribute("Stealing") ~= true then
            if held or next(parts) then restore() end
            return
        end
        local char = LP.Character
        if not char then return end

        if not (held and held.Parent) then
            held = findCarried(char)
            if held then release(held) end
            return
        end

        -- Keep it going down. The server keeps re-anchoring and re-welding the
        -- carried model, so this has to be re-applied rather than set once --
        -- and a model that stops being pushed drifts back onto the character.
        for _, p in ipairs(partsOf(held)) do
            pcall(function()
                if p.Anchored then p.Anchored = false end
                p.CanCollide = false
                if p.Position.Y > -500 then
                    p.AssemblyLinearVelocity = Vector3.new(0, -80, 0)
                else
                    -- far enough under the map: stop spending work on it
                    p.LocalTransparencyModifier = 1
                end
            end)
        end
    end

    -- Where does this game actually keep the brainrot while you carry it?
    -- findCarried above assumed it is parented into the character; it is not,
    -- so this reports what is really there. Run it while holding one:
    --
    --   _G.RyPrivFindCarried()
    --
    -- It lists the character's own children, anything parented to the root,
    -- and every model sitting within a few studs of you that is not a player.
    -- Whichever of those is the brainrot is what the finder needs to target.
    _G.RyPrivFindCarried = function()
        local char = LP.Character
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        if not char then pcall(_RP, "[find] no character") return end
        pcall(_RP, string.format("[find] Stealing=%s index=%s",
            tostring(LP:GetAttribute("Stealing")),
            tostring(LP:GetAttribute("StealingIndex"))))

        local out = {}
        for _, d in ipairs(char:GetChildren()) do
            out[#out + 1] = d.ClassName .. ":" .. d.Name
        end
        pcall(_RP, "[find] character children: " .. table.concat(out, ", "))

        if hrp then
            local kids = {}
            for _, d in ipairs(hrp:GetChildren()) do
                kids[#kids + 1] = d.ClassName .. ":" .. d.Name
            end
            pcall(_RP, "[find] root children: " .. table.concat(kids, ", "))

            -- anything model-shaped hovering near us that is not a player
            local near = {}
            for _, m in ipairs(workspace:GetDescendants()) do
                if m:IsA("Model") and m ~= char and not m:FindFirstChildOfClass("Humanoid") then
                    local ok, cf = pcall(m.GetPivot, m)
                    if ok and cf and (cf.Position - hrp.Position).Magnitude <= 8 then
                        near[#near + 1] = m:GetFullName()
                        if #near >= 8 then break end
                    end
                end
            end
            pcall(_RP, "[find] models within 8 studs: "
                .. (#near > 0 and table.concat(near, " | ") or "none"))
        end
    end

    _G.RyPrivSetDropView = function(on)
        _G.RyPrivDropView = on and true or false
        if not _G.RyPrivDropView then restore() end
        if conn == nil then
            conn = RunService.Heartbeat:Connect(function()
                local ok, err = pcall(tick)
                if not ok and not _G._ryDropSaid then
                    _G._ryDropSaid = true
                    pcall(_RP, "[dropview] " .. tostring(err))
                end
            end)
        end
        pcall(_G.RyPrivSaveSettings)
        return _G.RyPrivDropView
    end
    _G.RyPrivToggleDropView = function()
        return _G.RyPrivSetDropView(_G.RyPrivDropView ~= true)
    end

    -- a respawn takes the carried model with it
    LP.CharacterAdded:Connect(function() restore() end)

    if _G.RyPrivDropView == true then _G.RyPrivSetDropView(true) end
end
-- ════════════════════════════════════════════════════════════════════════
-- MOBILE DOCK
-- ════════════════════════════════════════════════════════════════════════
-- Every panel in this hub opens on a keybind -- Ctrl for Settings, U for
-- Utility, J for Invis, H for instant steal -- which means on a touch device
-- none of them can be opened at all. This puts the same actions on screen.
--
-- It only builds itself on a touch device without a keyboard. Set
-- RyPrivForceMobile = true to get it on a desktop, which is also how to test it.
--
-- The dock is draggable and remembers where it was put, through the same
-- position helpers every other panel uses. Tapping the handle collapses it to
-- a single button so it is not sitting over the game while you play.
do
    local function isMobile()
        if _G.RyPrivForceMobile == true then return true end
        local ok, v = pcall(function()
            return UIS.TouchEnabled and not UIS.KeyboardEnabled
        end)
        return ok and v == true
    end
    _G.RyPrivIsMobile = isMobile

    if not isMobile() then return end

    local C = {
        bg   = Color3.fromRGB(18, 19, 22),
        btn  = Color3.fromRGB(34, 37, 42),
        on   = Color3.fromRGB(96, 178, 235),
        line = Color3.fromRGB(70, 76, 86),
        txt  = Color3.fromRGB(238, 241, 245),
        dim  = Color3.fromRGB(150, 158, 170),
    }
    local FB = Enum.Font.GothamBold

    local function mk(cls, parent, props)
        local o = Instance.new(cls)
        for k, v in pairs(props or {}) do o[k] = v end
        o.Parent = parent
        return o
    end
    local function round(o, r)
        mk("UICorner", o, { CornerRadius = UDim.new(0, r or 6) })
        return o
    end

    task.spawn(function()
        if _G.RyPrivUIWait then _G.RyPrivUIWait() end
        local host = (gethui and gethui()) or game:GetService("CoreGui")
            or LP:WaitForChild("PlayerGui")
        pcall(function()
            local old = host:FindFirstChild("RyPrivMobileUI")
            if old then old:Destroy() end
        end)

        local sg = mk("ScreenGui", nil, {
            Name = "RyPrivMobileUI", ResetOnSpawn = false, IgnoreGuiInset = true,
            DisplayOrder = 999999, ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
        })
        pcall(function() sg.Parent = host end)
        if not sg.Parent then sg.Parent = LP:WaitForChild("PlayerGui") end

        -- Buttons are deliberately large: 44px is about the smallest a thumb
        -- hits reliably, and the panels behind this are already fiddly on a
        -- phone.
        local BTN_H, BTN_W, PAD = 44, 92, 6

        local rows = {
            { "SETTINGS", function() if _G.RyPrivToggleSettings then pcall(_G.RyPrivToggleSettings) end end },
            { "UTILITY",  function() if _G.RyPrivToggleUtility then pcall(_G.RyPrivToggleUtility) end end },
            { "INVIS",    function() if _G.RyPrivToggleInvisPanel then pcall(_G.RyPrivToggleInvisPanel) end end },
            { "INSTANT",  function() if _G.RyPrivToggleInstantSteal then pcall(_G.RyPrivToggleInstantSteal) end end,
                          function() return _G.RyPrivInstantSteal == true end },
            { "AUTO TP",  function()
                              _G.RyPrivAutoTP = not (_G.RyPrivAutoTP == true)
                              if _G.RyPrivRepaintAutoTP then pcall(_G.RyPrivRepaintAutoTP) end
                              pcall(_G.RyPrivSaveSettings)
                          end,
                          function() return _G.RyPrivAutoTP == true end },
            { "TP NOW",   function() task.spawn(function()
                              if _G.RyPrivStartSideTP then pcall(_G.RyPrivStartSideTP) end
                          end) end },
            { "DROP",     function() task.spawn(function()
                              if _G.DropBrainrot then pcall(_G.DropBrainrot) end
                          end) end },
            { "RESET",    function() task.spawn(function()
                              if _G.RyPrivInstaReset then pcall(_G.RyPrivInstaReset) end
                          end) end },
        }

        local panelH = (BTN_H + PAD) * #rows + PAD + 28
        local root = round(mk("Frame", sg, {
            Name = "Dock", Active = true, BorderSizePixel = 0,
            BackgroundColor3 = C.bg, BackgroundTransparency = 0.08,
            Size = UDim2.fromOffset(BTN_W + PAD * 2, panelH),
            Position = UDim2.fromOffset(12, 120),
        }), 10)
        mk("UIStroke", root, { Color = C.line, Thickness = 1, Transparency = 0.45 })

        -- the handle doubles as the drag grip and the collapse control
        local head = mk("TextButton", root, {
            Name = "Head", Size = UDim2.new(1, 0, 0, 26),
            BackgroundTransparency = 1, Text = "", AutoButtonColor = false,
            Active = true,
        })
        mk("TextLabel", head, {
            Size = UDim2.new(1, -10, 1, 0), Position = UDim2.fromOffset(10, 0),
            BackgroundTransparency = 1, Text = "RYMOGS", Font = FB, TextSize = 11,
            TextColor3 = C.dim, TextXAlignment = Enum.TextXAlignment.Left,
        })

        local body = mk("Frame", root, {
            Name = "Body", Position = UDim2.fromOffset(PAD, 28),
            Size = UDim2.new(1, -PAD * 2, 1, -28 - PAD),
            BackgroundTransparency = 1,
        })
        mk("UIListLayout", body, {
            Padding = UDim.new(0, PAD), SortOrder = Enum.SortOrder.LayoutOrder,
        })

        local paints = {}
        for i, r in ipairs(rows) do
            local label, fn, isOn = r[1], r[2], r[3]
            local b = round(mk("TextButton", body, {
                Size = UDim2.new(1, 0, 0, BTN_H), BackgroundColor3 = C.btn,
                Text = label, Font = FB, TextSize = 12, TextColor3 = C.txt,
                AutoButtonColor = false, BorderSizePixel = 0, LayoutOrder = i,
            }), 8)
            local st = mk("UIStroke", b, { Color = C.line, Thickness = 1, Transparency = 0.6 })
            local function paint()
                if not isOn then return end
                local on = isOn() == true
                b.BackgroundColor3 = on and C.on or C.btn
                b.TextColor3 = on and Color3.fromRGB(12, 14, 16) or C.txt
                st.Transparency = on and 0.25 or 0.6
            end
            b.MouseButton1Click:Connect(function()
                pcall(fn)
                paint()
            end)
            paint()
            if isOn then paints[#paints + 1] = paint end
        end

        -- Collapsed, the dock is just the handle, so it takes almost no screen.
        local open = true
        local fullH = panelH
        head.MouseButton1Click:Connect(function()
            open = not open
            body.Visible = open
            root.Size = UDim2.fromOffset(BTN_W + PAD * 2, open and fullH or 26)
        end)

        -- same drag and position persistence every other panel uses
        if _G.RyPrivPanelDrag then
            pcall(_G.RyPrivPanelDrag, root, head, "_ryPriv_mbX", "_ryPriv_mbY")
            pcall(_G.RyPrivApplyPos, root, "_ryPriv_mbX", "_ryPriv_mbY", nil)
            pcall(_G.RyPrivRegisterPanel, root, "_ryPriv_mbX", "_ryPriv_mbY")
        end

        -- keep the on/off buttons honest while the dock is open
        task.spawn(function()
            while sg.Parent do
                task.wait(0.4)
                if open then
                    for _, f in ipairs(paints) do pcall(f) end
                end
            end
        end)

        _G.RyPrivMobileDock = root
        _G.RyPrivToggleMobileDock = function() sg.Enabled = not sg.Enabled end
    end)
end
