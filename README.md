-- Prevenção de execução múltipla no mesmo ambiente de execução
if _G.LobbyTeleporterExecuted then
    return
end
_G.LobbyTeleporterExecuted = true

local Players = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")
local Workspace = game:GetService("Workspace")
local Player = Players.LocalPlayer

-- Função para reexecutar o script automaticamente ao trocar de servidor baixando diretamente do link
local queueOnTeleport = queue_on_teleport or syn.queue_on_teleport or fluxus and fluxus.queue_on_teleport
if queueOnTeleport then
    pcall(function()
        queueOnTeleport([[
            loadstring(game:HttpGet("https://raw.githubusercontent.com/sla35888/Slapple-farm/refs/heads/main/README.md"))()
        ]])
    end)
end

-- Função para executar o serverhop de forma segura
local function ServerHop()
    local success, err = pcall(function()
        local servers = {}
        local req = game:HttpGet("https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Asc&limit=100")
        local body = game:GetService("HttpService"):JSONDecode(req)
        
        if body and body.data then
            for _, s in ipairs(body.data) do
                if s.playing < s.maxPlayers and s.id ~= game.JobId then
                    table.insert(servers, s.id)
                end
            end
        end
        
        if #servers > 0 then
            TeleportService:TeleportToPlaceInstance(game.PlaceId, servers[math.random(1, #servers)], Player)
        else
            TeleportService:Teleport(game.PlaceId, Player)
        end
    end)
    
    if not success then
        TeleportService:Teleport(game.PlaceId, Player)
    end
end

local hrp = Player.Character and Player.Character:FindFirstChild("HumanoidRootPart")
if not hrp then
    Player.CharacterAdded:Wait()
    hrp = Player.Character:WaitForChild("HumanoidRootPart")
end

-- Etapa 1: Teleporte imediato para Lobby > Teleport1
local lobbyTeleport1 = Workspace:FindFirstChild("Lobby") and Workspace.Lobby:FindFirstChild("Teleport1")
if lobbyTeleport1 then
    if lobbyTeleport1:IsA("Model") then
        hrp.CFrame = lobbyTeleport1:GetPivot()
    elseif lobbyTeleport1:IsA("BaseBase") or lobbyTeleport1:IsA("BasePart") then
        hrp.CFrame = lobbyTeleport1.CFrame
    end
end

-- Espera exatamente 0.3 segundos e aguarda receber uma Tool no inventário ou personagem
task.wait(0.3)

local function HasTool()
    if Player.Character then
        if Player.Character:FindFirstChildOfClass("Tool") then
            return true
        end
    end
    if Player.Backpack and Player.Backpack:FindFirstChildOfClass("Tool") then
        return true
    end
    return false
end

-- Fica em espera até receber uma Tool (slapple)
while not HasTool() do
    task.wait(0.1)
end

-- Etapa 2: Localizar a pasta Arena > island5 > Slapples > Slapple > Glove
local arenaFolder = Workspace:FindFirstChild("Arena")
local island5 = arenaFolder and arenaFolder:FindFirstChild("island5")
local slapples = island5 and island5:FindFirstChild("Slapples")
local slapple = slapples and slapples:FindFirstChild("Slapple")
local gloveFolder = slapple and slapple:FindFirstChild("Glove")

if not gloveFolder then
    ServerHop()
    return
end

-- Coleta todas as luvas dentro da pasta Glove sem verificar transparência
local glovesToVisit = {}
for _, child in ipairs(gloveFolder:GetChildren()) do
    if child:IsA("BasePart") then
        table.insert(glovesToVisit, child)
    elseif child:IsA("Model") then
        table.insert(glovesToVisit, child)
    end
end

-- Se não houver luvas na pasta, executa serverhop
if #glovesToVisit == 0 then
    ServerHop()
    return
end

-- Etapa 3: Teleportar para cada glove existente
for _, gloveObj in ipairs(glovesToVisit) do
    if gloveObj:IsA("BasePart") then
        hrp.CFrame = gloveObj.CFrame + Vector3.new(0, 3, 0)
    elseif gloveObj:IsA("Model") then
        hrp.CFrame = gloveObj:GetPivot() + Vector3.new(0, 3, 0)
    end
    task.wait(0.5) -- Pequeno respiro entre os teleports para estabilizar
end

-- Ao terminar de percorrer todas as luvas, realiza o serverhop
ServerHop()
