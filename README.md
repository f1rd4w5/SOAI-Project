import math
import pygame
pygame.init()
pygame.mixer.init()

politie_sound = pygame.mixer.Sound('C:/Users/firda/.spyder-py3/Politie sound.mp3')
politie_duur = 10500

timer_duur = 60000

screen = pygame.display.set_mode([600,600])
clock = pygame.time.Clock()

#parameters, constanten en vaste lijsten
started = False
selected_car = None
win = False
lose = False
beweging = False
politie = False
politie_begintijd = None
huidige_tijd_politie = None
huidige_tijd_timer = None
timer_begin = None
moves_level4 = 12
exit_blocked = False
button_appear = False
button_coördinaat = (0,0)
einde = False
aantal_levels = 8
vakje_size = 60
vakje_aantal = 7
vakje_min = 0
vakje_max = 6
marge = 90
pixels = 600
kleuren = [(30,150,50),(30,100,200),(250,250,60),(200,100,50),(200,0,100)]
lengtes = [2,3]
oriëntaties = ['horizontaal','verticaal']   

class Voertuig:
    def __init__(self,x,y,lengte,oriëntatie,color):
        self.x = x
        self.y = y
        self.lengte = lengte
        self.oriëntatie = oriëntatie
        self.color = color

class Police_car(Voertuig):
    def __init__(self,x,y,lengte,oriëntatie,color):
        super().__init__(x,y,lengte,oriëntatie,color)

class Reversed_car(Voertuig):
    def __init__(self,x,y,lengte,oriëntatie,color):
        super().__init__(x,y,lengte,oriëntatie,color)

#vaste rode auto, lijst van voertuigen, en exit en button surface
red_car = Voertuig(0,3,2,'horizontaal',(200,0,0))
politie_car = Police_car(0,5,2,'verticaal',(230,230,230))
reversed_auto = Reversed_car(5,5,2,'verticaal',(30,100,200))
exit_rect = pygame.Rect(marge+(7*vakje_size)-(vakje_size/6),marge+(3*vakje_size),vakje_size/6,vakje_size)
button_vakje = pygame.Rect((button_coördinaat[0]*vakje_size)+marge,(button_coördinaat[1]*vakje_size)+marge,vakje_size,vakje_size)

#zelfgemaakte voertuigen om spel te testen
car1 = Voertuig(2,3,2,'verticaal',(30,150,50))
car2 = Voertuig(1,4,3,'verticaal',(250,250,60))
car3 = Voertuig(3,5,3,'horizontaal',(200,0,100))
car4 = Voertuig(1,4,2,'verticaal',(200,100,50))
car5 = Voertuig(3,1,2,'horizontaal',(30,100,200))
car6 = Voertuig(1,4,3,'horizontaal',(30,150,50))
car7 = Voertuig(1,4,3,'horizontaal',(30,150,50))
car8 = Voertuig(4,0,3,'verticaal',(30,100,200))
car9 = Voertuig(0,0,2,'verticaal',(200,0,100))
car10 = Voertuig(4,0,3,'verticaal',(30,100,200))
car11 = Voertuig(4,1,2,'horizontaal',(30,150,50))
car12 = Voertuig(0,5,3,'horizontaal',(250,250,60))
car13 = Voertuig(0,5,3,'horizontaal',(200,100,50))
car14 = Voertuig(2,0,2,'verticaal',(30,150,50))

#zelfgemaakte lijsten van voertuigen voor de levels om spel te testen
level1_voertuigen = [red_car,car1,car2]
level2_voertuigen = [red_car,car3,car4]
level3_voertuigen = [red_car,car5,car6]
level4_voertuigen = [red_car,car7,car8]
level5_voertuigen = [red_car,politie_car,car9,car10]
level6_voertuigen = [red_car,reversed_auto,car11,car12]
level7_voertuigen = [red_car,car13,car14]
level8_voertuigen = [red_car]

levels = [level1_voertuigen,level2_voertuigen,level3_voertuigen,level4_voertuigen,level5_voertuigen,level6_voertuigen,level7_voertuigen,level8_voertuigen]

current_level = 0
vehicles = levels[current_level]

begin_posities = []
for i in range(aantal_levels):
    begin_posities_per_level = []
    for car in levels[i]:
        (car_x_begin,car_y_begin) = (car.x,car.y)
        begin_posities_per_level.append((car_x_begin,car_y_begin))
    begin_posities.append(begin_posities_per_level)

def bezette_vakjes(lijst_voertuigen,selected_voertuig):
    bezet = []
    for car in lijst_voertuigen:
        if car != selected_voertuig:
            if car.oriëntatie == 'horizontaal':
                bezet.append((car.x,car.y))
                for i in range(car.lengte - 1):
                    bezet.append((car.x + 1 + i,car.y))
            else:
                bezet.append((car.x,car.y))
                for i in range(car.lengte - 1):
                    bezet.append((car.x,car.y + 1 + i))
    return bezet

def start_screen():
    screen.fill((30,30,30))
    font1 = pygame.font.Font(None,size = 100)
    font2 = pygame.font.Font(None,size = 30)
    text1 = font1.render("REBELRIDE",True,(250,250,250))
    text2 = font2.render("In this game, you must get the red car out of the",True,(250,250,250))
    text3 = font2.render("parking lot by moving the present vehicles around.",True,(250,250,250))
    text4 = font2.render("Controls:",True,(250,250,250))
    text5 = font2.render("left mouse click: select vehicle",True,(250,250,250))
    text6 = font2.render("arrows: move vehicles",True,(250,250,250))
    text7 = font2.render("Enjoy and keep your sound on, you know, just in case...",True,(250,250,250))
    text8 = font2.render("Press S to start the game",True,(250,250,250))
    locationX1 = (pixels/2) - (text1.get_width()/2)
    locationY1 = (pixels/4) - (text1.get_height()/2)
    locationX2 = (pixels/2) - (text2.get_width()/2)
    locationY2 = (pixels/2) - (2*text2.get_height())
    locationX3 = (pixels/2) - (text3.get_width()/2)
    locationY3 = (pixels/2) - (text2.get_height())
    locationX4 = (pixels/2) - (text4.get_width()/2)
    locationY4 = (3*pixels/5) - (text4.get_height()/2)
    locationX5 = (pixels/2) - (text5.get_width()/2)
    locationY5 = (3*pixels/5) + (text4.get_height())
    locationX6 = (pixels/2) - (text6.get_width()/2)
    locationY6 = (3*pixels/5) + (2*text4.get_height())
    locationX7 = (pixels/2) - (text7.get_width()/2)
    locationY7 = (3*pixels/4) + (text7.get_height()/2)
    locationX8 = (pixels/2) - (text8.get_width()/2)
    locationY8 = (7*pixels/8) - (text8.get_height()/2)
    screen.blit(text1,dest = (locationX1,locationY1))
    screen.blit(text2,dest = (locationX2,locationY2))
    screen.blit(text3,dest = (locationX3,locationY3))
    screen.blit(text4,dest = (locationX4,locationY4))
    screen.blit(text5,dest = (locationX5,locationY5))
    screen.blit(text6,dest = (locationX6,locationY6))
    screen.blit(text7,dest = (locationX7,locationY7))
    screen.blit(text8,dest = (locationX8,locationY8))
    
def teken_background(exit_block):
    screen.fill((30,30,30))
    pygame.draw.rect(screen,(160,160,160),(marge,marge,vakje_size*vakje_aantal,vakje_size*vakje_aantal))
    if exit_block == False:
        pygame.draw.rect(screen,(0,250,0),exit_rect)
    else:
        pygame.draw.rect(screen,(250,0,0),exit_rect)

def level_info(level,moves,huidige,begin):
    font1 = pygame.font.Font(None,size = 35)
    font2 = pygame.font.Font(None,size = 25)
    font3 = pygame.font.Font(None,size = 20)
    font4 = pygame.font.Font(None,size = 25)
    text1 = font1.render(f"Level {level + 1}/8",True,(250,250,250))
    locationX1 = (pixels/2) - (text1.get_width()/2)
    locationY1 = (pixels/14) - (text1.get_height()/2)
    screen.blit(text1,dest = (locationX1,locationY1))
    if lose == False and win == False:
        text4 = font4.render("Press R to restart level",True,(250,250,250))
        locationX4 = (pixels/2) - (text4.get_width()/2)
        locationY4 = (14*pixels/15)
        screen.blit(text4,dest = (locationX4,locationY4))
    if level == 0:
        text2 = font2.render("Welcome to the game!",True,(250,250,250))
    elif level == 1:
        text2 = font2.render("Okay, your skills aren't too bad...",True,(250,250,250))
    elif level == 2:
        text2 = font2.render("I see you're getting the hang of it.",True,(250,250,250))
    elif level == 3:
        text2 = font2.render("Diesel's getting expensive... don't move too much.",True,(250,250,250))
        if lose == False and win == False:
            text5 = font1.render(f"Moves left: {moves}",True,(250,250,250))
            locationX5 = (pixels/2) - (text5.get_width()/2)
            locationY5 = (7*pixels/8)
            screen.blit(text5,dest = (locationX5,locationY5))
    elif level == 4:
        text2 = font2.render("Don't move when the police is around!",True,(250,250,250))
    elif level == 5:
        text2 = font2.render("Everything seems pretty normal, right?",True,(250,250,250))
    elif level == 6:
        text2 = font2.render("Hurry up! The gate's going to close!",True,(250,250,250))
        if lose == False and win == False and huidige != None and begin != None:
            text6 = font1.render(f"Time left: {60 - ((huidige - begin)//1000)} seconds",True,(250,250,250))
            locationX6 = (pixels/2) - (text6.get_width()/2)
            locationY6 = (7*pixels/8)
            screen.blit(text6,dest = (locationX6,locationY6))
    elif level == 7:
        text2 = font2.render("You escaped on time! Or, did you?",True,(250,250,250))
        text3 = font3.render("Hint: find the red button to open the gate.",True,(250,250,250))
        locationX3 = (pixels/2) - (text3.get_width()/2)
        locationY3 = (pixels/12) + (5*text2.get_height()/4)
        screen.blit(text3,dest = (locationX3,locationY3))
    locationX2 = (pixels/2) - (text2.get_width()/2)
    locationY2 = (pixels/14) + (text1.get_height()/2)
    screen.blit(text2,dest = (locationX2,locationY2))

def teken_auto(vehicle):
    if vehicle.oriëntatie == 'horizontaal':
        pygame.draw.rect(screen,vehicle.color,((vehicle.x*vakje_size)+marge,(vehicle.y*vakje_size)+marge,vehicle.lengte*vakje_size,vakje_size),border_radius=5)
        pygame.draw.rect(screen,(130,220,250),(((vehicle.x+vehicle.lengte)*vakje_size)+marge-(5*vakje_size/12),(vehicle.y*vakje_size)+marge+((vakje_size-(5*vakje_size/6))/2),vakje_size/3,5*vakje_size/6),border_radius=3)
        if vehicle.lengte == 2:
            pygame.draw.rect(screen,(64,64,64),((vehicle.x*vakje_size)+marge+(vakje_size/12),(vehicle.y*vakje_size)+marge-(vakje_size/24),vakje_size/2,vakje_size/8),border_radius=8)
            pygame.draw.rect(screen,(64,64,64),((vehicle.x*vakje_size)+marge+(10*vakje_size/12),(vehicle.y*vakje_size)+marge-(vakje_size/24),vakje_size/2,vakje_size/8),border_radius=8)
            pygame.draw.rect(screen,(64,64,64),((vehicle.x*vakje_size)+marge+(vakje_size/12),(vehicle.y*vakje_size)+marge+(22*vakje_size/24),vakje_size/2,vakje_size/8),border_radius=8)
            pygame.draw.rect(screen,(64,64,64),((vehicle.x*vakje_size)+marge+(10*vakje_size/12),(vehicle.y*vakje_size)+marge+(22*vakje_size/24),vakje_size/2,vakje_size/8),border_radius=8)
        else:
            pygame.draw.rect(screen,(64,64,64),((vehicle.x*vakje_size)+marge+(vakje_size/2),(vehicle.y*vakje_size)+marge-(vakje_size/24),vakje_size/2,vakje_size/8),border_radius=8)
            pygame.draw.rect(screen,(64,64,64),((vehicle.x*vakje_size)+marge+(22*vakje_size/12),(vehicle.y*vakje_size)+marge-(vakje_size/24),vakje_size/2,vakje_size/8),border_radius=8)
            pygame.draw.rect(screen,(64,64,64),((vehicle.x*vakje_size)+marge+(vakje_size/2),(vehicle.y*vakje_size)+marge+(22*vakje_size/24),vakje_size/2,vakje_size/8),border_radius=8)
            pygame.draw.rect(screen,(64,64,64),((vehicle.x*vakje_size)+marge+(22*vakje_size/12),(vehicle.y*vakje_size)+marge+(22*vakje_size/24),vakje_size/2,vakje_size/8),border_radius=8)
        if isinstance(vehicle,Police_car):
            pygame.draw.rect(screen,(200,0,0),((vehicle.x*vakje_size)+marge+vakje_size,(vehicle.y*vakje_size)+marge+(vakje_size/4),vakje_size/6,vakje_size/4))
            pygame.draw.rect(screen,(30,100,200),((vehicle.x*vakje_size)+marge+vakje_size,(vehicle.y*vakje_size)+marge+(vakje_size/2),vakje_size/6,vakje_size/4))
    else:
        pygame.draw.rect(screen,vehicle.color,((vehicle.x*vakje_size)+marge,(vehicle.y*vakje_size)+marge,vakje_size,vehicle.lengte*vakje_size),border_radius=5)
        pygame.draw.rect(screen,(130,220,250),((vehicle.x*vakje_size)+marge+((vakje_size-(5*vakje_size/6))/2),(vehicle.y*vakje_size)+marge+(vakje_size/12),5*vakje_size/6,vakje_size/3),border_radius=3)
        if vehicle.lengte == 2:
            pygame.draw.rect(screen,(64,64,64),((vehicle.x*vakje_size)+marge-(vakje_size/24),(vehicle.y*vakje_size)+marge+(8*vakje_size/12),vakje_size/8,vakje_size/2),border_radius=8)
            pygame.draw.rect(screen,(64,64,64),((vehicle.x*vakje_size)+marge-(vakje_size/24),(vehicle.y*vakje_size)+marge+(17*vakje_size/12),vakje_size/8,vakje_size/2),border_radius=8)
            pygame.draw.rect(screen,(64,64,64),((vehicle.x*vakje_size)+marge+(22*vakje_size/24),(vehicle.y*vakje_size)+marge+(8*vakje_size/12),vakje_size/8,vakje_size/2),border_radius=8)
            pygame.draw.rect(screen,(64,64,64),((vehicle.x*vakje_size)+marge+(22*vakje_size/24),(vehicle.y*vakje_size)+marge+(17*vakje_size/12),vakje_size/8,vakje_size/2),border_radius=8)
        else:
            pygame.draw.rect(screen,(64,64,64),((vehicle.x*vakje_size)+marge-(vakje_size/24),(vehicle.y*vakje_size)+marge+(8*vakje_size/12),vakje_size/8,vakje_size/2),border_radius=8)
            pygame.draw.rect(screen,(64,64,64),((vehicle.x*vakje_size)+marge-(vakje_size/24),(vehicle.y*vakje_size)+marge+(2*vakje_size),vakje_size/8,vakje_size/2),border_radius=8)
            pygame.draw.rect(screen,(64,64,64),((vehicle.x*vakje_size)+marge+(22*vakje_size/24),(vehicle.y*vakje_size)+marge+(8*vakje_size/12),vakje_size/8,vakje_size/2),border_radius=8)
            pygame.draw.rect(screen,(64,64,64),((vehicle.x*vakje_size)+marge+(22*vakje_size/24),(vehicle.y*vakje_size)+marge+(2*vakje_size),vakje_size/8,vakje_size/2),border_radius=8)
        if isinstance(vehicle,Police_car):
            pygame.draw.rect(screen,(200,0,0),((vehicle.x*vakje_size)+marge+(vakje_size/4),(vehicle.y*vakje_size)+marge+(4*vakje_size/6),vakje_size/4,vakje_size/6))
            pygame.draw.rect(screen,(30,100,200),((vehicle.x*vakje_size)+marge+(vakje_size/2),(vehicle.y*vakje_size)+marge+(4*vakje_size/6),vakje_size/4,vakje_size/6))
  
def highlight_car(vehicle):
    if vehicle.oriëntatie == 'horizontaal':
        if isinstance(vehicle,Reversed_car):
            pygame.draw.rect(screen,(250,100,250),((vehicle.x*vakje_size)+marge-(vakje_size/12),(vehicle.y*vakje_size)+marge-(vakje_size/12),(vehicle.lengte*vakje_size)+(vakje_size/6),7*vakje_size/6),border_radius=5)
        else:
            pygame.draw.rect(screen,(150,250,50),((vehicle.x*vakje_size)+marge-(vakje_size/12),(vehicle.y*vakje_size)+marge-(vakje_size/12),(vehicle.lengte*vakje_size)+(vakje_size/6),7*vakje_size/6),border_radius=5)
    else:
        if isinstance(vehicle,Reversed_car):
            pygame.draw.rect(screen,(250,100,250),((vehicle.x*vakje_size)+marge-(vakje_size/12),(vehicle.y*vakje_size)+marge-(vakje_size/12),7*vakje_size/6,(vehicle.lengte*vakje_size)+(vakje_size/6)),border_radius=5)
        else:
            pygame.draw.rect(screen,(150,250,50),((vehicle.x*vakje_size)+marge-(vakje_size/12),(vehicle.y*vakje_size)+marge-(vakje_size/12),7*vakje_size/6,(vehicle.lengte*vakje_size)+(vakje_size/6)),border_radius=5)

def teken_button(button_vak):
    pygame.draw.rect(screen,(0,0,0),button_vak)
    pygame.draw.circle(screen,(250,0,0),(button_vak.x + (vakje_size/2),button_vak.y + (vakje_size/2)),vakje_size/6)

def win_animatie():
    pygame.draw.rect(screen,(0,250,0),(marge,marge,vakje_size*vakje_aantal,vakje_size*vakje_aantal))
    font1 = pygame.font.Font(None,size = 100)
    font2 = pygame.font.Font(None,size = 30)
    text1 = font1.render("YOU WIN",True,(0,0,0))
    locationX1 = (pixels/2) - (text1.get_width()/2)
    locationY1 = (pixels/2) - (text1.get_height()/2)
    screen.blit(text1,dest = (locationX1,locationY1))
    if  current_level == 7:
        text2 = font2.render("Press SPACE to go to the credits",True,(0,0,0))
    else:
        text2 = font2.render("Press SPACE to go to the next level",True,(0,0,0))
    locationX2 = (pixels/2) - (text2.get_width()/2)
    locationY2 = (3*pixels/5) - (text2.get_height()/2)
    screen.blit(text2,dest = (locationX2,locationY2))
    
def lose_animatie():
    pygame.draw.rect(screen,(250,0,0),(marge,marge,vakje_size*vakje_aantal,vakje_size*vakje_aantal))
    font1 = pygame.font.Font(None,size = 100)
    font2 = pygame.font.Font(None,size = 30)
    text1 = font1.render("YOU LOSE",True,(0,0,0))
    locationX1 = (pixels/2) - (text1.get_width()/2)
    locationY1 = (pixels/2) - (text1.get_height()/2)
    screen.blit(text1,dest = (locationX1,locationY1))
    text2 = font2.render("Press SPACE to restart the level",True,(0,0,0))
    locationX2 = (pixels/2) - (text2.get_width()/2)
    locationY2 = (3*pixels/5) - (text2.get_height()/2)
    screen.blit(text2,dest = (locationX2,locationY2))

def end_screen():
    screen.fill((30,30,30))
    font1 = pygame.font.Font(None,size = 70)
    font2 = pygame.font.Font(None,size = 35)
    text1 = font1.render("Thank you for playing!",True,(250,250,250))
    text2 = font2.render("A game by:",True,(250,250,250))
    text3 = font2.render("Aharchi Firdaws",True,(250,250,250))
    text4 = font2.render("Barouch Nisrine",True,(250,250,250))
    text5 = font2.render("El Ouassyf Soraya",True,(250,250,250))
    locationX1 = (pixels/2) - (text1.get_width()/2)
    locationY1 = (pixels/4) - (text1.get_height()/2)
    locationX2 = (pixels/2) - (text2.get_width()/2)
    locationY2 = (pixels/2) - (2*text2.get_height())
    locationX3 = (pixels/2) - (text3.get_width()/2)
    locationY3 = (pixels/2) - (text2.get_height()/2)
    locationX4 = (pixels/2) - (text4.get_width()/2)
    locationY4 = (pixels/2) + (text2.get_height()/2)
    locationX5 = (pixels/2) - (text5.get_width()/2)
    locationY5 = (pixels/2) + (3*text2.get_height()/2)
    screen.blit(text1,dest = (locationX1,locationY1))
    screen.blit(text2,dest = (locationX2,locationY2))
    screen.blit(text3,dest = (locationX3,locationY3))
    screen.blit(text4,dest = (locationX4,locationY4))
    screen.blit(text5,dest = (locationX5,locationY5))

def left_mouse_click():
    global selected_car,button_appear,exit_blocked
    pos = pygame.mouse.get_pos()
    for car in vehicles:
        if car.oriëntatie == 'horizontaal':
            if ((car.x*vakje_size)+marge) < pos[0] < (((car.x*vakje_size)+marge)+(car.lengte*vakje_size)) and ((car.y*vakje_size)+marge) < pos[1] < (((car.y*vakje_size)+marge)+vakje_size):
                selected_car = car
        else:
            if ((car.x*vakje_size)+marge) < pos[0] < (((car.x*vakje_size)+marge)+vakje_size) and ((car.y*vakje_size)+marge) < pos[1] < (((car.y*vakje_size)+marge)+(car.lengte*vakje_size)):
                selected_car = car
    if button_vakje.x < pos[0] < (button_vakje.x + vakje_size) and button_vakje.y < pos[1] < (button_vakje.y + vakje_size):
        if button_appear == False and exit_blocked == True:
            button_appear = True
        elif math.sqrt(((pos[0] - (button_vakje.x + (vakje_size/2)))**2) + ((pos[1] - (button_vakje.y + (vakje_size/2)))**2)) < (vakje_size/6):
            exit_blocked = False
            
def beweging_right():
    global selected_car,beweging,politie,politie_begintijd,lose,win,exit_blocked,moves_level4
    bezet = bezette_vakjes(vehicles,selected_car)
    collision = False
    for j in bezet:
        if j == (selected_car.x + selected_car.lengte,selected_car.y):
            collision = True
    if selected_car.oriëntatie == 'horizontaal' and (selected_car.x + selected_car.lengte) <= vakje_max and collision == False:
        selected_car.x += 1
        beweging = True
        if isinstance(selected_car,Police_car) and politie == False:
            politie_sound.stop()
            politie_sound.play()
            politie = True
            beweging = False
            politie_begintijd = pygame.time.get_ticks()
        if politie == True and beweging == True:
            lose = True
            politie_sound.stop()
        if current_level == 7:
            if button_appear == False and red_car.x == (vakje_max - red_car.lengte):
                exit_blocked = True
            if exit_blocked == True:
                win = False
            if button_appear == True and exit_blocked == False:
                if red_car.x == (vakje_max - red_car.lengte + 1):
                    win = True
        else:
            if red_car.x == (vakje_max - red_car.lengte + 1):
                win = True
        if current_level == 3:
            moves_level4 -= 1
            if moves_level4 == 0 and win == False:
                lose = True

def beweging_left():
    global selected_car,beweging,politie,politie_begintijd,lose,moves_level4
    bezet = bezette_vakjes(vehicles,selected_car)
    collision = False
    for j in bezet:
        if j == (selected_car.x - 1,selected_car.y):
            collision = True
    if selected_car.oriëntatie == 'horizontaal' and (selected_car.x - 1) >= vakje_min and collision == False:
        selected_car.x -= 1
        beweging = True
        if isinstance(selected_car,Police_car) and politie == False:
            politie_sound.stop()
            politie_sound.play()
            politie = True
            beweging = False
            politie_begintijd = pygame.time.get_ticks()
        if politie == True and beweging == True:
            lose = True
            politie_sound.stop()
        if current_level == 3:
            moves_level4 -= 1
            if moves_level4 == 0:
                lose = True

def beweging_down():
    global selected_car,beweging,politie,politie_begintijd,lose,moves_level4
    bezet = bezette_vakjes(vehicles,selected_car)
    collision = False
    for j in bezet:
        if j == (selected_car.x,selected_car.y + selected_car.lengte):
            collision = True
    if selected_car.oriëntatie == 'verticaal' and (selected_car.y + selected_car.lengte) <= vakje_max and collision == False:
        selected_car.y += 1
        beweging = True
        if isinstance(selected_car,Police_car) and politie == False:
            politie_sound.stop()
            politie_sound.play()
            politie = True
            beweging = False
            politie_begintijd = pygame.time.get_ticks()
        if politie == True and beweging == True:
            lose = True
            politie_sound.stop()
        if current_level == 3:
            moves_level4 -= 1
            if moves_level4 == 0:
                lose = True
                
def beweging_up():
    global selected_car,beweging,politie,politie_begintijd,lose,moves_level4
    bezet = bezette_vakjes(vehicles,selected_car)
    collision = False
    for j in bezet:
        if j == (selected_car.x,selected_car.y - 1):
            collision = True
    if selected_car.oriëntatie == 'verticaal' and (selected_car.y - 1) >= vakje_min and collision == False:
        selected_car.y -= 1
        beweging = True
        if isinstance(selected_car,Police_car) and politie == False:
            politie_sound.stop()
            politie_sound.play()
            politie = True
            beweging = False
            politie_begintijd = pygame.time.get_ticks()
        if politie == True and beweging == True:
            lose = True
            politie_sound.stop()
        if current_level == 3:
            moves_level4 -= 1
            if moves_level4 == 0:
                lose = True

def restart_level():
    global timer_begin
    index = 0
    for car in levels[current_level]:
        (car.x,car.y) = begin_posities[current_level][index]
        index += 1
    if current_level == 6:
        timer_begin = pygame.time.get_ticks()
        
def next_level():
    global einde,current_level,vehicles,timer_begin
    if current_level == (aantal_levels - 1):
        einde = True
    else:
        current_level += 1
        vehicles = levels[current_level]
        red_car.x,red_car.y = 0,3 
        if current_level == 6:
            timer_begin = pygame.time.get_ticks()

running = True
while running:

    clock.tick(20)
    
    if politie == True and politie_begintijd != None:
        huidige_tijd_politie = pygame.time.get_ticks()
        if huidige_tijd_politie - politie_begintijd > politie_duur:
            politie = False
            politie_begintijd = None
            
    if timer_begin:
        huidige_tijd_timer = pygame.time.get_ticks()
        if huidige_tijd_timer - timer_begin > timer_duur:
            lose = True
    
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        
        if event.type == pygame.MOUSEBUTTONDOWN:
            if event.button == 1:
                left_mouse_click()
            
        if event.type == pygame.KEYDOWN:
            if selected_car:
                key = event.key
                if isinstance(selected_car,Reversed_car):
                    if key == pygame.K_RIGHT:
                        key = pygame.K_LEFT
                    elif key == pygame.K_LEFT:
                        key = pygame.K_RIGHT
                    elif key == pygame.K_DOWN:
                        key = pygame.K_UP
                    elif key == pygame.K_UP:
                        key = pygame.K_DOWN
                if key == pygame.K_RIGHT:
                    beweging_right()
                elif key == pygame.K_LEFT:
                    beweging_left()
                elif key == pygame.K_DOWN:
                    beweging_down()
                elif key == pygame.K_UP:
                    beweging_up()
            
            if started == False:
                if event.key == pygame.K_s:
                    started = True
                    
            if started == True and lose == False and win == False:
                if event.key == pygame.K_r:
                    selected_car = None
                    politie = False
                    timer_begin = None
                    beweging = False
                    restart_level()

            if win == True:
                if event.key == pygame.K_SPACE:
                    win = False
                    next_level()
                        
            if lose == True:
                if event.key == pygame.K_SPACE:
                    lose = False
                    restart_level()
    
    teken_background(exit_blocked)
    
    if selected_car:
        highlight_car(selected_car)
    
    for car in vehicles:
        teken_auto(car)
     
    if button_appear == True:
        teken_button(button_vakje)
    
    if started == False:
        start_screen()
        
    if started == True:
        level_info(current_level,moves_level4,huidige_tijd_timer,timer_begin)
    
    if win == True:
        selected_car = None
        timer_begin = None
        win_animatie()
    
    if lose == True:
        selected_car = None
        moves_level4 = 12
        politie = False
        timer_begin = None
        beweging = False
        lose_animatie()
    
    if einde == True:
        end_screen()

    pygame.display.flip()

pygame.quit()
