<style>
    * {
        background-color: #352e2e;
        font-family: monospace;
        color: rgb(60, 255, 1);
        padding: 0px;
    }

    button,
    datalist {
        background-color: rgb(85, 85, 85);
    }

    input[type=text] {
        color: rgb(179, 255, 179);
        background-color: rgb(102, 86, 86);
        border: 1px solid;
        border-color: #696 #363 #363 #696;
    }

    [type="checkbox"] {
        vertical-align: middle;
    }

    #serialResults {
        font-family: monospace;
        white-space: pre;
        height: calc(100% - 120px);
        width: calc(100% - 20px);
        border-style: solid;
        overflow: scroll;
        background-color: rgb(88, 92, 92);
        padding: 10px;
        margin: 0px;
    }
</style>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Fastest Serial terminal in your browser for Chrome.</title>
<meta name="Description" content="Set your baud speed and hit connect.
 A serial terminal that runs with out any plugins in chrome.">
<button class="active" onclick="location.href='https://sites.google.com/view/tecnovenzscanneri2c/inicio'">Inicio &#128435</button>
<button onclick="connectSerial()">Connect</button>
Baud:
<input type="text" id="baud" list="baudList" style="width: 10ch;" onclick="this.value = ''"
    onchange="localStorage.baud = this.value">
<datalist id="baudList">
    <option value="9600">9600</option>
    <option value="14400">14400</option>
    <option value="19200">19200</option>
    <option value="38400">38400</option>
    <option value="57600">57600</option>
    <option value="115200">115200</option>
</datalist>
<button onclick="serialResultsDiv.innerHTML = '';">Clear</button>
<input type="button" value="Monitor Serial" style="font-size: 20px; padding: 3px 300px; border: 2px solid #FFFF00;">
                 
<br>
<input type="text" id="lineToSend" style="width:calc(100% - 165px)">
<button onclick="sendSerialLine()" style="width:45px">Send</button>

<br>

<label for="carriageReturn">
    <input type="checkbox" id="carriageReturn" onclick="localStorage.carriageReturn = this.checked;" checked>send with
    /r
</label>

<label for="addLine">
    <input type="checkbox" id="addLine" onclick="localStorage.addLine = this.checked;" checked>send with /n
</label>

<label for="echoOn">
    <input type="checkbox" id="echoOn" onclick="localStorage.echoOn = this.checked;" checked>echo
</label>


<br>
<div id="serialResults">
</div>
<script>
    var port, textEncoder, writableStreamClosed, writer, historyIndex = -1;
    const lineHistory = [];
    async function connectSerial() {
        try {
            // Prompt user to select any serial port.
            port = await navigator.serial.requestPort();
            await port.open({ baudRate: document.getElementById("baud").value });
            let settings = {};

            if (localStorage.dtrOn == "true") settings.dataTerminalReady = true;
            if (localStorage.rtsOn == "true") settings.requestToSend = true;
            if (Object.keys(settings).length > 0) await port.setSignals(settings);
  
            
            textEncoder = new TextEncoderStream();
            writableStreamClosed = textEncoder.readable.pipeTo(port.writable);
            writer = textEncoder.writable.getWriter();
            await listenToPort();
        } catch (e){
            alert("Serial Connection Failed" + e);
        }
    }
    async function sendCharacterNumber() {
        document.getElementById("lineToSend").value = String.fromCharCode(document.getElementById("lineToSend").value);
    }
    async function sendSerialLine() {
        dataToSend = document.getElementById("lineToSend").value;
        lineHistory.unshift(dataToSend);
        historyIndex = -1; // No history entry selected
        if (document.getElementById("carriageReturn").checked == true) dataToSend = dataToSend + "\r";
        if (document.getElementById("addLine").checked == true) dataToSend = dataToSend + "\n";
        if (document.getElementById("echoOn").checked == true) appendToTerminal("> " + dataToSend);
        await writer.write(dataToSend);
        if (dataToSend.trim().startsWith('\x03')) echo(false);
        document.getElementById("lineToSend").value = "";
        document.getElementById("lineToSend").value = "";
    }
    async function listenToPort() {
        const textDecoder = new TextDecoderStream();
        const readableStreamClosed = port.readable.pipeTo(textDecoder.writable);
        const reader = textDecoder.readable.getReader();

        // Listen to data coming from the serial device.
        while (true) {
            const { value, done } = await reader.read();
            if (done) {
                // Allow the serial port to be closed later.
                console.log('[readLoop] DONE', done);
                reader.releaseLock();
                break;
            }
            // value is a string.
            appendToTerminal(value);
        }
    }
    const serialResultsDiv = document.getElementById("serialResults");
    async function appendToTerminal(newStuff) {
        serialResultsDiv.innerHTML += newStuff;
        if (serialResultsDiv.innerHTML.length > 3000) serialResultsDiv.innerHTML = serialResultsDiv.innerHTML.slice(serialResultsDiv.innerHTML.length - 3000);

        //scroll down to bottom of div
        serialResultsDiv.scrollTop = serialResultsDiv.scrollHeight;
    }
    function scrollHistory(direction) {
        // Clamp the value between -1 and history length
        historyIndex = Math.max(Math.min(historyIndex + direction, lineHistory.length - 1), -1);
        if (historyIndex >= 0) {
            document.getElementById("lineToSend").value = lineHistory[historyIndex];
        } else {
            document.getElementById("lineToSend").value = "";
        }
    }
    document.getElementById("lineToSend").addEventListener("keyup", async function (event) {
        if (event.keyCode === 13) {
            sendSerialLine();
        } else if (event.keyCode === 38) { // Key up
            scrollHistory(1);
        } else if (event.keyCode === 40) { // Key down
            scrollHistory(-1);
        }
    })
    document.getElementById("baud").value = (localStorage.baud == undefined ? 9600 : localStorage.baud);
    document.getElementById("addLine").checked = (localStorage.addLine == "false" ? false : true);
    document.getElementById("carriageReturn").checked = (localStorage.carriageReturn == "false" ? false : true);
    document.getElementById("echoOn").checked = (localStorage.echoOn == "false" ? false : true);
</script>
<br> © 2024 TECNOVENZ SCANNER-I2C <a
    href="https://sites.google.com/view/tecnovenzscanneri2c/inicio">https://sites.google.com/view/tecnovenzscanneri2c/inicio</a>
