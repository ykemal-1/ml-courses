# Samenvatting ter voorbereiding – ML Principles use-case

Persoonlijk hulpdocument, niet om in te dienen. Het notebook en de presentatieleidraad staan in dezelfde map.

## De case in drie zinnen

Een telecombedrijf wil weten welke klanten dreigen te vertrekken (*churn*), zodat het gericht retentieacties kan doen. De Kaggle-auteur Borbekov traint daarvoor één beslisboom op de IBM-dataset *Telco Customer Churn* (7043 klanten) en snoeit die met cross-validatie. Jij legt uit hoe de cursusconcepten in die bestaande analyse terugkomen en beoordeelt de keuzes. Je hebt het model **niet** zelf opnieuw getraind.

## Kernbegrippen, toegepast op de case

| Begrip (cursus) | In je eigen woorden | In de case |
|---|---|---|
| Taak | wat het model moet doen | binaire classificatie: vertrekt (1) of blijft (0) |
| Ervaring | hoe het model leert | supervised: de kolom `Churn` is de ground truth |
| Gestructureerde data | tabel met vaste kolommen | 1 rij per klant, numerieke en categorische kolommen |
| Ongebalanceerdheid | één klasse komt veel vaker voor | 73,5 % blijft, 26,5 % vertrekt, dus accuracy is misleidend |
| Missing data | ontbrekende of onbruikbare waarden | 11 lege `TotalCharges`, allemaal nieuwe klanten, opgevuld met 0 |
| Feature engineering | inputs omzetten naar bruikbare getallen | one-hot encoding geeft 30 features; ID weg; geen schaling (een boom vergelijkt alleen drempels) |
| Model en parameters | de splitsingen worden uit de data geleerd | het aantal splits ligt niet vooraf vast; de complexiteit groeit mee |
| Leeralgoritme | hoe de parameters gevonden worden | greedy CART; criterium niet opgegeven, dus **vermoedelijk Gini** (standaardwaarde) |
| Hyperparameter en tuning | een instelling die je kiest, niet leert | `ccp_alpha`, gekozen met `GridSearchCV` op F1 |
| Training / validatie / test | leren, kiezen, eindoordeel | 80/20-split met `stratify`; validatie via 5-fold CV binnen de trainingsset; de test pas op het einde |
| Overfitting / generalisatie | de trainingsdata van buiten leren tegenover goed doen op nieuwe data | een volledige boom kan sterk overfitten; snoeien maakt hem eenvoudiger en kan de generalisatie verbeteren (in de case niet gekwantificeerd) |
| Evaluatie | scoren op ongeziene data | confusion matrix, precision, recall, F1 |

## De resultaten van de auteur en wat ze betekenen

- Gesnoeide boom: `ccp_alpha` ≈ 0,00073, **34 bladeren**, CV-F1 0,570, test-F1 0,562.
- Confusion matrix: 182 TP, 92 FP, 192 FN, 943 TN.
- **Recall 0,49**: de boom vindt de helft van de vertrekkers. Een **FN** is een gemiste klant.
- **Precision 0,66**: 2 op 3 signalen zijn terecht. Een **FP** is een onnodige retentiekost.
- **Accuracy 0,80** zegt weinig: de baseline "altijd blijft" haalt al 0,735 zonder één vertrekker te vinden.

## Je kritische punten

1. **F1 is het harmonisch gemiddelde van precision en recall.** Het houdt geen rekening met de verschillende bedrijfskosten van een gemiste vertrekker en een vals alarm, terwijl de auteur zelf zegt dat een gemiste vertrekker duurder is. Een kostenmatrix, de $F_\beta$-score of een lagere drempel past beter.
2. **CV 0,570 en test 0,562 liggen dicht bij elkaar op deze ene split.** Dat is geruststellend, maar bewijst niet dat de keuze van `ccp_alpha` niet een beetje op de validatiefolds is afgestemd. Het verschil tussen trainings- en validatiescore wordt niet getoond, en de ongesnoeide boom ook niet. De winst door pruning is dus niet te kwantificeren.
3. **Eén testsplit**, zonder spreiding of betrouwbaarheidsinterval.
4. **Momentopname**: willekeurige split, niet in de tijd. Of het model volgende maand werkt, is niet getest.
5. **Explainability niet benut**: de regels van de boom worden niet getoond, terwijl dat het argument voor één boom is.
6. **Kleine methodologische kanttekeningen:**
   - `get_dummies` bepaalt de categorieën op de volledige dataset, vóór de split. Strikter fit je de encoder alleen op train.
   - De kandidaat-alpha's komen uit de volledige trainingsset, dus ook uit rijen die later validatiefold zijn.
   - De testset blijft in beide gevallen buiten schot.

En altijd: **verband is geen oorzaak**. Contracttype hangt samen met churn, maar dat bewijst niet dat een langer contract het vertrek voorkomt.

## Dit moet je vlot kunnen uitleggen

- Waarom `stratify=y` en waarom F1 in plaats van accuracy (de onbalans).
- Het verschil tussen een parameter (de splitsingen) en een hyperparameter (`ccp_alpha`).
- Wat cross-validatie doet en waarom de testset buiten de tuning blijft.
- Wat een FP en een FN betekenen voor het bedrijf.
- Wat 📎 van de auteur komt en wat 🔎 van jou: zijn code en cijfers, jouw uitleg en kritiek.

## Valkuilen tijdens de presentatie

- Zeg niet "ik heb getraind": je **bespreekt** de training van de auteur.
- Zeg niet "de auteur gebruikt Gini": hij geeft geen criterium op, dus Gini is een afleiding uit de standaardwaarde van scikit-learn.
- Vergelijk niet met andere modellen: dat valt buiten de goedgekeurde afbakening.
- Zeg niet "er is geen leakage" of "pruning verbetert de prestaties hier": noem de nuance (§4.1) en zeg dat het effect van pruning niet gerapporteerd wordt (§5.2).
