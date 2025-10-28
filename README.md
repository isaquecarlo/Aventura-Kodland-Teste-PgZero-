# Aventura-Kodland-Teste-PgZero-


#Jogo de aventura top-down criado com Python e PgZero, cumprindo requisitos de teste (OOP, animação, estado de jogo, etc.).       _Créditos: Assets de imagem por O_LOBSTER (itch.io)
# -*- coding: utf-8 -*-
import pgzrun
from pgzero.actor import Actor
import random
import math
from pygame import Rect
from pgzero.builtins import images, music, sounds

WIDTH = 800
HEIGHT = 640
TITLE = "AVENTURA KODLAND"

FRAME_WIDTH = 16
FRAME_HEIGHT = 16
ANIMATION_SPEED = 0.15
TILE_SIZE = 32


class Player(Actor):
    def __init__(self, **kwargs):
        super().__init__('hero_idle_down', anchor=('center', 'center'), **kwargs)
        self.animations = {
            'idle_down': self._surf,
            'run_down': images.load('hero_run_down.png'),
            'run_up': images.load('hero_run_up.png'),
            'run_left': images.load('hero_run_left.png'),
            'run_right': images.load('hero_run_right.png')
        }
        self.frames_per_anim = {'idle_down': 6, 'run_down': 6, 'run_up': 6, 'run_left': 6, 'run_right': 6}
        self.current_anim = 'idle_down'
        self.current_frame = 0
        self.animation_timer = 0
        self.frames = {}
        for anim_name, strip_image in self.animations.items():
            num_frames = self.frames_per_anim[anim_name]
            self.frames[anim_name] = [Rect((i * FRAME_WIDTH, 0), (FRAME_WIDTH, FRAME_HEIGHT)) for i in range(num_frames)]

    def animate(self):
        self.animation_timer += 1
        if self.animation_timer > (ANIMATION_SPEED * 60):
            self.animation_timer = 0
            num_frames = self.frames_per_anim[self.current_anim]
            self.current_frame = (self.current_frame + 1) % num_frames
            strip_image = self.animations[self.current_anim]
            frame_rect = self.frames[self.current_anim][self.current_frame]
            self._surf = strip_image.subsurface(frame_rect)

    def update(self):
        last_anim = self.current_anim
        dx, dy = 0, 0
        if keyboard.left: dx = -3; self.current_anim = 'run_left'
        elif keyboard.right: dx = 3; self.current_anim = 'run_right'
        elif keyboard.up: dy = -3; self.current_anim = 'run_up'
        elif keyboard.down: dy = 3; self.current_anim = 'run_down'
        else: self.current_anim = 'idle_down'
        if self.current_anim != last_anim:
            self.current_frame = 0
            self.animation_timer = 0
        self.x += dx
        self.y += dy
        if self.left < 0: self.left = 0
        if self.right > WIDTH: self.right = WIDTH
        if self.top < 0: self.top = 0
        if self.bottom > HEIGHT: self.bottom = HEIGHT
        self.animate()

    def reset_position(self):
        self.pos = (50, HEIGHT - 50)

class Enemy(Actor):
    def __init__(self, start_pos, territory_left, territory_right, **kwargs):
        super().__init__('spider_idle', anchor=('center', 'center'), pos=start_pos, **kwargs)
        self.animations = {'idle': self._surf, 'run': images.load('spider_run.png')}
        self.frames_per_anim = {'idle': 4, 'run': 4}
        self.current_anim = 'run'
        self.current_frame = 0
        self.animation_timer = 0
        self.speed = random.uniform(0.8, 1.5)
        self.direction = random.choice([1, -1])
        self.territory_left = territory_left
        self.territory_right = territory_right
        self.x = max(territory_left + FRAME_WIDTH, min(self.x, territory_right - FRAME_WIDTH))
        self.frames = {}
        for anim_name, strip_image in self.animations.items():
            num_frames = self.frames_per_anim[anim_name]
            self.frames[anim_name] = [Rect((i * FRAME_WIDTH, 0), (FRAME_WIDTH, FRAME_HEIGHT)) for i in range(num_frames)]

    def animate(self):
        self.animation_timer += 1
        if self.animation_timer > (ANIMATION_SPEED * 60):
            self.animation_timer = 0
            num_frames = self.frames_per_anim[self.current_anim]
            self.current_frame = (self.current_frame + 1) % num_frames
            strip_image = self.animations[self.current_anim]
            frame_rect = self.frames[self.current_anim][self.current_frame]
            self._surf = strip_image.subsurface(frame_rect)

    def update(self):
        self.current_anim = 'run'
        self.x += self.direction * self.speed
        if self.x > self.territory_right or self.x < self.territory_left:
            self.direction *= -1
            if self.x > self.territory_right: self.x = self.territory_right - 1
            if self.x < self.territory_left: self.x = self.territory_left + 1
        self.animate()

player = Player()
star = Actor('star', pos=(100, 100))

enemies = []
section_width = WIDTH / 3
enemy_positions = [
    ((section_width / 2, 100), 0, section_width),
    ((WIDTH / 2, HEIGHT / 2), section_width, 2 * section_width),
    ((WIDTH - section_width / 2, HEIGHT - 100), 2 * section_width, WIDTH),
    ((section_width / 2, HEIGHT - 200), 0, section_width),
    ((WIDTH - section_width / 2, 200), 2 * section_width, WIDTH)
]
for pos, t_left, t_right in enemy_positions:
    enemies.append(Enemy(start_pos=pos, territory_left=t_left, territory_right=t_right))

game_state = "menu"
music_on = True
pontos = 0
vidas = 5

start_button = Rect(WIDTH / 2 - 100, HEIGHT / 2 - 50, 200, 40)
music_button = Rect(WIDTH / 2 - 100, HEIGHT / 2 + 10, 200, 40)
exit_button = Rect(WIDTH / 2 - 100, HEIGHT / 2 + 70, 200, 40)


try:
    menu_background = images.load('background.png')

    if menu_background.get_width() != WIDTH or menu_background.get_height() != HEIGHT:
        print(f"AVISO: Imagem 'background.png' ({menu_background.get_width()}x{menu_background.get_height()}) "
              f"não tem o tamanho exato da tela ({WIDTH}x{HEIGHT}). Será desenhada a partir de (0,0).")
except Exception as e:
    menu_background = None
    print(f"AVISO: Imagem 'background.png' não carregada para o menu. Erro: {e}")


try:
    tile_images = [images.load('tile_dark.png'), images.load('tile_sand.png'), images.load('tile_water.png')]
    if tile_images[0].get_width() != TILE_SIZE or tile_images[0].get_height() != TILE_SIZE:
         print(f"AVISO: Tamanho do tile ({tile_images[0].get_width()}x{tile_images[0].get_height()}) não corresponde a TILE_SIZE ({TILE_SIZE}).")
except FileNotFoundError:
    tile_images = None
    print("AVISO: Uma ou mais imagens de tile não encontradas.")

section_1_end = round(WIDTH / 3)
section_2_end = round(2 * WIDTH / 3)
territory_rects = [
    Rect(0, 0, section_1_end, HEIGHT),
    Rect(section_1_end, 0, section_2_end - section_1_end, HEIGHT),
    Rect(section_2_end, 0, WIDTH - section_2_end, HEIGHT)
]
territory_colors = [(0, 0, 0), (245, 245, 220), (135, 206, 250)]

VERY_LIGHT_GRAY = (220, 220, 220)
DARK_GRAY = (100, 100, 100)
BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
YELLOW = (255, 255, 0)
RED = (200, 0, 0)

def pos_star():
    star.x = random.randint(50, WIDTH - 50)
    star.y = random.randint(50, HEIGHT - 50)

def update_music():
    if music_on:
        if not music.is_playing('sound.loop'): music.play('sound.loop')
        music.set_volume(0.5)
    else:
        music.stop()

def draw():
    if game_state == "menu":
        if menu_background:
            screen.blit(menu_background, (0, 0))
        else:
            screen.fill((20, 20, 80))
        
            
        
        screen.draw.text(TITLE.upper(), center=(WIDTH / 2, HEIGHT / 3), fontsize=60, color=VERY_LIGHT_GRAY)
        screen.draw.filled_rect(start_button, "green")
        screen.draw.text("COMEÇAR O JOGO", center=start_button.center, fontsize=20, color=BLACK)
        music_text = f"MÚSICA: {'ON' if music_on else 'OFF'}"
        screen.draw.filled_rect(music_button, "orange")
        screen.draw.text(music_text.upper(), center=music_button.center, fontsize=20, color=BLACK)
        screen.draw.filled_rect(exit_button, "red")
        screen.draw.text("SAIR", center=exit_button.center, fontsize=20, color=BLACK)
        screen.draw.text("ASSETS POR O_LOBSTER (ITCH.IO)", bottomleft=(10, HEIGHT - 10), fontsize=15, color="gray")
        

    elif game_state == "playing":
        if tile_images:
            for i in range(len(territory_rects)):
                rect_area = territory_rects[i]
                tile_img = tile_images[i]
                for y in range(rect_area.top, rect_area.bottom, TILE_SIZE):
                    for x in range(rect_area.left, rect_area.right, TILE_SIZE):
                        screen.blit(tile_img, (x, y))
        else:
            for i in range(len(territory_rects)):
                screen.draw.filled_rect(territory_rects[i], territory_colors[i])
        player.draw(); star.draw()
        for enemy in enemies: enemy.draw()
     
        screen.draw.text(f"PONTOS: {pontos}", (10, 10), color='white', fontsize=30)
        screen.draw.text(f"VIDAS: {vidas}", (WIDTH - 150, 10), color=BLACK, fontsize=30)
        

    elif game_state == "game_over":
        screen.fill(RED)
        screen.draw.text("GAME OVER", center=(WIDTH / 2, HEIGHT / 2 - 50), fontsize=50, color=WHITE)
        screen.draw.text(f"PONTUAÇÃO FINAL: {pontos}", center=(WIDTH / 2, HEIGHT / 2 + 20), fontsize=25, color=YELLOW)
        screen.draw.text("CLIQUE PARA VOLTAR AO MENU", center=(WIDTH / 2, HEIGHT / 2 + 70), fontsize=25, color=WHITE)

def update():
    global vidas, game_state, pontos
    if game_state == "playing":
        player.update()
        for enemy in enemies: enemy.update()
        if player.colliderect(star):
            pontos += 1
            sounds.click.play()
            pos_star()
        for enemy in enemies:
            if player.colliderect(enemy):
                vidas -= 1
                sounds.hit.play()
                player.reset_position()
                if vidas <= 0:
                    game_state = "game_over"
                    music.stop()
                    print("GAME OVER!")
                    break

def on_mouse_down(pos, button):
    global game_state, music_on, vidas, pontos
    if game_state == "menu":
        if button == mouse.LEFT:
            if start_button.collidepoint(pos):
                vidas = 5; pontos = 0; game_state = "playing"
                player.reset_position(); pos_star()
                update_music(); sounds.click.play()
            elif music_button.collidepoint(pos):
                music_on = not music_on; update_music(); sounds.click.play()
            elif exit_button.collidepoint(pos):
                music.stop(); quit()
    elif game_state == "game_over":
         if button == mouse.LEFT: game_state = "menu"

pgzrun.go()
