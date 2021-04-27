## Shutdown

The server has the Server::ShutdownState object, which remembers the shutdown request (along with delay, message and reconnect suggestion for clients) upon the Server::requestShutdown() call. The server main loop continuously checks this object via Server::isShutdownRequested() and stops if that returns true. When server object is later destroyed, it kicks all players by sending access denied packet with the shutdown reason, message and reconnect flag set to true.

The client creates the ClientLauncher object in its main(). The launcher makes `the_game()` call, which creates and runs the Game object (src/client/game.cpp). The launcher passes the pointers to error message and reconnect flag to the Game object. After the client receives from server the access denied packet with shutdown reason, message and reconnect flag set to true, the game object polls that from client object in its main loop, copies the flag and message values over to launcher's and shuts down. The launcher then detects the error and that reconnect was requested and passes it down to formspecs menu handling lua code, which shows the access denied error along with the message and Reconnect button.

## Bans

Banning is provided to mods through server lua api (`src/script/lua_api/l_server.cpp`) functions: `ban_player`, `unban_player`, etc. Those functions call the Server class functions (setIpBanned(), unsetIpBanned(), etc) which use the BanManager object (`src/ban.cpp`) to manage ban data. BanManager is a wrapper around ip => player_name string map. Bans are saved into a plain text file in the world directory. Bans are enforced by ip each time the packet arrives from the client.

## Chat

Lua's `chat_send_all()` and `chat_send_player()` both employ Server::SendChatMessage() under the hood, but the former uses `PEER_ID_INEXISTENT` as player id, which is interpreted as "send to all peers".

### Chat commands

Chat commands are registered with `core.register_chatcommand()` defined in builtin/common/chatcommands.lua. Command definitions are stored in core.registered_chatcommands.

All chat messages are processed and commands are run by a single callback registered with `core.register_on_chat_message()` in builtin/game/chat.lua. After registration callback is stored in `core.registered_on_chat_messages` defined in builtin/game/register.lua. All such callbacks are called by `ScriptApiServer::on_chat_message()` in `src/script/cpp_api/s_server.cpp`.

`core.registered_on_chat_messages` (list of callbacks), `core.register_on_chat_message` (callback registration func) are created by `make_registration()` in builtin/game/register.lua. Many (all?) other callbacks (e.g. `register_globalstep`, `register_on_mods_loaded`, `register_on_punchnode`) are done there the same way.


## Auth

Auth API is defined in builtin/game/auth.lua. Auth backend is implemented by a handler. By default, there is `core.builtin_auth_handler`, which can be overridden with `core.register_authentication_handler()`. `core.get_auth_handler()` returns current auth handler, preferring the registered one, but defaulting to builtin, if there is none.

Builtin auth handler itself uses API provided by lua core.auth object (`src/script/lua_api/l_auth.cpp`). For credentials storage core.auth utilizes the database object returned by ServerEnvironment::getAuthDatabase(), which by default is an sqlite3 database implemented by AuthDatabaseSQLite3 class (src/database/database-sqlite3.cpp), but can also be PostgreSQL, etc.

**TODO** describe db backend selection (see reading world.mt in ServerEnvironment ctor)
**TODO** how exactly sqlite3 becomes default?
