# TDT4265-Snow-pole-detection

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




