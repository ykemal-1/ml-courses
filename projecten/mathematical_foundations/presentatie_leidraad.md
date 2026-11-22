# Presentatieleidraad – Mathematical Foundations mini-tutorial (CART, Gini en pruning)

**Formaat:** korte mondelinge presentatie en een korte vragenronde in de labo's van december. De fiche vermeldt geen duur of datum. Gebruik 10 minuten als **oefentiming** en pas aan als de lector iets anders vraagt.
**Wat je toont:** het notebook `ml-courses/projecten/mathematical_foundations/math_foundations_tutorial_cart_gini_pruning.ipynb`.
Dit is een persoonlijk hulpdocument en hoort niet bij de indiening.

**Waar de lector op let:** volgens de fiche peilt de vragenronde naar *je eigen begrip*: je moet de wiskunde in je eigen woorden kunnen uitleggen. Lees de formules dus niet voor, maar zeg telkens wat ze betekenen. Het notebook bevat alle berekeningen. Mondeling reken je er bewust maar een paar hardop uit.

## Vooraf

- Open het notebook in de devcontainer van de cursus en voer het **vooraf** volledig uit (*Run All*, enkele seconden). In de container staat het in de verkenner onder `projecten/mathematical_foundations/` (`/workspaces/ml-courses/projecten/mathematical_foundations/`).
- **Oefen op papier:** split B, split A en de pruningdrempel $\tfrac{5}{72}$. De wortel ($\tfrac29$) hou je achter de hand voor vragen.

## Compacte route (± 10 minuten oefentiming)

| Tijd | Notebook | Wat je zegt (kern) |
|---|---|---|
| 0:00–1:00 | §1 Inleiding | Een beslisboom splitst de data met ja/nee-vragen. Twee vragen: **welke split kiest CART?** en **hoe hou je de boom klein genoeg?** Kort de link met de churncase. |
| 1:00–2:30 | §2.1–2.2 | $G = 1 - \sum p_k^2$ meet hoe gemengd een knoop is: zuiver geeft 0, 50/50 geeft 0,5. Een split beoordeel je met het **gewogen** gemiddelde van de kinderen. CART kiest greedy de grootste daling $\Delta G$. |
| 2:30–5:00 | §3.1–3.4 | Didactische dataset: 12 verzonnen klanten, alleen `tenure`. Wortel 6/6, dus $G = 0{,}5$. **Reken split B (≤ 12) hardop:** kinderen 5/1 en 1/5, elk $G = 1 - \tfrac{25}{36} - \tfrac{1}{36} = \tfrac{10}{36}$; gewogen $\tfrac{10}{36} \approx 0{,}278$; $\Delta G \approx 0{,}222$. **Dan kort split A (≤ 6):** die laat een grote, gemengde rechterknoop (3/5, 8 klanten) staan, dus gewogen 0,4375 en $\Delta G$ maar 0,0625. B is beter, en het staafdiagram toont dat B ook de beste van alle 11 drempels is. |
| 5:00–5:30 | §3.5 | Een blad voorspelt de meerderheidsklasse: links $\tfrac56$ churn, dus churn; rechts blijft. |
| 5:30–6:30 | §3.6 | De onbegrensde boom: 6 zuivere bladeren, 100 % correct op de training. Twee bladeren steunen op één klant (C04 en C11). Dat **illustreert het risico op overfitting**. Of de boom echt slechter generaliseert, toon je pas met nieuwe data aan, en die zit niet in dit voorbeeld. |
| 6:30–8:30 | §2.4, §3.7 | $R_\alpha(T) = R(T) + \alpha \lvert \tilde T \rvert$: trainingsimpurity plus een prijs per blad. Een hogere $\alpha$ geeft minder bladeren en meer trainingsimpurity. **Leid één drempel af:** de rechtertak als blad kost $R(t) = \tfrac{6}{12} \cdot \tfrac{10}{36} = \tfrac{5}{36}$; de deelboom eronder heeft 3 zuivere bladeren ($R = 0$); dus $g = \tfrac{5/36}{3-1} = \tfrac{5}{72} \approx 0{,}069$. Vanaf die $\alpha$ worden beide takken gesnoeid en blijft split B over. Toon de grafiek met de drie lijnen. |
| 8:30–10:00 | §3.8, §4, §5 | `max_depth` beperkt vooraf alle takken op dezelfde diepte; pruning beoordeelt achteraf elke tak apart. De juiste $\alpha$ kies je met validatie, zoals de Kaggle-auteur met 5-fold CV. Gini is daar een afleiding uit de standaardwaarde van scikit-learn. Sluit af. |

**Afsluitzin:** "CART kiest telkens de vraag die de groepen het zuiverst maakt, gemeten met Gini. Pruning zet daar een prijs per blad tegenover, zodat alleen splits blijven die genoeg opleveren. Hoe hoog die prijs moet zijn, laat je de validatiedata beslissen."

## Spiekbriefje

| | Waarde |
|---|---|
| Gini van een binaire knoop | $2p(1-p)$; zuiver 0, 50/50 0,5 |
| Wortel | 6 churn / 6 blijft, $G = 0{,}5$ |
| Split A (≤ 6) | links 3/1 ($G = 0{,}375$), rechts 3/5 ($G = 0{,}46875$), gewogen 0,4375, $\Delta G = 0{,}0625$ |
| Split B (≤ 12) | links 5/1, rechts 1/5 (elk $G = \tfrac{10}{36} \approx 0{,}2778$), gewogen 0,2778, $\Delta G = 0{,}2222$ |
| Volledige boom | 6 zuivere bladeren, diepte 3, trainingsaccuracy 1,0 |
| Snoeipad | $\alpha = 0 \to$ 6 bladeren; $\tfrac{5}{72} \approx 0{,}069 \to$ 2 bladeren; $\tfrac29 \approx 0{,}222 \to$ 1 blad |
| $R(T)$ per boom | 0 / $\tfrac{10}{36} \approx 0{,}278$ / 0,5 |

## Waarschijnlijke vragen (kort antwoord)

- **Waarom is 0,5 het maximum bij twee klassen?** $G = 2p(1-p)$ is een bergparabool. De afgeleide $2 - 4p$ is nul bij $p = \tfrac12$, en daar is $G = 0{,}5$. Bij $K$ klassen is het maximum $1 - \tfrac1K$, bij een gelijke verdeling.
- **Waarom de kwadraten in de formule?** $\sum p_k^2$ is de kans dat twee willekeurig getrokken voorbeelden dezelfde klasse hebben. $1 - \sum p_k^2$ is dus de kans dat ze verschillen, oftewel hoe gemengd de knoop is.
- **Waarom wegen met $n_L/n_t$?** Anders lijkt een split die één voorbeeld afzondert sterk. Voorbeeld uit de data: bij drempel 1,5 is het linkerkind (1 klant) zuiver en het rechterkind 5/11 ($G \approx 0{,}496$). Het ongewogen gemiddelde ($\approx 0{,}248$) zou zelfs beter lijken dan split B, maar gewogen is het 0,4545 ($\Delta G \approx 0{,}045$).
- **Hoeveel drempels test CART?** Het aantal verschillende waarden min één, telkens het midden tussen twee opeenvolgende waarden. Hier zijn dat 11.
- **Waarom "greedy" en wat is het nadeel?** Elke split is de beste *op dat moment*. CART kijkt niet vooruit, dus een split die zelf weinig oplevert maar daaronder een heel goede split mogelijk maakt, wordt gemist. Daarom is de boom niet gegarandeerd globaal optimaal.
- **Groeit een onbegrensde boom altijd tot elk blad zuiver is?** Nee. Een tak stopt ook als er geen geldige splitsing meer mogelijk is, bijvoorbeeld bij voorbeelden met identieke features maar een ander label. In ons voorbeeld zijn alle `tenure`-waarden verschillend, dus daar worden alle bladeren wel zuiver.
- **Toont je voorbeeld aan dat de volledige boom slechter generaliseert?** Nee. Het toont het *risico*: 100 % trainingsaccuracy en regels die op één klant steunen. Aantonen kan pas met evaluatie op nieuwe data.
- **Gini of entropie?** Beide meten onzuiverheid en zijn 0 bij een zuivere knoop. Entropie ($-\sum p_k \log p_k$) komt uit de informatietheorie. Gini vraagt geen logaritme en is de standaard in scikit-learn. In de praktijk kiezen ze meestal gelijkaardige splits.
- **Waarom niet gewoon de foutclassificatieratio ($1 - \max_k p_k$)?** Die is minder gevoelig voor veranderingen in de klassenverhoudingen. Twee splits kunnen hetzelfde aantal fouten geven, terwijl de ene veel zuiverdere kinderen oplevert. Gini en entropie zien dat verschil wel.
- **Waarom voorspelt een blad de meerderheidsklasse?** Dat geeft in dat blad het kleinste aantal fout geclassificeerde trainingsvoorbeelden.
- **Hoe kom je aan de formule voor $g(t)$?** Als blad kost de knoop $R(t) + \alpha$, als deelboom $R(T_t) + \alpha \lvert \tilde T_t \rvert$. Zet ze gelijk en los op naar $\alpha$: $\alpha = \frac{R(t) - R(T_t)}{\lvert \tilde T_t \rvert - 1}$. Boven die $\alpha$ is snoeien goedkoper.
- **En de drempel van de wortel?** Na het snoeien van beide takken: $g = \frac{0{,}5 - 10/36}{2 - 1} = \tfrac29$. Bij de wortel is $n_t = n$, dus dat is precies $\Delta G$ van split B.
- **Waarom staat $\tfrac{5}{72}$ twee keer in het pad?** Het algoritme snoeit één tak per stap. Beide takken hebben dezelfde $g$, dus de tweede valt bij dezelfde $\alpha$ weg.
- **Wat gebeurt er bij $\alpha = 0$ en bij een heel grote $\alpha$?** Bij 0 zijn bladeren gratis en blijft de volledige boom staan, met de laagste trainingsimpurity. Dat is $R(T) = 0$ in ons voorbeeld omdat alle bladeren zuiver zijn, maar niet in het algemeen. Bij een heel grote $\alpha$ blijft alleen de wortel over. Hier is dat een gelijke stand van 6/6, en dan kiest scikit-learn klasse 0.
- **Waarom is de $\alpha$ van de Kaggle-auteur zo veel kleiner?** Die waarde hangt af van zijn boom en data, en hij kiest ze met cross-validatie. Ze is niet rechtstreeks vergelijkbaar met de drempels uit dit voorbeeld.
- **Hoe kies je $\alpha$ in de praktijk?** Niet met de trainingsdata, want daar wint altijd $\alpha = 0$. Je kiest met validatie, bijvoorbeeld K-fold cross-validatie zoals de Kaggle-auteur.
- **Is je dataset echt?** Nee. Ze is zelf samengesteld en zo gemarkeerd in het notebook, zodat alles met de hand na te rekenen is. Ze volgt wel het patroon uit de echte data: nieuwe klanten vertrekken vaker.
