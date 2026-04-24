import pygame
pygame.init()
pygame.mixer.init()

politie_sound = pygame.mixer.Sound('C:/Users/firda/.spyder-py3/autosave/Police sirene.wav')
politie_duur = 10500

screen = pygame.display.set_mode([600, 600])
clock = pygame.time.Clock()

#parameters, constanten en vaste lijsten
selected_car = None
win = False
lose = False
politie = False
moves_level4 = 12
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

class Locked_Car(Voertuig):
    def __init__(self,x,y,lengte,oriëntatie,color):
        super().__init__(x,y,lengte,oriëntatie,color)

#vaste rode auto, lijst van voertuigen en exit surface
red_car = Voertuig(0,3,2,'horizontaal',(200,0,0))
politie_car = Police_car(0,5,2,'horizontaal',(30,30,30))
exit_rect = pygame.Rect(marge+(7*vakje_size)-(vakje_size/6),marge+(3*vakje_size),vakje_size/6,vakje_size)

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
level6_voertuigen = [red_car,car11,car12]
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
  
def win_animatie():
    pygame.draw.rect(screen,(0,250,0),(marge,marge,vakje_size*vakje_aantal,vakje_size*vakje_aantal))
    font_size = 100
    font = pygame.font.Font(None,size = font_size)
    text = font.render("YOU WIN",True,(0,0,0))
    locationX = (pixels/2) - (text.get_width()/2)
    locationY = (pixels/2) - (text.get_height()/2)
    screen.blit(text,dest = (locationX,locationY))
    
def lose_animatie():
    pygame.draw.rect(screen,(250,0,0),(marge,marge,vakje_size*vakje_aantal,vakje_size*vakje_aantal))
    font_size = 100
    font = pygame.font.Font(None,size = font_size)
    text = font.render("YOU LOSE",True,(0,0,0))
    locationX = (pixels/2) - (text.get_width()/2)
    locationY = (pixels/2) - (text.get_height()/2)
    screen.blit(text,dest = (locationX,locationY))

running = True
while running:

    clock.tick(20)
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        
        if event.type == pygame.MOUSEBUTTONDOWN:
            if event.button == 1:
                pos = pygame.mouse.get_pos()
                for car in vehicles:
                    if car.oriëntatie == 'horizontaal':
                        if ((car.x*vakje_size)+marge) < pos[0] < (((car.x*vakje_size)+marge)+(car.lengte*vakje_size)) and ((car.y*vakje_size)+marge) < pos[1] < (((car.y*vakje_size)+marge)+vakje_size):
                            selected_car = car
                    else:
                        if ((car.x*vakje_size)+marge) < pos[0] < (((car.x*vakje_size)+marge)+vakje_size) and ((car.y*vakje_size)+marge) < pos[1] < (((car.y*vakje_size)+marge)+(car.lengte*vakje_size)):
                            selected_car = car
            
        if event.type == pygame.KEYDOWN:
            if selected_car:
                bezet = bezette_vakjes(vehicles,selected_car)
                if event.key == pygame.K_RIGHT:
                    collision = False
                    beweging = False
                    for j in bezet:
                        if j == (selected_car.x + selected_car.lengte,selected_car.y):
                            collision = True
                    if selected_car.oriëntatie == 'horizontaal' and (selected_car.x + selected_car.lengte) <= vakje_max and collision == False:
                        selected_car.x += 1
                        beweging = True
                        if isinstance(selected_car,Police_car) and politie == False:
                            politie_sound.play()
                            politie = True
                            beweging = False
                            politie_begintijd = pygame.time.get_ticks()
                        if politie == True:
                            huidige_tijd = pygame.time.get_ticks()
                            if huidige_tijd - politie_begintijd > politie_duur:
                                politie = False
                        if politie == True and beweging == True:
                            lose = True
                            politie_sound.stop()
                        if red_car.x == (vakje_max - red_car.lengte + 1):
                            win = True
                        if current_level == 3:
                            moves_level4 -= 1
                            if moves_level4 == 0 and win == False:
                                lose = True
                if event.key == pygame.K_LEFT:
                    collision = False
                    beweging = False
                    for j in bezet:
                        if j == (selected_car.x - 1,selected_car.y):
                            collision = True
                    if selected_car.oriëntatie == 'horizontaal' and (selected_car.x - 1) >= vakje_min and collision == False:
                        selected_car.x -= 1
                        beweging = True
                        if isinstance(selected_car,Police_car) and politie == False:
                            politie_sound.play()
                            politie = True
                            beweging = False
                            politie_begintijd = pygame.time.get_ticks()
                        if politie == True:
                            huidige_tijd = pygame.time.get_ticks()
                            if huidige_tijd - politie_begintijd > politie_duur:
                                politie = False
                        if politie == True and beweging == True:
                            lose = True
                            politie_sound.stop()
                        if current_level == 3:
                            moves_level4 -= 1
                            if moves_level4 == 0:
                                lose = True
                if event.key == pygame.K_DOWN:
                    collision = False
                    beweging = False
                    for j in bezet:
                        if j == (selected_car.x,selected_car.y + selected_car.lengte):
                            collision = True
                    if selected_car.oriëntatie == 'verticaal' and (selected_car.y + selected_car.lengte) <= vakje_max and collision == False:
                        selected_car.y += 1
                        beweging = True
                        if isinstance(selected_car,Police_car) and politie == False:
                            politie_sound.play()
                            politie = True
                            beweging = False
                            politie_begintijd = pygame.time.get_ticks()
                        if politie == True:
                            huidige_tijd = pygame.time.get_ticks()
                            if huidige_tijd - politie_begintijd > politie_duur:
                                politie = False
                        if politie == True and beweging == True:
                            lose = True
                            politie_sound.stop()
                        if current_level == 3:
                            moves_level4 -= 1
                            if moves_level4 == 0:
                                lose = True
                if event.key == pygame.K_UP:
                    collision = False
                    beweging = False
                    for j in bezet:
                        if j == (selected_car.x,selected_car.y - 1):
                            collision = True
                    if selected_car.oriëntatie == 'verticaal' and (selected_car.y - 1) >= vakje_min and collision == False:
                        selected_car.y -= 1
                        beweging = True
                        if isinstance(selected_car,Police_car) and politie == False:
                            politie_sound.play()
                            politie = True
                            beweging = False
                            politie_begintijd = pygame.time.get_ticks()
                        if politie == True:
                            huidige_tijd = pygame.time.get_ticks()
                            if huidige_tijd - politie_begintijd > politie_duur:
                                politie = False
                        if politie == True and beweging == True:
                            lose = True
                            politie_sound.stop()
                        if current_level == 3:
                            moves_level4 -= 1
                            if moves_level4 == 0:
                                lose = True
            
            if win == True:
                if event.key == pygame.K_SPACE:
                    win = False
                    if current_level == (aantal_levels - 1):
                        einde = True
                    else:
                        current_level += 1
                        vehicles = levels[current_level]
                        red_car.x,red_car.y = 0,3 
                        
            if lose == True:
                if event.key == pygame.K_SPACE:
                    lose = False
                    index = 0
                    for car in levels[current_level]:
                        (car.x,car.y) = begin_posities[current_level][index]
                        index += 1
    
    #donker grijze achtergrond, lichtgrijze parking en licht groene exit
    screen.fill((30,30,30))
    pygame.draw.rect(screen,(160,160,160),(marge,marge,vakje_size*vakje_aantal,vakje_size*vakje_aantal))
    pygame.draw.rect(screen,(0,250,0),exit_rect)
    
    #selected_car highlighten
    if selected_car:
        if selected_car.oriëntatie == 'horizontaal':
            pygame.draw.rect(screen,(150,250,50),((selected_car.x*vakje_size)+marge-(vakje_size/12),(selected_car.y*vakje_size)+marge-(vakje_size/12),(selected_car.lengte*vakje_size)+(vakje_size/6),7*vakje_size/6),border_radius=5)
        else:
            pygame.draw.rect(screen,(150,250,50),((selected_car.x*vakje_size)+marge-(vakje_size/12),(selected_car.y*vakje_size)+marge-(vakje_size/12),7*vakje_size/6,(selected_car.lengte*vakje_size)+(vakje_size/6)),border_radius=5)
    
    for car in vehicles:
        teken_auto(car)
            
    if win == True:
        selected_car = None
        win_animatie()
    
    if lose == True:
        selected_car = None
        politie = False
        beweging = False
        lose_animatie()
    
    if einde == True:
        pygame.draw.rect(screen,(0,0,0),(marge,marge,vakje_size*vakje_aantal,vakje_size*vakje_aantal))

    pygame.display.flip()

pygame.quit()
