# 🚀 Galactic.CC Custom Plugin Development Guide

Create unlimited custom mods for Galactic.CC without modifying the main menu code!

## 📋 Table of Contents
- [Overview](#overview)
- [Quick Start](#quick-start)
- [Plugin Structure](#plugin-structure)
- [API Reference](#api-reference)
- [Examples](#examples)
- [Building & Installation](#building--installation)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

---

## 🎯 Overview

The Galactic.CC plugin system allows developers to create custom mods that seamlessly integrate into the menu. Plugins are loaded dynamically from DLL files, making it easy to share and distribute your creations.

### Key Features
- ✅ **Full Integration** - Your mods appear in the "Custom Plugins" menu category
- ✅ **No Menu Modification** - Create mods without touching the main menu code

---

## 🚀 Quick Start

### Prerequisites
- .NET SDK (for building)
- Galactic.CC.dll (compiled menu)
- Unity DLLs from Gorilla Tag installation
- Basic C# knowledge

### 1. Create Your Project

```bash
# Create a new class library
dotnet new classlib -n MyAwesomePlugin -f netstandard2.1
cd MyAwesomePlugin
```

### 2. Add References

Edit `MyAwesomePlugin.csproj`:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>netstandard2.1</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <!-- Reference to Galactic.CC -->
    <Reference Include="Galactic.CC">
      <HintPath>path\to\Galactic.CC.dll</HintPath>
      <Private>false</Private>
    </Reference>

    <!-- Unity Engine -->
    <Reference Include="UnityEngine">
      <HintPath>path\to\UnityEngine.dll</HintPath>
      <Private>false</Private>
    </Reference>
    
    <Reference Include="UnityEngine.CoreModule">
      <HintPath>path\to\UnityEngine.CoreModule.dll</HintPath>
      <Private>false</Private>
    </Reference>
  </ItemGroup>
</Project>
```

### 3. Create Your Plugin

```csharp
using UnityEngine;
using Galactic.CustomPlugins;

namespace MyAwesomePlugin
{
    public class MyPlugin : ICustomPlugin
    {
        public string PluginName => "My Awesome Plugin";
        public string Version => "1.0.0";
        public string Author => "YourName";
        public string Description => "Does awesome things!";

        public void OnLoad()
        {
            Debug.Log("[MyPlugin] Loaded successfully!");
        }

        public CustomButtonInfo[] GetMods()
        {
            return new CustomButtonInfo[]
            {
                new CustomButtonInfo 
                { 
                    buttonText = "My First Mod", 
                    method = () => MyFirstMod(), 
                    isTogglable = true, 
                    toolTip = "This is my first custom mod!" 
                },
            };
        }

        private void MyFirstMod()
        {
            Debug.Log("[MyPlugin] Mod is running!");
        }
    }
}
```

### 4. Build & Install

```bash
dotnet build -c Release
```

### 5. Test In-Game

1. Launch Gorilla Tag
2. Open Galactic menu
3. Navigate to "Custom Plugins"
4. Your mods should appear!

---

## 🏗️ Plugin Structure

### ICustomPlugin Interface

Every plugin must implement `ICustomPlugin`:

```csharp
public interface ICustomPlugin
{
    string PluginName { get; }      // Display name in menu
    string Version { get; }          // Plugin version
    string Author { get; }           // Your name
    string Description { get; }      // What it does
    
    void OnLoad();                   // Called when plugin loads
    CustomButtonInfo[] GetMods();    // Returns your mod buttons
}
```

### CustomButtonInfo Class

Defines a button in the menu:

```csharp
public class CustomButtonInfo
{
    public string buttonText { get; set; }        // Button label
    public Action method { get; set; }            // Main method
    public Action enableMethod { get; set; }      // Called on enable
    public Action disableMethod { get; set; }     // Called on disable
    public bool isTogglable { get; set; }         // Toggle or button?
    public string toolTip { get; set; }           // Hover description
    public bool enabled { get; set; }             // Start enabled?
}
```

---

## 📚 API Reference

### Button Types

#### 1. Toggle Mod (Runs Every Frame)
```csharp
new CustomButtonInfo 
{ 
    buttonText = "Fly Mod", 
    method = () => FlyMod(), 
    isTogglable = true, 
    toolTip = "Fly around the map" 
}

private void FlyMod()
{

}
```

#### 2. One-Time Button
```csharp
new CustomButtonInfo 
{ 
    buttonText = "Teleport Home", 
    method = () => TeleportHome(), 
    isTogglable = false, 
    toolTip = "Teleport to spawn" 
}

private void TeleportHome()
{

}
```

#### 3. Enable/Disable Methods
```csharp
new CustomButtonInfo 
{ 
    buttonText = "Speed Boost", 
    enableMethod = () => EnableSpeed(), 
    disableMethod = () => DisableSpeed(), 
    isTogglable = true, 
    toolTip = "Run faster" 
}

private void EnableSpeed()
{
}

private void DisableSpeed()
{
}
```

---

## 💡 Examples

### Example 1: Position Saver

```csharp
public class PositionSaver : ICustomPlugin
{
    public string PluginName => "Position Saver";
    public string Version => "1.0.0";
    public string Author => "Developer";
    public string Description => "Save and load your position";

    private Vector3 savedPosition;
    private bool hasSaved = false;

    public void OnLoad() { }

    public CustomButtonInfo[] GetMods()
    {
        return new CustomButtonInfo[]
        {
            new CustomButtonInfo 
            { 
                buttonText = "Save Position", 
                method = () => SavePosition(), 
                isTogglable = false, 
                toolTip = "Save current position" 
            },
            new CustomButtonInfo 
            { 
                buttonText = "Load Position", 
                method = () => LoadPosition(), 
                isTogglable = false, 
                toolTip = "Teleport to saved position" 
            },
        };
    }

    private void SavePosition()
    {
        var player = GorillaLocomotion.Player.Instance;
        if (player != null)
        {
            savedPosition = player.transform.position;
            hasSaved = true;
            Debug.Log($"Position saved: {savedPosition}");
        }
    }

    private void LoadPosition()
    {
        if (!hasSaved)
        {
            Debug.LogWarning("No saved position!");
            return;
        }

        var player = GorillaLocomotion.Player.Instance;
        if (player != null)
        {
            player.transform.position = savedPosition;
            Debug.Log($"Teleported to: {savedPosition}");
        }
    }
}
```

### Example 2: FPS Display

```csharp
public class FPSDisplay : ICustomPlugin
{
    public string PluginName => "FPS Display";
    public string Version => "1.0.0";
    public string Author => "Developer";
    public string Description => "Shows FPS in console";

    private float lastLog = 0f;

    public void OnLoad() { }

    public CustomButtonInfo[] GetMods()
    {
        return new CustomButtonInfo[]
        {
            new CustomButtonInfo 
            { 
                buttonText = "FPS Logger", 
                method = () => LogFPS(), 
                isTogglable = true, 
                toolTip = "Logs FPS every second" 
            },
        };
    }

    private void LogFPS()
    {
        if (Time.time - lastLog >= 1f)
        {
            float fps = 1f / Time.deltaTime;
            Debug.Log($"[FPS] {fps:F1}");
            lastLog = Time.time;
        }
    }
}
```

### Example 3: Player Info

```csharp
public class PlayerInfo : ICustomPlugin
{
    public string PluginName => "Player Info";
    public string Version => "1.0.0";
    public string Author => "Developer";
    public string Description => "Display player information";

    public void OnLoad() { }

    public CustomButtonInfo[] GetMods()
    {
        return new CustomButtonInfo[]
        {
            new CustomButtonInfo 
            { 
                buttonText = "Show Player Count", 
                method = () => ShowPlayerCount(), 
                isTogglable = false, 
                toolTip = "Shows number of players" 
            },
            new CustomButtonInfo 
            { 
                buttonText = "List Players", 
                method = () => ListPlayers(), 
                isTogglable = false, 
                toolTip = "Lists all player names" 
            },
        };
    }

    private void ShowPlayerCount()
    {
        int count = PhotonNetwork.PlayerList.Length;
        Debug.Log($"Players in room: {count}");
    }

    private void ListPlayers()
    {
        Debug.Log("=== Player List ===");
        foreach (var player in PhotonNetwork.PlayerList)
        {
            Debug.Log($"- {player.NickName}");
        }
    }
}
```

---

## 🔨 Building & Installation

### Build Commands

```bash
dotnet build

dotnet build -c Release

dotnet clean

dotnet build -c Release
```

### Installation Path

Copy your compiled DLL to:
```
[Gorilla Tag Directory]/Galactic/Plugins/YourPlugin.dll
```

Example paths:
- **Steam**: `C:\Program Files (x86)\Steam\steamapps\common\Gorilla Tag\Galactic\Plugins\`
- **Oculus**: `C:\Program Files\Oculus\Software\another-axiom-gorilla-tag\Galactic\Plugins\`

### Hot Reload

Use the "Reload Plugins" button in the Custom Plugins menu to reload without restarting!

---

## ✅ Best Practices

### Performance

1. **Keep Toggle Mods Lightweight**
   - They run every frame (60+ times per second)
   - Use frame counters for periodic actions
   ```csharp
   if (Time.frameCount % 60 == 0) // Every ~1 second
   {
   }
   ```

2. **Cache References**
   ```csharp
   private GorillaLocomotion.Player player;
   
   private void EnableMod()
   {
       player = GorillaLocomotion.Player.Instance;
   }
   
   private void ModUpdate()
   {
       if (player != null)
       {
       }
   }
   ```

3. **Use Enable/Disable Methods**
   - Better than running checks every frame
   ```csharp
   // Good
   enableMethod = () => StartCoroutine(),
   disableMethod = () => StopCoroutine()
   
   // Bad
   method = () => { if (enabled) DoStuff(); }
   ```

### Error Handling

Always handle errors gracefully:

```csharp
private void SafeMod()
{
    try
    {
        // Your mod code
    }
    catch (Exception ex)
    {
        Debug.LogError($"[MyPlugin] Error: {ex.Message}");
    }
}
```

### Null Checks

```csharp
var player = GorillaLocomotion.Player.Instance;
if (player == null)
{
    Debug.LogWarning("Player not found!");
    return;
}
```

### Logging

Use descriptive logs with your plugin name:

```csharp
Debug.Log($"[{PluginName}] Mod enabled");
Debug.LogWarning($"[{PluginName}] Something went wrong");
Debug.LogError($"[{PluginName}] Critical error!");
```

---

## 🐛 Troubleshooting

### Plugin Doesn't Load

**Check BepInEx console for errors:**
1. Look for `[Galactic Plugin Loader]` messages

**Common issues:**
- ❌ Wrong .NET version → Use netstandard2.1
- ❌ Missing references → Check Galactic.CC.dll path
- ❌ Wrong folder → Must be in `Galactic/Plugins/`
- ❌ Didn't implement ICustomPlugin → Check interface

### Buttons Don't Appear

```csharp
public CustomButtonInfo[] GetMods()
{
    return new CustomButtonInfo[] { /* your buttons */ };
}
```

### Mods Don't Work

1. **Check for exceptions** in BepInEx console
2. **Add debug logs** to see if methods are called
3. **Test with simple mods first** (like logging)
4. **Verify game objects exist** before accessing them

### Build Errors

**"Could not find Galactic.CC"**
- Update the `<HintPath>` in your .csproj

**"Could not find UnityEngine"**
- Add Unity DLL references from Gorilla Tag installation

**"Type or namespace not found"**
- Make sure you're using `Galactic.CustomPlugins` namespace
- Check that Galactic.CC.dll is referenced

---

## 📦 Distribution

### Sharing Your Plugin

1. Build in Release mode
2. Find DLL in `bin/Release/netstandard2.1/`
3. Share the DLL file
4. Include installation instructions

### Example README for Users

```markdown
# My Awesome Plugin

## Installation
1. Download `MyAwesomePlugin.dll`
2. Copy to `[Gorilla Tag]/Galactic/Plugins/`
3. Launch game
4. Open Galactic menu → Custom Plugins

## Features
- Feature 1
- Feature 2
- Feature 3

## Usage
[Instructions here]
```

---

## 🤝 Contributing

Have questions or want to share your plugins?
- Join the Galactic Discord
- Check out example plugins
- Share your creations with the community!

---

## 📄 License

This plugin system is part of Galactic.CC. Check the main repository for license information.

---

## 🎓 Additional Resources

- [Example Plugin Project](https://github.com/FTSyxcal/Galactic-Custom-Plugins/releases/tag/Galactic-Custom-Plugins)
- [Test Plugin](https://github.com/FTSyxcal/Galactic-Custom-Plugins/releases/tag/Galactic-Custom-Plugins)
- [Unity Scripting Reference](https://docs.unity3d.com/ScriptReference/)
- [Our Discord](https://discord.gg/eQp3PwWkGw)

---

**Happy Modding!**
