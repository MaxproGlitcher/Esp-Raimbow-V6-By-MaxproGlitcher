# Esp-Raimbow-V5-By-MaxproGlitcher
# This is a very complex esp raimbow script 
# The script isn't 100% finished, but it's only half-done, and I'm continuing to improve it. 


local lpr = game.Players.LocalPlayer
local camera = game:GetService("Workspace").CurrentCamera
local worldToViewportPoint = camera.WorldToViewportPoint

_G.Teamcheck = false -- La valeur "true" est utilisée pour le contrôle de l'équipe.

-- Fonction permettant de générer des couleurs arc-en-ciel
local function rainbowColor(frequency)
    local r = math.floor(math.sin(frequency + 0) * 127 + 128)
    local g = math.floor(math.sin(frequency + 2) * 127 + 128)
    local b = math.floor(math.sin(frequency + 4) * 127 + 128)
    return Color3.fromRGB(r, g, b)
end

-- Tableaux contenant les lignes de dessin, le BillboardGui et les boîtes de frappe de chaque joueur.
local playerTracers = {}
local playerInfoTexts = {}
local playerHitboxes = {}

-- Créer et configurer BillboardGui pour le joueur
local function createBillboardGui(player)
    local billboardGui = Instance.new("BillboardGui", player.Character)
    billboardGui.Size = UDim2.new(0, 200, 0, 75)
    billboardGui.StudsOffset = Vector3.new(0, 3, 0)
    billboardGui.AlwaysOnTop = true

    local nameLabel = Instance.new("TextLabel", billboardGui)
    nameLabel.Size = UDim2.new(1, 0, 0.33, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.TextColor3 = rainbowColor(0)
    nameLabel.TextStrokeTransparency = 0
    nameLabel.TextSize = 14
    nameLabel.Text = player.Name

    local distanceLabel = Instance.new("TextLabel", billboardGui)
    distanceLabel.Position = UDim2.new(0, 0, 0.33, 0)
    distanceLabel.Size = UDim2.new(1, 0, 0.33, 0)
    distanceLabel.BackgroundTransparency = 1
    distanceLabel.TextColor3 = rainbowColor(0)
    distanceLabel.TextStrokeTransparency = 0
    distanceLabel.TextSize = 14

    local healthLabel = Instance.new("TextLabel", billboardGui)
    healthLabel.Position = UDim2.new(0, 0, 0.66, 0)
    healthLabel.Size = UDim2.new(1, 0, 0.33, 0)
    healthLabel.BackgroundTransparency = 1
    healthLabel.TextColor3 = rainbowColor(0)
    healthLabel.TextStrokeTransparency = 0
    healthLabel.TextSize = 14

    return billboardGui, nameLabel, distanceLabel, healthLabel
end

-- Mettre à jour le texte sur la distance et la santé
local function updatePlayerInfo(player, distanceLabel, healthLabel, time)
    local rootPart = player.Character:FindFirstChild("HumanoidRootPart")
    if rootPart then
        local distance = (rootPart.Position - lpr.Character.HumanoidRootPart.Position).magnitude
        distanceLabel.Text = string.format("Distance : %.2f", distance)
        distanceLabel.TextColor3 = rainbowColor(time + player.UserId)
    end

    local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        local health = humanoid.Health
        healthLabel.Text = string.format("Santé : %.0f", health)
        healthLabel.TextColor3 = rainbowColor(time + player.UserId)
    end
end

-- Créer une hitbox pour le joueur
local function createHitbox()
    local hitbox = Drawing.new("Square")
    hitbox.Thickness = 2
    hitbox.Transparency = 1
    return hitbox
end

-- Mise à jour des traceurs, des textes de distance et des hitboxes à chaque image
local function update()
    local time = tick()
    for _, player in pairs(game.Players:GetPlayers()) do
        if player ~= lpr and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local rootPart = player.Character:FindFirstChild("HumanoidRootPart")
            local head = player.Character:FindFirstChild("Head")
            local screenPos, onScreen = worldToViewportPoint(camera, rootPart.Position)
            local headPos, headOnScreen = worldToViewportPoint(camera, head.Position)

            if _G.Teamcheck and player.Team == lpr.Team then
                if playerTracers[player] then playerTracers[player].Visible = false end
                if playerInfoTexts[player] then playerInfoTexts[player].gui.Enabled = false end
                if playerHitboxes[player] then playerHitboxes[player].Visible = false end
            else
                if not playerTracers[player] then
                    playerTracers[player] = Drawing.new("Line")
                end
                if not playerInfoTexts[player] then
                    local billboardGui, nameLabel, distanceLabel, healthLabel = createBillboardGui(player)
                    playerInfoTexts[player] = { gui = billboardGui, name = nameLabel, distance = distanceLabel, health = healthLabel }
                end
                if not playerHitboxes[player] then
                    playerHitboxes[player] = createHitbox()
                end

                local tracer = playerTracers[player]
                local billboardGui = playerInfoTexts[player].gui
                local distanceLabel = playerInfoTexts[player].distance
                local healthLabel = playerInfoTexts[player].health
                local hitbox = playerHitboxes[player]

                if onScreen then
                    tracer.From = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y)
                    tracer.To = Vector2.new(screenPos.X, screenPos.Y)
                    tracer.Color = rainbowColor(time + player.UserId)
                    tracer.Thickness = 2
                    tracer.Transparency = 1
                    tracer.Visible = true

                    if headOnScreen then
                        updatePlayerInfo(player, distanceLabel, healthLabel, time)
                        billboardGui.Enabled = true
                        billboardGui.StudsOffset = Vector3.new(0, 3, 0)
                        local nameLabel = playerInfoTexts[player].name
                        nameLabel.TextColor3 = rainbowColor(time + player.UserId)

                        -- Mise à jour de la hitbox
                        local hitboxSize = (headPos - screenPos).magnitude * 2
                        hitbox.Size = Vector2.new(hitboxSize, hitboxSize * 2.5)
                        hitbox.Position = Vector2.new(screenPos.X - hitboxSize / 2, screenPos.Y - hitboxSize * 1.25)
                        hitbox.Color = rainbowColor(time + player.UserId)
                        hitbox.Visible = true
                    else
                        billboardGui.Enabled = false
                        hitbox.Visible = false
                    end
                else
                    tracer.Visible = false
                    billboardGui.Enabled = false
                    hitbox.Visible = false
                end
            end
        else
            if playerTracers[player] then playerTracers[player].Visible = false end
            if playerInfoTexts[player] then playerInfoTexts[player].gui.Enabled = false end
            if playerHitboxes[player] then playerHitboxes[player].Visible = false end
        end
    end
end

-- Fonction de gestion des nouveaux joueurs
local function onPlayerAdded(player)
    player.CharacterAdded:Connect(function()
        wait(1) -- Donner au personnage le temps de se charger
        update()
    end)
end

-- Connecter l'événement PlayerAdded
game.Players.PlayerAdded:Connect(onPlayerAdded)

-- Mise à jour des traceurs, des textes de distance et des hitboxes à chaque image
game:GetService("RunService").RenderStepped:Connect(update)

-- Nettoyer les lieux après le départ d'un joueur
game.Players.PlayerRemoving:Connect(function(player)
    if playerTracers[player] then playerTracers[player]:Remove() playerTracers[player] = nil end
    if playerInfoTexts[player] then playerInfoTexts[player].gui:Destroy() playerInfoTexts[player] = nil end
    if playerHitboxes[player] then playerHitboxes[player]:Remove() playerHitboxes[player] = nil end
end)

-- Gérer les joueurs existants
for _, player in pairs(game.Players:GetPlayers()) do
    onPlayerAdded(player)
end
