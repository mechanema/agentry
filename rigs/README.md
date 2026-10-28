# Rigs

One OpenRig rig per Agentry department (ADR-0001).

| Rig                           | Seats | Steered by            | Works on                   | Derived from                          |
| ----------------------------- | ----- | --------------------- | -------------------------- | ------------------------------------- |
| [`factory`](factory/)         | 7     | workflow engine       | a product repository       | OpenRig `factory-rsi`                 |
| [`agentry-dev`](agentry-dev/) | 4     | a human, via the lead | Agentry's own repositories | `pilotariak/fabrika` `pilotariak-dev` |

Validate a rig locally (no daemon needed):

```bash
rig spec validate rigs/<rig>/rig.yaml
rig spec audit rigs/<rig>/rig.yaml
```
