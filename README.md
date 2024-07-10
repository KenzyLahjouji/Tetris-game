# Tetris-game

import pygame
import sys
import random

pygame.init()

screen_width = 800
screen_height = 600

lightblue = (153, 204, 255)
white = (255, 255, 255)
black = (0, 0, 0)

screen = pygame.display.set_mode((screen_width, screen_height))
pygame.display.set_caption('Tetris Introduction')

font = pygame.font.Font(None, 36)

def draw_text(text, font, color, surface, x, y):
    textobj = font.render(text, True, color)
    textrect = textobj.get_rect()
    textrect.topleft = (x, y)
    surface.blit(textobj, textrect)

def introduction_screen():
    while True:
        screen.fill(lightblue)
        draw_text('Welcome to this new Tetris!', font, white, screen, 20, 20)
        draw_text('How to play:', font, white, screen, 20, 60)
        draw_text('1 - Use the key down to move the piece faster', font, white, screen, 20, 100)
        draw_text('2 - Use the keys left and right to move the piece', font, white, screen, 20, 140)
        draw_text('3 - Use the key up to rotate the piece', font, white, screen, 20, 180)
        draw_text('Press the space key to start the game', font, white, screen, 20, 220)

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if event.type == pygame.KEYDOWN:
                main_game()
        

        pygame.display.update()

def main_game():

    shapecolors = [
    (255, 255, 0), #Yellow
    (153, 51, 255),#Purple
    (0, 255, 0), #Green
    (255, 0, 0), #Red
    (0, 255, 255), #Cyan
    (255, 0, 255), #Pink
    (0, 0, 255), #Blue
    (76, 153, 0), #Dark Green
    ]

    class Shapes:
        x=0
        y=0

        shapes=[
            [[0,4,8,12],[0,1,2,3]],#Basic Line'''
            [[0,4,1,5]],#Square'''
            [[0,4,8,1,5,9],[4,5,6,8,9,10]],#Rectangle'''
            [[1,4,5,6],[1,5,9,6],[4,5,6,9],[1,5,9,4]],#T'''
            [[13,14,10,11],[13,9,8,4]],#Z1'''
            [[14,13,9,8],[14,10,11,7]],#Z2'''
            [[4,5,6,8,9,10,12,13,14]],#Cube'''
            [[4,8,12,13,14],[12,13,14,10,6],[14,10,6,5,4], [6,5,4,8,12]],#L'''
        ]

        def __init__(self,x,y):
            self.x=x
            self.y=y
            self.type=random.randint(0, len(self.shapes)-1)
            self.color=random.randint(1, len(shapecolors)-1)
            self.rotation=0
            
        def display(self):
            return self.shapes[self.type][self.rotation]
    
        def rotate(self):
            self.rotation=(self.rotation+1)% len(self.shapes[self.type])

    class tetristry1:
    
        def __init__(self, height, width):
            self.score=0
            self.level=1
            self.state="Start"
            self.field=[]
            self.height = 0
            self.width = 0
            self.x = 100
            self.y = 60
            self.zoom = 20
            self.shapes = None

            self.height = height
            self.width = width
            self.field = []
            self.score = 0
            self.state = "Start"

            for i in range (height):
                '''On reinitilalise une nouvelle ligne pour la nouvelle forme'''
                new_line=[]
                for j in range(width):
                    new_line.append(0)
                self.field.append(new_line)

        def newshape(self):
            '''On cree une nouvelle forme pour continuer le jeu'''
            self.shapes=Shapes(3,0)

        def possibleintersection(self):
            '''Est ce que ça depasse du champ'''
            intersection = False
            for i in range(4):
                for j in range(4):
                    if i * 4 + j in self.shapes.display():
                        if i + self.shapes.y > self.height - 1 or \
                                j + self.shapes.x > self.width - 1 or \
                                j + self.shapes.x < 0 or \
                                self.field[i + self.shapes.y][j + self.shapes.x] > 0:
                            intersection=True
            return intersection

        def eliminating_lines(self):
            lines_cleared = 0
            for i in range(self.height):
                if all(self.field[i][j] != 0 for j in range(self.width)):
                    lines_cleared += 1
            # Shift down lines above the cleared line
                    for i1 in range(i, 0, -1):
                        for j in range(self.width):
                            self.field[i1][j] = self.field[i1 - 1][j]
            # Clear the topmost line
                    for j in range(self.width):
                        self.field[0][j] = 0

            self.score += lines_cleared  # Increment score by number of lines cleared

            return lines_cleared
    
        def moveshapes(self):
            while not self.eliminating_lines():
                self.shapes.y += 1
            self.shapes.y -=1
            self.freeze()

        def enddown(self):
            self.shapes.y += 1
            if self.possibleintersection():
                self.shapes.y-=1
                self.freeze()

        def freeze(self):
            for i in range(4):
                for j in range(4):
                    if i * 4 + j in self.shapes.display():
                        self.field[i + self.shapes.y][j + self.shapes.x] = self.shapes.color
            self.eliminating_lines()
            self.newshape()
            if self.possibleintersection():
                self.state = "gameover"


        def movetotheside(self, dx):
            oldx=self.shapes.x
            self.shapes.x+=dx
            if self.possibleintersection():
                self.shapes.x=oldx

        def rotate(self):
            oldrotation=self.shapes.rotation
            self.shapes.rotate()
            if self.possibleintersection():
                self.shapes.rotation=oldrotation

    pygame.init()

    music=pygame.mixer.music.load("Original Tetris theme (Tetris Soundtrack).mp3")
    pygame.mixer.music.play(-1)

    lightblue=(153,204,255)
    black=(0,0,0)
    white=(255,255,255)

    size=(600,700)
    screen=pygame.display.set_mode(size)
    pygame.display.set_caption("Kenzy's tetris")

    done=False
    clock=pygame.time.Clock()
    fps=20
    game=tetristry1(30,15)
    counter=0

    pressing_down=False

    while not done:
        if game.shapes is None:
            game.newshape()
        counter+=1

        if counter % (fps // game.level // 1) == 0 or pressing_down:
            if game.state == "Start":
                game.enddown()

        for event in pygame.event.get():
            if event.type==pygame.QUIT:
                done=True
            if event.type==pygame.KEYDOWN:
                if event.key==pygame.K_UP:
                    game.rotate()
                if event.key==pygame.K_DOWN:
                    pressing_down=True
                if event.key==pygame.K_LEFT:
                    game.movetotheside(-1)
                if event.key==pygame.K_RIGHT:
                    game.movetotheside(1)
                if event.key==pygame.K_SPACE:
                    game.moveshapes()
                if event.key == pygame.K_ESCAPE:
                    game.__init__(20, 10)

            if event.type == pygame.KEYUP:
                if event.key == pygame.K_DOWN:
                    pressing_down = False

        screen.fill(lightblue)

        for i in range(game.height):
            for j in range(game.width):
                pygame.draw.rect(screen, black, [game.x + game.zoom * j, game.y + game.zoom * i, game.zoom, game.zoom], 1)
                if game.field[i][j] > 0:
                    pygame.draw.rect(screen, shapecolors[game.field[i][j]],
                                     [game.x + game.zoom * j + 1, game.y + game.zoom * i + 1, game.zoom - 2, game.zoom - 1])
        if game.shapes is not None:
            for i in range (4):
                for j in range(4):
                    p = i * 4 + j
                    if p in game.shapes.display():
                        pygame.draw.rect(screen, shapecolors[game.shapes.color],
                                         [game.x + game.zoom * (j + game.shapes.x) + 1,
                                          game.y + game.zoom * (i + game.shapes.y) + 1,
                                          game.zoom - 2, game.zoom - 2])

        Font1=pygame.font.SysFont('Arial', 25, True, False)
        Font2=pygame.font.SysFont('Arial', 65, True, False)
        Text=Font1.render("Score: " + str(game.score), True, white)
        Textwhengameover1=Font2.render("Game Over", True, (255, 0, 0))
        Textwhengameover2=Font2.render("Press esc",True, black)

        screen.blit(Text, [0, 0])
        if game.state=="gameover":
            screen.blit(Textwhengameover1, [20, 200])
            screen.blit(Textwhengameover2, [25, 265])

        pygame.display.flip()
        clock.tick(fps)

    
        
    pygame.quit()

    pass
introduction_screen()
  



