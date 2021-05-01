## ItemStack

ItemStack api is exported via LuaItemStack class in src/script/lua_api/l_item.cpp. ItemStack() function in Lua is a lua_register()'ed LuaItemStack::create_object(). From itemstring it creates a userdata which holds a pointer to a new LuaItemStack object, which itself holds a struct ItemStack (src/inventory.h). Itemstring is parsed by read_item() from src/script/common/c_content.cpp.

**TODO** why store LuaItemStack (that contains ItemStack) inside userdata and not just directly ItemStack struct?

Registration of ItemStack api in lua is done by LuaItemStack::Register() and consists of 2 steps:

* Creating metatable used for new ItemStack userdata values. Metatable, roughly speaking, consists of 2 parts: metatable itself and a method table populated by ItemStack api methods (e.g. get\_name, get\_metadata). Metatable's \__metatable and \__index fields point to that method table, so metatable itself is not visible from lua. Metatable is stored in lua's registry w/ the LuaItemStack key.

* Registering LuaItemStack::create_object() as ItemStack() lua function. This function creates ItemStack userdata values from itemstrings and sets their metatable.

Every item has stack_max property (see struct ItemDefinition in src/itemdef.h), that limits the number of such items that you can put into a stack.

## Inventory

The inventory is represented by InvRef object in lua and is created by InvRef::create (src/script/lua_api/l_inventory.cpp). InvRef doesn't hold the Inventory object, but an InventoryLocation, which is used to get Inventory through a call to ServerInventoryManager::getInventory().

**TODO** Is this done to not keep refs to InventoryObjects around in lua, but to get them when needed (i.e. InvRef::l_add_item) and keep only for the duration of the operation?

Inventory object doesn't deal w/ ItemStack objects directly. Instead it stores InventoryLists, which in turn store ItemStacks. There are 4 lists:

* "main" stores the inventory of the game.
* "craft" stores the items on the crafting grid.
* "craftpreview" is a one-item list that stores the preview result of crafting.
* "craftresult" is a one-item hidden list for internal usage by engine...

**TODO** find out and describe how craftresult list is used

List contents can be returned w/ `<inventory_obj>:get_list(<list_name>)` for a specific list or `<inventory_obj>:get_lists()` to retrieve all lists. List contents can be set w/ `set_list()` and `set_lists()` functions. See InvRef API.
