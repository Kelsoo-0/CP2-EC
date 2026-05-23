# 🍷 Vinheria Inteligente

> Projeto acadêmico desenvolvido em Arduino utilizando C++ para monitoramento de ambiente de armazenamento de vinhos.

## 📖 Sobre o Projeto:

Este projeto foi criado com o objetivo de desenvolver um sistema inteligente para uma vinheria tradicional, permitindo o monitoramento das condições do ambiente para garantir a qualidade dos vinhos armazenados.

O sistema realiza:
- leitura de temperatura 🌡️
- monitoramento de umidade 💧
- controle de luminosidade 💡
- alertas com LEDs e buzzer 🚨
- exibição de dados em display LCD 📺

Tudo desenvolvido utilizando Arduino e programação em C++.

## 🛠️ Tecnologias Utilizadas:

- Arduino
- C++

## 🔌 Componentes Utilizados:

- Arduino UNO
- Sensor DHT11
- Sensor de Luminosidade
- Display LCD
- LEDs
- Buzzer
- Protoboard
- Resistores
- Jumpers

## ⚙️ Funcionalidades:

- ✅ Monitoramento da temperatura
- ✅ Monitoramento da umidade
- ✅ Monitoramento da luminosidade
- ✅ Exibição dos valores no display LCD
- ✅ Sistema de alerta com LEDs
- ✅ Sistema de alerta sonoro com buzzer
- ✅ Avisos quando os níveis estiverem críticos

## 📚 Bibliotecas Utilizadas:

```
#include <DHT.h>
#include <LiquidCrystal.h>
```
## Código para DHT11:

<details>

```
#include <Wire.h>

#include <LiquidCrystal_I2C.h>

#include <DHT.h>
 
// -----------------------------

// LCD 16x2 I2C

// -----------------------------

// Se nao funcionar com 0x27, troque para 0x3F

LiquidCrystal_I2C lcd(0x27, 16, 2);
 
// -----------------------------

// DHT11

// -----------------------------

#define DHTPIN 2

#define DHTTYPE DHT11
 
DHT dht(DHTPIN, DHTTYPE);
 
// -----------------------------

// Sensores

// -----------------------------

int ldrPin = A0;
 
// -----------------------------

// LEDs e buzzer

// -----------------------------

int ledVerde = 9;

int ledAmarelo = 10;

int ledVermelho = 11;

int buzzer = 8;
 
// -----------------------------

// Controle da tela do LCD

// -----------------------------

int telaAtual = 0;
 
void setup() {

  pinMode(ledVerde, OUTPUT);

  pinMode(ledAmarelo, OUTPUT);

  pinMode(ledVermelho, OUTPUT);

  pinMode(buzzer, OUTPUT);
 
  digitalWrite(ledVerde, LOW);

  digitalWrite(ledAmarelo, LOW);

  digitalWrite(ledVermelho, LOW);

  noTone(buzzer);
 
  Serial.begin(9600);
 
  // Inicializa o DHT11

  dht.begin();
 
  // Inicializa o LCD

  lcd.init();

  lcd.backlight();

  lcd.clear();
 
  lcd.setCursor(0, 0);

  lcd.print("Sistema");

  lcd.setCursor(0, 1);

  lcd.print("Vinheria");

  delay(2000);

  lcd.clear();

}
 
void loop() {

  float luz = 0;
 
  // Faz a media de 5 leituras do LDR

  for (int i = 0; i < 5; i++) {

    luz += analogRead(ldrPin);

    delay(200);

  }
 
  luz = luz / 5;
 
  // Leitura do DHT11

  float temperatura = dht.readTemperature();

  float umidade = dht.readHumidity();
 
  // Verifica se o DHT11 leu corretamente

  if (isnan(temperatura) || isnan(umidade)) {

    Serial.println("Erro ao ler DHT11");
 
    lcd.clear();

    lcd.setCursor(0, 0);

    lcd.print("Erro DHT11");

    lcd.setCursor(0, 1);

    lcd.print("Verifique fios");
 
    digitalWrite(ledVerde, LOW);

    digitalWrite(ledAmarelo, LOW);

    digitalWrite(ledVermelho, HIGH);

    tone(buzzer, 1000);
 
    delay(3000);

    return;

  }
 
  Serial.print("Luz: ");

  Serial.print(luz);
 
  Serial.print(" | Temp: ");

  Serial.print(temperatura);

  Serial.print(" C");
 
  Serial.print(" | Umidade: ");

  Serial.print(umidade);

  Serial.println("%");
 
  controlarLedsEBuzzer(luz, temperatura, umidade);

  mostrarNoDisplay(luz, temperatura, umidade);
 
  telaAtual++;
 
  if (telaAtual > 2) {

    telaAtual = 0;

  }
 
  delay(5000);

}
 
void controlarLedsEBuzzer(float luz, float temperatura, float umidade) {

  digitalWrite(ledVerde, LOW);

  digitalWrite(ledAmarelo, LOW);

  digitalWrite(ledVermelho, LOW);

  noTone(buzzer);
 
  bool problemaLuz = false;

  bool alertaLuz = false;

  bool luzOk = false;
 
  bool problemaTemperatura = false;

  bool problemaUmidade = false;
 
  // -----------------------------

  // Luminosidade

  // -----------------------------
 
  if (luz < 100) {

    luzOk = true;

  }
 
  if (luz >= 100 && luz < 450) {

    alertaLuz = true;

  }
 
  if (luz >= 450) {

    problemaLuz = true;

  }
 
  // -----------------------------

  // Temperatura ideal: 5 C ate 23 C

  // -----------------------------
 
  if (temperatura < 5 || temperatura > 23) {

    problemaTemperatura = true;

  }
 
  // -----------------------------

  // Umidade ideal: 40% ate 70%

  // -----------------------------
 
  if (umidade < 40 || umidade > 70) {

    problemaUmidade = true;

  }
 
  // -----------------------------

  // Controle dos LEDs

  // -----------------------------
 
  // Vermelho: luz muito clara ou umidade fora da faixa

  if (problemaLuz == true || problemaUmidade == true) {

    digitalWrite(ledVermelho, HIGH);

  }
 
  // Amarelo: luz em alerta ou temperatura fora da faixa

  if (alertaLuz == true || problemaTemperatura == true) {

    digitalWrite(ledAmarelo, HIGH);

  }
 
  // Verde: tudo dentro do ideal

  if (luzOk == true && problemaTemperatura == false && problemaUmidade == false) {

    digitalWrite(ledVerde, HIGH);

  }
 
  // -----------------------------

  // Controle do buzzer

  // -----------------------------
 
  if (problemaLuz == true || problemaTemperatura == true || problemaUmidade == true) {

    tone(buzzer, 1000);

  } else {

    noTone(buzzer);

  }

}
 
void mostrarNoDisplay(float luz, float temperatura, float umidade) {

  lcd.clear();
 
  if (telaAtual == 0) {

    mostrarLuz(luz);

  }
 
  if (telaAtual == 1) {

    mostrarTemperatura(temperatura);

  }
 
  if (telaAtual == 2) {

    mostrarUmidade(umidade);

  }

}
 
void mostrarLuz(float luz) {

  if (luz < 100) {

    lcd.setCursor(0, 0);

    lcd.print("Ambiente escuro");
 
    lcd.setCursor(0, 1);

    lcd.print("Luz OK");

  }
 
  if (luz >= 100 && luz < 450) {

    lcd.setCursor(0, 0);

    lcd.print("Ambiente meia");
 
    lcd.setCursor(0, 1);

    lcd.print("luz");

  }
 
  if (luz >= 450) {

    lcd.setCursor(0, 0);

    lcd.print("Ambiente muito");
 
    lcd.setCursor(0, 1);

    lcd.print("CLARO");

  }

}
 
void mostrarTemperatura(float temperatura) {

  if (temperatura >= 5 && temperatura <= 23) {

    lcd.setCursor(0, 0);

    lcd.print("Temperatura OK");

  }
 
  if (temperatura > 23) {

    lcd.setCursor(0, 0);

    lcd.print("Temp. ALTA");

  }
 
  if (temperatura < 5) {

    lcd.setCursor(0, 0);

    lcd.print("Temp. BAIXA");

  }
 
  lcd.setCursor(0, 1);

  lcd.print("Temp = ");

  lcd.print(temperatura, 1);

  lcd.print(" C");

}
 
void mostrarUmidade(float umidade) {

  if (umidade >= 40 && umidade <= 70) {

    lcd.setCursor(0, 0);

    lcd.print("Umidade OK");

  }
 
  if (umidade > 70) {

    lcd.setCursor(0, 0);

    lcd.print("Umidade ALTA");

  }
 
  if (umidade < 40) {

    lcd.setCursor(0, 0);

    lcd.print("Umidade BAIXA");

  }
 
  lcd.setCursor(0, 1);

  lcd.print("Umidade = ");

  lcd.print(umidade, 0);

  lcd.print("%");

}
```

</details>



## Link para vídeo:
```
https://youtu.be/SqTrzKh6jWI
```

## Link para simulação:
```
https://www.tinkercad.com/things/iKN4fUdDBRs-vinharia-agnello-cp2/editel?returnTo=https%3A%2F%2Fwww.tinkercad.com%2Fdashboard&sharecode=EE9UFmaJDPdA1rO3EHS9JDFeoR8n-BWUr1gIWW4r8nU
```

## 👨‍💻 Integrantes:

- Gabriel Flausino -> RM572486
- Kelso Oliveira -> RM573719
- Joaquim Gaspardo -> RM572208
- Felipe Kenji -> 568739