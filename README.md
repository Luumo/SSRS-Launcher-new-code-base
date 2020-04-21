
# SSRS Launcher

## Syfte:
Syftet med arbetet är att separera backend från frontend, för att köra koden mer lättläslig, enklare att felsöka och använda. 

## Mål:
* Separera styrfunktioner och frontend
* Mer lättläslig kod (funktionsnamn, kommentarer, kommentarer på parameterar)
* lägga till errorhantering
* Möjlighet att använda styrfunktionerna via remote access, samt via HTTP (som det är för tillfället).

## Tidigare struktur:
* Flask framework och alla styrfunktioner för drönarlaunchern är sammansatta.
* All errorhantering är baserat på encoder_ready och http return-meddelanden
* Många funktioner som gör samma sak . 
  T.ex: function_pitch_up() och function_pitch_down().
  
## Ny struktur:
* Styrfunktioner för launchern är separerade från Flask-koden.
* Flask kan istället anropa styrfunktionerna.
* många funktioner är sammansatta och använder parametrar istället. T ex pitch_control(cmd) som tar in ett kommando, istället för 4st olika funktioner, för 4 olika kommandon som det var i tidgare program.

## Fördelar:
* Enklare att bygga vidare och ändra koden
* Bättre möjlighet att unit-testa koden
* Möjlighet att använda koden med flera handlers (remote, lokalt, http)

## Att tänka på
* Inga hårdkodningar i styrfunktionerna.
* Om en variabel ska ändras, skriv en funktion som utför arbetet. Det blir mer lättläsligt och enklare att förstå vad som händer.
* kommentera funktioner, argument och returns.
* Använd styrfunktionerna för att reglera motorerna och launchern.

* Separera styrfunktioner från handlern (t ex flask). 

## TODO:
* Jämför detta program med ursprungliga launcher-programmet och skriv klart de funktioner som är kvar. (Alla funktioner behövs ej)
* Testa varje enskild funktion, så de fungerar som tänkt.
* Se över remote-handlern från Airpelago. Den kan behövas skrivas om också.

# Översikt av funktioner


```python
launcher = Launcher()

# Manuella Funktioner, används för att manuellt kontrollera pitch, rotation, lift och launch
# cmd för pitch kan till exempel vara: PitchCMD.up, PitchCMD.down eller PitchCMD.stop
pitch_control(cmd)
rotation_control(cmd)
lift_control(cmd)
launch_control(cmd)

# Rörelse Funktioner. Parametern motsvarar det värde som pitch,rotation,lift,launch ska ställas in till.
launcher.set_pitch_position(Pitch)
launcher.set_rotation_position(Rotation)
launcher.set_lift_position(Lift)
launcher.set_launch_position(Launch)

#Auto funktioner
launcher.set_launch_variables(pitch_position, rotation_position, lift_position) # sparar ner de angivna launch-variablerna
launcher_prepare_launch() # förbereder launchern enligt angivna parametrar ovan
launcher.launch() # avfyrar drönaren
launcher.standby() # återgår till startläge

# Övriga funktioner:

launcher.stop() # Stoppar alla motorer.
launcher.encoder_ready_check() # kollar om encodern är redo. Detta är mer lättlässligt än att kontrollera en specifik variabel i koden istället
launcher.reset_encoders() # sätter self.encoder_ready = 1, alltså att den är redo.
launcher.battery_voltage() # returnerar spänningen på batteriet som en float. Denna struktur kan användas som mall för övrig telemtri-data

```

# Case 1: Manuell styrning  av motorerna
Användaren vill manuellt reglera pitch och rotation

* Notera att motorerna sätts i rörelse så fort funktionerna exekveras.


```python
# användaren vill testa pitch-motorns "up"-rörelse
launch.pitch_control(PitchCMD.up) # motorn rör på sig

# användaren vill stoppa rörelsen:
launch.pitch_control(PitchCMD.stop) # motorn stannar

# användaren vill testa rotationsmotorn
launch.rotation_control(RotationCMD.right) # roterar höger
launch.rotation_control(RotationCMD.left) # roterar vänster

launch.rotation_control(RotationCMD.stop) # rotationen stannar

```

# Case 2: Styrning med specifika värden
Användaren vill ställa in pitch på 30 grader, och rotation till 70 grader
* Notera att mototerna sätts i rörelse direkt när funktionerna exekveras. 



```python
launcher.set_pitch_position(30) # pitch-motorerna ställer in sig till 30 grader
launcher.set_rotation_position(70) # rotationsmotoroerna ställer in sig på 70 grader

```

# Case 3: Förberedning och uppskjutning
* Användaren vill förbereda launchern för uppskutnnig.
* Användaren vill ställa in pitch, rotation och lift position.
* Användaren vill INTE att rörelsen ska exekveras, enbart FÖRBEREDA launchern innan uppskutning.
* Därefter, när launch() kallas, så skjuts drönaren upp


```python

# Steg 1: Ställ in variablerna:
launcher.set_launch_variables(30, 70, 100) # pitch 30 grader, rotation 70 grader, lift 100cm

# Steg 2: Ställ in launchern på följande värden
launcher.prepare_launch() # ställer in launchern enligt de satta variablerna i steg 1 (motorerna sätts i rörelse här)

#steg 3: avfyra launchern
launcher.launch() # avfyrar drönaren från launchern
```
