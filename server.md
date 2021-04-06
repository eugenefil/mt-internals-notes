# Shutdown

The server has the Server::ShutdownState object, which remembers the shutdown request (along with delay, message and reconnect suggestion for clients) upon the Server::requestShutdown() call. The server main loop continuously checks this object via Server::isShutdownRequested() and stops if that returns true. When server object is later destroyed, it kicks all players by sending access denied packet with the shutdown reason, message and reconnect flag set to true.

The client creates the ClientLauncher object in its main(). The launcher makes `the_game()` call, which creates and runs the Game object (src/client/game.cpp). The launcher passes the pointers to error message and reconnect flag to the Game object. After the client receives from server the access denied packet with shutdown reason, message and reconnect flag set to true, the game object polls that from client object in its main loop, copies the flag and message values over to launcher's and shuts down. The launcher then detects the error and that reconnect was requested and passes it down to formspecs menu handling lua code, which shows the access denied error along with the message and Reconnect button.

# Banning

Banning is provided to mods through server lua api (`src/script/lua_api/l_server.cpp`) functions: `ban_player`, `unban_player`, etc. Those functions call the Server class functions (setIpBanned(), unsetIpBanned(), etc) which use the BanManager object (`src/ban.cpp`) to manage ban data. BanManager is a wrapper around ip => player_name string map. Bans are saved into a plain text file in the world directory. Bans are enforced by ip each time the packet arrives from the client.
