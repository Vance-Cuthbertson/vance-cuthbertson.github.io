# vance-cuthbertson.github.io

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Feather BLE Controller</title>
</head>
<body>
  <h1>Feather BLE Controller</h1>
  <button id="connect">Connect to BLE</button>
  <button id="ledOn">Turn LED ON</button>
  <button id="ledOff">Turn LED OFF</button>
  <p id="status">Status: Not Connected</p>

  <script>
    let device;
    let server;
    let uartService;
    let txCharacteristic;
    let rxCharacteristic;
    let encoder = new TextEncoder();

    async function connectBLE() {
      try {
        device = await navigator.bluetooth.requestDevice({
          acceptAllDevices: true,
          optionalServices: ['6e400001-b5a3-f393-e0a9-e50e24dcca9e'] // UART Service
        });

        server = await device.gatt.connect();
        uartService = await server.getPrimaryService('6e400001-b5a3-f393-e0a9-e50e24dcca9e');
        
        txCharacteristic = await uartService.getCharacteristic('6e400002-b5a3-f393-e0a9-e50e24dcca9e'); // TX (Send Data)
        rxCharacteristic = await uartService.getCharacteristic('6e400003-b5a3-f393-e0a9-e50e24dcca9e'); // RX (Receive Data)

        rxCharacteristic.addEventListener('characteristicvaluechanged', (event) => {
          let value = new TextDecoder().decode(event.target.value);
          document.getElementById('status').textContent = "Received: " + value;
        });

        await rxCharacteristic.startNotifications();

        document.getElementById('status').textContent = "Connected!";
      } catch (error) {
        console.error(error);
        document.getElementById('status').textContent = "Error: " + error.message;
      }
    }

    async function sendCommand(command) {
      if (!txCharacteristic) {
        alert("Not connected!");
        return;
      }
      await txCharacteristic.writeValue(encoder.encode(command));
    }

    document.getElementById('connect').addEventListener('click', connectBLE);
    document.getElementById('ledOn').addEventListener('click', () => sendCommand('1'));
    document.getElementById('ledOff').addEventListener('click', () => sendCommand('0'));
  </script>
</body>
</html>
