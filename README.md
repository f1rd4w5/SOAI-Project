import pygame
pygame.init()

screen = pygame.display.set_mode([600, 600])
clock = pygame.time.Clock()

selected_car = None
win = False
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

red_car = Voertuig(0,3,2,'horizontaal',(200,0,0))
vehicles = [red_car]
exit_rect = pygame.Rect(marge + (7*vakje_size) - (vakje_size/6),marge + (3*vakje_size),vakje_size/6,vakje_size)
car1 = Voertuig(2,3,2,'verticaal',(30,150,50))
vehicles.append(car1)
car2 = Voertuig(1,4,3,'verticaal',(250,250,60))
vehicles.append(car2)

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
                bezet = []
                for car in vehicles:
                    if car != selected_car:
                        if car.oriëntatie == 'horizontaal':
                            bezet.append((car.x,car.y))
                            for i in range(car.lengte - 1):
                                bezet.append((car.x + 1 + i,car.y))
                        else:
                            bezet.append((car.x,car.y))
                            for i in range(car.lengte - 1):
                                bezet.append((car.x,car.y + 1 + i))
                if event.key == pygame.K_RIGHT:
                    collision = False
                    for j in bezet:
                        if j == (selected_car.x + selected_car.lengte,selected_car.y):
                            collision = True
                    if selected_car.oriëntatie == 'horizontaal' and (selected_car.x + selected_car.lengte) <= vakje_max and collision == False:
                        selected_car.x += 1
                        if red_car.x == (vakje_max - red_car.lengte + 1):
                            win = True
                if event.key == pygame.K_LEFT:
                    collision = False
                    for j in bezet:
                        if j == (selected_car.x - 1,selected_car.y):
                            collision = True
                    if selected_car.oriëntatie == 'horizontaal' and (selected_car.x - 1) >= vakje_min and collision == False:
                        selected_car.x -= 1
                if event.key == pygame.K_DOWN:
                    collision = False
                    for j in bezet:
                        if j == (selected_car.x,selected_car.y + selected_car.lengte):
                            collision = True
                    if selected_car.oriëntatie == 'verticaal' and (selected_car.y + selected_car.lengte) <= vakje_max and collision == False:
                        selected_car.y += 1
                if event.key == pygame.K_UP:
                    collision = False
                    for j in bezet:
                        if j == (selected_car.x,selected_car.y - 1):
                            collision = True
                    if selected_car.oriëntatie == 'verticaal' and (selected_car.y - 1) >= vakje_min and collision == False:
                        selected_car.y -= 1
    
    screen.fill((30,30,30))
    pygame.draw.rect(screen,(160,160,160),(marge,marge,vakje_size*vakje_aantal,vakje_size*vakje_aantal))
    pygame.draw.rect(screen,(0,250,0),exit_rect)
    
    for car in vehicles:
        if car.oriëntatie == 'horizontaal':
            pygame.draw.rect(screen,car.color,((car.x*vakje_size)+marge,(car.y*vakje_size)+marge,car.lengte*vakje_size,vakje_size),border_radius=5)
        else:
            pygame.draw.rect(screen,car.color,((car.x*vakje_size)+marge,(car.y*vakje_size)+marge,vakje_size,car.lengte*vakje_size),border_radius=5)
            
    if win == True:
        pygame.draw.rect(screen,(0,250,0),(marge,marge,vakje_size*vakje_aantal,vakje_size*vakje_aantal))
        font_size = 100
        font = pygame.font.Font(None,size = font_size)
        text = font.render("YOU WIN",True,(0,0,0))
        locationX = (pixels/2) - (text.get_width()/2)
        locationY = (pixels/2) - (text.get_height()/2)
        screen.blit(text,dest = (locationX,locationY))

    pygame.display.flip()

pygame.quit()
