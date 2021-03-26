Emerge is controlled by [EmergeManager](https://github.com/minetest/minetest/blob/5.4.0/src/emerge.h#L113), which employes [EmergeThread](https://github.com/minetest/minetest/blob/5.4.0/src/emerge.cpp#L48)s to do the actual work.

Each thread has a queue of blocks to emerge. Blocks are queued by [EmergeManager::enqueueBlockEmerge](https://github.com/minetest/minetest/blob/5.4.0/src/emerge.cpp#L305) or [EmergeManager::enqueueBlockEmergeEx](https://github.com/minetest/minetest/blob/5.4.0/src/emerge.cpp#L321) from different parts of the engine.

BLOCK_EMERGE_FORCE_QUEUE flag means ignore queue limits when enqueueing blocks.

Note, that threads do not store information (emerge completion callbacks, flags, etc) about the block to emerge, just its position. That information is stored by [EmergeManager::pushBlockEmergeData()](https://github.com/minetest/minetest/blob/5.4.0/src/emerge.cpp#L321) in EmergeManager's hashmap of enqueued blocks keyed by block position.

In thread's main [loop](https://github.com/minetest/minetest/blob/5.4.0/src/emerge.cpp#L657) a block is popped from the queue and emerged by [EmergeThread::getBlockOrStartGen()](https://github.com/minetest/minetest/blob/5.4.0/src/emerge.cpp#L575), which tries the following methods in succession and moves to next method if previous fails:

1. Get the block from memory.
2. Load the block from database.
3. Generate the block's chunk with mapgen.

Emerged blocks are passed to server's RemoteClient objects by [Server::SetBlocksNotSent()](https://github.com/minetest/minetest/blob/5.4.0/src/server.cpp#L1213) to be later sent over the wire.
