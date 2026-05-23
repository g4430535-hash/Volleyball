return(function()
    local Players = game:GetService("Players")
    local RunService = game:GetService("RunService")
    local VIM = game:GetService("VirtualInputManager")
    local UserInputService = game:GetService("UserInputService")

    repeat task.wait() until game:GetService("Players").LocalPlayer
    local lp = Players.LocalPlayer
    repeat task.wait() until lp.Character and lp.Character:FindFirstChild("HumanoidRootPart")
    local char = lp.Character
    local root = char:WaitForChild("HumanoidRootPart")
    local humanoid = char:WaitForChild("Humanoid")

    lp.CharacterAdded:Connect(function(c)
        char = c
        root = c:WaitForChild("HumanoidRootPart")
        humanoid = c:WaitForChild("Humanoid")
    end)

    local RADIUS = 11
    local FAR_RESET = 20
    local BASE_LEAD = 0.35
    local MAX_LEAD = 0.65
    local SPEED_SCALE = 0.055
    local SPIKE_BASE_LEAD = 0.7
    local SPIKE_MAX_LEAD = 1.1
    local SPIKE_SPEED_SCALE = 0.06
    local SPIKE_VY_THRESHOLD = -60
    local SPIKE_CLOSING_MIN = 20
    local GRAVITY = Vector3.new(0, -196.2, 0)
    local RADIUS_SQ = RADIUS * RADIUS
    local FAR_SQ = FAR_RESET * FAR_RESET

    local circle = Instance.new("Part")
    circle.Size = Vector3.new(0.1, RADIUS * 2, RADIUS * 2)
    circle.Shape = Enum.PartType.Cylinder
    circle.Color = Color3.fromRGB(255, 80, 120)
    circle.Material = Enum.Material.Neon
    circle.Transparency = 0.35
    circle.CanCollide = false
    circle.Anchored = true
    circle.CastShadow = false
    circle.Parent = workspace

    local ball = nil

    local function findBall()
        for _, v in ipairs(workspace:GetDescendants()) do
            if (v:IsA("Part") and v.Shape == Enum.PartType.Ball)
            or (v:IsA("BasePart") and v.Name:lower():find("ball")) then
                return v
            end
        end
    end

    ball = findBall()

    workspace.DescendantAdded:Connect(function(v)
        if ball then return end
        if (v:IsA("Part") and v.Shape == Enum.PartType.Ball)
        or (v:IsA("BasePart") and v.Name:lower():find("ball")) then
            ball = v
        end
    end)

    workspace.DescendantRemoving:Connect(function(v)
        if v == ball then ball = nil end
    end)

    local airStates = {
        [Enum.HumanoidStateType.Jumping] = true,
        [Enum.HumanoidStateType.Freefall] = true,
        [Enum.HumanoidStateType.Flying] = true,
    }

    local function inAir()
        return humanoid and airStates[humanoid:GetState()] == true
    end

    local function pressL2()
        VIM:SendKeyEvent(true, Enum.KeyCode.ButtonL2, false, game)
        VIM:SendKeyEvent(false, Enum.KeyCode.ButtonL2, false, game)
        VIM:SendGamepadButtonEvent(0, Enum.KeyCode.ButtonL2, true, game)
        VIM:SendGamepadButtonEvent(0, Enum.KeyCode.ButtonL2, false, game)
    end

    local function predictPos(pos, vel, t)
        return pos + vel * t + GRAVITY * (0.5 * t * t)
    end

    local function isSpiking(vel, toPlayer)
        if vel.Y > SPIKE_VY_THRESHOLD then return false end
        return (vel.X * toPlayer.X + vel.Z * toPlayer.Z) >= SPIKE_CLOSING_MIN
    end

    local autoEnabled = true
    local hasReceived = false

    UserInputService.InputBegan:Connect(function(input, gpe)
        if gpe then return end
        if input.KeyCode == Enum.KeyCode.ButtonL3 then
            autoEnabled = not autoEnabled
            print("[Volleyball] Auto receive:", autoEnabled and "ON" or "OFF")
        end
    end)

    RunService.PreSimulation:Connect(function()
        if not (root and root.Parent and ball and ball.Parent) then return end

        local rPos = root.Position
        local bPos = ball.Position
        local vel = ball.AssemblyLinearVelocity
        local speed = vel.Magnitude
        local diff = bPos - rPos
        local distSq = diff.X^2 + diff.Y^2 + diff.Z^2

        if hasReceived and distSq > FAR_SQ then hasReceived = false end

        local toPlayer = Vector3.new(-diff.X, 0, -diff.Z)
        local spike = isSpiking(vel, toPlayer)
        local lead = spike
            and math.clamp(speed * SPIKE_SPEED_SCALE, SPIKE_BASE_LEAD, SPIKE_MAX_LEAD)
            or math.clamp(speed * SPEED_SCALE, BASE_LEAD, MAX_LEAD)

        local inRange = distSq <= RADIUS_SQ
        if not inRange then
            local steps = spike and 6 or 4
            for i = 1, steps do
                local pd = predictPos(bPos, vel, lead * (i / steps)) - rPos
                if pd.X^2 + pd.Y^2 + pd.Z^2 <= RADIUS_SQ then
                    inRange = true
                    break
                end
            end
        end

        local airborne = inAir()

        circle.Color = (not autoEnabled) and Color3.fromRGB(80, 80, 80)
            or airborne and Color3.fromRGB(255, 180, 0)
            or Color3.fromRGB(255, 80, 120)

        if autoEnabled and inRange and not airborne and not hasReceived then
            hasReceived = true
            pressL2()
            circle.Color = Color3.fromRGB(255, 255, 255)
            task.delay(0.3, function() circle.Color = Color3.fromRGB(255, 80, 120) end)
            print(string.format("[Volleyball] Received! spike=%s velY=%.1f lead=%.2fs",
                tostring(spike), vel.Y, lead))
        end
    end)

    RunService:BindToRenderStep("CircleFollow", Enum.RenderPriority.Last.Value, function()
        if root and root.Parent then
            circle.CFrame = CFrame.new(root.Position.X, root.Position.Y - 2.9, root.Position.Z)
                * CFrame.Angles(0, 0, math.rad(90))
        end
    end)

    print("[Volleyball] Ready!")
end)()
