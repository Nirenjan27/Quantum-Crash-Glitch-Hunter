# Quantum-Crash-Glitch-Hunter
import pygame
import random
import sys
import math

# Initialize game
pygame.init()
WIDTH, HEIGHT = 800, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Quantum Crash – The Multiverse Maintenance Manual")
clock = pygame.time.Clock()

# Colors and fonts
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
GREEN = (0, 255, 0)
BLUE = (0, 0, 255)
YELLOW = (255, 255, 0)
PURPLE = (180, 0, 220)
CYAN = (0, 255, 255)
ORANGE = (255, 165, 0)
FONT = pygame.font.SysFont("Arial", 28)
BIGFONT = pygame.font.SysFont("Arial", 40)
SMALLFONT = pygame.font.SysFont("Arial", 16)

# Particle system
class Particle:
    def __init__(self, x, y, color, velocity, life):
        self.x = x
        self.y = y
        self.color = color
        self.velocity = velocity
        self.life = life
        self.max_life = life
        self.size = random.randint(2, 6)
    
    def update(self):
        self.x += self.velocity[0]
        self.y += self.velocity[1]
        self.life -= 1
        # Fade out
        alpha = int(255 * (self.life / self.max_life))
        self.color = (*self.color[:3], max(0, alpha))
    
    def draw(self, surface):
        if self.life > 0:
            # Create a surface with per-pixel alpha
            particle_surf = pygame.Surface((self.size * 2, self.size * 2), pygame.SRCALPHA)
            alpha = int(255 * (self.life / self.max_life))
            color_with_alpha = (*self.color[:3], alpha)
            pygame.draw.circle(particle_surf, color_with_alpha, (self.size, self.size), self.size)
            surface.blit(particle_surf, (self.x - self.size, self.y - self.size))

# Player setup
player = pygame.Rect(WIDTH // 2, HEIGHT - 80, 50, 50)
bat = pygame.Rect(player.x + 20, player.y - 60, 10, 60)
glitches = []
particles = []
score = 0
boss_mode = False
boss_hp = 10
max_boss_hp = 10
galaxy_mode = False
galaxy_level = 1
max_galaxy_level = 5
galaxy_enemies = []
game_over = False
victory = False
player_trail = []
screen_shake = 0
background_stars = []
glitch_spawn_timer = 0

# Create background stars
for _ in range(100):
    star_x = random.randint(0, WIDTH)
    star_y = random.randint(0, HEIGHT)
    star_brightness = random.randint(50, 255)
    background_stars.append([star_x, star_y, star_brightness])

# Glitch thoughts (surreal prompts)
questions = [
    "Are you playing the game, or is it playing you?",
    "Would you delete yourself to fix the code?",
    "If glitches are reality, what is truth?",
    "The cricket bat remembers everything.",
]

galaxy_messages = [
    f"🌌 GALAXY LEVEL {galaxy_level} 🌌",
    "The universe bends to your will!",
    "Reality fragments across dimensions!",
    "You are becoming one with the cosmos!",
    "The final frontier awaits..."
]

def create_particles(x, y, color, count=10):
    for _ in range(count):
        velocity = (random.randint(-3, 3), random.randint(-3, 3))
        life = random.randint(20, 40)
        particles.append(Particle(x, y, color, velocity, life))

def create_explosion(x, y, color, count=20):
    # Main explosion particles
    for _ in range(count):
        angle = random.uniform(0, 2 * math.pi)
        speed = random.uniform(2, 8)
        velocity = (math.cos(angle) * speed, math.sin(angle) * speed)
        life = random.randint(30, 60)
        particles.append(Particle(x, y, color, velocity, life))
    
    # Add sparkle particles for extra polish
    for _ in range(count // 2):
        angle = random.uniform(0, 2 * math.pi)
        speed = random.uniform(1, 4)
        velocity = (math.cos(angle) * speed, math.sin(angle) * speed)
        life = random.randint(40, 80)
        sparkle_color = (255, 255, 255)  # White sparkles
        particles.append(Particle(x, y, sparkle_color, velocity, life))

def draw_player_with_glow(surface, rect, color):
    # Draw glow effect
    glow_surf = pygame.Surface((rect.width + 20, rect.height + 20), pygame.SRCALPHA)
    glow_color = (*color, 100)
    pygame.draw.rect(glow_surf, glow_color, (0, 0, rect.width + 20, rect.height + 20), border_radius=10)
    surface.blit(glow_surf, (rect.x - 10, rect.y - 10))
    
    # Draw main player
    pygame.draw.rect(surface, color, rect, border_radius=5)
    # Add highlight
    highlight_rect = pygame.Rect(rect.x + 5, rect.y + 5, rect.width - 10, rect.height // 3)
    highlight_color = tuple(min(255, c + 50) for c in color)
    pygame.draw.rect(surface, highlight_color, highlight_rect, border_radius=3)

def draw_glitch_with_effects(surface, rect):
    # Main glitch body
    colors = [CYAN, (0, 255, 100), (100, 255, 255)]
    main_color = random.choice(colors)
    
    # Glitch effect - multiple overlapping rectangles
    for i in range(3):
        offset_x = random.randint(-2, 2)
        offset_y = random.randint(-2, 2)
        glitch_rect = pygame.Rect(rect.x + offset_x, rect.y + offset_y, rect.width, rect.height)
        color = (*main_color, 150 - i * 30)
        
        glitch_surf = pygame.Surface((rect.width, rect.height), pygame.SRCALPHA)
        pygame.draw.rect(glitch_surf, color, (0, 0, rect.width, rect.height))
        surface.blit(glitch_surf, (glitch_rect.x, glitch_rect.y))
    
    # Add scanlines
    for y in range(rect.y, rect.y + rect.height, 4):
        pygame.draw.line(surface, (255, 255, 255, 100), (rect.x, y), (rect.x + rect.width, y), 1)

def draw_bat_with_shine(surface, rect):
    # Bat shadow
    shadow_rect = pygame.Rect(rect.x + 2, rect.y + 2, rect.width, rect.height)
    pygame.draw.rect(surface, (150, 150, 0), shadow_rect, border_radius=3)
    
    # Main bat
    pygame.draw.rect(surface, YELLOW, rect, border_radius=3)
    
    # Shine effect
    shine_rect = pygame.Rect(rect.x + 2, rect.y + 2, rect.width - 4, rect.height // 4)
    pygame.draw.rect(surface, (255, 255, 200), shine_rect, border_radius=2)

def draw_galaxy_enemy(surface, rect, enemy_type):
    if enemy_type == "star":
        # Draw a star-shaped enemy
        center_x, center_y = rect.centerx, rect.centery
        points = []
        for i in range(10):
            angle = i * math.pi / 5
            if i % 2 == 0:
                radius = rect.width // 2
            else:
                radius = rect.width // 4
            x = center_x + radius * math.cos(angle)
            y = center_y + radius * math.sin(angle)
            points.append((x, y))
        pygame.draw.polygon(surface, (255, 215, 0), points)
        pygame.draw.polygon(surface, (255, 255, 255), points, 2)
    elif enemy_type == "comet":
        # Draw a comet with tail
        pygame.draw.circle(surface, (0, 191, 255), rect.center, rect.width // 2)
        # Comet tail
        tail_points = [
            (rect.centerx, rect.centery),
            (rect.centerx - 20, rect.centery - 10),
            (rect.centerx - 30, rect.centery),
            (rect.centerx - 20, rect.centery + 10)
        ]
        pygame.draw.polygon(surface, (173, 216, 230), tail_points)
    else:  # asteroid
        # Draw an irregular asteroid
        pygame.draw.circle(surface, (139, 69, 19), rect.center, rect.width // 2)
        pygame.draw.circle(surface, (160, 82, 45), (rect.centerx - 5, rect.centery - 5), rect.width // 3)

def draw_victory_boy_with_bat(surface, x, y, scale=1.0):
    # Boy's body
    body_width = int(40 * scale)
    body_height = int(60 * scale)
    body_rect = pygame.Rect(x, y, body_width, body_height)
    
    # Head
    head_radius = int(15 * scale)
    head_center = (x + body_width // 2, y - head_radius)
    pygame.draw.circle(surface, (255, 220, 177), head_center, head_radius)  # Skin color
    
    # Hair
    hair_rect = pygame.Rect(x + int(5 * scale), y - int(25 * scale), int(30 * scale), int(15 * scale))
    pygame.draw.ellipse(surface, (139, 69, 19), hair_rect)  # Brown hair
    
    # Eyes
    left_eye = (x + int(10 * scale), y - int(10 * scale))
    right_eye = (x + int(25 * scale), y - int(10 * scale))
    pygame.draw.circle(surface, BLACK, left_eye, int(2 * scale))
    pygame.draw.circle(surface, BLACK, right_eye, int(2 * scale))
    
    # Smile
    smile_start = (x + int(12 * scale), y - int(5 * scale))
    smile_end = (x + int(23 * scale), y - int(5 * scale))
    pygame.draw.arc(surface, BLACK, (x + int(10 * scale), y - int(10 * scale), int(20 * scale), int(10 * scale)), 0, math.pi, 2)
    
    # Body (shirt)
    pygame.draw.rect(surface, (0, 100, 200), body_rect)  # Blue shirt
    
    # Arms
    left_arm = pygame.Rect(x - int(15 * scale), y + int(10 * scale), int(15 * scale), int(30 * scale))
    right_arm = pygame.Rect(x + body_width, y + int(10 * scale), int(15 * scale), int(30 * scale))
    pygame.draw.rect(surface, (255, 220, 177), left_arm)  # Left arm
    pygame.draw.rect(surface, (255, 220, 177), right_arm)  # Right arm
    
    # Legs
    left_leg = pygame.Rect(x + int(5 * scale), y + body_height, int(12 * scale), int(40 * scale))
    right_leg = pygame.Rect(x + int(23 * scale), y + body_height, int(12 * scale), int(40 * scale))
    pygame.draw.rect(surface, (0, 0, 139), left_leg)  # Dark blue pants
    pygame.draw.rect(surface, (0, 0, 139), right_leg)
    
    # Shoes
    left_shoe = pygame.Rect(x + int(2 * scale), y + body_height + int(35 * scale), int(18 * scale), int(8 * scale))
    right_shoe = pygame.Rect(x + int(20 * scale), y + body_height + int(35 * scale), int(18 * scale), int(8 * scale))
    pygame.draw.rect(surface, BLACK, left_shoe)
    pygame.draw.rect(surface, BLACK, right_shoe)
    
    # Cricket bat in hands
    bat_x = x + body_width + int(5 * scale)
    bat_y = y - int(10 * scale)
    bat_width = int(8 * scale)
    bat_height = int(80 * scale)
    bat_rect = pygame.Rect(bat_x, bat_y, bat_width, bat_height)
    
    # Bat handle
    handle_rect = pygame.Rect(bat_x, bat_y + int(60 * scale), bat_width, int(20 * scale))
    pygame.draw.rect(surface, (139, 69, 19), handle_rect)  # Brown handle
    
    # Bat blade
    blade_rect = pygame.Rect(bat_x, bat_y, bat_width, int(60 * scale))
    pygame.draw.rect(surface, (222, 184, 135), blade_rect)  # Light wood color
    
    # Bat details
    for i in range(3):
        line_y = bat_y + int(15 * scale) + i * int(15 * scale)
        pygame.draw.line(surface, (160, 82, 45), (bat_x, line_y), (bat_x + bat_width, line_y), 1)
    
    # Victory sparkles around the boy
    sparkle_time = pygame.time.get_ticks()
    for i in range(8):
        angle = (sparkle_time / 200 + i * math.pi / 4) % (2 * math.pi)
        sparkle_x = x + body_width // 2 + int(math.cos(angle) * 50 * scale)
        sparkle_y = y + body_height // 2 + int(math.sin(angle) * 50 * scale)
        pygame.draw.circle(surface, (255, 255, 0), (sparkle_x, sparkle_y), int(3 * scale))
        pygame.draw.circle(surface, (255, 255, 255), (sparkle_x, sparkle_y), int(2 * scale))

def draw_hud_with_effects(surface):
    # Score with glow
    score_text = FONT.render(f"Reality Score: {score}", True, WHITE)
    glow_text = FONT.render(f"Reality Score: {score}", True, CYAN)
    
    # Draw glow
    for offset in [(1, 1), (-1, -1), (1, -1), (-1, 1)]:
        surface.blit(glow_text, (20 + offset[0], 20 + offset[1]))
    surface.blit(score_text, (20, 20))
    
    # Health bar for boss
    if boss_mode:
        bar_width = 300
        bar_height = 20
        bar_x = WIDTH // 2 - bar_width // 2
        bar_y = 50
        
        # Background
        pygame.draw.rect(surface, (50, 50, 50), (bar_x - 2, bar_y - 2, bar_width + 4, bar_height + 4))
        pygame.draw.rect(surface, BLACK, (bar_x, bar_y, bar_width, bar_height))
        
        # Health
        health_width = int(bar_width * (boss_hp / max_boss_hp))
        health_color = RED if boss_hp < 3 else ORANGE if boss_hp < 6 else GREEN
        pygame.draw.rect(surface, health_color, (bar_x, bar_y, health_width, bar_height))
        
        # Boss text
        boss_text = BIGFONT.render("⚔️ Face Your Past Self", True, RED)
        text_rect = boss_text.get_rect(center=(WIDTH // 2, 30))
        surface.blit(boss_text, text_rect)
    
    # Galaxy mode HUD
    if galaxy_mode:
        galaxy_text = BIGFONT.render(f"🌌 GALAXY {galaxy_level}/{max_galaxy_level} 🌌", True, (255, 215, 0))
        text_rect = galaxy_text.get_rect(center=(WIDTH // 2, 30))
        surface.blit(galaxy_text, text_rect)
        
        # Galaxy progress bar
        bar_width = 200
        bar_height = 15
        bar_x = WIDTH // 2 - bar_width // 2
        bar_y = 60
        
        pygame.draw.rect(surface, (50, 50, 50), (bar_x - 2, bar_y - 2, bar_width + 4, bar_height + 4))
        pygame.draw.rect(surface, BLACK, (bar_x, bar_y, bar_width, bar_height))
        
        progress_width = int(bar_width * (galaxy_level / max_galaxy_level))
        galaxy_colors = [(255, 0, 255), (0, 255, 255), (255, 255, 0), (255, 165, 0), (255, 0, 0)]
        progress_color = galaxy_colors[min(galaxy_level - 1, len(galaxy_colors) - 1)]
        pygame.draw.rect(surface, progress_color, (bar_x, bar_y, progress_width, bar_height))

def update_background():
    # Animate background stars
    for star in background_stars:
        star[1] += 0.5
        if star[1] > HEIGHT:
            star[1] = 0
            star[0] = random.randint(0, WIDTH)

def draw_background(surface):
    # Gradient background
    for y in range(HEIGHT):
        color_ratio = y / HEIGHT
        r = int(20 + color_ratio * 10)
        g = int(20 + color_ratio * 15)
        b = int(40 + color_ratio * 20)
        pygame.draw.line(surface, (r, g, b), (0, y), (WIDTH, y))
    
    # Draw stars
    for star in background_stars:
        brightness = star[2]
        color = (brightness, brightness, brightness)
        pygame.draw.circle(surface, color, (int(star[0]), int(star[1])), 1)

def apply_screen_shake():
    global screen_shake
    if screen_shake > 0:
        shake_x = random.randint(-screen_shake, screen_shake)
        shake_y = random.randint(-screen_shake, screen_shake)
        screen_shake -= 1
        return shake_x, shake_y
    return 0, 0

def draw():
    global screen_shake
    
    # Apply screen shake
    shake_x, shake_y = apply_screen_shake()
    
    # Draw background
    draw_background(screen)
    
    # Player trail effect
    player_trail.append((player.x + 25, player.y + 25))
    if len(player_trail) > 10:
        player_trail.pop(0)
    
    for i, pos in enumerate(player_trail):
        alpha = int(255 * (i / len(player_trail)) * 0.3)
        trail_surf = pygame.Surface((10, 10), pygame.SRCALPHA)
        pygame.draw.circle(trail_surf, (*PURPLE, alpha), (5, 5), 5)
        screen.blit(trail_surf, (pos[0] - 5 + shake_x, pos[1] - 5 + shake_y))

    # Player and bat with effects
    draw_player_with_glow(screen, pygame.Rect(player.x + shake_x, player.y + shake_y, player.width, player.height), PURPLE)
    draw_bat_with_shine(screen, pygame.Rect(bat.x + shake_x, bat.y + shake_y, bat.width, bat.height))

    # Glitches with effects
    for g in glitches:
        draw_glitch_with_effects(screen, pygame.Rect(g.x + shake_x, g.y + shake_y, g.width, g.height))

    # Galaxy enemies
    for enemy in galaxy_enemies:
        draw_galaxy_enemy(screen, pygame.Rect(enemy["rect"].x + shake_x, enemy["rect"].y + shake_y, 
                                            enemy["rect"].width, enemy["rect"].height), enemy["type"])

    # Draw particles
    for particle in particles[:]:
        particle.update()
        if particle.life <= 0:
            particles.remove(particle)
        else:
            particle.draw(screen)

    # HUD
    draw_hud_with_effects(screen)

    # Galaxy messages
    if galaxy_mode and score % 10 == 0:
        message = galaxy_messages[min(galaxy_level - 1, len(galaxy_messages) - 1)]
        # Add cosmic glow effect to text
        for i in range(3):
            offset_x = random.randint(-2, 2)
            offset_y = random.randint(-2, 2)
            glow_color = random.choice([(255, 215, 0), (255, 0, 255), (0, 255, 255)])
            msg_text = FONT.render(message, True, glow_color)
            screen.blit(msg_text, (WIDTH//2 - 200 + offset_x, HEIGHT//2 + offset_y))

    # Glitch thoughts with typewriter effect
    elif score >= 15 and not boss_mode and not galaxy_mode and score % 5 == 0:
        q = random.choice(questions)
        # Add glitch effect to text
        for i in range(3):
            offset_x = random.randint(-1, 1)
            offset_y = random.randint(-1, 1)
            glitch_color = random.choice([CYAN, (255, 0, 255), (0, 255, 0)])
            q_text = FONT.render(q, True, glitch_color)
            screen.blit(q_text, (WIDTH//2 - 300 + offset_x, HEIGHT//2 + offset_y))

    pygame.display.flip()

# Loading screen variables
loading_screen = True
manual_screen = False
loading_time = 0
loading_particles = []
title_glow = 0
manual_scroll = 0

def create_loading_particles():
    for _ in range(5):
        x = random.randint(0, WIDTH)
        y = random.randint(0, HEIGHT)
        velocity = (random.uniform(-1, 1), random.uniform(-2, -0.5))
        life = random.randint(60, 120)
        colors = [(0, 255, 255), (255, 215, 0), (255, 0, 255), (0, 255, 0)]
        color = random.choice(colors)
        loading_particles.append(Particle(x, y, color, velocity, life))

def draw_loading_screen(surface):
    global title_glow, loading_time
    
    # Dark space background with gradient
    for y in range(HEIGHT):
        color_ratio = y / HEIGHT
        r = int(10 + color_ratio * 20)
        g = int(5 + color_ratio * 25)
        b = int(30 + color_ratio * 40)
        pygame.draw.line(surface, (r, g, b), (0, y), (WIDTH, y))
    
    # Draw stars
    for star in background_stars:
        brightness = star[2]
        color = (brightness, brightness, brightness)
        pygame.draw.circle(surface, color, (int(star[0]), int(star[1])), 1)
    
    # Update and draw loading particles
    for particle in loading_particles[:]:
        particle.update()
        if particle.life <= 0:
            loading_particles.remove(particle)
        else:
            particle.draw(surface)
    
    # Title animation
    title_glow = (title_glow + 2) % 360
    glow_intensity = abs(math.sin(math.radians(title_glow))) * 100
    
    # QUANTUM text
    quantum_text = pygame.font.Font(None, 80).render("QUANTUM", True, (0, 255, 255))
    # Add glow effect
    for offset in [(3, 3), (-3, -3), (3, -3), (-3, 3), (0, 3), (0, -3), (3, 0), (-3, 0)]:
        glow_surface = pygame.font.Font(None, 80).render("QUANTUM", True, (0, int(glow_intensity), int(glow_intensity)))
        surface.blit(glow_surface, (WIDTH//2 - 150 + offset[0], 150 + offset[1]))
    surface.blit(quantum_text, (WIDTH//2 - 150, 150))
    
    # CRASH text
    crash_text = pygame.font.Font(None, 80).render("CRASH", True, (255, 165, 0))
    # Add glow effect
    for offset in [(3, 3), (-3, -3), (3, -3), (-3, 3), (0, 3), (0, -3), (3, 0), (-3, 0)]:
        glow_surface = pygame.font.Font(None, 80).render("CRASH", True, (int(glow_intensity), int(glow_intensity/2), 0))
        surface.blit(glow_surface, (WIDTH//2 - 100 + offset[0], 220 + offset[1]))
    surface.blit(crash_text, (WIDTH//2 - 100, 220))
    
    # Subtitle
    subtitle_text = FONT.render("THE MULTIVERSE MAINTENANCE MANUAL", True, (100, 255, 200))
    subtitle_rect = subtitle_text.get_rect(center=(WIDTH//2, 290))
    surface.blit(subtitle_text, subtitle_rect)
    
    # Loading elements with glitch effects
    loading_dots = "." * ((loading_time // 20) % 4)
    loading_text = FONT.render(f"Initializing Reality{loading_dots}", True, WHITE)
    loading_rect = loading_text.get_rect(center=(WIDTH//2, 400))
    
    # Add glitch effect to loading text
    if random.randint(1, 10) == 1:
        glitch_colors = [(255, 0, 255), (0, 255, 255), (255, 255, 0)]
        glitch_color = random.choice(glitch_colors)
        glitch_offset = (random.randint(-2, 2), random.randint(-2, 2))
        glitch_text = FONT.render(f"Initializing Reality{loading_dots}", True, glitch_color)
        surface.blit(glitch_text, (loading_rect.x + glitch_offset[0], loading_rect.y + glitch_offset[1]))
    
    surface.blit(loading_text, loading_rect)
    
    # Progress bar
    bar_width = 300
    bar_height = 20
    bar_x = WIDTH // 2 - bar_width // 2
    bar_y = 450
    
    pygame.draw.rect(surface, (50, 50, 50), (bar_x - 2, bar_y - 2, bar_width + 4, bar_height + 4))
    pygame.draw.rect(surface, BLACK, (bar_x, bar_y, bar_width, bar_height))
    
    # Animated progress
    progress = min(loading_time / 300, 1.0)  # 5 seconds loading time
    progress_width = int(bar_width * progress)
    
    # Rainbow progress bar
    progress_colors = [
        (255, 0, 255),   # Magenta
        (0, 255, 255),   # Cyan
        (255, 255, 0),   # Yellow
        (255, 165, 0),   # Orange
        (0, 255, 0)      # Green
    ]
    
    for i in range(progress_width):
        color_index = int((i / bar_width) * len(progress_colors))
        color_index = min(color_index, len(progress_colors) - 1)
        color = progress_colors[color_index]
        pygame.draw.line(surface, color, (bar_x + i, bar_y), (bar_x + i, bar_y + bar_height))
    
    # Multiverse elements (floating geometric shapes)
    time_offset = pygame.time.get_ticks() / 1000
    
    # Floating cubes
    for i in range(5):
        cube_x = 100 + i * 150 + math.sin(time_offset + i) * 20
        cube_y = 350 + math.cos(time_offset + i * 0.5) * 15
        cube_size = 15 + math.sin(time_offset * 2 + i) * 5
        
        # Cube shadow
        shadow_points = [
            (cube_x + cube_size + 5, cube_y + cube_size + 5),
            (cube_x + cube_size * 2 + 5, cube_y + 5),
            (cube_x + cube_size * 2 + 5, cube_y + cube_size + 5),
            (cube_x + cube_size + 5, cube_y + cube_size * 2 + 5)
        ]
        pygame.draw.polygon(surface, (20, 20, 20), shadow_points)
        
        # Main cube
        cube_points = [
            (cube_x, cube_y + cube_size),
            (cube_x + cube_size, cube_y),
            (cube_x + cube_size * 2, cube_y + cube_size),
            (cube_x + cube_size, cube_y + cube_size * 2)
        ]
        cube_colors = [(0, 100, 200), (0, 150, 255), (100, 200, 255)]
        pygame.draw.polygon(surface, random.choice(cube_colors), cube_points)
    
    # Project info smaller
    project_text = SMALLFONT.render("School Project 12/07/2025", True, (150, 150, 200))
    project_rect = project_text.get_rect(center=(WIDTH//2, 480))
    surface.blit(project_text, project_rect)
    
    # Instruction text
    if loading_time > 200:  # Show after a delay
        instruction_text = SMALLFONT.render("Press SPACE to begin your quantum journey...", True, (200, 200, 200))
        instruction_rect = instruction_text.get_rect(center=(WIDTH//2, 500))
        
        # Blinking effect
        alpha = int(255 * (0.5 + 0.5 * math.sin(time_offset * 4)))
        instruction_surface = pygame.Surface(instruction_text.get_size(), pygame.SRCALPHA)
        instruction_surface.fill((*instruction_text.get_at((0, 0))[:3], alpha))
        instruction_text.set_alpha(alpha)
        surface.blit(instruction_text, instruction_rect)
    
    # Version text
    version_text = pygame.font.Font(None, 16).render("v1.0 Final Edition", True, (100, 100, 100))
    surface.blit(version_text, (10, HEIGHT - 20))

def draw_manual_screen(surface):
    global manual_scroll
    
    # Dark background with subtle gradient
    for y in range(HEIGHT):
        color_ratio = y / HEIGHT
        r = int(15 + color_ratio * 10)
        g = int(15 + color_ratio * 15)
        b = int(25 + color_ratio * 25)
        pygame.draw.line(surface, (r, g, b), (0, y), (WIDTH, y))
    
    # Draw some background stars
    for star in background_stars[::3]:  # Use fewer stars
        brightness = star[2] // 2
        color = (brightness, brightness, brightness)
        pygame.draw.circle(surface, color, (int(star[0]), int(star[1])), 1)
    
    # Manual content
    y_offset = 50 - manual_scroll
    
    # Title
    title_text = pygame.font.Font(None, 48).render("QUANTUM CRASH - USER MANUAL", True, (0, 255, 255))
    title_rect = title_text.get_rect(center=(WIDTH//2, y_offset))
    surface.blit(title_text, title_rect)
    
    y_offset += 80
    
    # What Is This Game section
    section_font = pygame.font.Font(None, 32)
    content_font = pygame.font.Font(None, 24)
    small_font = pygame.font.Font(None, 20)
    
    # Section: What Is This Game?
    what_title = section_font.render("What Is This Game?", True, (255, 215, 0))
    surface.blit(what_title, (50, y_offset))
    y_offset += 40
    
    what_text = [
        "Reality is collapsing and only your cricket bat can save the multiverse.",
        "You play Rizik, a dimension-hopping janitor armed with a glitch-splitting bat.",
        "Smash anomalies, unlock surreal glitch thoughts, and face off against your",
        "cheating past self in the final boss showdown."
    ]
    
    for line in what_text:
        text = content_font.render(line, True, WHITE)
        surface.blit(text, (70, y_offset))
        y_offset += 25
    
    y_offset += 20
    
    # Section: Core Mechanics
    mechanics_title = section_font.render("🧪 Core Mechanics", True, (255, 100, 255))
    surface.blit(mechanics_title, (50, y_offset))
    y_offset += 40
    
    mechanics_text = [
        "• Smash falling glitches to gain Reality Points",
        "• Unlock Level 2 & 3 with increasing glitch speed and size",
        "• Face your Past Self in Boss Mode at score ≥ 30",
        "• Answer bizarre philosophical questions mid-game (just because)",
        "• Reach Galaxy Mode after defeating your past self",
        "• Conquer 5 galaxy levels to become the Galaxy Master"
    ]
    
    for line in mechanics_text:
        text = content_font.render(line, True, (200, 255, 200))
        surface.blit(text, (70, y_offset))
        y_offset += 25
    
    y_offset += 20
    
    # Section: Controls
    controls_title = section_font.render("🎯 Controls", True, (255, 165, 0))
    surface.blit(controls_title, (50, y_offset))
    y_offset += 40
    
    controls_text = [
        "← / → Arrow Keys — Move Rizik left and right",
        "SPACE — Swing bat in Boss Mode to damage your past self",
        "R — Reboot reality after victory",
        "Q — Quit when you've had enough multiverse madness"
    ]
    
    for line in controls_text:
        text = content_font.render(line, True, (255, 255, 150))
        surface.blit(text, (70, y_offset))
        y_offset += 25
    
    y_offset += 20
    
    # Section: Bonus Features
    bonus_title = section_font.render("🎬 Bonus Features", True, (0, 255, 150))
    surface.blit(bonus_title, (50, y_offset))
    y_offset += 40
    
    bonus_text = [
        "• Dynamic level progression with visual effects",
        "• Philosophical glitch pop-ups that make you question reality",
        "• Multiple endings: Defeat boss or conquer the galaxy",
        "• Particle effects, screen shake, and cosmic visuals",
        "• Playable completely in Replit with just one click!"
    ]
    
    for line in bonus_text:
        text = content_font.render(line, True, (150, 255, 255))
        surface.blit(text, (70, y_offset))
        y_offset += 25
    
    y_offset += 30
    
    # Instructions to continue
    continue_text = section_font.render("Press SPACE to begin your quantum journey!", True, (255, 255, 0))
    continue_rect = continue_text.get_rect(center=(WIDTH//2, y_offset))
    
    # Blinking effect
    time_offset = pygame.time.get_ticks() / 1000
    alpha = int(255 * (0.5 + 0.5 * math.sin(time_offset * 3)))
    continue_text.set_alpha(alpha)
    surface.blit(continue_text, continue_rect)
    
    # Scroll instructions if content is too long
    if y_offset > HEIGHT - 100:
        scroll_text = small_font.render("Use ↑↓ arrows to scroll", True, (150, 150, 150))
        scroll_rect = scroll_text.get_rect(center=(WIDTH//2, HEIGHT - 30))
        surface.blit(scroll_text, scroll_rect)

# Game loop
running = True
while running:
    clock.tick(60)
    
    # Loading screen phase
    if loading_screen:
        loading_time += 1
        
        # Create loading particles
        if loading_time % 10 == 0:
            create_loading_particles()
        
        # Update background stars
        update_background()
        
        # Draw loading screen
        draw_loading_screen(screen)
        pygame.display.flip()
        
        # Handle loading screen events
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE and loading_time > 200:
                    loading_screen = False
                    manual_screen = True
                    loading_particles.clear()
        
        # Auto-advance after 8 seconds
        if loading_time > 480:
            loading_screen = False
            manual_screen = True
            loading_particles.clear()
        
        continue
    
    # Manual screen phase
    if manual_screen:
        # Update background stars
        update_background()
        
        # Draw manual screen
        draw_manual_screen(screen)
        pygame.display.flip()
        
        # Handle manual screen events
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE:
                    manual_screen = False
                elif event.key == pygame.K_UP:
                    manual_scroll = max(0, manual_scroll - 20)
                elif event.key == pygame.K_DOWN:
                    manual_scroll = min(300, manual_scroll + 20)
        
        continue

    if not game_over:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False

        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT] and player.x > 5:
            player.x -= 5
        if keys[pygame.K_RIGHT] and player.x < WIDTH - 55:
            player.x += 5

        bat.x = player.x + 20
        bat.y = player.y - 60

        # Update background
        update_background()

        # Add glitches until boss mode
        if not boss_mode and not galaxy_mode and score < 30:
            glitch_spawn_timer += 1
            spawn_rate = 25
            if glitch_spawn_timer >= spawn_rate:
                glitches.append(pygame.Rect(random.randint(0, WIDTH - 40), 0, 40, 40))
                glitch_spawn_timer = 0

        # Glitch movement and collision (faster after boss)
        glitch_speed = 8 if boss_mode or galaxy_mode else 5
        for g in glitches[:]:
            g.y += glitch_speed
            if g.colliderect(player) or g.colliderect(bat):
                score += 1
                glitches.remove(g)
                # Create particle explosion
                create_explosion(g.x + 20, g.y + 20, CYAN, 15)
                screen_shake = 5
            elif g.y > HEIGHT:
                glitches.remove(g)

        # Galaxy mode enemy spawning and movement
        if galaxy_mode:
            # Spawn galaxy enemies more frequently and faster
            if random.randint(1, 8 - galaxy_level) == 1:
                enemy_types = ["star", "comet", "asteroid"]
                enemy_type = random.choice(enemy_types)
                enemy_size = random.randint(30, 50)
                enemy = {
                    "rect": pygame.Rect(random.randint(0, WIDTH - enemy_size), 0, enemy_size, enemy_size),
                    "type": enemy_type,
                    "speed": random.randint(6 + galaxy_level, 10 + galaxy_level * 2)
                }
                galaxy_enemies.append(enemy)

            # Move and check collision for galaxy enemies
            for enemy in galaxy_enemies[:]:
                enemy["rect"].y += enemy["speed"]
                if enemy["rect"].colliderect(player) or enemy["rect"].colliderect(bat):
                    score += 2  # Galaxy enemies give more points
                    galaxy_enemies.remove(enemy)
                    # Different explosion colors for different enemy types
                    explosion_colors = {"star": (255, 215, 0), "comet": (0, 191, 255), "asteroid": (139, 69, 19)}
                    create_explosion(enemy["rect"].x + 20, enemy["rect"].y + 20, explosion_colors[enemy["type"]], 20)
                    screen_shake = 7
                elif enemy["rect"].y > HEIGHT:
                    galaxy_enemies.remove(enemy)

        # Trigger boss mode
        if score >= 30 and not boss_mode and not galaxy_mode:
            boss_mode = True
            glitches.clear()
            create_explosion(WIDTH // 2, HEIGHT // 2, RED, 50)
            screen_shake = 20

        # Boss battle
        if boss_mode:
            # Keep spawning faster glitches during boss fight
            if random.randint(1, 8) == 1:
                glitches.append(pygame.Rect(random.randint(0, WIDTH - 40), 0, 40, 40))
            
            if keys[pygame.K_SPACE]:
                boss_hp -= 1
                create_particles(WIDTH // 2, HEIGHT // 2, RED, 20)
                screen_shake = 8
            if boss_hp <= 0:
                boss_mode = False
                galaxy_mode = True
                galaxy_level = 1
                glitches.clear()
                create_explosion(WIDTH // 2, HEIGHT // 2, (255, 215, 0), 100)

        # Galaxy progression
        if galaxy_mode:
            # Check if player has enough score to advance galaxy level
            required_score = 30 + (galaxy_level * 15)
            if score >= required_score and galaxy_level < max_galaxy_level:
                galaxy_level += 1
                create_explosion(WIDTH // 2, HEIGHT // 2, (255, 215, 0), 50)
                screen_shake = 15
            elif galaxy_level >= max_galaxy_level and score >= 30 + (max_galaxy_level * 15):
                victory = True
                game_over = True
                create_explosion(WIDTH // 2, HEIGHT // 2, (255, 215, 0), 200)

        draw()

    else:
        # Gradient background for end screen
        for y in range(HEIGHT):
            if victory:
                # Galaxy victory - cosmic gradient
                color_ratio = y / HEIGHT
                r = int(20 + color_ratio * 30)
                g = int(10 + color_ratio * 25)
                b = int(50 + color_ratio * 40)
            else:
                # Boss victory - green tech gradient
                color_ratio = y / HEIGHT
                r = int(10 + color_ratio * 15)
                g = int(30 + color_ratio * 50)
                b = int(10 + color_ratio * 20)
            pygame.draw.line(screen, (r, g, b), (0, y), (WIDTH, y))
        
        # Victory particles
        if random.randint(1, 3) == 1:
            if victory:
                # Galaxy victory - gold and white particles
                create_particles(random.randint(0, WIDTH), random.randint(0, HEIGHT), (255, 215, 0), 5)
                create_particles(random.randint(0, WIDTH), random.randint(0, HEIGHT), (255, 255, 255), 3)
            else:
                # Boss victory - green and cyan particles
                create_particles(random.randint(0, WIDTH), random.randint(0, HEIGHT), GREEN, 3)
                create_particles(random.randint(0, WIDTH), random.randint(0, HEIGHT), CYAN, 2)
        
        # Draw particles
        for particle in particles[:]:
            particle.update()
            if particle.life <= 0:
                particles.remove(particle)
            else:
                particle.draw(screen)
        
        if victory:
            end_text = BIGFONT.render("🌌 GALAXY MASTER - UNIVERSE CONQUERED! 🌌", True, (255, 215, 0))
            sub_text = FONT.render("You have transcended reality itself!", True, (255, 255, 255))
            achievement_text = FONT.render("🏆 Achievement Unlocked: Multiverse Champion 🏆", True, (255, 215, 0))
            choice_text = FONT.render("Press R to Restart the Universe or Q to Quit", True, WHITE)
            # Large creator credit
            creator_font = pygame.font.Font(None, 36)
            creator_credit = creator_font.render("Game by Nirenjan Nair", True, (255, 215, 0))
            creator_rect = creator_credit.get_rect(center=(WIDTH//2, 320))
            
            # Add cosmic glow to victory text
            for offset in [(3, 3), (-3, -3), (3, -3), (-3, 3)]:
                glow_text = BIGFONT.render("🌌 GALAXY MASTER - UNIVERSE CONQUERED! 🌌", True, (100, 100, 0))
                screen.blit(glow_text, (50 + offset[0], 160 + offset[1]))
            
            # Add glow to creator name
            for offset in [(2, 2), (-2, -2), (2, -2), (-2, 2)]:
                glow_creator = creator_font.render("Game by Nirenjan Nair", True, (100, 100, 0))
                screen.blit(glow_creator, (creator_rect.x + offset[0], creator_rect.y + offset[1]))
            
            screen.blit(end_text, (50, 160))
            screen.blit(sub_text, (220, 200))
            screen.blit(achievement_text, (150, 230))
            screen.blit(choice_text, (180, 260))
            
            # Draw victorious boy with bat
            draw_victory_boy_with_bat(screen, WIDTH // 2 - 50, HEIGHT - 240, 1.5)
            
            # Creator credit at bottom
            creator_rect = creator_credit.get_rect(center=(WIDTH//2, HEIGHT - 50))
            screen.blit(creator_credit, creator_rect)
            
            # Project date smaller
            date_text = SMALLFONT.render("12/07/2025", True, (150, 150, 200))
            date_rect = date_text.get_rect(center=(WIDTH//2, HEIGHT - 20))
            screen.blit(date_text, date_rect)
        else:
            end_text = BIGFONT.render("✅ You Defeated Your Past Self", True, GREEN)
            achievement_text = FONT.render("🎯 Achievement: Reality Debugger", True, (0, 255, 100))
            choice_text = FONT.render("Press R to Reboot Reality or Q to Quit", True, WHITE)
            # Large creator credit
            creator_font = pygame.font.Font(None, 36)
            creator_credit = creator_font.render("Game by Nirenjan Nair", True, (0, 255, 100))
            creator_rect = creator_credit.get_rect(center=(WIDTH//2, 310))
            
            # Add glow to victory text
            for offset in [(2, 2), (-2, -2), (2, -2), (-2, 2)]:
                glow_text = BIGFONT.render("✅ You Defeated Your Past Self", True, (0, 100, 0))
                screen.blit(glow_text, (150 + offset[0], 180 + offset[1]))
            
            # Add glow to creator name
            for offset in [(2, 2), (-2, -2), (2, -2), (-2, 2)]:
                glow_creator = creator_font.render("Game by Nirenjan Nair", True, (0, 100, 0))
                screen.blit(glow_creator, (creator_rect.x + offset[0], creator_rect.y + offset[1]))
            
            screen.blit(end_text, (150, 180))
            screen.blit(achievement_text, (200, 220))
            screen.blit(choice_text, (190, 250))
            
            # Draw victorious boy with bat (smaller for boss victory)
            draw_victory_boy_with_bat(screen, WIDTH // 2 - 30, HEIGHT - 200, 1.2)
            
            # Creator credit at bottom
            creator_rect = creator_credit.get_rect(center=(WIDTH//2, HEIGHT - 50))
            screen.blit(creator_credit, creator_rect)
            
            # Project date smaller
            date_text = SMALLFONT.render("12/07/2025", True, (150, 150, 200))
            date_rect = date_text.get_rect(center=(WIDTH//2, HEIGHT - 20))
            screen.blit(date_text, date_rect)
        
        pygame.display.flip()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_r:
                    # Reset everything
                    score = 0
                    boss_hp = max_boss_hp
                    boss_mode = False
                    galaxy_mode = False
                    galaxy_level = 1
                    game_over = False
                    victory = False
                    player.x = WIDTH // 2
                    particles.clear()
                    player_trail.clear()
                    galaxy_enemies.clear()
                    glitches.clear()
                    screen_shake = 0
                    glitch_spawn_timer = 0
                    manual_scroll = 0
                if event.key == pygame.K_q:
                    running = False

pygame.quit()
import pygame
import random
import sys
import math

# Initialize game
pygame.init()
WIDTH, HEIGHT = 800, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Quantum Crash – The Multiverse Maintenance Manual")
clock = pygame.time.Clock()

# Colors and fonts
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
GREEN = (0, 255, 0)
BLUE = (0, 0, 255)
YELLOW = (255, 255, 0)
PURPLE = (180, 0, 220)
CYAN = (0, 255, 255)
ORANGE = (255, 165, 0)
FONT = pygame.font.SysFont("Arial", 28)
BIGFONT = pygame.font.SysFont("Arial", 40)
SMALLFONT = pygame.font.SysFont("Arial", 16)

# Particle system
class Particle:
    def __init__(self, x, y, color, velocity, life):
        self.x = x
        self.y = y
        self.color = color
        self.velocity = velocity
        self.life = life
        self.max_life = life
        self.size = random.randint(2, 6)
    
    def update(self):
        self.x += self.velocity[0]
        self.y += self.velocity[1]
        self.life -= 1
        # Fade out
        alpha = int(255 * (self.life / self.max_life))
        self.color = (*self.color[:3], max(0, alpha))
    
    def draw(self, surface):
        if self.life > 0:
            # Create a surface with per-pixel alpha
            particle_surf = pygame.Surface((self.size * 2, self.size * 2), pygame.SRCALPHA)
            alpha = int(255 * (self.life / self.max_life))
            color_with_alpha = (*self.color[:3], alpha)
            pygame.draw.circle(particle_surf, color_with_alpha, (self.size, self.size), self.size)
            surface.blit(particle_surf, (self.x - self.size, self.y - self.size))

# Player setup
player = pygame.Rect(WIDTH // 2, HEIGHT - 80, 50, 50)
bat = pygame.Rect(player.x + 20, player.y - 60, 10, 60)
glitches = []
particles = []
score = 0
boss_mode = False
boss_hp = 10
max_boss_hp = 10
galaxy_mode = False
galaxy_level = 1
max_galaxy_level = 5
galaxy_enemies = []
game_over = False
victory = False
player_trail = []
screen_shake = 0
background_stars = []
glitch_spawn_timer = 0

# Create background stars
for _ in range(100):
    star_x = random.randint(0, WIDTH)
    star_y = random.randint(0, HEIGHT)
    star_brightness = random.randint(50, 255)
    background_stars.append([star_x, star_y, star_brightness])

# Glitch thoughts (surreal prompts)
questions = [
    "Are you playing the game, or is it playing you?",
    "Would you delete yourself to fix the code?",
    "If glitches are reality, what is truth?",
    "The cricket bat remembers everything.",
]

galaxy_messages = [
    f"🌌 GALAXY LEVEL {galaxy_level} 🌌",
    "The universe bends to your will!",
    "Reality fragments across dimensions!",
    "You are becoming one with the cosmos!",
    "The final frontier awaits..."
]

def create_particles(x, y, color, count=10):
    for _ in range(count):
        velocity = (random.randint(-3, 3), random.randint(-3, 3))
        life = random.randint(20, 40)
        particles.append(Particle(x, y, color, velocity, life))

def create_explosion(x, y, color, count=20):
    # Main explosion particles
    for _ in range(count):
        angle = random.uniform(0, 2 * math.pi)
        speed = random.uniform(2, 8)
        velocity = (math.cos(angle) * speed, math.sin(angle) * speed)
        life = random.randint(30, 60)
        particles.append(Particle(x, y, color, velocity, life))
    
    # Add sparkle particles for extra polish
    for _ in range(count // 2):
        angle = random.uniform(0, 2 * math.pi)
        speed = random.uniform(1, 4)
        velocity = (math.cos(angle) * speed, math.sin(angle) * speed)
        life = random.randint(40, 80)
        sparkle_color = (255, 255, 255)  # White sparkles
        particles.append(Particle(x, y, sparkle_color, velocity, life))

def draw_player_with_glow(surface, rect, color):
    # Draw glow effect
    glow_surf = pygame.Surface((rect.width + 20, rect.height + 20), pygame.SRCALPHA)
    glow_color = (*color, 100)
    pygame.draw.rect(glow_surf, glow_color, (0, 0, rect.width + 20, rect.height + 20), border_radius=10)
    surface.blit(glow_surf, (rect.x - 10, rect.y - 10))
    
    # Draw main player
    pygame.draw.rect(surface, color, rect, border_radius=5)
    # Add highlight
    highlight_rect = pygame.Rect(rect.x + 5, rect.y + 5, rect.width - 10, rect.height // 3)
    highlight_color = tuple(min(255, c + 50) for c in color)
    pygame.draw.rect(surface, highlight_color, highlight_rect, border_radius=3)

def draw_glitch_with_effects(surface, rect):
    # Main glitch body
    colors = [CYAN, (0, 255, 100), (100, 255, 255)]
    main_color = random.choice(colors)
    
    # Glitch effect - multiple overlapping rectangles
    for i in range(3):
        offset_x = random.randint(-2, 2)
        offset_y = random.randint(-2, 2)
        glitch_rect = pygame.Rect(rect.x + offset_x, rect.y + offset_y, rect.width, rect.height)
        color = (*main_color, 150 - i * 30)
        
        glitch_surf = pygame.Surface((rect.width, rect.height), pygame.SRCALPHA)
        pygame.draw.rect(glitch_surf, color, (0, 0, rect.width, rect.height))
        surface.blit(glitch_surf, (glitch_rect.x, glitch_rect.y))
    
    # Add scanlines
    for y in range(rect.y, rect.y + rect.height, 4):
        pygame.draw.line(surface, (255, 255, 255, 100), (rect.x, y), (rect.x + rect.width, y), 1)

def draw_bat_with_shine(surface, rect):
    # Bat shadow
    shadow_rect = pygame.Rect(rect.x + 2, rect.y + 2, rect.width, rect.height)
    pygame.draw.rect(surface, (150, 150, 0), shadow_rect, border_radius=3)
    
    # Main bat
    pygame.draw.rect(surface, YELLOW, rect, border_radius=3)
    
    # Shine effect
    shine_rect = pygame.Rect(rect.x + 2, rect.y + 2, rect.width - 4, rect.height // 4)
    pygame.draw.rect(surface, (255, 255, 200), shine_rect, border_radius=2)

def draw_galaxy_enemy(surface, rect, enemy_type):
    if enemy_type == "star":
        # Draw a star-shaped enemy
        center_x, center_y = rect.centerx, rect.centery
        points = []
        for i in range(10):
            angle = i * math.pi / 5
            if i % 2 == 0:
                radius = rect.width // 2
            else:
                radius = rect.width // 4
            x = center_x + radius * math.cos(angle)
            y = center_y + radius * math.sin(angle)
            points.append((x, y))
        pygame.draw.polygon(surface, (255, 215, 0), points)
        pygame.draw.polygon(surface, (255, 255, 255), points, 2)
    elif enemy_type == "comet":
        # Draw a comet with tail
        pygame.draw.circle(surface, (0, 191, 255), rect.center, rect.width // 2)
        # Comet tail
        tail_points = [
            (rect.centerx, rect.centery),
            (rect.centerx - 20, rect.centery - 10),
            (rect.centerx - 30, rect.centery),
            (rect.centerx - 20, rect.centery + 10)
        ]
        pygame.draw.polygon(surface, (173, 216, 230), tail_points)
    else:  # asteroid
        # Draw an irregular asteroid
        pygame.draw.circle(surface, (139, 69, 19), rect.center, rect.width // 2)
        pygame.draw.circle(surface, (160, 82, 45), (rect.centerx - 5, rect.centery - 5), rect.width // 3)

def draw_victory_boy_with_bat(surface, x, y, scale=1.0):
    # Boy's body
    body_width = int(40 * scale)
    body_height = int(60 * scale)
    body_rect = pygame.Rect(x, y, body_width, body_height)
    
    # Head
    head_radius = int(15 * scale)
    head_center = (x + body_width // 2, y - head_radius)
    pygame.draw.circle(surface, (255, 220, 177), head_center, head_radius)  # Skin color
    
    # Hair
    hair_rect = pygame.Rect(x + int(5 * scale), y - int(25 * scale), int(30 * scale), int(15 * scale))
    pygame.draw.ellipse(surface, (139, 69, 19), hair_rect)  # Brown hair
    
    # Eyes
    left_eye = (x + int(10 * scale), y - int(10 * scale))
    right_eye = (x + int(25 * scale), y - int(10 * scale))
    pygame.draw.circle(surface, BLACK, left_eye, int(2 * scale))
    pygame.draw.circle(surface, BLACK, right_eye, int(2 * scale))
    
    # Smile
    smile_start = (x + int(12 * scale), y - int(5 * scale))
    smile_end = (x + int(23 * scale), y - int(5 * scale))
    pygame.draw.arc(surface, BLACK, (x + int(10 * scale), y - int(10 * scale), int(20 * scale), int(10 * scale)), 0, math.pi, 2)
    
    # Body (shirt)
    pygame.draw.rect(surface, (0, 100, 200), body_rect)  # Blue shirt
    
    # Arms
    left_arm = pygame.Rect(x - int(15 * scale), y + int(10 * scale), int(15 * scale), int(30 * scale))
    right_arm = pygame.Rect(x + body_width, y + int(10 * scale), int(15 * scale), int(30 * scale))
    pygame.draw.rect(surface, (255, 220, 177), left_arm)  # Left arm
    pygame.draw.rect(surface, (255, 220, 177), right_arm)  # Right arm
    
    # Legs
    left_leg = pygame.Rect(x + int(5 * scale), y + body_height, int(12 * scale), int(40 * scale))
    right_leg = pygame.Rect(x + int(23 * scale), y + body_height, int(12 * scale), int(40 * scale))
    pygame.draw.rect(surface, (0, 0, 139), left_leg)  # Dark blue pants
    pygame.draw.rect(surface, (0, 0, 139), right_leg)
    
    # Shoes
    left_shoe = pygame.Rect(x + int(2 * scale), y + body_height + int(35 * scale), int(18 * scale), int(8 * scale))
    right_shoe = pygame.Rect(x + int(20 * scale), y + body_height + int(35 * scale), int(18 * scale), int(8 * scale))
    pygame.draw.rect(surface, BLACK, left_shoe)
    pygame.draw.rect(surface, BLACK, right_shoe)
    
    # Cricket bat in hands
    bat_x = x + body_width + int(5 * scale)
    bat_y = y - int(10 * scale)
    bat_width = int(8 * scale)
    bat_height = int(80 * scale)
    bat_rect = pygame.Rect(bat_x, bat_y, bat_width, bat_height)
    
    # Bat handle
    handle_rect = pygame.Rect(bat_x, bat_y + int(60 * scale), bat_width, int(20 * scale))
    pygame.draw.rect(surface, (139, 69, 19), handle_rect)  # Brown handle
    
    # Bat blade
    blade_rect = pygame.Rect(bat_x, bat_y, bat_width, int(60 * scale))
    pygame.draw.rect(surface, (222, 184, 135), blade_rect)  # Light wood color
    
    # Bat details
    for i in range(3):
        line_y = bat_y + int(15 * scale) + i * int(15 * scale)
        pygame.draw.line(surface, (160, 82, 45), (bat_x, line_y), (bat_x + bat_width, line_y), 1)
    
    # Victory sparkles around the boy
    sparkle_time = pygame.time.get_ticks()
    for i in range(8):
        angle = (sparkle_time / 200 + i * math.pi / 4) % (2 * math.pi)
        sparkle_x = x + body_width // 2 + int(math.cos(angle) * 50 * scale)
        sparkle_y = y + body_height // 2 + int(math.sin(angle) * 50 * scale)
        pygame.draw.circle(surface, (255, 255, 0), (sparkle_x, sparkle_y), int(3 * scale))
        pygame.draw.circle(surface, (255, 255, 255), (sparkle_x, sparkle_y), int(2 * scale))

def draw_hud_with_effects(surface):
    # Score with glow
    score_text = FONT.render(f"Reality Score: {score}", True, WHITE)
    glow_text = FONT.render(f"Reality Score: {score}", True, CYAN)
    
    # Draw glow
    for offset in [(1, 1), (-1, -1), (1, -1), (-1, 1)]:
        surface.blit(glow_text, (20 + offset[0], 20 + offset[1]))
    surface.blit(score_text, (20, 20))
    
    # Health bar for boss
    if boss_mode:
        bar_width = 300
        bar_height = 20
        bar_x = WIDTH // 2 - bar_width // 2
        bar_y = 50
        
        # Background
        pygame.draw.rect(surface, (50, 50, 50), (bar_x - 2, bar_y - 2, bar_width + 4, bar_height + 4))
        pygame.draw.rect(surface, BLACK, (bar_x, bar_y, bar_width, bar_height))
        
        # Health
        health_width = int(bar_width * (boss_hp / max_boss_hp))
        health_color = RED if boss_hp < 3 else ORANGE if boss_hp < 6 else GREEN
        pygame.draw.rect(surface, health_color, (bar_x, bar_y, health_width, bar_height))
        
        # Boss text
        boss_text = BIGFONT.render("⚔️ Face Your Past Self", True, RED)
        text_rect = boss_text.get_rect(center=(WIDTH // 2, 30))
        surface.blit(boss_text, text_rect)
    
    # Galaxy mode HUD
    if galaxy_mode:
        galaxy_text = BIGFONT.render(f"🌌 GALAXY {galaxy_level}/{max_galaxy_level} 🌌", True, (255, 215, 0))
        text_rect = galaxy_text.get_rect(center=(WIDTH // 2, 30))
        surface.blit(galaxy_text, text_rect)
        
        # Galaxy progress bar
        bar_width = 200
        bar_height = 15
        bar_x = WIDTH // 2 - bar_width // 2
        bar_y = 60
        
        pygame.draw.rect(surface, (50, 50, 50), (bar_x - 2, bar_y - 2, bar_width + 4, bar_height + 4))
        pygame.draw.rect(surface, BLACK, (bar_x, bar_y, bar_width, bar_height))
        
        progress_width = int(bar_width * (galaxy_level / max_galaxy_level))
        galaxy_colors = [(255, 0, 255), (0, 255, 255), (255, 255, 0), (255, 165, 0), (255, 0, 0)]
        progress_color = galaxy_colors[min(galaxy_level - 1, len(galaxy_colors) - 1)]
        pygame.draw.rect(surface, progress_color, (bar_x, bar_y, progress_width, bar_height))

def update_background():
    # Animate background stars
    for star in background_stars:
        star[1] += 0.5
        if star[1] > HEIGHT:
            star[1] = 0
            star[0] = random.randint(0, WIDTH)

def draw_background(surface):
    # Gradient background
    for y in range(HEIGHT):
        color_ratio = y / HEIGHT
        r = int(20 + color_ratio * 10)
        g = int(20 + color_ratio * 15)
        b = int(40 + color_ratio * 20)
        pygame.draw.line(surface, (r, g, b), (0, y), (WIDTH, y))
    
    # Draw stars
    for star in background_stars:
        brightness = star[2]
        color = (brightness, brightness, brightness)
        pygame.draw.circle(surface, color, (int(star[0]), int(star[1])), 1)

def apply_screen_shake():
    global screen_shake
    if screen_shake > 0:
        shake_x = random.randint(-screen_shake, screen_shake)
        shake_y = random.randint(-screen_shake, screen_shake)
        screen_shake -= 1
        return shake_x, shake_y
    return 0, 0

def draw():
    global screen_shake
    
    # Apply screen shake
    shake_x, shake_y = apply_screen_shake()
    
    # Draw background
    draw_background(screen)
    
    # Player trail effect
    player_trail.append((player.x + 25, player.y + 25))
    if len(player_trail) > 10:
        player_trail.pop(0)
    
    for i, pos in enumerate(player_trail):
        alpha = int(255 * (i / len(player_trail)) * 0.3)
        trail_surf = pygame.Surface((10, 10), pygame.SRCALPHA)
        pygame.draw.circle(trail_surf, (*PURPLE, alpha), (5, 5), 5)
        screen.blit(trail_surf, (pos[0] - 5 + shake_x, pos[1] - 5 + shake_y))

    # Player and bat with effects
    draw_player_with_glow(screen, pygame.Rect(player.x + shake_x, player.y + shake_y, player.width, player.height), PURPLE)
    draw_bat_with_shine(screen, pygame.Rect(bat.x + shake_x, bat.y + shake_y, bat.width, bat.height))

    # Glitches with effects
    for g in glitches:
        draw_glitch_with_effects(screen, pygame.Rect(g.x + shake_x, g.y + shake_y, g.width, g.height))

    # Galaxy enemies
    for enemy in galaxy_enemies:
        draw_galaxy_enemy(screen, pygame.Rect(enemy["rect"].x + shake_x, enemy["rect"].y + shake_y, 
                                            enemy["rect"].width, enemy["rect"].height), enemy["type"])

    # Draw particles
    for particle in particles[:]:
        particle.update()
        if particle.life <= 0:
            particles.remove(particle)
        else:
            particle.draw(screen)

    # HUD
    draw_hud_with_effects(screen)

    # Galaxy messages
    if galaxy_mode and score % 10 == 0:
        message = galaxy_messages[min(galaxy_level - 1, len(galaxy_messages) - 1)]
        # Add cosmic glow effect to text
        for i in range(3):
            offset_x = random.randint(-2, 2)
            offset_y = random.randint(-2, 2)
            glow_color = random.choice([(255, 215, 0), (255, 0, 255), (0, 255, 255)])
            msg_text = FONT.render(message, True, glow_color)
            screen.blit(msg_text, (WIDTH//2 - 200 + offset_x, HEIGHT//2 + offset_y))

    # Glitch thoughts with typewriter effect
    elif score >= 15 and not boss_mode and not galaxy_mode and score % 5 == 0:
        q = random.choice(questions)
        # Add glitch effect to text
        for i in range(3):
            offset_x = random.randint(-1, 1)
            offset_y = random.randint(-1, 1)
            glitch_color = random.choice([CYAN, (255, 0, 255), (0, 255, 0)])
            q_text = FONT.render(q, True, glitch_color)
            screen.blit(q_text, (WIDTH//2 - 300 + offset_x, HEIGHT//2 + offset_y))

    pygame.display.flip()

# Loading screen variables
loading_screen = True
manual_screen = False
loading_time = 0
loading_particles = []
title_glow = 0
manual_scroll = 0

def create_loading_particles():
    for _ in range(5):
        x = random.randint(0, WIDTH)
        y = random.randint(0, HEIGHT)
        velocity = (random.uniform(-1, 1), random.uniform(-2, -0.5))
        life = random.randint(60, 120)
        colors = [(0, 255, 255), (255, 215, 0), (255, 0, 255), (0, 255, 0)]
        color = random.choice(colors)
        loading_particles.append(Particle(x, y, color, velocity, life))

def draw_loading_screen(surface):
    global title_glow, loading_time
    
    # Dark space background with gradient
    for y in range(HEIGHT):
        color_ratio = y / HEIGHT
        r = int(10 + color_ratio * 20)
        g = int(5 + color_ratio * 25)
        b = int(30 + color_ratio * 40)
        pygame.draw.line(surface, (r, g, b), (0, y), (WIDTH, y))
    
    # Draw stars
    for star in background_stars:
        brightness = star[2]
        color = (brightness, brightness, brightness)
        pygame.draw.circle(surface, color, (int(star[0]), int(star[1])), 1)
    
    # Update and draw loading particles
    for particle in loading_particles[:]:
        particle.update()
        if particle.life <= 0:
            loading_particles.remove(particle)
        else:
            particle.draw(surface)
    
    # Title animation
    title_glow = (title_glow + 2) % 360
    glow_intensity = abs(math.sin(math.radians(title_glow))) * 100
    
    # QUANTUM text
    quantum_text = pygame.font.Font(None, 80).render("QUANTUM", True, (0, 255, 255))
    # Add glow effect
    for offset in [(3, 3), (-3, -3), (3, -3), (-3, 3), (0, 3), (0, -3), (3, 0), (-3, 0)]:
        glow_surface = pygame.font.Font(None, 80).render("QUANTUM", True, (0, int(glow_intensity), int(glow_intensity)))
        surface.blit(glow_surface, (WIDTH//2 - 150 + offset[0], 150 + offset[1]))
    surface.blit(quantum_text, (WIDTH//2 - 150, 150))
    
    # CRASH text
    crash_text = pygame.font.Font(None, 80).render("CRASH", True, (255, 165, 0))
    # Add glow effect
    for offset in [(3, 3), (-3, -3), (3, -3), (-3, 3), (0, 3), (0, -3), (3, 0), (-3, 0)]:
        glow_surface = pygame.font.Font(None, 80).render("CRASH", True, (int(glow_intensity), int(glow_intensity/2), 0))
        surface.blit(glow_surface, (WIDTH//2 - 100 + offset[0], 220 + offset[1]))
    surface.blit(crash_text, (WIDTH//2 - 100, 220))
    
    # Subtitle
    subtitle_text = FONT.render("THE MULTIVERSE MAINTENANCE MANUAL", True, (100, 255, 200))
    subtitle_rect = subtitle_text.get_rect(center=(WIDTH//2, 290))
    surface.blit(subtitle_text, subtitle_rect)
    
    # Loading elements with glitch effects
    loading_dots = "." * ((loading_time // 20) % 4)
    loading_text = FONT.render(f"Initializing Reality{loading_dots}", True, WHITE)
    loading_rect = loading_text.get_rect(center=(WIDTH//2, 400))
    
    # Add glitch effect to loading text
    if random.randint(1, 10) == 1:
        glitch_colors = [(255, 0, 255), (0, 255, 255), (255, 255, 0)]
        glitch_color = random.choice(glitch_colors)
        glitch_offset = (random.randint(-2, 2), random.randint(-2, 2))
        glitch_text = FONT.render(f"Initializing Reality{loading_dots}", True, glitch_color)
        surface.blit(glitch_text, (loading_rect.x + glitch_offset[0], loading_rect.y + glitch_offset[1]))
    
    surface.blit(loading_text, loading_rect)
    
    # Progress bar
    bar_width = 300
    bar_height = 20
    bar_x = WIDTH // 2 - bar_width // 2
    bar_y = 450
    
    pygame.draw.rect(surface, (50, 50, 50), (bar_x - 2, bar_y - 2, bar_width + 4, bar_height + 4))
    pygame.draw.rect(surface, BLACK, (bar_x, bar_y, bar_width, bar_height))
    
    # Animated progress
    progress = min(loading_time / 300, 1.0)  # 5 seconds loading time
    progress_width = int(bar_width * progress)
    
    # Rainbow progress bar
    progress_colors = [
        (255, 0, 255),   # Magenta
        (0, 255, 255),   # Cyan
        (255, 255, 0),   # Yellow
        (255, 165, 0),   # Orange
        (0, 255, 0)      # Green
    ]
    
    for i in range(progress_width):
        color_index = int((i / bar_width) * len(progress_colors))
        color_index = min(color_index, len(progress_colors) - 1)
        color = progress_colors[color_index]
        pygame.draw.line(surface, color, (bar_x + i, bar_y), (bar_x + i, bar_y + bar_height))
    
    # Multiverse elements (floating geometric shapes)
    time_offset = pygame.time.get_ticks() / 1000
    
    # Floating cubes
    for i in range(5):
        cube_x = 100 + i * 150 + math.sin(time_offset + i) * 20
        cube_y = 350 + math.cos(time_offset + i * 0.5) * 15
        cube_size = 15 + math.sin(time_offset * 2 + i) * 5
        
        # Cube shadow
        shadow_points = [
            (cube_x + cube_size + 5, cube_y + cube_size + 5),
            (cube_x + cube_size * 2 + 5, cube_y + 5),
            (cube_x + cube_size * 2 + 5, cube_y + cube_size + 5),
            (cube_x + cube_size + 5, cube_y + cube_size * 2 + 5)
        ]
        pygame.draw.polygon(surface, (20, 20, 20), shadow_points)
        
        # Main cube
        cube_points = [
            (cube_x, cube_y + cube_size),
            (cube_x + cube_size, cube_y),
            (cube_x + cube_size * 2, cube_y + cube_size),
            (cube_x + cube_size, cube_y + cube_size * 2)
        ]
        cube_colors = [(0, 100, 200), (0, 150, 255), (100, 200, 255)]
        pygame.draw.polygon(surface, random.choice(cube_colors), cube_points)
    
    # Project info smaller
    project_text = SMALLFONT.render("School Project 12/07/2025", True, (150, 150, 200))
    project_rect = project_text.get_rect(center=(WIDTH//2, 480))
    surface.blit(project_text, project_rect)
    
    # Instruction text
    if loading_time > 200:  # Show after a delay
        instruction_text = SMALLFONT.render("Press SPACE to begin your quantum journey...", True, (200, 200, 200))
        instruction_rect = instruction_text.get_rect(center=(WIDTH//2, 500))
        
        # Blinking effect
        alpha = int(255 * (0.5 + 0.5 * math.sin(time_offset * 4)))
        instruction_surface = pygame.Surface(instruction_text.get_size(), pygame.SRCALPHA)
        instruction_surface.fill((*instruction_text.get_at((0, 0))[:3], alpha))
        instruction_text.set_alpha(alpha)
        surface.blit(instruction_text, instruction_rect)
    
    # Version text
    version_text = pygame.font.Font(None, 16).render("v1.0 Final Edition", True, (100, 100, 100))
    surface.blit(version_text, (10, HEIGHT - 20))

def draw_manual_screen(surface):
    global manual_scroll
    
    # Dark background with subtle gradient
    for y in range(HEIGHT):
        color_ratio = y / HEIGHT
        r = int(15 + color_ratio * 10)
        g = int(15 + color_ratio * 15)
        b = int(25 + color_ratio * 25)
        pygame.draw.line(surface, (r, g, b), (0, y), (WIDTH, y))
    
    # Draw some background stars
    for star in background_stars[::3]:  # Use fewer stars
        brightness = star[2] // 2
        color = (brightness, brightness, brightness)
        pygame.draw.circle(surface, color, (int(star[0]), int(star[1])), 1)
    
    # Manual content
    y_offset = 50 - manual_scroll
    
    # Title
    title_text = pygame.font.Font(None, 48).render("QUANTUM CRASH - USER MANUAL", True, (0, 255, 255))
    title_rect = title_text.get_rect(center=(WIDTH//2, y_offset))
    surface.blit(title_text, title_rect)
    
    y_offset += 80
    
    # What Is This Game section
    section_font = pygame.font.Font(None, 32)
    content_font = pygame.font.Font(None, 24)
    small_font = pygame.font.Font(None, 20)
    
    # Section: What Is This Game?
    what_title = section_font.render("What Is This Game?", True, (255, 215, 0))
    surface.blit(what_title, (50, y_offset))
    y_offset += 40
    
    what_text = [
        "Reality is collapsing and only your cricket bat can save the multiverse.",
        "You play Rizik, a dimension-hopping janitor armed with a glitch-splitting bat.",
        "Smash anomalies, unlock surreal glitch thoughts, and face off against your",
        "cheating past self in the final boss showdown."
    ]
    
    for line in what_text:
        text = content_font.render(line, True, WHITE)
        surface.blit(text, (70, y_offset))
        y_offset += 25
    
    y_offset += 20
    
    # Section: Core Mechanics
    mechanics_title = section_font.render("🧪 Core Mechanics", True, (255, 100, 255))
    surface.blit(mechanics_title, (50, y_offset))
    y_offset += 40
    
    mechanics_text = [
        "• Smash falling glitches to gain Reality Points",
        "• Unlock Level 2 & 3 with increasing glitch speed and size",
        "• Face your Past Self in Boss Mode at score ≥ 30",
        "• Answer bizarre philosophical questions mid-game (just because)",
        "• Reach Galaxy Mode after defeating your past self",
        "• Conquer 5 galaxy levels to become the Galaxy Master"
    ]
    
    for line in mechanics_text:
        text = content_font.render(line, True, (200, 255, 200))
        surface.blit(text, (70, y_offset))
        y_offset += 25
    
    y_offset += 20
    
    # Section: Controls
    controls_title = section_font.render("🎯 Controls", True, (255, 165, 0))
    surface.blit(controls_title, (50, y_offset))
    y_offset += 40
    
    controls_text = [
        "← / → Arrow Keys — Move Rizik left and right",
        "SPACE — Swing bat in Boss Mode to damage your past self",
        "R — Reboot reality after victory",
        "Q — Quit when you've had enough multiverse madness"
    ]
    
    for line in controls_text:
        text = content_font.render(line, True, (255, 255, 150))
        surface.blit(text, (70, y_offset))
        y_offset += 25
    
    y_offset += 20
    
    # Section: Bonus Features
    bonus_title = section_font.render("🎬 Bonus Features", True, (0, 255, 150))
    surface.blit(bonus_title, (50, y_offset))
    y_offset += 40
    
    bonus_text = [
        "• Dynamic level progression with visual effects",
        "• Philosophical glitch pop-ups that make you question reality",
        "• Multiple endings: Defeat boss or conquer the galaxy",
        "• Particle effects, screen shake, and cosmic visuals",
        "• Playable completely in Replit with just one click!"
    ]
    
    for line in bonus_text:
        text = content_font.render(line, True, (150, 255, 255))
        surface.blit(text, (70, y_offset))
        y_offset += 25
    
    y_offset += 30
    
    # Instructions to continue
    continue_text = section_font.render("Press SPACE to begin your quantum journey!", True, (255, 255, 0))
    continue_rect = continue_text.get_rect(center=(WIDTH//2, y_offset))
    
    # Blinking effect
    time_offset = pygame.time.get_ticks() / 1000
    alpha = int(255 * (0.5 + 0.5 * math.sin(time_offset * 3)))
    continue_text.set_alpha(alpha)
    surface.blit(continue_text, continue_rect)
    
    # Scroll instructions if content is too long
    if y_offset > HEIGHT - 100:
        scroll_text = small_font.render("Use ↑↓ arrows to scroll", True, (150, 150, 150))
        scroll_rect = scroll_text.get_rect(center=(WIDTH//2, HEIGHT - 30))
        surface.blit(scroll_text, scroll_rect)

# Game loop
running = True
while running:
    clock.tick(60)
    
    # Loading screen phase
    if loading_screen:
        loading_time += 1
        
        # Create loading particles
        if loading_time % 10 == 0:
            create_loading_particles()
        
        # Update background stars
        update_background()
        
        # Draw loading screen
        draw_loading_screen(screen)
        pygame.display.flip()
        
        # Handle loading screen events
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE and loading_time > 200:
                    loading_screen = False
                    manual_screen = True
                    loading_particles.clear()
        
        # Auto-advance after 8 seconds
        if loading_time > 480:
            loading_screen = False
            manual_screen = True
            loading_particles.clear()
        
        continue
    
    # Manual screen phase
    if manual_screen:
        # Update background stars
        update_background()
        
        # Draw manual screen
        draw_manual_screen(screen)
        pygame.display.flip()
        
        # Handle manual screen events
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE:
                    manual_screen = False
                elif event.key == pygame.K_UP:
                    manual_scroll = max(0, manual_scroll - 20)
                elif event.key == pygame.K_DOWN:
                    manual_scroll = min(300, manual_scroll + 20)
        
        continue

    if not game_over:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False

        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT] and player.x > 5:
            player.x -= 5
        if keys[pygame.K_RIGHT] and player.x < WIDTH - 55:
            player.x += 5

        bat.x = player.x + 20
        bat.y = player.y - 60

        # Update background
        update_background()

        # Add glitches until boss mode
        if not boss_mode and not galaxy_mode and score < 30:
            glitch_spawn_timer += 1
            spawn_rate = 25
            if glitch_spawn_timer >= spawn_rate:
                glitches.append(pygame.Rect(random.randint(0, WIDTH - 40), 0, 40, 40))
                glitch_spawn_timer = 0

        # Glitch movement and collision (faster after boss)
        glitch_speed = 8 if boss_mode or galaxy_mode else 5
        for g in glitches[:]:
            g.y += glitch_speed
            if g.colliderect(player) or g.colliderect(bat):
                score += 1
                glitches.remove(g)
                # Create particle explosion
                create_explosion(g.x + 20, g.y + 20, CYAN, 15)
                screen_shake = 5
            elif g.y > HEIGHT:
                glitches.remove(g)

        # Galaxy mode enemy spawning and movement
        if galaxy_mode:
            # Spawn galaxy enemies more frequently and faster
            if random.randint(1, 8 - galaxy_level) == 1:
                enemy_types = ["star", "comet", "asteroid"]
                enemy_type = random.choice(enemy_types)
                enemy_size = random.randint(30, 50)
                enemy = {
                    "rect": pygame.Rect(random.randint(0, WIDTH - enemy_size), 0, enemy_size, enemy_size),
                    "type": enemy_type,
                    "speed": random.randint(6 + galaxy_level, 10 + galaxy_level * 2)
                }
                galaxy_enemies.append(enemy)

            # Move and check collision for galaxy enemies
            for enemy in galaxy_enemies[:]:
                enemy["rect"].y += enemy["speed"]
                if enemy["rect"].colliderect(player) or enemy["rect"].colliderect(bat):
                    score += 2  # Galaxy enemies give more points
                    galaxy_enemies.remove(enemy)
                    # Different explosion colors for different enemy types
                    explosion_colors = {"star": (255, 215, 0), "comet": (0, 191, 255), "asteroid": (139, 69, 19)}
                    create_explosion(enemy["rect"].x + 20, enemy["rect"].y + 20, explosion_colors[enemy["type"]], 20)
                    screen_shake = 7
                elif enemy["rect"].y > HEIGHT:
                    galaxy_enemies.remove(enemy)

        # Trigger boss mode
        if score >= 30 and not boss_mode and not galaxy_mode:
            boss_mode = True
            glitches.clear()
            create_explosion(WIDTH // 2, HEIGHT // 2, RED, 50)
            screen_shake = 20

        # Boss battle
        if boss_mode:
            # Keep spawning faster glitches during boss fight
            if random.randint(1, 8) == 1:
                glitches.append(pygame.Rect(random.randint(0, WIDTH - 40), 0, 40, 40))
            
            if keys[pygame.K_SPACE]:
                boss_hp -= 1
                create_particles(WIDTH // 2, HEIGHT // 2, RED, 20)
                screen_shake = 8
            if boss_hp <= 0:
                boss_mode = False
                galaxy_mode = True
                galaxy_level = 1
                glitches.clear()
                create_explosion(WIDTH // 2, HEIGHT // 2, (255, 215, 0), 100)

        # Galaxy progression
        if galaxy_mode:
            # Check if player has enough score to advance galaxy level
            required_score = 30 + (galaxy_level * 15)
            if score >= required_score and galaxy_level < max_galaxy_level:
                galaxy_level += 1
                create_explosion(WIDTH // 2, HEIGHT // 2, (255, 215, 0), 50)
                screen_shake = 15
            elif galaxy_level >= max_galaxy_level and score >= 30 + (max_galaxy_level * 15):
                victory = True
                game_over = True
                create_explosion(WIDTH // 2, HEIGHT // 2, (255, 215, 0), 200)

        draw()

    else:
        # Gradient background for end screen
        for y in range(HEIGHT):
            if victory:
                # Galaxy victory - cosmic gradient
                color_ratio = y / HEIGHT
                r = int(20 + color_ratio * 30)
                g = int(10 + color_ratio * 25)
                b = int(50 + color_ratio * 40)
            else:
                # Boss victory - green tech gradient
                color_ratio = y / HEIGHT
                r = int(10 + color_ratio * 15)
                g = int(30 + color_ratio * 50)
                b = int(10 + color_ratio * 20)
            pygame.draw.line(screen, (r, g, b), (0, y), (WIDTH, y))
        
        # Victory particles
        if random.randint(1, 3) == 1:
            if victory:
                # Galaxy victory - gold and white particles
                create_particles(random.randint(0, WIDTH), random.randint(0, HEIGHT), (255, 215, 0), 5)
                create_particles(random.randint(0, WIDTH), random.randint(0, HEIGHT), (255, 255, 255), 3)
            else:
                # Boss victory - green and cyan particles
                create_particles(random.randint(0, WIDTH), random.randint(0, HEIGHT), GREEN, 3)
                create_particles(random.randint(0, WIDTH), random.randint(0, HEIGHT), CYAN, 2)
        
        # Draw particles
        for particle in particles[:]:
            particle.update()
            if particle.life <= 0:
                particles.remove(particle)
            else:
                particle.draw(screen)
        
        if victory:
            end_text = BIGFONT.render("🌌 GALAXY MASTER - UNIVERSE CONQUERED! 🌌", True, (255, 215, 0))
            sub_text = FONT.render("You have transcended reality itself!", True, (255, 255, 255))
            achievement_text = FONT.render("🏆 Achievement Unlocked: Multiverse Champion 🏆", True, (255, 215, 0))
            choice_text = FONT.render("Press R to Restart the Universe or Q to Quit", True, WHITE)
            # Large creator credit
            creator_font = pygame.font.Font(None, 36)
            creator_credit = creator_font.render("Game by Nirenjan Nair", True, (255, 215, 0))
            creator_rect = creator_credit.get_rect(center=(WIDTH//2, 320))
            
            # Add cosmic glow to victory text
            for offset in [(3, 3), (-3, -3), (3, -3), (-3, 3)]:
                glow_text = BIGFONT.render("🌌 GALAXY MASTER - UNIVERSE CONQUERED! 🌌", True, (100, 100, 0))
                screen.blit(glow_text, (50 + offset[0], 160 + offset[1]))
            
            # Add glow to creator name
            for offset in [(2, 2), (-2, -2), (2, -2), (-2, 2)]:
                glow_creator = creator_font.render("Game by Nirenjan Nair", True, (100, 100, 0))
                screen.blit(glow_creator, (creator_rect.x + offset[0], creator_rect.y + offset[1]))
            
            screen.blit(end_text, (50, 160))
            screen.blit(sub_text, (220, 200))
            screen.blit(achievement_text, (150, 230))
            screen.blit(choice_text, (180, 260))
            
            # Draw victorious boy with bat
            draw_victory_boy_with_bat(screen, WIDTH // 2 - 50, HEIGHT - 240, 1.5)
            
            # Creator credit at bottom
            creator_rect = creator_credit.get_rect(center=(WIDTH//2, HEIGHT - 50))
            screen.blit(creator_credit, creator_rect)
            
            # Project date smaller
            date_text = SMALLFONT.render("12/07/2025", True, (150, 150, 200))
            date_rect = date_text.get_rect(center=(WIDTH//2, HEIGHT - 20))
            screen.blit(date_text, date_rect)
        else:
            end_text = BIGFONT.render("✅ You Defeated Your Past Self", True, GREEN)
            achievement_text = FONT.render("🎯 Achievement: Reality Debugger", True, (0, 255, 100))
            choice_text = FONT.render("Press R to Reboot Reality or Q to Quit", True, WHITE)
            # Large creator credit
            creator_font = pygame.font.Font(None, 36)
            creator_credit = creator_font.render("Game by Nirenjan Nair", True, (0, 255, 100))
            creator_rect = creator_credit.get_rect(center=(WIDTH//2, 310))
            
            # Add glow to victory text
            for offset in [(2, 2), (-2, -2), (2, -2), (-2, 2)]:
                glow_text = BIGFONT.render("✅ You Defeated Your Past Self", True, (0, 100, 0))
                screen.blit(glow_text, (150 + offset[0], 180 + offset[1]))
            
            # Add glow to creator name
            for offset in [(2, 2), (-2, -2), (2, -2), (-2, 2)]:
                glow_creator = creator_font.render("Game by Nirenjan Nair", True, (0, 100, 0))
                screen.blit(glow_creator, (creator_rect.x + offset[0], creator_rect.y + offset[1]))
            
            screen.blit(end_text, (150, 180))
            screen.blit(achievement_text, (200, 220))
            screen.blit(choice_text, (190, 250))
            
            # Draw victorious boy with bat (smaller for boss victory)
            draw_victory_boy_with_bat(screen, WIDTH // 2 - 30, HEIGHT - 200, 1.2)
            
            # Creator credit at bottom
            creator_rect = creator_credit.get_rect(center=(WIDTH//2, HEIGHT - 50))
            screen.blit(creator_credit, creator_rect)
            
            # Project date smaller
            date_text = SMALLFONT.render("12/07/2025", True, (150, 150, 200))
            date_rect = date_text.get_rect(center=(WIDTH//2, HEIGHT - 20))
            screen.blit(date_text, date_rect)
        
        pygame.display.flip()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_r:
                    # Reset everything
                    score = 0
                    boss_hp = max_boss_hp
                    boss_mode = False
                    galaxy_mode = False
                    galaxy_level = 1
                    game_over = False
                    victory = False
                    player.x = WIDTH // 2
                    particles.clear()
                    player_trail.clear()
                    galaxy_enemies.clear()
                    glitches.clear()
                    screen_shake = 0
                    glitch_spawn_timer = 0
                    manual_scroll = 0
                if event.key == pygame.K_q:
                    running = False

pygame.quit()
