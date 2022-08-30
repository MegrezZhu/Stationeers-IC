# Advanced Furnace with Temperature & Pressure Maintenance

## Prerequisite

* A tank with hot gas >= 2000K
* A tank with cold gas <= 500K
* hot and cold gas should have similar mixture, hence similar heat capacity

## Important fomula

* Ideal gas: $pV=nRT$, $R=8.314$, with kPa, K, Liter
* Gas internal energy: $E=CnT$, where $C$ is heat capacity

## Modules

### Temperature Mixing

Given hot@($T_h$ K, $p_h$ kPa) and cold@($T_c$ K, $p_h$ kPa) gases, we want $x$ mols @ $T$ K using two volume pumps. Q: what should the hot and cold pumps pump, $a$ and $b$ L (their `Setting` values)?

Note: Volume pump takes `Setting` L of gases from its input pipe network.

$$
p_cV_c=n_cRT_c
$$
$$
p_hV_h=n_hRT_h
$$

In one tick, the hot volume pump takes $a/V_c$ gas, which has $\Delta{n_h}=n_h*\frac{a}{V_h}=\frac{p_h}{RT_h}a$, similarly for the cold pump, $\Delta{n_c}=\frac{p_c}{RT_c}b$.

Temperature of the gas mixture:
$$
T=\frac{c_h\Delta{n_h}T_h+c_c\Delta{n_c}T_c}{c_h\Delta{n_h}+c_c\Delta{n_c}}
$$

Let $k=x/y$, then we have (with some calculation):

$$
k=\frac{c_cp_c}{c_hp_h}*\frac{\frac{T}{T_c}-1}{1-\frac{T}{T_h}}
$$

#### Mixing to Exact Amount of Mols

$$
\Delta{n}=\Delta{n_h}+\Delta{n_c}=\frac{p_h}{RT_h}a+\frac{p_c}{RT_c}b
$$
so with $k$ and $\Delta{n}$, we have
$$
b=\frac{\Delta{n}R}{\frac{p_hk}{T_h}+\frac{p_c}{T_c}}
$$

### Furnace Temperature and Pressure Control

#### Methodology

#### Temperature Adjustment

Say the furnace currently has gases@($T_0,p_0,n_0$), we want to raise the temperature to $T_1$ using hot gas at $T_h$, ignoring the pressure limit and assuming identical heat capacity, how many mols, $n_h$, do we need?

$$
T_1=\frac{cn_0T_0+cn_hT_h}{cn_0+cn_h}
$$
so,
$$
n_h=\frac{n_0(T_1-T_0)}{T_h-T_1}
$$

(Note: This result is also applicable to cooling, just replace $T_h$ with $T_c$ and ditto for the others).

#### Pressure Management

Adding hot gases and only de-pressurize the furnace is simplest, but it's not only dangerous, but also wasting: Say we already have the furnace with cold gases@(1K, 1MPa, 1000L), which is ~ 120kmol, to make it 1000K we would need to pump in another ~120kmol of hot gas @ 2000K to make it 1000K mixture.

In comparison, if we clear the furnace, we only need 120mol of 1000K gases, or similar amount of hot and cold combination. So it would be more efficient to pump out excessive gas before pumping in hot gases.

Say we want $T_1,p_1$, with existing $T_0,p_0$ (note that furnace $V$ is always 1000L), then in the end state,

$$
n_1=\frac{p_1V}{RT_1}
$$

but since we will need to add another $n_h$ mols of gases, we need to pump out $n'=n_0+n_h-n_1$ mols of gases in advance.

**How to pump x mols from the furnace?**

The furnace input / output is also volume pumps, so we would want to pump $\Delta{v}$ out, where

$$
p\Delta{v}=\Delta{n}RT
$$
so $\Delta{v}=\frac{\Delta{n}RT}{p}$.

(Note: this is because in Stationeers gas temperature does not change with pressure change, see [Wiki](https://stationeers-wiki.com/Pressure,_Volume,_Quantity,_and_Temperature), while pressure does change with temperature change, so it's temperature -> pressure, not vice versa).

### When pressure too high, for whatever reason

Current pressure is $p_0$, we want to lower it down to $p_1$ by removing $\Delta{v}$ mols, (note again, temperature does not change in this process in Stationeers).

$$
% \Delta{p}V=\Delta{n}RT=(\frac{p_0V}{RT}-\frac{p_1V}{RT})RT
p_0(V-\Delta{v})=p_1V
$$
so,
$$
\Delta{v}=(1-\frac{p_1}{p_0})V
$$

### Finally, pseudo code

```py
while True:
  sleepOneTick()
  if pressure >= targetPressure:
    outputSetting = ...
  elif pressure < targetPressure or temperature != targetTemperature:
    if needDepressurize():
      outputSetting = ...
    else
      if temperature < targetTemperature:
        mediaTemp = HIGH
      elif temperature > targetTemperature:
        mediaTemp = COLD
      else:
        mediaTemp = targetTemperature
      mediaMols = ...
```