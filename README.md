# NMEA and Air Surveillance Simulator V2.2

Simulates NMEA and SBS data via TCP/UDP connections.

General hint: Some field are rotated within their valid range; without any kind of validation to corresponding other fields.
This means, numerical they should be all correct; semantically there might be no relationship between them.

## Recommended Requirements

 - Java 8
 - Maven 3.9.10
 
These versions are always recommended; no guarantee, that older versions are compatible.

## Supported sentences

### ADSB

Simulates SBS data: MSG-Messages with subtypes 1, 3 and 4

[Source](http://woodair.net/sbs/Article/Barebones42_Socket_Data.htm)

### AIS

Simulates AIS data: Message types: 1 and 5

[Source](https://www.navcen.uscg.gov/?pageName=AISMessages)

### GPS

Simulates NMEA based `$GPRMC`, `$GPGGA`, `$GPHDT` messages.

[Source 1](http://aprs.gids.nl/nmea/#rmc)
[Source 2](http://aprs.gids.nl/nmea/#gga)
[Source 3](http://aprs.gids.nl/nmea/#hdt)

### RADAR

Simulates NMEA based `$RATTM` messages.

[Source](http://www.nmea.de/nmea0183datensaetze.html#ttm)

### WEATHER

Simulates NMEA based `$WIMDA` messages.

[Source](https://gpsd.gitlab.io/gpsd/NMEA.html#_mda_meteorological_composite)

### COURSE

Simulates NMEA based `$GPHDT` and `$HEHDT` messages.

[Source](https://www.trimble.com/OEM_ReceiverHelp/V4.44/en/NMEA-0183messages_HDT.html)

### DF

Simulates messages for direction finders. This simulator contains a generic data structure for DF, which can be overwritten if a file df.txt is relative to the jar file.
Inside of this file, several variables can be used for replacing values.

| Attribute | Variablename | Replaced by |
| ---      | ---      | ---      |
| DF-Name   | ${name} | value of frequency in scenario.xml |
| Frequency | ${frequency} | value of frequency in scenario.xml |
| Bandwidth | ${bandwidth} | generated in simulator |
| Latitude   | ${lat} | generated in simulator |
| Longitude   | ${lon} | generated in simulator |
| Id   | ${id} | generated in simulator |
| Bearing   | ${bearing} | generated in simulator |
| Time   | ${time} | generated in simulator |

The default format is defined as:

```
<DF><EmitterTyp>Statisch</EmitterTyp><Frequenz>${frequency}</Frequenz><Bandbreite>${bandwidth}</Bandbreite><Peiler><Peilername>${name}</Peilername><Latitude>${lat}</Latitude><Longitude>${lon}</Longitude><Id>${id}</Id><Winkel>${bearing}</Winkel><Winkelabweichung>1.0</Winkelabweichung><Qualitaet>92</Qualitaet><Level>29.9</Level><Levelabweichung>0.7</Levelabweichung><Detektion>${time}</Detektion></Peiler></DF>\r\n
```


## How to build

```shell
mvn clean package
```

## How to start

Either execute the file `start.bat` or use the following command

For standard simulator:
```shell
java -DlogPath="." -Dlogback.configurationFile="logback.xml" -jar simulator.jar [-configFile=config.xml]
```

While starting this program in standard mdoe, the simulator expects the file `config.xml` relative to itself - this can be set to another filename with parameter `-configFile=config.xml`.

For scenario replay:
```shell
java -DlogPath="." -Dlogback.configurationFile="logback.xml" -jar simulator.jar -scenarioFile=scenario.xml
```

### Configuration (for standard simulator)

The content of `config.xml` must have the following structure 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<AppConfig>
   <General>
      <ConfigReloadTime>15</ConfigReloadTime> <!-- in [s] -->
      <ActivateDataLogging>false</ActivateDataLogging>
   </General>
   <AdditionalFeatures>
      <AirplanesOverBalticSea>false</AirplanesOverBalticSea>
      <HighSpeedFighters>true</HighSpeedFighters>
      <PatrolVessels>false</PatrolVessels>
      <FishingVessels>false</FishingVessels>
      <FerryVessels>true</FerryVessels>
   </AdditionalFeatures>
   <DataStreams>
       <Config type="adsb"    sink="tcp" active="true"  ip="" port="10300" nroftrack="1" initTrackId="500" />
       <Config type="ais"     sink="tcp" active="false" ip="" port="10200" nroftrack="1" sleeptime="600000" initTrackId="500" />
       <Config type="radar"   sink="tcp" active="false" ip="" port="10400" radius="20000" />
       <Config type="gps"     sink="udp" active="true"  ip="192.168.2.100" port="10500" />
       <Config type="weather" sink="tcp" active="false" ip="" port="10600" />
       <Config type="course"  sink="tcp" active="false" ip="" port="10700" />
       <Config type="DF"      sink="tcp" active="true"  ip="" port="10800" radius="20000" detection="single" />
   </DataStreams>
</AppConfig>
```

where

 * `type` equals to a stream type. Possible values: `adsb`, `ais`, `gps`, `radar`, `weather`
 * `sink` equals to a sink type. Possible values: `tcp`, `udp`
 * `active` is the flag, to decide, if this stream is active or not. Possible values: `true`, `false` 
 * `ip` is used in case of `sink=udp` and is the target address.
 * `radius` is used in case of radar or df.
 * `port` is the TCP-Socket-Port or in case of `sink=udp` the target port.
 * `nroftracks` equals to the number of generated tracks.
 * `sleeptime` is the length of the time break, when an AIS route is reinitialized again
 * `initTrackId` - Offset of counter of tracking id (e.g. MMSI or HexIdent)
 * `detection` is the mode of detections of DF. Valid values are {single, multiple}. DF will detect one single object (the closest) or multiple ones (within the radius).

### Configuration (for scenario simulator)

A scenario file (xml) must have a well-defined structure. You can find multiple examples  [here](src/main/config/scenarios/scenario.xml).

### Advices:

 * General advices:
   - The simulator checks every `15s`, if the file `config.xml` was modified (based on MD5 hash). In case of changes, the configuration file is reloaded automatically.
   - It is recommended, that the IP-Address should be set with the default IPv4 structure `x.x.x.x`.

 * Advices for `gps`:
   - If `radar` or `weather` are active, `gps` will be automatically activated as well (in this case the `active`-flag of `gps` will be ignored).
   - The variable `nroftrack` will be ignored. GPS sentences correspond just to a single object.

 * Advices for `radar`:
   - The variable `nroftrack` will be ignored.
   - The simulator handles a list of fixed positions of track objects. 
   - RATTM-messages are generated, if the GPS position is close enough to the stored track objects.

 * Advices for `weather`:
   - The variable `nroftrack` will be ignored. 
   - We expect, that produced weather sentences correspond to current GPS position.

* Advices for `df`:
   - The variable `nroftrack` will be ignored.
   - The simulator handles a list of fixed positions of track objects.
   - DF-messages are generated, if the GPS position is close enough to the stored track objects.

## Changelog

2025-06-30: V2.2
- General refactoring of class structure for distinguishing between standard simulator and scenario simulator
- Scenario: Added functionality of replaying scenarios
- Scenario: Refactoring of detecting terminated tracks
- Simulator: Added param `configFile=config.xml` as optional alternativ to default config filename
- Simulator: Removed `<Mode>`-Tag in config file
- Added Course as source
- Fixed: Missing messages for all types in case of UDP
- ADSB: Add additional vessels: Highspeed Fighters
- AIS: Set realistic navigational status values of vessels.
- Weather: Fixed bug of wrong generation time ranges of weather messages
- Radar: Set variable values for speed and course
- DF: External file for specific data structure
- DF: Two type of DFs: One following the own vessel and secondly (in scenario mode) constant dfs.
- Update of dependencies: Lombok 1.18.38, Maven-Shade-Plugin 3.5.3, Maven-Site-Plugin 3.21.0, Maven-Compiler-Plugin 3.14.0, Maven-shade-plugin 3.6.0, Slf4J 2.0.17, Junit 5.12.2


2024-10-19: V2.1

- Added DF as source
- Added Feature of ActivateDataLogging
- Fixed: Missing xml tag <Mode> should make a clean shutdown
- ADSB: Add dynamic calculated flight height
- AIS: Set value to destination field at additional ships
- AIS: Add additional vessels: Ferry vessels
- Recommended version of Maven is now 3.9.9
- Update of dependencies: Logback to 1.3.14, Slf4J to 2.0.16, Junit 5.10.3, Lombok 1.18.34, Maven-Compiler-Plugin 3.13.0, Jaxb 2.3.3, Maven-Site-Plugin 3.20.0, Commons-Lang3 3.17.0, Maven-Jar-Plugin 3.4.2

2024-02-28 : V2.0.1

- ADSB/AIS: Fix of duplicated identifying tokens

2023-11-26 : V2.0.0

- Reduce memory usage while having 500+ objects
- Refactoring of track building
- Usage of optimized garbage collection
- AIS: Usage of a more complex sea graph
- AIS: More variability at ship creations (names, dimensions, speed, callsigns, draughts, number of published messages)
- AIS: Bugfix at Astar-Algorithm to avoid duplicate waypoints
- Recommended version of Maven is now 3.8.7
- Update of dependencies: JUnit to 5.10.1, Logback to 1.3.6, maven-jar-plugin to 3.3.0, mvn-compiler-plugin to 3.11.0, Slf4J to 2.0.6