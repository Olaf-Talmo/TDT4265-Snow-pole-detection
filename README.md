# TDT4265-Snow-pole-detection

### We modified the data.yaml file.
Modified file

train: images/train
val: images/valid
test: images/test

nc: 1
names: ['pole']

### Hva skal modellene trenes og testes på

Virker som det er en innlevering for kun RGB, og en innlevering for kun LIDAR. Altså ikke relevant å lage en kombinert modell

### Domenekunnskap: Hva vet vi om brøtestikkene

- De er alltid på siden av veien. 
- De er stort sett plassert ut med jevne mellomrom
- Antar at bilen kjører fremover: Størrelse øker, og posisjonen går som regel lenger ned og ut til siden. (unntak ved  sving).
- Store brøytestikker er ofte lavt og langt til siden på bildet. Små er ofte sentrert og langt oppe.
- De er alltid lange og tynne, små variasjoner i form. 
- De har stort sett samme farger, rød eller bambus med noe refleks.


### Oppgaver / Ideer

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

### Runs

- train3_normal: small model with 250 epochs. Normal yolo. Basic yolo trained on this dataset
- modified_loss_olaf_1: Loss function changed to penalize deviations from 0.25 aspect ratio in loss.py. Also, changed loss.py so that all boxes would be penalized, not only the ones making positive predictions. Tried linear, quadratic and qubic penalization with weighting up to 1000. Neglictable difference for all instances. This seems a bit weird to me, and I want to explore modifying the cost function further. I am not confidemt that the changes in the code had the appropriate effect, so debugging and visualizing/printing intermediate results may be a good idea. Perhaps more epochs would make it better as well.
- modified_augmentation_olaf. Endret box_candidates() i augmentation.py slik at bounding boxes som har aspect ratio > 0.25 rejectes. Sligthly more long and thin boxes, but not a lot. And worse performance overall. This makes sense, the effect of this is that the model gets worse at recognizing poles in augmented pictures, because the poles may be strecthed to be quadratic and the algorithm is not allowed to predict quadratic boxes. This is the cost we have to pay to avoid teaching the model that quadratic boxes is ok. In this instance, the tradeoff was clearly not worth it.

train_lidar_1 - første trening med liten endring i loss funksjon




