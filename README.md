Slave DeviceS
#include <SPI.h>
#include <LoRa.h>

const int csPin = 15;
const int resetPin = 4;
const int irqPin = 5;

const byte slaveAddress = 0x01;
const byte masterAddress = 0xAA;
const byte syncWord = 0x22;

void setup() {
    Serial.begin(115200);

    while (!Serial);

    // Setup LoRa
    LoRa.setPins(csPin, resetPin, irqPin);
    if (!LoRa.begin(433E6)) {
        Serial.println("Starting LoRa failed!");
        while (1);
    }
    LoRa.setSpreadingFactor(7); // Optimized spreading factor
    LoRa.setSignalBandwidth(125E3); // Optimized bandwidth
    LoRa.setCodingRate4(5); // Optimized coding rate
    LoRa.setSyncWord(syncWord);

    Serial.println("LoRa Initialized");
}

void loop() {
  // Check for incoming messages
  receiveMessage();

  // If serial data is available, send it over LoRa
  if (Serial.available()) {
    String message = Serial.readString();
    sendMessage(message);
  }
}

void sendMessage(String message) {
  LoRa.beginPacket();
  LoRa.write(slaveAddress); // Source address (node)
  LoRa.write(masterAddress); // Destination address (master)
  LoRa.print(message);
  LoRa.endPacket();

  Serial.print("Sending to Master: ");
  Serial.println(message);
}

void receiveMessage() {
  int packetSize = LoRa.parsePacket();
  if (packetSize >= 3) {
    byte senderAddress = LoRa.read(); // Sender's address
    byte receiverAddress = LoRa.read(); // Receiver's address

    if (receiverAddress == slaveAddress) {
      String receivedMessage = "";
      while (LoRa.available()) {
        receivedMessage += (char)LoRa.read();
      }

      Serial.print("Received from Master: ");
      Serial.println(receivedMessage);
    }
  }
}
