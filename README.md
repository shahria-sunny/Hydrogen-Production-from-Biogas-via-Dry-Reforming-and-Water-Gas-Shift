# Hydrogen Production from Biogas — Dry Reforming + Water-Gas Shift

**Aspen Plus V14.0 simulation | Steady-state | PR-MHV2 property method**

---
## Why this matters
Biogas is roughly equal parts methane and CO₂. Most hydrogen production routes treat the CO₂ as a problem to remove before the real chemistry starts. Dry reforming takes the opposite approach — the CO₂ stays in and reacts with the methane directly, converting both into syngas. Whether that's clever or inconvenient depends on what you're trying to optimize. In this simulation, it's both.
The carbon accounting is where this route earns its place. Steam methane reforming on a biogas feed makes little thermodynamic sense when 40% of the feed is CO₂ diluent. DRM uses that CO₂ as a reactant, and the numbers reflect it: net CO₂e balance lands at −292.26 kg/hr. The process removes more carbon than it emits — an unusual result for an industrial hydrogen route.
The coke problem is real and the simulation doesn't hide it. At 16 bar, 4.95 kg/hr of solid carbon forms from a 38.51 kg/hr feed — 12.9% of feed mass going out as coke. That would foul a Ni catalyst bed fast. The fix is either adding steam to the reformer feed or dropping operating pressure toward 1–5 bar. That's the first thing any follow-on work should tackle.

## What this is

A steady-state process simulation of hydrogen production from biogas using dry reforming of methane (DRM) followed by dual-stage water-gas shift (WGS) and pressure swing adsorption (PSA) purification. Built and run in Aspen Plus V14.0.

The feedstock is biogas at 38.51 kg/hr — 59.7 vol% CH₄, 40.06 vol% CO₂, 0.2 vol% N₂, 0.04 vol% O₂. What makes DRM different from the standard steam methane reforming route is that it uses the CO₂ already present in the biogas as a co-reactant rather than treating it as a diluent to remove. The process converts both CH₄ and CO₂ into syngas simultaneously, which is either an elegant use of a waste-rich feed or a thermodynamic headache depending on what you're trying to optimize. In this case, both things are true.

---

## Process overview

```
BIOGAS ──► Compressor (3-stage, 1→16 bar) ──► Pre-heater E1 ──► Dry Reformer (909°C, 16 bar)
                                                                         │
                                                               CARBON (solid, 0.412 kmol/hr)
                                                                         │
WATER ──► Pump P1 ──► Steam generator E2 ──► Mixer B9 ◄─────────────────┘
                                                  │
                                          HT-WGS (350°C, 16 bar, 75% CO conv.)
                                                  │
                                          LT-WGS (210°C, 15.7 bar, 75% CO conv.)
                                                  │
                                          PSA (38°C, 15.65 bar, 79% H₂ sep. eff.)
                                          │                │
                                      H₂ product      Tail gas ──► Combustor (2303°C) ──► Heat cascade
```

Tail gas from the PSA feeds the combustor alongside air (79.68 kg/hr). The adiabatic combustor reaches 2302.6°C, and that heat cascades back through E1 (biogas pre-heat), E2 (steam generation), and E6 (combustion air pre-heat) before leaving as flue gas at 200°C. The reformer runs entirely on this recovered heat — no external fuel.

---

## Key results

| Parameter | Value |
|---|---|
| Biogas feed | 38.51 kg/hr |
| Reformer conditions | 909°C, 16 bar |
| CH₄ conversion | 80% |
| Cumulative CO conversion (both WGS stages) | 93.75% |
| H₂ in product stream | 0.384 kmol/hr (0.774 kg/hr) |
| H₂ yield per kg biogas | 0.0201 kg H₂ / kg biogas |
| Solid carbon (coke) produced | 0.412 kmol/hr (4.95 kg/hr) |
| Combustor adiabatic temperature | 2302.6°C |
| Total compressor work | 4.259 kW |
| Reformer heat duty | 7534 cal/s (endothermic) |
| Heat recovered (E1 + E2 + E6) | 7679 cal/s |
| Net CO₂e reduction | −292.26 kg CO₂e/hr |
| Convergence | 3 iterations (Wegstein) |
| Max. relative convergence error | 9.10×10⁻¹¹ |

The coke number is worth paying attention to. 4.95 kg/hr of solid carbon from a 38.51 kg/hr biogas feed means 12.9% of feed mass goes out as coke. That is a known DRM problem at elevated pressure — the Boudouard equilibrium and methane cracking both produce solid carbon, and running at 16 bar makes it worse. It would be the first thing to address in any follow-on work, either by adding steam to the reformer feed or by dropping operating pressure closer to 1–5 bar.

---

## Flowsheet components

| Block | Model | Role |
|---|---|---|
| C1 | MCOMPR (3-stage) | Biogas compression, 1 → 16 bar |
| P1 | PUMP | Water pressurisation, 1 → 16 bar |
| E1 | HEATX (countercurrent) | Biogas pre-heat, 155°C → 909°C |
| E2 | HEATX (countercurrent) | Steam generation, 25°C → 350°C |
| E3 | HEATER | Syngas cooling, 909°C → 350°C |
| E4 | HEATER | HT-WGS outlet cooling, 551°C → 210°C |
| E5 | HEATER | LT-WGS outlet cooling, 264°C → 38°C |
| E6 | HEATX (countercurrent) | Combustion air pre-heat, 29°C → 250°C |
| E7 | HEATER | Final flue gas cooling to 200°C |
| REFORMER | RGIBBS | Dry reformer — Gibbs free energy minimisation |
| HT-WGS | RSTOIC | High-temperature WGS, 75% CO conversion |
| LT-WGS | RSTOIC | Low-temperature WGS, 75% CO conversion |
| PSA | SEP | Pressure swing adsorption — component split shortcut |
| COMBUSTO | RGIBBS | Adiabatic tail gas combustion |
| VALVE | VALVE | PSA tail gas pressure letdown, 15.7 → 1 bar |
| B9, B16 | MIXER | Stream mixing |

---

## Simulation setup

**Property method:** PR-MHV2 (Peng-Robinson with Modified Huron-Vidal mixing rules). Handles the full range of conditions in this flowsheet — liquid water at 25°C and 1 bar through to combustion gas at 2303°C — without needing to switch property packages.

**Components:** CH₄, CO₂, CO, H₂, H₂O, N₂, O₂, C (carbon-graphite solid).

**Reactor model choices:**
- RGIBBS for the reformer and combustor — multiple simultaneous reactions, no need to specify stoichiometry, equilibrium-appropriate at these temperatures
- RSTOIC for WGS reactors — 75% per-stage CO conversion taken directly from the industrial design basis for Fe-Cr and Cu-Zn catalyst systems
- SEP for PSA — shortcut model with defined component split fractions

**Convergence:** One tear stream (COMB-F). Wegstein acceleration, tolerance 1×10⁻⁴. Converged in 3 iterations.

---

## Limitations

- **No catalyst deactivation.** The coke produced at these conditions would foul a real Ni catalyst bed quickly. RGibbs predicts equilibrium — it says nothing about how long the catalyst lasts.
- **No pressure drop across reactor beds.** Real packed beds typically drop 0.5–2 bar per bed, which affects downstream separation and recycle pressures.
- **PSA is a shortcut model.** The SEP block splits by specified fractions. It does not simulate adsorption cycles, bed saturation, purge gas consumption, or cycle time.
- **The reformer heat duty is not explicitly connected to the combustor.** The reformer runs with an assigned duty of 7534 cal/s; the combustor runs adiabatically. They are not linked by an explicit heat stream, which is why the overall flowsheet enthalpy balance shows a 26.9% discrepancy. Individual unit operation results are consistent — the gap is a global accounting issue.
- **No economic analysis.** No capital cost, OPEX, or levelized cost of hydrogen figures are in scope here.


---
## Part of a portfolio

This is the final project in a four-part simulation arc.

The first three projects follow conventional refinery and gas treating 
operations: crude distillation, amine-based H₂S removal, Claus sulfur 
recovery. Each manages hydrocarbons and acid gases as they move through 
an industrial facility. This project asks a different question — what 
happens when the feedstock is waste biomass and the goal is green hydrogen 
with a net-negative carbon balance.

The arc runs from fossil crude to biogas-derived hydrogen. Same simulation 
tools, same engineering discipline, different destination.

← Previous: [Claus SRU — Sulfur Recovery](https://github.com/shahria-sunny/Claus-SRU-Aspen-Plus)



## Author

**Shahriar Hossain Suny**  
Department of Chemical Engineering  
Jashore University of Science and Technology (JUST), Bangladesh  
AIChE Member ID: 009905932744  
[LinkedIn](https://www.linkedin.com/in/shahriarhossainsuny/) · [GitHub](https://github.com/shahria-sunny)
