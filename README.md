local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

local player = Players.LocalPlayer

print("Sistema carregado!")

local Modules = ReplicatedStorage:WaitForChild("Modules")

local AimAssist = require(Modules:WaitForChild("AimAssist"))
local Highlight = require(Modules:WaitForChild("Highlight"))

-- teste highlight
Highlight.Toggle(player, true)
