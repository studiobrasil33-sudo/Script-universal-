-- Carregar a Rayfield
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- Criar a janela principal com tema personalizado roxo/preto
local Window = Rayfield:CreateWindow({
   Name = "⚡ Hub br  ⚡",
   Icon = 0,
   LoadingTitle = "Carregando Sistema Completo...",
   LoadingSubtitle = "by Desenvolvedor",
   ShowText = "Forças Hub",
   Theme = {
      TextColor = Color3.fromRGB(255, 255, 255),
      Background = Color3.fromRGB(20, 0, 30),
      Topbar = Color3.fromRGB(75, 0, 130),
      Shadow = Color3.fromRGB(10, 0, 15),
      NotificationBackground = Color3.fromRGB(30, 0, 40),
      NotificationActionsBackground = Color3.fromRGB(147, 112, 219),
      TabBackground = Color3.fromRGB(106, 13, 173),
      TabStroke = Color3.fromRGB(138, 43, 226),
      TabBackgroundSelected = Color3.fromRGB(186, 85, 211),
      TabTextColor = Color3.fromRGB(255, 255, 255),
      SelectedTabTextColor = Color3.fromRGB(0, 0, 0),
      ElementBackground = Color3.fromRGB(45, 0, 65),
      ElementBackgroundHover = Color3.fromRGB(60, 0, 85),
      SecondaryElementBackground = Color3.fromRGB(30, 0, 45),
      ElementStroke = Color3.fromRGB(106, 13, 173),
      SecondaryElementStroke = Color3.fromRGB(75, 0, 130),
      SliderBackground = Color3.fromRGB(138, 43, 226),
      SliderProgress = Color3.fromRGB(186, 85, 211),
      SliderStroke = Color3.fromRGB(147, 112, 219),
      ToggleBackground = Color3.fromRGB(30, 0, 40),
      ToggleEnabled = Color3.fromRGB(138, 43, 226),
      ToggleDisabled = Color3.fromRGB(75, 0, 130),
      ToggleEnabledStroke = Color3.fromRGB(186, 85, 211),
      ToggleDisabledStroke = Color3.fromRGB(106, 13, 173),
      ToggleEnabledOuterStroke = Color3.fromRGB(147, 112, 219),
      ToggleDisabledOuterStroke = Color3.fromRGB(60, 0, 80),
      DropdownSelected = Color3.fromRGB(106, 13, 173),
      DropdownUnselected = Color3.fromRGB(45, 0, 65),
      InputBackground = Color3.fromRGB(30, 0, 40),
      InputStroke = Color3.fromRGB(106, 13, 173),
      PlaceholderColor = Color3.fromRGB(186, 85, 211)
   },

   ToggleUIKeybind = "K",
   DisableRayfieldPrompts = false,
   DisableBuildWarnings = false,

   ConfigurationSaving = {
      Enabled = true,
      FolderName = "ForcasHub",
      FileName = "Configuracoes"
   },

   Discord = {
      Enabled = false,
      Invite = "noinvitelink",
      RememberJoins = true
   },

   KeySystem = false
})

-- ==================== VARIÁVEIS GLOBAIS ====================
local flyEnabled = false
local flySpeed = 50
local speedEnabled = false
local speedValue = 50
local jumpPowerEnabled = false
local jumpPowerValue = 50
local gravityEnabled = false
local gravityValue = 196.2
local noclipEnabled = false
local infiniteJumpEnabled = false
local espEnabled = false
local wallhackEnabled = false
local playerList = {}
local selectedPlayer = nil
local bodyVelocity = nil
local bodyGyro = nil
local espObjects = {}
local aimEnabled = false
local circleEnabled = false
local aimTarget = nil
local circle = nil
local aimAssistEnabled = false
local aimSmoothness = 5
local aimMode = "Cabeça"
local triggerbotEnabled = false
local triggerbotDelay = 0.1
local autofarmEnabled = false
local autofarmDistance = 30

-- ==================== FUNÇÕES ÚTEIS ====================
local function getPlayers()
    local players = {}
    for _, player in ipairs(game:GetService("Players"):GetPlayers()) do
        if player ~= game:GetService("Players").LocalPlayer then
            table.insert(players, player.Name)
        end
    end
    return players
end

local function updatePlayerList()
    playerList = getPlayers()
    if #playerList == 0 then
        table.insert(playerList, "Nenhum jogador")
    end
    return playerList
end

-- ==================== SISTEMA DE FLY ====================
local FlyTab = Window:CreateTab("Fly", "plane")
local FlySection = FlyTab:CreateSection("Sistema de Voo")

local FlyToggle = FlyTab:CreateToggle({
   Name = "Ativar Fly",
   CurrentValue = false,
   Flag = "FlyToggle",
   Callback = function(Value)
      flyEnabled = Value
      local player = game:GetService("Players").LocalPlayer
      local character = player.Character
      
      if not character or not character:FindFirstChild("HumanoidRootPart") then
          Rayfield:Notify({Title = "Erro", Content = "Personagem não encontrado!", Duration = 3, Image = "alert-triangle"})
          return
      end
      
      local hrp = character.HumanoidRootPart
      local humanoid = character:FindFirstChild("Humanoid")
      
      if Value then
          if humanoid then humanoid.PlatformStand = true end
          
          bodyVelocity = Instance.new("BodyVelocity")
          bodyVelocity.MaxForce = Vector3.new(4000, 4000, 4000)
          bodyVelocity.Parent = hrp
          
          bodyGyro = Instance.new("BodyGyro")
          bodyGyro.MaxTorque = Vector3.new(4000, 4000, 4000)
          bodyGyro.P = 1000
          bodyGyro.Parent = hrp
          
          game:GetService("RunService"):BindToRenderStep("FlyLoop", Enum.RenderPriority.Character.Value, function()
              if not flyEnabled or not hrp or not hrp.Parent then return end
              
              local moveDirection = Vector3.new()
              local camera = workspace.CurrentCamera
              
              if game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.W) then moveDirection = moveDirection + camera.CFrame.LookVector end
              if game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.S) then moveDirection = moveDirection - camera.CFrame.LookVector end
              if game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.A) then moveDirection = moveDirection - camera.CFrame.RightVector end
              if game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.D) then moveDirection = moveDirection + camera.CFrame.RightVector end
              if game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.Space) then moveDirection = moveDirection + Vector3.new(0, 1, 0) end
              if game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.LeftShift) then moveDirection = moveDirection - Vector3.new(0, 1, 0) end
              
              if bodyVelocity then bodyVelocity.Velocity = moveDirection * flySpeed end
              if bodyGyro then bodyGyro.CFrame = CFrame.new(hrp.Position, hrp.Position + camera.CFrame.LookVector) end
          end)
          
          Rayfield:Notify({Title = "Fly Ativado", Content = "Use WASD + Espaço/Shift", Duration = 3, Image = "plane"})
      else
          if bodyVelocity then bodyVelocity:Destroy() bodyVelocity = nil end
          if bodyGyro then bodyGyro:Destroy() bodyGyro = nil end
          if humanoid then humanoid.PlatformStand = false end
          game:GetService("RunService"):UnbindFromRenderStep("FlyLoop")
          Rayfield:Notify({Title = "Fly Desativado", Content = "Sistema desligado", Duration = 3, Image = "plane-off"})
      end
   end
})

local FlySpeedSlider = FlyTab:CreateSlider({
   Name = "Velocidade do Fly",
   Range = {10, 500},
   Increment = 10,
   Suffix = "studs/s",
   CurrentValue = 50,
   Flag = "FlySpeed",
   Callback = function(Value) flySpeed = Value end
})

-- ==================== SISTEMA DE VELOCIDADE ====================
local SpeedTab = Window:CreateTab("Velocidade", "zap")
local SpeedSection = SpeedTab:CreateSection("Super Velocidade")

local SpeedToggle = SpeedTab:CreateToggle({
   Name = "Super Velocidade",
   CurrentValue = false,
   Flag = "SpeedToggle",
   Callback = function(Value)
      speedEnabled = Value
      local player = game:GetService("Players").LocalPlayer
      local character = player.Character
      
      if not character or not character:FindFirstChild("Humanoid") then
          Rayfield:Notify({Title = "Erro", Content = "Personagem não encontrado!", Duration = 3, Image = "alert-triangle"})
          return
      end
      
      local humanoid = character:FindFirstChild("Humanoid")
      
      if Value then
          humanoid:SetAttribute("OriginalWalkSpeed", humanoid.WalkSpeed)
          humanoid.WalkSpeed = speedValue
          Rayfield:Notify({Title = "Super Velocidade", Content = "Velocidade: " .. speedValue, Duration = 3, Image = "zap"})
      else
          local originalSpeed = humanoid:GetAttribute("OriginalWalkSpeed") or 16
          humanoid.WalkSpeed = originalSpeed
          Rayfield:Notify({Title = "Velocidade Desativada", Content = "Velocidade normal", Duration = 3, Image = "zap-off"})
      end
   end
})

local SpeedSlider = SpeedTab:CreateSlider({
   Name = "Velocidade",
   Range = {16, 500},
   Increment = 10,
   Suffix = "studs/s",
   CurrentValue = 50,
   Flag = "SpeedValue",
   Callback = function(Value)
      speedValue = Value
      if speedEnabled then
          local player = game:GetService("Players").LocalPlayer
          local character = player.Character
          if character and character:FindFirstChild("Humanoid") then
              character.Humanoid.WalkSpeed = Value
          end
      end
   end
})

-- ==================== SISTEMA DE PULO ====================
local JumpTab = Window:CreateTab("Pulo", "arrow-up")
local JumpSection = JumpTab:CreateSection("Super Pulo")

local JumpToggle = JumpTab:CreateToggle({
   Name = "Super Pulo",
   CurrentValue = false,
   Flag = "JumpToggle",
   Callback = function(Value)
      jumpPowerEnabled = Value
      local player = game:GetService("Players").LocalPlayer
      local character = player.Character
      
      if not character or not character:FindFirstChild("Humanoid") then
          Rayfield:Notify({Title = "Erro", Content = "Personagem não encontrado!", Duration = 3, Image = "alert-triangle"})
          return
      end
      
      local humanoid = character:FindFirstChild("Humanoid")
      
      if Value then
          humanoid:SetAttribute("OriginalJumpPower", humanoid.JumpPower)
          humanoid.JumpPower = jumpPowerValue
          Rayfield:Notify({Title = "Super Pulo", Content = "Altura: " .. jumpPowerValue, Duration = 3, Image = "arrow-up"})
      else
          local originalJump = humanoid:GetAttribute("OriginalJumpPower") or 50
          humanoid.JumpPower = originalJump
          Rayfield:Notify({Title = "Pulo Desativado", Content = "Altura normal", Duration = 3, Image = "arrow-down"})
      end
   end
})

local JumpSlider = JumpTab:CreateSlider({
   Name = "Altura do Pulo",
   Range = {50, 500},
   Increment = 10,
   Suffix = "força",
   CurrentValue = 50,
   Flag = "JumpValue",
   Callback = function(Value)
      jumpPowerValue = Value
      if jumpPowerEnabled then
          local player = game:GetService("Players").LocalPlayer
          local character = player.Character
          if character and character:FindFirstChild("Humanoid") then
              character.Humanoid.JumpPower = Value
          end
      end
   end
})

local InfiniteJumpToggle = JumpTab:CreateToggle({
   Name = "Pulo Infinito",
   CurrentValue = false,
   Flag = "InfiniteJump",
   Callback = function(Value)
      infiniteJumpEnabled = Value
      local userInput = game:GetService("UserInputService")
      
      if Value then
          local connection
          connection = userInput.JumpRequest:Connect(function()
              if infiniteJumpEnabled then
                  local player = game:GetService("Players").LocalPlayer
                  local character = player.Character
                  if character and character:FindFirstChild("Humanoid") then
                      character.Humanoid:ChangeState("Jumping")
                  end
              end
          end)
          
          -- Armazenar conexão para desconectar depois
          _G.InfiniteJumpConnection = connection
          Rayfield:Notify({Title = "Pulo Infinito", Content = "Ativado", Duration = 3, Image = "repeat"})
      else
          if _G.InfiniteJumpConnection then
              _G.InfiniteJumpConnection:Disconnect()
              _G.InfiniteJumpConnection = nil
          end
          Rayfield:Notify({Title = "Pulo Infinito", Content = "Desativado", Duration = 3, Image = "x"})
      end
   end
})

-- ==================== SISTEMA DE GRAVIDADE ====================
local GravityTab = Window:CreateTab("Gravidade", "moon")
local GravitySection = GravityTab:CreateSection("Controle de Gravidade")

local GravityToggle = GravityTab:CreateToggle({
   Name = "Modificar Gravidade",
   CurrentValue = false,
   Flag = "GravityToggle",
   Callback = function(Value)
      gravityEnabled = Value
      if Value then
          workspace.Gravity = gravityValue
          Rayfield:Notify({Title = "Gravidade Modificada", Content = "Valor: " .. gravityValue, Duration = 3, Image = "moon"})
      else
          workspace.Gravity = 196.2
          Rayfield:Notify({Title = "Gravidade Normal", Content = "Restaurada", Duration = 3, Image = "sun"})
      end
   end
})

local GravitySlider = GravityTab:CreateSlider({
   Name = "Força da Gravidade",
   Range = {0, 500},
   Increment = 10,
   Suffix = "força",
   CurrentValue = 196.2,
   Flag = "GravityValue",
   Callback = function(Value)
      gravityValue = Value
      if gravityEnabled then
          workspace.Gravity = Value
      end
   end
})

-- ==================== SISTEMA DE NOCLIP ====================
local NoclipTab = Window:CreateTab("Noclip", "wand")
local NoclipSection = NoclipTab:CreateSection("Atravessar Paredes")

local NoclipToggle = NoclipTab:CreateToggle({
   Name = "Noclip",
   CurrentValue = false,
   Flag = "NoclipToggle",
   Callback = function(Value)
      noclipEnabled = Value
      
      if Value then
          game:GetService("RunService"):BindToRenderStep("NoclipLoop", Enum.RenderPriority.Character.Value, function()
              if noclipEnabled then
                  local player = game:GetService("Players").LocalPlayer
                  local character = player.Character
                  if character then
                      for _, part in ipairs(character:GetDescendants()) do
                          if part:IsA("BasePart") then
                              part.CanCollide = false
                          end
                      end
                  end
              end
          end)
          Rayfield:Notify({Title = "Noclip Ativado", Content = "Atravesse paredes!", Duration = 3, Image = "wand"})
      else
          game:GetService("RunService"):UnbindFromRenderStep("NoclipLoop")
          local player = game:GetService("Players").LocalPlayer
          local character = player.Character
          if character then
              for _, part in ipairs(character:GetDescendants()) do
                  if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
                      part.CanCollide = true
                  end
              end
          end
          Rayfield:Notify({Title = "Noclip Desativado", Content = "Colisões normais", Duration = 3, Image = "x"})
      end
   end
})

-- ==================== SISTEMA DE ESP ====================
local ESPTab = Window:CreateTab("ESP", "eye")
local ESPsection = ESPTab:CreateSection("Visualizar Jogadores")

local function createESP(player)
    if espObjects[player] then return end
    
    local esp = {}
    
    -- Caixa ao redor do jogador
    local box = Drawing.new("Square")
    box.Visible = false
    box.Color = Color3.fromRGB(186, 85, 211)
    box.Thickness = 2
    box.Filled = false
    box.Transparency = 1
    
    -- Nome do jogador
    local name = Drawing.new("Text")
    name.Visible = false
    name.Color = Color3.fromRGB(255, 255, 255)
    name.Size = 16
    name.Center = true
    name.Outline = true
    name.Font = 2
    
    -- Distância
    local distance = Drawing.new("Text")
    distance.Visible = false
    distance.Color = Color3.fromRGB(186, 85, 211)
    distance.Size = 14
    distance.Center = true
    distance.Outline = true
    distance.Font = 2
    
    -- Saúde
    local health = Drawing.new("Text")
    health.Visible = false
    health.Color = Color3.fromRGB(0, 255, 0)
    health.Size = 14
    health.Center = true
    health.Outline = true
    health.Font = 2
    
    esp.Box = box
    esp.Name = name
    esp.Distance = distance
    esp.Health = health
    
    espObjects[player] = esp
end

local ESPToggle = ESPTab:CreateToggle({
   Name = "Ativar ESP",
   CurrentValue = false,
   Flag = "ESPToggle",
   Callback = function(Value)
      espEnabled = Value
      
      if Value then
          -- Criar ESP para todos os jogadores
          for _, player in ipairs(game:GetService("Players"):GetPlayers()) do
              if player ~= game:GetService("Players").LocalPlayer then
                  createESP(player)
              end
          end
          
          -- Loop de atualização do ESP
          game:GetService("RunService"):BindToRenderStep("ESPLoop", Enum.RenderPriority.Camera.Value, function()
              if not espEnabled then return end
              
              local camera = workspace.CurrentCamera
              local localPlayer = game:GetService("Players").LocalPlayer
              
              for player, esp in pairs(espObjects) do
                  if player and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Humanoid") then
                      local hrp = player.Character.HumanoidRootPart
                      local humanoid = player.Character.Humanoid
                      local screenPos, onScreen = camera:WorldToViewportPoint(hrp.Position)
                      
                      if onScreen then
                          -- Calcular tamanho da caixa baseado na distância
                          local dist = (camera.CFrame.Position - hrp.Position).Magnitude
                          local scale = 500 / dist
                          local boxSize = Vector2.new(scale * 2, scale * 3)
                          
                          -- Posicionar caixa
                          esp.Box.Position = Vector2.new(screenPos.X - boxSize.X/2, screenPos.Y - boxSize.Y/2)
                          esp.Box.Size = boxSize
                          esp.Box.Visible = true
                          
                          -- Nome do jogador
                          esp.Name.Position = Vector2.new(screenPos.X, screenPos.Y - boxSize.Y/2 - 20)
                          esp.Name.Text = player.Name
                          esp.Name.Visible = true
                          
                          -- Distância
                          esp.Distance.Position = Vector2.new(screenPos.X, screenPos.Y - boxSize.Y/2 - 5)
                          esp.Distance.Text = math.floor(dist) .. " studs"
                          esp.Distance.Visible = true
                          
                          -- Saúde
                          local healthColor = Color3.fromRGB(255 * (1 - humanoid.Health/humanoid.MaxHealth), 255 * (humanoid.Health/humanoid.MaxHealth), 0)
                          esp.Health.Color = healthColor
                          esp.Health.Position = Vector2.new(screenPos.X, screenPos.Y + boxSize.Y/2 + 5)
                          esp.Health.Text = math.floor(humanoid.Health) .. "/" .. math.floor(humanoid.MaxHealth) .. " HP"
                          esp.Health.Visible = true
                      else
                          esp.Box.Visible = false
                          esp.Name.Visible = false
                          esp.Distance.Visible = false
                          esp.Health.Visible = false
                      end
                  else
                      esp.Box.Visible = false
                      esp.Name.Visible = false
                      esp.Distance.Visible = false
                      esp.Health.Visible = false
                  end
              end
          end)
          
          Rayfield:Notify({Title = "ESP Ativado", Content = "Visualização de jogadores", Duration = 3, Image = "eye"})
      else
          game:GetService("RunService"):UnbindFromRenderStep("ESPLoop")
          for _, esp in pairs(espObjects) do
              esp.Box.Visible = false
              esp.Box:Remove()
              esp.Name.Visible = false
              esp.Name:Remove()
              esp.Distance.Visible = false
              esp.Distance:Remove()
              esp.Health.Visible = false
              esp.Health:Remove()
          end
          espObjects = {}
          Rayfield:Notify({Title = "ESP Desativado", Content = "Visualização desligada", Duration = 3, Image = "eye-off"})
      end
   end
})

local WallhackToggle = ESPTab:CreateToggle({
   Name = "Wallhack (Ver através de paredes)",
   CurrentValue = false,
   Flag = "Wallhack",
   Callback = function(Value)
      wallhackEnabled = Value
      
      if Value then
          -- Criar highlight para todos os jogadores
          for _, player in ipairs(game:GetService("Players"):GetPlayers()) do
              if player ~= game:GetService("Players").LocalPlayer and player.Character then
                  local highlight = Instance.new("Highlight")
                  highlight.Name = "WallhackHighlight"
                  highlight.FillColor = Color3.fromRGB(186, 85, 211)
                  highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                  highlight.FillTransparency = 0.5
                  highlight.Parent = player.Character
              end
          end
          
          -- Detectar novos jogadores
          game:GetService("Players").PlayerAdded:Connect(function(player)
              player.CharacterAdded:Connect(function(character)
                  if wallhackEnabled then
                      wait(1)
                      local highlight = Instance.new("Highlight")
                      highlight.Name = "WallhackHighlight"
                      highlight.FillColor = Color3.fromRGB(186, 85, 211)
                      highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                      highlight.FillTransparency = 0.5
                      highlight.Parent = character
                  end
              end)
          end)
          
          Rayfield:Notify({Title = "Wallhack Ativado", Content = "Veja através das paredes", Duration = 3, Image = "eye"})
      else
          -- Remover highlights
          for _, player in ipairs(game:GetService("Players"):GetPlayers()) do
              if player.Character and player.Character:FindFirstChild("WallhackHighlight") then
                  player.Character.WallhackHighlight:Destroy()
              end
          end
          Rayfield:Notify({Title = "Wallhack Desativado", Content = "Visão normal", Duration = 3, Image = "eye-off"})
      end
   end
})

-- ==================== SISTEMA DE MIRA COMPLETO ====================
local AimTab = Window:CreateTab("Sistema de Mira", "target")
local AimSection = AimTab:CreateSection("Configurações da Mira")

local fovSize = 150

local function createCircle()
    if circle then circle:Destroy() end
    circle = Drawing.new("Circle")
    circle.Visible = circleEnabled
    circle.Radius = fovSize
    circle.Color = Color3.fromRGB(186, 85, 211)
    circle.Thickness = 2
    circle.NumSides = 64
    circle.Filled = false
    circle.Transparency = 1
    circle.Position = Vector2.new(workspace.CurrentCamera.ViewportSize.X / 2, workspace.CurrentCamera.ViewportSize.Y / 2)
end

workspace.CurrentCamera:GetPropertyChangedSignal("ViewportSize"):Connect(function()
    if circle then
        circle.Position = Vector2.new(workspace.CurrentCamera.ViewportSize.X / 2, workspace.CurrentCamera.ViewportSize.Y / 2)
    end
end)

local function getTargetPart(character)
    if not character then return nil end
    if aimMode == "Cabeça" then
        return character:FindFirstChild("Head") or character:FindFirstChild("HumanoidRootPart")
    elseif aimMode == "Tronco" then
        return character:FindFirstChild("Torso") or character:FindFirstChild("UpperTorso") or character:FindFirstChild("HumanoidRootPart")
    else
        local parts = {"Head", "Torso", "UpperTorso", "LowerTorso", "HumanoidRootPart"}
        for _, partName in ipairs(parts) do
            local part = character:FindFirstChild(partName)
            if part then return part end
        end
    end
    return character:FindFirstChild("HumanoidRootPart")
end

local function getClosestPlayerToCenter()
    local closestPlayer = nil
    local closestDistance = fovSize
    local camera = workspace.CurrentCamera
    local center = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)
    
    for _, player in ipairs(game:GetService("Players"):GetPlayers()) do
        if player ~= game:GetService("Players").LocalPlayer and player.Character then
            local targetPart = getTargetPart(player.Character)
            if targetPart then
                local screenPoint, onScreen = camera:WorldToViewportPoint(targetPart.Position)
                if onScreen then
                    local distance = (Vector2.new(screenPoint.X, screenPoint.Y) - center).Magnitude
                    if distance < closestDistance then
                        closestDistance = distance
                        closestPlayer = player
                    end
                end
            end
        end
    end
    return closestPlayer
end

game:GetService("RunService"):BindToRenderStep("AimLoop", Enum.RenderPriority.Input.Value, function()
    if not aimEnabled or not aimAssistEnabled then
        if circle then circle.Visible = circleEnabled end
        return
    end
    
    if circle then
        circle.Visible = circleEnabled
        circle.Radius = fovSize
        circle.Position = Vector2.new(workspace.CurrentCamera.ViewportSize.X / 2, workspace.CurrentCamera.ViewportSize.Y / 2)
    end
    
    local targetPlayer = nil
    if aimTarget == "Selecionado" and selectedPlayer then
        targetPlayer = game:GetService("Players"):FindFirstChild(selectedPlayer)
    else
        targetPlayer = getClosestPlayerToCenter()
    end
    
    if targetPlayer and targetPlayer.Character then
        local targetPart = getTargetPart(targetPlayer.Character)
        if targetPart then
            local camera = workspace.CurrentCamera
            local targetPosition = targetPart.Position
            local lookVector = (targetPosition - camera.CFrame.Position).Unit
            local newCFrame = CFrame.lookAt(camera.CFrame.Position, camera.CFrame.Position + lookVector)
            camera.CFrame = camera.CFrame:Lerp(newCFrame, aimSmoothness / 10)
        end
    end
end)

-- Sistema de Triggerbot (atirar automaticamente)
local function checkTriggerbot()
    if not triggerbotEnabled or not aimEnabled then return end
    
    local camera = workspace.CurrentCamera
    local center = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)
    
    for _, player in ipairs(game:GetService("Players"):GetPlayers()) do
        if player ~= game:GetService("Players").LocalPlayer and player.Character then
            local targetPart = getTargetPart(player.Character)
            if targetPart then
                local screenPoint, onScreen = camera:WorldToViewportPoint(targetPart.Position)
                if onScreen then
                    local distance = (Vector2.new(screenPoint.X, screenPoint.Y) - center).Magnitude
                    if distance < fovSize then
                        -- Simular clique do mouse
                        mouse1press()
                        wait(triggerbotDelay)
                        mouse1release()
                        break
                    end
                end
            end
        end
    end
end

-- Loop do triggerbot
game:GetService("RunService"):BindToRenderStep("TriggerLoop", Enum.RenderPriority.Input.Value, checkTriggerbot)

local AimToggle = AimTab:CreateToggle({
    Name = "Ativar Sistema de Mira",
    CurrentValue = false,
    Flag = "AimToggle",
    Callback = function(Value)
        aimEnabled = Value
        if not Value then aimAssistEnabled = false end
        Rayfield:Notify({Title = Value and "Mira Ativada" or "Mira Desativada", Content = Value and "Sistema pronto" or "Sistema desligado", Duration = 3, Image = "target"})
    end
})

local CircleToggle = AimTab:CreateToggle({
    Name = "Mostrar Círculo na Tela",
    CurrentValue = false,
    Flag = "CircleToggle",
    Callback = function(Value)
        circleEnabled = Value
        if not circle then createCircle() else circle.Visible = Value end
    end
})

local AimAssistToggle = AimTab:CreateToggle({
    Name = "Aimbot Automático",
    CurrentValue = false,
    Flag = "AimAssistToggle",
    Callback = function(Value)
        aimAssistEnabled = Value and aimEnabled
        if Value and not aimEnabled then
            Rayfield:Notify({Title = "Aviso", Content = "Ative a mira primeiro!", Duration = 3, Image = "alert-triangle"})
            AimAssistToggle:Set(false)
        end
    end
})

local TriggerbotToggle = AimTab:CreateToggle({
    Name = "Triggerbot (Atirar Automático)",
    CurrentValue = false,
    Flag = "Triggerbot",
    Callback = function(Value)
        triggerbotEnabled = Value
        Rayfield:Notify({Title = Value and "Triggerbot Ativado" or "Triggerbot Desativado", Content = Value and "Atira automaticamente" or "Modo manual", Duration = 3, Image = "mouse-pointer"})
    end
})

local AimTargetDropdown = AimTab:CreateDropdown({
    Name = "Mirar em",
    Options = {"Mais Próximo do Centro", "Selecionado"},
    CurrentOption = {"Mais Próximo do Centro"},
    MultipleOptions = false,
    Flag = "AimTarget",
    Callback = function(Options) aimTarget = Options[1] == "Selecionado" and "Selecionado" or "Proximo" end
})

local AimModeDropdown = AimTab:CreateDropdown({
    Name = "Parte do Corpo",
    Options = {"Cabeça", "Tronco", "Aleatório"},
    CurrentOption = {"Cabeça"},
    MultipleOptions = false,
    Flag = "AimMode",
    Callback = function(Options) aimMode = Options[1] end
})

local FovSlider = AimTab:CreateSlider({
    Name = "Tamanho do Círculo (FOV)",
    Range = {50, 500},
    Increment = 10,
    Suffix = "px",
    CurrentValue = 150,
    Flag = "FovSize",
    Callback = function(Value)
        fovSize = Value
        if circle then circle.Radius = Value end
    end
})

local SmoothSlider = AimTab:CreateSlider({
    Name = "Suavidade da Mira",
    Range = {1, 20},
    Increment = 1,
    Suffix = "%",
    CurrentValue = 5,
    Flag = "AimSmooth",
    Callback = function(Value) aimSmoothness = Value end
})

local TriggerDelaySlider = AimTab:CreateSlider({
    Name = "Delay do Triggerbot",
    Range = {0, 1},
    Increment = 0.05,
    Suffix = "s",
    CurrentValue = 0.1,
    Flag = "TriggerDelay",
    Callback = function(Value) triggerbotDelay = Value end
})

local CircleColorPicker = AimTab:CreateColorPicker({
    Name = "Cor do Círculo",
    Color = Color3.fromRGB(186, 85, 211),
    Flag = "CircleColor",
    Callback = function(Value) if circle then circle.Color = Value end end
})

-- ==================== SISTEMA DE TELEPORTE ====================
local TeleportTab = Window:CreateTab("Teleportes", "map-pin")
local TeleportSection = TeleportTab:CreateSection("Teleportes em Tempo Real")

local PlayerDropdown = TeleportTab:CreateDropdown({
   Name = "Selecionar Jogador",
   Options = updatePlayerList(),
   CurrentOption = {updatePlayerList()[1]},
   MultipleOptions = false,
   Flag = "PlayerSelect",
   Callback = function(Options) selectedPlayer = Options[1] end
})

local TpButton = TeleportTab:CreateButton({
   Name = "Teleportar para Jogador",
   Callback = function()
      if not selectedPlayer or selectedPlayer == "Nenhum jogador" then
          Rayfield:Notify({Title = "Erro", Content = "Selecione um jogador!", Duration = 3, Image = "alert-circle"})
          return
      end
      
      local targetPlayer = game:GetService("Players"):FindFirstChild(selectedPlayer)
      if targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
          local localPlayer = game:GetService("Players").LocalPlayer
          if localPlayer.Character and localPlayer.Character:FindFirstChild("HumanoidRootPart") then
              localPlayer.Character.HumanoidRootPart.CFrame = targetPlayer.Character.HumanoidRootPart.CFrame
              Rayfield:Notify({Title = "Teleportado!", Content = "Para " .. selectedPlayer, Duration = 3, Image = "map-pin"})
          end
      end
   end
})

local TpToMouse = TeleportTab:CreateButton({
   Name = "Teleportar para o Mouse",
   Callback = function()
      local localPlayer = game:GetService("Players").LocalPlayer
      if localPlayer.Character and localPlayer.Character:FindFirstChild("HumanoidRootPart") then
          local mouse = localPlayer:GetMouse()
          local target = mouse.Hit
          if target then
              localPlayer.Character.HumanoidRootPart.CFrame = target
              Rayfield:Notify({Title = "Teleportado!", Content = "Para posição do mouse", Duration = 3, Image = "mouse-pointer"})
          end
      end
   end
})

local RefreshButton = TeleportTab:CreateButton({
   Name = "Atualizar Lista",
   Callback = function()
      local newList = updatePlayerList()
      PlayerDropdown:Refresh(newList)
      Rayfield:Notify({Title = "Lista Atualizada", Content = #playerList - 1 .. " jogadores", Duration = 2, Image = "refresh-cw"})
   end
})

-- ==================== SISTEMA DE AUTOFARM ====================
local AutofarmTab = Window:CreateTab("Autofarm", "bot")
local AutofarmSection = AutofarmTab:CreateSection("Farm Automático")

local AutofarmToggle = AutofarmTab:CreateToggle({
   Name = "Autofarm",
   CurrentValue = false,
   Flag = "Autofarm",
   Callback = function(Value)
      autofarmEnabled = Value
      
      if Value then
          game:GetService("RunService"):BindToRenderStep("AutofarmLoop", Enum.RenderPriority.Character.Value, function()
              if not autofarmEnabled then return end
              
              local localPlayer = game:GetService("Players").LocalPlayer
              if not localPlayer.Character or not localPlayer.Character:FindFirstChild("HumanoidRootPart") then return end
              
              local hrp = localPlayer.Character.HumanoidRootPart
              
              -- Procurar por drops/coletáveis próximos
              for _, obj in ipairs(workspace:GetDescendants()) do
                  if obj:IsA("BasePart") and obj:FindFirstChild("TouchInterest") and (obj.Position - hrp.Position).Magnitude < autofarmDistance then
                      -- Mover para o objeto
                      hrp.CFrame = CFrame.new(obj.Position)
                  end
              end
              
              -- Procurar por inimigos
              for _, player in ipairs(game:GetService("Players"):GetPlayers()) do
                  if player ~= localPlayer and player.Character and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
                      local distance = (player.Character.HumanoidRootPart.Position - hrp.Position).Magnitude
                      if distance < autofarmDistance then
                          -- Atacar
                          local args = {player.Character.HumanoidRootPart, player.Character.Humanoid}
                          game:GetService("ReplicatedStorage"):FindFirstChildWhichIsA("RemoteEvent"):FireServer(unpack(args))
                      end
                  end
              end
          end)
          
          Rayfield:Notify({Title = "Autofarm Ativado", Content = "Farm automático ligado", Duration = 3, Image = "bot"})
      else
          game:GetService("RunService"):UnbindFromRenderStep("AutofarmLoop")
          Rayfield:Notify({Title = "Autofarm Desativado", Content = "Farm desligado", Duration = 3, Image = "x"})
      end
   end
})

local AutofarmDistance = AutofarmTab:CreateSlider({
   Name = "Distância de Farm",
   Range = {10, 100},
   Increment = 5,
   Suffix = "studs",
   CurrentValue = 30,
   Flag = "AutofarmDistance",
   Callback = function(Value) autofarmDistance = Value end
})

-- ==================== JOGADORES EM TEMPO REAL ====================
local PlayersTab = Window:CreateTab("Jogadores", "users")
local PlayersSection = PlayersTab:CreateSection("Jogadores Online")

local playerLabels = {}

local function updatePlayerLabels()
    for _, label in pairs(playerLabels) do
        label:Set("", 0, Color3.new(1,1,1), false)
    end
    playerLabels = {}
    
    local players = game:GetService("Players"):GetPlayers()
    for i, player in ipairs(players) do
        if player ~= game:GetService("Players").LocalPlayer then
            local status = "Online"
            if player.Character and player.Character:FindFirstChild("Humanoid") then
                local health = math.floor(player.Character.Humanoid.Health)
                status = "HP: " .. health .. "/" .. math.floor(player.Character.Humanoid.MaxHealth)
            end
            
            local label = PlayersTab:CreateLabel(
                player.Name .. " - " .. status,
                "user",
                Color3.fromRGB(186, 85, 211),
                false
            )
            table.insert(playerLabels, label)
        end
    end
    
    if #playerLabels == 0 then
        local label = PlayersTab:CreateLabel(
            "Nenhum outro jogador",
            "users",
            Color3.fromRGB(255, 255, 255),
            false
        )
        table.insert(playerLabels, label)
    end
end

spawn(function()
    while true do
        wait(5)
        updatePlayerLabels()
        local newList = updatePlayerList()
        if PlayerDropdown then PlayerDropdown:Refresh(newList) end
    end
end)

-- ==================== ESTRUTURA DO JOGO ====================
local StructureTab = Window:CreateTab("Estrutura", "box")
local StructureSection = StructureTab:CreateSection("Informações do Jogo")

local gameNameLabel = StructureTab:CreateLabel(
    "Jogo: " .. game:GetService("MarketplaceService"):GetProductInfo(game.PlaceId).Name,
    "gamepad-2",
    Color3.fromRGB(255, 255, 255),
    false
)

local placeIdLabel = StructureTab:CreateLabel(
    "Place ID: " .. game.PlaceId,
    "hash",
    Color3.fromRGB(255, 255, 255),
    false
)

local playerCountLabel = StructureTab:CreateLabel(
    "Jogadores: " .. #game:GetService("Players"):GetPlayers() .. "/" .. game:GetService("Players").MaxPlayers,
    "users",
    Color3.fromRGB(255, 255, 255),
    false
)

local serverTimeLabel = StructureTab:CreateLabel(
    "Tempo de Servidor: Carregando...",
    "clock",
    Color3.fromRGB(255, 255, 255),
    false
)

local fpsLabel = StructureTab:CreateLabel(
    "FPS: Calculando...",
    "activity",
    Color3.fromRGB(255, 255, 255),
    false
)

-- Contador de FPS
local frameCount = 0
local lastTime = tick()
local fps = 0

game:GetService("RunService").RenderStepped:Connect(function()
    frameCount = frameCount + 1
    local currentTime = tick()
    if currentTime - lastTime >= 1 then
        fps = frameCount
        frameCount = 0
        lastTime = currentTime
        fpsLabel:Set("FPS: " .. fps, "activity", Color3.fromRGB(255, 255, 255), false)
    end
end)

spawn(function()
    while true do
        wait(1)
        playerCountLabel:Set(
            "Jogadores: " .. #game:GetService("Players"):GetPlayers() .. "/" .. game:GetService("Players").MaxPlayers,
            "users",
            Color3.fromRGB(255, 255, 255),
            false
        )
        
        local time = math.floor(workspace.DistributedGameTime)
        local minutes = math.floor(time / 60)
        local seconds = time % 60
        serverTimeLabel:Set(
            string.format("Tempo de Servidor: %02d:%02d", minutes, seconds),
            "clock",
            Color3.fromRGB(255, 255, 255),
            false
        )
    end
end)

StructureTab:CreateParagraph({
    Title = "Informações do Sistema",
    Content = "Executor: " .. identifyexecutor() .. "\nRayfield: v2.0.0\nTema: Roxo/Preto\nDesenvolvedor: Hub Completo"
})

-- ==================== CONFIGURAÇÕES ====================
local SettingsTab = Window:CreateTab("Configurações", "settings")
local SettingsSection = SettingsTab:CreateSection("Configurações do UI")

SettingsTab:CreateButton({
    Name = "Reiniciar Interface",
    Callback = function()
        Rayfield:Notify({Title = "Reiniciando", Content = "Em 3 segundos...", Duration = 3, Image = "refresh-cw"})
        wait(3)
        loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
    end
})

SettingsTab:CreateButton({
    Name = "Destruir UI",
    Callback = function()
        Rayfield:Destroy()
    end
})

-- Criar círculo inicial
createCircle()

-- Notificação de inicialização
Rayfield:Notify({
    Title = "⚡ HUB COMPLETO CARREGADO ⚡",
    Content = "Sistema de forças totais ativado!\nPressione K para abrir/fechar\n10+ funcionalidades disponíveis",
    Duration = 8,
    Image = "zap"
})

-- Atualizar lista inicial
updatePlayerLabels()
