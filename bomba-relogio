#include <TM1637Display.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <Keypad.h>

#define CLK A3
#define DIO A2
#define BUZZER 12
#define OLED_RESET -1

// Display de 7 segmentos
TM1637Display display(CLK, DIO);

// Configuração do display OLED
Adafruit_SSD1306 oled(128, 64, &Wire, OLED_RESET);

// Configuração do teclado matricial
const byte ROWS = 4;
const byte COLS = 4;
char keys[ROWS][COLS] = {
  {'1','2','3','A'},
  {'4','5','6','B'},
  {'7','8','9','C'},
  {'*','0','#','D'}
};
byte rowPins[ROWS] = {9, 8, 7, 6};    // Pinos das linhas do teclado
byte colPins[COLS] = {5, 4, 3, 2};    // Pinos das colunas do teclado
Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);

// Variáveis para o sistema de senha e cronômetro
String senha = "12"; // Senha correta
String tentativa = "";
int tentativasRestantes = 3;
bool bombaDesarmada = false;
bool jogoAtivo = true;

unsigned long tempoInicial = 60000; // 1 minuto em milissegundos
unsigned long tempoRestante = tempoInicial;
unsigned long ultimoTempo = millis();

void setup() {
  // Iniciar componentes
  display.setBrightness(0x0f);
  oled.begin(SSD1306_SWITCHCAPVCC, 0x3C);
  pinMode(BUZZER, OUTPUT);
  Serial.begin(9600);
  iniciarJogo();
}

void loop() {
  
  if (!jogoAtivo) { // Verifica se o jogo está inativo e aguarda pressionar #
    char key = keypad.getKey();
    if (key == '#') { // Reinicia o jogo ao pressionar #
      iniciarJogo();
    }
    return;
  }

  // Atualizar o tempo restante
  if (millis() - ultimoTempo >= 1000) { // Atualizar a cada segundo
    ultimoTempo = millis();
    tempoRestante -= 1000;

    int minutos = tempoRestante / 60000;
    int segundos = (tempoRestante % 60000) / 1000;
    display.showNumberDecEx(minutos * 100 + segundos, 0x40, true); // Exibe o tempo no formato MM:SS
  }

  if (tempoRestante <= 0) {
    explodirBomba(); // Explode a bomba se o tempo acabar
    return;
  }

  // Captura a entrada do teclado
  char key = keypad.getKey(); 
  
  Serial.println(key);
  
  if (key) {
    if (key == '#') {
      // Checar a senha ao pressionar '#'
      if (tentativa == senha) {
        desarmarBomba();
      } else {
        tentativasRestantes--;
        tempoRestante -= 15000; // Reduz 15 segundos a cada erro
        if (tentativasRestantes > 0) {
          oled.clearDisplay();
          oled.setCursor(0, 10);
          oled.print("Bommmm");
          oled.display();
          tone(BUZZER, 1000, 500); // Buzzer toca som agudo por meio segundo
          delay(1000); // Espera um segundo
        } else {
          explodirBomba();
          jogoAtivo = false; // Desativa o jogo até pressionar #
        }
      }
      tentativa = ""; // Reinicia a tentativa
    } else if (key >= '0' && key <= '9') {
      tentativa += key; // Adiciona o número à tentativa atual
    }
  }
}

void iniciarJogo() {
  bombaDesarmada = false;
  jogoAtivo = true;
  tentativasRestantes = 3;
  tempoRestante = tempoInicial;
  tentativa = "";

  // Mensagem inicial
  oled.clearDisplay();
  oled.setTextSize(2);
  oled.setTextColor(WHITE);
  oled.setCursor(0, 10);
  oled.print("Bomba ativa");
  oled.display();
}

void desarmarBomba() {
  bombaDesarmada = true;
  oled.clearDisplay();
  oled.setCursor(0, 10);
  oled.print("Bomba desarmada");
  oled.display();
  noTone(BUZZER); // Para o som, caso esteja tocando
  jogoAtivo = false; // Desativa o jogo até pressionar #
}

void explodirBomba() {
  bombaDesarmada = true;
  oled.clearDisplay();
  oled.setCursor(0, 10);
  oled.print("Explosao");
  oled.display();
  for (int i = 0; i < 10; i++) {
    tone(BUZZER, 2000, 500); // Som mais intenso e contínuo
    delay(500);
  }
  noTone(BUZZER);
  jogoAtivo = false; // Desativa o jogo até pressionar #
}
