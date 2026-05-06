import random
import queue
import math
import pygame
pygame.init()
pygame.mixer.init()

#soundeffect van de politie en vaste tijden (in ms)
politie_sound = pygame.mixer.Sound('C:/Users/firda/.spyder-py3/Politie sound.mp3')
politie_duur = 10500
timer_duur = 60000

screen = pygame.display.set_mode([600,600])
clock = pygame.time.Clock()

#variabelen, constanten en vaste lijsten
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
einde = False
aantal_levels = 8
vakje_size = 60
vakje_aantal = 7
vakje_min = 0
vakje_max = 6
marge = 90
pixels = 600
lengtes = [2,3]
oriëntaties = ['horizontaal','verticaal'] 
kleuren = [(30,150,50),(30,100,200),(250,250,60),(200,100,50),(200,0,100)]
x_posities = [0,1,2,3,4,5,6]
y_posities = [0,1,2,3,4,5,6] 
aantal_auto_makkelijk = 8
aantal_auto_matig = 10
aantal_auto_moeilijk = 12 
aantal_politie_auto = 2
aantal_reversed_auto = 2

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

kans_button = random.random()
if kans_button < 0.25:
    button_coördinaat = (0,0) 
elif 0.25 <= kans_button < 0.5:
    button_coördinaat = (0,6)
elif 0.5 <= kans_button < 0.75:
    button_coördinaat = (6,0)
else:
    button_coördinaat = (6,6)    

#vaste rode auto, en exit en button surface
red_car = Voertuig(0,3,2,'horizontaal',(200,0,0))
exit_rect = pygame.Rect(marge+(7*vakje_size)-(vakje_size/6),marge+(3*vakje_size),vakje_size/6,vakje_size)
button_vakje = pygame.Rect((button_coördinaat[0]*vakje_size)+marge,(button_coördinaat[1]*vakje_size)+marge,vakje_size,vakje_size)

level1_voertuigen = [red_car]
level2_voertuigen = [red_car]
level3_voertuigen = [red_car]
level4_voertuigen = [red_car]
level5_voertuigen = [red_car]
level6_voertuigen = [red_car]
level7_voertuigen = [red_car]
level8_voertuigen = [red_car]

levels = [level1_voertuigen,level2_voertuigen,level3_voertuigen,level4_voertuigen,level5_voertuigen,level6_voertuigen,level7_voertuigen,level8_voertuigen]

def genereer_eigenschappen():
    lengte = None
    oriëntatie = None
    color = None
    x = None
    y = None
    
    kans_lengte = random.random()
    if kans_lengte < 0.5:
        lengte = lengtes[0]
    else:
        lengte = lengtes[1]

    oriëntatie_lengte = random.random()
    if oriëntatie_lengte < 0.5:
        oriëntatie = oriëntaties[0]
    else:
        oriëntatie = oriëntaties[1]

    kans_color = random.random()
    if kans_color < 0.2:
        color = kleuren[0]
    elif 0.2 <= kans_color < 0.4:
        color = kleuren[1]
    elif 0.4 <= kans_color < 0.6:
        color = kleuren[2]
    elif 0.6 <= kans_color < 0.8:
        color = kleuren[3]
    else:
        color = kleuren[4]
        
    kans_x = random.random()
    if kans_x < (1/7):
        x = x_posities[0]
    elif (1/7) <= kans_x < (2/7):
        x = x_posities[1]
    elif (2/7) <= kans_x < (3/7):
        x = x_posities[2]
    elif (3/7) <= kans_x < (4/7):
        x = x_posities[3]
    elif (4/7) <= kans_x < (5/7):
        x = x_posities[4]
    elif (5/7) <= kans_x < (6/7):
        x = x_posities[5]
    else:
        x = x_posities[6]
        
    kans_y = random.random()
    if kans_y < (1/7):
        y = y_posities[0]
    elif (1/7) <= kans_y < (2/7):
        y = y_posities[1]
    elif (2/7) <= kans_y < (3/7):
        y = y_posities[2]
    elif (3/7) <= kans_y < (4/7):
        y = y_posities[3]
    elif (4/7) <= kans_y < (5/7):
        y = y_posities[4]
    elif (5/7) <= kans_y < (6/7):
        y = y_posities[5]
    else:
        y = y_posities[6]
        
    return x,y,lengte,oriëntatie,color

def genereer_auto(level_voertuigen):
    teller = 0
    genereer = True
    while genereer == True:
        x,y,lengte,oriëntatie,color = genereer_eigenschappen()

        auto = Voertuig(x,y,lengte,oriëntatie,color)

        auto_vakjes = []
        if auto.oriëntatie == 'horizontaal':
            auto_vakjes.append((auto.x,auto.y))
            for i in range(auto.lengte - 1):
                auto_vakjes.append((auto.x + 1 + i,auto.y))
        else:
            auto_vakjes.append((auto.x,auto.y))
            for i in range(auto.lengte - 1):
                auto_vakjes.append((auto.x,auto.y + 1 + i))
          
        buiten_grid = False
        for i in range(len(auto_vakjes)):
            if auto_vakjes[i][0] > 6:
                buiten_grid = True
                break
            if auto_vakjes[i][1] > 6:
                buiten_grid = True
                break
                
        bezet = []
        for car in level_voertuigen:
            if car.oriëntatie == 'horizontaal':
                bezet.append((car.x,car.y))
                for i in range(car.lengte - 1):
                    bezet.append((car.x + 1 + i,car.y))
            else:
                bezet.append((car.x,car.y))
                for i in range(car.lengte - 1):
                    bezet.append((car.x,car.y + 1 + i))
         
        is_bezet = False           
        for vak in auto_vakjes:
            if vak in bezet:
                is_bezet = True
                break
                    
        auto_blocked = False
        if auto.oriëntatie == 'horizontaal' and auto.y == 3:
            auto_blocked = True
            
        if is_bezet == False and buiten_grid == False and auto_blocked == False:
            level_voertuigen.append(auto)
            genereer = False
        
        teller += 1 
        if teller == 100:
            print('Too many attempts at generating a level')
            break
  
def genereer_politie_auto(level_voertuigen):
    teller = 0
    genereer = True
    while genereer == True:
        x,y,lengte,oriëntatie,color = genereer_eigenschappen()

        auto = Police_car(x,y,2,oriëntatie,(230,230,230))

        auto_vakjes = []
        if auto.oriëntatie == 'horizontaal':
            auto_vakjes.append((auto.x,auto.y))
            for i in range(auto.lengte - 1):
                auto_vakjes.append((auto.x + 1 + i,auto.y))
        else:
            auto_vakjes.append((auto.x,auto.y))
            for i in range(auto.lengte - 1):
                auto_vakjes.append((auto.x,auto.y + 1 + i))
          
        buiten_grid = False
        for i in range(len(auto_vakjes)):
            if auto_vakjes[i][0] > 6:
                buiten_grid = True
                break
            if auto_vakjes[i][1] > 6:
                buiten_grid = True
                break
                
        bezet = []
        for car in level_voertuigen:
            if car.oriëntatie == 'horizontaal':
                bezet.append((car.x,car.y))
                for i in range(car.lengte - 1):
                    bezet.append((car.x + 1 + i,car.y))
            else:
                bezet.append((car.x,car.y))
                for i in range(car.lengte - 1):
                    bezet.append((car.x,car.y + 1 + i))
         
        is_bezet = False           
        for vak in auto_vakjes:
            if vak in bezet:
                is_bezet = True
                break
                    
        auto_blocked = False
        if auto.oriëntatie == 'horizontaal' and auto.y == 3:
            auto_blocked = True
            
        if is_bezet == False and buiten_grid == False and auto_blocked == False:
            level_voertuigen.append(auto)
            genereer = False
        
        teller += 1 
        if teller == 100:
            print('Too many attempts at generating a level')
            break
        
def genereer_reversed_auto(level_voertuigen):
    teller = 0
    genereer = True
    while genereer == True:
        x,y,lengte,oriëntatie,color = genereer_eigenschappen()

        auto = Reversed_car(x,y,lengte,oriëntatie,color)

        auto_vakjes = []
        if auto.oriëntatie == 'horizontaal':
            auto_vakjes.append((auto.x,auto.y))
            for i in range(auto.lengte - 1):
                auto_vakjes.append((auto.x + 1 + i,auto.y))
        else:
            auto_vakjes.append((auto.x,auto.y))
            for i in range(auto.lengte - 1):
                auto_vakjes.append((auto.x,auto.y + 1 + i))
          
        buiten_grid = False
        for i in range(len(auto_vakjes)):
            if auto_vakjes[i][0] > 6:
                buiten_grid = True
                break
            if auto_vakjes[i][1] > 6:
                buiten_grid = True
                break
                
        bezet = []
        for car in level_voertuigen:
            if car.oriëntatie == 'horizontaal':
                bezet.append((car.x,car.y))
                for i in range(car.lengte - 1):
                    bezet.append((car.x + 1 + i,car.y))
            else:
                bezet.append((car.x,car.y))
                for i in range(car.lengte - 1):
                    bezet.append((car.x,car.y + 1 + i))
         
        is_bezet = False           
        for vak in auto_vakjes:
            if vak in bezet:
                is_bezet = True
                break
                    
        auto_blocked = False
        if auto.oriëntatie == 'horizontaal' and auto.y == 3:
            auto_blocked = True
            
        if is_bezet == False and buiten_grid == False and auto_blocked == False:
            level_voertuigen.append(auto)
            genereer = False
        
        teller += 1 
        if teller == 100:
            print('Too many attempts at generating a level')
            break

def bezette_vakjes_voor_solver(level,state):
    vehicles = levels[level]
    bezet = []
    index2 = 0
    for auto in vehicles:
        x2,y2 = state[index2]
        if auto.oriëntatie == 'horizontaal':
            bezet.append((x2,y2))
            for i in range(auto.lengte - 1):
                bezet.append((x2 + 1 + i,y2))
        else:
            bezet.append((x2,y2))
            for i in range(auto.lengte - 1):
                bezet.append((x2,y2 + 1 + i)) 
        index2 += 1
    return bezet

def solver(level):
    vehicles = levels[level]
    
    q = queue.Queue()
    visited = set()
    
    begin_state = []
    for car in vehicles:
        positie = (car.x,car.y)
        begin_state.append(positie)
        
    begin_state_tuple = tuple(begin_state)
    q.put(begin_state_tuple)

    visited = {begin_state_tuple}
            
    while not q.empty():
        current_state = q.get()
        new_state = list(current_state)
            
        mogelijke_moves = []
        index = 0
        
        if new_state[0][0] + red_car.lengte - 1 == vakje_max:
            return True
        
        bezet = set(bezette_vakjes_voor_solver(level,new_state))
        
        for car in vehicles:
            x,y = new_state[index]
            
            collision = False
            if (x + car.lengte,y) in bezet:
                collision = True
            if car.oriëntatie == 'horizontaal' and (x + car.lengte) <= vakje_max and collision == False:
                mogelijke_moves.append((car,'right'))

            collision = False
            if (x - 1,y) in bezet:
                collision = True
            if car.oriëntatie == 'horizontaal' and (x - 1) >= vakje_min and collision == False:
                mogelijke_moves.append((car,'left'))

            collision = False
            if (x,y + car.lengte) in bezet:
                collision = True
            if car.oriëntatie == 'verticaal' and (y + car.lengte) <= vakje_max and collision == False:
                mogelijke_moves.append((car,'down'))

            collision = False
            if (x,y - 1) in bezet:
                collision = True
            if car.oriëntatie == 'verticaal' and (y - 1) >= vakje_min and collision == False:
                mogelijke_moves.append((car,'up'))
            
            index += 1
            
        for auto,move in mogelijke_moves:
            index = 0
            for car in vehicles:
                if auto != car:
                    index += 1 
                if move == 'right':
                    new_state = list(current_state)
                    x,y = new_state[index]
                    new_state[index] = (x+1,y) 
                    if tuple(new_state) not in visited:
                        visited.add(tuple(new_state))
                        q.put(tuple(new_state))
                elif move == 'left':
                    new_state = list(current_state)
                    x,y = new_state[index]
                    new_state[index] = (x-1,y) 
                    if tuple(new_state) not in visited:
                        visited.add(tuple(new_state))
                        q.put(tuple(new_state))
                elif move == 'down':
                    new_state = list(current_state)
                    x,y = new_state[index]
                    new_state[index] = (x,y+1) 
                    if tuple(new_state) not in visited:
                        visited.add(tuple(new_state))
                        q.put(tuple(new_state))
                else:
                    new_state = list(current_state)
                    x,y = new_state[index]
                    new_state[index] = (x,y-1)  
                    if tuple(new_state) not in visited:
                        visited.add(tuple(new_state))
                        q.put(tuple(new_state))
    return False

oplossing = False
while oplossing == False:
    for i in range(aantal_auto_makkelijk):
        genereer_auto(level1_voertuigen)
    check = solver(0)
    if check == True:
        oplossing = True
        
oplossing = False
while oplossing == False:
    for i in range(aantal_auto_matig):
        genereer_auto(level2_voertuigen)
    check = solver(1)
    if check == True:
        oplossing = True
    
oplossing = False
while oplossing == False:
    for i in range(aantal_auto_moeilijk):
        genereer_auto(level3_voertuigen)
    check = solver(2)
    if check == True:
        oplossing = True

oplossing = False
while oplossing == False:
    for i in range(aantal_auto_matig):
        genereer_auto(level4_voertuigen)
    check = solver(3)
    if check == True:
        oplossing = True
    
oplossing = False
while oplossing == False:
    for i in range(aantal_auto_matig - aantal_politie_auto):
        genereer_auto(level5_voertuigen)
    for i in range(aantal_politie_auto):
        genereer_politie_auto(level5_voertuigen)
    check = solver(4)
    if check == True:
        oplossing = True

oplossing = False
while oplossing == False:
    for i in range(aantal_auto_moeilijk - aantal_reversed_auto):
        genereer_auto(level6_voertuigen)
    for i in range(aantal_reversed_auto):
        genereer_reversed_auto(level6_voertuigen)
    check = solver(5)
    if check == True:
        oplossing = True
    
oplossing = False
while oplossing == False:
    for i in range(aantal_auto_moeilijk):
        genereer_auto(level7_voertuigen)
    check = solver(6)
    if check == True:
        oplossing = True

#current_level 0 betekent level 1
current_level = 0
vehicles = levels[current_level]

#lijst van de beginposities van de auto's
begin_posities = []
for i in range(aantal_levels):
    begin_posities_per_level = []
    for car in levels[i]:
        (car_x_begin,car_y_begin) = (car.x,car.y)
        begin_posities_per_level.append((car_x_begin,car_y_begin))
    begin_posities.append(begin_posities_per_level)

#functie voor de vakjes die door auto's bezet zijn
def bezette_vakjes(lijst_voertuigen,selected_voertuig):
    bezet = []
    for car in lijst_voertuigen:
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
    
#functie om de achtergrond, de parking en de exit te tekenen
def teken_background(exit_block):
    screen.fill((30,30,30))
    pygame.draw.rect(screen,(160,160,160),(marge,marge,vakje_size*vakje_aantal,vakje_size*vakje_aantal))
    if exit_block == False:
        pygame.draw.rect(screen,(0,250,0),exit_rect)
    else:
        pygame.draw.rect(screen,(250,0,0),exit_rect)

#functie om alle teksten voor de levels te schrijven
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
  
def highlight_auto(vehicle):
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

#functie voor de events als je op de left mouse click drukt
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
            
#functie voor level 5 (met de politie)
def politie_check():
    global politie,beweging,politie_begintijd,lose
    if isinstance(selected_car,Police_car) and politie == False:
        politie_sound.stop()
        politie_sound.play()
        politie = True
        beweging = False
        politie_begintijd = pygame.time.get_ticks()
    if politie == True and beweging == True:
        lose = True
        politie_sound.stop()
   
#functie voor level 4 (met beperkte bewegingen)
def limited_moves():
   global moves_level4,lose
   if current_level == 3:
       moves_level4 -= 1
       if moves_level4 == 0 and win == False:
           lose = True

#de 4 functies voor de events als je met de arrows in de 4 richtingen beweegt

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
        politie_check()
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
        limited_moves()

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
        politie_check()
        limited_moves()

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
        politie_check()
        limited_moves()
                
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
        politie_check()
        limited_moves()

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

#functie om bepaalde states te resetten als je wint        
def reset_state_win():
    global selected_car,timer_begin
    selected_car = None
    timer_begin = None

#functie om bepaalde states te resetten na lose of als je een level restart
def reset_state_lose_restart():
    global selected_car,moves_level4,politie,timer_begin,beweging
    selected_car = None
    moves_level4 = 12
    politie = False
    timer_begin = None
    beweging = False
    

running = True
while running:

    clock.tick(20)
    
    #detecteert wanneer de sound effect voor de politie gestopt is
    if politie == True and politie_begintijd != None:
        huidige_tijd_politie = pygame.time.get_ticks()
        if huidige_tijd_politie - politie_begintijd > politie_duur:
            politie = False
            politie_begintijd = None
        
    #detecteert wanneer de tijd van de timer om is 
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
                #wisselt de keys voor de reversed auto
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
            
            #begint het spel
            if started == False:
                if event.key == pygame.K_s:
                    started = True
                    
            #reset de level
            if started == True and lose == False and win == False:
                if event.key == pygame.K_r:
                    reset_state_lose_restart()
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
        highlight_auto(selected_car)
    
    for car in vehicles:
        teken_auto(car)
     
    if button_appear == True:
        teken_button(button_vakje)
    
    if started == False:
        start_screen()
        
    if started == True:
        level_info(current_level,moves_level4,huidige_tijd_timer,timer_begin)
    
    if win == True:
        reset_state_win()
        win_animatie()
    
    if lose == True:
        reset_state_lose_restart()
        lose_animatie()
    
    if einde == True:
        end_screen()

    pygame.display.flip()

pygame.quit()
