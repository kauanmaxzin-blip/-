-- ╔══════════════════════════════════════════════════════════════╗
-- ║      PAINEL TRICK · SISTEMA DE KEY ONLINE + PREMIUM UI      ║
-- ║      Keys validadas no servidor  |  Delta Executor          ║
-- ╚══════════════════════════════════════════════════════════════╝

-- ► COLOQUE A URL DO SEU SERVIDOR
local SERVER_URL  = "https://key-system2-3.onrender.com"
local LINK_DISCORD = "https://discord.gg/seulink"
local ARQUIVO_KEY  = "pt_key_salva.txt"

local _CoreGui    = game:GetService("CoreGui")
local _Players    = game:GetService("Players")
local _TweenSvc   = game:GetService("TweenService")

-- ══════════════════════════════════════════════════════════════
--  HTTP HELPER
-- ══════════════════════════════════════════════════════════════
local function HttpPost(endpoint, bodyTable)
    local url  = SERVER_URL .. endpoint
    local body = game:GetService("HttpService"):JSONEncode(bodyTable)
    local headers = {["Content-Type"] = "application/json"}
    local ok, response = pcall(function()
        if syn and syn.request then
            return syn.request({Url=url,Method="POST",Headers=headers,Body=body})
        elseif http_request then
            return http_request({Url=url,Method="POST",Headers=headers,Body=body})
        elseif request then
            return request({Url=url,Method="POST",Headers=headers,Body=body})
        elseif fluxus and fluxus.request then
            return fluxus.request({Url=url,Method="POST",Headers=headers,Body=body})
        else error("Sem suporte HTTP!") end
    end)
    if not ok then return nil, "Sem conexão!" end
    local parsed
    pcall(function()
        parsed = game:GetService("HttpService"):JSONDecode(response.Body)
    end)
    return parsed, nil
end

-- ══════════════════════════════════════════════════════════════
--  VERIFICAÇÃO ONLINE
-- ══════════════════════════════════════════════════════════════
local _userId = tostring(game:GetService("Players").LocalPlayer.UserId)

local function SalvarKey(key)
    pcall(function() writefile(ARQUIVO_KEY, key) end)
end

local function LerKeySalva()
    local k = ""
    pcall(function()
        if isfile(ARQUIVO_KEY) then k = readfile(ARQUIVO_KEY) end
    end)
    return string.gsub(k, "%s+", "")
end

local function VerificarKeyOnline(key)
    local data, err = HttpPost("/validate", {key=key, userId=_userId})
    if not data then return false, "❌ " .. (err or "Erro de conexão!") end
    if data.valid then
        local expSec = math.floor((data.expiresAt or 0) / 1000)
        return true, data.message or "Key válida!", expSec
    else
        return false, data.message or "Key inválida!"
    end
end

-- ══════════════════════════════════════════════════════════════
--  SCRIPT PRINCIPAL (carregado após validar key)
-- ══════════════════════════════════════════════════════════════
local function CarregarScriptPrincipal()
    print("✅ Painel Trick carregado e autenticado!")

    local TweenService = game:GetService("TweenService")
    local Players = game:GetService("Players")
    local CoreGui = game:GetService("CoreGui")
    local UserInputService = game:GetService("UserInputService")
    local RunService = game:GetService("RunService")
    local Lighting = game:GetService("Lighting")

    local LocalPlayer = Players.LocalPlayer
    local Camera = workspace.CurrentCamera

    local OptionsState = {
        ["AIMBOT"] = false, 
        ["HOLOGRAMA"] = false, 
        ["ESQUELETO"] = false,
        ["GIRA 360"] = false, 
        ["MAGNITUDE"] = false, 
        ["LINHAS"] = false, 
        ["FAST SHOOT"] = false,
        ["OTIMIZAR FPS"] = false,
        ["SILENT AIM"] = false
    }
    
    local IsActive = false
    local AimbotMode = "ATIRAR"
    local SilentAimWallbang = false
    local CurrentPage = 1

    local Theme = {
        Background = Color3.fromRGB(15, 15, 15),
        Surface = Color3.fromRGB(22, 22, 22),
        Accent = Color3.fromRGB(214, 10, 20),
        AccentDark = Color3.fromRGB(150, 5, 10),
        Text = Color3.fromRGB(255, 255, 255),
        TextDim = Color3.fromRGB(180, 180, 180),
        CheckboxBg = Color3.fromRGB(35, 35, 35)
    }

    local TweenInfos = {
        Hover = TweenInfo.new(0.2, Enum.EasingStyle.Sine, Enum.EasingDirection.Out),
        Click = TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.InOut, 0, true),
        Open  = TweenInfo.new(0.5, Enum.EasingStyle.Quint, Enum.EasingDirection.Out),
        Toggle = TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    }

    local existingGui = CoreGui:FindFirstChild("PainelTrickUI") or LocalPlayer:WaitForChild("PlayerGui"):FindFirstChild("PainelTrickUI")
    if existingGui then existingGui:Destroy() end

    local function AnimateButton(button, isCheckbox)
        local uiScale = Instance.new("UIScale")
        uiScale.Parent = button
        button.MouseEnter:Connect(function()
            if not isCheckbox then TweenService:Create(button, TweenInfos.Hover, {BackgroundColor3 = Theme.AccentDark}):Play() end
        end)
        button.MouseLeave:Connect(function()
            if not isCheckbox then TweenService:Create(button, TweenInfos.Hover, {BackgroundColor3 = Theme.Accent}):Play() end
        end)
        button.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                TweenService:Create(uiScale, TweenInfos.Click, {Scale = 0.92}):Play()
            end
        end)
    end

    -- ==========================================
    -- CONSTRUÇÃO DA UI PRINCIPAL
    -- ==========================================
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "PainelTrickUI"
    ScreenGui.ResetOnSpawn = false
    ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    local success = pcall(function() ScreenGui.Parent = (gethui and gethui()) or CoreGui end)
    if not success then ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui") end

    local MainFrame = Instance.new("Frame")
    MainFrame.Name = "Main"
    MainFrame.Size = UDim2.new(0, 300, 0, 300) -- TAMANHO FIXO CORRIGIDO (300x300)
    MainFrame.Position = UDim2.new(0.5, -150, 0.5, -150)
    MainFrame.BackgroundColor3 = Theme.Background
    MainFrame.BackgroundTransparency = 0.05
    MainFrame.BorderSizePixel = 0
    MainFrame.Parent = ScreenGui
    MainFrame.ClipsDescendants = false
    Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)
    local MainStroke = Instance.new("UIStroke", MainFrame)
    MainStroke.Color = Color3.fromRGB(40, 40, 40)
    MainStroke.Thickness = 1
    local MainScale = Instance.new("UIScale", MainFrame)
    MainScale.Scale = 0
    TweenService:Create(MainScale, TweenInfos.Open, {Scale = 1}):Play()

    -- Drag System
    local dragging, dragInput, dragStart, startPos
    local function updateDrag(input)
        local delta = input.Position - dragStart
        TweenService:Create(MainFrame, TweenInfo.new(0.1), {Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)}):Play()
    end
    MainFrame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true dragStart = input.Position startPos = MainFrame.Position
            input.Changed:Connect(function() if input.UserInputState == Enum.UserInputState.End then dragging = false end end)
        end
    end)
    MainFrame.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then dragInput = input end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then updateDrag(input) end
    end)

    -- Topo e Header
    local TopBar = Instance.new("Frame")
    TopBar.Size = UDim2.new(1, 0, 0, 40)
    TopBar.BackgroundTransparency = 1
    TopBar.Parent = MainFrame

    local Title = Instance.new("TextLabel")
    Title.Size = UDim2.new(0, 200, 1, 0)
    Title.Position = UDim2.new(0, 15, 0, 0)
    Title.BackgroundTransparency = 1
    Title.Text = 'Painel <font color="#d60a14">Trick</font>'
    Title.RichText = true
    Title.Font = Enum.Font.GothamBold
    Title.TextSize = 18
    Title.TextColor3 = Theme.Text
    Title.TextXAlignment = Enum.TextXAlignment.Left
    Title.Parent = TopBar

    local CloseBtn = Instance.new("TextButton")
    CloseBtn.Size = UDim2.new(0, 40, 0, 40)
    CloseBtn.Position = UDim2.new(1, -40, 0, 0)
    CloseBtn.BackgroundTransparency = 1
    CloseBtn.Text = "X"
    CloseBtn.Font = Enum.Font.GothamBold
    CloseBtn.TextSize = 16
    CloseBtn.TextColor3 = Theme.TextDim
    CloseBtn.Parent = TopBar

    local MinBtn = Instance.new("TextButton")
    MinBtn.Size = UDim2.new(0, 40, 0, 40)
    MinBtn.Position = UDim2.new(1, -80, 0, 0)
    MinBtn.BackgroundTransparency = 1
    MinBtn.Text = "—"
    MinBtn.Font = Enum.Font.GothamBold
    MinBtn.TextSize = 14
    MinBtn.TextColor3 = Theme.TextDim
    MinBtn.Parent = TopBar

    local HeaderArea = Instance.new("Frame")
    HeaderArea.Size = UDim2.new(1, -30, 0, 40)
    HeaderArea.Position = UDim2.new(0, 15, 0, 40)
    HeaderArea.BackgroundTransparency = 1
    HeaderArea.ZIndex = 5
    HeaderArea.Parent = MainFrame

    local ProfileIcon = Instance.new("ImageLabel")
    ProfileIcon.Size = UDim2.new(0, 35, 0, 35)
    ProfileIcon.Position = UDim2.new(0, 0, 0.5, -17)
    ProfileIcon.BackgroundColor3 = Theme.Surface
    ProfileIcon.Image = "rbxthumb://type=AvatarHeadShot&id=" .. LocalPlayer.UserId .. "&w=150&h=150"
    ProfileIcon.Parent = HeaderArea
    Instance.new("UICorner", ProfileIcon).CornerRadius = UDim.new(1, 0)

    local UserText = Instance.new("TextLabel")
    UserText.Size = UDim2.new(0, 80, 1, 0)
    UserText.Position = UDim2.new(0, 45, 0, 0)
    UserText.BackgroundTransparency = 1
    UserText.Text = "USER: " .. string.sub(LocalPlayer.Name, 1, 10)
    UserText.Font = Enum.Font.GothamSemibold
    UserText.TextSize = 12
    UserText.TextColor3 = Theme.Text
    UserText.TextXAlignment = Enum.TextXAlignment.Left
    UserText.Parent = HeaderArea

    -- ==========================================
    -- SISTEMA DE PÁGINAS (DROPDOWN MENU)
    -- ==========================================
    local Dropdown = Instance.new("TextButton")
    Dropdown.Size = UDim2.new(0, 110, 0, 26)
    Dropdown.Position = UDim2.new(1, -110, 0.5, -13)
    Dropdown.BackgroundColor3 = Theme.Background
    Dropdown.Text = "  PARTE 1"
    Dropdown.Font = Enum.Font.GothamSemibold
    Dropdown.TextSize = 11
    Dropdown.TextColor3 = Theme.Text
    Dropdown.TextXAlignment = Enum.TextXAlignment.Left
    Dropdown.AutoButtonColor = false
    Dropdown.ZIndex = 6
    Dropdown.Parent = HeaderArea
    Instance.new("UICorner", Dropdown).CornerRadius = UDim.new(0, 6)
    Instance.new("UIStroke", Dropdown).Color = Color3.fromRGB(50, 50, 50)

    local DropArrow = Instance.new("TextLabel")
    DropArrow.Size = UDim2.new(0, 20, 1, 0)
    DropArrow.Position = UDim2.new(1, -20, 0, 0)
    DropArrow.BackgroundTransparency = 1
    DropArrow.Text = "v"
    DropArrow.Font = Enum.Font.GothamBold
    DropArrow.TextSize = 12
    DropArrow.TextColor3 = Theme.TextDim
    DropArrow.ZIndex = 6
    DropArrow.Parent = Dropdown

    -- Lista do Dropdown
    local DropMenu = Instance.new("Frame")
    DropMenu.Size = UDim2.new(0, 110, 0, 52)
    DropMenu.Position = UDim2.new(1, -110, 0.5, 15)
    DropMenu.BackgroundColor3 = Theme.Surface
    DropMenu.Visible = false
    DropMenu.ZIndex = 10
    DropMenu.Parent = HeaderArea
    Instance.new("UICorner", DropMenu).CornerRadius = UDim.new(0, 6)
    Instance.new("UIStroke", DropMenu).Color = Color3.fromRGB(80, 80, 80)

    local DropLayout = Instance.new("UIListLayout")
    DropLayout.SortOrder = Enum.SortOrder.LayoutOrder
    DropLayout.Parent = DropMenu

    local BtnPagina1 = Instance.new("TextButton")
    BtnPagina1.Size = UDim2.new(1, 0, 0, 26)
    BtnPagina1.BackgroundTransparency = 1
    BtnPagina1.Text = "  PARTE 1"
    BtnPagina1.Font = Enum.Font.GothamSemibold
    BtnPagina1.TextSize = 11
    BtnPagina1.TextColor3 = Theme.Text
    BtnPagina1.TextXAlignment = Enum.TextXAlignment.Left
    BtnPagina1.ZIndex = 10
    BtnPagina1.LayoutOrder = 1
    BtnPagina1.Parent = DropMenu

    local BtnPagina2 = Instance.new("TextButton")
    BtnPagina2.Size = UDim2.new(1, 0, 0, 26)
    BtnPagina2.BackgroundTransparency = 1
    BtnPagina2.Text = "  PARTE 2"
    BtnPagina2.Font = Enum.Font.GothamSemibold
    BtnPagina2.TextSize = 11
    BtnPagina2.TextColor3 = Theme.Text
    BtnPagina2.TextXAlignment = Enum.TextXAlignment.Left
    BtnPagina2.ZIndex = 10
    BtnPagina2.LayoutOrder = 2
    BtnPagina2.Parent = DropMenu


    -- ==========================================
    -- SISTEMAS / HACKS (FUNÇÕES DE OTIMIZACAO E FAST SHOOT)
    -- ==========================================

    local OriginalValues = {}
    local OriginalEnvs = {}
    local FastShootLoop = nil

    local function ApplyFastShoot()
        if FastShootLoop then FastShootLoop.Disconnect() FastShootLoop = nil end
        local isFastEnabled = (IsActive and OptionsState["FAST SHOOT"])
        
        if not isFastEnabled then
            for obj, orig in pairs(OriginalValues) do
                pcall(function() if obj and obj.Parent then obj.Value = orig end end)
            end
            OriginalValues = {}
            for scriptObj, envData in pairs(OriginalEnvs) do
                pcall(function()
                    if scriptObj and scriptObj.Parent then
                        local env = (getsenv and getsenv(scriptObj)) or getfenv(scriptObj)
                        for key, orig in pairs(envData) do env[key] = orig end
                    end
                end)
            end
            OriginalEnvs = {}
            return
        end

        local function PatchToolValues(tool)
            for _, v in ipairs(tool:GetDescendants()) do
                if v:IsA("NumberValue") or v:IsA("IntValue") then
                    local n = v.Name:lower()
                    if n == "firerate" or n == "fire_rate" or n == "shootcooldown" or n == "cooldown"
                    or n == "delay" or n == "waittime" or n == "rpm" or n == "firecooldown" then
                        if OriginalValues[v] == nil then OriginalValues[v] = v.Value end
                        local orig = OriginalValues[v]
                        if n == "rpm" then pcall(function() v.Value = orig * 5 end)
                        else pcall(function() v.Value = math.max(0.01, orig / 5) end) end
                    end
                end
                if v:IsA("LocalScript") or v:IsA("Script") then
                    pcall(function()
                        local env = (getsenv and getsenv(v)) or getfenv(v)
                        local names = {"FireRate","fireRate","fire_rate","Cooldown","cooldown","ShootCooldown","shootCooldown","Delay","delay","WaitTime","waitTime"}
                        for _, key in ipairs(names) do
                            if env[key] ~= nil and type(env[key]) == "number" then
                                if OriginalEnvs[v] == nil then OriginalEnvs[v] = {} end
                                if OriginalEnvs[v][key] == nil then OriginalEnvs[v][key] = env[key] end
                                env[key] = math.max(0.01, OriginalEnvs[v][key] / 5)
                            end
                        end
                    end)
                end
            end
        end

        local running = true
        FastShootLoop = { Disconnect = function() running = false end }
        task.spawn(function()
            while running and (IsActive and OptionsState["FAST SHOOT"]) do
                local char = LocalPlayer.Character
                if char then
                    for _, tool in ipairs(char:GetChildren()) do
                        if tool:IsA("Tool") then
                            PatchToolValues(tool)
                            for _, v in ipairs(tool:GetDescendants()) do
                                if v:IsA("LocalScript") or v:IsA("Script") then
                                    pcall(function()
                                        local env = (getsenv and getsenv(v)) or getfenv(v)
                                        if env.canShoot    == false then env.canShoot    = true  end
                                        if env.CanShoot    == false then env.CanShoot    = true  end
                                        if env.canFire     == false then env.canFire     = true  end
                                        if env.CanFire     == false then env.CanFire     = true  end
                                        if env.isShooting  == true  then env.isShooting  = false end
                                        if env.shooting    == true  then env.shooting    = false end
                                        if env.isReloading == true  then env.isReloading = false end
                                    end)
                                end
                            end
                        end
                    end
                end
                task.wait(0)
            end
        end)
    end

    local optimized = false
    local OriginalOptProps = {}
    local OptConnection = nil

    local function ApplyOptimization()
        local isOptEnabled = (IsActive and OptionsState["OTIMIZAR FPS"])

        if isOptEnabled then
            if optimized then return end 
            optimized = true

            OriginalOptProps["_Lighting"] = {
                GlobalShadows = Lighting.GlobalShadows,
                FogEnd = Lighting.FogEnd,
                Brightness = Lighting.Brightness
            }
            OriginalOptProps["_Terrain"] = {
                WaterWaveSize = workspace.Terrain.WaterWaveSize,
                WaterWaveSpeed = workspace.Terrain.WaterWaveSpeed,
                WaterReflectance = workspace.Terrain.WaterReflectance,
                WaterTransparency = workspace.Terrain.WaterTransparency
            }

            if setfpscap then pcall(function() setfpscap(120) end) end

            local function OptimizePart(v)
                if not v or not v.Parent then return end
                if v:IsA("BasePart") and not v:IsA("MeshPart") then
                    if not OriginalOptProps[v] then OriginalOptProps[v] = {Material = v.Material, Reflectance = v.Reflectance} end
                    v.Material = Enum.Material.SmoothPlastic
                    v.Reflectance = 0
                elseif v:IsA("Decal") or v:IsA("Texture") then
                    if not OriginalOptProps[v] then OriginalOptProps[v] = {Transparency = v.Transparency} end
                    v.Transparency = 1
                elseif v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Sparkles") or v:IsA("Fire") or v:IsA("Smoke") then
                    if not OriginalOptProps[v] then OriginalOptProps[v] = {Enabled = v.Enabled} end
                    v.Enabled = false
                elseif v:IsA("PostEffect") or v:IsA("BlurEffect") or v:IsA("SunRaysEffect") or v:IsA("BloomEffect") or v:IsA("ColorCorrectionEffect") then
                    if not OriginalOptProps[v] then OriginalOptProps[v] = {Enabled = v.Enabled} end
                    v.Enabled = false
                end
            end

            task.spawn(function()
                local count = 0
                for _, v in pairs(workspace:GetDescendants()) do
                    pcall(OptimizePart, v)
                    count = count + 1
                    if count % 1000 == 0 then task.wait() end
                end
            end)

            OptConnection = workspace.DescendantAdded:Connect(function(v)
                if optimized then
                    task.spawn(function()
                        task.wait(0.1)
                        pcall(OptimizePart, v)
                    end)
                end
            end)

            pcall(function()
                Lighting.GlobalShadows = false
                Lighting.FogEnd = 9e9
                Lighting.Brightness = 2
                workspace.Terrain.WaterWaveSize = 0
                workspace.Terrain.WaterWaveSpeed = 0
                workspace.Terrain.WaterReflectance = 0
                workspace.Terrain.WaterTransparency = 0
                settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
            end)
        else
            if not optimized then return end
            optimized = false

            if OptConnection then 
                OptConnection:Disconnect() 
                OptConnection = nil 
            end
            if setfpscap then pcall(function() setfpscap(60) end) end

            for obj, props in pairs(OriginalOptProps) do
                if typeof(obj) == "Instance" and obj.Parent then
                    pcall(function()
                        for propName, propVal in pairs(props) do
                            obj[propName] = propVal
                        end
                    end)
                end
            end

            pcall(function()
                if OriginalOptProps["_Lighting"] then
                    Lighting.GlobalShadows = OriginalOptProps["_Lighting"].GlobalShadows
                    Lighting.FogEnd = OriginalOptProps["_Lighting"].FogEnd
                    Lighting.Brightness = OriginalOptProps["_Lighting"].Brightness
                end
                if OriginalOptProps["_Terrain"] then
                    workspace.Terrain.WaterWaveSize = OriginalOptProps["_Terrain"].WaterWaveSize
                    workspace.Terrain.WaterWaveSpeed = OriginalOptProps["_Terrain"].WaterWaveSpeed
                    workspace.Terrain.WaterReflectance = OriginalOptProps["_Terrain"].WaterReflectance
                    workspace.Terrain.WaterTransparency = OriginalOptProps["_Terrain"].WaterTransparency
                end
                settings().Rendering.QualityLevel = Enum.QualityLevel.Automatic
            end)

            OriginalOptProps = {}
        end
    end

    -- ==========================================
    -- CONTAINERS DAS PÁGINAS (PARTE 1 E PARTE 2)
    -- ==========================================
    local Page1Container = Instance.new("Frame")
    Page1Container.Size = UDim2.new(1, -30, 0, 115) 
    Page1Container.Position = UDim2.new(0, 15, 0, 85)
    Page1Container.BackgroundTransparency = 1
    Page1Container.Visible = true 
    Page1Container.Parent = MainFrame

    local GridLayout1 = Instance.new("UIGridLayout")
    GridLayout1.CellSize = UDim2.new(0, 130, 0, 24)
    GridLayout1.CellPadding = UDim2.new(0, 10, 0, 6)
    GridLayout1.FillDirection = Enum.FillDirection.Horizontal
    GridLayout1.SortOrder = Enum.SortOrder.LayoutOrder
    GridLayout1.Parent = Page1Container

    local Page2Container = Instance.new("Frame")
    Page2Container.Size = UDim2.new(1, -30, 0, 115) 
    Page2Container.Position = UDim2.new(0, 15, 0, 85)
    Page2Container.BackgroundTransparency = 1
    Page2Container.Visible = false 
    Page2Container.Parent = MainFrame

    local GridLayout2 = Instance.new("UIGridLayout")
    GridLayout2.CellSize = UDim2.new(0, 130, 0, 24)
    GridLayout2.CellPadding = UDim2.new(0, 10, 0, 6)
    GridLayout2.FillDirection = Enum.FillDirection.Horizontal
    GridLayout2.SortOrder = Enum.SortOrder.LayoutOrder
    GridLayout2.Parent = Page2Container

    -- SUBMENUS
    local AimbotSubMenu = Instance.new("Frame")
    AimbotSubMenu.Size = UDim2.new(1, -30, 0, 25)
    AimbotSubMenu.Position = UDim2.new(0, 15, 0, 205)
    AimbotSubMenu.BackgroundTransparency = 1
    AimbotSubMenu.Visible = false
    AimbotSubMenu.Parent = MainFrame

    local SilentAimSubMenu = Instance.new("Frame")
    SilentAimSubMenu.Size = UDim2.new(1, -30, 0, 25)
    SilentAimSubMenu.Position = UDim2.new(0, 15, 0, 205)
    SilentAimSubMenu.BackgroundTransparency = 1
    SilentAimSubMenu.Visible = false
    SilentAimSubMenu.Parent = MainFrame

    local function CreateRadioButton(name, posX, isDefault, parentFrame)
        local Btn = Instance.new("TextButton")
        Btn.Size = UDim2.new(0, 120, 1, 0)
        Btn.Position = UDim2.new(0, posX, 0, 0)
        Btn.BackgroundTransparency = 1
        Btn.Text = ""
        Btn.Parent = parentFrame
        local Circle = Instance.new("Frame")
        Circle.Size = UDim2.new(0, 16, 0, 16)
        Circle.Position = UDim2.new(0, 0, 0.5, -8)
        Circle.BackgroundColor3 = Theme.CheckboxBg
        Circle.Parent = Btn
        Instance.new("UICorner", Circle).CornerRadius = UDim.new(1, 0)
        Instance.new("UIStroke", Circle).Color = Theme.AccentDark
        local InnerCircle = Instance.new("Frame")
        InnerCircle.Size = UDim2.new(0, 8, 0, 8)
        InnerCircle.Position = UDim2.new(0.5, -4, 0.5, -4)
        InnerCircle.BackgroundColor3 = Theme.Accent
        InnerCircle.BackgroundTransparency = isDefault and 0 or 1
        InnerCircle.Parent = Circle
        Instance.new("UICorner", InnerCircle).CornerRadius = UDim.new(1, 0)
        local Label = Instance.new("TextLabel")
        Label.Size = UDim2.new(1, -22, 1, 0)
        Label.Position = UDim2.new(0, 22, 0, 0)
        Label.BackgroundTransparency = 1
        Label.Text = name
        Label.Font = Enum.Font.GothamSemibold
        Label.TextSize = 10
        Label.TextColor3 = Theme.TextDim
        Label.TextXAlignment = Enum.TextXAlignment.Left
        Label.Parent = Btn
        AnimateButton(Btn, true)
        return Btn, InnerCircle
    end

    -- Radio Buttons do Aimbot
    local BtnAoOlhar, CircleAoOlhar = CreateRadioButton("AO OLHAR", 0, false, AimbotSubMenu)
    local BtnAoAtirar, CircleAoAtirar = CreateRadioButton("AO ATIRAR", 140, true, AimbotSubMenu)

    BtnAoOlhar.MouseButton1Click:Connect(function()
        AimbotMode = "OLHAR"
        TweenService:Create(CircleAoOlhar, TweenInfos.Toggle, {BackgroundTransparency = 0}):Play()
        TweenService:Create(CircleAoAtirar, TweenInfos.Toggle, {BackgroundTransparency = 1}):Play()
    end)
    BtnAoAtirar.MouseButton1Click:Connect(function()
        AimbotMode = "ATIRAR"
        TweenService:Create(CircleAoAtirar, TweenInfos.Toggle, {BackgroundTransparency = 0}):Play()
        TweenService:Create(CircleAoOlhar, TweenInfos.Toggle, {BackgroundTransparency = 1}):Play()
    end)

    -- Radio Buttons do Silent Aim (Wallbang)
    local BtnNaoAtravessa, CircleNaoAtravessa = CreateRadioButton("NÃO ATRAVESSA", 0, true, SilentAimSubMenu)
    local BtnAtravessa, CircleAtravessa = CreateRadioButton("ATRAVESSA", 140, false, SilentAimSubMenu)

    BtnNaoAtravessa.MouseButton1Click:Connect(function()
        SilentAimWallbang = false
        TweenService:Create(CircleNaoAtravessa, TweenInfos.Toggle, {BackgroundTransparency = 0}):Play()
        TweenService:Create(CircleAtravessa, TweenInfos.Toggle, {BackgroundTransparency = 1}):Play()
    end)
    BtnAtravessa.MouseButton1Click:Connect(function()
        SilentAimWallbang = true
        TweenService:Create(CircleAtravessa, TweenInfos.Toggle, {BackgroundTransparency = 0}):Play()
        TweenService:Create(CircleNaoAtravessa, TweenInfos.Toggle, {BackgroundTransparency = 1}):Play()
    end)


    -- LÓGICA DO DROPDOWN (MUDAR DE PÁGINA)
    Dropdown.MouseButton1Click:Connect(function()
        DropMenu.Visible = not DropMenu.Visible
    end)

    local function TrocarPagina(numPagina)
        CurrentPage = numPagina
        DropMenu.Visible = false
        if numPagina == 1 then
            Dropdown.Text = "  PARTE 1"
            Page1Container.Visible = true
            Page2Container.Visible = false
            if OptionsState["AIMBOT"] then AimbotSubMenu.Visible = true end
            SilentAimSubMenu.Visible = false
        elseif numPagina == 2 then
            Dropdown.Text = "  PARTE 2"
            Page1Container.Visible = false
            Page2Container.Visible = true
            AimbotSubMenu.Visible = false
            if OptionsState["SILENT AIM"] then SilentAimSubMenu.Visible = true end
        end
    end

    BtnPagina1.MouseButton1Click:Connect(function() TrocarPagina(1) end)
    BtnPagina2.MouseButton1Click:Connect(function() TrocarPagina(2) end)

    -- ADICIONANDO AS OPÇÕES NA PARTE 1
    local OptionsPage1 = {"AIMBOT", "HOLOGRAMA", "ESQUELETO", "GIRA 360", "MAGNITUDE", "LINHAS", "FAST SHOOT", "OTIMIZAR FPS"}
    for i, optName in ipairs(OptionsPage1) do
        local ToggleBtn2 = Instance.new("TextButton")
        ToggleBtn2.Size = UDim2.new(1, 0, 1, 0)
        ToggleBtn2.BackgroundTransparency = 1
        ToggleBtn2.Text = ""
        ToggleBtn2.LayoutOrder = i
        ToggleBtn2.Parent = Page1Container
        
        local Box = Instance.new("Frame")
        Box.Size = UDim2.new(0, 20, 0, 20)
        Box.Position = UDim2.new(0, 0, 0.5, -10)
        Box.BackgroundColor3 = Theme.CheckboxBg
        Box.Parent = ToggleBtn2
        Instance.new("UICorner", Box).CornerRadius = UDim.new(0, 4)
        
        local CheckMark = Instance.new("TextLabel")
        CheckMark.Size = UDim2.new(1, 0, 1, 0)
        CheckMark.BackgroundTransparency = 1
        CheckMark.Text = "✓"
        CheckMark.Font = Enum.Font.GothamBold
        CheckMark.TextSize = 12
        CheckMark.TextColor3 = Theme.Accent
        CheckMark.TextTransparency = 1
        CheckMark.Parent = Box
        
        local Label2 = Instance.new("TextLabel")
        Label2.Size = UDim2.new(1, -26, 1, 0)
        Label2.Position = UDim2.new(0, 26, 0, 0)
        Label2.BackgroundTransparency = 1
        Label2.Text = optName
        Label2.Font = Enum.Font.GothamSemibold
        Label2.TextSize = 11
        Label2.TextColor3 = Theme.Text
        Label2.TextXAlignment = Enum.TextXAlignment.Left
        Label2.Parent = ToggleBtn2
        
        local isToggled = false
        ToggleBtn2.MouseButton1Click:Connect(function()
            isToggled = not isToggled
            OptionsState[optName] = isToggled
            
            if optName == "AIMBOT" and CurrentPage == 1 then AimbotSubMenu.Visible = isToggled end
            
            if isToggled then
                TweenService:Create(CheckMark, TweenInfos.Toggle, {TextTransparency = 0}):Play()
                TweenService:Create(Box, TweenInfos.Toggle, {BackgroundColor3 = Color3.fromRGB(50, 20, 20)}):Play()
            else
                TweenService:Create(CheckMark, TweenInfos.Toggle, {TextTransparency = 1}):Play()
                TweenService:Create(Box, TweenInfos.Toggle, {BackgroundColor3 = Theme.CheckboxBg}):Play()
            end

            if IsActive then
                if optName == "FAST SHOOT" then ApplyFastShoot()
                elseif optName == "OTIMIZAR FPS" then ApplyOptimization() end
            end
        end)
        AnimateButton(ToggleBtn2, true)
    end

    -- ADICIONANDO AS OPÇÕES NA PARTE 2
    local OptionsPage2 = {"SILENT AIM"}
    for i, optName in ipairs(OptionsPage2) do
        local ToggleBtn2 = Instance.new("TextButton")
        ToggleBtn2.Size = UDim2.new(1, 0, 1, 0)
        ToggleBtn2.BackgroundTransparency = 1
        ToggleBtn2.Text = ""
        ToggleBtn2.LayoutOrder = i
        ToggleBtn2.Parent = Page2Container
        
        local Box = Instance.new("Frame")
        Box.Size = UDim2.new(0, 20, 0, 20)
        Box.Position = UDim2.new(0, 0, 0.5, -10)
        Box.BackgroundColor3 = Theme.CheckboxBg
        Box.Parent = ToggleBtn2
        Instance.new("UICorner", Box).CornerRadius = UDim.new(0, 4)
        
        local CheckMark = Instance.new("TextLabel")
        CheckMark.Size = UDim2.new(1, 0, 1, 0)
        CheckMark.BackgroundTransparency = 1
        CheckMark.Text = "✓"
        CheckMark.Font = Enum.Font.GothamBold
        CheckMark.TextSize = 12
        CheckMark.TextColor3 = Theme.Accent
        CheckMark.TextTransparency = 1
        CheckMark.Parent = Box
        
        local Label2 = Instance.new("TextLabel")
        Label2.Size = UDim2.new(1, -26, 1, 0)
        Label2.Position = UDim2.new(0, 26, 0, 0)
        Label2.BackgroundTransparency = 1
        Label2.Text = optName
        Label2.Font = Enum.Font.GothamSemibold
        Label2.TextSize = 11
        Label2.TextColor3 = Theme.Text
        Label2.TextXAlignment = Enum.TextXAlignment.Left
        Label2.Parent = ToggleBtn2
        
        local isToggled = false
        ToggleBtn2.MouseButton1Click:Connect(function()
            isToggled = not isToggled
            OptionsState[optName] = isToggled
            
            if optName == "SILENT AIM" and CurrentPage == 2 then 
                SilentAimSubMenu.Visible = isToggled 
            end
            
            if isToggled then
                TweenService:Create(CheckMark, TweenInfos.Toggle, {TextTransparency = 0}):Play()
                TweenService:Create(Box, TweenInfos.Toggle, {BackgroundColor3 = Color3.fromRGB(50, 20, 20)}):Play()
            else
                TweenService:Create(CheckMark, TweenInfos.Toggle, {TextTransparency = 1}):Play()
                TweenService:Create(Box, TweenInfos.Toggle, {BackgroundColor3 = Theme.CheckboxBg}):Play()
            end
        end)
        AnimateButton(ToggleBtn2, true)
    end


    -- BOTÕES DE AÇÃO
    local BottomArea = Instance.new("Frame")
    BottomArea.Size = UDim2.new(1, -30, 0, 40)
    BottomArea.Position = UDim2.new(0, 15, 1, -55)
    BottomArea.BackgroundTransparency = 1
    BottomArea.Parent = MainFrame

    local BtnAtivar = Instance.new("TextButton")
    BtnAtivar.Size = UDim2.new(0.48, 0, 1, 0)
    BtnAtivar.Position = UDim2.new(0, 0, 0, 0)
    BtnAtivar.BackgroundColor3 = Theme.Accent
    BtnAtivar.Text = "ATIVAR"
    BtnAtivar.Font = Enum.Font.GothamBold
    BtnAtivar.TextSize = 13
    BtnAtivar.TextColor3 = Theme.Text
    BtnAtivar.AutoButtonColor = false
    BtnAtivar.Parent = BottomArea
    Instance.new("UICorner", BtnAtivar).CornerRadius = UDim.new(0, 8)
    AnimateButton(BtnAtivar, false)
    
    BtnAtivar.MouseButton1Click:Connect(function() 
        IsActive = true 
        ApplyFastShoot()
        ApplyOptimization()
    end)

    local BtnDesativar = Instance.new("TextButton")
    BtnDesativar.Size = UDim2.new(0.48, 0, 1, 0)
    BtnDesativar.Position = UDim2.new(0.52, 0, 0, 0)
    BtnDesativar.BackgroundColor3 = Theme.AccentDark
    BtnDesativar.Text = "DESATIVAR"
    BtnDesativar.Font = Enum.Font.GothamBold
    BtnDesativar.TextSize = 13
    BtnDesativar.TextColor3 = Theme.Text
    BtnDesativar.AutoButtonColor = false
    BtnDesativar.Parent = BottomArea
    Instance.new("UICorner", BtnDesativar).CornerRadius = UDim.new(0, 8)
    AnimateButton(BtnDesativar, false)
    
    BtnDesativar.MouseButton1Click:Connect(function() 
        IsActive = false 
        ApplyFastShoot() 
        ApplyOptimization()
    end)

    CloseBtn.MouseButton1Click:Connect(function()
        local closeTween = TweenService:Create(MainScale, TweenInfos.Open, {Scale = 0})
        closeTween:Play() closeTween.Completed:Wait() ScreenGui:Destroy()
    end)

    local minimized = false
    MinBtn.MouseButton1Click:Connect(function()
        minimized = not minimized
        if minimized then
            HeaderArea.Visible = false 
            Page1Container.Visible = false 
            Page2Container.Visible = false
            DropMenu.Visible = false
            AimbotSubMenu.Visible = false 
            SilentAimSubMenu.Visible = false
            BottomArea.Visible = false
            TweenService:Create(MainFrame, TweenInfos.Open, {Size = UDim2.new(0, 300, 0, 40)}):Play()
        else
            HeaderArea.Visible = true 
            BottomArea.Visible = true
            TweenService:Create(MainFrame, TweenInfos.Open, {Size = UDim2.new(0, 300, 0, 300)}):Play()
            
            if CurrentPage == 1 then
                Page1Container.Visible = true
                if OptionsState["AIMBOT"] then AimbotSubMenu.Visible = true end
            else
                Page2Container.Visible = true
                if OptionsState["SILENT AIM"] then SilentAimSubMenu.Visible = true end
            end
        end
    end)


    -- ==========================================
    -- SISTEMA DE TOGGLE VISIBILIDADE (4 DEDOS OU SHIFT)
    -- ==========================================
    local activeTouches = {}
    local fourFingerTriggered = false
    local isPanelVisible = true

    UserInputService.InputBegan:Connect(function(input, gpe)
        if input.UserInputType == Enum.UserInputType.Touch then
            activeTouches[input] = true
            local count = 0
            for _ in pairs(activeTouches) do count = count + 1 end
            if count >= 4 and not fourFingerTriggered then
                fourFingerTriggered = true
                isPanelVisible = not isPanelVisible
                ScreenGui.Enabled = isPanelVisible
            end
        end

        if not gpe and input.KeyCode == Enum.KeyCode.RightShift then
            isPanelVisible = not isPanelVisible
            ScreenGui.Enabled = isPanelVisible
        end
    end)

    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.Touch then
            activeTouches[input] = nil
            local count = 0
            for _ in pairs(activeTouches) do count = count + 1 end
            if count < 4 then
                fourFingerTriggered = false
            end
        end
    end)


    -- ==========================================
    -- CORE DOS HACKS (AIMBOT, ESP E CROSSHAIR)
    -- ==========================================
    local isShooting = false
    local activeShootTouches = {}
    UserInputService.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            isShooting = true
        elseif input.UserInputType == Enum.UserInputType.Touch then
            local viewport = Camera.ViewportSize
            local isTouchingShootButton = (input.Position.X > viewport.X * 0.65) and (input.Position.Y > viewport.Y * 0.55)
            if isTouchingShootButton then activeShootTouches[input] = true isShooting = true end
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            isShooting = false
        elseif input.UserInputType == Enum.UserInputType.Touch then
            if activeShootTouches[input] then
                activeShootTouches[input] = nil
                local stillShooting = false
                for _ in pairs(activeShootTouches) do stillShooting = true break end
                isShooting = stillShooting
            end
        end
    end)

    local function GetClosestTargetToCrosshair()
        local closestPart, shortestDistance = nil, math.huge
        local localChar = LocalPlayer.Character
        if not localChar or not localChar:FindFirstChild("HumanoidRootPart") then return nil end
        local centerScreen = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
        
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
                local targetChar = player.Character
                local targetPart = targetChar:FindFirstChild("Head") or targetChar:FindFirstChild("UpperTorso") or targetChar:FindFirstChild("HumanoidRootPart")
                if targetPart then
                    local pos, onScreen = Camera:WorldToViewportPoint(targetPart.Position)
                    if onScreen then
                        local distanceToCenter = (Vector2.new(pos.X, pos.Y) - centerScreen).Magnitude
                        if distanceToCenter < shortestDistance then
                            
                            local canSee = false
                            if SilentAimWallbang then
                                canSee = true
                            else
                                local rayParams = RaycastParams.new()
                                rayParams.FilterType = Enum.RaycastFilterType.Exclude
                                rayParams.FilterDescendantsInstances = {localChar, Camera}
                                rayParams.IgnoreWater = true
                                local rayResult = workspace:Raycast(Camera.CFrame.Position, (targetPart.Position - Camera.CFrame.Position), rayParams)
                                if not rayResult or (rayResult.Instance and rayResult.Instance:IsDescendantOf(targetChar)) then
                                    canSee = true
                                end
                            end

                            if canSee then
                                shortestDistance = distanceToCenter 
                                closestPart = targetPart
                            end

                        end
                    end
                end
            end
        end
        return closestPart
    end

    local Skeletons, Tracers = {}, {}
    local function GetSkeletonsLines(player)
        if not Skeletons[player] then
            Skeletons[player] = {}
            for i = 1, 15 do
                local line = Drawing.new("Line")
                line.Visible = false line.Color = Color3.fromRGB(255,255,255)
                line.Thickness = 1.5 line.Transparency = 1
                table.insert(Skeletons[player], line)
            end
        end
        return Skeletons[player]
    end
    
    local function GetTracerLine(player)
        if not Tracers[player] then
            local line = Drawing.new("Line")
            line.Visible = false line.Color = Theme.Accent
            line.Thickness = 1.5 line.Transparency = 1
            Tracers[player] = line
        end
        return Tracers[player]
    end
    
    local function DrawBone(line, part1, part2)
        if part1 and part2 then
            local pos1, vis1 = Camera:WorldToViewportPoint(part1.Position)
            local pos2, vis2 = Camera:WorldToViewportPoint(part2.Position)
            if vis1 and vis2 then
                line.From = Vector2.new(pos1.X, pos1.Y) line.To = Vector2.new(pos2.X, pos2.Y)
                line.Visible = true return
            end
        end
        line.Visible = false
    end

    local CurrentSilentTarget = nil

    RunService.RenderStepped:Connect(function()
        if IsActive and OptionsState["SILENT AIM"] then
            CurrentSilentTarget = GetClosestTargetToCrosshair()
        else
            CurrentSilentTarget = nil
        end

        local char = LocalPlayer.Character
        if IsActive and OptionsState["AIMBOT"] then
            local shouldAim = (AimbotMode == "OLHAR") or (AimbotMode == "ATIRAR" and isShooting)
            if shouldAim then
                local targetPart = GetClosestTargetToCrosshair()
                if targetPart then Camera.CFrame = CFrame.new(Camera.CFrame.Position, targetPart.Position) end
            end
        end
        
        if IsActive and OptionsState["MAGNITUDE"] and isShooting then
            local targetPart = GetClosestTargetToCrosshair()
            if targetPart and targetPart.Parent and targetPart.Parent:FindFirstChild("HumanoidRootPart") then
                local targetHRP = targetPart.Parent.HumanoidRootPart
                targetHRP.CFrame = Camera.CFrame * CFrame.new(0, 0, -5)
                targetHRP.AssemblyLinearVelocity = Vector3.new(0,0,0)
                targetHRP.AssemblyAngularVelocity = Vector3.new(0,0,0)
            end
        end
        
        if char and char:FindFirstChild("HumanoidRootPart") and char:FindFirstChild("Humanoid") then
            if IsActive and OptionsState["GIRA 360"] then
                char.Humanoid.AutoRotate = false
                char.HumanoidRootPart.CFrame = char.HumanoidRootPart.CFrame * CFrame.Angles(0, math.rad(45), 0)
            else char.Humanoid.AutoRotate = true end
        end
        
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer then
                local targetChar = player.Character
                if targetChar then
                    local highlight = targetChar:FindFirstChild("TrickHologram")
                    if IsActive and OptionsState["HOLOGRAMA"] then
                        if not highlight then
                            highlight = Instance.new("Highlight")
                            highlight.Name = "TrickHologram"
                            highlight.FillColor = Color3.fromRGB(0,255,255)
                            highlight.OutlineColor = Color3.fromRGB(255,255,255)
                            highlight.FillTransparency = 0.5
                            highlight.OutlineTransparency = 0.1
                            highlight.Parent = targetChar
                        end
                    else if highlight then highlight:Destroy() end end
                end
                
                local skelLines = GetSkeletonsLines(player)
                if IsActive and OptionsState["ESQUELETO"] and targetChar and targetChar:FindFirstChild("HumanoidRootPart") and targetChar:FindFirstChild("Humanoid") and targetChar.Humanoid.Health > 0 then
                    if targetChar:FindFirstChild("UpperTorso") then
                        DrawBone(skelLines[1], targetChar:FindFirstChild("Head"), targetChar:FindFirstChild("UpperTorso"))
                        DrawBone(skelLines[2], targetChar:FindFirstChild("UpperTorso"), targetChar:FindFirstChild("LowerTorso"))
                        DrawBone(skelLines[3], targetChar:FindFirstChild("UpperTorso"), targetChar:FindFirstChild("LeftUpperArm"))
                        DrawBone(skelLines[4], targetChar:FindFirstChild("LeftUpperArm"), targetChar:FindFirstChild("LeftLowerArm"))
                        DrawBone(skelLines[5], targetChar:FindFirstChild("LeftLowerArm"), targetChar:FindFirstChild("LeftHand"))
                        DrawBone(skelLines[6], targetChar:FindFirstChild("UpperTorso"), targetChar:FindFirstChild("RightUpperArm"))
                        DrawBone(skelLines[7], targetChar:FindFirstChild("RightUpperArm"), targetChar:FindFirstChild("RightLowerArm"))
                        DrawBone(skelLines[8], targetChar:FindFirstChild("RightLowerArm"), targetChar:FindFirstChild("RightHand"))
                        DrawBone(skelLines[9], targetChar:FindFirstChild("LowerTorso"), targetChar:FindFirstChild("LeftUpperLeg"))
                        DrawBone(skelLines[10], targetChar:FindFirstChild("LeftUpperLeg"), targetChar:FindFirstChild("LeftLowerLeg"))
                        DrawBone(skelLines[11], targetChar:FindFirstChild("LeftLowerLeg"), targetChar:FindFirstChild("LeftFoot"))
                        DrawBone(skelLines[12], targetChar:FindFirstChild("LowerTorso"), targetChar:FindFirstChild("RightUpperLeg"))
                        DrawBone(skelLines[13], targetChar:FindFirstChild("RightUpperLeg"), targetChar:FindFirstChild("RightLowerLeg"))
                        DrawBone(skelLines[14], targetChar:FindFirstChild("RightLowerLeg"), targetChar:FindFirstChild("RightFoot"))
                        skelLines[15].Visible = false
                    elseif targetChar:FindFirstChild("Torso") then
                        DrawBone(skelLines[1], targetChar:FindFirstChild("Head"), targetChar:FindFirstChild("Torso"))
                        DrawBone(skelLines[2], targetChar:FindFirstChild("Torso"), targetChar:FindFirstChild("Left Arm"))
                        DrawBone(skelLines[3], targetChar:FindFirstChild("Torso"), targetChar:FindFirstChild("Right Arm"))
                        DrawBone(skelLines[4], targetChar:FindFirstChild("Torso"), targetChar:FindFirstChild("Left Leg"))
                        DrawBone(skelLines[5], targetChar:FindFirstChild("Torso"), targetChar:FindFirstChild("Right Leg"))
                        for i = 6, 15 do skelLines[i].Visible = false end
                    end
                else for _, line in ipairs(skelLines) do line.Visible = false end end

                local tracerLine = GetTracerLine(player)
                if IsActive and OptionsState["LINHAS"] and targetChar and targetChar:FindFirstChild("HumanoidRootPart") and targetChar:FindFirstChild("Humanoid") and targetChar.Humanoid.Health > 0 then
                    local pos, onScreen = Camera:WorldToViewportPoint(targetChar.HumanoidRootPart.Position)
                    if onScreen then
                        tracerLine.From = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y)
                        tracerLine.To = Vector2.new(pos.X, pos.Y) tracerLine.Visible = true
                    else tracerLine.Visible = false end
                else tracerLine.Visible = false end
            end
        end
    end)

    Players.PlayerRemoving:Connect(function(player)
        if Skeletons[player] then
            for _, line in ipairs(Skeletons[player]) do line:Remove() end
            Skeletons[player] = nil
        end
        if Tracers[player] then Tracers[player]:Remove() Tracers[player] = nil end
    end)

    -- ==========================================
    -- SISTEMA DE BALA MÁGICA (SILENT AIM OTIMIZADO)
    -- ==========================================
    local oldNamecall
    local oldIndex

    pcall(function()
        local mouse = LocalPlayer:GetMouse()
        oldIndex = hookmetamethod(game, "__index", function(self, key)
            if not checkcaller() and IsActive and OptionsState["SILENT AIM"] then
                if self == mouse and (key == "Hit" or key == "Target") then
                    if CurrentSilentTarget then
                        if key == "Hit" then return CurrentSilentTarget.CFrame end
                        if key == "Target" then return CurrentSilentTarget end
                    end
                end
            end
            return oldIndex(self, key)
        end)
    end)

    pcall(function()
        oldNamecall = hookmetamethod(game, "__namecall", function(self, ...)
            local args = {...}
            local method = getnamecallmethod()

            if not checkcaller() and IsActive and OptionsState["SILENT AIM"] then
                if method == "FindPartOnRayWithIgnoreList" or method == "FindPartOnRayWithWhitelist" or method == "FindPartOnRay" or method == "Raycast" then
                    if CurrentSilentTarget then
                        if method == "Raycast" then
                            args[2] = (CurrentSilentTarget.Position - args[1]).Unit * 1000
                            return oldNamecall(self, unpack(args))
                        elseif method == "FindPartOnRayWithIgnoreList" or method == "FindPartOnRayWithWhitelist" or method == "FindPartOnRay" then
                            args[1] = Ray.new(args[1].Origin, (CurrentSilentTarget.Position - args[1].Origin).Unit * 1000)
                            return oldNamecall(self, unpack(args))
                        end
                    end
                end
            end
            return oldNamecall(self, ...)
        end)
    end)

end -- fim CarregarScriptPrincipal


-- ══════════════════════════════════════════════════════════════
--  GUI DE KEY (AUTENTICAÇÃO)
-- ══════════════════════════════════════════════════════════════
local function CriarSistemaKey()
    local keySalva = LerKeySalva()
    if #keySalva > 0 then
        local valido, msg, expTime = VerificarKeyOnline(keySalva)
        if valido then
            print("🔑 Key salva verificada. Entrando...")
            CarregarScriptPrincipal()
            return
        end
    end

    local KeyUI = Instance.new("ScreenGui")
    KeyUI.Name = "PT_KeySystem"
    KeyUI.ResetOnSpawn = false
    KeyUI.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    KeyUI.IgnoreGuiInset = true
    local parentGui = pcall(function() return _CoreGui end) and _CoreGui or _Players.LocalPlayer:WaitForChild("PlayerGui")
    KeyUI.Parent = parentGui

    local BlurBg = Instance.new("Frame", KeyUI)
    BlurBg.Size = UDim2.new(1, 0, 1, 0)
    BlurBg.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    BlurBg.BackgroundTransparency = 0.4
    BlurBg.Active = true

    local MainFrame2 = Instance.new("Frame", KeyUI)
    MainFrame2.Size = UDim2.new(0, 210, 0, 140)
    MainFrame2.Position = UDim2.new(0.5, -105, 0.5, -70)
    MainFrame2.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
    MainFrame2.BorderSizePixel = 0
    MainFrame2.ClipsDescendants = true
    local mCorner = Instance.new("UICorner", MainFrame2) mCorner.CornerRadius = UDim.new(0, 10)
    local mStroke = Instance.new("UIStroke", MainFrame2) mStroke.Color = Color3.fromRGB(214, 10, 20) mStroke.Thickness = 2

    local TopBar2 = Instance.new("Frame", MainFrame2)
    TopBar2.Size = UDim2.new(1, 0, 0, 4)
    TopBar2.BackgroundColor3 = Color3.fromRGB(214, 10, 20)
    TopBar2.BorderSizePixel = 0

    local Title2 = Instance.new("TextLabel", MainFrame2)
    Title2.Size = UDim2.new(1, 0, 0, 40)
    Title2.Position = UDim2.new(0, 0, 0, 6)
    Title2.BackgroundTransparency = 1
    Title2.Text = '⚡ Painel <font color="#d60a14">Trick</font>'
    Title2.RichText = true
    Title2.TextColor3 = Color3.fromRGB(255, 255, 255)
    Title2.TextSize = 13
    Title2.Font = Enum.Font.GothamBold

    local SubTitle2 = Instance.new("TextLabel", MainFrame2)
    SubTitle2.Size = UDim2.new(1, 0, 0, 20)
    SubTitle2.Position = UDim2.new(0, 0, 0, 22)
    SubTitle2.BackgroundTransparency = 1
    SubTitle2.Text = "AUTENTICAÇÃO ONLINE"
    SubTitle2.TextColor3 = Color3.fromRGB(180, 50, 50)
    SubTitle2.TextSize = 8
    SubTitle2.Font = Enum.Font.Gotham

    local InputFrame2 = Instance.new("Frame", MainFrame2)
    InputFrame2.Size = UDim2.new(0, 185, 0, 30)
    InputFrame2.Position = UDim2.new(0.5, -92, 0, 42)
    InputFrame2.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    local iCorner = Instance.new("UICorner", InputFrame2) iCorner.CornerRadius = UDim.new(0, 8)
    local iStroke = Instance.new("UIStroke", InputFrame2) iStroke.Color = Color3.fromRGB(50, 50, 50)

    local TextBox2 = Instance.new("TextBox", InputFrame2)
    TextBox2.Size = UDim2.new(1, -20, 1, 0)
    TextBox2.Position = UDim2.new(0, 10, 0, 0)
    TextBox2.BackgroundTransparency = 1
    TextBox2.Text = ""
    TextBox2.PlaceholderText = "Ex: 7B4-8P9-8HP"
    TextBox2.TextColor3 = Color3.fromRGB(255, 255, 255)
    TextBox2.TextSize = 10
    TextBox2.Font = Enum.Font.GothamBold
    TextBox2.ClearTextOnFocus = false

    local StatusLbl2 = Instance.new("TextLabel", MainFrame2)
    StatusLbl2.Size = UDim2.new(1, -20, 0, 25)
    StatusLbl2.Position = UDim2.new(0, 8, 0, 76)
    StatusLbl2.BackgroundTransparency = 1
    StatusLbl2.Text = "Aguardando sua Key..."
    StatusLbl2.TextColor3 = Color3.fromRGB(150, 150, 150)
    StatusLbl2.TextSize = 8
    StatusLbl2.Font = Enum.Font.Gotham
    StatusLbl2.TextWrapped = true

    local VerifyBtn2 = Instance.new("TextButton", MainFrame2)
    VerifyBtn2.Size = UDim2.new(0, 82, 0, 27)
    VerifyBtn2.Position = UDim2.new(0, 8, 0, 100)
    VerifyBtn2.BackgroundColor3 = Color3.fromRGB(214, 10, 20)
    VerifyBtn2.Text = "Verificar Key"
    VerifyBtn2.TextColor3 = Color3.fromRGB(255, 255, 255)
    VerifyBtn2.TextSize = 10
    VerifyBtn2.Font = Enum.Font.GothamBold
    local vCorner = Instance.new("UICorner", VerifyBtn2) vCorner.CornerRadius = UDim.new(0, 8)

    local GetKeyBtn2 = Instance.new("TextButton", MainFrame2)
    GetKeyBtn2.Size = UDim2.new(0, 82, 0, 27)
    GetKeyBtn2.Position = UDim2.new(0, 105, 0, 100)
    GetKeyBtn2.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    GetKeyBtn2.Text = "Pegar Key"
    GetKeyBtn2.TextColor3 = Color3.fromRGB(200, 200, 200)
    GetKeyBtn2.TextSize = 10
    GetKeyBtn2.Font = Enum.Font.GothamBold
    local gCorner = Instance.new("UICorner", GetKeyBtn2) gCorner.CornerRadius = UDim.new(0, 8)

    VerifyBtn2.MouseButton1Click:Connect(function()
        local keyFormatada = string.upper(string.gsub(TextBox2.Text, "%s+", ""))
        if #keyFormatada == 0 then
            StatusLbl2.Text = "⚠ Digite sua Key primeiro!"
            StatusLbl2.TextColor3 = Color3.fromRGB(255, 180, 0)
            return
        end
        VerifyBtn2.Text = "Verificando..."
        VerifyBtn2.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
        StatusLbl2.Text = "🔄 Conectando ao servidor..."
        StatusLbl2.TextColor3 = Color3.fromRGB(255, 200, 50)
        task.wait(0.1)
        local valido, msg, expTime = VerificarKeyOnline(keyFormatada)
        VerifyBtn2.Text = "Verificar Key"
        VerifyBtn2.BackgroundColor3 = Color3.fromRGB(214, 10, 20)
        if valido then
            StatusLbl2.Text = "✅ " .. msg
            StatusLbl2.TextColor3 = Color3.fromRGB(50, 255, 50)
            SalvarKey(keyFormatada)
            task.wait(1)
            KeyUI:Destroy()
            CarregarScriptPrincipal()
        else
            StatusLbl2.Text = "❌ " .. msg
            StatusLbl2.TextColor3 = Color3.fromRGB(255, 50, 50)
            local posOrig = MainFrame2.Position
            for i = 1, 3 do
                MainFrame2.Position = posOrig + UDim2.new(0, 10, 0, 0) task.wait(0.05)
                MainFrame2.Position = posOrig - UDim2.new(0, 10, 0, 0) task.wait(0.05)
            end
            MainFrame2.Position = posOrig
        end
    end)

    GetKeyBtn2.MouseButton1Click:Connect(function()
        if setclipboard then pcall(function() setclipboard(LINK_DISCORD) end) end
        StatusLbl2.Text = "📱 Acesse o Discord para pegar sua Key!"
        StatusLbl2.TextColor3 = Color3.fromRGB(100, 200, 255)
    end)
end

CriarSistemaKey()
