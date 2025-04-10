# TDT4265-Snow-pole-detection

### We modified the data.yaml file.
Modified file

train: images/train
val: images/valid
test: images/test

nc: 1
names: ['pole']

### v1: yolov12
- train3: small model 250 epochs
- v2

### Hva skal modellene trenes og testes på

Virker som det er en innlevering for kun RGB, og en innlevering for kun LIDAR. Altså ikke relevant å lage en kombinert modell

### Hva vet vi om brøtestikkene

- De er alltid på siden av veien. 
- De er stort sett plassert ut med jevne mellomrom
- Antar at bilen kjører fremover: Størrelse øker, og posisjonen går som regel lenger ned og ut til siden. (unntak ved  sving).
- Store brøytestikker er ofte lavt og langt til siden på bildet. Små er ofte sentrert og langt oppe.
- De er alltid lange og tynne, små variasjoner i form. 
- De har stort sett samme farger, rød eller bambus med noe refleks.

### Hvordan endre anchor boxes

Størrelse, dimensjon og plassering av anchor boksene er ikke hardkodet. De defineres dynamisk basert på et featuremap. 
Head.py: feats, shapes = self._get_encoder_input(x)

### Ideer
- Usikker på om det faktisk vil hjelpe modellen å enforce små bounding boxes, eller om det er noe modellen lærer seg etterhvert uansett.
Derfor vil jeg prøve å visualisere, i.e. plotte, alle bounding boxene, også de som er rejected. Dersom nesten alle de som er rejected har riktig form str, er det ikke behov for å gjøre noe. 

train_lidar_1 - første trening med liten endring i loss funksjon




