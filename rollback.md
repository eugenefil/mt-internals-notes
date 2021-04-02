## Rollback

Rollback functions are available through server's RollbackManager object which implements IRollbackManager interface. RollbackManager uses sqlite database which records actions when `enable_rollback_recording` config option is enabled.

At the moment only players direct actions with nodes and inventories can be reverted. An indirect change such as tnt explosion made by a player cannot. The revert itself is done in Server::rollbackRevertActions().

Rollback recording for dug nodes is done in Map::addNodeAndUpdate(). When reverting player's actions the change being rolled back is recorded itself, so the rollback can itself be rolled back.

The action is recorded by RollbackManager::reportAction(). The action does not supply the actor. Instead the manager has the notion of the current actor which is set using RollbackScopeActor class. A series of code (e.g. node digging, executing a chat command) that does rollback recording is wrapped by a RollbackScopeActor object, which sets the current actor at the start and restores the previous actor at the end.

If the actor is unknown precisely, there is a heuristic algorithm to find a suspect actor based on the list of previous actions. The suspect from some previous action is the more probable the closer in time and space that action took place to the current action.
