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

train: /home/omtalmo/Olaf_TTK4265//Poles/rgb/images/train
val: /home/omtalmo/Olaf_TTK4265//Poles/rgb/images/valid
test: /home/omtalmo/Olaf_TTK4265//Poles/rgb/images/test

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

- Endre inference funksjonene så de kun lagrer de to førte bildene. Alle bildene tar for mye lagringsplass. 
- Kjøre alt på nytt med bedre oppløsning på inference bilder, kanksje 1536 isteden for 640. Bør ha myyye å si. Bruke benchmarking til å se hvor mye tid det koster oss. Hvordan bør vi bruke tiden, på større modell eller bedre oppløsning?
- Se mer på pre-prossessering av bilder. Øke kontrast bør funke? Legge på fargefilter så rødt blir ekstra tydelig? Alle sånne ting er gode løsninger dersom de kan generaliseres, i.e. ikke bare gjelder disse bildene. F.eks, si at 50% av brøytestikker er røde. Da kan et rødfilter være fornuftig, selv om modellen fortsatt bør kunne håndtere brune og svarte stikker.
- Lage syntetisk data. Utforske cut and paste method
- Implementere CDD
- Debugge loss funksjon endringer. Prøve CARLosses: https://pdfs.semanticscholar.org/3ed9/298851a85e6ee2d9568266fc8c64fcc4ebf3.pdf?_gl=1*s2zpg1*_gcl_au*NzIxMjEwMzY3LjE3NDQ0NTAzOTA.*_ga*MTc0NzAxNjM3MS4xNzQ0NDUwMzky*_ga_H7P4ZT52H5*MTc0NDQ1MDM5MS4xLjEuMTc0NDQ1MDYzNC40OC4wLjA.
- Implementere dynamisk confidence treshold for inference, i.e. kalibrere inference slik at sjansen for at en deteksjon er en falsk positiv er 5%, altså standard. Nå er det så store variasjoner i hvor confidente de ulike modellene er, at det er vanskelig å sammenligne dem. 

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

### Observasjoner
- v5 er mer confident enn v12. 
- v12 predikerer mye smalere bokser fra start av.
- Cropping gjør modellene dårligere (må være fordi himmelen er kontekst eller noe?)
- Å øke oppløsning på treningsbildene 640 --> 1536 gjør ikke nødvendigvis modellen bedre på 640 test-bilder. Blir god på det man trener på. 


### YOLO v12

- train3_normal: small model with 250 epochs. Normal yolo. Basic yolo trained on this dataset
- modified_loss_olaf_1: Loss function changed to penalize deviations from 0.25 aspect ratio in loss.py. Also, changed loss.py so that all boxes would be penalized, not only the ones making positive predictions. Tried linear, quadratic and qubic penalization with weighting up to 1000. Neglictable difference for all instances. This seems a bit weird to me, I want to explore modifying the cost function further. I am not confidemt that the changes in the code had the appropriate effect, so debugging and visualizing/printing intermediate results may be a good idea. Perhaps more epochs would make it better as well.
- modified_augmentation_olaf. Endret box_candidates() i augmentation.py slik at bounding boxes som har aspect ratio > 0.25 rejectes. Sligthly more long and thin boxes, but not a lot. And worse performance overall. This makes sense, the effect of this is that the model gets worse at recognizing poles in augmented pictures, because the poles may be strecthed to be quadratic and the algorithm is not allowed to predict quadratic boxes. This is the cost we have to pay to avoid teaching the model that quadratic boxes is ok. In this instance, the tradeoff was clearly not worth it.

train_lidar_1 - første trening med liten endring i loss funksjon

- Plain small model
- Plain medium model
- Plain large model

(Continue with best)

- Cropped images
- Loss function
- Augmentation

### Yolo v5

- Plain small model
- Plain medium model
- Plain large model

(Continue with best)

- Cropped images
- Different img hyperparameter

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

NB: Fikk: "WARNING: Extremely small objects found: 15 of 392 labels are <3 pixels in size"
Er kanskje viktig å beholde høy oppløsing i trening (og inference?) for å kunne detekte små poles. 
Juster hyperparameter img i yolov5. 
Bildene våre har bredde = 1960 pixels. 


## Generating synthetic images
Possible approaches
- GAN's. very good, but complicated, possibly not ideal for bounding boxes. 
- Use dall-e 3. The problem here is that its difficoult to iterate on the same image: If you ask dall-e to generate an image of a pole with a box around, it does that. But if you ask it to generate a picture of a pole, and then ask it to generate the same picture only with a box around the pole, the box location is wrong. 
Generate synthetic images with boxes around them: Detect box positions and create labels. use ai photoshop to remove box from picture. 
This can be fully automated, and the chatgpt api is only 0.08 USD per image. 
Conclusion: Not a good solution, as the bounding boxes are slightly wrong and will likely confuse the model more than help it. or so I think... It is a lot of work for something that may have no effect, or negative effect.
- generate heatmap of box position. Generate a formula for pole height vs y-position in image. Paste pole on different backgrounds according to the heatmap and height function. Will look unnatural


