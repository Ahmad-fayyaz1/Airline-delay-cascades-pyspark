# Detecting and Quantifying Airline Delay Cascades at Scale Using PySpark RDDs

**Course:** STA 473 – Tools and Techniques for Data Science
**Team:** Ahmad Fayyaz (SP24-BST-002) and Hajira Shahbaz (SP24-BST-010)
**Stage:** Midterm (45%)

## What this project does
When an aircraft lands late, its next flight often leaves late too. A run of such flights on the same plane is a **delay cascade**. This project finds these cascades in about 5.7 million US flights and measures how many minutes of delay each one passes on, using the PySpark **RDD** API (no DataFrame logic in the analysis).

For each pair of back-to-back flights `a -> b` of the same aircraft:

```
slack     = b.planned_departure - a.planned_arrival
inherited = min( max(0, a.arrival_delay - slack), b.departure_delay )
```

A link exists when `a` landed at least 15 min late (T), the slack is at most 60 min (S), `b` left late with a recorded late-aircraft delay, and `inherited > 0`.
**Cascade Impact Score** of a chain = total inherited minutes along the chain.

## Data
2015 Flight Delays and Cancellations (US Bureau of Transportation Statistics), on Kaggle: `kaggle.com/datasets/usdot/flight-delays`.
The data is **not stored in this repository** (size and Kaggle terms). Download `flights.csv` yourself and place the zip in your Google Drive.

## RDD techniques used
`textFile`, `filter`, `map`, `flatMap`, `groupByKey`, `reduceByKey`, `aggregateByKey`, `takeOrdered`, `takeSample`, `cache` / `persist`, **accumulators** (cleaning counters) and a **broadcast variable** (duplicate keys).

## Cleaning results (full year)
| Step | Rows |
|---|---|
| Rows read | 5,819,079 |
| Malformed | 0 |
| Cancelled or diverted | 105,071 |
| Missing tail number | 0 |
| Duplicate keys removed | 1,194 |
| **Clean flights** | **5,712,814** |

## Prototype results (January and February 2015, 864,516 flights)
- 839,235 consecutive pairs, 37,143 propagation links (4.4%), 24,349 chains
- Chain lengths 2 / 3 / 4 / 5 / 6 / 7 / 8: 16,498 / 4,617 / 2,033 / 808 / 295 / 81 / 17
- Airports passing on the most delay (total minutes): ORD, LGA, DEN, SFO, ATL

## How to run (Google Colab)
1. Open `notebooks/Airline_Delay_Cascades_Clean.ipynb` in Colab.
2. Run the setup cells (Java, PySpark 4.0.4, Drive mount, Spark session).
3. Set `ZIP_PATH` and `PARQUET_PATH` in the paths cell.
4. Part 1 (cleaning) needs the raw zip. Parts 2 and 3 only need the saved Parquet.
5. `SAVE = False` by default so the cleaned data is never overwritten by accident.

## Repository layout
```
notebooks/   the Colab notebook (cleaning, cascade analysis, exploratory)
report/      midterm report (PDF)
docs/        literature review
requirements.txt
```

## Roadmap
| When | Work |
|---|---|
| Done | Cleaning pipeline, chain detector, example chains, exploratory analysis, report |
| Next | Slides; small synthetic Pakistan-style dataset with planted cascades to check the detector |
| Weeks 5-7 | Run on the full year; impact scores per flight, airport and route; rates per flight for fair airport comparison |
| Weeks 8-9 | Test S = 45 / 60 / 90 min; RDD vs DataFrame speed comparison; check results; recommendations |
| Week 10 | Final report and presentation |

## Limitations
Historical batch analysis only (no live streaming). Chains rely on the BTS late-aircraft field, which is recorded only for flights 15+ minutes late. A high spread ratio means one delay was carried through several flights, not that the delay grew.

## References
Brueckner, Czerny & Gaggero (2022); AhmadBeygi, Cohn, Guan & Belobaba (2008); AhmadBeygi, Cohn & Lapp (2010); Arikan, Deshpande & Sohoni (2013); Fleurquin, Ramasco & Eguiluz (2013); Pyrgiotis, Malone & Odoni (2013). Full list in `docs/`.
