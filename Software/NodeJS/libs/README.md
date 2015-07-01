GrovePi for Node.js
=======

GrovePi is an open source platform for connecting Grove Sensors to the Raspberry Pi.

These files have been made available online through a [Creative Commons Attribution-ShareAlike 3.0](http://creativecommons.org/licenses/by-sa/3.0/) license.

## Quick start

Before to start you should install Node.js on your RaspberryPi and clone the repo on your local environment.
Be sure to have npm installed and then you can proceed installing the package.

Go inside your Node.js application folder and type
```bash
$ npm install node-grovepi
```

Now you can include the module inside your application:
```javascript
var GrovePi = require('node-grovepi').GrovePi
```

At this point you may need to include the GrovePi base classes:
```javascript
var Commands = GrovePi.commands
var Board = GrovePi.board
```

If the sensor/component you need to use already has the classes then you can include them:
```javascript
var AccelerationI2cSensor = GrovePi.sensors.AccelerationI2C
var UltrasonicDigitalSensor = GrovePi.sensors.UltrasonicDigital
var AirQualityAnalogSensor = GrovePi.sensors.AirQualityAnalog
var DHTDigitalSensor = GrovePi.sensors.DHTDigital
var LightAnalogSensor = GrovePi.sensors.LightAnalog
```

Now you can instanciate the GrovePi and your sensors/components, for example:
```javascript
var board = new Board({
    debug: true,
    onError: function(err) {
      console.log('Something wrong just happened')
      console.log(err)
    },
    onInit: function(res) {
      if (res) {
        console.log('GrovePi Version :: ' + board.version())

        var lightSensor = new LightAnalogSensor(2)
        console.log('Light Analog Sensor (start watch)')
        lightSensor.on('change', function(res) {
          console.log('Light onChange value=' + res)
        })
        lightSensor.watch()
      }
    }
  })
```

If there is no class for your sensors or components then you can write your own functions for them:
_Note: every custom function must be called only after the Board init._
```javascript
function customAccelerationReading() {
  var write = board.writeBytes(Commands.acc_xyz.concat([Commands.unused, Commands.unused, Commands.unused]))
  if (write) {
    board.wait(100)
    board.readByte()
    var bytes = board.readBytes()
    if (bytes instanceof Buffer) {
      var x = bytes[1] > 32 ? -(bytes[1]-224) : bytes[1]
      var y = bytes[2] > 32 ? -(bytes[2]-224) : bytes[2]
      var z = bytes[3] > 32 ? -(bytes[3]-224) : bytes[3]
      return [x, y, z]
    } else {
      return false
    }
  } else {
    return false
  }
}
```

When you are ready to go you should call the init method
```javascript
board.init()
```

Each sensor/component has 3 methods to get access to the data:
- **read()** - Read data from the sensor/component
- **stream(delay, callback)** - Start a stream with the sensor/component, each N milliseconds (delay) sends data to the callback. You can use stopStream() to close the connection.
- **watch(delay)** - Start a polling routine which will fire a "change" event only when there are new data coming from the sensor/component. The internal timer will use the given delay value or 100 milliseconds as default. You can use stopWatch() to stop the polling.

And 1 method to write data:
- **write(value)** - Write a value on the sensor/component

You'll find a more complex example in the "basicTest.js" file under the "tests" folder.