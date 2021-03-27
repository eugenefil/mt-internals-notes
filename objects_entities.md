The base class for active server objects (players, entities) is ServerActiveObject (src/server/serveractiveobject.cpp).

## Creating an entity

A new entity created from lua starts off as a bare-bones instance of class LuaEntitySAO (src/server/luaentity_sao.cpp), which inherits from ServerActiveObject. In ServerEnvironment::addActiveObjectRaw() (src/serverenvironment.cpp) the new object is added to the system as follows next.

First, the object is registered w/ server's ActiveObjectMgr, which assigns it an id.

Second, the pointer to object is wrapped into ObjectRef w/ ScriptApiBase::addObjectReference() (src/script/cpp_api/s_base.cpp) and is saved in core.object_refs table under its id as a key.

Next in LuaEntitySAO::addedToEnvironment() the entity itself is created (ScriptApiEntity::luaentity_Add) in lua using the prototype from core.registered\_entities. The entity object in lua stores a reference to its corresponding ObjectRef in 'object' field. The entity object is saved in core.luaentities under its id as key.

Creation of entity in lua from inside c++ and further manipulations w/ it are done using c++ entity api - ScriptApiEntity class (src/script/cpp\_api/s\_entity.cpp), which is only visible to c++.

The initial properties of an entity are stored in the 'initial\_properties' field of its prototype, which becomes its metatable upon creation. These properties are from object definition table. After creating entity in lua they are read back into c++ LuaEntitySAO object (actually its part inherited from UnitSAO) w/ ScriptApiEntity::luaentity\_GetProperties() which calls read\_object\_properties() (src/script/common/c\_content.cpp).

Next lua entity is activated by calling its on_activate.

In the end lua entity, being a static object (i.e. an object that persists when its containing MapBlock is unloaded and must be reloaded on block reload), is added to its MapBlock's list of static objects and stores the block's position.

Summing up, lua entity consists of 2 parts: pure lua object and ObjectRef. The former is made from the prototype registered earlier w/ core.register_entity. The prototype serves as metatable after creation. The former contains a reference to the latter in the 'object' field. The latter is the main api to an object via ObjectRef methods in `src/script/lua_api/l_object.cpp`. The reference to the lua entity from ObjectRef is available via get_luaentity() call.
