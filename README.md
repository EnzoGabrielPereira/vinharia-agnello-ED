# 🍷 Sistema de Monitoramento Ambiental — Vinheria Agnello

Projeto desenvolvido para a disciplina de **Edge Computing & Computer Systems** da FIAP.

O objetivo é monitorar as condições ambientais de um espaço de armazenamento de vinhos — **temperatura**, **umidade** e **luminosidade** — exibindo o status em tempo real em um display LCD e emitindo alertas visuais (LEDs) e sonoros (buzzer) sempre que algum parâmetro sair da faixa ideal.

---

## 👥 Time Nimbus Tech — 1ESPJ

| Nome |
|------|
| Enzo Gabriel RM:570659 |
| Henrique Gumbys RM:570914 |
| Pedro Moura RM:570715 |

---

## 🍾 Contexto do Problema

Vinhos são produtos extremamente sensíveis ao ambiente em que são armazenados. Variações de temperatura, umidade e luminosidade podem comprometer diretamente a qualidade da bebida — acelerando o envelhecimento indevido, alterando o sabor ou até estragando garrafas inteiras.

A **Vinheria Agnello** precisava de uma solução acessível e eficiente para monitorar essas três variáveis em sua adega. Este projeto propõe um sistema embarcado com Arduino que lê os dados dos sensores continuamente, exibe as informações em um display LCD e alerta os responsáveis sempre que as condições saírem da faixa segura para armazenamento de vinhos.

---

## ⚙️ O que o projeto faz

A cada ciclo, o sistema realiza uma **média de 5 leituras** de cada sensor para reduzir ruído, e então avalia cada variável separadamente, exibindo o resultado no LCD por alguns segundos antes de passar para a próxima.

### 🌡️ Temperatura (DHT22)

| Condição | LED | Buzzer | Display |
|---|---|---|---|
| Abaixo de 10°C | 🟡 Amarelo | ✅ Ligado | `TEMP. BAIXA` |
| Entre 10°C e 15°C | — Nenhum | ❌ Desligado | `Temperatura OK` |
| Acima de 15°C | 🟡 Amarelo | ✅ Ligado | `TEMP. ALTA` |

### 💧 Umidade (DHT22)

| Condição | LED | Buzzer | Display |
|---|---|---|---|
| Abaixo de 50% | 🔴 Vermelho | ✅ Ligado | `Umidade BAIXA` |
| Entre 50% e 70% | — Nenhum | ❌ Desligado | `Umidade OK` |
| Acima de 70% | 🔴 Vermelho | ✅ Ligado | `Umidade ALTA` |

### 💡 Luminosidade (LDR)

| Condição | Valor LDR | LED | Buzzer | Display |
|---|---|---|---|---|
| Ambiente muito claro | < 500 | 🔴 Vermelho | ✅ Ligado | `Ambiente muito CLARO` |
| Meia luz | 500 – 800 | 🟡 Amarelo | ❌ Desligado | `Ambiente a meia luz` |
| Ambiente escuro | > 800 | 🟢 Verde | ❌ Desligado | `Ambiente ESCURO` |

> **ℹ️ Por que os valores do LDR parecem invertidos?**
> O LDR configurado com resistor pull-down no GND gera valores **altos** quando há pouca luz e valores **baixos** quando há muita luz. Por isso, `LDR < 500` indica ambiente **claro** (situação de alerta) e `LDR > 800` indica ambiente **escuro** (situação ideal para vinhos).

---

## 🧰 Componentes necessários

| Qtd | Componente |
|-----|-----------|
| 1x | Arduino Uno |
| 1x | Sensor DHT22 (temperatura e umidade) |
| 1x | Sensor LDR (fotoresistor) |
| 1x | Resistor 10kΩ (para o LDR) |
| 1x | Display LCD 16x2 com módulo I2C (endereço `0x27`) |
| 1x | LED Verde 5mm |
| 1x | LED Amarelo 5mm |
| 1x | LED Vermelho 5mm |
| 3x | Resistores 220Ω (um por LED) |
| 1x | Buzzer passivo |
| 1x | Protoboard |
| — | Jumpers (fios de conexão) |
| 1x | Cabo USB tipo B |

---

## 📦 Bibliotecas necessárias

Instale as bibliotecas abaixo antes de carregar o código. Acesse: **Sketch → Incluir Biblioteca → Gerenciar Bibliotecas**

| Biblioteca | Autor |
|-----------|-------|
| DHT sensor library | Adafruit |
| LiquidCrystal I2C | Frank de Brabander |

---

## 🔌 Esquema de Ligação

![Esquema de ligação do projeto](./circuito.png)

### Tabela de pinos

| Componente | Pino no Arduino |
|-----------|----------------|
| DHT22 (DATA) | Digital 2 |
| LDR (saída) | Analógico A0 |
| LCD SDA | Analógico A4 |
| LCD SCL | Analógico A5 |
| LED Vermelho | Digital 11 |
| LED Amarelo | Digital 8 |
| LED Verde | Digital 7 |
| Buzzer | Digital 4 |

### Detalhes de cada componente

**LDR:**
```
Arduino 5V ──── LDR ──┬──── Arduino A0
                      │
                 Resistor 10kΩ
                      │
                 Arduino GND
```

**DHT22:**
```
VCC  ──── Arduino 5V
DATA ──── Arduino Digital 2
GND  ──── Arduino GND
```

**Display LCD (I2C):**
```
VCC ──── Arduino 5V
GND ──── Arduino GND
SDA ──── Arduino A4
SCL ──── Arduino A5
```

**LEDs** (repetir para cada um com seu pino respectivo):
```
Arduino Pino ──── Resistor 220Ω ──── Anodo (+) LED ──── Catodo (−) ──── GND
```

**Buzzer:**
```
(+) ──── Arduino Digital 4
(−) ──── Arduino GND
```

---

## 💻 Código

O arquivo `.ino` está disponível neste repositório: [`vinheria_agnello.ino`](./vinheria_agnello.ino)

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <DHT.h>

// CONFIGURAÇÕES

#define DHTPIN 2
#define DHTTYPE DHT22 // 22 para o wokiwi e 11 para projeto físico

DHT dht(DHTPIN, DHTTYPE);

LiquidCrystal_I2C lcd(0x27, 16, 2);

int ledVerm = 11;
int ledVerd = 7;
int ledAmar = 8;
int buzzer = 4;
int ldr = A0;


// CARACTERE CUSTOMIZADO


byte coracao[8] = {
  B00000,
  B01010,
  B11111,
  B11111,
  B01110,
  B00100,
  B00000,
  B00000,
};

void setup()
{
  lcd.init();
  lcd.backlight();

  // Registra o caractere na posição 0 da memória do LCD
  lcd.createChar(0, coracao);

  lcd.setCursor(0, 0);
  lcd.print("Iniciando...");
  lcd.setCursor(15, 0);
  lcd.write(byte(0)); // mostra o coração

  pinMode(ledVerm, OUTPUT);
  pinMode(ledVerd, OUTPUT);
  pinMode(ledAmar, OUTPUT);
  pinMode(buzzer, OUTPUT);

  Serial.begin(9600);
  dht.begin();

  delay(2000);
  lcd.clear();
}

void loop()
{
  float somaTemp = 0;
  float somaUmid = 0;
  float somaLuz  = 0;

  for (int i = 0; i < 5; i++)
  {
    somaTemp += dht.readTemperature();
    somaUmid += dht.readHumidity();
    somaLuz  += analogRead(ldr);
    delay(200);
  }

  float temperatura = somaTemp / 5;
  float umidade     = somaUmid / 5;
  float luz         = somaLuz  / 5;

  if (isnan(temperatura) || isnan(umidade))
  {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Erro no DHT22"); // 22 para o wokiwi e 11 para projeto físico
    lcd.setCursor(15, 0);
    lcd.write(byte(0));
    delay(3000);
    return;
  }

  digitalWrite(ledVerd, LOW);
  digitalWrite(ledAmar, LOW);
  digitalWrite(ledVerm, LOW);
  noTone(buzzer);

  // ---------- TEMPERATURA ----------
  lcd.clear();

  if (temperatura < 10)
  {
    digitalWrite(ledAmar, HIGH);
    tone(buzzer, 1000);
    lcd.setCursor(0, 0); lcd.print("TEMP. BAIXA");
    lcd.setCursor(0, 1); lcd.print("Temp.= "); lcd.print(temperatura); lcd.print("C");
  }
  else if (temperatura >= 10 && temperatura <= 15)
  {
    lcd.setCursor(0, 0); lcd.print("Temperatura OK");
    lcd.setCursor(0, 1); lcd.print("Temp.= "); lcd.print(temperatura); lcd.print("C");
  }
  else
  {
    digitalWrite(ledAmar, HIGH);
    tone(buzzer, 1000);
    lcd.setCursor(0, 0); lcd.print("TEMP. ALTA");
    lcd.setCursor(0, 1); lcd.print("Temp.= "); lcd.print(temperatura); lcd.print("C");
  }

  lcd.setCursor(15, 0);
  lcd.write(byte(0)); // ♥
  delay(3000);

  // ---------- UMIDADE ----------
  digitalWrite(ledVerd, LOW);
  digitalWrite(ledAmar, LOW);
  digitalWrite(ledVerm, LOW);
  noTone(buzzer);
  lcd.clear();

  if (umidade < 50)
  {
    digitalWrite(ledVerm, HIGH);
    tone(buzzer, 1000);
    lcd.setCursor(0, 0); lcd.print("Umidade BAIXA");
    lcd.setCursor(0, 1); lcd.print("Umidade= "); lcd.print(umidade); lcd.print("%");
  }
  else if (umidade >= 50 && umidade <= 70)
  {
    lcd.setCursor(0, 0); lcd.print("Umidade OK");
    lcd.setCursor(0, 1); lcd.print("Umidade= "); lcd.print(umidade); lcd.print("%");
  }
  else
  {
    digitalWrite(ledVerm, HIGH);
    tone(buzzer, 1000);
    lcd.setCursor(0, 0); lcd.print("Umidade ALTA");
    lcd.setCursor(0, 1); lcd.print("Umidade= "); lcd.print(umidade); lcd.print("%");
  }

  lcd.setCursor(15, 0);
  lcd.write(byte(0)); // ♥
  delay(3000);

  // ---------- LUMINOSIDADE ----------
  digitalWrite(ledVerd, LOW);
  digitalWrite(ledAmar, LOW);
  digitalWrite(ledVerm, LOW);
  noTone(buzzer);
  lcd.clear();

  if (luz < 500)
  {
    digitalWrite(ledVerm, HIGH);
    tone(buzzer, 1000);
    lcd.setCursor(0, 0); lcd.print("Ambiente muito");
    lcd.setCursor(0, 1); lcd.print("CLARO");
  }
  else if (luz < 800)
  {
    digitalWrite(ledAmar, HIGH);
    lcd.setCursor(0, 0); lcd.print("Ambiente a meia");
    lcd.setCursor(0, 1); lcd.print("luz");
  }
  else
  {
    digitalWrite(ledVerd, HIGH);
    lcd.setCursor(0, 0); lcd.print("Ambiente");
    lcd.setCursor(0, 1); lcd.print("ESCURO");
  }

  lcd.setCursor(15, 0);
  lcd.write(byte(0)); // ♥
  delay(5000);
}
```

---

## 🧪 Como testar

| Ação | Resultado esperado |
|------|-------------------|
| Cobrir o LDR com a mão | LED Verde acende — ambiente escuro (ideal) |
| Apontar lanterna para o LDR | LED Vermelho acende + buzzer — ambiente muito claro |
| Aquecer o DHT22 com a mão | LED Amarelo + buzzer — temperatura alta |
| Soprar no DHT22 | Pode causar variação de umidade |
| Abrir o Serial Monitor (`Ctrl+Shift+M`) | Monitorar os valores brutos em 9600 baud |

---

## 📝 Observações

- Os limiares **500** e **800** do LDR foram calibrados durante os testes. Dependendo da iluminação do seu ambiente, pode ser necessário ajustá-los observando os valores no Serial Monitor.
- A faixa ideal de temperatura para vinhos (10°C – 15°C) e umidade (50% – 70%) foi definida com base em referências gerais de enologia.
- Se o display não inicializar, verifique o endereço I2C do módulo LCD com um scanner I2C — o endereço padrão é `0x27`, mas alguns módulos usam `0x3F`.

---

## 🔗 Links

- 🔧 [Simulação no Wokwi](https://wokwi.com/projects/464677943829242881)
- 🎥 [Vídeo explicando o projeto](https://youtu.be/o5yvvT7Hh_Q?si=EswJ9Kua4drWJI_V)
