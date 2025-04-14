# TDT4265-Snow-pole-detection

### We modified the data.yaml file.
Modified file

------------------- 1:

train: images/train
val: images/valid
test: images/test

nc: 1
names: ['pole']

------------------- 2:

train: home/omtalmo/Olaf_TTK4265//Poles/rgb/images/train
val: home/omtalmo/Olaf_TTK4265//Poles/rgb/images/valid
test: home/omtalmo/Olaf_TTK4265//Poles/rgb/images/test

nc: 1
names: ['pole']

### Hva skal modellene trenes og testes på

Virker som det er en innlevering for kun RGB, og en innlevering for kun LIDAR. Altså ikke relevant å lage en kombinert modell

### Domenekunnskap: Hva vet vi om brøtestikkene

- De er alltid på siden av veien. 
- De er alltid på nedre halvdel av bildet (hvertfall nedre 75%)
- De er stort sett plassert ut med jevne mellomrom
- Antar at bilen kjører fremover: Størrelse øker, og posisjonen går som regel lenger ned og ut til siden. (unntak ved  sving).
- Store brøytestikker er ofte lavt og langt til siden på bildet. Små er ofte sentrert og langt oppe.
- De er alltid lange og tynne, små variasjoner i form. 
- De har stort sett samme farger, rød eller bambus med noe refleks.


### Oppgaver / Ideer

- Fjern øverste halvdel av bildet. Brøtestikkene er jo aldri der. Og hvis de er det så er de så små at de ikke oppdages uansett.
Bør være en god løsning dersom vi kan anta at kamerates posisjon og vinkel ikke endrer seg. Det er vel som regel tilfelle i real life? Da er beskjæring en del av å kalibrere modellen til kamera setuppet. Burde analysere y-pos til brøtestikkene for å avgjøre hvor vi skal beskjære.
Resultat: Høyde-analyse i label_anlysis.ipynb. Det er ingen brøtestikker innenfor øvre 40% av bildet. --> Vi cropper de øverste 40%, både på trening og inference. All data som kan fjernes uten av vi fjerner relevant informasjon må jo være bra. 
Egentlig bør dette kunne implementers dynamisk uten tilgang på ground truth, fordi i dette tilfellet er det ikke risiko for å plutselig predikere en brøtestick helt øverst i bildet.
- Se mer på pre-prossessering av bilder. Kan være mer sånne ting ^. Øke kontrast bør funke? Legge på fargefilter så rødt blir ekstra tydelig? Alle sånne ting er gode løsninger dersom de kan generaliseres, i.e. ikke bare gjelder disse bildene. F.eks, si at 50% av brøytestikker er røde. Da kan et rødfilter være fornuftig, selv om modellen fortsatt bør kunne håndtere brune og svarte stikker.
- Lage syntetisk data
- Fortsette med brøytestick analyse. Lag statistikk plots som viser forhold mellom høyde på brøytestick og y-posisjon. Samme med x-posisjon, men i stedet for absolutt tar jeg her avstand fra midten. 
- Implementere annen arkitektur, f.eks yolov5, som bruker anchor-boxes og definerer str og pos eksplisitt. Kanskje cdd og.
- Debugge loss funksjon endringer. Prøve CARLosses: https://pdfs.semanticscholar.org/3ed9/298851a85e6ee2d9568266fc8c64fcc4ebf3.pdf?_gl=1*s2zpg1*_gcl_au*NzIxMjEwMzY3LjE3NDQ0NTAzOTA.*_ga*MTc0NzAxNjM3MS4xNzQ0NDUwMzky*_ga_H7P4ZT52H5*MTc0NDQ1MDM5MS4xLjEuMTc0NDQ1MDYzNC40OC4wLjA.

### Modifisert infernece: Hvrofor det er uaktuelt

Ide: Fjerne boxer så tidlig som mulig i inference, før sansynlighet og klassifisering. Håpet at det kunne gjøre inferencen raskere. (ikke bedre. ) 

Konklusjon: Dette er ikke en god løsning for yolo eller andre one-stage arkitekturer. Boksenes posisjon og sansynlighet regnes ut parallellt. Det er derfor umulig å forkaste bokser basert på str og dim før sansynlighet og klassifikasjon regnes ut. Å forkaste boksene etter at sansynlighet regnes ut er ikke hensiktsmessig, fordi alle bokser med feil aspect ratio allerede har svært lav sansynlighet, og det er ikke raskere å forkaste basert på asect ratio enn basert på sansynlighet. Å forkaste basert på aspect ratio er kun aktuelt dersom algoritmen feilaktig setter høy sansynlighet på bokser med feil aspect ratio, men det skjer aldri. 

Dersom modifisert inference skal implementeres må det gjøres med two-stage arkitekturer, som f.eks faster r-cnn. Der regnes boksenens pos og dim ut førsy, og deretter regnes klassifisering og sansynlighet ut for hver boks. Det er derfor mulig å forkaste bokser før sansynlighet og klassifissering.
Spørsmålet blir da: Er dette tilstrekkelig, vil det gjøre arkitekturene raske nok for real-time prossessing?
Først. Hvor raske er de by defoult: Faster R-CNN: På dyre GPU-er som Nvidia RTX: 5-10 fps. På vanlige pc-er med integrert gpu: 1 fps. Stage 1, altså CNN-et, feature mappet, backbone, krever mest regnekraft, si 60-70% (Opptil 90% for faster r-cnn, ifølge white paperet). Stage 2, proposal og classification, krever 30-40% (10%). Dersom vi kan forkastet 80% av boksene med neglisjerbar overhead (tilleggs-utregninger), som er sansynlig, kan vi potensielt gjøre algoritmen 40% * 80% = 32% (8%) raskere.
Det er ikke i nærheten av nok. 
Er det mulig å gjøre backbone raskere? Nei, det er uavhengig av domenekunnskapen, og uavhengig av hvor mange bokser som lages. Man må analysere bildet like mye uavhenegig av hvor mange bokser man skal teste. 
Konklusjon: Two stage - detectors er uaktuelt. Modifisert inference er uaktuelt. 

Derfor er ikke benchamarking.ipynb så aktuelt lenger, så har ikke laget en yolov5 versjon.

## Runs

### YOLO v12

- train3_normal: small model with 250 epochs. Normal yolo. Basic yolo trained on this dataset
- modified_loss_olaf_1: Loss function changed to penalize deviations from 0.25 aspect ratio in loss.py. Also, changed loss.py so that all boxes would be penalized, not only the ones making positive predictions. Tried linear, quadratic and qubic penalization with weighting up to 1000. Neglictable difference for all instances. This seems a bit weird to me, I want to explore modifying the cost function further. I am not confidemt that the changes in the code had the appropriate effect, so debugging and visualizing/printing intermediate results may be a good idea. Perhaps more epochs would make it better as well.
- modified_augmentation_olaf. Endret box_candidates() i augmentation.py slik at bounding boxes som har aspect ratio > 0.25 rejectes. Sligthly more long and thin boxes, but not a lot. And worse performance overall. This makes sense, the effect of this is that the model gets worse at recognizing poles in augmented pictures, because the poles may be strecthed to be quadratic and the algorithm is not allowed to predict quadratic boxes. This is the cost we have to pay to avoid teaching the model that quadratic boxes is ok. In this instance, the tradeoff was clearly not worth it.

train_lidar_1 - første trening med liten endring i loss funksjon

### Yolo v5



### Yolov5 Autoanchors returned these anchors:

3,26 (width=3, height=26) → aspect ratio ≈ 1:8.7
4,30 (width=4, height=30) → aspect ratio ≈ 1:7.5
3,42 (width=3, height=42) → aspect ratio ≈ 1:14
5,40 (width=5, height=40) → aspect ratio ≈ 1:8
5,67 (width=5, height=67) → aspect ratio ≈ 1:13.4
8,78 (width=8, height=78) → aspect ratio ≈ 1:9.8
21,73 (width=21, height=73) → aspect ratio ≈ 1:3.5
16,140 (width=16, height=140) → aspect ratio ≈ 1:8.8
41,129 (width=41, height=129) → aspect ratio ≈ 1:3.1


Konklusjon: Ser bra ut, ingen grunn til å hardkode dem. 


