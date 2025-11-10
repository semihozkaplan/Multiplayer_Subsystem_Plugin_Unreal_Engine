# Unreal Engine 5: Multiplayer Sessions Plugin

This plugin provides a robust and easy-to-use C++ framework for managing multiplayer sessions in Unreal Engine 5. It is built on a clean, decoupled architecture that separates the core session logic from the UI, making it highly reusable and scalable.

The system uses a `UGameInstanceSubsystem` (`UMultiplayerSessionsSubsystem`) to handle all interactions with Unreal's Online Subsystem (OSS), and includes a sample `UUserWidget` (`UMenu`) that demonstrates how to use the subsystem.

## 🏛️ Architecture

This plugin is built on a "backend" and "frontend" design:

1.  **Backend: `UMultiplayerSessionsSubsystem`**
    * This `UGameInstanceSubsystem` is the core of the plugin. It wraps all the complex, asynchronous logic of the `IOnlineSessionInterface`.
    * It handles creating, finding, and joining sessions.
    * It manages the internal Online Subsystem delegates (`FOnCreateSessionCompleteDelegate`, `FOnFindSessionsCompleteDelegate`, etc.) and their associated `DelegateHandles`.
    * It exposes its own set of custom, broadcastable delegates (e.g., `MultiplayerOnCreateSessionComplete`, `MultiplayerOnFindSessionsComplete`) for other classes to subscribe to.

2.  **Frontend: `UMenu` (Example UI)**
    * This `UUserWidget` class serves as a working example of how to implement the subsystem.
    * It does **not** contain any direct OSS logic. Instead, it gets the `UMultiplayerSessionsSubsystem` from the Game Instance.
    * In its `MenuSetup` function, it binds its own callback functions (like `OnCreateSession`, `OnFindSessions`) to the subsystem's custom delegates.
    * When the "Host" or "Join" buttons are clicked, it simply calls the subsystem's public functions (`CreateSession`, `FindSessions`).
    * When the subsystem finishes an operation, it broadcasts its delegate, which the `UMenu` class receives, allowing it to update the UI or travel to the lobby.

## 🚀 Features

* **Host Session:** Create a new online session.
    * Handles destroying an existing session first if one is found.
    * Configures session settings like `NumPublicConnections`, `MatchType`, and `bIsLANMatch` (based on the "NULL" subsystem).
* **Find Sessions:** Search for active game sessions.
    * Configures search parameters like `MaxSearchResults` and `bIsLanQuery`.
* **Join Session:** Join a selected session from the search results.
    * Filters search results by `MatchType` to find a compatible game.
* **Destroy Session:** Cleanly tears down a session.
* **Delegate-Driven:** Uses custom delegates for clean, asynchronous event handling.
* **UI Included:** Comes with a complete C++ `UUserWidget` (`UMenu`) that is ready to be used in a Widget Blueprint.
* **Input Mode Management:** Correctly switches between `FInputModeUIOnly` and `FInputModeGameOnly` when the menu is created and destroyed.
* **Travel Logic:** Handles both `ServerTravel` for the host (on session creation) and `ClientTravel` for the client (on session join).

## 🔧 Setup & Configuration

1.  **Install Plugin:**
    * Place the `MultiplayerSessions` folder into your project's `Plugins` directory.

2.  **Enable an Online Subsystem:**
    * This plugin relies on an active OSS. Go to **Edit > Plugins** and enable an appropriate subsystem, such as **Online Subsystem Steam** or **Online Subsystem NULL** (for LAN testing).

3.  **Configure `DefaultEngine.ini`:**
    * You must tell Unreal Engine to use your chosen subsystem. Open `Config/DefaultEngine.ini` and add the following sections.

    * **For LAN (NULL Subsystem):**
        ```ini
        [OnlineSubsystem]
        DefaultPlatformService=NULL
        ```

    * **For Steam:**
        ```ini
        [OnlineSubsystem]
        DefaultPlatformService=Steam

        [OnlineSubsystemSteam]
        bEnabled=true
        SteamDevAppId=480
        ; Use your own App ID if you have one
        ```

4.  **Generate & Build:**
    * Right-click your `.uproject` file and "Generate Visual Studio project files."
    * Open the solution in Visual Studio and build the project.

## 💻 How to Use

This plugin is designed to be used with the included `UMenu` widget.

1.  **Create a Widget Blueprint:**
    * In the Unreal Editor, right-click in the Content Browser and select **User Interface > Widget Blueprint**.
    * In the "Pick Parent Class" dialog, search for and select **`Menu`**.
    * Name this new Widget Blueprint (e.g., `WBP_MainMenu`).

2.  **Design the Widget:**
    * Open `WBP_MainMenu`.
    * Go to the **Graph** view. In the **Class Defaults** panel, you will **not** see the `HostButton` or `JoinButton` variables.
    * Switch to the **Designer** view.
    * Drag two **Button** widgets from the palette onto the canvas.
    * Name the first button **`HostButton`** (exactly).
    * Name the second button **`JoinButton`** (exactly).
    * The C++ parent class `UMenu` will automatically find and bind these buttons because of the `meta = (BindWidget)` tag.

3.  **Display the Menu:**
    * In a location like your Player Controller or Game Mode, create the widget and call `MenuSetup`.

    * **Blueprint Example (in a Player Controller):**
        ```blueprint
        [Event BeginPlay]
        -->
        [Create Widget]
            Class: WBP_MainMenu
            Owning Player: [Get Self]
        -->
        [Set (local var 'MainMenu')] --> [Menu Setup]
                                        Target: (local var 'MainMenu')
                                        Number Of Public Conections: 4
                                        Type Of Match: "FreeForAll"
                                        Lobby Path: "/Game/Maps/LobbyMap" (Your map path)
        ```

4.  **You're Done!**
    * The `MenuSetup` function will handle adding the widget to the viewport and setting the input mode.
    * When the user clicks the "Host" button, the C++ code will call `HostButtonClicked()`, which triggers the `UMultiplayerSessionsSubsystem`.
    * The subsystem will broadcast its delegate, and the `UMenu`'s `OnCreateSession` callback will fire, automatically traveling to the lobby map.
