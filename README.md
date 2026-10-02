-- ============================================================
-- Resultado de la desofuscación estática de XeroHub / MoonVeil
-- Recuperado de 3.lua sin ejecutar la carga útil original.
-- La carga útil incrustada se decodificó desde Base64 y se le aplicó XOR con
-- la clave repetida: XeroKev_DFX_v3_2026
-- Este archivo conserva la lógica Lua/Luau recuperada; no hay comportamiento
-- se realizaron sustituciones.
-- ============================================================

función de retorno(host)
--[[
XeroHub | Efectos de muerte en DUELOS · CurrentCamera R36 · Corrección de duplicados · Base original
Kev

Objetivo:
- VOLVER al comportamiento que ya funcionaba.
- Descarga el V2 limpio del catálogo final.
- Reconstruye el activo faltante bajo ReplicatedStorage.ReplicatedSkins.Effects.
- Observa a los jugadores enemigos durante la ronda.
- Cuando un enemigo muere, sólo continúa si el kill puede atribuirse al jugador local.
- Utilice un PROXY LOCAL externo; el personaje enemigo queda intacto/solo lectura.
- Llama la lógica NATIVA del juego:
      DeathEffectPreview.play(localProxy, effectName, cleaner)
  para que el propio módulo aplique deformaciones, transparencias, cambios del
  cuerpo, animaciones, etc., no sólo partículas/sonidos.

No compra/equipa nada y no llama controles remotos de tienda.
]]

local Players = game:GetService("Players")
local ReplicatedStorage = juego:GetService("ReplicatedStorage")
local HttpService = game:GetService("HttpService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
Espacio de trabajo local = juego:GetService("Espacio de trabajo")

LP local = Jugadores.JugadorLocal
local PlayerGui = LP:WaitForChild("PlayerGui")

local sharedEnv = (getgenv y getgenv()) o _G
local anterior = sharedEnv.__XERO_DEATH_BRIDGE_RUNTIME
Si previous y previous.Cleanup están presentes, entonces pcall(previous.Cleanup) fin
ENV local = {XERO_DEATH_FINAL_ROOT = sharedEnv.XERO_DEATH_FINAL_ROOT}

-- Ciclo de vida R34: cada nueva ejecución limpia completamente la ejecución R33 ANTERIOR.
-- Las versiones R29-R32 más antiguas no tenían un sharedEnv.__XERO_DFX_RUNTIME global, por lo que un servidor nuevo/reconexión es
-- Se recomienda realizarlo una sola vez al migrar a R34; después de eso, la reejecución se realiza automáticamente.
si ENV.__XERO_DEATH_BRIDGE_RUNTIME
    y type(ENV.__XERO_DEATH_BRIDGE_RUNTIME.Cleanup) == "function" entonces
    pcall(ENV.__XERO_DEATH_BRIDGE_RUNTIME.Cleanup)
fin

ENV.__XERO_DEATH_BRIDGE_RUNTIME = {
    Vivo = verdadero,
    KillSignalConnections = nil,
    Conexiones de equipo local = nulo,
    PlayerConnections = nil,
    PlayerAddedConnection = nil,
    PlayerRemovingConnection = nil,
    Gui = nulo,
    HiddenParts = setmetatable({}, {__mode = "k"}),
    PersonajesOcultos = setmetatable({}, {__mode = "k"}),
    OcultarConexiones = establecermetatable({}, {__modo = "k"}),
}

-- R38: publicar el propietario. El código antiguo LEÍA esta ranura pero nunca escribía la nueva.
-- sharedEnv.__XERO_DFX_RUNTIME de vuelta, para que dos motores de efectos de muerte pudieran coexistir después de una reconstrucción.
sharedEnv.__XERO_DEATH_BRIDGE_RUNTIME = ENV.__XERO_DEATH_BRIDGE_RUNTIME

ENV.__XERO_DEATH_BRIDGE_RUNTIME.RestoreCharacter = function(character)
    local rt = ENV.__XERO_DEATH_BRIDGE_RUNTIME
    partes locales = rt.HiddenCharacters[character]
    si partes entonces
        para cada parte en pares(partes) hacer
            local oldLTM = rt.HiddenParts[part]
            Si oldLTM ~= nil y part y part.Parent y part:IsA("BasePart") entonces
                pcall(function() part.LocalTransparencyModifier = oldLTM end)
            fin
            rt.HiddenParts[parte] = nil
        fin
        rt.HiddenCharacters[character] = nil
    fin

    local conn = rt.HideConnections[character]
    si conecta entonces
        pcall(function() conn:Disconnect() end)
        rt.HideConnections[character] = nil
    fin
fin

ENV.__XERO_DEATH_BRIDGE_RUNTIME.RestoreHidden = función()
    local rt = ENV.__XERO_DEATH_BRIDGE_RUNTIME
    caracteres locales = {}
    para cada carácter en pares(rt.HiddenCharacters) hacer
        caracteres[#caracteres + 1] = carácter
    fin
    para _, carácter en ipairs(chars) hacer
        rt.RestoreCharacter(character)
    fin

    -- Recurso alternativo para una pieza cuyo personaje desapareció antes de que terminara la contabilidad.
    para cada parte, oldLTM en pares(rt.HiddenParts) hacer
        Si part y part.Parent y part:IsA("BasePart") entonces
            pcall(function() part.LocalTransparencyModifier = oldLTM end)
        fin
        rt.HiddenParts[parte] = nil
    fin
fin

ENV.__XERO_DEATH_BRIDGE_RUNTIME.HideCharacterLocal = function(character)
    Si no es un personaje o no es un personaje: es un ("Modelo") entonces devuelve 0.
    local rt = ENV.__XERO_DEATH_BRIDGE_RUNTIME
    partes locales = rt.HiddenCharacters[character]
    si no son piezas entonces
        partes = establecermetatable({}, {__modo = "k"})
        rt.HiddenCharacters[character] = partes
    fin

    función local hidePart(obj)
        Si no es obj o no es obj:IsA("BasePart") entonces devuelve falso.
        Si rt.HiddenParts[obj] == nil entonces
            rt.HiddenParts[obj] = obj.LocalTransparencyModifier
        fin
        partes[obj] = verdadero
        pcall(function() obj.LocalTransparencyModifier = 1 end)
        devolver verdadero
    fin

    recuento local = 0
    para _, obj en ipairs(character:GetDescendants()) hacer
        Si hidePart(obj) entonces count += 1 fin
    fin

    Si no rt.HideConnections[character] entonces
        rt.HideConnections[character] = character.DescendantAdded:Connect(function(obj)
            Si rt.Alive y character.Parent entonces
                ocultarParte(obj)
            fin
        fin)
    fin

    recuento de devoluciones
fin

ENV.__XERO_DEATH_BRIDGE_RUNTIME.Cleanup = function()
    local rt = ENV.__XERO_DEATH_BRIDGE_RUNTIME
    Si no rt o rt.Alive es falso, entonces regresa.
    rt.Alive = false

    Si sharedEnv.__XERO_DEATH_BRIDGE_RUNTIME == rt entonces
        sharedEnv.__XERO_DEATH_BRIDGE_RUNTIME = nil
    fin

    función local disconnectList(lista)
        para _, conn en ipairs(list o {}) hacer
            pcall(function() conn:Disconnect() end)
        fin
    fin

    desconectarList(rt.KillSignalConnections)
    desconectarList(rt.ConexionesDeEquipoLocal)
    para _, entrada en ipairs(rt.PendingDeaths o {}) hacer
        entrada.eliminada = verdadero
        desconectarList(entrada.conexiones)
    fin
    para _, información en pares(rt.TrackedHumanoids o {}) hacer
        listaDesconectar(info.conexiones)
    fin

    para _, lista en pares(rt.PlayerConnections o {}) hacer
        desconectarLista(lista)
    fin

    pcall(function() if rt.PlayerAddedConnection then rt.PlayerAddedConnection:Disconnect() end end)
    pcall(function() if rt.PlayerRemovingConnection then rt.PlayerRemovingConnection:Disconnect() end end)

    Si rt.RestoreHidden entonces pcall(rt.RestoreHidden) fin

    si ENV.__XERO_DEATH_PROXY
        y type(ENV.__XERO_DEATH_PROXY.Cleanup) == "function" entonces
        pcall(ENV.__XERO_DEATH_PROXY.Cleanup)
    fin

    local g = rt.Gui
    rt.Gui = nil
    Si g y g.Parent entonces pcall(function() g:Destroy() end)
fin
host.RegisterCleanup(ENV.__XERO_DEATH_BRIDGE_RUNTIME.Cleanup)
función local assertActive()
    si no es host.Alive() entonces error("Carga de efectos cancelada", 0) end
fin
assertActive()
BASE local = ENV.XERO_DEATH_FINAL_ROOT
    o "https://raw.githubusercontent.com/OnyxDevv/Onyx-web/main/death_effects_final"

local requestFn = (syn y syn.request) o (http y http.request) o http_request o request

función local httpGet(url)
    si requestFn entonces
        local ok, respuesta = pcall(función()
            devolver requestFn({
                URL = URL,
                Método = "GET",
                Encabezados = {
                    ["User-Agent"] = "XeroHub-DeathEffects-NativeBridge/1.0",
                    ["Aceptar"] = "application/json",
                },
            })
        fin)
        Si está bien y responde entonces
            código local = tonumber(respuesta.StatusCode o respuesta.Status o 0) o 0
            cuerpo local = response.Body o response.body
            Si el código es mayor o igual a 200 y el código es menor o igual a 300 y el tipo (cuerpo) es igual a "cadena" entonces
                cuerpo de retorno
            fin
        fin
    fin

    local ok, cuerpo = pcall(function()
        juego de retorno:HttpGet(url)
    fin)
    Si está bien y el tipo (cuerpo) es igual a "cadena" entonces
        cuerpo de retorno
    fin
    devolver cero
fin

-- ============================================================
-- Repositorio / caché
-- ============================================================
local CACHE_FOLDER = "XeroHub/DeathEffectsNativeEnemyR18"

función local ensureFolder(ruta)
    Si type(makefolder) ~= "function" entonces regresar fin
    pcall(función()
        Si no es una carpeta o no es una carpeta (ruta), entonces crea una carpeta (ruta).
    fin)
fin

función local ensureCache()
    asegurarCarpeta("XeroHub")
    asegurarCarpeta(CARPETA_CACHE)
fin

función local safeFileName(s)
    return tostring(s o ""):gsub("[^%w%._%-]", "_")
fin

función local readCache(fileName)
    Si type(isfile) ~= "function" o type(readfile) ~= "function" entonces devuelve nil.
    ruta local = CARPETA_CACHE .. "/" .. nombreArchivoSeguro(nombreArchivo)
    Si no es archivo(ruta), entonces devuelve nil.
    local ok, cuerpo = pcall(readfile, ruta)
    devolver ok y cuerpo o nulo
fin

función local writeCache(fileName, body)
    Si type(writefile) ~= "function" o type(body) ~= "string" entonces regresar fin
    asegurarCache()
    pcall(writefile, CACHE_FOLDER .. "/" .. safeFileName(fileName), body)
fin

local manifestBody = httpGet(BASE .. "/renderer_base/manifest.json")
si no manifiestaBody entonces
    error("Xero Native Bridge: no pude descargar manifest.json")
fin

local okManifest, manifest = pcall(function()
    devolver HttpService:JSONDecode(manifestBody)
fin)
Si no okManifest o type(manifest) ~= "table" o type(manifest.effects) ~= "table" entonces
    error("Xero Native Bridge: manifiesto inválido")
fin


EFECTOS locales = {}
local POR_NOMBRE = {}

función local isJellyEffect(nombre)
    return type(name) == "string"
        y string.find(name, "Jelly", 1, true) ~= nil
fin

para _, entrada en ipairs(manifest.effects) hacer
    si type(entry) == "table"
        y type(entry.name) == "string"
        y type(entry.v2) == "string"
        y no isJellyEffect(entrada.nombre)
        y no string.find(string.lower(entry.name), "ufo", 1, true)
        y no string.find(string.lower(entry.name), "venom", 1, true)
        y entry.name ~= "Latido del corazón"
        y entry.name ~= "SoulReaper"
        y entry.name ~= "BlackvalkEffect"
        y entry.name ~= "GhostbringerEffect"
        y entry.name ~= "Decorado"
        y entry.name ~= "ClanFlagEffect"
        y no string.find(string.lower(entry.name), "confetti", 1, true) entonces
        EFECTOS[#EFECTOS + 1] = nombre.entrada
        POR_NOMBRE[entrada.nombre] = entrada
    fin
fin
tabla.ordenar(EFECTOS)

local decodedCache = {}

función local validWrapper(datos)
    tipo de retorno(datos) == "tabla"
        y tonumber(data.schemaVersion) == 2
        y type(data.snapshot) == "table"
        y type(data.snapshot.nodes) == "table"
        y #data.snapshot.nodes > 0
fin

función local decodeWrapper(cuerpo)
    Si type(body) ~= "string" entonces devuelve nil.
    local ok, datos = pcall(función()
        devolver HttpService:JSONDecode(cuerpo)
    fin)
    Si es correcto y validWrapper(data), entonces devuelve data.
    devolver cero
fin

función local fetchEffect(nombre)
    Si decodedCache[name] entonces devuelve decodedCache[name] fin

    entrada local = POR_NOMBRE[nombre]
    Si no hay entrada, entonces devuelve nil, "No existe en manifiesto final." fin

    ruta local = entrada.v2
    cuerpo local = readCache(ruta)
    datos locales = decodeWrapper(cuerpo)

    si no hay datos entonces
        cuerpo = httpGet(BASE .. "/" .. ruta)
        datos = decodeWrapper(cuerpo)
        Si hay datos y cuerpo, entonces escribir en caché (ruta, cuerpo)
    fin

    si no hay datos entonces
        return nil, "No pude bajar V2 final: " .. tostring(ruta)
    fin

    decodedCache[nombre] = datos
    datos de retorno
fin


-- ============================================================
-- PRECARGA DE INICIO DE R14
-- Descargue todos los V2 seleccionables al ejecutar el script.
-- También fetchEffect() al seleccionar ya la venta de decodedCache.
-- ============================================================

local PRELOAD_STATS = {
    total = #EFECTOS,
    cargado = 0,
    fallido = 0,
    errores = {},
}

-- Los recursos se cargan al seleccionarlos, antes de habilitar las eliminaciones. No hay precarga del catálogo completo.
assertActive()

-- ============================================================

local decodedV3Cache = {}


-- Decodificador V2 tipificado
-- ============================================================
función local decodeTyped(valor)
    Si type(value) ~= "table" entonces devolver valor, verdadero fin

    local t = valor.t
    Si no es t, entonces devuelve el valor verdadero.

    si t == "nil" entonces
        devolver nil, falso
    elseif t == "Vector3" entonces
        return Vector3.new(valor.x o 0, valor.y o 0, valor.z o 0), true
    elseif t == "Vector2" entonces
        devolver Vector2.new(valor.x o 0, valor.y o 0), verdadero
    elseif t == "Color3" entonces
        return Color3.new(valor.r o 0, valor.g o 0, valor.b o 0), verdadero
    elseif t == "CFrame" entonces
        local c = valor.c
        si type(c) == "table" y #c >= 12 entonces
            devolver CFrame.new(
                c[1], c[2], c[3],
                c[4], c[5], c[6],
                c[7], c[8], c[9],
                c[10], c[11], c[12]
            ), verdadero
        fin
    elseif t == "NumberRange" entonces
        return NumberRange.new(valor.min o 0, valor.max o valor.min o 0), true
    elseif t == "NumberSequence" entonces
        pts locales = {}
        para _, kp en ipairs(value.keypoints o {}) hacer
            pts[#pts + 1] = NumberSequenceKeypoint.new(
                kp.time o 0, kp.value o 0, kp.envelope o 0
            )
        fin
        Si #pts >= 2, entonces devuelve NumberSequence.new(pts), verdadero.
        Si #pts == 1, entonces devuelve NumberSequence.new(pts[1].Value), verdadero.
    elseif t == "ColorSequence" entonces
        pts locales = {}
        para _, kp en ipairs(value.keypoints o {}) hacer
            local c = kp.color o {}
            pts[#pts + 1] = ColorSequenceKeypoint.nuevo(
                kp.tiempo o 0,
                Color3.new(cr o 0, cg o 0, cb o 0)
            )
        fin
        Si #pts >= 2, entonces devuelve ColorSequence.new(pts), verdadero.
        Si #pts == 1, entonces devuelve ColorSequence.new(pts[1].Value), verdadero.
    elseif t == "EnumItem" entonces
        local enumType = Enum[valor.enum o ""]
        si enumType entonces
            local ok, item = pcall(function()
                devolver enumType[valor.nombre o ""]
            fin)
            Si está bien y el artículo, entonces devuelve el artículo, verdadero fin
        fin
    elseif t == "BrickColor" entonces
        local ok, resultado = pcall(BrickColor.new, value.name o "Gris piedra medio")
        Si está bien, entonces devuelve el resultado, verdadero.
    elseif t == "UDim2" entonces
        local x, y = valor.x o {}, valor.y o {}
        devolver UDim2.new(
            x.scale o 0, x.offset o 0,
            escala y o 0, desplazamiento y o 0
        ), verdadero
    elseif t == "Rect" entonces
        local mn, mx = valor.min o {}, valor.max o {}
        devolver Rect.new(
            mn.x o 0, mn.y o 0,
            mx.x o 0, mx.y o 0
        ), verdadero
    elseif t == "PropiedadesPsicológicas" entonces
        local ok, resultado = pcall(
            PropiedadesFísicas.nuevas,
            valor.densidad o .7,
            valor.fricción o .3,
            valor.elasticidad o .5,
            valor.peso de fricción o 1,
            valor.elasticidadPeso o 1
        )
        Si está bien, entonces devuelve el resultado, verdadero.
    elseif t == "InstanceRef" entonces
        valor de retorno, "ref"
    fin

    devolver nil, falso
fin

función local rawCFrame(raw)
    si type(raw) == "table" y raw.t == "CFrame" entonces
        valor local, ok = decodeTyped(raw)
        Si ok == verdadero, entonces devolver valor.
    fin
fin

función local sourceAnchor(nodos)
    local byId = {}
    para _, nodo en ipairs(nodos) hacer byId[nodo.id] = nodo fin

    para _, nodo en ipairs(nodos) hacer
        si node.class == "Model" entonces
            local raw = (node.properties o {}).PrimaryPart
            si type(raw) == "table" y raw.t == "InstanceRef" entonces
                local partNode = porId[raw.id]
                si partNode entonces
                    local cf = rawCFrame((partNode.properties o {}).CFrame)
                    Si cf entonces devolver cf fin
                fin
            fin
        fin
    fin

    mejor local, mejorScore = nil, -1
    para _, nodo en ipairs(nodos) hacer
        Si node.class == "Part" o node.class == "MeshPart" entonces
            local n = string.lower(tostring(node.name or ""))
            Puntuación local = 0
            Si n == "efecto", entonces la puntuación es igual a 100.
            elseif string.find(n, "root", 1, true) then score = 90
            elseif string.find(n, "muerte", 1, true) then score = 80
            elseif string.find(n, "effect", 1, true) then score = 75
            fin
            Si score > bestScore y rawCFrame((node.properties o {}).CFrame) entonces
                mejor, mejorScore = nodo, puntuación
            fin
        fin
    fin

    si es mejor entonces
        return rawCFrame((best.properties o {}).CFrame)
    fin

    para _, nodo en ipairs(nodos) hacer
        Si node.class == "Part" o node.class == "MeshPart" entonces
            local cf = rawCFrame((node.properties or {}).CFrame)
            Si cf entonces devolver cf fin
        fin
    fin

    devolver CFrame.new()
fin

local SALTAR = {
    Padre = verdadero,
    WorldPivot = verdadero,
    PhysicsRepRootRef = verdadero,
    AudioContent = verdadero,
    AnimationContent = verdadero,
    Posición mundial = verdadero,
    Orientación mundial = verdadero,
    WorldCFrame = verdadero,
    WorldAxis = verdadero,
    EjeSegundoMundial = verdadero,
    TransformedWorldCFrame = verdadero,
}

función local setAttributes(obj, attrs)
    para nombre, sin procesar en pares(attrs o {}) hacer
        valor local, ok = decodeTyped(raw)
        si ok == verdadero entonces
            pcall(function() obj:SetAttribute(name, value) end)
        elseif type(raw) ~= "table" then
            pcall(function() obj:SetAttribute(name, raw) end)
        fin
    fin
fin

-- ============================================================
-- Reconstruir el efecto como una PLANTILLA DE ACTIVOS dentro de ReplicatedStorage.
-- Conservamos las transformaciones relativas en lugar de posicionarlo sobre el objeto ficticio;
-- DeathEffectPreview es el responsable de adjuntarlo/renderizarlo.
-- ============================================================
local injectionRoots = {}
local previewInjected = {}
copias de seguridad de vista previa local = {}

función local ensureSkinsFolders()
    locales skins = ReplicatedStorage:FindFirstChild("ReplicatedSkins")
    si no son aspectos entonces
        skins = Instancia.new("Carpeta")
        skins.Name = "Pieles replicadas"
        skins.Parent = ReplicatedStorage
    fin
