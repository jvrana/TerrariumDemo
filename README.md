# TerrariumDemo
Terrarium v0.1 is a [dynamic computer-aided process planner (CAPP)](https://en.wikipedia.org/wiki/Computer-aided_process_planning) for biology designed for agile manufacturing of biological products, such as E Coli & Yeast strains, or Mammalian cell lines.

This project demonstrates the forward planning algorithm of the software.

## Usage

### Installation

```python
pip install terrarium-capp==0.1.5 -U
```

### Running examples

```bash
make example1
make example2
```

### Running arbitrary JSON files

```bash
terrarium login --username <username> --url <aq_url> --password <password> design <input_json>
```

#### Description of JSON file (Terrarium v0.1)

The input JSON is comprised of *four* main keys:
```json
{
    "TRAIN": {...},
    "MODEL_PATH": "terrarium.pkl",
    "GOALS": [...],
    "GLOBAL_CONSTRAINTS": {...}
}
```
##### TRAIN
This is the query to search Aquarium for experimental plans to train the main algorithm. This training process can take a few minutes depending on the size of the data being used to train. A common query may be:

```json
  {
        "method": "last",
        "args": [500],
        "query": {
            "user_id": 66
        }
    }
```

Which will use the last 500 plans corresponding to user with user_id == 66 (i.e. the id for Justin Vrana, you can find your user_id using trident via `session.User.one(query={'login': 'mylogin'})`)

##### MODEL_PATH

After training, Terrarium will store the trained model as a serialized pickle for future runs. Where this file is stored or read in the future is indicated by the `MODEL_PATH`

##### GOALS

This query indicates the type of goals Terrarium will attempt to plan. The following example plans a sample called `CEN.PK2 - MAT alpha | pyMOD-URA-URA3.A.1-pGPD-yeVenus-tCYC1` in the **SAMPLE** key and attempts to find a route to produce a `Yeast Glycerol Stock` in the **OBJECT** key. Additionally, we specify how to construct the `pyMOD-URA-URA3.A.1-pGPD-yeVenus-tCYC1` plasmid from a list of fragments in the **EDGES** key. The plan is to be named `MyPlan`. 

```json
"GOALS": [
        {
            "PLAN_ID": "MyPlan",
            "SAMPLE": {
                "model_class": "Sample",
                "query": {
                    "name": "CEN.PK2 - MAT alpha | pyMOD-URA-URA3.A.1-pGPD-yeVenus-tCYC1"
                }
            },
            "OBJECT_TYPE": {
                "model_class": "ObjectType",
                "query": {
                    "name": "Yeast Glycerol Stock"
                }
            },
            "EDGES": [
                ["DTBA_backboneA_splitAMP", "pyMOD-URA-URA3.A.1-pGPD-yeVenus-tCYC1"],
                ["T1MC_NatMX-Cassette_MCT2 (JV)", "pyMOD-URA-URA3.A.1-pGPD-yeVenus-tCYC1"],
                ["BBUT_URA3.A.0_homology1_UTP1 (from genome)", "pyMOD-URA-URA3.A.1-pGPD-yeVenus-tCYC1"],
                ["DH5alpha", "pyMOD-URA-URA3.A.1-pGPD-yeVenus-tCYC1"],
                ["TP-IRES-EGFP-TS", "pyMOD-URA-URA3.A.1-pGPD-yeVenus-tCYC1"]
            ]
        }
    ]
```

##### GLOBAL_CONSTRAINTS

This key specifies operations, items, or samples to avoid in the planning process. Here we avoid ordering addgene plasmids, performing matings, or using library cloning.

```
"GLOBAL_CONSTRAINTS": {
        "EXCLUDE": [
            {
                "model_class": "OperationType",
                "query": [
                    {"category": ["Control Blocks", "Library Cloning"]},
                    {"name": "Yeast Mating"},
                    {"name": "Yeast Auxotrophic Plate Mating"},
                    {"name": "Order Addgene Plasmid"}
                ]
            },
            {
                "model_class": "Item",
                "query": []
            }
        ]
    }
```

