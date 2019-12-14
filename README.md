# IDD-Fa19-Final-Project

# Project Idea:

The goal of this project is to create a miniature arcade basketball game similar to the life size ones you see in arcades.  Users will be able to shoot miniature size basketballs into a hoop using a mini sized catapult.  One player or two players will get to play against one another.  There will also be a scoredboard which will keep track of the users score.  The scoreboard will also include a display for the timer to indicate how much longer the round will last.  There will also be a permanent leaderboard similar to arcades so the best of the best can show off their skills. 

# Team:

Michael Chan (mkc233)

# Rough Form
![Picture](https://github.com/mkc233/IDD-Fa19-Final-Project/blob/master/IMG_2204.jpg)

# Expected Parts:

1. Plastic (possibly 3D printed)/cardboard for the physical portion of the game (the catapult, the hoop, the court, the display housing)

2. Arduino

3. Display to show score, timer, high score

4. A couple of simple buttons to start the game, pause the game, restart the game.

5. Battery to power Arduino.

6. Potentially some speakers/LEDS to improve the aesthetics.

7. Motion sensor to detect made shots




# State Diagram

![Picture](https://github.com/mkc233/IDD-Fa19-Final-Project/blob/master/IMG_2235.jpg)

The arduino boots up and on the screen it displays the start menu.  Every 5 seconds this screen will change to the high score screen which shows the current high score and the name of the person who achieved that score.  Pressing select on either of these screens will take you to the select game mode screen.  Here users can either pick 1 player or 2 player mode depending on how many users are playing.  Regardless of the game mode the get ready screen will display which counts down 5 seconds before the game starts.  If in 1 player mode only the left basketball will be activate, scoring is kept by a photocell.  Prior to the game beginining the photocell takes a baseline reading.  When a basketball passes the photocell the difference between that reading and the photocell baseline determines when a basket is scored.  If in 2 player mode, both photocells and baskets will be activate.  In 1 player mode, at the end of 60 seconds the display will show the final score.  If the score is higher than the high score in the memory, it will display the enter intials screen where the user can enter 3 intials to save their score to the memory of the arduino.  States are similar in 2 player mode, first there will be a check for which users won, if player1 then the lcd screen will indicate that and the same for player2.  Then the code will check for whether or the winners score is higher than the high score, if so it will again go to the enter intials state.  After this the display will show game over, and will reset to the main menu/high score display, ready for users to start a new game. 

# In Action:

[Start Menu/ Game mode select](https://github.com/mkc233/IDD-Fa19-Final-Project/blob/master/IMG_2238.mp4)

[Gameplay](https://github.com/mkc233/IDD-Fa19-Final-Project/blob/master/IMG_2241.mp4)

[High Score Entry](https://github.com/mkc233/IDD-Fa19-Final-Project/blob/master/IMG_2240.mp4)


# Overview
![Overview](https://github.com/mkc233/IDD-Fa19-Final-Project/blob/master/IMG_2245.JPG)

# Rims/Backboards:
![Close up of Rim](https://github.com/mkc233/IDD-Fa19-Final-Project/blob/master/IMG_2242.JPG)

# Photocell locations
![PhotoCell](https://github.com/mkc233/IDD-Fa19-Final-Project/blob/master/IMG_2243.JPG)

# Arduino and Wiring:
![Wiring](https://github.com/mkc233/IDD-Fa19-Final-Project/blob/master/IMG_2244.JPG)


# Code:

```


// include the library code:
#include <LiquidCrystal.h>
#include <Wire.h>
#include <Adafruit_RGBLCDShield.h>
#include <utility/Adafruit_MCP23017.h>
#include <EEPROM.h>
Adafruit_RGBLCDShield lcd = Adafruit_RGBLCDShield();

// Colors defined for LCD Shield
#define RED 0x1
#define YELLOW 0x3
#define GREEN 0x2
#define TEAL 0x6
#define BLUE 0x4
#define VIOLET 0x5
#define WHITE 0x7



// intialize four sections of basketball logo:
byte Ball_0[8] = {

  B00011,
  B00111,
  B01111,
  B10111,
  B11011,
  B11101,
  B11101,
  B11101
};

byte Ball_1[8] = {
  B11000,
  B11100,
  B11110,
  B11101,
  B11011,
  B10111,
  B10111,
  B10111
};

byte Ball_2[8] = {
  B11101,
  B11101,
  B11101,
  B11011,
  B10111,
  B01111,
  B00111,
  B00011
};

byte Ball_3[8] = {
  B10111,
  B10111,
  B10111,
  B11011,
  B11101,
  B11110,
  B11100,
  B11000
};

//Intialize variables
unsigned long previousTimeHome = 0;
unsigned long previousTimeGameStart = 0;
unsigned long previousTimeScoreKeeping = 0;
unsigned long previousTimeGameMode1 = 0;
unsigned long previousTimeGameMode2 = 0;
int homePageState = 0;
//Game state, display start menu or start game
int state = 0;
//Player 1 score
int score1 = 0;
//Player 2 score
int score2 = 0;
//One or two player game mode
int gameMode = 0;
int highScore = 50;
// Who won, player1 (1) or player2 (2)
int winner = 1;
//High Score
char highScoreName[] = "MKC";
//Set alphabet list for name entry
char alphabet[] = " ABCDEFGHIJKLMNOPQRSTUVWXYZ";
int eeAddress = 0;



void setup() {
  //Start Serial read
  Serial.begin(9600);
  //Setup LED outputs
  pinMode(6, OUTPUT);
  pinMode(7,OUTPUT);

  //Pull current highscore and name from memory
  EEPROM.get(eeAddress, highScoreName);
  EEPROM.get(eeAddress+sizeof(highScoreName),highScore);
  
  // initialize LCD and set up the number of columns and rows, set the background color to teal.
  lcd.begin(16, 2);
  lcd.setBacklight(TEAL);
  
  //Draw 4 sectors of the basketball logo
  lcd.createChar(0, Ball_0);
  lcd.createChar(1, Ball_1);
  lcd.createChar(2, Ball_2);
  lcd.createChar(3, Ball_3);

  //intial screen
  printLogo();
  lcd.setCursor(0, 0);
  lcd.print("NBA SHOOTOUT");
  lcd.setCursor(0, 1);
  lcd.print("PRESS START");
}

void loop() {
  //Starts game 
  if (lcd.readButtons()) {
    if (lcd.readButtons() & BUTTON_SELECT) {
      state = 1;
    }
  }
  
  //Resets highscore by pressing up and down at the same time
  if (lcd.readButtons()){
    if(lcd.readButtons() & BUTTON_UP && lcd.readButtons() & BUTTON_DOWN){
       lcd.clear();
       lcd.setCursor(0,0);
       lcd.print("HIGH SCORE RESET");
       for (int i = 0 ; i < EEPROM.length() ; i++) {
          EEPROM.write(i, 0);
       }
       EEPROM.get(eeAddress, highScoreName);
       EEPROM.get(eeAddress+sizeof(highScoreName),highScore);
    }
  }
  //Controls gamestate between main menu and gamestart
  switch (state) {
    case 0:
      startScreen();
      break;
    case 1:
      lcd.clear();
      gameStart();
      lcd.clear();
      getReady();
      lcd.clear();
      if (gameMode ==1){
        winner = 1;
        scoreKeeping1();
      }
      else if (gameMode ==2){
        scoreKeeping2();
      }
      digitalWrite(6,LOW);
      digitalWrite(7,LOW);
      state = 0;
      break;
  }
}

//Prints basketball logP
void printLogo() {
  lcd.setCursor(13, 0);
  lcd.write(byte(0));
  lcd.setCursor(14, 0);
  lcd.write(byte(1));
  lcd.setCursor(13, 1);
  lcd.write(byte(2));
  lcd.setCursor(14, 1);
  lcd.write(byte(3));
}

// Dsiplays start screen, alternates between showing high score and the main menu
void startScreen() {
  lcd.setBacklight(TEAL);
  unsigned long currentTimeHome = millis();
  if (currentTimeHome - previousTimeHome >= 5000) {
    previousTimeHome = currentTimeHome;
    if (homePageState == 0) {
      homePageState = 1;
    } else {
      homePageState = 0;
    }
    if (homePageState == 0) {
      //Show Homepage
      lcd.clear();
      printLogo();
      lcd.setCursor(0, 0);
      lcd.print("NBA SHOOTOUT");
      lcd.setCursor(0, 1);
      lcd.print("PRESS START");
    }
    else if (homePageState == 1) {
      //Show High scores
      lcd.clear();
      printLogo();
      lcd.setCursor(0, 0);
      lcd.print("HIGH SCORE");
      lcd.setCursor(0, 1);
      lcd.print(highScoreName);
      lcd.setCursor(5,1);
      lcd.print(highScore);
    }
  }
}

//Starts game
void gameStart() {
  lcd.clear();
  int start = 0;
  gameMode = 1;
  int blink1 = 0;
  int blink2 = 0;
  unsigned long currentTimeGameMode1 = millis();
  unsigned long currentTimeGameMode2 = millis();
  lcd.setCursor(0,0);
  lcd.print("SELECT GAMEMODE");
  digitalWrite(6,HIGH);
  while(start == 0){
    while(gameMode==1){
      digitalWrite(7,LOW);
      currentTimeGameMode1 = millis();
      if (currentTimeGameMode1 - previousTimeGameMode1 >= 500){
        previousTimeGameMode1 = currentTimeGameMode1;
        if (blink1 == 0){
          blink1 = 1;
        } else {
          blink1 = 0;
        }
        if (blink1 == 0){
          lcd.setCursor(0,1);
          lcd.print("1Player");
          lcd.setCursor(9,1);
          lcd.print("2Player");
        }
        else if (blink1 == 1){
          lcd.setCursor(0,1);
          lcd.print("       ");
          lcd.setCursor(9,1);
          lcd.print("2Player");
        }
      }
      if(lcd.readButtons() & BUTTON_RIGHT){
        gameMode = 2;
      }
      if(lcd.readButtons() & BUTTON_SELECT){
        start = 1;
        break;
      }
    }
    while(gameMode==2){
      digitalWrite(7,HIGH);
      currentTimeGameMode2 = millis();
      if (currentTimeGameMode2 - previousTimeGameMode2 >= 500){
        previousTimeGameMode2 = currentTimeGameMode2;
        if (blink2 == 0){
          blink2 = 1;
        } else {
          blink2 = 0;
        }
        if (blink2 == 0){
          lcd.setCursor(0,1);
          lcd.print("1Player");
          lcd.setCursor(9,1);
          lcd.print("2Player");
        }
        else if (blink2 == 1){
          lcd.setCursor(0,1);
          lcd.print("1Player");
          lcd.setCursor(9,1);
          lcd.print("       ");
        }
      }
      if(lcd.readButtons() & BUTTON_LEFT){
        gameMode = 1;
      }
      if(lcd.readButtons() & BUTTON_SELECT){
        start = 1;
        break;
      }
    }
  }
}

///Displays "Get Ready" text
void getReady(){
   previousTimeGameStart = millis();
   while (millis() - previousTimeGameStart < 3000) {
    printLogo();
    lcd.setCursor(2, 0);
    lcd.print("GET READY");
  }
  for (int i = 4000; i <= 8000; i = i + 1000) {
    while (millis() - previousTimeGameStart < i) {
      lcd.setCursor(((i - 4000) / 500), 1);
      lcd.print(round(-0.001 * i + 9));
    }
  }
  while (millis() - previousTimeGameStart < 9000) {
    lcd.setCursor(10, 1);
    lcd.print("GO!");
  }
}


//Starts keeping track of scores by reading difference in photocell readings for player1 game mode
void scoreKeeping1() {
  winner = 1;
  unsigned long wait = millis() + 500;
  unsigned long wait2 = millis() + 500;
  int baselineReading = analogRead(1);
  int baselineReading2 = analogRead(2);
  int hoop = 1;
  printLogo();
  lcd.setCursor(0, 0);
  lcd.print("TIME");
  lcd.setCursor(0, 1);
  lcd.print("SCORE");
  score1 = 0;
  previousTimeScoreKeeping = millis();
  unsigned long previousTimeScoreMode = millis();
  for (unsigned long j = 0; j <= 61000; j = j + 1000) {
      while (millis() - previousTimeScoreKeeping < j ) {
        Serial.println(analogRead(1));
        lcd.setCursor(8, 0);
        lcd.print(round(-0.001 * j + 61));
        lcd.setCursor(8, 1);
        lcd.print(score1);
        if (baselineReading - analogRead(1) > 15 && millis() > wait) {
          score1 = score1 + 2;
          wait = millis() + 500;
        }
      }
    lcd.setCursor(9, 0);
    lcd.print(" ");
  }
  lcd.clear();
  if (score1 > highScore) {
    lcd.setCursor(0, 0);
    lcd.print("NEW HIGH SCORE!!");
    lcd.setCursor(7, 1);
    lcd.print(score1);
    highScore = score1;
    delay(3000);
    nameEntry();
    lcd.clear();
    lcd.setCursor(3,0);
    lcd.print("GAME OVER");
    delay(3000);
  }
  else {
    lcd.setCursor(3, 0);
    lcd.print("FINAL SCORE");
    lcd.setCursor(7, 1);
    lcd.print(score1);
    delay(3000);
    lcd.clear();
    lcd.setCursor(3,0);
    lcd.print("GAME OVER");
    delay(3000);
  }
}


//Starts keeping track of scores by reading difference in photocell readings for 2 player game mode
void scoreKeeping2() {
  unsigned long wait = millis() + 500;
  unsigned long wait2 = millis() + 500;
  int baselineReading = analogRead(1);
  int baselineReading2 = analogRead(2);
  printLogo();
  lcd.setCursor(0, 0);
  lcd.print("TIME");
  lcd.setCursor(0, 1);
  lcd.print("P1");
  lcd.setCursor(7,1);
  lcd.print("P2");
  score1 = 0;
  score2 = 0;
  previousTimeScoreKeeping = millis();
  for (unsigned long j = 0; j <= 61000; j = j + 1000) {
    while (millis() - previousTimeScoreKeeping < j ) {
      lcd.setCursor(8, 0);
      lcd.print(round(-0.001 * j + 61));
      lcd.setCursor(3, 1);
      lcd.print(score1);
      lcd.setCursor(10,1);
      lcd.print(score2);
      Serial.println(analogRead(2));
      if (baselineReading - analogRead(1) > 15 && millis() > wait) {
        score1 = score1 + 2;
        wait = millis() + 500;
      }
      if (baselineReading2 - analogRead(2) > 15 && millis() > wait2) {
        score2 = score2 + 2;
        wait2 = millis() + 500;
      }
    }
    lcd.setCursor(9, 0);
    lcd.print(" ");
  }
  lcd.clear();
  printLogo();
  lcd.setCursor(0, 1);
  lcd.print("P1");
  lcd.setCursor(7,1);
  lcd.print("P2");
  lcd.setCursor(3, 1);
  lcd.print(score1);
  lcd.setCursor(10,1);
  lcd.print(score2);
  
  if (score1 > score2){
    digitalWrite(7,LOW);
    winner = 1;
    lcd.setCursor(0, 0);
    lcd.print("PLAYER1 WINS!");
    lcd.setCursor(5,1);
    lcd.print("<-");
    delay(5000);
    lcd.setBacklight(TEAL);
    if (score1 > highScore){
      lcd.clear();
      lcd.setCursor(0, 0);
      lcd.print("P1 HIGH SCORE");
      lcd.setCursor(7, 1);
      lcd.print(score1);
      highScore = score1;
      delay(3000);
      nameEntry();
    }
    digitalWrite(6,LOW);
    lcd.clear();
    lcd.setCursor(3,0);
    lcd.print("GAME OVER");
    delay(3000);
  }
  else if (score1 < score2) {
    digitalWrite(6,LOW);
    winner = 2;
    lcd.setCursor(0, 0);
    lcd.print("PLAYER2 WINS!");
    lcd.setCursor(5,1);
    lcd.print("->");
    delay(5000);
    lcd.setBacklight(TEAL);
    if (score2 > highScore){
      lcd.clear();
      lcd.setCursor(0, 0);
      lcd.print("P2 HIGH SCORE");
      lcd.setCursor(7, 1);
      lcd.print(score2);
      highScore = score2;
      delay(3000);
      nameEntry();
    }
    digitalWrite(7,LOW);
    lcd.clear();
    lcd.setCursor(3,0);
    lcd.print("GAME OVER");
    delay(3000);
  }
  else {
    digitalWrite(6,LOW);
    digitalWrite(7,LOW);
    lcd.setCursor(0,0);
    lcd.print("Tie Game!!");
    delay(5000);
    lcd.clear();
    lcd.setCursor(3,0);
    lcd.print("GAME OVER");
    delay(3000);
  }
}

//Name entry screen for entering your high score name and writes information to memory.
void nameEntry() {
  int k = 0;
  int x = 0;
  lcd.clear();
  lcd.setCursor(2, 0);
  lcd.print("ENTER INITIALS");
  lcd.setCursor(10,1);
  if(winner == 1){
    lcd.print(score1);
    highScore = score1;
  }
  else {
    lcd.print(score2);
    highScore = score2;
  }
  lcd.cursor();
  for(int i = 0;i<=2;i++){
    k = 0;
    x = 0;
    while(k == 0){
      lcd.setCursor(i+5,1); 
      lcd.cursor();
      if (lcd.readButtons()) {
        if (lcd.readButtons() & BUTTON_RIGHT && x <26) {
          x++;
          lcd.print(alphabet[x]);
          lcd.noCursor();
          delay(100);
        }
        else if (lcd.readButtons() & BUTTON_LEFT && x >0){
          x--;
          lcd.print(alphabet[x]);
          lcd.noCursor();
          delay(100);
        }
        else if (lcd.readButtons() & BUTTON_SELECT){
          highScoreName[i] = alphabet[x];
          k = 1;
          delay(800);
        }
      }
    }
  }
  lcd.noCursor();
  for(int y = 0;y < 4;y++){
    lcd.setCursor(0,1);
    lcd.print("                ");
    delay(500);
    lcd.setCursor(5,1);
    lcd.print(highScoreName);
    lcd.setCursor(10,1);
    lcd.print(highScore);
    delay(500);
  }
   for (int i = 0 ; i < EEPROM.length() ; i++) {
    EEPROM.write(i, 0);
  }
  EEPROM.put(eeAddress, highScoreName);
  if (winner == 1){
    EEPROM.put(eeAddress+sizeof(highScoreName), highScore);
  }
  else if (winner ==2) {
    EEPROM.put(eeAddress+sizeof(highScoreName), highScore);
  }
}

```
