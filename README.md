#include <Ultrasonic.h>
Ultrasonic ultrasonic(8, 7); // TRIG = 8, ECHO = 7

// Sensores de linha
const int sensorTras = 13; //AD
const int sensorEsq = 12; //FE
const int sensorDir = 11; //FD

// Botão de ativação
const int botao = 2;

// Ponte H (Motores)
const int motorEsq_A = 5;
const int motorEsq_B = 6;
const int motorDir_A = 9;
const int motorDir_B = 10;

// Parâmetros
const int velocidade = 150;          // PWM ajustado para ~195RPM
const int tempoRecuo = 10;          // ms
const int tempoGiro = 10;           // ms
const int alcanceAtaque = 10;        // cm
const int tempoInicial = 500;       // ms (contagem regressiva)

// ============ SETUP ============
void setup() {
  // Sensores de linha
  pinMode(sensorTras, INPUT);
  pinMode(sensorEsq, INPUT);
  pinMode(sensorDir, INPUT);

  // Botão
  pinMode(botao, INPUT_PULLUP);

  // Motores
  pinMode(motorEsq_A, OUTPUT);
  pinMode(motorEsq_B, OUTPUT);
  pinMode(motorDir_A, OUTPUT);
  pinMode(motorDir_B, OUTPUT);

  Serial.begin(9600);
  parar();

  // Aguardar botão pressionado
  Serial.println("Aguardando botão...");
  while (digitalRead(botao) == LOW) {
    parar();
    delay(10);
  }

  Serial.println("Ativando em 1 segundos...");
  delay(tempoInicial);
}

// ============ LOOP PRINCIPAL ============
void loop() {
  // Proteção de borda
  if (digitalRead(sensorDir) == LOW || digitalRead(sensorEsq) == LOW || digitalRead(sensorTras) == LOW) {
    Serial.println("Linha detectada! Recuando e girando...");
    re();
    delay(tempoRecuo);
    girar();
    delay(tempoGiro);
    return;
  }

  // Leitura da distância
  int cm = ultrasonic.read(CM);

  if (cm > 0 && cm <= alcanceAtaque) {
    Serial.println("Inimigo detectado! Atacando...");
    atacar();
    delay(0); // pequeno impulso
    return;
  }

  // Movimento padrão de busca
  frente();
  delay(1250);
}

// ============ FUNÇÕES DE MOVIMENTO ============

void frente() {
  Serial.println("Frente");
  analogWrite(motorEsq_A, velocidade);
  analogWrite(motorEsq_B, 0);
  analogWrite(motorDir_A, velocidade);
  analogWrite(motorDir_B, 0);
}

void re() {
  Serial.println("Ré");
  analogWrite(motorEsq_A, 0);
  analogWrite(motorEsq_B, velocidade);
  analogWrite(motorDir_A, 0);
  analogWrite(motorDir_B, velocidade);
}

void girar() {
  Serial.println("Girando");
  analogWrite(motorEsq_A, velocidade);
  analogWrite(motorEsq_B, 0);
  analogWrite(motorDir_A, 0);
  analogWrite(motorDir_B, velocidade);
}

void atacar() {
  Serial.println("Atacar!");
  analogWrite(motorEsq_A, velocidade);
  analogWrite(motorEsq_B, 0);
  analogWrite(motorDir_A, velocidade);
  analogWrite(motorDir_B, 0);
}

void parar() {
  analogWrite(motorEsq_A, 0);
  analogWrite(motorEsq_B, 0);
  analogWrite(motorDir_A, 0);
  analogWrite(motorDir_B, 0);
}
