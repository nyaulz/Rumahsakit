-- Script Auto Surg Support Magplant & Genta Hax + LimitTea Logic (Clean Final Build v2)
-- Remaked & Configured By #nYauL1763 | Type /surg To Start Script Menu

Config = {
   AutoSurg = true,
   AutoPut = false,
   LegitMode = false,
   UseMagplant = false,
   DelayMS = 1000,
   ItemID = 4296,
   Position = "RIGHT",
   
   -- Config Auto Consume (Genta Hax Style)
   Tea = false,
   Spice = false,
   TeaID = 5114,
   SpiceID = 6912,
   TeaDelay = 125,      -- 2 menit 5 detik (125 detik)
   SpiceDelay = 1805,   -- 30 menit 5 detik (1805 detik)
   Tick = 1
}

local tool = "0"
local isProcessing = false
local isConsumeRunning = false
local teaTimer, spiceTimer = 0, 0
local surgerySuccessCount = 0
local isSurgeryActive = false

local function logMsg(text)
   pcall(function()
      if Log then
         Log(text)
      elseif logToConsole then
         logToConsole(text)
      else
         print("[#nYauL1763-Surg] " .. tostring(text))
      end
   end)
end

function getItemAmount(itemID)
   local inv = getInventory()
   if not inv then return 0 end
   for _, item in pairs(inv) do
      if tonumber(item.id) == tonumber(itemID) then
         return tonumber(item.amount) or 0
      end
   end
   return 0
end

-- Tabel ID Alat Medis Original LIMITTEA
local toolIDs = {
    ["Sponge"]        = 1258,
    ["Scalpel"]       = 1260,
    ["Anesthetic"]    = 1262,
    ["Antibiotic"]    = 1266,
    ["Splint"]        = 1268,
    ["Stitches"]      = 1270,
    ["Fix it"]        = 1296,
    ["Pins"]          = 4308,
    ["Transfusion"]   = 4310,
    ["Defibrillator"] = 4312,
    ["Clamp"]         = 4314,
    ["Ultrasound"]    = 4316,
    ["Lab kit"]       = 4318
}

local function showLegitHelper(toolName, surgeryDialog)
   local toolID = toolIDs[toolName]
   local combinedDialog = surgeryDialog

   if toolID then
      if combinedDialog:find("tool" .. toolID) then
         combinedDialog = combinedDialog:gsub("(%w+|tool" .. toolID .. "|)([^|\n]+)", function(prefix, label)
            return prefix .. "`e" .. toolName
         end, 1)
      end
   end

   if sendVariant then
      sendVariant({
         [0] = "OnDialogRequest",
         [1] = combinedDialog
      })
   end

   logMsg("`6[`6#nYauL1763 Helper Surg`6] `eRecommended: " .. tostring(toolName))
end

-- ==================== UI BUILDERS ====================
function buildSurgMenuDialog()
   local chkSurg = Config.AutoSurg and "1" or "0"
   local chkPut = Config.AutoPut and "1" or "0"
   local chkLegit = Config.LegitMode and "1" or "0"
   local chkMag = Config.UseMagplant and "1" or "0"
   local chkTea = Config.Tea and "1" or "0"
   local chkSpice = Config.Spice and "1" or "0"

   local consumeStatus = isConsumeRunning and "`2[ACTIVE]``" or "`4[INACTIVE]``"
   local world = (getWorld() and getWorld().name) or "EXIT"
   local user = (getLocal() and getLocal().name) or "PLAYER"

   local ui = {}
   table.insert(ui, "add_label_with_icon|big|`6#nYauL1763 AUTO SURGERY KONTOL|left|4300|")
   table.insert(ui, "add_spacer|small|")
   table.insert(ui, "add_label_with_icon|small|`6Enjoy Using Our `6#nYauL1763 AutoSurg! `6/surg To Start|left|4626|")
   table.insert(ui, "add_label_with_icon|small|" .. user .. " `6Currently In World `7" .. world .. " `6!|left|3802|")
   table.insert(ui, "add_label_with_icon|small|`6Successful Surgeries: `2" .. surgerySuccessCount .. " `6Patients Saved|left|4308|")
   table.insert(ui, "add_label_with_icon_button|small|`6<-- Press Icon To Check Script Info|left|9472|btn_about_info|")
   table.insert(ui, "add_spacer|small|")
   
   table.insert(ui, "add_textbox|`6=== Custom Your Settings Here ===|left|32|")
   table.insert(ui, "add_text_input|delay_ms|`wDelay Using Tools (ms):|" .. tostring(Config.DelayMS) .. "|4|")
   table.insert(ui, "add_text_input|item_id|`wCustom Item ID (Dummy):|" .. tostring(Config.ItemID) .. "|4|")
   table.insert(ui, "add_text_input|put_pos|`wPosition Place(UP/MID/LEFT/RIGHT):|" .. Config.Position .. "|5|")
   table.insert(ui, "add_spacer|small|")

   table.insert(ui, "add_textbox|`6=== AutoSurg Feature Toggles ===|left|5114|")
   table.insert(ui, "add_checkbox|enable_surg|`wEnable Auto Surgery|" .. chkSurg .. "|")
   table.insert(ui, "add_checkbox|enable_put|`wEnable Auto Put / Place Item|" .. chkPut .. "|")
   table.insert(ui, "add_checkbox|enable_legit|`wEnable Legit Helper Mode|" .. chkLegit .. "|")
   table.insert(ui, "add_checkbox|enable_mag|`wEnable Using Magplant Remote|" .. chkMag .. "|")
   table.insert(ui, "add_spacer|small|")

   table.insert(ui, "add_textbox|`6=== Auto Consume Manager " .. consumeStatus .. " ===|left|")
   table.insert(ui, "add_checkbox|cb_tea|`wTea (2m) [Stock: `e" .. getItemAmount(Config.TeaID) .. "`w]|" .. chkTea .. "|")
   table.insert(ui, "add_checkbox|cb_spice|`wSkill Spice (30m) [Stock: `e" .. getItemAmount(Config.SpiceID) .. "`w]|" .. chkSpice .. "|")
   table.insert(ui, "add_text_input|tea_id|`wTea Item ID:|" .. tostring(Config.TeaID) .. "|5|")
   table.insert(ui, "add_text_input|spice_id|`wSkill Spice Item ID:|" .. tostring(Config.SpiceID) .. "|5|")
   table.insert(ui, "add_spacer|small|")

   table.insert(ui, "add_quick_exit||")
   table.insert(ui, "end_dialog|autosurg_settings|Cancel|Save Settings|")

   return table.concat(ui, "\n")
end

function buildAboutDialog()
   local ui = {}
   table.insert(ui, "add_label_with_icon|big|`6Welcome To #nYauL1763 Script Info|left|9472|")
   table.insert(ui, "add_spacer|small|")
   table.insert(ui, "add_label_with_icon|small|`wScript Remaked By: `6#nYauL1763|left|3524|")
   table.insert(ui, "add_label_with_icon|small|`wFeatures Added: `6Legit Helper & Auto Consume|left|1366|")
   table.insert(ui, "add_label_with_icon|small|`6#nYauL1763:`w Main Scripter & UI Customizer|left|998|")
   table.insert(ui, "add_spacer|medium|")
   table.insert(ui, "add_button|btn_prev_menu|Back to Menu|0|0|")
   table.insert(ui, "end_dialog|autosurg_about|||")

   return table.concat(ui, "\n")
end

function showSurgMenu()
   local var = {}
   var[0] = "OnDialogRequest"
   var[1] = buildSurgMenuDialog()
   sendVariant(var, -1, 0)
end

function showAboutMenu()
   local var = {}
   var[0] = "OnDialogRequest"
   var[1] = buildAboutDialog()
   sendVariant(var, -1, 0)
end

-- ==================== AUTO CONSUME LOGIC ====================
function consumeItem(itemID, itemName)
   local count = getItemAmount(itemID)
   if count <= 0 then
      return false
   end

   local player = getLocal()
   if player then
      local tileX = math.floor(player.pos.x / 32)
      local tileY = math.floor(player.pos.y / 32)
      
      local success = pcall(function()
         requestTileChange(tileX, tileY, itemID)
      end)
      
      return success
   end
   return false
end

function consumeLoop()
   if Config.Tea then consumeItem(Config.TeaID, "Tea") end
   if Config.Spice then 
      sleep(1200) 
      consumeItem(Config.SpiceID, "Skill Spice") 
   end

   teaTimer, spiceTimer = 0, 0

   while isConsumeRunning do
      sleep(Config.Tick * 1000)
      if not isConsumeRunning then break end

      teaTimer = teaTimer + Config.Tick
      spiceTimer = spiceTimer + Config.Tick

      if Config.Tea and teaTimer >= Config.TeaDelay then
         consumeItem(Config.TeaID, "Tea")
         teaTimer = 0
      end
      if Config.Spice and spiceTimer >= Config.SpiceDelay then
         consumeItem(Config.SpiceID, "Skill Spice")
         spiceTimer = 0
      end
   end
end

-- ==================== SURGERY AUTOMATION ====================
function auto()
    local itool = toolIDs[tool]
    if not itool then return end
    
    sendPacket(2, "action|dialog_return\ndialog_name|surgery\nbuttonClicked|tool" .. itool)
    logMsg("`6[`6#nYauL1763 Tools`6] `c" .. tool)
end

function afterSurg()
   sleep(1000)
   isSurgeryActive = false
   if not Config.AutoPut or not Config.AutoSurg then return end

   local checkID = Config.UseMagplant and 5640 or Config.ItemID
   local count = getItemAmount(checkID)
   if count <= 0 then
      logMsg("`6[`6#nYauL1763 AutoSurg`6] `4Item ID " .. checkID .. " Empty! Auto Put Deactivated.")
      Config.AutoPut = false
      return
   end

   local localPlayer = getLocal()
   if not localPlayer then return end

   local px = math.floor(localPlayer.pos.x / 32)
   local py = math.floor(localPlayer.pos.y / 32)

   local targetX, targetY = px, py
   local posUpper = Config.Position:upper()

   if posUpper == "UP" then targetY = py - 1
   elseif posUpper == "LEFT" then targetX = px - 1
   elseif posUpper == "RIGHT" then targetX = px + 1
   elseif posUpper == "MID" then targetX, targetY = px, py
   else targetX = px + 1 end

   requestTileChange(targetX, targetY, checkID)
   sleep(1500)

   sendPacketRaw(false, {
      type = 3,
      value = 32,
      x = localPlayer.pos.x,
      y = localPlayer.pos.y,
      punchx = targetX,
      punchy = targetY
   })
end

-- ==================== HOOKS ====================
AddHook("OnTextPacket", "SurgCmdHook", function(type_pkt, pkt)
   if not pkt then return false end

   if pkt:find("/surg") or pkt:find("/consume") then
      showSurgMenu()
      return true
   end

   if pkt:find("dialog_name|autosurg_settings") then
      local clicked = pkt:match("buttonClicked|(%S+)")
      
      if clicked == "btn_about_info" then showAboutMenu(); return true end

      Config.AutoSurg = (pkt:match("enable_surg|(%d+)") == "1")
      Config.AutoPut = (pkt:match("enable_put|(%d+)") == "1")
      Config.LegitMode = (pkt:match("enable_legit|(%d+)") == "1")
      Config.UseMagplant = (pkt:match("enable_mag|(%d+)") == "1")
      Config.Tea = (pkt:match("cb_tea|(%d+)") == "1")
      Config.Spice = (pkt:match("cb_spice|(%d+)") == "1")

      Config.DelayMS = tonumber(pkt:match("delay_ms|(%d+)")) or Config.DelayMS
      Config.ItemID = tonumber(pkt:match("item_id|(%d+)")) or Config.ItemID
      Config.Position = tostring(pkt:match("put_pos|(%a+)") or Config.Position):upper()
      Config.TeaID = tonumber(pkt:match("tea_id|(%d+)")) or Config.TeaID
      Config.SpiceID = tonumber(pkt:match("spice_id|(%d+)")) or Config.SpiceID

      -- Rangkum Status Toggles Feature
      local activeToggles = {}
      if Config.AutoSurg then table.insert(activeToggles, "Auto Surgery") end
      if Config.AutoPut then table.insert(activeToggles, "Auto Place") end
      if Config.LegitMode then table.insert(activeToggles, "Helper Mode") end
      if Config.UseMagplant then table.insert(activeToggles, "Magplant") end
      local togglesStr = #activeToggles > 0 and table.concat(activeToggles, " / ") or "None"

      -- Rangkum Status Auto Consume + Stok
      local activeConsume = {}
      if Config.Tea then 
         table.insert(activeConsume, "Tea (" .. getItemAmount(Config.TeaID) .. ")") 
      end
      if Config.Spice then 
         table.insert(activeConsume, "Skill Spice (" .. getItemAmount(Config.SpiceID) .. ")") 
      end
      local consumeStr = #activeConsume > 0 and table.concat(activeConsume, " + ") or "OFF"

      -- Output Custom Log Rangkuman Save Settings
      logMsg("`6[Auto Surg Settings] `2" .. togglesStr .. " `o| `9Auto Consume : `2" .. consumeStr)

      if Config.Tea or Config.Spice then
         if not isConsumeRunning then
            isConsumeRunning = true
            runThread(consumeLoop)
         else
            isConsumeRunning = false
            sleep(300)
            isConsumeRunning = true
            runThread(consumeLoop)
         end
      else
         isConsumeRunning = false
      end

      return true
   end

   if pkt:find("dialog_name|autosurg_about") then
      if pkt:match("buttonClicked|btn_prev_menu") then showSurgMenu(); return true end
   end

   return false
end)

AddHook("OnVarlist", "SurgMainHook", function(var)
   if not var then return false end

   local v0 = tostring(var[0] or var.v0 or "")
   local v1 = tostring(var[1] or var.v1 or "")

   if v0 == "OnConsoleMessage" and v1:find("YOU SAVED YOUR PATIENT!") then
      surgerySuccessCount = surgerySuccessCount + 1
      logMsg("`6Successful Surgeries: `2" .. surgerySuccessCount)
      if Config.AutoSurg then 
         runThread(afterSurg) 
      end
      return false
   end

   -- Auto Skip Dialog Dummy (100% Silent)
   if Config.AutoSurg and not Config.LegitMode and v0 == "OnDialogRequest" and (v1:find("Anatomical Dummy") or v1:find("dialog_name|surge")) then
      local tilex = v1:match("embed_data|tilex|(%d+)")
      local tiley = v1:match("embed_data|tiley|(%d+)")

      if tilex and tiley then
         runThread(function()
            sleep(300)
            sendPacket(2, "action|dialog_return\ndialog_name|surge\ntilex|" .. tilex .. "\ntiley|" .. tiley .. "\nbuttonClicked|okay")
         end)
         return true
      end
   end

   if not Config.AutoSurg then return false end
   if not (v0 == "OnDialogRequest" and (v1:find("surgery") or v1:find("tool1260"))) then return false end
   
   -- Print "Ready Surg" Cuman Sekali di Awal Pas Operasi Dimulai
   if not isSurgeryActive then
      isSurgeryActive = true
      logMsg("`6[`6#nYauL1763 AutoSurg`6] `2Ready Surg...")
   end

   if isProcessing then return false end

   -- Logika LimitTea Diagnosis Murni
   if v1:find("`4The patient wakes up!") and v1:find("tool1262") then tool = "Anesthetic"
   elseif v1:find("`4The patient screams and flails!") and v1:find("tool1262") then tool = "Anesthetic"
   elseif v1:find("Status: `4Heart stopped!(.+)") and v1:find("tool4312") then tool = "Defibrillator"
   elseif v1:find("Status: `6Coming to(.+)") and v1:find("tool1262") then tool = "Anesthetic"
   elseif v1:find("Pulse: `4(.+)") and v1:find("tool4310") then tool = "Transfusion"
   elseif v1:find("Temp: `4(%d+)(.+)") and v1:find("tool1266") then tool = "Antibiotic"
   elseif v1:find("Temp: `4(%d+)(.+)") and v1:find("tool4318") then tool = "Lab kit"
   elseif v1:find("Temp: `6(%d+)(.+)") and v1:find("tool1266") then tool = "Antibiotic"
   elseif v1:find("Temp: `6(%d+)(.+)") and v1:find("tool4318") then tool = "Lab kit"
   elseif v1:find("Temp: `3(%d+)(.+)") and v1:find("tool1266") then tool = "Antibiotic"
   elseif v1:find("Temp: `3(%d+)(.+)") and v1:find("tool4318") then tool = "Lab kit"
   elseif v1:find("Patient is losing blood `4very quickly!(.+)") and v1:find("tool4314") then tool = "Clamp"
   elseif v1:find("Patient is losing blood `4very quickly!(.+)") and v1:find("tool1270") then tool = "Stitches"
   elseif v1:find("Patient is `6losing blood!(.+)") and v1:find("tool4314") then tool = "Clamp"
   elseif v1:find("Patient is `6losing blood!(.+)") and v1:find("tool1270") then tool = "Stitches"
   elseif v1:find("Incisions: `20(.+)") and v1:find("tool1296") then tool = "Fix it"
   elseif v1:find("Incisions: `30(.+)") and v1:find("tool1296") then tool = "Fix it"
   elseif v1:find("The patient has not been diagnosed.") and v1:find("tool4316") then tool = "Ultrasound"
   elseif v1:find("Status: `4Awake(.+)") and v1:find("tool1262") then tool = "Anesthetic"
   elseif v1:find("Bones: `6(.+) broken``") and v1:find("tool1268") then tool = "Splint"
   elseif v1:find("Bones: `4(.+) broken``") and v1:find("tool1268") then tool = "Splint"
   elseif v1:find("Patient broke his arm.") and v1:find("tool1270") then tool = "Stitches"
   elseif v1:find("Status: `3Awake(.+)") and v1:find("tool1262") then tool = "Anesthetic"
   elseif v1:find("Pulse: `6(.+)") and v1:find("tool4310") then tool = "Transfusion"
   elseif v1:find("`4You can't see what you are doing!(.+)") and v1:find("tool1258") then tool = "Sponge"
   elseif v1:find("tool1296") and v1:find("tool1270") then tool = "Stitches"
   elseif v1:find("Bones: `6(.+), `6(.+) shattered(.+)") and v1:find("tool4308") then tool = "Pins"
   elseif v1:find("Bones: `6(.+), `6(.+) shattered(.+)") and v1:find("tool1260") then tool = "Scalpel"
   elseif v1:find("Bones: `4(.+), `6(.+) shattered(.+)") and v1:find("tool4308") then tool = "Pins"
   elseif v1:find("Bones: `4(.+), `6(.+) shattered(.+)") and v1:find("tool1260") then tool = "Scalpel"
   elseif v1:find("Bones: `6(.+), `4(.+) shattered(.+)") and v1:find("tool4308") then tool = "Pins"
   elseif v1:find("Bones: `6(.+), `4(.+) shattered(.+)") and v1:find("tool1260") then tool = "Scalpel"
   elseif v1:find("Bones: `4(.+), `4(.+) shattered(.+)") and v1:find("tool4308") then tool = "Pins"
   elseif v1:find("Bones: `4(.+), `4(.+) shattered(.+)") and v1:find("tool1260") then tool = "Scalpel"
   elseif v1:find("Bones: `6(.+) shattered(.+)") and v1:find("tool4308") then tool = "Pins"
   elseif v1:find("Bones: `6(.+) shattered(.+)") and v1:find("tool1260") then tool = "Scalpel"
   elseif v1:find("Bones: `4(.+) shattered(.+)") and v1:find("tool4308") then tool = "Pins"
   elseif v1:find("Bones: `4(.+) shattered(.+)") and v1:find("tool1260") then tool = "Scalpel"
   elseif v1:find("Patient broke his leg.") and v1:find("tool1270") then tool = "Stitches"
   elseif v1:find("Patient is losing blood `3slowly.(.+)") and v1:find("tool4314") then tool = "Clamp"
   elseif v1:find("tool1260") then tool = "Scalpel"
   else return false end

   if Config.LegitMode then
      showLegitHelper(tool, v1)
      return true
   end

   isProcessing = true
   runThread(function()
      sleep(Config.DelayMS)
      auto()
      sleep(100)
      isProcessing = false
   end)
   return true
end)

logMsg("`6[`6#nYauL1763 SYSTEM`6] `aAuto Surgery Script Ready! Type /surg")
showSurgMenu()
