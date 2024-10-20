# appliedXeto_skyposium2024

Introduction
---

This repo consists of *most* of the raw materials from my presentation at SkyPosium 2024: Applied Xeto. The intent of the presentation was to provide a practical look at using some of the features of Xeto available today within SkySpark.

The ppt presentation is presented here, but will also be available when SkyFoundry posts all presentations and recordings from this year's SkyPosium. If I remember, I'll update this readme with a link then!

Feel free to use or extend this code as you see fit or reach out to me via email at mmelillo@albireoenergy.com with any questions or if you'd like to collaborate on extending this library. This repo in itself will remain as-is, but I'm open to the idea of a community project around speeding up the adoption of Xeto.

**Note, this work was done with SkySpark rev 3.1.10, newer versions & updated versions of base xeto libs may cause conflicts**

Utilizing the code in this repo
---

The `funcs.trio` file consists of all functions I used during the presentation. You would be able to use any of these within a SkySpark (or Haxall) environment. 

The setup within SkySpark involves the following steps (this would require having one of the make/model sensors I've created specs for in the `mm.elsys` lib):

- Project uiMeta: {uiMeta, floorNav: "siteRef", equipNav: "floorRef"}
- Add the following libs under `src/xeto/`:
  - `mm.elsys, mm.pxeto, mm.ttnmqtt`
  - You may need to restart after adding libraries
  - Enable libriaries in your project under `Settings|Using`
  - Run `xetoReload()` in shell
- In `Builder` create a Site & Floor to contain imported devices & points
- Create an MQTT Connector for incoming traffic
- Create a Task to ingest incoming MQTT payloads
  - `connRef: your mqtt conn`
  - `observes: obsMqtt`
  - `mqttTopic: #`
  - `taskExpr: (obs) => mqtt_onboardTTN(obs) end`

Absent those sensors, you can use the function `xeto_samplePayload()` to send a sample payload I saved into that function. This should auto-create an `elt-2` sensor device with associated points with the topic `testing`. I've slashed out several bits of identifying info for my personal broker instance here, but the payload will still consume as valid JSON.

If you want to blow up builder and start over, there is a utility function `clearBuilder()` which can be used to remove all points, equips, spaces, floors, and sites. This is handy if you want to modify your spec libraries, quickly clear the project, and test a follow up import.

Spec libraries
---

During the presentation demo, the `Zone Light Sensor` did not correctly pull a `curVal`. I believe this is because the spec lib `mm.pxeto` which creates a spec for a `LightPoint` and `LightSensor` does not define a specific unit. So while `NumberPoint` causes a `unit` tag to be added to the instantiated record, there is no valid value for this tag, and SkySpark chokes when trying to commit a curVal to a rec where effectively `unit: ""`. Typically in SkySpark, a unitless value would simply not have the `unit` tag, or a custom unit could be assigned using `unit: "_yourUnitHere"`.

These specs are presented below. I've left the libs in this repo as they were for the presentation in case you want to mess around with modifying these values yourself.

```
LightPoint : NumberPoint <abstract> {
  light
  point
  //add unit: "lm" here
}

LightSensor: LightPoint {
  sensor
}
```

Control flow
---

- Incoming MQTT Message gets picked up by `obsMqtt` task and calls `mqtt_onboardTTN`
- Onboarder function determines if a new device needs to be instantiated
- After instantiating (or finding the existing device) `mqtt_parseTTN` is called to ingest historical data

Small disclaimer
---

This code is provided as-is, see LICENSE for further details. It was purpose built for my home test-bench while I was initially learning how to use specs, and my hope is that the both the presentation and the source code will help others begin to experiment and incorporate Xeto into their toolkit.

There are several hard-coded bits (like how the device is committed ref'd to a site & space), that I plan to revisit before incorporating this in any production environment. This will probably involve looking at Things Network codecs and investigating adding meta data on that server that can be transferred via the MQTT Broker. There is similar work we are doing on our (Albireo Energy's) hosted services for both SkySpark and Niagara to leverage LoRaWAN & MQTT to get observability in hard to manage places (both physical and network).


Other links
---
[SkyFoundry](https://skyfoundry.com/)

[Haxall](https://haxall.io/)

[Project-Haystack](https://project-haystack.org/)

[Xeto Github](https://github.com/Project-Haystack/xeto)

If you're interested to learn more about the work we do, visit [Albireo Energy](https://www.albireoenergy.com/) or reach out to me.


