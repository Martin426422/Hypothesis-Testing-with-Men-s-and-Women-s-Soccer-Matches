
# Czy w kobiecym futbolu międzynarodowym pada więcej goli niż w męskim? — Mistrzostwa Świata FIFA (od 2002)

## Przegląd
Jako dziennikarz sportowy sprawdzamy hipotezę, czy w **meczach kobiet** pada **więcej goli** niż w **meczach mężczyzn**. Aby ograniczyć wpływ zmian historycznych i różnych turniejów, analizę zawężamy do **oficjalnych meczów Mistrzostw Świata FIFA** (bez eliminacji) **od 2002‑01‑01**.

Stosujemy **nieparametryczny test** Manna–Whitneya (jednostronny, prawa strona) przy **α = 0.10**.

- **H0 (zerowa):** Średnia liczba goli w meczach kobiet = meczach mężczyzn.
- **H1 (alternatywna):** Średnia liczba goli w meczach kobiet jest **większa** niż w meczach mężczyzn.

## Dane
Dwa pliki CSV z wynikami meczów od XIX wieku, następnie **filtrowane** do finałów MŚ:
- `men_results.csv`
- `women_results.csv`

**Kolumny oczekiwane**: `date`, `tournament`, `home_score`, `away_score` (+ inne metadane).

## Metodologia
1. **Filtr**: `tournament == 'FIFA World Cup'` i `date > '2002-01-01'`.
2. **Gole na mecz**: `home_score + away_score`.
3. **Porównanie rozkładów** goli (kobiety vs mężczyźni) testem **Manna–Whitneya**.
4. Test **jednostronny** (`alternative='greater'`) — hipoteza: kobiety > mężczyźni.
5. **Decyzja** przy **α = 0.10**: odrzucamy H0, gdy *p* < 0.10.

## Kod (MVP)
```python
import pandas as pd
from scipy.stats import mannwhitneyu

men    = pd.read_csv("men_results.csv")
women  = pd.read_csv("women_results.csv")

men["date"]   = pd.to_datetime(men["date"]) 
women["date"] = pd.to_datetime(women["date"]) 

men_wc   = men[(men["date"] > "2002-01-01") & (men["tournament"] == "FIFA World Cup")]
women_wc = women[(women["date"] > "2002-01-01") & (women["tournament"] == "FIFA World Cup")]

men_goals   = men_wc["home_score"] + men_wc["away_score"]
women_goals = women_wc["home_score"] + women_wc["away_score"]

stat, p_val = mannwhitneyu(x=women_goals, y=men_goals, alternative="greater")
result = "reject" if p_val < 0.10 else "fail to reject"
print({"p_val": p_val, "result": result})
```

## Przykładowy wynik
```text
{"p_val": 0.005106609825443641, "result": "reject"}
```
Interpretacja: przy **α = 0.10** **odrzucamy H0** — mecze kobiet na MŚ od 2002 r. mają **więcej goli** niż mecze mężczyzn (statystycznie istotna różnica w rozkładach).

## Założenia i kontrola jakości
- Niezależność obserwacji między grupami.
- Taki sam typ turnieju (finały MŚ), porównywalny kontekst.
- Analizy wrażliwości: podział na okresy; oddzielnie regulaminowy czas vs. doliczony; modelowanie Poisson/NB.

## Wizualizacja (opcjonalnie)
```python
import matplotlib.pyplot as plt
import seaborn as sns

both = pd.concat([
    women_wc.assign(group="women", goals_scored=women_goals),
    men_wc.assign(group="men", goals_scored=men_goals)
])

sns.boxplot(data=both, x="group", y="goals_scored")
plt.title("Gole na mecz — MŚ FIFA (od 2002)")
plt.show()
```

## Uruchomienie
1. Umieść `men_results.csv` i `women_results.csv` w `data/`.
2. Zainstaluj zależności: `pip install pandas scipy seaborn matplotlib jupyter`.
3. Uruchom notebook lub skrypt.

## Ograniczenia
- Tylko finały MŚ; **bez eliminacji**, sparingów i rozgrywek kontynentalnych.
- Zmiany zasad/formatu w czasie mogą wpływać na liczby bramek.
- Test Manna–Whitneya bada **rangi/mediany**; do średnich rozważ **Welcha** lub bootstrap.

## Licencja
MIT License © 2025 Martin Karwacki
