# Presentatieleidraad – ML Principles use-case (10 december 2026)

**Formaat:** 10 minuten presentatie + 5 minuten vragen.
**Wat je toont:** het notebook zelf: `ml-courses/projecten/ml_principles/ml_principles_usecase_telco_churn.ipynb`.
Dit is een persoonlijk hulpdocument en hoort niet bij de indiening.

**Rode draad:** je bespreekt de *bestaande* analyse van de Kaggle-auteur. Je toont waar de cursusconcepten terugkomen en welke keuzes en beperkingen erbij horen. Je hebt het model niet zelf opnieuw getraind.

## Vooraf

- Open het notebook in de devcontainer van de cursus en voer het **vooraf** volledig uit (*Run All*). De codecellen halen alleen de data op en herhalen de verkennende analyse van de auteur; dat duurt een paar seconden.
- In de container staat het notebook in de verkenner onder `projecten/ml_principles/` (`/workspaces/ml-courses/projecten/ml_principles/`).
- Leg dit document of het spiekbriefje onderaan naast je.

## Verloop (± 10 minuten)

| Tijd | Notebook | Wat je zegt (kern) |
|---|---|---|
| 0:00–1:00 | intro + §1 | Bestaande case: Kaggle-notebook van Borbekov, alleen het onderdeel *Decision Tree*. 📎 = auteur, 🔎 = jouw bespreking. Churn = klant vertrekt (hier: in de laatste maand). Het model levert een lijst risicoklanten voor retentieacties. Taak: **binaire classificatie**, ervaring: **supervised** met `Churn` als ground truth. |
| 1:00–2:30 | §2 Dataset + EDA | 7043 klanten, opgehaald met `kagglehub`. **Ongebalanceerd** (73,5 % / 26,5 %). 11 lege `TotalCharges` bij nieuwe klanten, opgevuld met 0. Toon de figuur: maand-tot-maand 43 % churn tegenover 3 % bij 2 jaar; vertrekkers zijn vooral nieuwe klanten. Verband, geen oorzaak. |
| 2:30–3:30 | §3 Modeltype | Eén beslisboom: parameters = de geleerde splitsingen. Criterium niet opgegeven, dus **vermoedelijk Gini** (standaardwaarde, afleiding). Waarom: heterogene data, geen schaling nodig, uitlegbaar. Beperking: kan sterk overfitten. |
| 3:30–4:30 | §4 Modeldefinitie | One-hot encoding geeft 30 features; ID is geen feature. Kanttekening: de categorieën worden op de volledige dataset bepaald, vóór de split. Strikter is de encoder alleen op train te fitten. Enige getunede hyperparameter: `ccp_alpha`. |
| 4:30–6:00 | §5 Training en tuning | 80/20-split met `stratify`. Volledige boom, kandidaat-alpha's uit `cost_complexity_pruning_path`, 5-fold CV op **F1**. Resultaat: alpha ≈ 0,00073, 34 bladeren, CV-F1 0,570. De testset blijft opzij. De ongesnoeide boom wordt niet gerapporteerd, dus de winst door pruning is niet te kwantificeren. |
| 6:00–8:30 | §6 Performantie | **Begin bij de baseline:** "altijd blijft" haalt al 73,5 % accuracy, maar vindt geen enkele vertrekker. **Dan de confusion matrix:** 182 vertrekkers gevonden, **192 gemist** (klant en omzet weg, geen actie) en **92 valse alarmen** (onnodige retentiekost). Dus recall 0,49 en precision 0,66. F1 houdt geen rekening met die verschillende kosten. |
| 8:30–10:00 | §7 Conclusie | Wel ondersteund: een bruikbare risicolijst, beter dan de baseline. Niet ondersteund: oorzaken, kostenoptimale inzet, de winst door pruning, prestaties in de tijd. |

**Afsluitzin:** "De gesnoeide boom van de auteur vindt de helft van de vertrekkers met twee op drie juiste signalen. Of dat genoeg is, hangt af van wat een gemiste klant kost tegenover een retentieactie, en die afweging zit nog niet in de analyse."

## Spiekbriefje cijfers (alle van de auteur)

| | Waarde |
|---|---|
| Klanten / kolommen / features na encoding | 7043 / 21 / 30 |
| Churn | 26,5 % (1869 klanten) |
| Train / test | 5634 / 1409 (waarvan 374 vertrekkers) |
| Gekozen `ccp_alpha` | ≈ 0,000726 |
| Bladeren gesnoeide boom | 34 |
| CV-F1 / test-F1 | 0,570 / 0,562 |
| Confusion matrix (TN, FP, FN, TP) | 943, 92, 192, 182 |
| Accuracy / precision / recall | 0,798 / 0,664 / 0,487 |
| Baseline (altijd "blijft") | accuracy 0,735, F1 0 |

## Achtergrond voor vragen (niet actief presenteren)

- **Formules** (§6.1): precision $= \frac{182}{274}$, recall $= \frac{182}{374}$, F1 $= \frac{2\,TP}{2\,TP + FP + FN}$.
- **Mogelijke uitbreidingen** (§6.3): drempel verlagen via `predict_proba`, PR-curve of ROC-AUC, $F_\beta$ of een kostenmatrix, spreiding over meerdere splits, de regels van de boom tonen.
- **Overige kanttekeningen** (§6.3): één split, momentopname, explainability niet benut.

## Waarschijnlijke vragen (kort antwoord)

- **Waarom F1 en geen accuracy?** Door de onbalans haalt "altijd blijft" al 73,5 % accuracy zonder één vertrekker te vinden. F1 kijkt naar de churnklasse.
- **Wat is F1 precies?** Het harmonisch gemiddelde van precision en recall. F1 houdt geen rekening met de verschillende bedrijfskosten van een gemiste vertrekker en een vals alarm.
- **Wat doet `ccp_alpha`?** Het is een prijs per blad in R(T) + α·|bladeren|. Een hogere α geeft minder bladeren en meer trainingsimpurity. Een tak blijft alleen staan als hij meer impurity wegwerkt dan hij aan bladkost toevoegt.
- **Verschil met `max_depth`?** `max_depth` stopt de groei vooraf op een vaste diepte voor alle takken. Pruning laat de boom eerst volledig groeien en beoordeelt daarna elke tak afzonderlijk.
- **Is een volledig uitgegroeide boom altijd zuiver?** Nee. Met `max_depth=None` en `min_samples_leaf=1` stopt een tak wanneer de knoop zuiver is of geen geldige splitsing meer mogelijk is, bijvoorbeeld bij identieke features met een ander label.
- **Hoe kiest de boom een split?** Hij kiest de split met de grootste daling van de gewogen Gini-impurity, greedy, knoop per knoop. Zie de tutorial.
- **Waarom `stratify=y`?** Zo behouden train en test dezelfde 73,5/26,5-verhouding. Anders kan de kleine churnklasse toevallig scheef verdeeld raken.
- **Is er data leakage?** Het churnlabel wordt niet gebruikt bij de encoding, en de opvulling met 0 hangt niet af van andere rijen. Wel bepaalt `get_dummies` de categorieën op de volledige dataset, vóór de split. Strikter fit je de encoder alleen op train. De kandidaat-alpha's komen uit de volledige trainingsset, dus ook uit rijen die later validatiefold zijn. De testset blijft overal buiten.
- **Hoe verhoog je de recall?** Verlaag de beslissingsdrempel op `predict_proba` (nu 0,5). Je vindt dan meer vertrekkers, ten koste van precision.
- **CV 0,570 en test 0,562: bewijst dat dat er geen overfitting is?** Nee. Op deze ene split doet de boom het op de test ongeveer even goed als bij de validatie, wat geruststellend is. Het bewijst niet dat de keuze van `ccp_alpha` niet een beetje op de validatiefolds is afgestemd. Of de boom de trainingsdata van buiten leert, zie je aan het verschil tussen trainings- en validatiescore, en dat rapporteert de auteur niet.
- **Wat is van jou en wat van de auteur?** Code, keuzes en alle cijfers van het model komen van de auteur (📎). De koppeling met de cursus, de betekenis van FP en FN, en de kritische kanttekeningen en mogelijke aanvullingen zijn van mij (🔎).
- **Waarom heb je het model niet zelf opnieuw getraind?** De opdracht is een bestaande case uitleggen en beoordelen. De code draait wel: de data wordt opgehaald en de verkennende analyse wordt herhaald. De modelcode en resultaten staan als citaat met bronvermelding.
- **Waarom geen random forest of logistische regressie?** De afbakening die u goedkeurde is één beslisboom. De vergelijking uit de bron valt bewust buiten de use-case.
