## `repair_block_light()`

### STEP 1

Get sunlight propagating from lowest node layer of the above block as a boolean [MAP_BLOCKSIZE,MAP_BLOCKSIZE] mask w/ `is_sunlight_above_block()`.

Propagate this mask down through the target block w/ `fill_with_sunlight()`. Set day light (`LIGHTBANK_DAY`) of each sunlit propagating node to full sunlight value (`LIGHT_SUN`), otherwise to 0. Set night light (LIGHTBANK_NIGHT) of each node to 0. At the end the mask contains the sunlight/shadow for the below block.

Propagate the mask further down through the below block w/ propagate_block_sunlight(). But this time don't reset the light for every node. Instead propagate down column-wise until the node w/ the expected light value is found, i.e. only propagate where the light needs to be fixed. Move down fixing lower blocks until there are no nodes to fix.

### STEP 2

TODO Why we do this step?

Add each exterior node (i.e. node on the face) of the target block to unlight queue, if it's dimmer than LIGHT_SUN. Do it for both day and night light banks.
