# Samenvatting ter voorbereiding – Mathematical Foundations tutorial

Persoonlijk hulpdocument, niet om in te dienen. Het notebook en de presentatieleidraad staan in dezelfde map.

## Het idee in drie zinnen

CART bouwt een beslisboom door telkens de ja/nee-vraag te kiezen die de groepen het **zuiverst** maakt, gemeten met de Gini-impurity. Zonder rem groeit de boom door tot een knoop zuiver is of niet meer gesplitst kan worden, en loopt hij het risico uitzonderingen van buiten te leren (overfitting). Cost-complexity pruning zet daarom een prijs $\alpha$ per blad tegenover de winst in zuiverheid, zodat alleen splits blijven die genoeg opleveren.

## De formules op één blad

| Formule | Betekenis in eigen woorden |
|---|---|
| $p_k = \dfrac{n_{t,k}}{n_t}$ | aandeel van klasse $k$ in knoop $t$ |
| $G(t) = 1 - \sum_k p_k^2$ | kans dat twee willekeurige voorbeelden uit de knoop een andere klasse hebben; 0 = zuiver |
| $G = 2p(1-p)$ | dezelfde formule bij twee klassen; maximaal 0,5 bij een 50/50-knoop |
| $G_{\text{split}} = \frac{n_L}{n_t} G(t_L) + \frac{n_R}{n_t} G(t_R)$ | gewogen impurity van de kinderen; grote kinderen tellen zwaarder |
| $\Delta G = G(t) - G_{\text{split}}$ | hoeveel de split de onzuiverheid verlaagt |
| $s^* = \arg\max_s \Delta G(s)$ | CART kiest de split met de grootste daling (greedy) |
| $\hat y = \arg\max_k p_k(t)$ | een blad voorspelt de meerderheidsklasse |
| $R(T) = \sum_{t \in \tilde T} \frac{n_t}{n} G(t)$ | trainingsimpurity van de hele boom (scikit-learn) |
| $R_\alpha(T) = R(T) + \alpha \lvert \tilde T \rvert$ | kost = impurity + prijs per blad |
| $g(t) = \dfrac{R(t) - R(T_t)}{\lvert \tilde T_t \rvert - 1}$ | impurity-winst per extra blad; bij $\alpha \ge g(t)$ wordt de tak gesnoeid |

## Het rekenvoorbeeld (zelf verzonnen, 12 klanten)

- **Wortel:** 6 churn / 6 blijft, dus $G = 1 - \tfrac14 - \tfrac14 = 0{,}5$.
- **Split A (tenure ≤ 6):** links 3/1 → $G = 0{,}375$; rechts 3/5 → $G = 0{,}46875$.
  Gewogen: $\tfrac{4}{12} \cdot 0{,}375 + \tfrac{8}{12} \cdot 0{,}46875 = 0{,}4375$, dus $\Delta G = 0{,}0625$.
- **Split B (tenure ≤ 12):** links 5/1, rechts 1/5 → beide $G = \tfrac{10}{36} \approx 0{,}2778$.
  Gewogen: 0,2778, dus $\Delta G \approx 0{,}2222$. **B wint**, en is ook de beste van alle 11 drempels.
- **Bladen na B:** links $p_1 = \tfrac56$ → churn; rechts $p_1 = \tfrac16$ → blijft.
- **Volledige boom:** 6 bladeren, hier allemaal zuiver ($R(T) = 0$), want alle `tenure`-waarden zijn verschillend. 100 % correct op de training, maar twee bladeren steunen op één klant (C04 en C11). Dat **illustreert het risico op overfitting**. Dat de boom echt slechter generaliseert, toon je pas aan met nieuwe data, en die zit niet in het voorbeeld.
- **Snoeien:**
  - elke tak onder de wortel: $g = \dfrac{6/12 \cdot 10/36}{3 - 1} = \tfrac{5}{72} \approx 0{,}069$;
  - de wortel: $g = 0{,}5 - \tfrac{10}{36} = \tfrac29 \approx 0{,}222$.

| $\alpha$ | gekozen boom | bladeren | $R(T)$ |
|---|---|---|---|
| $0 \le \alpha < \tfrac{5}{72}$ | volledige boom | 6 | 0 |
| $\tfrac{5}{72} \le \alpha < \tfrac29$ | stomp (split B) | 2 | 0,278 |
| $\alpha \ge \tfrac29$ | alleen de wortel | 1 | 0,5 |

Hoe hoger $\alpha$, hoe minder bladeren en hoe hoger de trainingsimpurity.

## `max_depth` tegenover `ccp_alpha`

- **`max_depth` (vooraf):** stopt de groei op een vaste diepte, voor alle takken gelijk, nuttig of niet.
- **`ccp_alpha` (achteraf):** laat de boom eerst volledig groeien en weegt daarna per tak de winst af tegen de bladkost.
- In het voorbeeld geven ze toevallig dezelfde boom (de takken zijn symmetrisch). Op echte data meestal niet.
- De juiste $\alpha$ kies je **met validatie**, niet met de trainingsdata, want daar wint altijd $\alpha = 0$.

## Link met de churncase

De Kaggle-auteur laat een volledige boom groeien, berekent met `cost_complexity_pruning_path` alle kandidaat-alpha's en kiest met 5-fold CV `ccp_alpha` ≈ 0,00073 (34 bladeren). Hij geeft geen criterium op, dus Gini is daar **een afleiding** uit de standaardwaarde van scikit-learn. Zijn $\alpha$ is niet rechtstreeks vergelijkbaar met de drempels uit het voorbeeld: ze hangt af van zijn boom en data, en is gekozen met CV.

## Kan ik dit? (checklist)

- [ ] Gini van een knoop uitrekenen en uitleggen wat het getal betekent.
- [ ] Aantonen dat het maximum bij twee klassen 0,5 is (afgeleide $2 - 4p = 0$).
- [ ] Split B hardop uitrekenen en in één zin zeggen waarom A slechter scoort.
- [ ] De drempel $\tfrac{5}{72}$ voor een tak afleiden.
- [ ] Uitleggen waarom het voorbeeld het risico op overfitting toont, maar het niet bewijst.
- [ ] Uitleggen waarom je weegt met $n_L/n_t$ (voorbeeld: drempel 1,5 met één zuivere klant).
- [ ] $g(t)$ afleiden door $R(t) + \alpha$ gelijk te stellen aan $R(T_t) + \alpha \lvert \tilde T_t \rvert$.
- [ ] Uitleggen wat een hogere `ccp_alpha` doet en hoe je hem kiest.
- [ ] Het verschil tussen `max_depth` en pruning in twee zinnen geven.
