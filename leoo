--[[
    LEO HUB | Painel com navegação, filtros e miniaturas dos pets
    Uso no Studio: LocalScript em StarterPlayer > StarterPlayerScripts.

    SEÇÕES: configurações / utilidades / base automática / interface /
            dados e imagens / miniaturas / ações / cartões / atualização /
            controles e encerramento.

    Imagens: configuração manual > dados do pet > interface vinculada >
    recursos com o mesmo nome em ReplicatedStorage > miniatura 3D.
    Não pesquisa fotos na internet e não executa módulos do jogo.
    Catálogo independente do spawn: modelos/definições replicados + espécies
    observadas + CONFIG.Collection.PetCatalog + lista em StringValue replicada.
    Não pode descobrir espécies armazenadas exclusivamente no servidor.
    Reset: troca do Runtime, atributo de rodada, ou esgotamento da população anterior.
    Se o jogo reaproveitar modelos sem sinal de rodada, configure ResetAttributes.
    A base é localizada no mapa pelo dono, vínculo do jogador ou placa com nome/retrato.
    O destino é um ponto identificado DENTRO da base encontrada.
    Nunca registra a posição atual do jogador nem o local onde o script iniciou.
    Se a base ou o ponto não puderem ser identificados, o retorno fica indisponível.
    CONFIG.Base contém convenções configuráveis, não a estrutura confirmada do jogo.
    TP e interação continuam sujeitos às regras e à validação do jogo.
]]

-- 1. CONFIGURAÇÕES ------------------------------------------------------------

local CONFIG = {
    Branding = {
        Title = "LEO HUB",
        -- Use apenas a nova logo do fantasma; ID opcional caso publique a imagem no Roblox.
        LogoAssetId = "",
        -- No Studio, alternativa: imagem publicada chamada LeoHubGhostLogo no ReplicatedStorage.
        ReplicatedLogoName = "LeoHubGhostLogo",
        -- Caminho exclusivo da nova logo para evitar carregar arquivos antigos.
        LocalLogoFile = "LEO_HUB_FANTASMA.png",
        -- Ícones da navegação (imagem de sprite):
        -- No Studio, publique LEO_HUB_ICONES.png como imagem e informe o ID.
        IconsAssetId = "",
        LocalIconsFile = "LEO_HUB_ICONES.png",
    },
    RuntimeName = "Runtime",
    MaintenanceSeconds = 2,
    -- Evita varreduras completas desnecessárias; mantém conferência periódica.
    BaseRescanSeconds = 18,
    BaseRetrySeconds = 4,
    PreviewDebounceSeconds = 0.11,
    PreviewBatchSize = 1,
    Collection = {
        ResetSettleSeconds = 0.75,
        BetweenPetsSeconds = 0.35,
        -- Agrupa spawns, remoções e mudanças do mesmo pet sem atualizar a lista em loop.
        PetUpdateDebounceSeconds = 0.23,
        -- Mudanças contínuas em textos/valores do pet não causam redesenho a cada frame.
        PetChangeMinimumSeconds = 0.55,
        PetUpdateBatchSize = 3,
        ResetAttributes = { "RoundId", "WaveId", "ResetVersion" },
        -- Pastas que podem conter TODAS as espécies, sem precisar aparecer no Runtime.
        CatalogFolders = {
            "Pets", "PetModels", "PetTemplates", "PetCatalog", "PetDefinitions",
            "Exus", "ExuModels", "ExuCatalog", "ExuDefinitions", "ExuTemplates",
            "AllExus", "AllPets", "PetData", "ExuData", "PetList", "ExuList",
            "PetAssets", "ExuAssets", "PetStorage", "ExuStorage",
        },
        -- Opcional no ReplicatedStorage: StringValue com JSON array ou nomes separados por linha.
        CatalogValueNames = { "LeoHubPetCatalog", "AllPetNames", "AllExuNames", "PetNames", "ExuNames" },
        NameFields = { "PetName", "Species", "SpeciesName", "PetType", "TypeName" },
        -- Complete com nomes oficiais caso só o servidor conheça todas as espécies.
        -- Exemplo: PetCatalog = { "Exu Brasa", "Exu Capa Preta" }
        PetCatalog = {},
        -- Variação do nome no mapa -> nome oficial da espécie.
        Aliases = {},
    },
    Width = 920,
    Height = 560,
    ImageTimeout = 5,
    MaxPreviewParts = 350,
    -- Opcional: informe o nome exato do pet e um ID de imagem do Roblox.
    -- PetIcons = { ["Exu_NomeDoPet"] = "rbxassetid://123456789" },
    PetIcons = {},
    Ranking = {
        -- Ordem convencional, do comum ao raro; ajuste às categorias reais do jogo.
        -- Categorias desconhecidas são exibidas sem receber um nível inventado.
        Rarities = {
            { name = "Comum", aliases = { "Comum", "Common" } },
            { name = "Incomum", aliases = { "Incomum", "Uncommon" } },
            { name = "Raro", aliases = { "Raro", "Rare" } },
            { name = "Épico", aliases = { "Épico", "Epic" } },
            { name = "Lendário", aliases = { "Lendário", "Legendary" } },
            { name = "Mítico", aliases = { "Mítico", "Mythic", "Mythical" } },
            { name = "Divino", aliases = { "Divino", "Divine" } },
            { name = "Secreto", aliases = { "Secreto", "Secret" } },
        },
        Fields = {
            rarity = { "Rarity", "Raridade", "RarityName", "RarityTier" },
        },
    },
    Base = {
        RefreshSeconds = 2,
        ReturnHeight = 3.5,
        PointMaxDepth = 3,
        ContainerNames = { "Bases", "Plots", "Tycoons", "PlayerBases", "PlayerPlots", "Lotes" },
        OwnerFields = { "Owner", "OwnerId", "OwnerUserId", "OwnerName", "Player", "PlayerId",
            "PlayerUserId", "Dono", "Proprietario" },
        -- Ordem de preferência. Ajuste se o jogo usar nomes diferentes.
        PointNames = { "BaseTeleport", "TeleportPoint", "ReturnPoint", "PlayerSpawn",
            "SpawnPoint", "SpawnLocation", "Spawn", "Entrance", "Entrada" },
    },
}

local THEME = {
    Background = Color3.fromRGB(12, 19, 30),
    Rail = Color3.fromRGB(17, 28, 40),
    Surface = Color3.fromRGB(22, 34, 47),
    Input = Color3.fromRGB(30, 45, 60),
    Border = Color3.fromRGB(52, 72, 88),
    Text = Color3.fromRGB(255, 255, 255),
    Muted = Color3.fromRGB(255, 255, 255),
    Accent = Color3.fromRGB(66, 218, 191),
    Success = Color3.fromRGB(101, 227, 174),
    Warning = Color3.fromRGB(255, 194, 116),
    Hover = Color3.fromRGB(62, 84, 102),
    Highlight = Color3.fromRGB(98, 245, 213),
}

local FLOATING_LOGO_BLUE_BORDER = Color3.fromRGB(36, 115, 255)

local THEME_PRESETS = {
    roxo = {
        Accent = Color3.fromRGB(168, 85, 247),
        Highlight = Color3.fromRGB(208, 162, 255),
        Warning = Color3.fromRGB(255, 194, 116),
    },
    vermelho = {
        Accent = Color3.fromRGB(255, 92, 92),
        Highlight = Color3.fromRGB(255, 160, 160),
        Warning = Color3.fromRGB(255, 203, 124),
    },
    amarelo = {
        Accent = Color3.fromRGB(255, 211, 78),
        Highlight = Color3.fromRGB(255, 236, 156),
        Warning = Color3.fromRGB(255, 186, 92),
    },
    azul = {
        Accent = Color3.fromRGB(36, 115, 255),
        Highlight = Color3.fromRGB(111, 181, 255),
        Warning = Color3.fromRGB(255, 194, 116),
    },
}

local Settings = {
    theme = "azul",
    antiAfk = true,
}

local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local VirtualUser = game:GetService("VirtualUser")
local player = Players.LocalPlayer
if not player then
    warn("Leo Hub: execute este código no cliente, em um LocalScript.")
    return
end

local state = {
    running = true,
    minimized = false,
    page = "home",
    snapshotRows = {},
    rowsByObject = {}, -- Um registro por pet: cartões existentes são reaproveitados.
    showPreviews = true,
    showDistance = true,
    -- O destino pertence à base do mapa, independentemente do personagem.
    base = nil,
    baseObject = nil,
    basePoint = nil,
    baseMessage = "Procurando sua base...",
    baseContainers = {},
    runtime = nil,
    busy = false,
    sortMode = 1,
    cardSequence = 0,
    cards = {},
    connections = {},
    linkedGuis = {},
    resources = {},
    activePrompt = nil,
}

-- 2. UTILIDADES ---------------------------------------------------------------

local function connect(signal, callback)
    local connection = signal:Connect(callback)
    table.insert(state.connections, connection)
    return connection
end

local function create(className, properties, parent)
    local instance = Instance.new(className)
    for key, value in pairs(properties) do instance[key] = value end
    instance.Parent = parent
    return instance
end


local function markSurface(instance, role)
    instance:SetAttribute("SurfaceRole", role)
    return instance
end

local function markText(instance, role)
    instance:SetAttribute("TextRole", role)
    return instance
end

local function round(instance, radius)
    create("UICorner", { CornerRadius = UDim.new(0, radius or 10) }, instance)
end

local function outline(instance)
    local stroke = create("UIStroke", { Color = THEME.Border, Transparency = 0.45 }, instance)
    stroke:SetAttribute("StyledOutline", true)
    return stroke
end

local function label(parent, text, size, position, dimensions, color)
    local role = "text"
    if color == THEME.Muted then role = "muted" elseif color == THEME.Accent then role = "accent" end
    return markText(create("TextLabel", {
        BackgroundTransparency = 1,
        Text = text,
        TextColor3 = Color3.new(1, 1, 1),
        TextStrokeColor3 = Color3.fromRGB(0, 0, 0),
        TextStrokeTransparency = 0.75,
        TextSize = math.max(10, size + 2),
        Font = Enum.Font.FredokaOne,
        TextXAlignment = Enum.TextXAlignment.Left,
        TextTruncate = Enum.TextTruncate.AtEnd,
        Position = position,
        Size = dimensions,
    }, parent), role)
end

local QUICK = TweenInfo.new(0.16, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
local MENU_TWEEN = TweenInfo.new(0.24, Enum.EasingStyle.Quart, Enum.EasingDirection.Out)
local PAGE_TWEEN = TweenInfo.new(0.23, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
local function tween(object, info, goals)
    if not object or not object.Parent then return end
    local animation = TweenService:Create(object, info, goals)
    animation:Play()
    return animation
end

local function button(parent, text, position, dimensions, accent)
    local item = create("TextButton", {
        BackgroundColor3 = accent and THEME.Accent or THEME.Input,
        BorderSizePixel = 0,
        Text = text,
        TextColor3 = Color3.new(1, 1, 1),
        TextStrokeColor3 = Color3.fromRGB(0, 0, 0),
        TextStrokeTransparency = 0.75,
        TextSize = 14,
        Font = Enum.Font.FredokaOne,
        Position = position,
        Size = dimensions,
        AutoButtonColor = false,
        BackgroundTransparency = 0.05,
    }, parent)
    item:SetAttribute("StyledButton", true)
    item:SetAttribute("ButtonAccent", accent and true or false)
    round(item, 12)
    local stroke = create("UIStroke", {
        Color = accent and THEME.Highlight or THEME.Border,
        Thickness = 1.1, Transparency = accent and 0.50 or 0.58,
        ApplyStrokeMode = Enum.ApplyStrokeMode.Border,
    }, item)
    stroke:SetAttribute("ButtonStroke", true)
    create("UIGradient", {
        Rotation = 90,
        Color = ColorSequence.new(Color3.new(1, 1, 1), Color3.fromRGB(207, 222, 234)),
    }, item)
    local scale = create("UIScale", { Scale = 1 }, item)
    item.MouseEnter:Connect(function()
        if not state.running then return end
        tween(scale, QUICK, { Scale = 1.025 })
        tween(stroke, QUICK, { Transparency = 0.12 })
    end)
    item.MouseLeave:Connect(function()
        tween(scale, QUICK, { Scale = 1 })
        tween(stroke, QUICK, { Transparency = 0.58 })
    end)
    item.MouseButton1Down:Connect(function()
        tween(scale, QUICK, { Scale = 0.975 })
    end)
    item.MouseButton1Up:Connect(function()
        tween(scale, QUICK, { Scale = 1.025 })
    end)
    return item
end

local ACCENTS = {
    ["á"] = "a", ["à"] = "a", ["â"] = "a", ["ã"] = "a", ["ä"] = "a",
    ["Á"] = "a", ["À"] = "a", ["Â"] = "a", ["Ã"] = "a", ["Ä"] = "a",
    ["é"] = "e", ["ê"] = "e", ["É"] = "e", ["Ê"] = "e",
    ["í"] = "i", ["Í"] = "i", ["ó"] = "o", ["ô"] = "o", ["õ"] = "o",
    ["Ó"] = "o", ["Ô"] = "o", ["Õ"] = "o", ["ú"] = "u", ["Ú"] = "u",
    ["ü"] = "u", ["Ü"] = "u", ["ç"] = "c", ["Ç"] = "c",
}

local function normalize(value)
    local text = tostring(value)
    for from, to in pairs(ACCENTS) do text = text:gsub(from, to) end
    return string.lower(text)
end

local function cleanName(name)
    local result = name:gsub("^Exu_Exu", "Exu "):gsub("^Exu_", "")
        :gsub("_", " "):gsub("(%l)(%u)", "%1 %2")
    return result
end

local function nameKey(name)
    return normalize(cleanName(name)):gsub("[^%w]", "")
end

local function getRoot()
    local character = player.Character
    local root = character and character:FindFirstChild("HumanoidRootPart")
    local humanoid = character and character:FindFirstChildOfClass("Humanoid")
    if not character or not character:IsDescendantOf(Workspace)
        or not root or not root:IsA("BasePart")
        or not humanoid or humanoid.Health <= 0 then
        return nil, nil
    end
    return character, root
end

local function getPart(object)
    if object:IsA("BasePart") then return object end
    if object:IsA("Model") and object.PrimaryPart then return object.PrimaryPart end
    return object:FindFirstChildWhichIsA("BasePart", true)
end

local function isAvailable(object)
    return state.running and state.runtime and object:IsDescendantOf(state.runtime)
end

-- 3. LOCALIZAÇÃO DA BASE NO MAPA -----------------------------------------------
-- Não usa posição, pivot, distância ou histórico de spawn do personagem.
-- Procura vínculo de propriedade ou placa do jogador e um ponto dentro da base.

local onBaseChanged = function() end

local function key(value)
    return normalize(value):gsub("[^%w]", "")
end

local function makeSet(values)
    local result = {}
    for _, value in ipairs(values) do result[key(value)] = true end
    return result
end

local baseContainerNames = makeSet(CONFIG.Base.ContainerNames)
local ownerFields = makeSet(CONFIG.Base.OwnerFields)
local referenceFields = makeSet({ "Base", "Plot", "Tycoon", "OwnedBase", "AssignedBase" })
local metadataNames = makeSet({ "Data", "Settings", "Config", "Configuration", "Metadata", "Values" })
local pointRanks = {}
for rank, name in ipairs(CONFIG.Base.PointNames) do pointRanks[key(name)] = rank end

local function isMapObject(object)
    if not object or not object:IsDescendantOf(Workspace) then return false end
    local cursor = object
    while cursor and cursor ~= Workspace do
        if cursor:IsA("Model") and cursor:FindFirstChildOfClass("Humanoid") then return false end
        cursor = cursor.Parent
    end
    return true
end

local function indexBaseContainer(object)
    if (object:IsA("Folder") or object:IsA("Model")) and baseContainerNames[key(object.Name)] then
        state.baseContainers[object] = true
    end
end

local function releasePrompt()
    local prompt = state.activePrompt
    state.activePrompt = nil
    if prompt then pcall(function() prompt:InputHoldEnd() end) end
end

local function matchesPlayer(value)
    if typeof(value) == "Instance" then return value == player end
    if typeof(value) == "number" then return value == player.UserId end
    if typeof(value) ~= "string" then return false end
    local text = value:match("^%s*(.-)%s*$")
    return tonumber(text) == player.UserId
        or string.lower(text) == string.lower(player.Name)
        or string.lower(text) == "@" .. string.lower(player.Name)
end

local function readOwner(base)
    local found, ours, conflicting = false, false, false
    local function inspectValue(value)
        found = true
        if matchesPlayer(value) then ours = true else conflicting = true end
    end
    local function inspect(object)
        for name, value in pairs(object:GetAttributes()) do
            if ownerFields[key(name)] then inspectValue(value) end
        end
        for _, child in ipairs(object:GetChildren()) do
            if ownerFields[key(child.Name)] and child:IsA("ValueBase") then
                inspectValue(child.Value)
            end
        end
    end
    inspect(base)
    for _, child in ipairs(base:GetChildren()) do
        if metadataNames[key(child.Name)] then inspect(child) end
    end
    return found, ours and not conflicting
end

local function pointCFrame(point)
    if not isMapObject(point) then return nil end
    if point:IsA("Attachment") then return point.WorldCFrame end
    if point:IsA("CFrameValue") then return point.Value end
    if point:IsA("BasePart") then
        -- Pontos em peças usam o topo como referência, sem enterrar o personagem.
        return point.CFrame + Vector3.new(0, point.Size.Y / 2 + CONFIG.Base.ReturnHeight, 0)
    end
end

local function findBasePoint(base)
    local best, bestRank, ambiguous = nil, math.huge, false
    for _, object in ipairs(base:GetDescendants()) do
        local rank = pointRanks[key(object.Name)]
        if object:IsA("SpawnLocation") then rank = rank or #CONFIG.Base.PointNames + 1 end
        if rank and (object:IsA("BasePart") or object:IsA("Attachment") or object:IsA("CFrameValue")) then
            -- Não usa spawns de pets, máquinas ou outros objetos profundamente aninhados.
            local depth, cursor = 0, object.Parent
            while cursor and cursor ~= base do depth = depth + 1; cursor = cursor.Parent end
            if cursor == base and depth <= CONFIG.Base.PointMaxDepth then
                local score = rank * 10 + depth
                if score < bestRank then
                    best, bestRank, ambiguous = object, score, false
                elseif score == bestRank then
                    ambiguous = true
                end
            end
        end
    end
    if ambiguous then return nil end
    return best
end

local function baseForSign(anchor)
    if not isMapObject(anchor) then return nil end
    local cursor = anchor
    while cursor and cursor ~= Workspace do
        if cursor:IsA("Model") or cursor:IsA("Folder") then
            local parent = cursor.Parent
            local name = key(cursor.Name)
            local namedBase = name:match("^base%d*$") or name:match("^plot%d*$")
                or name:match("^tycoon%d*$") or name:match("^lote%d*$")
            if parent and (baseContainerNames[key(parent.Name)] or namedBase) then return cursor end
        end
        cursor = cursor.Parent
    end
    -- Sem nomes conhecidos, exige um modelo que contenha a placa e um ponto de retorno.
    cursor = anchor.Parent
    while cursor and cursor ~= Workspace do
        local name = key(cursor.Name)
        if baseContainerNames[name] or name == "map" or name == "world"
            or name == "runtime" or name == "environment" then return nil end
        if cursor:IsA("Model") and findBasePoint(cursor) then return cursor end
        cursor = cursor.Parent
    end
end

local function signMatches(interface, allowDisplayName)
    local username = string.lower(player.Name)
    local displayName = string.lower(player.DisplayName)
    local function matchesText(text)
        text = text:gsub("<[^>]*>", "")
        for line in text:gmatch("[^\r\n]+") do
            line = string.lower(line:match("^%s*(.-)%s*$"))
            for _, name in ipairs({ username, allowDisplayName and displayName or username }) do
                if line == name or line == "@" .. name or line == "base de " .. name
                    or line == "base do " .. name or line == "base da " .. name
                    or line == name .. "'s base" or line == name .. "'s plot" then return true end
            end
        end
        return false
    end
    for _, object in ipairs(interface:GetDescendants()) do
        if (object:IsA("TextLabel") or object:IsA("TextButton")) and matchesText(object.Text) then
            return true
        end
        if object:IsA("ImageLabel") or object:IsA("ImageButton") then
            local uri = string.lower(object.Image)
            -- URLs opacas de imagens não permitem deduzir o dono pelo retrato.
            if uri:match("^rbxthumb://") and (uri:find("type=avatarheadshot", 1, true)
                or uri:find("type=avatar&", 1, true) or uri:find("type=avatarbust", 1, true)) then
                if tonumber(uri:match("[?&]id=(%d+)")) == player.UserId then return true end
            end
        end
    end
    return false
end

local function resolveBase()
    local candidates, signals = {}, {}
    local function add(base, signal)
        if not base or not (base:IsA("Model") or base:IsA("Folder")) or not isMapObject(base) then return end
        candidates[base] = true
        if signal and signals[base] ~= "vínculo do jogador" then signals[base] = signal end
    end
    for container in pairs(state.baseContainers) do
        if container:IsDescendantOf(Workspace) then
            for _, base in ipairs(container:GetChildren()) do
                add(base, matchesPlayer(base.Name) and "nome da base" or nil)
            end
        end
    end
    -- Referências explícitas do jogo para a base têm preferência sobre sinais visuais.
    for _, child in ipairs(player:GetChildren()) do
        if child:IsA("ObjectValue") and referenceFields[key(child.Name)] then
            add(child.Value, "vínculo do jogador")
        end
    end
    -- Se a propriedade oficial identifica uma única base, não inspeciona
    -- todos os BillboardGui/SurfaceGui do mapa a cada conferência.
    local definite = {}
    for base in pairs(candidates) do
        local hasOwner, ours = readOwner(base)
        if ours or (not hasOwner and signals[base] == "vínculo do jogador") then
            table.insert(definite, base)
        end
    end
    if #definite == 1 then
        local point = findBasePoint(definite[1])
        if point then return definite[1], point, "" end
        return nil, nil, "Sua base foi encontrada, mas o ponto de retorno não foi identificado."
    end

    local uniqueDisplay = true
    for _, other in ipairs(Players:GetPlayers()) do
        if other ~= player and string.lower(other.DisplayName) == string.lower(player.DisplayName) then
            uniqueDisplay = false
        end
    end
    for interface in pairs(state.linkedGuis) do
        local anchor = interface.Adornee or interface.Parent
        if signMatches(interface, uniqueDisplay) then
            add(baseForSign(anchor), "placa ou retrato")
        end
    end
    local owned, authoritative = {}, {}
    for base in pairs(candidates) do
        local hasOwner, ours = readOwner(base)
        if ours or (not hasOwner and signals[base]) then
            table.insert(owned, base)
            if ours or signals[base] == "vínculo do jogador" then table.insert(authoritative, base) end
        end
    end
    local matches = #authoritative > 0 and authoritative or owned
    if #matches == 0 then return nil, nil, "Procurando sua base pelo dono ou pela placa..." end
    if #matches > 1 then return nil, nil, "Mais de uma base corresponde ao jogador; destino não definido." end
    local base = matches[1]
    local point = findBasePoint(base)
    if not point then return nil, nil, "Sua base foi encontrada, mas o ponto de retorno não foi identificado." end
    return base, point, ""
end

local nextBaseScan = 0
local function refreshBase(force)
    if not state.running then return end
    local now = os.clock()
    local cachedBase, cachedPoint = state.baseObject, state.basePoint
    local destination
    if cachedBase and cachedPoint and isMapObject(cachedBase)
        and cachedPoint:IsDescendantOf(cachedBase) then
        local found, ours = readOwner(cachedBase)
        if not found or ours then destination = pointCFrame(cachedPoint) end
    end
    -- Usa o CFrame atualizado do ponto existente, sem procurar a base inteira.
    if destination and not force and now < nextBaseScan then
        state.base = destination
        return
    end
    if not destination and not force and now < nextBaseScan and not cachedBase then return end
    local base, point, message = resolveBase()
    local newDestination = point and pointCFrame(point)
    if not newDestination then base, point = nil, nil end
    nextBaseScan = now + (newDestination and CONFIG.BaseRescanSeconds or CONFIG.BaseRetrySeconds)
    local changed = base ~= state.baseObject or point ~= state.basePoint
        or message ~= state.baseMessage
    state.baseObject, state.basePoint, state.base = base, point, newDestination
    state.baseMessage = message
    if changed then onBaseChanged() end
end

local function hasBase()
    return state.running and state.base ~= nil and isMapObject(state.baseObject)
        and state.basePoint ~= nil and state.basePoint:IsDescendantOf(state.baseObject)
end

connect(player.CharacterRemoving, releasePrompt)
connect(player.CharacterAdded, function()
    -- A base é um objeto do mapa e continua sendo a mesma após renascer.
    task.defer(function() if state.running then refreshBase(true) end end)
end)

-- 4. INTERFACE: NAVEGAÇÃO, FILTROS E CATÁLOGO -----------------------------------
local playerGui = player:WaitForChild("PlayerGui")
local oldGui = playerGui:FindFirstChild("LeoHub") or playerGui:FindFirstChild("ExuFinder")
if oldGui then oldGui:Destroy() end
local oldFloating = playerGui:FindFirstChild("LeoHubFloating")
if oldFloating then oldFloating:Destroy() end

local gui = create("ScreenGui", {
    Name = "LeoHub", ResetOnSpawn = false,
    ZIndexBehavior = Enum.ZIndexBehavior.Sibling, DisplayOrder = 20,
}, playerGui)
local main = markSurface(create("Frame", {
    Name = "Panel", AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.fromScale(0.5, 0.5), Size = UDim2.fromOffset(CONFIG.Width, CONFIG.Height),
    BackgroundColor3 = THEME.Background, BackgroundTransparency = 0.07, BorderSizePixel = 0,
    ClipsDescendants = true, Active = true,
}, gui), "background")
round(main, 18)
outline(main)
local mainAccentBorder = create("UIStroke", {
    Color = THEME.Accent, Thickness = 1.8, Transparency = 0.18,
    ApplyStrokeMode = Enum.ApplyStrokeMode.Border,
}, main)
mainAccentBorder:SetAttribute("MainAccentBorder", true)
create("UIGradient", {
    Rotation = 25,
    Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, THEME.Highlight),
        ColorSequenceKeypoint.new(0.5, THEME.Accent),
        ColorSequenceKeypoint.new(1, THEME.Highlight),
    })
}, mainAccentBorder)
local uiScale = create("UIScale", { Scale = 1 }, main)

local function surface(parent, name, position, dimensions, color)
    local role = (color or THEME.Surface) == THEME.Rail and "rail" or "surface"
    local frame = markSurface(create("Frame", {
        Name = name, Position = position, Size = dimensions,
        BackgroundColor3 = color or THEME.Surface, BackgroundTransparency = 0.08, BorderSizePixel = 0,
    }, parent), role)
    round(frame, 12)
    outline(frame)
    return frame
end

local header = create("Frame", {
    Size = UDim2.new(1, 0, 0, 64), BackgroundTransparency = 1, Active = true,
}, main)
-- Marca LEO HUB: a imagem da logo aparece aqui ao configurar o asset Roblox.
-- Enquanto a imagem não estiver disponível, a marca LH permanece legível.
local brand = label(header, "LH", 24, UDim2.fromOffset(20, 10), UDim2.fromOffset(50, 43), THEME.Accent)
brand.Font = Enum.Font.FredokaOne
brand.TextXAlignment = Enum.TextXAlignment.Center
brand.TextYAlignment = Enum.TextYAlignment.Center
-- Esta é SOMENTE a nova logo de fantasma enviada pelo usuário.
-- Usa asset publicado no Studio ou imagem integrada em ambientes compatíveis.
-- A antiga logo foi eliminada das referências e dos arquivos de cache.
state.embeddedLogo = [==[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vLDwG/eE8/s3BCkeAAAACXBIWXMAAC4jAAAuIwF4pT92AAADZGlUWHRYTUw6Y29tLmFkb2JlLnhtcAAAAAAAPD94cGFja2V0IGJlZ2luPSfvu78nIGlkPSdXNU0wTXBDZWhpSHpyZVN6TlRjemtjOWQnPz4NCjx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iPjxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+PHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9InV1aWQ6ZmFmNWJkZDUtYmEzZC0xMWRhLWFkMzEtZDMzZDc1MTgyZjFiIiB4bWxuczpkYz0iaHR0cDovL3B1cmwub3JnL2RjL2VsZW1lbnRzLzEuMS8iPjxkYzpjcmVhdG9yPjxyZGY6U2VxIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+PHJkZjpsaT5Vc2VyPC9yZGY6bGk+PC9yZGY6U2VxPg0KCQkJPC9kYzpjcmVhdG9yPjwvcmRmOkRlc2NyaXB0aW9uPjxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSJ1dWlkOmZhZjViZGQ1LWJhM2QtMTFkYS1hZDMxLWQzM2Q3NTE4MmYxYiIgeG1sbnM6ZXhpZj0iaHR0cDovL25zLmFkb2JlLmNvbS9leGlmLzEuMC8iPjxleGlmOkRhdGVUaW1lT3JpZ2luYWw+MjAyNi0xMC0zMFQxMDo0MjoyMC44Mzk8L2V4aWY6RGF0ZVRpbWVPcmlnaW5hbD48L3JkZjpEZXNjcmlwdGlvbj48cmRmOkRlc2NyaXB0aW9uIHJkZjphYm91dD0idXVpZDpmYWY1YmRkNS1iYTNkLTExZGEtYWQzMS1kMzNkNzUxODJmMWIiIHhtbG5zOnhtcD0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLyI+PHhtcDpDcmVhdGVEYXRlPjIwMjYtMTAtMzBUMTA6NDI6MjAuODM5PC94bXA6Q3JlYXRlRGF0ZT48L3JkZjpEZXNjcmlwdGlvbj48L3JkZjpSREY+PC94OnhtcG1ldGE+DQo8P3hwYWNrZXQg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]==]
state.decodeEmbeddedLogo = function(data)
    local alphabet = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"
    local bytes = {}
    for i = 1, #data, 4 do
        local c1 = alphabet:find(data:sub(i, i), 1, true)
        local c2 = alphabet:find(data:sub(i + 1, i + 1), 1, true)
        local c3 = alphabet:find(data:sub(i + 2, i + 2), 1, true)
        local c4 = alphabet:find(data:sub(i + 3, i + 3), 1, true)
        if not c1 or not c2 then break end
        local packed = (c1 - 1) * 262144 + (c2 - 1) * 4096
            + ((c3 or 1) - 1) * 64 + ((c4 or 1) - 1)
        bytes[#bytes + 1] = string.char(math.floor(packed / 65536) % 256)
        if c3 then bytes[#bytes + 1] = string.char(math.floor(packed / 256) % 256) end
        if c4 then bytes[#bytes + 1] = string.char(packed % 256) end
    end
    return table.concat(bytes)
end
state.brandLogo = create("ImageLabel", {
    Name = "LeoHubGhostLogo", Position = UDim2.fromOffset(18, 5),
    Size = UDim2.fromOffset(56, 54),
    BackgroundTransparency = 1, BorderSizePixel = 0,
    ScaleType = Enum.ScaleType.Fit, Image = "", Visible = false,
}, header)
state.configureBrandLogo = function()
    -- Prioridade: ID da nova logo, recurso replicado com novo nome, imagem integrada.
    -- Nunca procura a antiga LEO_HUB_LOGO.png nem reutiliza o cache antigo.
    local uri = tostring(CONFIG.Branding.LogoAssetId or "")
    if uri:match("^%d+$") then uri = "rbxassetid://" .. uri end
    if uri == "" then
        local asset = ReplicatedStorage:FindFirstChild(CONFIG.Branding.ReplicatedLogoName, true)
        if asset then
            if asset:IsA("Decal") or asset:IsA("Texture") then
                uri = asset.Texture
            elseif asset:IsA("ImageLabel") or asset:IsA("ImageButton") then
                uri = asset.Image
            elseif asset:IsA("StringValue") then
                uri = tostring(asset.Value)
            end
        end
    end
    if uri == "" then
        local loadAsset = getcustomasset or getsynasset
        if type(loadAsset) == "function" then
            local ok, assetUri = pcall(function()
                if type(writefile) == "function" then
                    -- Nome exclusivo + gravação sempre atualizada: jamais recupera a imagem antiga.
                    local filename = "LEO_HUB_FANTASMA_NOVO_20261030.png"
                    writefile(filename, state.decodeEmbeddedLogo(state.embeddedLogo))
                    return loadAsset(filename)
                end
                if type(isfile) == "function" and isfile(CONFIG.Branding.LocalLogoFile) then
                    return loadAsset(CONFIG.Branding.LocalLogoFile)
                end
            end)
            if ok and type(assetUri) == "string" then uri = assetUri end
        end
    end
    if uri ~= "" then
        state.brandLogo.Image = uri
        state.brandLogo.Visible = true
        brand.Visible = not state.brandLogo.IsLoaded
    else
        state.brandLogo.Image = ""
        state.brandLogo.Visible = false
        brand.Visible = true
    end
end
connect(state.brandLogo:GetPropertyChangedSignal("IsLoaded"), function()
    if state.running and state.brandLogo.Visible then
        brand.Visible = not state.brandLogo.IsLoaded
    end
end)
state.configureBrandLogo()

-- Cinco ícones profissionais reunidos em um sprite PNG (5 x 96 px).
-- Ícones Font Awesome Free 6.7.2, Icons: CC BY 4.0 (Fonticons, Inc.)
state.embeddedIcons = [==[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]==]
state.configureIconSprite = function()
    state.iconSpriteUri = tostring(CONFIG.Branding.IconsAssetId or "")
    if state.iconSpriteUri:match("^%d+$") then
        state.iconSpriteUri = "rbxassetid://" .. state.iconSpriteUri
    end
    if state.iconSpriteUri == "" then
        local asset = ReplicatedStorage:FindFirstChild("LeoHubIcons", true)
        if asset then
            if asset:IsA("Decal") or asset:IsA("Texture") then
                state.iconSpriteUri = asset.Texture
            elseif asset:IsA("ImageLabel") or asset:IsA("ImageButton") then
                state.iconSpriteUri = asset.Image
            elseif asset:IsA("StringValue") then
                state.iconSpriteUri = tostring(asset.Value)
            end
        end
    end
    if state.iconSpriteUri == "" then
        local loadAsset = getcustomasset or getsynasset
        if type(loadAsset) == "function" then
            local ok, uri = pcall(function()
                if type(isfile) == "function" and isfile(CONFIG.Branding.LocalIconsFile) then
                    return loadAsset(CONFIG.Branding.LocalIconsFile)
                end
                if type(writefile) == "function" then
                    local filename = "LEO_HUB_ICONES_INTEGRADOS.png"
                    if type(isfile) ~= "function" or not isfile(filename) then
                        writefile(filename, state.decodeEmbeddedLogo(state.embeddedIcons))
                    end
                    return loadAsset(filename)
                end
            end)
            if ok and type(uri) == "string" then state.iconSpriteUri = uri end
        end
    end
end
state.configureIconSprite()
state.embeddedIcons = nil
state.configureIconSprite = nil
state.embeddedLogo = nil
state.decodeEmbeddedLogo = nil
local title = label(header, CONFIG.Branding.Title, 21, UDim2.fromOffset(87, 10), UDim2.fromOffset(260, 27))
title.Font = Enum.Font.FredokaOne
label(header, "SEUS PETS. DO SEU JEITO.", 9, UDim2.fromOffset(88, 39), UDim2.fromOffset(280, 15), THEME.Muted)
local minimize = button(header, "−", UDim2.new(1, -88, 0, 17), UDim2.fromOffset(30, 30))
local close = button(header, "×", UDim2.new(1, -48, 0, 17), UDim2.fromOffset(30, 30))
create("Frame", { Position = UDim2.new(0, 16, 1, -1), Size = UDim2.new(1, -32, 0, 1),
    BackgroundColor3 = THEME.Border, BackgroundTransparency = 0.4, BorderSizePixel = 0 }, header)

local body = create("Frame", { BackgroundTransparency = 1,
    Position = UDim2.fromOffset(0, 64), Size = UDim2.new(1, 0, 1, -64) }, main)
local rail = surface(body, "Navigation", UDim2.fromOffset(12, 12), UDim2.new(0, 62, 1, -24), THEME.Rail)
local homeNav = button(rail, "", UDim2.fromOffset(7, 12), UDim2.fromOffset(48, 54), true)
local settingsNav = button(rail, "", UDim2.fromOffset(7, 78), UDim2.fromOffset(48, 54))
local aboutNav = button(rail, "", UDim2.fromOffset(7, 144), UDim2.fromOffset(48, 54))
-- Ícones em sprite: casa, planta, controles, informações e marca Discord.
-- Fallback desenhado no próprio script, inspirado nos símbolos enviados pelo usuário.
state.iconPiece = function(parent, x, y, w, h, rotation, radius, color, transparency)
    local part = create("Frame", {
        Position = UDim2.fromOffset(x, y), Size = UDim2.fromOffset(w, h),
        BackgroundColor3 = color or Color3.new(1, 1, 1), BorderSizePixel = 0,
        BackgroundTransparency = transparency or 0,
        Rotation = rotation or 0, ZIndex = parent.ZIndex + 1,
    }, parent)
    part:SetAttribute("LeoIconPiece", true)
    if radius then round(part, radius) end
    return part
end
state.iconStrokeBox = function(parent, x, y, w, h, radius, thickness)
    local box = create("Frame", {
        Position = UDim2.fromOffset(x, y), Size = UDim2.fromOffset(w, h),
        BackgroundTransparency = 1, BorderSizePixel = 0,
        ZIndex = parent.ZIndex + 1,
    }, parent)
    box:SetAttribute("LeoIconPiece", true)
    round(box, radius or 6)
    create("UIStroke", { Color = Color3.new(1,1,1), Thickness = thickness or 2.4 }, box)
    return box
end
state.fallbackIcon = function(holder, index)
    local piece = state.iconPiece
    local strokeBox = state.iconStrokeBox
    if index == 1 then -- CASA: contorno limpo, com telhado e paredes (sem desenho antigo)
        -- Cada barra usa as extremidades exatas para formar uma casa de linha fina.
        local function edge(x1, y1, x2, y2, thickness)
            local dx, dy = x2 - x1, y2 - y1
            local length = math.sqrt(dx * dx + dy * dy)
            local midX, midY = (x1 + x2) * 0.5, (y1 + y2) * 0.5
            piece(holder, midX - length * 0.5, midY - thickness * 0.5,
                length, thickness, math.deg(math.atan2(dy, dx)), thickness)
        end
        edge(5, 17, 18, 6, 3)
        edge(18, 6, 31, 17, 3)
        edge(8, 16, 8, 29, 3)
        edge(28, 16, 28, 29, 3)
        edge(8, 29, 28, 29, 3)
    elseif index == 3 then -- três sliders verticais arredondados
        piece(holder, 7, 5, 4, 24, 0, 4)
        piece(holder, 16, 5, 4, 24, 0, 4)
        piece(holder, 25, 5, 4, 24, 0, 4)
        piece(holder, 5, 18, 8, 8, 0, 8)
        piece(holder, 14, 11, 8, 8, 0, 8)
        piece(holder, 23, 20, 8, 8, 0, 8)
    elseif index == 4 then -- info elegante
        local ring = create("Frame", { Position = UDim2.fromOffset(5, 5),
            Size = UDim2.fromOffset(27, 27), BackgroundTransparency = 1,
            BorderSizePixel = 0, ZIndex = holder.ZIndex + 1 }, holder)
        ring:SetAttribute("LeoIconPiece", true)
        round(ring, 20)
        create("UIStroke", { Color = Color3.new(1,1,1), Thickness = 2.6 }, ring)
        piece(holder, 17, 10, 4, 4, 0, 3)
        piece(holder, 17, 16, 4, 10, 0, 2)
    end
end
state.attachIcon = function(parent, index, pos, size)
    local fallback = create("Frame", {
        Name = "IconFallback", Position = pos, Size = UDim2.fromOffset(size, size),
        BackgroundTransparency = 1, BorderSizePixel = 0,
        ZIndex = parent.ZIndex + 2,
    }, parent)
    state.fallbackIcon(fallback, index)
end
state.attachIcon(homeNav, 1, UDim2.fromOffset(7, 3), 34)
state.attachIcon(settingsNav, 3, UDim2.fromOffset(7, 3), 34)
state.attachIcon(aboutNav, 4, UDim2.fromOffset(7, 3), 34)
for _, entry in ipairs({ { homeNav, "INÍCIO" },
    { settingsNav, "AJUSTES" }, { aboutNav, "SOBRE" } }) do
    local caption = label(entry[1], entry[2], 8, UDim2.fromOffset(0, 37), UDim2.new(1, 0, 0, 15))
    caption.TextXAlignment = Enum.TextXAlignment.Center
end
local railMark = label(rail, "LH", 10, UDim2.new(0, 0, 1, -28), UDim2.new(1, 0, 0, 18), THEME.Muted)
railMark.TextXAlignment = Enum.TextXAlignment.Center

local pages = {}
for _, name in ipairs({ "home", "settings", "about" }) do
    pages[name] = create("CanvasGroup", { Name = name, BackgroundTransparency = 1, GroupTransparency = 0,
        Position = UDim2.fromOffset(92, 12), Size = UDim2.new(1, -108, 1, -54),
        Visible = name == "home" }, body)
end
local home = pages.home
local left = create("Frame", { BackgroundTransparency = 1, Size = UDim2.new(0, 236, 1, 0) }, home)
local sectionTitle = label(left, "⚡ Coleta automática", 16, UDim2.fromOffset(0, 0), UDim2.new(1, 0, 0, 23))
sectionTitle.Font = Enum.Font.FredokaOne
label(left, "Filtre por pet, tamanho ou ambos.", 11, UDim2.fromOffset(0, 27), UDim2.new(1, 0, 0, 18), THEME.Muted)
local filters = surface(left, "Filters", UDim2.fromOffset(0, 58), UDim2.new(1, 0, 0, 254))
label(filters, "PETS", 9, UDim2.fromOffset(14, 14), UDim2.new(1, -28, 0, 14), THEME.Muted)
local petSelect = button(filters, "Selecionar pets (0)", UDim2.fromOffset(14, 34), UDim2.new(1, -28, 0, 34))
label(filters, "TAMANHOS", 9, UDim2.fromOffset(14, 80), UDim2.new(1, -28, 0, 14), THEME.Muted)
local sizeSelect = button(filters, "Selecionar tamanhos (0)", UDim2.fromOffset(14, 100), UDim2.new(1, -28, 0, 34))
-- O filtro de tamanhos já é autoexplicativo: a dica antiga foi removida.
local autoToggle = button(filters, "Ativar coleta automática", UDim2.fromOffset(14, 160), UDim2.new(1, -28, 0, 34), true)
-- Indicador de carregamento desenhado com UIStroke, sem emojis nem imagens externas.
-- O tween é criado uma vez e permanece pausado enquanto a coleta está desativada.
do
    autoToggle.TextSize = 13
    state.autoSpinner = create("Frame", {
        Name = "ColetaSpinner",
        AnchorPoint = Vector2.new(0, 0.5),
        Position = UDim2.new(0, 12, 0.5, 0),
        Size = UDim2.fromOffset(16, 16),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        Visible = false,
        Active = false,
        ZIndex = autoToggle.ZIndex + 2,
    }, autoToggle)
    round(state.autoSpinner, 100)
    state.autoSpinnerStroke = create("UIStroke", {
        Color = Color3.new(1, 1, 1),
        Thickness = 2.6,
        Transparency = 0.02,
    }, state.autoSpinner)
    state.autoSpinnerGradient = create("UIGradient", {
        Rotation = 0,
        Transparency = NumberSequence.new({
            NumberSequenceKeypoint.new(0, 0),
            NumberSequenceKeypoint.new(0.22, 0),
            NumberSequenceKeypoint.new(0.58, 0.55),
            NumberSequenceKeypoint.new(1, 1),
        }),
    }, state.autoSpinnerStroke)
    state.autoSpinnerTween = TweenService:Create(
        state.autoSpinnerGradient,
        TweenInfo.new(0.85, Enum.EasingStyle.Linear, Enum.EasingDirection.InOut, -1),
        { Rotation = 360 }
    )
    state.autoSpinnerRunning = false
end
-- O rodapé "Desativada • espécies conhecidas" foi removido da interface.
local autoStatus = label(filters, "", 9, UDim2.fromOffset(14, 202), UDim2.new(1, -28, 0, 26), THEME.Muted)
autoStatus.Visible = false
autoStatus.TextWrapped = true
autoStatus.TextTruncate = Enum.TextTruncate.None

local basePanel = surface(left, "Base", UDim2.fromOffset(0, 326), UDim2.new(1, 0, 0, 108), THEME.Rail)
label(basePanel, "🏠 SUA BASE", 9, UDim2.fromOffset(14, 12), UDim2.new(1, -28, 0, 16), THEME.Muted)
local baseStatus = label(basePanel, "Procurando no mapa...", 11, UDim2.fromOffset(14, 32), UDim2.new(1, -28, 0, 18))
local goBack = button(basePanel, "Voltar à base", UDim2.fromOffset(14, 62), UDim2.new(1, -28, 0, 32), true)

local right = create("Frame", { BackgroundTransparency = 1,
    Position = UDim2.fromOffset(252, 0), Size = UDim2.new(1, -252, 1, 0) }, home)
local statLabels = {}
for index, caption in ipairs({ "JOGADORES", "PING", "PETS NA RODADA" }) do
    local tile = surface(right, "Stat" .. index, UDim2.new((index - 1) / 3, (index - 1) * 4, 0, 0),
        UDim2.new(1 / 3, -8, 0, 72))
    label(tile, caption, 9, UDim2.fromOffset(14, 12), UDim2.new(1, -24, 0, 14), THEME.Muted)
    statLabels[index] = label(tile, "—", 22, UDim2.fromOffset(14, 31), UDim2.new(1, -24, 0, 27), THEME.Text)
    statLabels[index].Font = Enum.Font.FredokaOne
end
local catalog = surface(right, "Catalog", UDim2.fromOffset(0, 86), UDim2.new(1, 0, 1, -86))
local catalogTitle = label(catalog, "🐾 Pets spawnados", 15, UDim2.fromOffset(16, 12), UDim2.new(1, -160, 0, 23))
local sort = button(catalog, "Raridade", UDim2.new(1, -144, 0, 10), UDim2.fromOffset(128, 27))
catalogTitle.Font = Enum.Font.FredokaOne
local search = create("TextBox", {
    Position = UDim2.fromOffset(16, 44), Size = UDim2.new(1, -32, 0, 36),
    BackgroundColor3 = THEME.Input, BackgroundTransparency = 0.08, BorderSizePixel = 0, Text = "",
    PlaceholderText = "Pesquisar nome, raridade ou tamanho...",
    PlaceholderColor3 = Color3.new(1, 1, 1), TextColor3 = Color3.new(1, 1, 1),
    TextStrokeTransparency = 0.8, TextSize = 14, Font = Enum.Font.FredokaOne, ClearTextOnFocus = false,
    TextXAlignment = Enum.TextXAlignment.Left,
}, catalog)
round(search, 9)
create("UIPadding", { PaddingLeft = UDim.new(0, 12), PaddingRight = UDim.new(0, 12) }, search)
local info = label(catalog, "Procurando pets...", 10, UDim2.fromOffset(16, 86), UDim2.new(1, -32, 0, 18), THEME.Muted)
local scroll = create("ScrollingFrame", {
    Position = UDim2.fromOffset(12, 112), Size = UDim2.new(1, -24, 1, -124),
    BackgroundTransparency = 1, BorderSizePixel = 0,
    ScrollBarThickness = 3, ScrollBarImageColor3 = THEME.Accent,
    AutomaticCanvasSize = Enum.AutomaticSize.Y, CanvasSize = UDim2.new(),
    ScrollingDirection = Enum.ScrollingDirection.Y,
}, catalog)
create("UIListLayout", { Padding = UDim.new(0, 8), SortOrder = Enum.SortOrder.LayoutOrder }, scroll)
create("UIPadding", { PaddingBottom = UDim.new(0, 4) }, scroll)
local empty = label(catalog, "Aguardando pets...", 13, UDim2.fromOffset(24, 130), UDim2.new(1, -48, 0, 90), THEME.Muted)
empty.TextWrapped = true
empty.TextTruncate = Enum.TextTruncate.None
empty.TextXAlignment = Enum.TextXAlignment.Center

local function pageIntro(page, titleText, subtitle)
    local heading = label(page, titleText, 22, UDim2.fromOffset(0, 4), UDim2.new(1, 0, 0, 30))
    heading.Font = Enum.Font.FredokaOne
    label(page, subtitle, 12, UDim2.fromOffset(0, 42), UDim2.new(1, 0, 0, 24), THEME.Muted)
end
pageIntro(pages.settings, "Ajustes do script", "Escolha o tema e ajuste recursos do painel.")
local settingsBox = surface(pages.settings, "Preferences", UDim2.fromOffset(0, 84), UDim2.new(1, 0, 0, 178))
label(settingsBox, "Tema do script", 14, UDim2.fromOffset(20, 20), UDim2.new(1, -180, 0, 22))
label(settingsBox, "Escolha entre roxo, vermelho, amarelo e azul.", 11, UDim2.fromOffset(20, 46), UDim2.new(1, -220, 0, 32), THEME.Muted)
local themeSelect = button(settingsBox, "Tema: Azul", UDim2.new(1, -168, 0, 27), UDim2.fromOffset(148, 34))

create("Frame", { Position = UDim2.fromOffset(20, 88), Size = UDim2.new(1, -40, 0, 1), BackgroundColor3 = THEME.Border, BackgroundTransparency = 0.55, BorderSizePixel = 0 }, settingsBox)
label(settingsBox, "Anti AFK", 14, UDim2.fromOffset(20, 102), UDim2.new(1, -180, 0, 22))
local antiAfkToggle = button(settingsBox, "Ativado", UDim2.new(1, -148, 0, 111), UDim2.fromOffset(128, 34), true)

-- SOBRE: visual inspirado no perfil fornecido. A aparência 3D é carregada do Roblox.
local CREATOR_USERNAME = "armazenatudo"
pageIntro(pages.about, "Sobre o LEO HUB", "")
local aboutBox = surface(pages.about, "CreatorProfile", UDim2.fromOffset(0, 54),
    UDim2.new(1, 0, 1, -66), THEME.Rail)
aboutBox.ClipsDescendants = true

local aboutPortrait = surface(aboutBox, "CreatorAvatarPanel", UDim2.fromOffset(12, 12),
    UDim2.new(0, 282, 1, -24), THEME.Background)
aboutPortrait.ClipsDescendants = true
aboutPortrait:SetAttribute("SurfaceRole", "background")
local portraitAccent = create("UIStroke", {
    Color = THEME.Accent, Thickness = 1.6, Transparency = 0.36,
    ApplyStrokeMode = Enum.ApplyStrokeMode.Border,
}, aboutPortrait)
portraitAccent:SetAttribute("MainAccentBorder", true)
local portraitGlow = create("Frame", {
    BackgroundColor3 = THEME.Accent, BackgroundTransparency = 0.88,
    Position = UDim2.new(0.5, -74, 0, -79), Size = UDim2.fromOffset(148, 148),
    BorderSizePixel = 0, ZIndex = 1,
}, aboutPortrait)
round(portraitGlow, 74)
portraitGlow:SetAttribute("AboutAccentSurface", true)
-- Imagem oficial de corpo inteiro fica visível até o 3D carregar e serve de fallback.
-- Ela também previne um quadro vazio se o jogo não renderizar o WorldModel.
local avatarThumbnail = create("ImageLabel", {
    Name = "AvatarFallback", BackgroundTransparency = 1,
    Position = UDim2.fromOffset(10, 8), Size = UDim2.new(1, -20, 1, -93),
    Image = "", ScaleType = Enum.ScaleType.Fit,
    ImageTransparency = 0, ZIndex = 2,
}, aboutPortrait)
local avatarViewport = create("ViewportFrame", {
    Name = "Avatar3D", BackgroundTransparency = 1,
    Position = UDim2.fromOffset(10, 8), Size = UDim2.new(1, -20, 1, -93),
    Ambient = Color3.fromRGB(255, 255, 255),
    LightColor = Color3.fromRGB(255, 255, 255),
    LightDirection = Vector3.new(-0.5, -0.8, -1),
    ZIndex = 3, Visible = false,
}, aboutPortrait)
local avatarWorld = create("WorldModel", { Name = "AvatarWorld" }, avatarViewport)
local avatarCamera = create("Camera", { FieldOfView = 31 }, avatarViewport)
avatarViewport.CurrentCamera = avatarCamera
local avatarStatus = label(aboutPortrait, "Carregando avatar 3D...", 12,
    UDim2.fromOffset(18, 104), UDim2.new(1, -36, 0, 64))
avatarStatus.ZIndex = 5
avatarStatus.TextWrapped = true
avatarStatus.TextTruncate = Enum.TextTruncate.None
avatarStatus.TextXAlignment = Enum.TextXAlignment.Center
avatarStatus.TextYAlignment = Enum.TextYAlignment.Center

local portraitFooter = create("Frame", {
    Position = UDim2.new(0, 0, 1, -80), Size = UDim2.new(1, 0, 0, 80),
    BackgroundColor3 = THEME.Surface, BackgroundTransparency = 0.07,
    BorderSizePixel = 0, ZIndex = 3,
}, aboutPortrait)
portraitFooter:SetAttribute("SurfaceRole", "surface")
local creatorCaption = label(portraitFooter, "@" .. CREATOR_USERNAME, 19,
    UDim2.fromOffset(12, 7), UDim2.new(1, -24, 0, 32))
creatorCaption.TextXAlignment = Enum.TextXAlignment.Center
creatorCaption.ZIndex = 4
local creatorSub = label(portraitFooter, "CRIADOR DO LEO HUB", 10,
    UDim2.fromOffset(12, 41), UDim2.new(1, -24, 0, 23))
creatorSub.TextXAlignment = Enum.TextXAlignment.Center
creatorSub.ZIndex = 4

local creatorInfo = surface(aboutBox, "CreatorInfo", UDim2.fromOffset(306, 12),
    UDim2.new(1, -318, 1, -24), THEME.Surface)
local infoAccent = create("Frame", {
    Position = UDim2.fromOffset(21, 25), Size = UDim2.fromOffset(5, 40),
    BackgroundColor3 = THEME.Accent, BorderSizePixel = 0,
}, creatorInfo)
round(infoAccent, 3)
infoAccent:SetAttribute("AboutAccentSurface", true)
local infoTag = label(creatorInfo, "DESENVOLVEDOR", 11,
    UDim2.fromOffset(40, 20), UDim2.new(1, -200, 0, 19))
local infoName = label(creatorInfo, CREATOR_USERNAME, 22,
    UDim2.fromOffset(38, 42), UDim2.new(1, -58, 0, 36))

local contact = surface(creatorInfo, "Contact", UDim2.fromOffset(20, 94),
    UDim2.new(1, -40, 0, 73), THEME.Rail)
-- Cartão de contato: apenas texto, sem ícone ou imagem do Discord.
local discordTitle = label(contact, "DISCORD", 11,
    UDim2.fromOffset(20, 12), UDim2.new(1, -40, 0, 20))
local discordNick = label(contact, "o.priorizado", 16,
    UDim2.fromOffset(20, 32), UDim2.new(1, -40, 0, 30))

local avatarRefresh = button(creatorInfo, "Atualizar avatar", UDim2.new(1, -161, 0, 17),
    UDim2.fromOffset(143, 34))

local footer = label(body, "Procurando sua base...", 10, UDim2.new(0, 96, 1, -32), UDim2.new(1, -114, 0, 25), THEME.Muted)
footer.TextWrapped = true
footer.TextTruncate = Enum.TextTruncate.None
local function notify(message, color)
    if not state.running then return end
    footer.Text = message
    footer.TextColor3 = Color3.new(1, 1, 1)
end

-- Menus de seleção são expansíveis e aparecem imediatamente abaixo de cada botão.
-- Carregamento de miniatura é delegado ao sistema visual, definido mais adiante.
local renderSelectionThumbnail = function(holder, option)
    local mark = label(holder, "?", 18, UDim2.new(), UDim2.fromScale(1, 1), THEME.Muted)
    mark.TextXAlignment = Enum.TextXAlignment.Center
end
local pickerOverlay, dropdownAnchor, dropdownArrow
local dropdownArrows = {}
local function addDropdownArrow(item)
    item.TextTransparency = 1 -- Botão continua interativo; texto fica no caption abaixo.
    local caption = label(item, item.Text, 12,
        UDim2.fromOffset(12, 0), UDim2.new(1, -43, 1, 0), THEME.Text)
    caption.Name = "DropdownCaption"
    caption.TextXAlignment = Enum.TextXAlignment.Left
    caption.TextYAlignment = Enum.TextYAlignment.Center
    caption.TextTruncate = Enum.TextTruncate.AtEnd
    caption.ZIndex = item.ZIndex + 1
    connect(item:GetPropertyChangedSignal("Text"), function()
        if caption.Parent then caption.Text = item.Text end
    end)

    -- Posicionamento relativo à extremidade DIREITA do botão, sem UIPadding.
    local arrow = label(item, "▼", 14,
        UDim2.new(1, -24, 0.5, -11), UDim2.fromOffset(20, 22), THEME.Text)
    arrow.Name = "DropdownArrow"
    arrow.TextXAlignment = Enum.TextXAlignment.Center
    arrow.TextYAlignment = Enum.TextYAlignment.Center
    arrow.Font = Enum.Font.FredokaOne
    arrow.ZIndex = item.ZIndex + 2
    dropdownArrows[item] = arrow
end
addDropdownArrow(petSelect)
addDropdownArrow(sizeSelect)
addDropdownArrow(sort)
addDropdownArrow(themeSelect)

local function closePicker(immediate)
    local overlay, arrow = pickerOverlay, dropdownArrow
    pickerOverlay, dropdownAnchor, dropdownArrow = nil, nil, nil
    if arrow and arrow.Parent then tween(arrow, QUICK, { Rotation = 0 }) end
    if not overlay or not overlay.Parent then return end
    local menu = overlay:FindFirstChild("DropdownPanel")
    if immediate or not menu then overlay:Destroy(); return end
    tween(menu, QUICK, { Size = UDim2.fromOffset(menu.Size.X.Offset, 0), GroupTransparency = 1 })
    task.delay(0.17, function() if overlay.Parent then overlay:Destroy() end end)
end

local function dropdown(anchor, requestedWidth, requestedHeight)
    -- Clicar no seletor já aberto apenas o fecha.
    if pickerOverlay and dropdownAnchor == anchor then closePicker(); return nil end
    closePicker(true)
    local scale = uiScale.Scale
    local x = (anchor.AbsolutePosition.X - main.AbsolutePosition.X) / scale
    local y = (anchor.AbsolutePosition.Y - main.AbsolutePosition.Y + anchor.AbsoluteSize.Y) / scale + 6
    local width = math.min(requestedWidth, CONFIG.Width - 24)
    if anchor == sort then x = x + anchor.AbsoluteSize.X / scale - width end
    x = math.clamp(x, 12, CONFIG.Width - width - 12)
    local freeHeight = CONFIG.Height - y - 12
    if freeHeight < 125 then y = math.max(12, y - anchor.AbsoluteSize.Y / scale - requestedHeight - 12) end
    local height = math.min(requestedHeight, CONFIG.Height - y - 12)

    local overlay = create("TextButton", {
        Name = "DropdownOverlay", Text = "", AutoButtonColor = false,
        BackgroundTransparency = 1, BorderSizePixel = 0,
        Size = UDim2.fromScale(1, 1), ZIndex = 50,
    }, main)
    pickerOverlay, dropdownAnchor = overlay, anchor
    dropdownArrow = dropdownArrows[anchor]
    if dropdownArrow and dropdownArrow.Parent then tween(dropdownArrow, QUICK, { Rotation = 180 }) end
    overlay.Activated:Connect(function() if pickerOverlay == overlay then closePicker() end end)

    local menu = create("CanvasGroup", {
        Name = "DropdownPanel", Position = UDim2.fromOffset(x, y),
        Size = UDim2.fromOffset(width, 0), GroupTransparency = 0.10,
        BackgroundColor3 = THEME.Surface, BorderSizePixel = 0,
        ClipsDescendants = true, Active = true, ZIndex = 51,
    }, overlay)
    round(menu, 12)
    create("UIStroke", { Color = THEME.Accent, Thickness = 1.1, Transparency = 0.46 }, menu)
    tween(menu, MENU_TWEEN, { Size = UDim2.fromOffset(width, height), GroupTransparency = 0 })
    return menu, width, height
end

local function openPicker(anchor, options, selected, onSelect)
    local menu, width, height = dropdown(anchor, 192, #options * 43 + 16)
    if not menu then return end
    for index, option in ipairs(options) do
        local row = button(menu, "", UDim2.fromOffset(8, 8 + (index - 1) * 43),
            UDim2.fromOffset(width - 16, 38), option.value == selected)
        row.ZIndex = 53
        local name = label(row, option.text, 12, UDim2.fromOffset(12, 8),
            UDim2.new(1, -35, 0, 22), option.value == selected and THEME.Background or THEME.Text)
        name.ZIndex = 54
        if option.value == selected then
            local check = label(row, "✓", 13, UDim2.new(1, -27, 0, 8),
                UDim2.fromOffset(19, 20), THEME.Background)
            check.ZIndex = 54
        end
        row.Activated:Connect(function()
            closePicker()
            onSelect(option.value, option.text)
        end)
    end
end

local function openMultiPicker(anchor, titleText, options, selected, onChanged, showPetThumbs)
    local desired = math.min(338, 153 + #options * 43)
    local menu, width, height = dropdown(anchor, showPetThumbs and 368 or 278, desired)
    if not menu then return end
    local heading = label(menu, titleText, 14, UDim2.fromOffset(13, 11),
        UDim2.new(1, -26, 0, 22), THEME.Text)
    heading.ZIndex, heading.Font = 53, Enum.Font.FredokaOne
    local hint = label(menu, "Marque uma ou mais opções", 10, UDim2.fromOffset(13, 35),
        UDim2.new(1, -26, 0, 16), THEME.Muted)
    hint.ZIndex = 53
    local input = create("TextBox", {
        Position = UDim2.fromOffset(11, 59), Size = UDim2.new(1, -22, 0, 32),
        BackgroundColor3 = THEME.Input, BackgroundTransparency = 0.08, BorderSizePixel = 0,
        Text = "", PlaceholderText = "Pesquisar...", PlaceholderColor3 = Color3.new(1, 1, 1),
        TextColor3 = Color3.new(1, 1, 1), TextStrokeTransparency = 0.8,
        TextSize = 14, Font = Enum.Font.FredokaOne,
        ClearTextOnFocus = false, TextXAlignment = Enum.TextXAlignment.Left, ZIndex = 53,
    }, menu)
    round(input, 8)
    create("UIPadding", { PaddingLeft = UDim.new(0, 11) }, input)
    local listHeight = math.max(40, height - 139)
    local list = create("ScrollingFrame", {
        BackgroundTransparency = 1, BorderSizePixel = 0,
        Position = UDim2.fromOffset(10, 99), Size = UDim2.new(1, -20, 0, listHeight),
        AutomaticCanvasSize = Enum.AutomaticSize.Y, CanvasSize = UDim2.new(),
        ScrollingDirection = Enum.ScrollingDirection.Y, ScrollBarThickness = 3,
        ScrollBarImageColor3 = THEME.Accent, ZIndex = 53,
    }, menu)
    create("UIListLayout", { Padding = UDim.new(0, 5), SortOrder = Enum.SortOrder.LayoutOrder }, list)
    create("UIPadding", { PaddingBottom = UDim.new(0, 5) }, list)
    local entries = {}
    local function paint()
        local query = normalize(input.Text)
        local filtered = 0
        for _, entry in ipairs(entries) do
            local active = selected[entry.option.value] == true
            entry.row.BackgroundColor3 = active and Color3.fromRGB(32, 98, 93) or THEME.Input
            entry.check.BackgroundColor3 = active and THEME.Accent or THEME.Rail
            entry.checkText.Text = active and "✓" or ""
            entry.checkText.TextColor3 = Color3.new(1, 1, 1)
            local visible = normalize(entry.option.searchText or entry.option.text):find(query, 1, true) ~= nil
            entry.row.Visible = visible
            if visible then filtered = filtered + 1 end
        end
        hint.Text = string.format("%d marcados  •  %d opções", (function()
            local n = 0
            for _ in pairs(selected) do n = n + 1 end
            return n
        end)(), filtered)
    end
    local function updateThumbnails()
        if not showPetThumbs or not menu.Parent then return end
        local top = list.AbsolutePosition.Y
        local bottom = top + list.AbsoluteSize.Y
        for _, entry in ipairs(entries) do
            if entry.thumb and not entry.thumbReady and entry.row.Visible then
                local yy = entry.row.AbsolutePosition.Y
                if yy + entry.row.AbsoluteSize.Y > top and yy < bottom then
                    entry.thumbReady = true
                    task.defer(function()
                        if state.running and entry.thumb.Parent and menu.Parent then
                            renderSelectionThumbnail(entry.thumb, entry.option)
                        end
                    end)
                end
            end
        end
    end
    for index, option in ipairs(options) do
        local row = button(list, "", UDim2.new(), UDim2.new(1, -4, 0, 39))
        row.LayoutOrder, row.ZIndex = index, 54
        local nameX = showPetThumbs and 50 or 13
        local name = label(row, option.text, 12, UDim2.fromOffset(nameX, 8),
            UDim2.new(1, -nameX - 39, 0, 23), THEME.Text)
        name.ZIndex = 55
        local thumb
        if showPetThumbs then
            thumb = create("Frame", {
                Position = UDim2.fromOffset(5, 3), Size = UDim2.fromOffset(33, 33),
                BackgroundColor3 = THEME.Rail, BorderSizePixel = 0,
                ClipsDescendants = true, ZIndex = 55,
            }, row)
            round(thumb, 7)
        end
        local check = create("Frame", {
            Position = UDim2.new(1, -33, 0, 9), Size = UDim2.fromOffset(20, 20),
            BackgroundColor3 = THEME.Rail, BorderSizePixel = 0, ZIndex = 55,
        }, row)
        round(check, 6)
        local checkText = label(check, "", 14, UDim2.new(), UDim2.fromScale(1, 1))
        checkText.ZIndex = 56
        checkText.TextXAlignment = Enum.TextXAlignment.Center
        local entry = { row = row, option = option, check = check,
            checkText = checkText, thumb = thumb, thumbReady = false }
        table.insert(entries, entry)
        row.Activated:Connect(function()
            selected[option.value] = not selected[option.value] or nil
            onChanged()
            paint()
        end)
    end
    if #options == 0 then
        hint.Text = "Sem catálogo replicado. Confira Collection.PetCatalog."
    end
    local footerY = height - 34
    local clear = button(menu, "Limpar", UDim2.fromOffset(11, footerY), UDim2.fromOffset(90, 27))
    local done = button(menu, "Concluir", UDim2.fromOffset(width - 105, footerY),
        UDim2.fromOffset(94, 27), true)
    clear.ZIndex, done.ZIndex = 53, 53
    clear.Activated:Connect(function() table.clear(selected); onChanged(); paint() end)
    done.Activated:Connect(function() closePicker() end)
    input:GetPropertyChangedSignal("Text"):Connect(function()
        paint()
        list.CanvasPosition = Vector2.zero
        task.delay(0.06, updateThumbnails)
    end)
    list:GetPropertyChangedSignal("CanvasPosition"):Connect(updateThumbnails)
    list:GetPropertyChangedSignal("AbsoluteCanvasSize"):Connect(updateThumbnails)
    paint()
    task.delay(0.27, updateThumbnails)
end


onBaseChanged = function()
    if not state.running then return end
    local ready = hasBase()
    baseStatus.Text = ready and "Base localizada" or "Procurando no mapa..."
    baseStatus.TextColor3 = Color3.new(1, 1, 1)
    goBack.Active = ready
    goBack.AutoButtonColor = ready
    goBack.BackgroundTransparency = ready and 0 or 0.5
    goBack.Text = ready and "Voltar à base" or "Aguardando base"
    notify(state.baseMessage, ready and THEME.Success or THEME.Muted)
end
onBaseChanged()
local function requireBase()
    refreshBase(true)
    if hasBase() then return true end
    notify(state.baseMessage, THEME.Warning)
    return false
end


-- 5. DADOS DOS PETS, CLASSIFICAÇÃO E ÍCONES ------------------------------------------------

local SIZES = {
    { "colossal", "Colossal" }, { "gigante", "Gigante" },
    { "enorme", "Enorme" }, { "grande", "Grande" },
    { "medio", "Médio" }, { "pequeno", "Pequeno" }, { "mini", "Mini" },
}
-- Somente a aparência das opções; os valores internos continuam iguais aos do jogo.
local SIZE_EMOJIS = {
    Mini = "🐜", Pequeno = "🐭", ["Médio"] = "🐕", Grande = "🐻",
    Enorme = "🐘", Gigante = "🐉", Colossal = "🌎",
}

local function identifySize(value)
    local text = normalize(value):gsub("_", " ")
    for _, entry in ipairs(SIZES) do
        if text:match("%f[%a]" .. entry[1] .. "%f[%A]") then return entry[2] end
    end
end

local function sizeField(name)
    local key = normalize(name)
    return key:find("size", 1, true) or key:find("tamanho", 1, true)
        or key:find("porte", 1, true)
end

local function readSize(instance)
    for key, value in pairs(instance:GetAttributes()) do
        if sizeField(key) then
            local result = identifySize(value)
            if result then return result end
        end
    end
    if instance:IsA("TextLabel") or instance:IsA("TextButton") then
        return identifySize(instance.Text)
    end
    if instance:IsA("StringValue") and sizeField(instance.Name) then
        return identifySize(instance.Value)
    end
end

local function linkedInterfaces(object)
    local result = {}
    for interface in pairs(state.linkedGuis) do
        local adornee = interface.Adornee
        if adornee and (adornee == object or adornee:IsDescendantOf(object)) then
            table.insert(result, interface)
        end
    end
    return result
end

local function getSize(object, descendants, interfaces)
    local result = readSize(object) or identifySize(object.Name)
    if result then return result end
    for _, child in ipairs(descendants) do
        result = readSize(child)
        if result then return result end
    end
    for _, interface in ipairs(interfaces) do
        for _, child in ipairs(interface:GetDescendants()) do
            result = readSize(child)
            if result then return result end
        end
    end
    return "Não identificado"
end

-- Classificação pelos dados de raridade do próprio pet.
local function plainText(value)
    return tostring(value):gsub("<[^>]*>", ""):match("^%s*(.-)%s*$")
end
local rarityLookup = {}
for rank, entry in ipairs(CONFIG.Ranking.Rarities) do
    for _, alias in ipairs(entry.aliases) do rarityLookup[key(alias)] = { name = entry.name, rank = rank } end
end
local function readRarity(value, explicit)
    local text = plainText(value)
    local name = text:match("^[Rr]aridade%s*:%s*(.-)$") or text:match("^[Rr]arity%s*:%s*(.-)$") or text
    local known = rarityLookup[key(name)]
    if known then return known.name, known.rank end
    if explicit and name ~= "" then return name, nil end
end
local rarityFields = {}
for priority, field in ipairs(CONFIG.Ranking.Fields.rarity) do rarityFields[key(field)] = priority end
local function getPetStats(object, descendants, interfaces)
    local best, seen = nil, {}
    local function offer(value, score, explicit)
        local name, rank = readRarity(value, explicit)
        if not name then return end
        if not best or score < best.score then best = { name = name, rank = rank, score = score }
        elseif score == best.score and best.name ~= name then best.conflict = true end
    end
    local function inspect(instance, score)
        if seen[instance] then return end
        seen[instance] = true
        for name, value in pairs(instance:GetAttributes()) do
            local priority = rarityFields[key(name)]
            if priority then offer(value, score + priority, true) end
        end
        local priority = rarityFields[key(instance.Name)]
        if instance:IsA("StringValue") or instance:IsA("NumberValue") or instance:IsA("IntValue") then
            if priority then offer(instance.Value, score + priority, true) end
        elseif instance:IsA("TextLabel") or instance:IsA("TextButton") then
            offer(instance.Text, score + (priority or 500), priority ~= nil)
        end
    end
    inspect(object, 0)
    for _, child in ipairs(descendants) do inspect(child, 1000) end
    for _, interface in ipairs(interfaces) do
        for _, child in ipairs(interface:GetDescendants()) do inspect(child, 1000) end
    end
    if best and not best.conflict then return { rarity = best.name, rarityRank = best.rank } end
    return { rarity = "Não identificada" }
end
local SIZE_RANKS = { Mini = 1, Pequeno = 2, ["Médio"] = 3, Grande = 4, Enorme = 5, Gigante = 6, Colossal = 7 }
local SORT_MODES = {
    { id = "rarityRank", text = "Raridade", description = "raridade" },
    { id = "sizeRank", text = "Tamanho", description = "maiores tamanhos" },
}
local function sortRows(rows)
    local mode = SORT_MODES[state.sortMode]
    table.sort(rows, function(a, b)
        local av, bv = a[mode.id], b[mode.id]
        if av ~= bv then
            if av == nil then return false end
            if bv == nil then return true end
            return av > bv
        end
        for _, secondary in ipairs({ "rarityRank", "sizeRank" }) do
            av, bv = a[secondary], b[secondary]
            if av ~= bv then
                if av == nil then return false end
                if bv == nil then return true end
                return av > bv
            end
        end
        if a.name ~= b.name then return a.name < b.name end
        return a.card.sequence < b.card.sequence
    end)
    return mode.description
end


local IMAGE_FIELDS = {
    "PetIcon", "PetImage", "PetThumbnail", "Thumbnail", "ThumbnailId", "Icon", "IconId", "IconImage",
    "Image", "ImageId", "Picture", "Portrait", "Foto", "Imagem", "Icone", "Ícone",
}
local imageFieldKeys = {}
for _, key in ipairs(IMAGE_FIELDS) do imageFieldKeys[normalize(key)] = true end

local function assetUri(value)
    if typeof(value) ~= "string" and typeof(value) ~= "number" then return nil end
    local text = tostring(value):match("^%s*(.-)%s*$")
    if text:match("^%d+$") and tonumber(text) > 0 then return "rbxassetid://" .. text end
    if text:match("^rbxassetid://%d+$") and tonumber(text:match("%d+")) > 0 then return text end
    if text:match("^rbxthumb://") or text:match("^rbxasset://") then return text end
    local id = text:match("^https?://www%.roblox%.com/asset/%?id=(%d+)")
        or text:match("^https?://www%.roblox%.com/asset%?id=(%d+)")
    if id and tonumber(id) > 0 then return "rbxassetid://" .. id end
end

local function imageDescriptor(value)
    local uri = assetUri(value)
    if uri then return { uri = uri, offset = Vector2.zero, size = Vector2.zero } end
end

local function readImage(instance, allowGenericImage)
    for _, key in ipairs(IMAGE_FIELDS) do
        local image = imageDescriptor(instance:GetAttribute(key))
        if image then return image end
    end
    -- Também aceita nomes de atributos em minúsculas.
    for key, value in pairs(instance:GetAttributes()) do
        if imageFieldKeys[normalize(key)] then
            local image = imageDescriptor(value)
            if image then return image end
        end
    end
    local namedAsImage = imageFieldKeys[normalize(instance.Name)]
    if namedAsImage and (instance:IsA("StringValue") or instance:IsA("NumberValue")
        or instance:IsA("IntValue")) then
        return imageDescriptor(instance.Value)
    end
    if (allowGenericImage or namedAsImage) and
        (instance:IsA("ImageLabel") or instance:IsA("ImageButton")) then
        local image = imageDescriptor(instance.Image)
        if image then
            -- Preserva o recorte quando o jogo usa uma folha de sprites.
            image.offset = instance.ImageRectOffset
            image.size = instance.ImageRectSize
            return image
        end
    end
end

local function getImage(object, descendants, interfaces)
    local manual = CONFIG.PetIcons[object.Name] or CONFIG.PetIcons[cleanName(object.Name)]
    local found = imageDescriptor(manual) or readImage(object, true)
    if found then return found end
    for _, child in ipairs(descendants) do
        found = readImage(child, false)
        if found then return found end
    end
    for _, interface in ipairs(interfaces) do
        for _, child in ipairs(interface:GetDescendants()) do
            found = readImage(child, false)
            if found then return found end
        end
    end
    local matches = state.resources[nameKey(object.Name)]
    if matches then
        for resource in pairs(matches) do
            found = readImage(resource, true)
            if found then return found end
            for _, child in ipairs(resource:GetDescendants()) do
                found = readImage(child, false)
                if found then return found end
            end
        end
    end
    -- Não usa TextureID/Decal como retrato: normalmente são apenas pedaços da pele.
end

-- 6. MINIATURAS: IMAGEM DO JOGO OU MODELO 3D ------------------------------------

local function clearPreview(card)
    card.previewVersion = card.previewVersion + 1
    -- Preserva o UICorner do retrato ao descarregar a imagem.
    for _, child in ipairs(card.preview:GetChildren()) do
        if not child:IsA("UICorner") and not child:IsA("UIStroke") then child:Destroy() end
    end
    card.previewReady = false
end

local function make3DPreview(card, object)
    local model, viewport
    local ok, failure = pcall(function()
        model = Instance.new("Model")
        model.Name = "PetPreview"
        local candidates = object:GetDescendants()
        if object:IsA("BasePart") then table.insert(candidates, 1, object) end
        local visibleParts = {}
        for _, part in ipairs(candidates) do
            if part:IsA("BasePart") and part.Transparency < 1 then
                table.insert(visibleParts, part)
            end
        end
        if #visibleParts == 0 or #visibleParts > CONFIG.MaxPreviewParts then
            error("Modelo vazio ou acima do limite de peças da miniatura")
        end
        -- Copia somente a aparência. Scripts e interações não entram no PlayerGui.
        for _, part in ipairs(visibleParts) do
            local clone = part:Clone()
            if clone then
                for _, child in ipairs(clone:GetChildren()) do
                    if not (child:IsA("DataModelMesh") or child:IsA("Decal")
                        or child:IsA("SurfaceAppearance")) then
                        child:Destroy()
                    else
                        for _, nested in ipairs(child:GetChildren()) do nested:Destroy() end
                    end
                end
                clone.Anchored = true
                clone.CanCollide = false
                clone.CanTouch = false
                clone.CanQuery = false
                clone.Parent = model
            end
        end
        if #model:GetChildren() == 0 then error("Peças não podem ser copiadas") end
        -- Centraliza longe das coordenadas do mapa; preserva a orientação do pet.
        local sourcePivot = object:IsA("Model") and object:GetPivot() or getPart(object).CFrame
        for _, part in ipairs(model:GetChildren()) do
            part.CFrame = sourcePivot:ToObjectSpace(part.CFrame)
        end
        local bounds, size = model:GetBoundingBox()
        viewport = create("ViewportFrame", {
            Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1,
            ZIndex = card.preview.ZIndex + 1,
            Ambient = Color3.fromRGB(200, 205, 230),
            LightColor = Color3.fromRGB(255, 245, 235),
            LightDirection = Vector3.new(-1, -1, -2),
        }, card.preview)
        model.Parent = viewport
        local camera = create("Camera", { FieldOfView = 34 }, viewport)
        local radius = math.max(size.Magnitude / 2, 0.1)
        local distance = radius / math.sin(math.rad(camera.FieldOfView / 2)) * 1.12
        camera.CFrame = CFrame.lookAt(bounds.Position + Vector3.new(0.8, 0.45, -1).Unit * distance,
            bounds.Position)
        viewport.CurrentCamera = camera
        card.imageKind.Text = "PRÉVIA 3D"
    end)
    if not ok then
        if viewport then viewport:Destroy() end
        if model then model:Destroy() end
        local placeholder = label(card.preview, "?", 30, UDim2.new(), UDim2.fromScale(1, 1), THEME.Muted)
        placeholder.TextXAlignment = Enum.TextXAlignment.Center
        placeholder.ZIndex = card.preview.ZIndex + 1
        card.imageKind.Text = "SEM PRÉVIA"
        -- Falha visual não interrompe a lista nem as ações.
        card.previewError = tostring(failure)
    end
end

-- Miniaturas do catálogo: usa imagem real quando disponível e 3D como alternativa.
-- Não tenta obter imagens externas e só prepara opções que estão visíveis.
renderSelectionThumbnail = function(holder, option)
    if not holder or not holder.Parent then return end
    local source = option.sample
    if not (source and source.Parent) then
        for object, row in pairs(state.rowsByObject) do
            if object.Parent and row and (row.speciesId == option.value
                or nameKey(object.Name) == nameKey(option.rawName or option.text)) then
                source = object
                break
            end
        end
    end
    if not (source and source.Parent) then
        local matches = state.resources[nameKey(option.text)]
        if matches then
            for object in pairs(matches) do
                if object.Parent and (object:IsA("Model") or object:IsA("BasePart")) then
                    source = object
                    break
                end
            end
            if not source then
                for object in pairs(matches) do
                    if object.Parent then source = object; break end
                end
            end
        end
    end
    local fakeCard = { preview = holder, imageKind = { Text = "" } }
    local descriptor
    if source then
        local ok, found = pcall(function()
            local descendants = source:GetDescendants()
            return getImage(source, descendants,
                source:IsDescendantOf(Workspace) and linkedInterfaces(source) or {})
        end)
        if ok then descriptor = found end
    end
    descriptor = descriptor or imageDescriptor(CONFIG.PetIcons[option.rawName or option.text])
    local function fallback()
        if not holder.Parent then return end
        if source and source.Parent and (source:IsA("BasePart") or source:IsA("Model")) then
            make3DPreview(fakeCard, source)
        else
            local mark = label(holder, "?", 19, UDim2.new(), UDim2.fromScale(1, 1), THEME.Muted)
            mark.TextXAlignment, mark.ZIndex = Enum.TextXAlignment.Center, 56
        end
    end
    if not descriptor then fallback(); return end
    local picture = create("ImageLabel", {
        BackgroundTransparency = 1, Size = UDim2.fromScale(1, 1), ZIndex = 56,
        Image = descriptor.uri, ImageRectOffset = descriptor.offset,
        ImageRectSize = descriptor.size, ScaleType = Enum.ScaleType.Fit,
    }, holder)
    task.spawn(function()
        local deadline = os.clock() + CONFIG.ImageTimeout
        while state.running and holder.Parent and picture.Parent and
            not picture.IsLoaded and os.clock() < deadline do
            task.wait(0.15)
        end
        if not state.running or not holder.Parent or not picture.Parent then return end
        if not picture.IsLoaded then picture:Destroy(); fallback() end
    end)
end

local function renderPreview(card, object)
    if card.previewReady or not isAvailable(object) then return end
    card.previewReady = true
    local version = card.previewVersion
    local descriptor = card.image
    if not descriptor then make3DPreview(card, object) return end
    card.imageKind.Text = "CARREGANDO"
    local image = create("ImageLabel", {
        Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1,
        Image = descriptor.uri, ImageRectOffset = descriptor.offset,
        ImageRectSize = descriptor.size, ScaleType = Enum.ScaleType.Fit,
    }, card.preview)
    task.spawn(function()
        local deadline = os.clock() + CONFIG.ImageTimeout
        while state.running and card.frame.Parent and card.previewVersion == version
            and not image.IsLoaded and os.clock() < deadline do
            task.wait(0.1)
        end
        if not state.running or not card.frame.Parent or card.previewVersion ~= version then return end
        if image.IsLoaded then
            card.imageKind.Text = "ÍCONE DO JOGO"
        else
            -- Um ID privado, inválido ou indisponível não deixa um quadrado vazio.
            image:Destroy()
            make3DPreview(card, object)
        end
    end)
end

local previewRefreshVersion = 0
local function updateVisiblePreviews()
    if not state.running then return end
    previewRefreshVersion = previewRefreshVersion + 1
    local version = previewRefreshVersion
    -- CanvasPosition dispara continuamente durante scroll; espera estabilizar.
    task.delay(CONFIG.PreviewDebounceSeconds, function()
        if not state.running or version ~= previewRefreshVersion then return end
        local top = scroll.AbsolutePosition.Y
        local bottom = top + scroll.AbsoluteSize.Y
        local prepared = 0
        for object, card in pairs(state.cards) do
            if not state.running or version ~= previewRefreshVersion then return end
            if card.frame.Parent then
                local y = card.frame.AbsolutePosition.Y
                local active = state.page == "home" and state.showPreviews
                    and not state.minimized and card.frame.Visible
                local visible = active and y + card.frame.AbsoluteSize.Y > top and y < bottom
                if visible and not card.previewReady then
                    renderPreview(card, object)
                    prepared = prepared + 1
                    if prepared >= CONFIG.PreviewBatchSize then
                        prepared = 0
                        -- Clonagens de ViewportFrame são divididas entre frames.
                        task.wait()
                    end
                elseif card.previewReady and (not active
                    or y + card.frame.AbsoluteSize.Y < top - scroll.AbsoluteSize.Y
                    or y > bottom + scroll.AbsoluteSize.Y) then
                    -- Mantém um pequeno cache de miniaturas próximas à área visível.
                    clearPreview(card)
                end
                if not state.showPreviews and card.imageKind.Text ~= "PRÉVIA OCULTA" then
                    card.imageKind.Text = "PRÉVIA OCULTA"
                end
            end
        end
    end)
end

-- 7. AÇÕES: TELEPORTE, RETORNO E INTERAÇÃO -----------------------------------------

local function teleport(destination)
    local character, root = getRoot()
    if not hasBase() or not character or not root then return false end
    local ok = pcall(function()
        root.AssemblyLinearVelocity = Vector3.zero
        root.AssemblyAngularVelocity = Vector3.zero
        character:PivotTo(destination)
    end)
    return ok
end

local function findPrompt(object)
    for _, child in ipairs(object:GetDescendants()) do
        if child:IsA("ProximityPrompt") and child.Enabled then return child end
    end
end

local function promptPosition(prompt)
    local parent = prompt.Parent
    if not parent then return nil end
    if parent:IsA("Attachment") then return parent.WorldPosition end
    if parent:IsA("BasePart") then return parent.Position end
    if parent:IsA("Model") then return parent:GetPivot().Position end
end

local function cardStatus(card, message, color)
    if card and card.frame.Parent and state.running then
        card.action.Text = message
        card.action.TextColor3 = Color3.new(1, 1, 1)
    end
end

local function interact(object, card, allowed)
    local function permitted(validateTarget) return state.running and (not allowed or allowed(validateTarget)) end
    if not permitted(true) then return "cancelled" end
    if state.busy or not requireBase() or not isAvailable(object) then return "unavailable" end
    state.busy = true
    local outcome = "unavailable"
    local triggeredConnection, heldPrompt
    local baseAtStart = state.baseObject
    local ok, failure = xpcall(function()
        local prompt = findPrompt(object)
        local position = prompt and promptPosition(prompt)
        if not position then cardStatus(card, "Sem interação disponível", THEME.Warning) return end
        local character = player.Character
        if not permitted(true) then outcome = "cancelled"; return end
        cardStatus(card, "Indo até o pet...", THEME.Accent)
        local offset = math.min(2, prompt.MaxActivationDistance / 3)
        if not teleport(CFrame.new(position + Vector3.new(0, offset, offset))) then return end
        outcome = "unconfirmed"
        task.wait(0.25)
        if not permitted(true) then outcome = "cancelled"; return end
        local _, root = getRoot()
        position = promptPosition(prompt)
        if not isAvailable(object) or not root or player.Character ~= character
            or not position or not prompt.Enabled then
            outcome = "unavailable"
            cardStatus(card, "Interação interrompida", THEME.Warning) return
        end
        if (root.Position - position).Magnitude > prompt.MaxActivationDistance then
            cardStatus(card, "Fora do alcance", THEME.Warning) return
        end
        local triggered = false
        triggeredConnection = prompt.Triggered:Connect(function(who)
            if who == player then triggered = true end
        end)
        heldPrompt, state.activePrompt = prompt, prompt
        cardStatus(card, "Interagindo...", THEME.Accent)
        prompt:InputHoldBegin()
        local deadline = os.clock() + prompt.HoldDuration + 0.15
        while not triggered and os.clock() < deadline do
            if not permitted(true) then outcome = "cancelled"; return end
            if not prompt.Parent or not prompt.Enabled or not hasBase()
                or player.Character ~= character then return end
            task.wait(0.05)
        end
        prompt:InputHoldEnd()
        heldPrompt, state.activePrompt = nil, nil
        local confirmDeadline = os.clock() + 0.8
        while not triggered and os.clock() < confirmDeadline do
            if not permitted() then outcome = "cancelled"; return end
            if player.Character ~= character then return end
            task.wait(0.05)
        end
        if not permitted() then outcome = "cancelled"; return end
        if player.Character ~= character then return end
        if triggered then
            outcome = "sent"
            refreshBase()
            if not hasBase() or state.baseObject ~= baseAtStart then
                cardStatus(card, "Base alterada ou indisponível", THEME.Warning) return
            end
            local returned = teleport(state.base)
            cardStatus(card, returned and "Interação enviada • voltou à base" or "Retorno falhou",
                returned and THEME.Success or THEME.Warning)
        else
            cardStatus(card, "Interação não confirmada", THEME.Warning)
        end
    end, debug.traceback)
    if heldPrompt then pcall(function() heldPrompt:InputHoldEnd() end) end
    if triggeredConnection then triggeredConnection:Disconnect() end
    state.activePrompt, state.busy = nil, false
    if not ok then
        outcome = "failed"
        cardStatus(card, "Não foi possível interagir", THEME.Warning)
        warn("Leo Hub: " .. tostring(failure))
    end
    return outcome
end


-- Catálogo permanente: lista espécies mesmo que não estejam no Runtime.
-- Apenas lê instâncias e metadados replicados (NÃO executa ModuleScripts do jogo).
local Catalog = { entries = {}, count = 0 }
function Catalog.name(raw)
    local alias = CONFIG.Collection.Aliases[tostring(raw)]
    local name = cleanName(alias or tostring(raw))
    if not alias then
        -- Remover indicador de tamanho evita registrar uma espécie diferente por tamanho.
        local words = {}
        for word in name:gmatch("%S+") do
            if not identifySize(word) then table.insert(words, word) end
        end
        if #words > 0 then name = table.concat(words, " ") end
    end
    return name:match("^%s*(.-)%s*$")
end
function Catalog.speciesKey(name)
    -- Exu_Brasa, Exu Brasa e Brasa apontam para a MESMA espécie.
    local cleaned = normalize(Catalog.name(name))
    cleaned = cleaned:gsub("^exu[%s_%-]+", "")
    return key(cleaned)
end
function Catalog.identity(object)
    for _, field in ipairs(CONFIG.Collection.NameFields) do
        local value = object:GetAttribute(field)
        local child = object:FindFirstChild(field)
        if child and child:IsA("StringValue") then value = child.Value end
        if typeof(value) == "string" and value ~= "" then
            local name = Catalog.name(value)
            return Catalog.speciesKey(name), name
        end
    end
    local name = Catalog.name(object.Name)
    return Catalog.speciesKey(name), name
end
function Catalog.remember(name, source, sample)
    if type(name) ~= "string" then return false end
    name = Catalog.name(name)
    local id = Catalog.speciesKey(name)
    if id == "" or #name > 90 then return false end
    local entry = Catalog.entries[id]
    local added = false
    if not entry then
        entry = { name = name, source = source }
        Catalog.entries[id] = entry
        Catalog.count = Catalog.count + 1
        added = true
    else
        -- Dá preferência ao nome completo conhecido da espécie, sem perder seleção.
        if name:match("^[Ee][Xx][Uu]%s") and not entry.name:match("^[Ee][Xx][Uu]%s") then
            entry.name = name
        end
        if source == "catálogo" and entry.source ~= "catálogo" then entry.source = source end
    end
    if sample and (sample:IsA("Model") or sample:IsA("BasePart")) then
        -- Preferir o modelo replicado, que continua disponível após um despawn.
        local sourceReplicated = sample:IsDescendantOf(ReplicatedStorage)
        if not entry.sample or not entry.sample.Parent or sourceReplicated then
            entry.sample = sample
        end
    end
    return added
end
function Catalog.observe(object)
    local _, name = Catalog.identity(object)
    Catalog.remember(name, "observado", object)
end
function Catalog.isPetNamed(name)
    local text = tostring(name)
    return text:match("^[Ee][Xx][Uu][_%s%-]") ~= nil
        or text:match("^[Ee][Xx][Uu]%u") ~= nil
end
Catalog.groupNames = {
    all=true, models=true, templates=true, definitions=true,
    entries=true, catalog=true, catalogue=true, data=true,
    assets=true, collections=true, variants=true, types=true,
    commons=true, skins=true, pets=true, exus=true,
    image=true, images=true, icons=true, meshes=true, textures=true,
    configurations=true, settings=true, config=true,
}
function Catalog.isGroup(name)
    local text = key(name)
    if rarityLookup[text] or identifySize(name) then return true end
    for _, folder in ipairs(CONFIG.Collection.CatalogFolders) do
        if text == key(folder) then return true end
    end
    return Catalog.groupNames[text] == true
end
function Catalog.readListValue(object)
    if not object:IsA("StringValue") then return end
    local value = object.Value
    if value == "" or #value > 120000 then return end
    local ok, decoded = pcall(function()
        return game:GetService("HttpService"):JSONDecode(value)
    end)
    if ok and type(decoded) == "table" then
        for index, entry in pairs(decoded) do
            if type(entry) == "string" then
                Catalog.remember(entry, "catálogo")
            elseif type(entry) == "table" then
                local title = entry.PetName or entry.Species or entry.Name or entry.name
                if type(title) == "string" then Catalog.remember(title, "catálogo") end
            elseif type(index) == "string" and entry == true then
                Catalog.remember(index, "catálogo")
            end
        end
    else
        for title in value:gmatch("[^\r\n,;]+") do
            local trimmed = title:match("^%s*(.-)%s*$")
            if trimmed and trimmed ~= "" then Catalog.remember(trimmed, "catálogo") end
        end
    end
end
function Catalog.index(instance)
    if not instance or not instance:IsDescendantOf(ReplicatedStorage) then return end
    local parent = instance.Parent
    if not parent then return end
    local isString = instance:IsA("StringValue")
    if not (instance:IsA("Model") or instance:IsA("Folder")
        or instance:IsA("Configuration") or instance:IsA("ModuleScript")
        or isString) then return end
    local instanceName = key(instance.Name)
    for _, label in ipairs(CONFIG.Collection.CatalogValueNames) do
        if isString and instanceName == key(label) then
            Catalog.readListValue(instance)
            return
        end
    end

    -- Determina se existe uma pasta de catálogo entre este objeto e ReplicatedStorage.
    local catalogRoot
    local cursor = parent
    while cursor and cursor ~= ReplicatedStorage do
        for _, name in ipairs(CONFIG.Collection.CatalogFolders) do
            if key(cursor.Name) == key(name) then
                catalogRoot = cursor
                break
            end
        end
        if catalogRoot then break end
        cursor = cursor.Parent
    end
    local explicitName = Catalog.isPetNamed(instance.Name)
    if not catalogRoot and not explicitName then return end
    if Catalog.isGroup(instance.Name) then return end

    -- Campos de nome também são válidos mesmo se forem filhos de um Model.
    if isString then
        for _, field in ipairs(CONFIG.Collection.NameFields) do
            if instanceName == key(field) then
                local value = instance.Value
                if value ~= "" then Catalog.remember(value, "catálogo", parent) end
                return
            end
        end
        if not catalogRoot and not explicitName then return end
    end

    -- Nunca cadastrar submodelos e acessórios pertencentes a um template de pet.
    cursor = parent
    while cursor and cursor ~= ReplicatedStorage do
        if cursor == catalogRoot then break end
        if (cursor:IsA("Model") or cursor:IsA("Configuration"))
            and not Catalog.isGroup(cursor.Name) then return end
        cursor = cursor.Parent
    end

    -- Uma pasta genérica dentro do catálogo não é uma espécie.
    if instance:IsA("Folder") and not explicitName then
        local hasDefinition = false
        for _, attribute in ipairs(CONFIG.Collection.NameFields) do
            if typeof(instance:GetAttribute(attribute)) == "string" then
                hasDefinition = true
                break
            end
        end
        if not hasDefinition then return end
    end

    local _, title = Catalog.identity(instance)
    Catalog.remember(title, "catálogo", instance)
end
for _, name in ipairs(CONFIG.Collection.PetCatalog) do
    Catalog.remember(name, "configurado")
end

-- Um único coletor processa a fila. A lista visual não é reconstruída por ele.
local Auto = {
    enabled = false, version = 0, working = false, active = false,
    pets = {}, sizes = {}, queue = {}, pending = {},
    attempted = setmetatable({}, { __mode = "k" }),
}
function Auto.count(set)
    local count = 0
    for _ in pairs(set) do count = count + 1 end
    return count
end
function Auto.updateUI(message)
    if not state.running then return end
    petSelect.Text = string.format("Selecionar pets (%d)", Auto.count(Auto.pets))
    sizeSelect.Text = string.format("Selecionar tamanhos (%d)", Auto.count(Auto.sizes))
    autoToggle.Text = Auto.enabled and "Parar coleta automática" or "Ativar coleta automática"
    autoToggle.BackgroundColor3 = Auto.enabled and THEME.Warning or THEME.Accent
    -- Só altera a animação quando o estado da coleta muda.
    if state.autoSpinner and state.autoSpinnerRunning ~= Auto.enabled then
        state.autoSpinnerRunning = Auto.enabled
        state.autoSpinner.Visible = Auto.enabled
        if Auto.enabled then
            state.autoSpinnerGradient.Rotation = 0
            state.autoSpinnerTween:Play()
        else
            state.autoSpinnerTween:Cancel()
            state.autoSpinnerGradient.Rotation = 0
        end
    end
    -- Evita recriar o aviso textual; o botão já indica se a coleta está ativa.
    autoStatus.Text = ""
end
function Auto.matches(object)
    if not isAvailable(object) or not getPart(object) then return false end
    local hasPetFilter = next(Auto.pets) ~= nil
    local hasSizeFilter = next(Auto.sizes) ~= nil
    -- Um único filtro é suficiente. Se ambos forem usados, ambos precisam combinar.
    if not hasPetFilter and not hasSizeFilter then return false end
    if hasPetFilter and not Auto.pets[Catalog.identity(object)] then return false end
    if hasSizeFilter then
        local size = getSize(object, object:GetDescendants(), linkedInterfaces(object))
        if not Auto.sizes[size] then return false end
    end
    return true
end
function Auto.cancel(message)
    Auto.version = Auto.version + 1
    table.clear(Auto.queue)
    table.clear(Auto.pending)
    if Auto.active then releasePrompt() end
    Auto.updateUI(message)
end
function Auto.stop(message)
    Auto.enabled = false
    Auto.cancel(message)
end
function Auto.changed()
    Auto.stop("Seleção alterada • ative para começar")
end
function Auto.pump()
    if Auto.working or not Auto.enabled or not state.running then return end
    Auto.working = true
    task.spawn(function()
        local ok, failure = xpcall(function()
            while state.running and Auto.enabled and #Auto.queue > 0 do
                -- Aguarda base e personagem sem percorrer novamente a lista de pets.
                local first = Auto.queue[1]
                if not isAvailable(first) or not Auto.matches(first) then
                    table.remove(Auto.queue, 1)
                    Auto.pending[first] = nil
                elseif state.busy or not hasBase() or not getRoot() then
                    Auto.updateUI("Ativa • aguardando base/personagem")
                    task.wait(0.2)
                else
                    local object = table.remove(Auto.queue, 1)
                    Auto.pending[object] = nil
                    if not Auto.attempted[object] and Auto.matches(object) and findPrompt(object) then
                        local version = Auto.version
                        local function allowed(validateTarget)
                            return Auto.enabled and Auto.version == version and state.running
                                and (not validateTarget or Auto.matches(object))
                        end
                        Auto.active, Auto.inflight = true, object
                        Auto.updateUI("Coletando: " .. Catalog.name(object.Name))
                        local outcome = interact(object, state.cards[object], allowed)
                        Auto.active, Auto.inflight = false, nil
                        if Auto.version ~= version and Auto.enabled then Auto.consider(object) end
                        if Auto.version == version and outcome ~= "cancelled" and outcome ~= "unavailable" then
                            Auto.attempted[object] = true
                        end
                        if outcome == "unavailable" and Auto.version == version
                            and Auto.enabled and Auto.matches(object) and not Auto.pending[object] then
                            Auto.pending[object] = true
                            table.insert(Auto.queue, object)
                        end
                        if state.running and Auto.enabled then Auto.updateUI() end
                        task.wait(CONFIG.Collection.BetweenPetsSeconds)
                    end
                end
            end
        end, debug.traceback)
        Auto.active, Auto.working, Auto.inflight = false, false, nil
        if not ok then
            releasePrompt()
            Auto.stop("Coleta interrompida por erro")
            warn("Leo Hub / coleta: " .. tostring(failure))
        end
        if state.running and Auto.enabled and #Auto.queue > 0 then Auto.pump() end
    end)
end
function Auto.consider(object)
    if not state.running or not isAvailable(object) or not getPart(object) then return end
    local oldCount = Catalog.count
    Catalog.observe(object)
    if not Auto.enabled and Catalog.count ~= oldCount then Auto.updateUI() end
    if Auto.enabled and object ~= Auto.inflight and not Auto.attempted[object] and not Auto.pending[object]
        and Auto.matches(object) and findPrompt(object) then
        Auto.pending[object] = true
        table.insert(Auto.queue, object)
        Auto.pump()
    end
end
function Auto.start()
    if not next(Auto.pets) and not next(Auto.sizes) then
        Auto.updateUI("Selecione pelo menos um pet OU um tamanho")
        return
    end
    refreshBase(true)
    Auto.cancel()
    Auto.attempted = setmetatable({}, { __mode = "k" })
    Auto.enabled = true
    Auto.updateUI()
    local startedVersion = Auto.version
    local objects = state.runtime and state.runtime:GetChildren() or {}
    task.spawn(function()
        for index, object in ipairs(objects) do
            if not state.running or not Auto.enabled or Auto.version ~= startedVersion then return end
            Auto.consider(object)
            if index % CONFIG.Collection.PetUpdateBatchSize == 0 then task.wait() end
        end
    end)
end


-- 8. CARTÕES ------------------------------------------------------------------

local function addCard(object)
    local frame = surface(scroll, "Pet", UDim2.new(), UDim2.new(1, -6, 0, 140), THEME.Input)
    local portrait = surface(frame, "Portrait", UDim2.fromOffset(12, 12), UDim2.fromOffset(72, 72), THEME.Background)
    portrait.ClipsDescendants = true
    state.cardSequence = state.cardSequence + 1
    local card = {
        sequence = state.cardSequence,
        frame = frame, preview = portrait, previewReady = false, previewVersion = 0,
        imageKey = "", image = nil,
        name = label(frame, cleanName(object.Name), 14, UDim2.fromOffset(98, 10), UDim2.new(1, -146, 0, 23)),
        rarity = label(frame, "Raridade: verificando", 11, UDim2.fromOffset(98, 35), UDim2.new(1, -112, 0, 18), THEME.Accent),
        size = label(frame, "Tamanho: verificando", 11, UDim2.fromOffset(98, 56), UDim2.new(1, -112, 0, 18), THEME.Muted),
        distance = label(frame, "", 10, UDim2.fromOffset(98, 77), UDim2.new(1, -112, 0, 17), THEME.Muted),
        action = label(frame, "Pronto", 9, UDim2.fromOffset(12, 115), UDim2.fromOffset(172, 16), THEME.Muted),
        position = label(frame, "", 10, UDim2.new(1, -44, 0, 12), UDim2.fromOffset(30, 19), THEME.Muted),
        imageKind = label(frame, "PRÉVIA", 7, UDim2.fromOffset(8, 87), UDim2.fromOffset(80, 16), THEME.Muted),
    }
    card.name.Font = Enum.Font.FredokaOne
    card.imageKind.TextXAlignment = Enum.TextXAlignment.Center
    card.position.TextXAlignment = Enum.TextXAlignment.Right
    local tp = button(frame, "📍 Ir até pet", UDim2.new(1, -214, 1, -38), UDim2.fromOffset(96, 28))
    tp.BackgroundColor3 = THEME.Surface
    local take = button(frame, "🐾 Pegar", UDim2.new(1, -108, 1, -38), UDim2.fromOffset(96, 28), true)
    tp.Activated:Connect(function()
        if state.busy then notify("Aguarde a interação atual terminar.", THEME.Warning) return end
        if not requireBase() then return end
        local part = isAvailable(object) and getPart(object)
        local moved = part and teleport(CFrame.new(part.Position + Vector3.new(0, 3, 0)))
        cardStatus(card, moved and "Teleporte solicitado" or "Pet indisponível", moved and THEME.Success or THEME.Warning)
    end)
    take.Activated:Connect(function() interact(object, card) end)
    state.cards[object] = card
    return card
end


-- 9. ÍNDICES E ATUALIZAÇÃO -----------------------------------------------------
-- O mapa inteiro é percorrido uma vez para indexar interfaces; depois usamos eventos.

local Engine = {}

local function indexGui(instance)
    indexBaseContainer(instance)
    if instance:IsA("BillboardGui") or instance:IsA("SurfaceGui") then
        state.linkedGuis[instance] = true
        if Engine.watchGui then Engine.watchGui(instance) end
    end
end

local resourceKeys = {}
local function removeResource(instance)
    local key = resourceKeys[instance]
    if key and state.resources[key] then
        state.resources[key][instance] = nil
        if next(state.resources[key]) == nil then state.resources[key] = nil end
    end
    resourceKeys[instance] = nil
end

local function indexResource(instance)
    -- Novas espécies podem chegar como atributos/valores depois do objeto replicar.
    if instance:IsA("StringValue") then
        local isRelevant = false
        local fieldKey = key(instance.Name)
        for _, field in ipairs(CONFIG.Collection.NameFields) do
            if key(field) == fieldKey then isRelevant = true; break end
        end
        if not isRelevant then
            for _, field in ipairs(CONFIG.Collection.CatalogValueNames) do
                if key(field) == fieldKey then isRelevant = true; break end
            end
        end
        if isRelevant then
            connect(instance:GetPropertyChangedSignal("Value"), function()
                if not state.running then return end
                local previous = Catalog.count
                Catalog.index(instance)
                if previous ~= Catalog.count then Auto.updateUI() end
            end)
        end
    end
    local previousCount = Catalog.count
    Catalog.index(instance)
    if Catalog.count ~= previousCount and not Auto.enabled then Auto.updateUI() end
    if not (instance:IsA("Model") or instance:IsA("Folder") or instance:IsA("Configuration")
        or instance:IsA("ImageLabel") or instance:IsA("ImageButton")) then return end
    local key = nameKey(instance.Name)
    state.resources[key] = state.resources[key] or {}
    state.resources[key][instance] = true
    resourceKeys[instance] = key
end

connect(Workspace.DescendantAdded, function(instance)
    indexGui(instance)
    if baseContainerNames[key(instance.Name)] or ownerFields[key(instance.Name)]
        or pointRanks[key(instance.Name)] or instance:IsA("BillboardGui")
        or instance:IsA("SurfaceGui") then
        nextBaseScan = 0
    end
end)
connect(Workspace.DescendantRemoving, function(instance)
    if instance == state.baseObject or instance == state.basePoint
        or ownerFields[key(instance.Name)] or baseContainerNames[key(instance.Name)] then
        nextBaseScan = 0
    end
    state.linkedGuis[instance] = nil
    if Engine.removeGui then Engine.removeGui(instance) end
    state.baseContainers[instance] = nil
end)
connect(playerGui.DescendantAdded, function(instance)
    indexGui(instance)
    if instance:IsA("BillboardGui") or instance:IsA("SurfaceGui") then nextBaseScan = 0 end
end)
connect(playerGui.DescendantRemoving, function(instance)
    state.linkedGuis[instance] = nil
    if Engine.removeGui then Engine.removeGui(instance) end
end)
connect(ReplicatedStorage.DescendantAdded, indexResource)
connect(ReplicatedStorage.DescendantRemoving, removeResource)
for _, instance in ipairs(Workspace:GetDescendants()) do indexGui(instance) end
for _, instance in ipairs(playerGui:GetDescendants()) do indexGui(instance) end
for _, instance in ipairs(ReplicatedStorage:GetDescendants()) do indexResource(instance) end

-- Busca e ordenação apenas reorganizam a fotografia da rodada; não varrem o mapa.
local function renderList()
    if not state.running then return end
    local rows = state.snapshotRows
    local query = normalize(search.Text):match("^%s*(.-)%s*$")
    local orderDescription = sortRows(rows)
    local visible = 0
    for index, row in ipairs(rows) do
        local card = row.card
        card.frame.LayoutOrder = index
        card.frame.Visible = query == "" or row.searchText:find(query, 1, true) ~= nil
        if card.frame.Visible then
            visible = visible + 1
            card.position.Text = "#" .. visible
        end
    end
    info.Text = string.format("%d de %d pets", visible, #rows)
    statLabels[3].Text = tostring(#rows)
    empty.Visible = visible == 0
    if #rows == 0 then empty.Text = "Aguardando novos pets spawnarem..."
    else empty.Text = "Nenhum pet corresponde à busca." end
    task.defer(updateVisiblePreviews)
end

-- Atualização incremental: compara os dados da instância antes de tocar na interface.
-- Movimento do jogador, ping, animações e manutenção NÃO modificam a lista.
local function removePetCard(object)
    local row = state.rowsByObject[object]
    if not row then return false end
    state.rowsByObject[object] = nil
    for index, item in ipairs(state.snapshotRows) do
        if item.object == object then table.remove(state.snapshotRows, index); break end
    end
    local card = state.cards[object]
    if card then
        clearPreview(card)
        card.frame:Destroy()
        state.cards[object] = nil
    end
    
    return true
end

local function upsertPetCard(object)
    if not state.runtime or object.Parent ~= state.runtime or not isAvailable(object)
        or not (object:IsA("Model") or object:IsA("Folder") or object:IsA("BasePart")) then
        return removePetCard(object)
    end
    local part = getPart(object)
    if not part then return removePetCard(object) end

    local descendants = object:GetDescendants()
    local interfaces = linkedInterfaces(object)
    local petSize = getSize(object, descendants, interfaces)
    local stats = getPetStats(object, descendants, interfaces)
    local name = cleanName(object.Name)
    local descriptor = getImage(object, descendants, interfaces)
    local imageKey = descriptor and (descriptor.uri .. tostring(descriptor.offset) .. tostring(descriptor.size)) or "3d"
    local speciesId = Catalog.identity(object)
    local signature = table.concat({ object.Name, name, speciesId, petSize, stats.rarity,
        tostring(stats.rarityRank), imageKey }, "\0")
    local row = state.rowsByObject[object]
    if row and row.signature == signature then return false end

    local oldCount = Catalog.count
    Catalog.observe(object)
    if oldCount ~= Catalog.count and not Auto.enabled then Auto.updateUI() end
    local card = state.cards[object] or addCard(object)
    if not row then
        row = { object = object, card = card }
        state.rowsByObject[object] = row
        table.insert(state.snapshotRows, row)
    end
    local _, root = getRoot()
    local distance = root and (root.Position - part.Position).Magnitude or math.huge
    row.signature = signature
    row.name = normalize(name)
    row.speciesId = speciesId
    row.distance = distance
    row.rarityRank = stats.rarityRank
    row.sizeRank = SIZE_RANKS[petSize]
    row.searchText = normalize(name .. " " .. object.Name .. " " .. petSize .. " " .. stats.rarity)
    card.frame.BackgroundTransparency = 0
    card.name.Text = name
    card.size.Text = "Tamanho: " .. petSize
    card.rarity.Text = "Raridade: " .. stats.rarity
    card.distance.Text = distance < math.huge and string.format("Na atualização: %d studs", math.floor(distance))
        or "Distância indisponível"
    card.rarityName = stats.rarity
    card.distance.Visible = state.showDistance
    if imageKey ~= card.imageKey then
        card.imageKey, card.image = imageKey, descriptor
        clearPreview(card)
    end
    
    -- Não redefine "Coletando" ou o resultado de uma interação em cartões existentes.
    return true
end

-- Varredura completa somente na inicialização/troca da população.
-- O processamento é dividido entre frames quando uma rodada tem muitos pets.
local function refresh()
    if not state.running then return end
    state.fullRefreshVersion = (state.fullRefreshVersion or 0) + 1
    local version, runtime = state.fullRefreshVersion, state.runtime
    task.spawn(function()
        local active, changed = {}, false
        local objects = runtime and runtime:GetChildren() or {}
        for index, object in ipairs(objects) do
            if not state.running or state.runtime ~= runtime
                or state.fullRefreshVersion ~= version then return end
            active[object] = true
            if upsertPetCard(object) then changed = true end
            if index % CONFIG.Collection.PetUpdateBatchSize == 0 then task.wait() end
        end
        if not state.running or state.runtime ~= runtime
            or state.fullRefreshVersion ~= version then return end
        for object in pairs(state.rowsByObject) do
            if not active[object] and removePetCard(object) then changed = true end
        end
        if changed then
            Auto.updateUI()
            renderList()
        end
    end)
end


-- Eventos do mapa controlam os spawns/despawns e as alterações de metadados.
-- Apenas os pets modificados são reavaliados; sem atualização da lista em loop.
Engine.petBuckets = {}
Engine.guiBuckets = {}
Engine.runtimeConnections = {}
Engine.baseline = {}
Engine.pendingRound = false
Engine.settleVersion = 0
Engine.snapshotScheduled = false
Engine.pendingUpdates = {}
Engine.refreshScheduled = false
Engine.refreshVersion = 0

local petListRenderScheduled = false
local function schedulePetListRender()
    if petListRenderScheduled or not state.running then return end
    petListRenderScheduled = true
    task.delay(0.10, function()
        petListRenderScheduled = false
        if not state.running then return end
        Auto.updateUI()
        renderList()
    end)
end

function Engine.requestPetUpdate(object)
    if not state.running or not object then return end
    Engine.pendingUpdates[object] = true
    if Engine.refreshScheduled then return end
    Engine.refreshScheduled = true
    local version = Engine.refreshVersion
    task.delay(CONFIG.Collection.PetUpdateDebounceSeconds, function()
        if not state.running or version ~= Engine.refreshVersion then return end
        task.spawn(function()
            while state.running and version == Engine.refreshVersion do
                local updates = Engine.pendingUpdates
                Engine.pendingUpdates = {}
                if not next(updates) then break end
                local processed = 0
                local changed = false
                for pet in pairs(updates) do
                    if not state.running or version ~= Engine.refreshVersion then return end
                    if upsertPetCard(pet) then changed = true end
                    processed = processed + 1
                    if processed >= CONFIG.Collection.PetUpdateBatchSize then
                        processed = 0
                        if changed then schedulePetListRender(); changed = false end
                        task.wait() -- divide trabalho pesado entre frames
                    end
                end
                if changed then schedulePetListRender() end
            end
            if version ~= Engine.refreshVersion then return end
            Engine.refreshScheduled = false
            -- Uma atualização que chegou no fim do processamento não se perde.
            if next(Engine.pendingUpdates) then
                local pending = Engine.pendingUpdates
                Engine.pendingUpdates = {}
                for pet in pairs(pending) do Engine.requestPetUpdate(pet) end
            end
        end)
    end)
end

function Engine.disconnect(bucket)
    for _, connection in ipairs(bucket.connections or {}) do connection:Disconnect() end
    for _, connections in pairs(bucket.nodes or {}) do
        for _, connection in ipairs(connections) do connection:Disconnect() end
    end
end
local petNameFields = makeSet(CONFIG.Collection.NameFields)
local function isPetMetadataAttribute(name)
    local normalized = key(name)
    return petNameFields[normalized] or rarityFields[normalized]
        or sizeField(name) or imageFieldKeys[normalize(name)]
end

function Engine.observeNode(bucket, object, changed)
    if bucket.nodes[object] then return end
    local connections = {}
    bucket.nodes[object] = connections
    -- Ignora atributos de animação/efeitos que não afetam o catálogo.
    table.insert(connections, object.AttributeChanged:Connect(function(attribute)
        if isPetMetadataAttribute(attribute) then changed() end
    end))
    table.insert(connections, object:GetPropertyChangedSignal("Name"):Connect(changed))
    if object:IsA("StringValue") or object:IsA("NumberValue") or object:IsA("IntValue") then
        table.insert(connections, object:GetPropertyChangedSignal("Value"):Connect(changed))
    elseif object:IsA("TextLabel") or object:IsA("TextButton") then
        table.insert(connections, object:GetPropertyChangedSignal("Text"):Connect(changed))
    elseif object:IsA("ProximityPrompt") then
        table.insert(connections, object:GetPropertyChangedSignal("Enabled"):Connect(changed))
    end
end
function Engine.removeNode(bucket, object)
    for _, connection in ipairs(bucket.nodes[object] or {}) do connection:Disconnect() end
    bucket.nodes[object] = nil
end
function Engine.scheduleSnapshot()
    if not state.running or not Engine.pendingRound or Engine.snapshotScheduled then return end
    Engine.snapshotScheduled = true
    local version, runtime = Engine.settleVersion, state.runtime
    task.delay(CONFIG.Collection.ResetSettleSeconds, function()
        if not state.running or version ~= Engine.settleVersion or runtime ~= state.runtime then return end
        Engine.snapshotScheduled = false
        local baseline = {}
        for _, object in ipairs(runtime and runtime:GetChildren() or {}) do
            if getPart(object) then baseline[object] = true end
        end
        -- Se ainda não há pets, o próximo spawn inicia uma nova tentativa.
        if not next(baseline) then return end
        Engine.baseline, Engine.pendingRound = baseline, false
        refresh()
    end)
end
function Engine.beginRound(cancelActive)
    if not state.running then return end
    state.fullRefreshVersion = (state.fullRefreshVersion or 0) + 1
    Engine.settleVersion = Engine.settleVersion + 1
    Engine.snapshotScheduled = false
    Engine.pendingRound = true
    if cancelActive then
        Auto.cancel("Rodada reiniciada • aguardando pets")
        Auto.attempted = setmetatable({}, { __mode = "k" })
    end
    Engine.scheduleSnapshot()
end
function Engine.changed(object)
    local bucket = Engine.petBuckets[object]
    if not bucket or bucket.scheduled then return end
    bucket.scheduled = true
    local elapsed = os.clock() - (bucket.lastChangeProcessed or -math.huge)
    local waitTime = math.max(0.08, CONFIG.Collection.PetChangeMinimumSeconds - elapsed)
    task.delay(waitTime, function()
        if not state.running or Engine.petBuckets[object] ~= bucket then return end
        bucket.scheduled = false
        bucket.lastChangeProcessed = os.clock()
        if Auto.enabled then Auto.consider(object) end
        if not Engine.pendingRound then Engine.requestPetUpdate(object) end
        if not bucket.ready and getPart(object) then
            bucket.ready = true
            if Engine.pendingRound then Engine.scheduleSnapshot() end
        end
    end)
end
function Engine.watch(object)
    if Engine.petBuckets[object] then return end
    if not (object:IsA("Model") or object:IsA("BasePart") or object:IsA("Folder")) then return end
    local bucket = { connections = {}, nodes = {} }
    Engine.petBuckets[object] = bucket
    local function changed() Engine.changed(object) end
    Engine.observeNode(bucket, object, changed)
    for _, node in ipairs(object:GetDescendants()) do Engine.observeNode(bucket, node, changed) end
    table.insert(bucket.connections, object.DescendantAdded:Connect(function(node)
        Engine.observeNode(bucket, node, changed)
        changed()
        if Engine.pendingRound then Engine.scheduleSnapshot() end
    end))
    table.insert(bucket.connections, object.DescendantRemoving:Connect(function(node)
        Engine.removeNode(bucket, node)
        changed()
    end))
    changed()
end
function Engine.unwatch(object)
    local bucket = Engine.petBuckets[object]
    if bucket then Engine.disconnect(bucket); Engine.petBuckets[object] = nil end
    Auto.pending[object] = nil
    local card = state.cards[object]
    if card then
        cardStatus(card, "Saiu do mapa", THEME.Muted)
        card.frame.BackgroundTransparency = 0.4
    end
    local belonged = Engine.baseline[object]
    Engine.baseline[object] = nil
    if belonged and not next(Engine.baseline) then Engine.beginRound(false) end
    Engine.requestPetUpdate(object)
    if Engine.pendingRound then Engine.scheduleSnapshot() end
end
function Engine.rootFor(anchor)
    if not anchor or not state.runtime then return nil end
    local cursor = anchor
    while cursor and cursor.Parent ~= state.runtime do cursor = cursor.Parent end
    return cursor
end
function Engine.watchGui(interface)
    if Engine.guiBuckets[interface] then return end
    local bucket = { connections = {}, nodes = {}, watching = false }
    Engine.guiBuckets[interface] = bucket
    local function changed()
        local root = Engine.rootFor(interface.Adornee or interface.Parent)
        if root and not bucket.watching then
            bucket.watching = true
            for _, node in ipairs(interface:GetDescendants()) do
                Engine.observeNode(bucket, node, changed)
            end
        elseif not root and bucket.watching then
            bucket.watching = false
            for node in pairs(bucket.nodes) do Engine.removeNode(bucket, node) end
        end
        if root then Engine.changed(root) end
    end
    -- GUIs de outras áreas do mapa não ganham centenas de listeners desnecessários.
    table.insert(bucket.connections, interface:GetPropertyChangedSignal("Adornee"):Connect(changed))
    table.insert(bucket.connections, interface.AncestryChanged:Connect(changed))
    table.insert(bucket.connections, interface.DescendantAdded:Connect(function(node)
        if bucket.watching then Engine.observeNode(bucket, node, changed); changed() end
    end))
    table.insert(bucket.connections, interface.DescendantRemoving:Connect(function(node)
        if bucket.watching then Engine.removeNode(bucket, node); changed() end
    end))
    changed()
end
function Engine.removeGui(interface)
    local bucket = Engine.guiBuckets[interface]
    if bucket then Engine.disconnect(bucket); Engine.guiBuckets[interface] = nil end
end
function Engine.bind(runtime)
    if runtime == state.runtime then return end
    Engine.settleVersion = Engine.settleVersion + 1
    Engine.snapshotScheduled = false
    Engine.refreshVersion = Engine.refreshVersion + 1
    Engine.refreshScheduled = false
    table.clear(Engine.pendingUpdates)
    for _, connection in ipairs(Engine.runtimeConnections) do connection:Disconnect() end
    table.clear(Engine.runtimeConnections)
    for object, bucket in pairs(Engine.petBuckets) do
        Engine.disconnect(bucket)
        Engine.petBuckets[object] = nil
    end
    table.clear(Engine.baseline)
    state.runtime = runtime
    Engine.beginRound(true)
    -- Remove imediatamente cartões de modelos que pertenciam ao Runtime anterior.
    local removed = false
    for object in pairs(state.rowsByObject) do
        if object.Parent ~= runtime and removePetCard(object) then removed = true end
    end
    if removed then renderList() end
    if not runtime then return end
    table.insert(Engine.runtimeConnections, runtime.ChildAdded:Connect(function(object)
        Engine.watch(object)
        if Engine.pendingRound then Engine.scheduleSnapshot() end
    end))
    table.insert(Engine.runtimeConnections, runtime.ChildRemoved:Connect(Engine.unwatch))
    -- Quando disponível, um identificador de rodada também detecta reutilização dos modelos.
    for _, name in ipairs(CONFIG.Collection.ResetAttributes) do
        table.insert(Engine.runtimeConnections, runtime:GetAttributeChangedSignal(name):Connect(function()
            Engine.beginRound(true)
            for _, object in ipairs(runtime:GetChildren()) do Engine.changed(object) end
        end))
    end
    for interface in pairs(state.linkedGuis) do
        Engine.removeGui(interface)
        Engine.watchGui(interface)
    end
    -- Modelos de pets são observados aos poucos na inicialização da rodada.
    local initial = runtime:GetChildren()
    local bindingVersion = Engine.settleVersion
    task.spawn(function()
        for index, object in ipairs(initial) do
            if not state.running or state.runtime ~= runtime
                or bindingVersion ~= Engine.settleVersion then return end
            if object.Parent == runtime then Engine.watch(object) end
            if index % CONFIG.Collection.PetUpdateBatchSize == 0 then task.wait() end
        end
        if state.running and state.runtime == runtime then Engine.scheduleSnapshot() end
    end)
    Engine.scheduleSnapshot()
end
function Engine.start()
    connect(Workspace.DescendantAdded, function(object)
        if object.Name == CONFIG.RuntimeName and (not state.runtime or not state.runtime:IsDescendantOf(Workspace)) then
            Engine.bind(object)
        end
    end)
    connect(Workspace.DescendantRemoving, function(object)
        if object == state.runtime then
            task.defer(function()
                if state.running then Engine.bind(Workspace:FindFirstChild(CONFIG.RuntimeName, true)) end
            end)
        end
    end)
    Engine.bind(Workspace:FindFirstChild(CONFIG.RuntimeName, true))
    Auto.updateUI()
end
function Engine.cleanup()
    Engine.settleVersion = Engine.settleVersion + 1
    Engine.refreshVersion = Engine.refreshVersion + 1
    Engine.snapshotScheduled = false
    Engine.refreshScheduled = false
    table.clear(Engine.pendingUpdates)
    Auto.stop()
    for _, connection in ipairs(Engine.runtimeConnections) do connection:Disconnect() end
    for _, bucket in pairs(Engine.petBuckets) do Engine.disconnect(bucket) end
    for _, bucket in pairs(Engine.guiBuckets) do Engine.disconnect(bucket) end
    table.clear(Engine.petBuckets)
    table.clear(Engine.guiBuckets)
    table.clear(Engine.runtimeConnections)
end


local function countSet(set)
    local n = 0
    for _ in pairs(set) do n = n + 1 end
    return n
end

local function styleToggle(item, enabled, onText, offText)
    item.Text = enabled and onText or offText
    item.BackgroundColor3 = enabled and THEME.Accent or THEME.Input
    item.TextColor3 = Color3.new(1, 1, 1)
    item:SetAttribute("ButtonAccent", enabled and true or false)
    for _, child in ipairs(item:GetChildren()) do
        if child:IsA("UIStroke") and child:GetAttribute("ButtonStroke") then
            child.Color = enabled and THEME.Highlight or THEME.Border
            child.Transparency = enabled and 0.50 or 0.58
        end
    end
end

local function applyThemeToTree(root)
    for _, object in ipairs(root:GetDescendants()) do
        local surfaceRole = object:GetAttribute("SurfaceRole")
        if (object:IsA("Frame") or object:IsA("CanvasGroup")) and object.BackgroundTransparency < 1 then
            if object:GetAttribute("AboutAccentSurface") then object.BackgroundColor3 = THEME.Accent
            elseif surfaceRole == "rail" then object.BackgroundColor3 = THEME.Rail
            elseif surfaceRole == "background" then object.BackgroundColor3 = THEME.Background
            elseif surfaceRole == "surface" then object.BackgroundColor3 = THEME.Surface end
        elseif object:IsA("TextLabel") then
            object.TextColor3 = Color3.new(1, 1, 1)
        elseif object:IsA("TextBox") then
            if object.BackgroundTransparency < 1 then object.BackgroundColor3 = THEME.Input end
            object.TextColor3 = Color3.new(1, 1, 1)
            object.PlaceholderColor3 = Color3.new(1, 1, 1)
        elseif object:IsA("TextButton") and object:GetAttribute("StyledButton") then
            local accent = object:GetAttribute("ButtonAccent") == true
            object.BackgroundColor3 = accent and THEME.Accent or THEME.Input
            object.TextColor3 = Color3.new(1, 1, 1)
        elseif object:IsA("ScrollingFrame") then
            object.ScrollBarImageColor3 = THEME.Accent
        elseif object:IsA("UIStroke") then
            if object:GetAttribute("StyledOutline") then
                object.Color = THEME.Border
            elseif object:GetAttribute("ButtonStroke") then
                local parent = object.Parent
                local accent = parent and parent:GetAttribute("ButtonAccent") == true
                object.Color = accent and THEME.Highlight or THEME.Border
            elseif object:GetAttribute("MainAccentBorder") then
                object.Color = THEME.Accent
            end
        end
    end
    main.BackgroundColor3 = THEME.Background
    main.BackgroundTransparency = 0.07
    footer.TextColor3 = Color3.new(1, 1, 1)
    autoToggle.BackgroundColor3 = Auto.enabled and THEME.Warning or THEME.Accent
    autoToggle.TextColor3 = Color3.new(1, 1, 1)
    for _, line in ipairs(settingsBox:GetChildren()) do
        if line:IsA("Frame") and line.Size.Y.Offset <= 2 then
            line.BackgroundColor3 = THEME.Border
        end
    end
    for _, card in pairs(state.cards) do
        if card and card.frame and card.frame.Parent then
            card.frame.BackgroundColor3 = THEME.Surface
            if card.preview then card.preview.BackgroundColor3 = THEME.Rail end
        end
    end
end

local function applyTheme(name)
    local preset = THEME_PRESETS[name] or THEME_PRESETS.roxo
    Settings.theme = name
    THEME.Accent = preset.Accent
    THEME.Highlight = preset.Highlight
    THEME.Warning = preset.Warning
    applyThemeToTree(gui)
    if state.page then
        local activePage = state.page
        state.page = nil
        showPage(activePage)
    end
end

local AntiAFK = { loopStarted = false }
function AntiAFK.pulse()
    if not state.running or not Settings.antiAfk then return end
    pcall(function()
        VirtualUser:CaptureController()
        local cameraCFrame = Workspace.CurrentCamera and Workspace.CurrentCamera.CFrame or CFrame.new()
        VirtualUser:Button2Down(Vector2.new(0, 0), cameraCFrame)
        task.wait(0.04)
        VirtualUser:Button2Up(Vector2.new(0, 0), cameraCFrame)
    end)
end
function AntiAFK.start()
    if AntiAFK.loopStarted then return end
    AntiAFK.loopStarted = true
    task.spawn(function()
        while state.running do
            task.wait(600)
            if state.running and Settings.antiAfk then AntiAFK.pulse() end
        end
    end)
    connect(player.Idled, function()
        if Settings.antiAfk then AntiAFK.pulse() end
    end)
end

do
-- Avatar do criador: primeiro consulta a aparência pública atual de @armazenatudo.
-- A câmera enquadra APENAS as partes do corpo (acessórios muito grandes não afastam
-- o enquadramento). Se a montagem 3D falhar, mantém a imagem oficial do perfil.
local avatarModel, avatarBasePivot
local avatarVersion = 0
local avatarStarted = false
local avatarAngle = 0
local avatarUserId

local function clearAboutAvatar()
    if avatarModel then
        avatarModel:Destroy()
        avatarModel = nil
    end
    avatarBasePivot = nil
    avatarAngle = 0
    avatarViewport.Visible = false
    avatarThumbnail.ImageTransparency = 0
end

local AVATAR_BODY_PARTS = {
    Head = true, Torso = true, UpperTorso = true, LowerTorso = true,
    LeftUpperArm = true, LeftLowerArm = true, LeftHand = true,
    RightUpperArm = true, RightLowerArm = true, RightHand = true,
    LeftUpperLeg = true, LeftLowerLeg = true, LeftFoot = true,
    RightUpperLeg = true, RightLowerLeg = true, RightFoot = true,
    ["Left Arm"] = true, ["Right Arm"] = true,
    ["Left Leg"] = true, ["Right Leg"] = true,
}

local function avatarBodyBounds(model)
    local low, high
    for _, part in ipairs(model:GetChildren()) do
        if part:IsA("BasePart") and AVATAR_BODY_PARTS[part.Name] then
            local half = part.Size * 0.5
            local bottom, top = part.Position - half, part.Position + half
            low = low and Vector3.new(math.min(low.X, bottom.X),
                math.min(low.Y, bottom.Y), math.min(low.Z, bottom.Z)) or bottom
            high = high and Vector3.new(math.max(high.X, top.X),
                math.max(high.Y, top.Y), math.max(high.Z, top.Z)) or top
        end
    end
    if low and high then
        return (low + high) * 0.5, high - low
    end
    -- Rigs incomuns: usa primeiro a cabeça/raiz, sem enquadrar acessórios enormes.
    local root = model:FindFirstChild("HumanoidRootPart") or model:FindFirstChild("Head")
    if root and root:IsA("BasePart") then
        return root.Position, Vector3.new(4, 6, 4)
    end
    return nil, nil
end

local function prepareAboutModel(model)
    -- Scripts, UI e efeitos não pertencem ao retrato 3D.
    for _, node in ipairs(model:GetDescendants()) do
        if node:IsA("Script") or node:IsA("LocalScript") or node:IsA("ModuleScript")
            or node:IsA("BillboardGui") or node:IsA("SurfaceGui")
            or node:IsA("Highlight") then
            node:Destroy()
        elseif node:IsA("BasePart") then
            node.CanCollide = false
            node.CanTouch = false
            node.CanQuery = false
        end
    end
    local center, bodySize = avatarBodyBounds(model)
    if not center then error("Avatar não contém peças de personagem") end

    -- O WorldModel permite que os encaixes dos acessórios sejam montados.
    model.Parent = avatarWorld
    RunService.Heartbeat:Wait()
    if not state.running or model.Parent ~= avatarWorld then return false end
    for _, node in ipairs(model:GetDescendants()) do
        if node:IsA("BasePart") then node.Anchored = true end
    end

    -- Posição baseada no corpo, não no BoundingBox de asas, auras ou acessórios.
    center, bodySize = avatarBodyBounds(model)
    if not center then error("Avatar sem corpo visível") end
    local originalPivot = model:GetPivot()
    local shift = -center
    model:PivotTo(CFrame.new(originalPivot.Position + shift) * originalPivot.Rotation)
    avatarBasePivot = model:GetPivot()
    local focus, size = avatarBodyBounds(model)
    if not focus then error("Não foi possível enquadrar o avatar") end

    -- Ajusta a distância ao quadro vertical, mantendo o corpo grande e centralizado.
    local height = math.clamp(size.Y, 3, 8)
    local width = math.clamp(math.max(size.X, size.Z), 2, 7)
    local fov = math.rad(avatarCamera.FieldOfView)
    local distance = math.clamp(math.max(height * 0.64 / math.tan(fov * 0.5),
        width * 1.8), 6.2, 14)
    avatarCamera.CFrame = CFrame.lookAt(focus + Vector3.new(0, height * 0.025, -distance), focus)
    avatarViewport.CurrentCamera = avatarCamera
    avatarModel = model
    avatarAngle = 0
    avatarViewport.Visible = true
    -- O retrato oficial permanece discretamente no fundo, como fallback visual.
    avatarThumbnail.ImageTransparency = 0.72
    return true
end

local function cloneCreatorCharacter()
    -- Somente como alternativa caso o serviço de avatar público falhe.
    if string.lower(player.Name) ~= string.lower(CREATOR_USERNAME) then return nil end
    local character = player.Character
    if not character then return nil end
    local originalArchivable = character.Archivable
    character.Archivable = true
    local ok, clone = pcall(function() return character:Clone() end)
    character.Archivable = originalArchivable
    if ok then return clone end
end

local function loadAboutAvatar(force)
    if not state.running then return end
    if avatarStarted and not force then return end
    avatarStarted = true
    avatarVersion = avatarVersion + 1
    local requestVersion = avatarVersion
    avatarRefresh.Text = "Carregando..."
    avatarStatus.Text = "Buscando avatar atualizado..."
    avatarStatus.Visible = true
    if force then clearAboutAvatar() end
    task.spawn(function()
        local userId = avatarUserId
        if not userId then
            local ok, id = pcall(function()
                if string.lower(player.Name) == string.lower(CREATOR_USERNAME) then
                    return player.UserId
                end
                return Players:GetUserIdFromNameAsync(CREATOR_USERNAME)
            end)
            if ok and type(id) == "number" and id > 0 then
                userId, avatarUserId = id, id
            end
        end
        if not state.running or requestVersion ~= avatarVersion then return end
        if not userId then
            avatarStatus.Text = "Não foi possível localizar o perfil Roblox."
            avatarRefresh.Text = "Tentar novamente"
            return
        end

        local thumbnailUri = string.format("rbxthumb://type=Avatar&id=%d&w=420&h=420", userId)
        avatarThumbnail.Image = thumbnailUri
        task.spawn(function()
            local ok, url = pcall(function()
                return Players:GetUserThumbnailAsync(userId,
                    Enum.ThumbnailType.AvatarThumbnail, Enum.ThumbnailSize.Size420x420)
            end)
            if ok and type(url) == "string" and url ~= "" and state.running
                and requestVersion == avatarVersion then
                avatarThumbnail.Image = url
            end
        end)

        -- Obtém a aparência mais recente. Quando um método não é permitido no
        -- cliente, tenta o outro e depois o personagem local do próprio criador.
        local ok, model = pcall(function()
            local description = Players:GetHumanoidDescriptionFromUserIdAsync(userId)
            return Players:CreateHumanoidModelFromDescriptionAsync(description,
                Enum.HumanoidRigType.R15)
        end)
        if not ok or not model or not model:IsA("Model") then
            ok, model = pcall(function()
                return Players:CreateHumanoidModelFromUserIdAsync(userId)
            end)
        end
        if not ok or not model or not model:IsA("Model") then
            model = cloneCreatorCharacter()
        end
        if not state.running or requestVersion ~= avatarVersion then
            if model then model:Destroy() end
            return
        end
        if not model or not model:IsA("Model") then
            avatarStatus.Visible = false
            avatarViewport.Visible = false
            avatarThumbnail.ImageTransparency = 0
            avatarRefresh.Text = "Atualizar avatar"
            return
        end

        local success, err = pcall(function()
            clearAboutAvatar()
            if prepareAboutModel(model) == false then
                error("Visualização 3D cancelada")
            end
        end)
        if not state.running or requestVersion ~= avatarVersion then
            if model and model.Parent then model:Destroy() end
            return
        end
        if success then
            avatarStatus.Visible = false
        else
            warn("Leo Hub / avatar: " .. tostring(err))
            if model and model.Parent then model:Destroy() end
            avatarViewport.Visible = false
            avatarThumbnail.ImageTransparency = 0
            avatarStatus.Visible = false -- Mostra a imagem oficial em vez do quadro vazio.
        end
        avatarRefresh.Text = "Atualizar avatar"
    end)
end

local avatarRenderAccumulator = 0
connect(RunService.RenderStepped, function(dt)
    if not state.running or state.page ~= "about" or state.minimized
        or not pages.about.Visible or not avatarModel or not avatarBasePivot then
        avatarRenderAccumulator = 0
        return
    end
    -- O retrato gira a 20 FPS; evita PivotTo pesado em todos os frames do jogo.
    avatarRenderAccumulator = avatarRenderAccumulator + math.min(dt, 0.08)
    if avatarRenderAccumulator < 0.05 then return end
    avatarAngle = (avatarAngle + avatarRenderAccumulator * math.rad(13)) % (math.pi * 2)
    avatarRenderAccumulator = 0
    avatarModel:PivotTo(avatarBasePivot * CFrame.Angles(0, avatarAngle, 0))
end)
connect(avatarRefresh.Activated, function() loadAboutAvatar(true) end)


-- Exporta apenas os controles necessários; os detalhes permanecem locais ao bloco.
state.loadAboutAvatar = loadAboutAvatar
state.closeAboutAvatar = function()
    avatarVersion = avatarVersion + 1
    clearAboutAvatar()
end
end

-- 10. CONTROLES, ARRASTE E ENCERRAMENTO -----------------------------------------

local function cleanup()
    if not state.running then return end
    state.running = false
    if state.floatingGui and state.floatingGui.Parent then
        state.floatingGui:Destroy()
    end
    if state.closeAboutAvatar then state.closeAboutAvatar() end
    Engine.cleanup()
    state.base = nil
    state.baseObject = nil
    state.basePoint = nil
    table.clear(state.baseContainers)
    onBaseChanged = function() end
    releasePrompt()
    for _, connection in ipairs(state.connections) do connection:Disconnect() end
    table.clear(state.connections)
    table.clear(state.linkedGuis)
    table.clear(state.resources)
    table.clear(resourceKeys)
    table.clear(state.cards)
    table.clear(state.rowsByObject)
    table.clear(state.snapshotRows)
end
connect(gui.Destroying, cleanup)
connect(close.Activated, function() gui:Destroy() end)

connect(goBack.Activated, function()
    if state.busy then notify("Aguarde a interação atual terminar.", THEME.Warning) return end
    if not requireBase() then return end
    local moved = teleport(state.base)
    notify(moved and "Retorno à base solicitado." or "Personagem indisponível.",
        moved and THEME.Success or THEME.Warning)
end)
local pageVersion = 0
function showPage(name)
    if name == state.page and pages[name].Visible then return end
    closePicker()
    pageVersion = pageVersion + 1
    local thisVersion = pageVersion
    local previous = pages[state.page]
    state.page = name
    local entering = pages[name]
    if previous and previous ~= entering and previous.Visible then
        tween(previous, QUICK, { GroupTransparency = 1,
            Position = UDim2.fromOffset(82, 12) })
        task.delay(0.17, function()
            if state.running and pageVersion == thisVersion then previous.Visible = false end
        end)
    end
    entering.Visible = true
    entering.Position = UDim2.fromOffset(105, 12)
    entering.GroupTransparency = 1
    tween(entering, PAGE_TWEEN, {
        Position = UDim2.fromOffset(92, 12), GroupTransparency = 0,
    })
    for pageName, item in pairs({ home = homeNav, settings = settingsNav, about = aboutNav }) do
        local selected = pageName == name
        item:SetAttribute("ButtonAccent", selected and true or false)
        tween(item, QUICK, { BackgroundColor3 = selected and THEME.Accent or THEME.Input })
        for _, child in ipairs(item:GetDescendants()) do
            if child:IsA("TextLabel") then
                tween(child, QUICK, { TextColor3 = Color3.new(1, 1, 1) })
            elseif child:IsA("Frame") and not child:GetAttribute("LeoIconPiece") then
                tween(child, QUICK, { BackgroundColor3 = selected and THEME.Background or THEME.Text })
            elseif child:IsA("UIStroke") and child:GetAttribute("ButtonStroke") then
                tween(child, QUICK, { Color = selected and THEME.Highlight or THEME.Border })
            end
        end
    end
    task.delay(0.26, function()
        if not state.running or pageVersion ~= thisVersion then return end
        for pageName, page in pairs(pages) do
            page.Visible = pageName == name
            if pageName ~= name then page.GroupTransparency = 1 end
        end
    end)
    updateVisiblePreviews()
end
connect(homeNav.Activated, function() showPage("home") end)
connect(settingsNav.Activated, function() showPage("settings") end)
connect(aboutNav.Activated, function() showPage("about"); if state.loadAboutAvatar then state.loadAboutAvatar(false) end end)

connect(petSelect.Activated, function()
    local options = {}
    for id, entry in pairs(Catalog.entries) do
        table.insert(options, { text = entry.name, searchText = entry.name,
            value = id, sample = entry.sample, rawName = entry.name })
    end
    table.sort(options, function(a, b) return normalize(a.text) < normalize(b.text) end)
    openMultiPicker(petSelect, "Selecionar pets", options, Auto.pets, Auto.changed, true)
end)
connect(sizeSelect.Activated, function()
    local options = {}
    for _, size in ipairs(SIZES) do
        table.insert(options, { text = size[2], searchText = size[2], value = size[2] })
    end
    openMultiPicker(sizeSelect, "Selecionar tamanhos", options, Auto.sizes, Auto.changed, false)
end)
connect(autoToggle.Activated, function()
    if Auto.enabled then Auto.stop("Coleta desativada") else Auto.start() end
end)
connect(sort.Activated, function()
    local options = {}
    for index, mode in ipairs(SORT_MODES) do table.insert(options, { text = mode.text, value = index }) end
    openPicker(sort, options, state.sortMode, function(value, text)
        state.sortMode = value
        sort.Text = text
        scroll.CanvasPosition = Vector2.zero
        renderList()
    end)
end)
connect(themeSelect.Activated, function()
    local options = {
        { text = "Roxo", value = "roxo" },
        { text = "Vermelho", value = "vermelho" },
        { text = "Amarelo", value = "amarelo" },
        { text = "Azul", value = "azul" },
    }
    openPicker(themeSelect, options, Settings.theme, function(value, text)
        themeSelect.Text = "Tema: " .. text
        applyTheme(value)
    end)
end)
connect(antiAfkToggle.Activated, function()
    Settings.antiAfk = not Settings.antiAfk
    styleToggle(antiAfkToggle, Settings.antiAfk, "Ativado", "Desativado")
end)
-- ESP removido da interface

styleToggle(antiAfkToggle, Settings.antiAfk, "Ativado", "Desativado")
applyTheme(Settings.theme)
AntiAFK.start()
showPage("home")


local searchVersion = 0
connect(search:GetPropertyChangedSignal("Text"), function()
    searchVersion = searchVersion + 1
    local version = searchVersion
    task.delay(0.12, function()
        if state.running and version == searchVersion then
            scroll.CanvasPosition = Vector2.zero
            renderList()
        end
    end)
end)
connect(scroll:GetPropertyChangedSignal("CanvasPosition"), updateVisiblePreviews)

-- Logo minimizada independente: o botão recebe todos os cliques e toques.
-- Mantido em uma função/bloco para não ultrapassar o limite de variáveis do Luau.
do
    local viewportScale = 1
    local animating = false
    local compactMode = false
    local homePanel = "coleta"
    local desktopW, desktopH = CONFIG.Width, CONFIG.Height
    -- No celular, o conteúdo da página inicial é separado em duas visualizações.
    -- Isso evita encolher o painel de 920px inteiro, mantendo controles legíveis.
    local mobileSwitch = button(home, "VER PETS  ›", UDim2.new(1, -122, 0, 0),
        UDim2.fromOffset(122, 30), true)
    mobileSwitch.Visible = false
    mobileSwitch.ZIndex = 20
    mobileSwitch.TextSize = 13
    local function showMobileHome(which)
        homePanel = which
        if not compactMode then
            left.Visible, right.Visible = true, true
            return
        end
        left.Visible = which == "coleta"
        right.Visible = which == "pets"
        mobileSwitch.Text = which == "coleta" and "VER PETS  ›" or "‹  VER COLETA"
        if which == "pets" then task.defer(updateVisiblePreviews) end
    end
    connect(mobileSwitch.Activated, function()
        showMobileHome(homePanel == "coleta" and "pets" or "coleta")
    end)
    local function responsiveLayout(compact)
        if compact == compactMode then return end
        compactMode = compact
        closePicker(true)
        if compact then
            CONFIG.Width, CONFIG.Height = 540, 540
            mobileSwitch.Visible = true
            left.Size = UDim2.new(1, 0, 1, 0)
            right.Position = UDim2.fromOffset(0, 36)
            right.Size = UDim2.new(1, 0, 1, -36)
            filters.Size = UDim2.new(1, 0, 0, 238)
            basePanel.Position = UDim2.fromOffset(0, 308)
            basePanel.Size = UDim2.new(1, 0, 0, 100)
            -- No perfil, avatar e contato ficam um abaixo do outro.
            aboutPortrait.Position = UDim2.fromOffset(12, 12)
            aboutPortrait.Size = UDim2.new(1, -24, 0, 162)
            portraitFooter.Position = UDim2.new(0, 0, 1, -45)
            portraitFooter.Size = UDim2.new(1, 0, 0, 45)
            avatarThumbnail.Size = UDim2.new(1, -20, 1, -54)
            avatarViewport.Size = UDim2.new(1, -20, 1, -54)
            avatarStatus.Position = UDim2.fromOffset(12, 48)
            avatarStatus.Size = UDim2.new(1, -24, 0, 48)
            creatorCaption.Position = UDim2.fromOffset(12, 0)
            creatorCaption.Size = UDim2.new(1, -24, 0, 26)
            creatorCaption.TextSize = 16
            creatorSub.Position = UDim2.fromOffset(12, 24)
            creatorSub.Size = UDim2.new(1, -24, 0, 16)
            creatorSub.TextSize = 10
            creatorInfo.Position = UDim2.fromOffset(12, 184)
            creatorInfo.Size = UDim2.new(1, -24, 1, -196)
            infoAccent.Position = UDim2.fromOffset(14, 12)
            infoAccent.Size = UDim2.fromOffset(4, 40)
            infoTag.Position = UDim2.fromOffset(28, 8)
            infoTag.Size = UDim2.new(1, -167, 0, 16)
            infoName.Position = UDim2.fromOffset(28, 28)
            infoName.Size = UDim2.new(1, -50, 0, 27)
            infoName.TextSize = 18
            avatarRefresh.Position = UDim2.new(1, -137, 0, 10)
            avatarRefresh.Size = UDim2.fromOffset(125, 30)
            contact.Position = UDim2.fromOffset(14, 66)
            contact.Size = UDim2.new(1, -28, 0, 59)
            discordTitle.Position = UDim2.fromOffset(14, 6)
            discordNick.Position = UDim2.fromOffset(14, 25)
            -- Botões para toque: ampliar apenas a tipografia, sem ampliar o painel.
            showMobileHome(homePanel)
        else
            CONFIG.Width, CONFIG.Height = desktopW, desktopH
            mobileSwitch.Visible = false
            left.Size = UDim2.new(0, 236, 1, 0)
            right.Position = UDim2.fromOffset(252, 0)
            right.Size = UDim2.new(1, -252, 1, 0)
            filters.Size = UDim2.new(1, 0, 0, 254)
            basePanel.Position = UDim2.fromOffset(0, 326)
            basePanel.Size = UDim2.new(1, 0, 0, 108)
            aboutPortrait.Position = UDim2.fromOffset(12, 12)
            aboutPortrait.Size = UDim2.new(0, 282, 1, -24)
            portraitFooter.Position = UDim2.new(0, 0, 1, -80)
            portraitFooter.Size = UDim2.new(1, 0, 0, 80)
            avatarThumbnail.Size = UDim2.new(1, -20, 1, -93)
            avatarViewport.Size = UDim2.new(1, -20, 1, -93)
            avatarStatus.Position = UDim2.fromOffset(18, 104)
            avatarStatus.Size = UDim2.new(1, -36, 0, 64)
            creatorCaption.Position = UDim2.fromOffset(12, 7)
            creatorCaption.Size = UDim2.new(1, -24, 0, 32)
            creatorCaption.TextSize = 19
            creatorSub.Position = UDim2.fromOffset(12, 41)
            creatorSub.Size = UDim2.new(1, -24, 0, 23)
            creatorSub.TextSize = 10
            creatorInfo.Position = UDim2.fromOffset(306, 12)
            creatorInfo.Size = UDim2.new(1, -318, 1, -24)
            infoAccent.Position = UDim2.fromOffset(21, 25)
            infoAccent.Size = UDim2.fromOffset(5, 40)
            infoTag.Position = UDim2.fromOffset(40, 20)
            infoTag.Size = UDim2.new(1, -200, 0, 19)
            infoName.Position = UDim2.fromOffset(38, 42)
            infoName.Size = UDim2.new(1, -58, 0, 36)
            infoName.TextSize = 22
            avatarRefresh.Position = UDim2.new(1, -161, 0, 17)
            avatarRefresh.Size = UDim2.fromOffset(143, 34)
            contact.Position = UDim2.fromOffset(20, 94)
            contact.Size = UDim2.new(1, -40, 0, 73)
            discordTitle.Position = UDim2.fromOffset(20, 12)
            discordNick.Position = UDim2.fromOffset(20, 32)
            showMobileHome(homePanel)
        end
        main.Size = UDim2.fromOffset(CONFIG.Width, CONFIG.Height)
        -- O painel volta ao centro após girar a tela ou alterar a resolução.
        main.Position = UDim2.fromScale(0.5, 0.5)
        task.defer(updateVisiblePreviews)
    end
    local lastViewport = nil
    local function fitScreen()
        local camera = Workspace.CurrentCamera
        if not camera then return end
        local viewport = camera.ViewportSize
        if viewport.X < 1 or viewport.Y < 1 then return end
        if lastViewport and viewport == lastViewport then return end
        local compact = viewport.X < 680 or (UIS.TouchEnabled and math.min(viewport.X, viewport.Y) < 680)
        responsiveLayout(compact)
        viewportScale = math.max(0.15, math.min(1,
            (viewport.X - 20) / CONFIG.Width,
            (viewport.Y - 28) / CONFIG.Height))
        if not animating then uiScale.Scale = viewportScale end
        -- Reduz o ícone apenas quando estiver fora da área visível por rotação.
        if lastViewport and (viewport - lastViewport).Magnitude > 15 then
            if state.minimized and state.floatingRoot and state.floatingRoot.Parent then
                state.floatingRoot.Position = UDim2.fromScale(0.5, 0.5)
            end
        end
        lastViewport = viewport
    end
    state.fitHubScreen = fitScreen
    fitScreen()
    -- Atualiza imediatamente na rotação/redimensionamento, sem polling custoso.
    state.watchHubCamera = function()
        if state.cameraViewportConnection then state.cameraViewportConnection:Disconnect() end
        local camera = Workspace.CurrentCamera
        state.cameraViewportConnection = camera
            and connect(camera:GetPropertyChangedSignal("ViewportSize"), fitScreen) or nil
        lastViewport = nil
        fitScreen()
    end
    connect(Workspace:GetPropertyChangedSignal("CurrentCamera"), state.watchHubCamera)
    state.watchHubCamera()

    local ok, err = pcall(function()
        -- ScreenGui fica ativo, mas o botão inicia oculto. Isso impede que a
        -- minimização dependa de um ScreenGui inteiro desabilitado.
        local bubbleGui = create("ScreenGui", {
            Name = "LeoHubFloating", ResetOnSpawn = false,
            DisplayOrder = 100, ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
            IgnoreGuiInset = false, Enabled = true,
        }, playerGui)
        state.floatingGui = bubbleGui
        local bubble = create("TextButton", {
            Name = "LeoHubAbrir", AnchorPoint = Vector2.new(0.5, 0.5),
            Position = UDim2.fromScale(0.5, 0.5),
            Size = UDim2.fromOffset(66, 66),
            BackgroundColor3 = THEME.Rail,
            BackgroundTransparency = 0.06,
            BorderSizePixel = 0, AutoButtonColor = false,
            Text = "LH", TextSize = 20,
            Font = Enum.Font.FredokaOne,
            TextColor3 = Color3.new(1, 1, 1),
            Visible = false, Active = true, ZIndex = 10,
        }, bubbleGui)
        round(bubble, 17)
        local bubbleStroke = create("UIStroke", {
            Color = FLOATING_LOGO_BLUE_BORDER, Thickness = 2.2,
            Transparency = 0.06, ApplyStrokeMode = Enum.ApplyStrokeMode.Border,
        }, bubble)
        local bubbleScale = create("UIScale", { Scale = 1 }, bubble)
        -- A imagem fica DENTRO do botão, não sobre outro botão invisível.
        local picture = create("ImageLabel", {
            Name = "Logo", BackgroundTransparency = 1,
            Position = UDim2.fromOffset(3, 3),
            Size = UDim2.new(1, -6, 1, -6),
            Image = "", ScaleType = Enum.ScaleType.Fit,
            ZIndex = 11, Active = false,
        }, bubble)
        local activeInput, firstInput, firstPosition
        local dragged = false
        local logoScale = 1
        local bubbleDiameter = 66
        local zoomTween = TweenInfo.new(0.28, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
        local smoothTween = TweenInfo.new(0.18, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)

        local function viewSize()
            local camera = Workspace.CurrentCamera
            return camera and camera.ViewportSize or Vector2.new(1280, 720)
        end
        local function clampPosition(position)
            local size = viewSize()
            local half = bubbleDiameter * logoScale / 2 + 8
            local x = position.X.Scale * size.X + position.X.Offset
            local y = position.Y.Scale * size.Y + position.Y.Offset
            return UDim2.fromOffset(
                math.clamp(x, half, math.max(half, size.X - half)),
                math.clamp(y, half, math.max(half, size.Y - half)))
        end
        local function refreshLogo()
            if not state.running or not bubble.Parent then return end
            local image = state.brandLogo and state.brandLogo.Image or ""
            picture.Image = image
            -- O botão LH continua visível se o asset ainda não estiver pronto.
            local loaded = false
            if image ~= "" then
                pcall(function() loaded = picture.IsLoaded end)
            end
            bubble.Text = loaded and "" or "LH"
            bubbleStroke.Color = FLOATING_LOGO_BLUE_BORDER
        end
        local function restore()
            if not state.running or not state.minimized then return end
            state.minimized = false
            closePicker(true)
            main.Visible = true
            main.Size = UDim2.fromOffset(CONFIG.Width, CONFIG.Height)
            body.Visible = true
            bubble.Visible = false
            animating = true
            fitScreen()
            uiScale.Scale = math.max(0.08, viewportScale * 0.86)
            local background = main.BackgroundTransparency
            main.BackgroundTransparency = math.min(0.65, background + 0.18)
            tween(uiScale, zoomTween, { Scale = viewportScale })
            tween(main, smoothTween, { BackgroundTransparency = background })
            task.delay(0.34, function()
                if not state.running or state.minimized then return end
                animating = false
                uiScale.Scale = viewportScale
                main.BackgroundTransparency = background
            end)
            updateVisiblePreviews()
        end
        local function minimizePanel()
            if not state.running or state.minimized then return end
            closePicker(true)
            fitScreen()
            refreshLogo()
            bubble.Position = clampPosition(main.Position)
            bubbleStroke.Color = FLOATING_LOGO_BLUE_BORDER
            logoScale = math.clamp(viewportScale, 0.86, 1)
            bubbleScale.Scale = logoScale
            bubble.Visible = true -- revela ANTES de esconder o painel
            state.minimized = true
            main.Visible = false
            bubbleScale.Scale = logoScale * 0.9
            tween(bubbleScale, zoomTween, { Scale = logoScale })
            updateVisiblePreviews()
            -- Atualiza a imagem assim que o Roblox concluir o carregamento.
            task.spawn(function()
                for _ = 1, 14 do
                    if not state.running or not state.minimized then break end
                    if picture.Image ~= "" and picture.IsLoaded then
                        bubble.Text = ""
                        break
                    end
                    task.wait(0.35)
                end
            end)
        end
        state.setMinimized = function(mini)
            if mini then minimizePanel() else restore() end
        end
        connect(minimize.Activated, minimizePanel)

        -- Drag controlado pela entrada do usuário: clique abre, arraste move.
        connect(bubble.InputBegan, function(input)
            if not state.running or not state.minimized or activeInput then return end
            if input.UserInputType ~= Enum.UserInputType.MouseButton1
                and input.UserInputType ~= Enum.UserInputType.Touch then return end
            activeInput = input
            firstInput = input.Position
            firstPosition = bubble.Position
            dragged = false
        end)
        connect(UIS.InputChanged, function(input)
            if not activeInput or not state.minimized then return end
            local isMouse = activeInput.UserInputType == Enum.UserInputType.MouseButton1
                and input.UserInputType == Enum.UserInputType.MouseMovement
            if input ~= activeInput and not isMouse then return end
            local delta = input.Position - firstInput
            if delta.Magnitude > 7 then dragged = true end
            if dragged then
                bubble.Position = clampPosition(UDim2.new(
                    firstPosition.X.Scale, firstPosition.X.Offset + delta.X,
                    firstPosition.Y.Scale, firstPosition.Y.Offset + delta.Y))
            end
        end)
        connect(UIS.InputEnded, function(input)
            if not activeInput then return end
            if input ~= activeInput and not (
                activeInput.UserInputType == Enum.UserInputType.MouseButton1
                and input.UserInputType == Enum.UserInputType.MouseButton1) then return end
            activeInput = nil
        end)
        connect(bubble.Activated, function()
            if not state.running or not state.minimized then return end
            if dragged then return end
            restore()
        end)
        -- Recuperação no PC mesmo caso a logo seja ocultada por outro elemento.
        connect(UIS.InputBegan, function(input, processed)
            if state.running and state.minimized and input.KeyCode == Enum.KeyCode.RightControl then
                restore()
            end
        end)
        -- Um botão criado sempre permanece acessível, mesmo sem PNG publicado.
        state.floatingRoot = bubble
    end)
    if not ok then
        warn("[LEO HUB] Ícone flutuante indisponível: " .. tostring(err))
        -- Fallback seguro: mantém o hub aberto, sem esconder a UI se a criação falhar.
        if state.floatingGui and state.floatingGui.Parent then
            state.floatingGui:Destroy()
            state.floatingGui = nil
        end
        connect(minimize.Activated, function()
            if not state.running then return end
            main.Visible = true
            body.Visible = true
            notify("Não foi possível minimizar. Verifique o Output (F9).", THEME.Warning)
        end)
    end
end

local dragInput, dragStart, startPosition
connect(header.InputBegan, function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
        dragStart = input.Position
        startPosition = main.Position
    end
end)
connect(UIS.InputChanged, function(input)
    if not dragInput then return end
    if input == dragInput or (dragInput.UserInputType == Enum.UserInputType.MouseButton1
        and input.UserInputType == Enum.UserInputType.MouseMovement) then
        local delta = input.Position - dragStart
        main.Position = UDim2.new(startPosition.X.Scale, startPosition.X.Offset + delta.X,
            startPosition.Y.Scale, startPosition.Y.Offset + delta.Y)
    end
end)
connect(UIS.InputEnded, function(input)
    if input == dragInput then dragInput = nil end
end)

state.fitHubScreen()
renderList()
Engine.start()
refreshBase()

-- Manutenção leve da janela e da base; NÃO atualiza ou varre a lista de pets.
task.spawn(function()
    while state.running and gui.Parent do
        task.wait(CONFIG.MaintenanceSeconds)
        if not state.running then break end
        state.fitHubScreen()
        local ok, failure = pcall(refreshBase)
        if not ok then warn("Leo Hub / base: " .. tostring(failure)) end
        statLabels[1].Text = tostring(#Players:GetPlayers())
        local pingOk, ping = pcall(function() return player:GetNetworkPing() end)
        statLabels[2].Text = pingOk and typeof(ping) == "number" and ping >= 0 and ping < math.huge
            and string.format("%d ms", math.floor(ping * 1000 + 0.5)) or "—"
    end
end)
