# Sistema de Monitoramento de Luminosidade - Vinheria Agnello

Projeto desenvolvido para a matéria de Edge Computing da FIAP. O objetivo é monitorar a luminosidade de um ambiente de armazenamento de vinhos usando Arduino, acendendo LEDs de alerta e acionando uma buzina quando a luz estiver alta demais.

---
## Time Nimbus Tech - 1ESPJ

- Enzo Gabriel
- Henrique Gumbys
- Pedro Moura
---

## O que o projeto faz

O sensor LDR lê a luminosidade do ambiente. Dependendo do valor lido, um dos três LEDs acende:

- **LED Verde** → luminosidade OK, ambiente adequado para os vinhos
- **LED Amarelo** → luminosidade em nível de atenção
- **LED Vermelho** → luminosidade alta demais, algo precisa ser feito. Além disso, um buzzer apita por 3 segundos. Se a luminosidade continuar alta, ele volta a apitar.

---

## Componentes necessários

Antes de começar, separe tudo que você vai precisar:

- 1x Arduino Uno
- 1x Sensor LDR (fotoresistor)
- 1x Resistor 10kΩ (para o LDR — geralmente é o marrom/preto/laranja/dourado)
- 1x LED Verde 5mm
- 1x LED Amarelo 5mm
- 1x LED Vermelho 5mm
- 3x Resistores 220Ω (um para cada LED — geralmente é o vermelho/vermelho/marrom/dourado)
- 1x Buzzer passivo
- 1x Protoboard
- Jumpers (fios de conexão)
- 1x Cabo USB tipo B (o mesmo de impressora, para conectar o Arduino no computador)

---

## Tutorial — passo a passo

### Passo 1 — Instalar a Arduino IDE

A Arduino IDE é o programa que você vai usar para escrever e enviar o código para o Arduino.

1. Acesse: https://www.arduino.cc/en/software
2. Baixe a versão para o seu sistema operacional (Windows, Mac ou Linux)
3. Instale normalmente como qualquer outro programa
4. Abra a Arduino IDE depois que a instalação terminar

---

### Passo 2 — Montar o circuito na protoboard

Antes de ligar qualquer coisa no computador, monte tudo na protoboard com o Arduino **desconectado**.

**Conectando o LDR:**
1. Encaixe o LDR na protoboard
2. Em um dos lados do LDR, conecte um jumper até o pino **5V** do Arduino
3. No outro lado do LDR, conecte um jumper até o pino **A0** do Arduino — esse mesmo lado também deve ter o resistor de **10kΩ** conectado até o **GND** do Arduino. Isso é o que permite o Arduino ler a luminosidade corretamente.

**Conectando os LEDs:**

Os LEDs têm duas perninhas: a maior é o positivo (+) e a menor é o negativo (−).

1. Encaixe os três LEDs na protoboard, cada um em uma coluna diferente
2. Na perninha longa (positivo) de cada LED, conecte um resistor de **220Ω**
3. Do outro lado do resistor, conecte um jumper até o pino digital do Arduino conforme a tabela abaixo
4. Na perninha curta (negativo) de cada LED, conecte um jumper até o **GND** do Arduino

| LED          | Pino no Arduino |
|--------------|-----------------|
| LED Vermelho | Digital 11      |
| LED Amarelo  | Digital 8       |
| LED Verde    | Digital 7       |

**Conectando o Buzzer:**

O buzzer também tem um lado positivo (+) e um negativo (−), normalmente indicado por uma marcação ou pelo tamanho das perninhas.

1. Encaixe o buzzer na protoboard
2. Na perninha positiva (+), conecte um jumper até o pino **Digital 4** do Arduino
3. Na perninha negativa (−), conecte um jumper até o **GND** do Arduino

---

### Passo 3 — Conectar o Arduino no computador

1. Pegue o cabo USB e conecte o Arduino no computador
2. O Arduino deve acender um LED verde indicando que está recebendo energia

---

### Passo 4 — Configurar a Arduino IDE

1. Abra a Arduino IDE
2. No menu superior, clique em **Ferramentas → Placa** e selecione **Arduino Uno**
3. Ainda em **Ferramentas**, clique em **Porta** e selecione a porta que aparece com o Arduino (geralmente algo como `COM5` no Windows ou `/dev/cu.usbmodem...` no Mac)
   - Se não aparecer nenhuma porta, tente outro cabo USB ou outra entrada do computador

---

### Passo 5 — Carregar o código

1. Na Arduino IDE, apague qualquer código que estiver na tela
2. Cole o código abaixo:

```cpp
int ledVerm = 11;
int ledVerd = 7;
int ledAmar = 8;
int buzzer = 4;


void setup()
{
  pinMode(ledVerm, OUTPUT);
  pinMode(ledVerd, OUTPUT);
  pinMode(ledAmar, OUTPUT);
  pinMode(buzzer, OUTPUT);

  Serial.begin(9600);
}

void loop()
{
  int valor = analogRead(A0);
  Serial.println(valor);

  if (valor < 500) {
    digitalWrite(ledVerd, LOW);
    digitalWrite(ledAmar, LOW);
    digitalWrite(ledVerm, HIGH);

    tone(buzzer, 800);
    delay(3000);
    noTone(buzzer);   
  }
  else if (valor < 800) {
    digitalWrite(ledVerd, LOW);
    digitalWrite(ledAmar, HIGH);
    digitalWrite(ledVerm, LOW);
  }
  else {
    digitalWrite(ledVerd, HIGH);
    digitalWrite(ledAmar, LOW);
    digitalWrite(ledVerm, LOW);  
  }

  delay(500);
}
```

3. Clique no botão **Upload** (ícone de seta →) no canto superior esquerdo da tela
4. Aguarde a mensagem **"Upload concluído"** aparecer na parte de baixo da tela

---

### Passo 6 — Testar o projeto

Com o código carregado, o projeto já está funcionando. Para testar:

- **Cubra o LDR com a mão** (simulando ambiente escuro) → o LED Verde deve acender
- **Aponte uma lanterna para o LDR** → o LED Amarelo deve acender, e se a luz for muito forte, o LED Vermelho acende e o buzzer apita por 3 segundos

Se quiser ver os números que o sensor está lendo, abra o **Serial Monitor** (`Ctrl+Shift+M`) e configure para **9600 baud** no canto inferior direito da janela. Os valores vão de 0 a 1023, sendo 0 escuro total e 1023 luz máxima.

---

## Observação

Os valores **500** e **800** foram os que funcionaram melhor durante nossos testes. Se no seu ambiente o sensor reagir diferente do esperado, você pode ajustar esses números diretamente no código, observando os valores que aparecem no Serial Monitor e decidindo onde faz mais sentido colocar cada limite.

---

## Links

- Link do projeto no Wokwi: [https://wokwi.com/projects/462057407545696257](https://wokwi.com/projects/462057407545696257)
- Link do vídeo explicando o projeto: [https://www.youtube.com/watch?v=3z30k7ACh4M](https://www.youtube.com/watch?v=3z30k7ACh4M)
