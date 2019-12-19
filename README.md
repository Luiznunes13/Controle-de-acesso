# Controle-de-acesso

// Programa de funcionamento do Leitor Biometrico FPM 10-A E RFID RC 522
// Traduzido e adaptado por Usinainfo

#include <Adafruit_Fingerprint.h>
#include <SoftwareSerial.h>

int getFingerprintIDez();

SoftwareSerial mySerial(2, 3);
Adafruit_Fingerprint finger = Adafruit_Fingerprint(&mySerial);

// Programa : RFID - Controle de acesso / cancela
// Autor : Arduino e Cia

#include <SPI.h>
#include <MFRC522.h>



#define SS_PIN 10
#define RST_PIN 9
// Definicoes pino modulo RC522
MFRC522 mfrc522(SS_PIN, RST_PIN);

// Leds indicadores acesso liberado ou negado
int led_liberado = 5;
int led_negado = 6;

char st[20];

void setup()
{
  pinMode(led_liberado, OUTPUT);
  pinMode(led_negado, OUTPUT);

  // Inicia  SPI bus
  SPI.begin();
  // Inicia MFRC522
  mfrc522.PCD_Init();
  // Mensagens iniciais no serial monitor
  Serial.println("Aproxime o seu cartao do leitor...");
  Serial.println();

  Serial.begin(9600);
  Serial.println("Iniciando Leitor Biometrico");
  pinMode(8, OUTPUT);
  pinMode(7, OUTPUT);

  finger.begin(57600);

  if (finger.verifyPassword()) {
    Serial.println("Leitor Biometrico Encontrado");
  } else {
    Serial.println("Leitor Biometrico nao encontrada");
    while (1);
  }
  Serial.println("Esperando Dedo para Verificar");
}
//Função: inicializa parâmetros de conexão MQTT(endereço do

void loop()
{
  getFingerprintIDez();
  digitalWrite(7, HIGH);
  delay(50);

  // Aguarda a aproximacao do cartao
  if ( ! mfrc522.PICC_IsNewCardPresent())
  {
    return;
  }
  // Seleciona um dos cartoes
  if ( ! mfrc522.PICC_ReadCardSerial())
  {
    return;
  }
  // Mostra UID na serial
  Serial.print("UID da tag :");
  String conteudo = "";
  byte letra;
  for (byte i = 0; i < mfrc522.uid.size; i++)
  {
    Serial.print(mfrc522.uid.uidByte[i] < 0x10 ? " 0" : " ");
    Serial.print(mfrc522.uid.uidByte[i], HEX);
    conteudo.concat(String(mfrc522.uid.uidByte[i] < 0x10 ? " 0" : " "));
    conteudo.concat(String(mfrc522.uid.uidByte[i], HEX));
  }
  Serial.println();
  Serial.print("Mensagem : ");
  conteudo.toUpperCase();

  // Testa se o cartao1 foi lido
  if (conteudo.substring(1) == "95 30 56 59")
  {
    digitalWrite(led_liberado, HIGH);
    Serial.println("Cartao1 - Acesso liberado !");
    Serial.println();
    delay(3000);
    digitalWrite(led_liberado, LOW);
  }

  // Testa se o cartao2 foi lido
  if (conteudo.substring(1) == "25 7A B4 11")
  {
    Serial.println("Cartao2 - Acesso negado !!");
    Serial.println();
    // Pisca o led vermelho
    for (int i = 1; i < 5 ; i++)
    {
      digitalWrite(led_negado, HIGH);
      delay(200);
      digitalWrite(led_negado, LOW);
      delay(200);
    }
  }
  delay(1000);
}

uint8_t getFingerprintID() {
  uint8_t p = finger.getImage();
  switch (p) {
    case FINGERPRINT_OK:
      Serial.println("Imagem Capturada");
      break;
    case FINGERPRINT_NOFINGER:
      Serial.println("Dedo nao Localizado");
      return p;
    case FINGERPRINT_PACKETRECIEVEERR:
      Serial.println("Erro ao se comunicar");
      return p;
    case FINGERPRINT_IMAGEFAIL:
      Serial.println("Erro ao Capturar");
      return p;
    default:
      Serial.println("Erro desconhecido");
      return p;
  }

  p = finger.image2Tz();
  switch (p) {
    case FINGERPRINT_OK:
      Serial.println("Imagem Convertida");
      break;
    case FINGERPRINT_IMAGEMESS:
      Serial.println("Imagem muito confusa");
      return p;
    case FINGERPRINT_PACKETRECIEVEERR:
      Serial.println("Erro ao se comunicar");
      return p;
    case FINGERPRINT_FEATUREFAIL:
      Serial.println("Impossivel localizar Digital");
      return p;
    case FINGERPRINT_INVALIDIMAGE:
      Serial.println("Impossivel Localizar Digital");
      return p;
    default:
      Serial.println("Erro Desconhecido");
      return p;
  }

  p = finger.fingerFastSearch();
  if (p == FINGERPRINT_OK) {
    Serial.println("Digital Encontrada");
  } else if (p == FINGERPRINT_PACKETRECIEVEERR) {
    Serial.println("Erro ao se comunicar");
    return p;
  } else if (p == FINGERPRINT_NOTFOUND) {
    Serial.println("Digital Desconhecida");
    return p;
  } else {
    Serial.println("Erro Desconhecido");
    return p;
  }

  Serial.print("ID # Encontrado");
  Serial.print(finger.fingerID);
  Serial.print(" com precisao de ");
  Serial.println(finger.confidence);
}

int getFingerprintIDez() {
  uint8_t p = finger.getImage();
  if (p != FINGERPRINT_OK)  return -1;

  p = finger.image2Tz();
  if (p != FINGERPRINT_OK)  return -1;

  p = finger.fingerFastSearch();
  if (p != FINGERPRINT_OK)  return -1;

  digitalWrite(7, LOW);
  digitalWrite(8, HIGH);
  delay(1000);
  digitalWrite(8, LOW);
  delay(1000);
  digitalWrite(7, HIGH);
  Serial.print("ID # Encontrado");
  Serial.print(finger.fingerID);
  Serial.print(" com precisao de ");
  Serial.println(finger.confidence);
  return finger.fingerID;



}
controle de acesso usando um arduino uno 
