#include <GxEPD2_BW.h>
#include <SPI.h>
#include <LittleFS.h>

const int BUTTON_PIN = 0;
String fullText = "";
int currentPage = 0;
int totalPages = 0;
const int MAX_PAGES = 200;
int pageStarts[MAX_PAGES];

const int CHARS_PER_PAGE = 350; 

GxEPD2_BW<GxEPD2_420_GDEY042T81, GxEPD2_420_GDEY042T81::HEIGHT> display(GxEPD2_420_GDEY042T81(/*CS=*/ 5, /*DC=*/ 17, /*RST=*/ 16, /*BUSY=*/ 4));

void printWrappedText(String text) {
  String currentWord = "";
  for (unsigned int i = 0; i <= text.length(); i++) {
    char c = (i < text.length()) ? text[i] : '\0';
    
    if (c == '\r') {
      continue;
    }
    
    if (c == ' ' || c == '\n' || c == '\0') {
      int16_t x1, y1;
      uint16_t w, h;
      display.getTextBounds(currentWord, display.getCursorX(), display.getCursorY(), &x1, &y1, &w, &h);
      
      if (display.getCursorX() + w > display.width() && currentWord.length() > 0) {
        display.println();
      }
      display.print(currentWord);
      
      if (c == '\n') {
        display.println();
      } else if (c == ' ') {
        display.print(" ");
      }
      currentWord = "";
    } else {
      currentWord += c;
    }
  }
}

void drawPage() {
  display.setPartialWindow(0, 0, display.width(), display.height());
  display.firstPage();
  do {
    display.fillScreen(GxEPD_WHITE);
    display.setTextColor(GxEPD_BLACK);
    display.setFont(); 
    display.setTextSize(2); 
    display.setCursor(0, 0);
    
    int startIdx = pageStarts[currentPage];
    int endIdx = (currentPage < totalPages - 1) ? pageStarts[currentPage + 1] : fullText.length();
    
    String pageText = fullText.substring(startIdx, endIdx);
    printWrappedText(pageText);
    
    display.fillRect(0, 275, 400, 25, GxEPD_WHITE); 
    
    String counterText = String(currentPage + 1) + " / " + String(totalPages);
    int16_t x1, y1;
    uint16_t w, h;
    display.getTextBounds(counterText, 0, 0, &x1, &y1, &w, &h);
    
    display.setCursor(display.width() - w - 10, 280);
    display.print(counterText);
  } while (display.nextPage());
}

void setup() {
  Serial.begin(115200);
  pinMode(BUTTON_PIN, INPUT_PULLUP);
  
  SPI.begin(18, 19, 23, 5); 
  
  display.init(115200, true, 2, false);
  display.setRotation(0);
  display.setTextWrap(false); 
  
  if (!LittleFS.begin(true)) {
    fullText = "LittleFS Mount Failed";
  } else {
    File file = LittleFS.open("/test123.txt", "r");
    if (!file) {
      fullText = "Failed to open test123.txt";
    } else {
      while (file.available()) {
        char c = file.read();
        if (c == (char)0xE2) { 
          char c2 = file.read();
          char c3 = file.read();
          if (c2 == (char)0x80 && (c3 == (char)0x98 || c3 == (char)0x99)) {
            fullText += '\'';
          } else if (c2 == (char)0x80 && (c3 == (char)0x9C || c3 == (char)0x9D)) {
            fullText += '"';
          } else {
            fullText += c;
            fullText += c2;
            fullText += c3;
          }
        } else {
          fullText += c;
        }
      }
      file.close();
    }
  }

  int currentIdx = 0;
  while (currentIdx < fullText.length() && totalPages < MAX_PAGES) {
    pageStarts[totalPages] = currentIdx;
    int nextIdx = currentIdx + CHARS_PER_PAGE;
    
    if (nextIdx >= fullText.length()) {
      nextIdx = fullText.length();
    } else {
      int spaceIdx = fullText.lastIndexOf(' ', nextIdx);
      int newlineIdx = fullText.lastIndexOf('\n', nextIdx);
      int breakIdx = max(spaceIdx, newlineIdx);
      
      if (breakIdx > currentIdx) {
        nextIdx = breakIdx + 1;
      }
    }
    currentIdx = nextIdx;
    totalPages++;
  }
  
  if (totalPages == 0) {
    totalPages = 1;
  }

  display.setFullWindow();
  display.firstPage();
  do {
    display.fillScreen(GxEPD_WHITE);
  } while (display.nextPage());
  
  drawPage();
}

void loop() {
  if (digitalRead(BUTTON_PIN) == LOW) {
    delay(50); 
    if (digitalRead(BUTTON_PIN) == LOW) {
      
      while(digitalRead(BUTTON_PIN) == LOW); 
      
      unsigned long releaseTime = millis();
      bool isDoublePress = false;
      
      while (millis() - releaseTime < 350) { 
        if (digitalRead(BUTTON_PIN) == LOW) {
          delay(50);
          if (digitalRead(BUTTON_PIN) == LOW) {
            isDoublePress = true;
            while(digitalRead(BUTTON_PIN) == LOW); 
            break;
          }
        }
      }
      
      if (isDoublePress) { 
        if (currentPage > 0) {
          currentPage--;
          drawPage();
        }
      } else { 
        if (currentPage < totalPages - 1) {
          currentPage++;
          drawPage();
        }
      }
    }
  }
}
