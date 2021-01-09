import pygame
import pygame_menu
import math
import random
import time

pygame.init()
winHeight = 480
winWidth = 700
win=pygame.display.set_mode((winWidth,winHeight))
pygame.display.set_caption("BLAST!")

BLACK = (0,0, 0)
WHITE = (255,255,255)


button_font = pygame.font.SysFont("comicsans", 30)
guess_font = pygame.font.SysFont("comicsans", 50)
lost_font = pygame.font.SysFont('comicsans', 45)
def_font = pygame.font.SysFont('comicsans', 25)
word = ''
buttons = []
guessed = []
dynamite = [pygame.image.load('dynamite0.jpg'), pygame.image.load('dynamite1.jpg'), pygame.image.load('dynamite2.jpg'), pygame.image.load('dynamite3.jpg'), pygame.image.load('dynamite4.jpg'), pygame.image.load('dynamite5.jpg'), pygame.image.load('dynamite6.jpg')]

limbs = 0

def redraw_game_window():
    global guessed
    global dynamite
    global limbs
    win.fill(WHITE)
    # Buttons
    for i in range(len(buttons)):
        if buttons[i][4]:
            pygame.draw.circle(win, BLACK, (buttons[i][1], buttons[i][2]), buttons[i][3])
            pygame.draw.circle(win, buttons[i][0], (buttons[i][1], buttons[i][2]), buttons[i][3] - 3
                               )
            label = button_font.render(chr(buttons[i][5]), 2, BLACK)
            win.blit(label, (buttons[i][1] - (label.get_width() / 2), buttons[i][2] - (label.get_height() / 2)))

    spaced = spacedOut(word, guessed)
    label1 = guess_font.render(spaced, 1, BLACK)
    rect = label1.get_rect()
    length = rect[2]
    
    win.blit(label1,(winWidth/2 - length/2, 400))

    pic = dynamite[limbs]
    win.blit(pic, (winWidth/2 - pic.get_width()/2 + 20, 150))
    pygame.display.update()



def randomWord():
    file = open('words.txt')
    f = file.readlines()
    i = random.randrange(0, len(f) - 1)

    return f[i][:-1]

def definition():
    file = open('def.txt')
    f = file.readlines()
    i = random.randrange(0, len(f) - 1)

    return f[i][:-1]

def boom(guess):
    global word
    if guess.lower() not in word.lower():
        return True
    else:
        return False


def spacedOut(word, guessed=[]):
    spacedWord = ''
    guessedLetters = guessed
    for x in range(len(word)):
        if word[x] != ' ':
            spacedWord += '_ '
            for i in range(len(guessedLetters)):
                if word[x].upper() == guessedLetters[i]:
                    spacedWord = spacedWord[:-2]
                    spacedWord += word[x].upper() + ' '
        elif word[x] == ' ':
            spacedWord += ' '
    return spacedWord
            

def buttonHit(x, y):
    for i in range(len(buttons)):
        if x < buttons[i][1] + 20 and x > buttons[i][1] - 20:
            if y < buttons[i][2] + 20 and y > buttons[i][2] - 20:
                return buttons[i][5]
    return None


def end(winner=False):
    global limbs
    lostTxt = 'Game Over :( Press any key to play again...'
    winTxt = 'Great Job! Press any key to play again...'
    redraw_game_window()
    pygame.time.delay(1000)
    win.fill(WHITE)

    if winner == True:
        label = lost_font.render(winTxt, 1, BLACK)

    else:
        label = lost_font.render(lostTxt, 1, BLACK)

    wordTxt = lost_font.render(word.upper(), 1, BLACK)
    wordWas = lost_font.render('The word was: ', 1, BLACK)
    defIs = def_font.render(definition(), 1, BLACK)

    win.blit(wordTxt, (winWidth/2 - wordTxt.get_width()/2, 250))
    win.blit(wordWas, (winWidth/2 - wordWas.get_width()/2, 200))
    win.blit(defIs, (winWidth / 2 - defIs.get_width() / 2, 300))
    win.blit(label, (winWidth / 2 - label.get_width() / 2, 100))
    pygame.display.update()
    again = True
    while again:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
            if event.type == pygame.KEYDOWN:
                again = False
    reset()


def reset():
    global limbs
    global guessed
    global buttons
    global word
    for i in range(len(buttons)):
        buttons[i][4] = True

    limbs = 0
    guessed = []
    word = randomWord()

#MAINLINE


# Setup buttons
increase = round(winWidth / 13)
for i in range(26):
    if i < 13:
        y = 40
        x = 25 + (increase * i)
    else:
        x = 25 + (increase * (i - 13))
        y = 85
    buttons.append([WHITE, x, y, 20, True, 65 + i])
    # buttons.append([color, x_pos, y_pos, radius, visible, char])

word = randomWord()
inPlay = True

while inPlay:
    redraw_game_window()
    pygame.time.delay(10)

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            inPlay = False
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_ESCAPE:
                inPlay = False
        if event.type == pygame.MOUSEBUTTONDOWN:
            clickPos = pygame.mouse.get_pos()
            letter = buttonHit(clickPos[0], clickPos[1])
            if letter != None:
                guessed.append(chr(letter))
                buttons[letter - 65][4] = False
                if boom(chr(letter)):
                    if limbs != 5:
                        limbs += 1
                    else:
                        end()
                else:
                    print(spacedOut(word, guessed))
                    if spacedOut(word, guessed).count('_') == 0:
                        end(True)

pygame.quit()



