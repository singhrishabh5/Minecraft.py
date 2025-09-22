from ursina import *
from ursina.prefabs.first_person_controller import FirstPersonController
from random import randint

app = Ursina()

window.title = 'Ultimate Mini-Minecraft - Ursina'
window.borderless = False
window.fullscreen = False
window.exit_button.visible = True
window.fps_counter.enabled = True

# --- Player ---
player = FirstPersonController()
player.gravity = 0.5
player.speed = 8
player.cursor.visible = True

# --- Sky and Lighting ---
Sky()
DirectionalLight(y=2, rotation=(45, -45, 45), shadows=False)

# --- Block Colors (No Textures Needed) ---
block_colors = {
    'grass': color.green,
    'dirt': color.brown,
    'stone': color.gray,
    'wood': color.Color(0.3, 0.15, 0, 1),  # Dark brown (RGBA - added alpha=1)
    'leaves': color.Color(0, 0.4, 0, 1),   # Dark green (RGBA - added alpha=1)
    'sand': color.yellow,
    'water': color.Color(0, 0, 1, 0.7)     # Blue with transparency
}

# --- World Storage ---
blocks = {}
chunk_size = 16
MAX_BLOCKS = 2000

# --- Hotbar ---
hotbar = ['grass', 'dirt', 'stone', 'wood', 'leaves', 'sand', 'water']
selected_block_index = 0

# --- Terrain Generation ---
def generate_flat_chunk(start_x=0, start_z=0):
    if len(blocks) > MAX_BLOCKS:
        return
        
    for x in range(start_x, start_x + chunk_size):
        for z in range(start_z, start_z + chunk_size):
            y = randint(0, 1)
            pos = (x, y, z)
            
            # Create ground block WITH COLOR
            block_color = block_colors['grass'] if y == 1 else block_colors['dirt']
            block = Entity(
                model='cube',
                color=block_color,
                position=pos,
                collider='box'
            )
            blocks[pos] = block

            # Add water below y=0
            if y == 0:
                water_pos = (x, -1, z)
                water_block = Entity(
                    model='cube',
                    color=block_colors['water'],
                    position=water_pos,
                    collider='box'
                )
                blocks[water_pos] = water_block

            # Random tree
            if y == 1 and randint(0, 20) > 18 and len(blocks) < MAX_BLOCKS - 30:
                generate_tree(x, y + 1, z)

# --- Tree Generation ---
def generate_tree(x, y, z):
    if len(blocks) > MAX_BLOCKS - 10:
        return
        
    height = randint(3, 4)
    
    # Trunk
    for i in range(height):
        pos = (x, y + i, z)
        trunk = Entity(model='cube', color=block_colors['wood'], position=pos, collider='box')
        blocks[pos] = trunk

    # Leaves
    leaf_count = 0
    max_leaves = 12
    
    for lx in range(-2, 3):
        for ly in range(height - 1, height + 2):
            for lz in range(-2, 3):
                if leaf_count >= max_leaves or len(blocks) >= MAX_BLOCKS:
                    return
                    
                if abs(lx) + abs(lz) < 3 and ly < height + 2:
                    pos = (x + lx, y + ly, z + lz)
                    if pos not in blocks:
                        leaf = Entity(
                            model='cube', 
                            color=block_colors['leaves'], 
                            position=pos, 
                            collider=None
                        )
                        blocks[pos] = leaf
                        leaf_count += 1

# Generate initial chunk
generate_flat_chunk(0, 0)
print(f"Generated {len(blocks)} blocks")

# --- Input Handling ---
def input(key):
    global selected_block_index
    
    try:
        if key == 'left mouse down' or key == 'right mouse down':
            if mouse.hovered_entity:
                block = mouse.hovered_entity
                pos = block.position
                
                if key == 'left mouse down' and mouse.normal:
                    new_pos = pos + mouse.normal
                    new_pos = (round(new_pos.x), round(new_pos.y), round(new_pos.z))
                    
                    if (new_pos not in blocks and len(blocks) < MAX_BLOCKS):
                        block_type = hotbar[selected_block_index]
                        new_block = Entity(
                            model='cube', 
                            color=block_colors[block_type],
                            position=new_pos, 
                            collider='box'
                        )
                        blocks[new_pos] = new_block
                        
                elif key == 'right mouse down':
                    pos_tuple = (round(pos.x), round(pos.y), round(pos.z))
                    if pos_tuple in blocks:
                        destroy(blocks[pos_tuple])
                        del blocks[pos_tuple]

        # Hotbar scrolling
        if key == 'scroll up':
            selected_block_index = (selected_block_index + 1) % len(hotbar)
            update_hotbar()
        elif key == 'scroll down':
            selected_block_index = (selected_block_index - 1) % len(hotbar)
            update_hotbar()
            
        # Number keys for hotbar
        if key in '1234567':
            index = int(key) - 1
            if index < len(hotbar):
                selected_block_index = index
                update_hotbar()
                
    except Exception as e:
        print(f"Input error: {e}")

# --- Update Function ---
def update():
    if held_keys['shift']:
        player.speed = 12
    else:
        player.speed = 8

# --- Hotbar UI (Now with colors) ---
hotbar_ui = []
for i, block_name in enumerate(hotbar):
    icon = Entity(
        parent=camera.ui, 
        model='quad', 
        color=block_colors[block_name],
        scale=0.06, 
        x=-0.3 + i * 0.1, 
        y=-0.42
    )
    hotbar_ui.append(icon)

def update_hotbar():
    for i, icon in enumerate(hotbar_ui):
        if i == selected_block_index:
            icon.scale = 0.07
        else:
            icon.scale = 0.06

update_hotbar()

print("Game started with colored blocks!")
app.run()# Minecraft.py
USing python you can create your own minecraftworld
